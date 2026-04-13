## Why

Los repositorios del proyecto carecen de un mecanismo centralizado y reutilizable para escaneos de calidad y seguridad de código con SonarCloud, lo que obliga a cada equipo a duplicar configuración y mantener pipelines inconsistentes. Centralizar el escaneo en un reusable workflow garantiza cobertura uniforme y reduce el esfuerzo de mantenimiento.

## What Changes

- Se crea un nuevo **reusable workflow** (`workflow-templates/sonarcloud-scan.yml`) que cualquier repositorio de la organización puede invocar mediante `workflow_call`.
- El workflow acepta inputs para indicar el tipo de lenguaje/ecosistema (`python`, `javascript`, `terraform`, `generic`) y parámetros opcionales de configuración.
- Se parametriza el token de SonarCloud y la organización/proyecto como secrets/inputs para que sea agnóstico al repositorio consumidor.
- Se incluye soporte para cacheo de dependencias según el ecosistema detectado (pip, npm, etc.).
- Se documenta el uso del workflow en `workflow-templates/sonarcloud-scan.properties.json` para integrarse con el catálogo de GitHub Actions.

## Capabilities

### New Capabilities

- `sonarcloud-reusable-scan`: Reusable workflow centralizado que ejecuta análisis estático con SonarCloud para múltiples ecosistemas (Python, JavaScript/TypeScript, Terraform, genérico), configurable vía inputs y secrets.

### Modified Capabilities

<!-- No hay capabilities existentes con requisitos cambiantes -->

## Impact

- **Nuevos archivos**: `workflow-templates/sonarcloud-scan.yml`, `workflow-templates/sonarcloud-scan.properties.json`
- **Repositorios consumidores**: deben proporcionar `SONAR_TOKEN` como secret y definir `sonar-project.properties` o pasar el project key como input.
- **Dependencias externas**: `SonarSource/sonarcloud-github-action`, `actions/checkout`, `actions/setup-python`, `actions/setup-node`.
- **Sin cambios breaking** para repositorios existentes; la adopción es opt-in.
