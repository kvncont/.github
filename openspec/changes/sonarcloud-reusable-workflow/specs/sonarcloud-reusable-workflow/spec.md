## ADDED Requirements

### Requirement: Workflow invocable como reusable workflow
El workflow SHALL ser definido con el trigger `on: workflow_call` para que pueda ser invocado desde otros repositorios de la organización.

#### Scenario: Invocación desde otro repositorio
- **WHEN** un workflow en otro repositorio usa `uses: kvncont/.github/.github/workflows/sonarcloud-scan.yml@main`
- **THEN** el workflow de escaneo se ejecuta en el contexto del repositorio llamante

### Requirement: Inputs parametrizables
El workflow SHALL aceptar los siguientes inputs:
- `language` (requerido): Lenguaje del proyecto. Valores válidos: `python`, `javascript`, `terraform`, `generic`
- `sonar_organization` (requerido): Clave de organización en SonarCloud
- `sonar_project_key` (requerido): Clave del proyecto en SonarCloud

#### Scenario: Input de lenguaje válido
- **WHEN** el workflow es invocado con `language: python`
- **THEN** el workflow ejecuta los pasos de instalación de dependencias Python antes del análisis

#### Scenario: Input de lenguaje no soportado
- **WHEN** el workflow es invocado con un valor de `language` no reconocido
- **THEN** el workflow ejecuta el análisis genérico sin instalación de dependencias específicas

### Requirement: Secret SONAR_TOKEN requerido
El workflow SHALL requerir el secret `sonar_token` para autenticarse con SonarCloud.

#### Scenario: Secret presente
- **WHEN** el repositorio consumidor pasa `secrets: sonar_token: ${{ secrets.SONAR_TOKEN }}`
- **THEN** el análisis se ejecuta con autenticación correcta

#### Scenario: Secret ausente
- **WHEN** el secret `sonar_token` no es proporcionado
- **THEN** el paso de análisis falla con error de autenticación

### Requirement: Preparación de dependencias por lenguaje
El workflow SHALL instalar dependencias específicas del lenguaje antes de ejecutar el scanner de SonarCloud.

#### Scenario: Python - instalación de dependencias
- **WHEN** `language` es `python` y existe un archivo `requirements.txt`
- **THEN** el workflow ejecuta `pip install -r requirements.txt` antes del análisis

#### Scenario: JavaScript - instalación de dependencias
- **WHEN** `language` es `javascript` y existe un archivo `package.json`
- **THEN** el workflow ejecuta `npm ci` antes del análisis

#### Scenario: Terraform - sin instalación de dependencias
- **WHEN** `language` es `terraform`
- **THEN** el workflow ejecuta el análisis directamente sin pasos adicionales de instalación

#### Scenario: Generic - sin instalación de dependencias
- **WHEN** `language` es `generic`
- **THEN** el workflow ejecuta el análisis directamente sin pasos adicionales de instalación

### Requirement: Análisis de código con SonarCloud
El workflow SHALL ejecutar el análisis de SonarCloud usando la action oficial `SonarSource/sonarcloud-github-action`.

#### Scenario: Análisis exitoso
- **WHEN** el workflow se ejecuta con credenciales válidas y el proyecto existe en SonarCloud
- **THEN** el análisis se completa y los resultados aparecen en el dashboard de SonarCloud y en el PR (si aplica)

#### Scenario: Análisis en Pull Request
- **WHEN** el workflow es disparado por un evento de Pull Request
- **THEN** SonarCloud agrega un comentario con el resumen del análisis en el PR
