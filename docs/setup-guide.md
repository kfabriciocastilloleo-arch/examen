# Setup Guide — Configuración necesaria en GitHub

## 1. Secrets (Settings > Secrets and variables > Actions)

### Repository secrets

| Secret | Valor | Uso |
|---|---|---|
| `CODECOV_TOKEN` | Token de Codecov | Subir cobertura en reusable-testing.yml |

### Environment secrets

Crear environments y sus secrets:

**Staging:**
```
Settings > Environments > staging
  - STAGING_API_KEY
```

**Production:**
```
Settings > Environments > production
  - PROD_API_KEY
  - ✓ Required reviewers (devops-leads, security-team)
  - ✓ Wait timer (10 minutes)
  - ✓ Deployment branches (main)
```

---

## 2. Variables (Settings > Secrets and variables > Actions)

| Variable | Valor | Uso |
|---|---|---|
| `NODE_VERSION` | `20` | Versión de Node por defecto |

---

## 3. Environments (Settings > Environments)

### Staging

```yaml
name: staging
url: https://staging.example.com
deployment_branch: main
protected: false
```

### Production

```yaml
name: production
url: https://production.example.com
deployment_branch: main
protected: true
required_reviewers:
  - devops-leads
  - security-team
wait_timer: 10
```

---

## 4. Self-hosted runners (Settings > Actions > Runners)

Configurar runners con labels:

| Runner Group | Labels | Repos |
|---|---|---|
| production | `self-hosted, linux, prod` | examen |
| staging | `self-hosted, linux, staging` | examen |

---

## 5. Branch protection (Settings > Branches > Add rule)

- Branch: `main`
- ✓ Require pull request reviews
- ✓ Require status checks (ci-cd-pipeline)
- ✓ Require conversation resolution
