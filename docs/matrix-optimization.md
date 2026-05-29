# Ejercicio 3 — Matrix y Optimización

## Matrix Strategy

La pipeline principal implementa una matriz multidimensional:

```yaml
test:
  strategy:
    fail-fast: false
    matrix:
      component: [frontend, backend]
      os: [ubuntu-latest, windows-latest]
      node-version: [18, 20]
      include:
        - os: ubuntu-latest
          node-version: "22"
          component: backend
      exclude:
        - os: windows-latest
          node-version: "18"
          component: frontend
```

### Combinaciones generadas

| # | Component | OS | Node |
|---|---|---|---|
| 1 | frontend | ubuntu-latest | 18 |
| 2 | frontend | ubuntu-latest | 20 |
| 3 | frontend | windows-latest | 20 |
| 4 | backend | ubuntu-latest | 18 |
| 5 | backend | ubuntu-latest | 20 |
| 6 | backend | ubuntu-latest | 22 (include) |
| 7 | backend | windows-latest | 18 |
| 8 | backend | windows-latest | 20 |

Total: **8 jobs** (se excluyó frontend + windows + node 18)

---

## Optimización

### Cache
- `actions/setup-node` con `cache: npm` cachea `node_modules` automáticamente
- `cache-dependency-path` apunta al `package-lock.json` del componente específico

### Paralelización
- `validate`, `test` y `compile` corren en paralelo independiente
- Matrix strategy ejecuta combinaciones en paralelo
- `fail-fast: false` evita cancelar jobs si uno falla

### Concurrency

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

- Agrupa ejecuciones por workflow + branch
- Cancela ejecuciones previdas del mismo grupo (ahorra recursos)
- Previene despliegues concurrentes sobre el mismo entorno

### Reutilización lógica
- Reusable workflows para testing, compilation y validations
- Shared GitHub Actions locales (`.github/actions/`) para lógica común
- Variables de entorno (`NODE_VERSION`) centralizadas

---

## Reporting

### Markdown summaries
El job `report` genera un resumen automático que GitHub muestra en la UI:

```markdown
# Pipeline Report

## Changed Components
- Frontend: true
- Backend: true
- Infrastructure: false
- Docs only: false

## Test Results
✅ All tests passed

## Build Status
✅ Builds successful

## Deployments
- Staging: success
```

### Artifacts
- Build artifacts: `build-<component>-<sha>` (subidos por reusable-compilation)
- Pipeline report: `pipeline-report-<sha>`
- Coverage reports (backend, vía Codecov)
