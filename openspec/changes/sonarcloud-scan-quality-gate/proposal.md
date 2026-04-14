## Why

Los repositorios del organización necesitan un mecanismo centralizado para ejecutar análisis de calidad de código con SonarCloud y validar el Quality Gate, asegurando que ningún cambio degradue la calidad del código sin ser detectado. La reutilización a través de un reusable workflow evita duplicación de configuración en cada repositorio.

## What Changes

- Nuevo reusable workflow `sonarcloud-scan-quality-gate.yml` en `.github/workflows/` que permite a cualquier repositorio de la organización ejecutar el escaneo de SonarCloud y validar el Quality Gate de forma estandarizada.
- El workflow acepta parámetros configurables (organización SonarCloud, project key, token) mediante inputs y secrets.
- Incluye paso de checkout con fetch-depth completo (requerido por SonarCloud para análisis de blame).
- Valida el Quality Gate tras el escaneo y falla el workflow si no se supera.

## Capabilities

### New Capabilities

- `sonarcloud-scan-quality-gate`: Reusable workflow que ejecuta el escaneo de SonarCloud sobre el código fuente y valida que el Quality Gate sea aprobado, fallando el job si no lo es.

### Modified Capabilities

## Impact

- Se agrega un nuevo archivo en `.github/workflows/sonarcloud-scan-quality-gate.yml`.
- Los repositorios consumidores deben tener configurado el secret `SONAR_TOKEN` y conocer su `project key` y `organization` en SonarCloud.
- Sin impacto en workflows existentes.
