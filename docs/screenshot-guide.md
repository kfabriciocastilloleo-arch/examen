# Guía de Capturas para el Examen

Cada ejercicio indica qué archivo y líneas capturar.

---

## Ejercicio 1 — Arquitectura pipeline enterprise

### Monorepo
```
Captura:   tree de carpetas raíz
Archivos:  frontend/, backend/, infrastructure/, docs/
```

### Selective execution
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    42-53   → dorny/paths-filter (detecta cambios por componente)
Líneas:    55-73   → Build dynamic matrix (genera JSON array solo con changed components)
Líneas:    78      → has-code-changes flag (controla si los jobs se ejecutan)
```

### Reusable workflows
```
Archivo:   .github/workflows/reusable-testing.yml     (completo)
Archivo:   .github/workflows/reusable-compilation.yml  (completo)
Archivo:   .github/workflows/reusable-validations.yml  (completo)
```

### Pipeline principal (flujo completo)
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Capturar flujo completo: detect-changes → validate → test → compile → deploy
```

---

## Ejercicio 2 — Seguridad enterprise

### Permissions mínimos globales
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    9-12     → permissions globales (contents: read, pull-requests: write, checks: write)
```

### Permissions específicos por job
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    25-27   → detect-changes: solo contents:read, pull-requests:read
Líneas:    159-161 → report: contents:read, checks:write
```

### Version pinning (actions con SHA/version exacta)
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    37      → actions/checkout@v4.2.2
Líneas:    122     → actions/download-artifact@v4.1.9
Líneas:    200     → actions/upload-artifact@v4.6.1
Líneas:    42      → dorny/paths-filter@v3.0.2

Archivo:   .github/workflows/reusable-testing.yml
Líneas:    38      → actions/checkout@v4.2.2
Líneas:    43      → actions/setup-node@v4.1.0
Líneas:    60      → codecov/codecov-action@v5.3.1

Archivo:   .github/workflows/reusable-compilation.yml
Líneas:    34      → actions/checkout@v4.2.2
Líneas:    39      → actions/setup-node@v4.1.0
Líneas:    54      → actions/upload-artifact@v4.6.1
```

### Supply chain
```
Archivo:   .github/workflows/reusable-testing.yml   → línea 50: npm ci
Archivo:   .github/workflows/ci-cd-pipeline.yml      → línea 38: persist-credentials: false
```

### Secrets
```
Documento: docs/setup-guide.md   → tabla completa de secrets creados
Documento: docs/security-enterprise.md   → sección "Secrets" (separación repository/org/environment)
```

### Environments (staging + production)
```
Archivo:   infrastructure/environments/staging.yml    → configuración staging
Archivo:   infrastructure/environments/production.yml → configuración production
Documento: docs/security-enterprise.md                → sección "Environments"
```

### OIDC
```
Documento: docs/security-enterprise.md   → sección "OIDC" completa
```

---

## Ejercicio 3 — Matrix y optimización

### Matrix multi-platform + multi-runtime
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    87-96   → test job: matrix 3D (component × os × node-version)
Líneas:    92      → fail-fast: false
```

### Dynamic matrix (fromJSON)
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    55-73   → Build dynamic matrix step
Líneas:    75      → ${{ fromJSON(needs.detect-changes.outputs.changed-components) }}
```

### Cache
```
Archivo:   .github/workflows/reusable-testing.yml
Líneas:    46-47   → cache con setup-node + cache-dependency-path
```

### Concurrency
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    14-16   → concurrency group + cancel-in-progress
```

### Reporting
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    158-192 → report job con GITHUB_STEP_SUMMARY
Líneas:    194-200 → upload-artifact del report
```

---

## Ejercicio 4 — Self-hosted runners

### runs-on: self-hosted
```
Archivo:   .github/workflows/ci-cd-pipeline.yml
Líneas:    125     → deploy-staging: runs-on: self-hosted
Líneas:    143     → deploy-production: runs-on: self-hosted
```

### Documentación completa
```
Documento: docs/self-hosted-runners.md
  → Tabla "¿Cuándo usar self-hosted?"
  → Riesgos de seguridad
  → Segmentación con labels
  → Estrategia de organización
  → Ventajas/Desventajas
```

---

## Ejercicio 5 — Troubleshooting

### Caso A: Pipeline no ejecuta deploy
```
Documento: docs/troubleshooting.md
  → Sección "Caso A"
  → Causas, Diagnóstico, Solución
```

### Caso B: Matrix genera más jobs de los esperados
```
Documento: docs/troubleshooting.md
  → Sección "Caso B"
  → Ejemplo problemático, explicación, solución
```

### Caso C: Reusable workflow no recibe outputs
```
Documento: docs/troubleshooting.md
  → Sección "Caso C"
  → Causas, Diagnóstico, Solución con ejemplos
```
