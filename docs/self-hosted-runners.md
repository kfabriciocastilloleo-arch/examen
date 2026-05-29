# Ejercicio 4 — Self-Hosted Runners

## ¿Cuándo usar self-hosted?

| Situación | Recomendación |
|---|---|
| Necesitas hardware específico (GPU, ARM) | Self-hosted |
| Costos elevados de runners GitHub | Self-hosted |
| Requisitos de compliance/data sovereignty | Self-hosted |
| Equipos pequeños, pocos builds | GitHub-hosted |
| Escalado automático necesario | GitHub-hosted |

## Riesgos de seguridad

1. **Superficie de ataque**: El runner ejecuta código arbitrario del workflow
2. **Aislamiento insuficiente**: Si un workflow malicioso accede al sistema host
3. **Secrets en memoria**: Los secrets del workflow quedan en el runner
4. **Network exposure**: Los runners necesitan acceso saliente a GitHub

## Mitigaciones

- No poner secrets del entorno en el runner, solo pasarlos desde GitHub
- Usar runners efímeros (autoescalado con containers)
- Segmentar runners por equipo/proyecto con labels
- Aplicar parches de seguridad regularmente

## Segmentación con labels

```yaml
runs-on: [self-hosted, linux, gpu]
```

Labels recomendados:
- `linux` / `windows` / `macos`
- `gpu` / `arm` / `high-mem`
- `prod` / `staging` / `build`
- `team-a` / `team-b`

## Estrategia de organización

```
Organización
├── Runner Group: production
│   ├── Label: prod
│   └── Repos autorizados: examen
├── Runner Group: staging
│   ├── Label: staging
│   └── Repos autorizados: examen
└── Runner Group: general
    ├── Labels: linux, windows, high-mem
    └── Repos autorizados: todos
```

## Ventajas

| Aspecto | Ventaja |
|---|---|
| Costo | Sin costo por minuto de ejecución |
| Hardware | Control total (CPU, RAM, GPU, storage) |
| Latencia | Red local sin salida a internet |
| Compliance | Datos nunca salen de la infraestructura |

## Desventajas

| Aspecto | Desventaja |
|---|---|
| Mantenimiento | Actualizaciones, parches, monitoreo |
| Escalado | No escala automáticamente (requiere auto-scaling) |
| Disponibilidad | Responsabilidad del equipo mantenerlos operativos |
| Seguridad | Mayor superficie de ataque que GitHub-hosted |

## Ejemplo runs-on

```yaml
jobs:
  deploy-prod:
    runs-on: [self-hosted, linux, prod]
    environment: production
    steps:
      - run: echo "Deploying to production on self-hosted runner"

  train-model:
    runs-on: [self-hosted, linux, gpu]
    steps:
      - run: echo "Training ML model on GPU runner"
```
