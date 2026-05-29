# Ejercicio 5 — Troubleshooting

## Caso A: Pipeline no ejecuta deploy aunque tests son correctos

### Posibles causas

1. **Condicionales incorrectos** (`if:` mal escrito)
   ```yaml
   # Error: el deploy depende de una condición que nunca se cumple
   if: github.ref == 'refs/heads/main'
   ```

2. **Dependencias fallidas**
   - El job deploy depende de `needs: [compile]`, pero compile nunca se ejecutó
   - Algun job intermedio tiene `if: false`

3. **Environment protection rules**
   - El environment `production` tiene `required_reviewers` y nadie aprobó
   - El environment tiene `deployment_branch` configurado a otra rama

4. **Path filtering excluyó cambios**
   - `paths-filter` detectó que solo se modificaron `docs/`, por lo tanto `deploy` tiene `if: false`

5. **Concurrency canceló el job**
   - Un deploy anterior del mismo grupo de concurrencia canceló el actual

### Diagnóstico

| Herramienta | Qué revisar |
|---|---|
| GitHub UI | Ir a Actions > workflow run > job > step logs |
| `needs` context | Revisar `needs.<job-id>.result` en cada condicional |
| Environment tab | Settings > Environments > production |
| Branch protection | Settings > Branches > main |

### Solución

1. Revisar logs del workflow en la UI de GitHub Actions
2. Verificar la evaluación de expresiones condicionales:
   ```yaml
   if: github.ref == 'refs/heads/main'  # Asegurar doble ==
   ```
3. Confirmar que el environment de producción acepta deploys desde `main`
4. Verificar que los reviewers aprobaron el deployment
5. Agregar logging temporal:
   ```yaml
   - name: Debug conditions
     run: |
       echo "Branch: ${{ github.ref }}"
       echo "Event: ${{ github.event_name }}"
       echo "Test result: ${{ needs.test.result }}"
   ```

---

## Caso B: Una matrix genera más jobs de los esperados

### Ejemplo problemático

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20]
    component: [frontend, backend]
```

Esto genera: 2 × 2 × 2 = **8 jobs**, cuando quizás solo se esperaban 4.

### ¿Por qué ocurre?

- El producto cartesiano de todas las dimensiones se ejecuta completo
- No se usaron `include`/`exclude` para limitar combinaciones
- Se agregaron dimensiones sin considerar el total resultante

### Identificar errores

1. Calcular el producto: `|os| × |node| × |component|`
2. Revisar si se necesita una dependencia entre dimensiones (ej: backend no necesita Windows+Node 18)
3. Buscar `include`/`exclude` faltantes

### Solución

Usar `exclude` para eliminar combinaciones no deseadas:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20]
    component: [frontend, backend]
    exclude:
      - os: windows-latest
        node: 18
        component: frontend
      - os: windows-latest
        component: backend
```

O mejor, usar una matriz más controlada:

```yaml
strategy:
  matrix:
    component: [frontend, backend]
    os: [ubuntu-latest]
    node: [20]
    include:
      - component: backend
        os: windows-latest
        node: 18
```

En este caso solo se ejecutan **3 jobs**: los dos por defecto + el incluido explícitamente.

---

## Caso C: Un reusable workflow no recibe correctamente outputs

### Posibles causas

1. **Outputs no declarados en el `workflow_call`**
   ```yaml
   # Error: falta declarar outputs en el workflow llamado
   on:
     workflow_call:
       inputs:
         component:
           required: true
   # outputs:  ← FALTA ESTA SECCIÓN
   ```

2. **Output scope incorrecto**
   - Los outputs se definen a nivel `job.output`, no a nivel `step`
   - El output del step no está accesible fuera del job sin `job.output`

3. **Referencia incorrecta desde el caller**
   ```yaml
   # Error: referencia incorrecta
   needs: resultado  # debería ser: needs.test.outputs.test-status
   ```

4. **Output no es serializable**
   - Intentar pasar un objeto complejo como output string

### Diagnóstico

1. Verificar que el workflow llamado tiene:
   ```yaml
   on:
     workflow_call:
       outputs:
         my-output:
           description: "..."
           value: ${{ jobs.my-job.outputs.my-output }}
   ```

2. Verificar que el job dentro del reusable declara:
   ```yaml
   jobs:
     my-job:
       outputs:
         my-output: ${{ steps.my-step.outcome }}
   ```

3. Verificar que el caller referencia correctamente:
   ```yaml
   jobs:
     caller-job:
       uses: ./.github/workflows/reusable.yml
       needs: [other-job]
       # Los outputs se acceden como: needs.caller-job.outputs.my-output
   ```

### Solución completa

**Reusable workflow (`reusable.yml`):**
```yaml
on:
  workflow_call:
    outputs:
      test-status:
        value: ${{ jobs.test.outputs.status }}
        description: "Test result"

jobs:
  test:
    outputs:
      status: ${{ steps.run-tests.outcome }}
    steps:
      - id: run-tests
        run: echo "Running tests..."
```

**Caller workflow:**
```yaml
jobs:
  reusable-job:
    uses: ./.github/workflows/reusable.yml

  deploy:
    needs: [reusable-job]
    if: needs.reusable-job.outputs.test-status == 'success'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```
