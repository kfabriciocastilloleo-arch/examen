# Ejercicio 2 — Seguridad Enterprise

## Permissions

### Mínimos globales
En el workflow principal se definen permisos mínimos de solo lectura:

```yaml
permissions:
  contents: read
  pull-requests: write
  checks: write
```

### Específicos por job
Cada job declara únicamente los permisos que necesita:

```yaml
detect-changes:
  permissions:
    contents: read
    pull-requests: read

report:
  permissions:
    contents: read
    checks: write
```

---

## Actions externas

### Version Pinning
Todas las actions se referencian con SHA o versión exacta:

| Action | Version |
|---|---|
| `actions/checkout` | `v4.2.2` |
| `actions/setup-node` | `v4.1.0` |
| `actions/upload-artifact` | `v4.6.1` |
| `actions/download-artifact` | `v4.1.9` |
| `dorny/paths-filter` | `v3.0.2` |
| `codecov/codecov-action` | `v5.3.1` |

### Supply chain
- Usar `npm ci` en lugar de `npm install` para instalaciones determinísticas
- Verificar hash de acciones descargadas mediante firmas (cuando estén disponibles)

---

## Secrets

### Separación de scopes

| Scope | Secret | Uso |
|---|---|---|
| Repository | `CODECOV_TOKEN` | Cobertura de código |
| Environment (staging) | `STAGING_API_KEY` | Despliegue a staging |
| Environment (production) | `PROD_API_KEY` | Despliegue a producción |
| Organization | `OIDC_IAM_ROLE` | Autenticación entre repos |

### Reglas de scope
- Repository: secrets compartidos entre workflows del mismo repo
- Environment: secrets específicos por entorno de despliegue
- Organization: secrets globales para múltiples repos

---

## Environments

### Configuración

**Staging:**
- Despliegue automático desde `main`
- Sin aprobación manual
- URL: https://staging.example.com

**Production:**
- Despliegue desde `main` con protección
- **Requiere aprobación** de al menos un reviewer de `devops-leads` o `security-team`
- **Límite de despliegues**: 1 despliegue concurrente
- URL: https://production.example.com
- Tiempo de espera entre despliegues: 10 minutos

### Ejemplo de configuración en GitHub

```
Settings > Environments > production
  ✓ Required reviewers (devops-leads, security-team)
  ✓ Wait timer (10 minutes)
  ✓ Deployment branches (main)
```

---

## OIDC (OpenID Connect)

### ¿Qué problema resuelve?
Tradicionalmente, para que GitHub Actions acceda a servicios cloud (AWS, Azure, GCP), se almacenan credenciales de larga duración (access keys) como secrets. Esto tiene varios problemas:
- Rotación manual de claves
- Riesgo de exposición en logs
- Dificultad para auditar qué workflow usó qué credencial

OIDC permite que GitHub Actions se autentique directamente contra el proveedor cloud **sin almacenar credenciales estáticas**.

### ¿Cómo mejora la seguridad?
1. **Sin secrets estáticos**: No hay access keys que rotar o que puedan filtrarse
2. **Autenticación efímera**: Cada ejecución genera un token temporal (válido por pocos minutos)
3. **Auditabilidad**: El cloud provider registra exactamente qué workflow solicitó el token
4. **Trust federation**: GitHub emite un JWT firmado que el cloud provider valida

### ¿Cuándo utilizarlo?
- Siempre que se despliegue a AWS, Azure o GCP desde GitHub Actions
- En pipelines que necesiten acceso a recursos cloud (S3, Lambda, EKS, etc.)
- En organizaciones que necesiten cumplir con SOC2, ISO 27001 o similares
- Cuando se quiera eliminar la gestión manual de access keys

### Ejemplo de configuración

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1

      - name: Deploy to ECS
        run: aws ecs update-service --cluster prod --service app --force-new-deployment
```
