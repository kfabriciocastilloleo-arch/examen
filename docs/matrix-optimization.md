# Ejercicio 3 — Matrix y Optimización

## Matrix Strategy

### Dynamic matrix con selective execution

La pipeline usa `fromJSON` para generar la matriz dinámicamente según los componentes modificados:

```yaml
test:
  strategy:
    fail-fast: false
    matrix:
      component: ${{ fromJSON(needs.detect-changes.outputs.changed-components) }}
      os: [ubuntu-latest, windows-latest]
      node-version: ["18", "20", "22"]
```

Si `detect-changes` detecta cambios solo en `frontend`, la matriz se resuelve como:

```yaml
matrix:
  component: [frontend]
  os: [ubuntu-latest, windows-latest]
  node-version: ["18", "20", "22"]
```

Generando **6 jobs**: frontend × (ubuntu, windows) × (18, 20, 22)

### Include / Exclude (demostración conceptual)

GitHub Actions también permite control manual:

```yaml
matrix:
  component: [frontend, backend]
  os: [ubuntu-latest, windows-latest]
  node: [18, 20]
  include:
    - os: ubuntu-latest
      node: "22"
      component: backend
  exclude:
    - os: windows-latest
      node: "18"
      component: frontend
```

Esto generaría 8 combinaciones específicas.

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
- Cancela ejecuciones previas del mismo grupo
- Previene despliegues concurrentes sobre el mismo entorno

### Reutilización lógica
- Reusable workflows para testing, compilation y validations
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
Tests passed

## Build Status
Builds successful

## Deployments
- Staging: success
```

### Artifacts
- Build artifacts: `build-<component>-<sha>` (subidos por reusable-compilation)
- Pipeline report: `pipeline-report-<sha>`
- Coverage reports (backend, vía Codecov)
