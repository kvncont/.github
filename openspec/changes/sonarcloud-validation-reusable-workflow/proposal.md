## Why

Los repositorios del proyecto necesitan validar la calidad del código mediante SonarCloud, pero actualmente no existe un mecanismo centralizado y reutilizable para ejecutar y validar los escaneos. Esto provoca duplicación de configuración en cada repositorio y dificulta garantizar estándares de calidad uniformes.

## What Changes

- Se crea un **reusable workflow** (`sonarcloud-validation.yml`) en `.github/workflows/` dentro del repositorio `.github` de la organización, que puede ser llamado por cualquier repositorio de `kvncont`.
- El workflow acepta inputs configurables: nombre del proyecto en SonarCloud, organización, y opciones de Quality Gate.
- El workflow valida que el escaneo de SonarCloud pase el Quality Gate y falla el job si no se cumplen los umbrales de calidad.
- Se expone como workflow reutilizable usando `workflow_call` para que cualquier repo lo pueda referenciar.

## Capabilities

### New Capabilities

- `sonarcloud-validation`: Reusable workflow que ejecuta y valida los escaneos de SonarCloud contra el Quality Gate, configurable por proyecto y organización.

### Modified Capabilities

*(ninguno — no hay specs existentes que cambien)*

## Impact

- **Workflows**: Se agrega `.github/workflows/sonarcloud-validation.yml` como workflow reutilizable en la organización.
- **Dependencias externas**: Requiere el action oficial `SonarSource/sonarqube-scan-action` y acceso al token de SonarCloud via secrets.
- **Repositorios consumidores**: Cualquier repo de `kvncont` podrá referenciar este workflow con `uses: kvncont/.github/.github/workflows/sonarcloud-validation.yml@main`.
- **Secrets**: Se necesita el secret `SONAR_TOKEN` disponible en los repositorios que lo consuman.
