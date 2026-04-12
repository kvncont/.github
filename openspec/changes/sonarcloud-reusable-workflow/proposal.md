## Why

Los equipos de desarrollo necesitan una forma estandarizada de ejecutar análisis de calidad y seguridad de código con SonarCloud en múltiples lenguajes (Python, JavaScript, Terraform, etc.) sin duplicar configuración en cada repositorio. Un reusable workflow centraliza la lógica de escaneo y reduce la carga de mantenimiento.

## What Changes

- Se crea un nuevo GitHub Actions reusable workflow en `.github/workflows/sonarcloud-scan.yml`
- El workflow acepta parámetros de entrada para lenguaje, organización de SonarCloud y clave del proyecto
- Soporta múltiples ecosistemas: Python, JavaScript/TypeScript, Terraform y proyectos genéricos
- Gestiona la instalación de dependencias específicas por lenguaje antes del análisis
- Expone el token de SonarCloud como secret requerido para autenticación segura

## Capabilities

### New Capabilities
- `sonarcloud-reusable-workflow`: Reusable workflow de GitHub Actions que encapsula la lógica de escaneo de SonarCloud, parametrizable por lenguaje y proyecto, reutilizable desde cualquier repositorio de la organización

### Modified Capabilities
<!-- No hay capabilities existentes que cambien sus requisitos -->

## Impact

- **Nuevo archivo**: `.github/workflows/sonarcloud-scan.yml`
- **Dependencias externas**: acción `SonarSource/sonarcloud-github-action@master`
- **Secretos requeridos**: `SONAR_TOKEN` (debe configurarse en los repositorios consumidores)
- **Repositorios afectados**: cualquier repositorio de la organización que quiera adoptar el workflow de escaneo
