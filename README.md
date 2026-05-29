# Examen — Arquitectura DevOps Enterprise

Monorepo con pipelines CI/CD enterprise, seguridad, optimización y troubleshooting.

## Estructura

```
/
├── frontend/              # Aplicación frontend
├── backend/               # API backend
├── infrastructure/        # Terraform y config de entornos
│   ├── environments/      # Config de staging y production
│   └── terraform/         # Infraestructura como código
├── docs/                  # Documentación del examen
│   ├── README.md
│   ├── security-enterprise.md
│   ├── matrix-optimization.md
│   ├── self-hosted-runners.md
│   └── troubleshooting.md
└── .github/
    ├── workflows/
    │   ├── ci-cd-pipeline.yml          # Pipeline principal
    │   ├── reusable-testing.yml         # Reusable: testing
    │   ├── reusable-compilation.yml     # Reusable: compilación
    │   └── reusable-validations.yml     # Reusable: validaciones
    └── actions/                         # Acciones compartidas
```

## Ejercicios

1. **Arquitectura pipeline enterprise** — Monorepo, selective execution, reusable workflows
2. **Seguridad enterprise** — Permissions, actions pinning, secrets, environments, OIDC
3. **Matrix y optimización** — Multi-plataforma, cache, parallelización, reporting
4. **Self-hosted runners** — Estrategia, riesgos, segmentación, labels
5. **Troubleshooting** — Análisis de pipelines defectuosos
