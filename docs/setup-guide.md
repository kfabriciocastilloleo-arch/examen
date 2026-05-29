# Setup Guide — Configuración en GitHub

## ✅ Ya creado vía API

| Recurso | Nombre | Ámbito |
|---|---|---|
| Secret | `CODECOV_TOKEN` | Repositorio |
| Secret | `STAGING_API_KEY` | Environment staging |
| Secret | `PROD_API_KEY` | Environment production |
| Variable | `NODE_VERSION = 20` | Repositorio |
| Environment | `staging` | branch_policy: main |
| Environment | `production` | wait_timer: 10min, branch_policy: main |

## ⚠️ Pendiente de crear manualmente

### 1. Required reviewers (production)
```
Settings > Environments > production > Protection rules
  → Add "Required reviewers"
  → Seleccionar usuarios/teams (ej: devops-leads, security-team)
```

### 2. Self-hosted runners
```
Settings > Actions > Runners > Add runner
  → Crear grupos: production (label: prod), staging (label: staging)
  → Ambos con label: self-hosted, linux
```

### 3. Branch protection (main)
```
Settings > Branches > Add rule
  → Branch: main
  → ✓ Require a pull request before merging
  → ✓ Require status checks (ci-cd-pipeline)
```
