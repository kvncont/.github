## ADDED Requirements

### Requirement: Invocación como reusable workflow
El workflow `sonarcloud-scan.yml` SHALL ser invocable mediante `workflow_call` desde cualquier repositorio de la organización, sin necesidad de duplicar lógica de escaneo.

#### Scenario: Repositorio llama al workflow con inputs mínimos
- **WHEN** un repositorio define un job con `uses: <org>/.github/.github/workflow-templates/sonarcloud-scan.yml@main` y proporciona `sonar-organization` y `sonar-project-key`
- **THEN** el workflow ejecuta el escaneo de SonarCloud y publica los resultados en el dashboard del proyecto

#### Scenario: Workflow omite input requerido
- **WHEN** el repositorio llamador omite `sonar-organization` o `sonar-project-key`
- **THEN** GitHub rechaza la ejecución con un error de validación de inputs antes de iniciar ningún job

---

### Requirement: Soporte multi-ecosistema vía input `language`
El workflow SHALL aceptar un input `language` que determine el setup del entorno de ejecución previo al escaneo.

#### Scenario: Escaneo de proyecto Python
- **WHEN** el llamador pasa `language: python`
- **THEN** el workflow ejecuta `actions/setup-python` con la versión especificada por `python-version` y luego lanza el escaneo

#### Scenario: Escaneo de proyecto JavaScript/TypeScript
- **WHEN** el llamador pasa `language: javascript`
- **THEN** el workflow ejecuta `actions/setup-node` con la versión especificada por `node-version` y luego lanza el escaneo

#### Scenario: Escaneo de proyecto Terraform
- **WHEN** el llamador pasa `language: terraform`
- **THEN** el workflow omite el setup de runtime y lanza directamente el escaneo (Sonar Scanner CLI soporta HCL nativamente)

#### Scenario: Escaneo en modo genérico
- **WHEN** el llamador pasa `language: generic` o no especifica el input (valor por defecto)
- **THEN** el workflow omite cualquier setup de runtime y ejecuta el escaneo con la configuración base de SonarCloud

#### Scenario: Valor de `language` no reconocido
- **WHEN** el llamador pasa un valor de `language` distinto de `python`, `javascript`, `terraform` o `generic`
- **THEN** el workflow falla en el paso de validación con un mensaje de error descriptivo

---

### Requirement: Gestión segura del token de SonarCloud
El workflow SHALL requerir el secret `SONAR_TOKEN` y SHALL utilizarlo exclusivamente para autenticarse con SonarCloud.

#### Scenario: Token disponible vía secrets heredados
- **WHEN** el repositorio llamador tiene `SONAR_TOKEN` definido en sus secrets y el workflow se invoca con `secrets: inherit`
- **THEN** el escaneo se autentica correctamente con SonarCloud sin que el token quede expuesto en los logs

#### Scenario: Token ausente
- **WHEN** `SONAR_TOKEN` no está definido en el repositorio llamador
- **THEN** el paso de escaneo falla con un error de autenticación de SonarCloud

---

### Requirement: Configuración opcional de parámetros de escaneo
El workflow SHALL aceptar inputs opcionales para personalizar el escaneo sin requerir un archivo `sonar-project.properties` en el repositorio.

#### Scenario: Llamador define directorio de fuentes
- **WHEN** el llamador pasa `sonar-sources: src/`
- **THEN** el escaneo analiza únicamente el directorio `src/`

#### Scenario: Llamador define exclusiones
- **WHEN** el llamador pasa `sonar-exclusions: '**/tests/**,**/*.spec.ts'`
- **THEN** el escaneo omite los archivos que coincidan con los patrones indicados

#### Scenario: Llamador proporciona reporte de cobertura
- **WHEN** el llamador pasa `coverage-report-path: coverage/lcov.info`
- **THEN** el escaneo importa el reporte de cobertura y lo publica en SonarCloud

#### Scenario: Llamador no define ningún input opcional
- **WHEN** el llamador solo proporciona los inputs requeridos
- **THEN** el workflow usa valores por defecto razonables (fuentes en `.`, sin exclusiones, sin cobertura)

---

### Requirement: Publicación como template de organización
El workflow SHALL estar acompañado de un archivo `sonarcloud-scan.properties.json` que lo registre en el catálogo de workflow templates de la organización.

#### Scenario: Acceso al template desde la UI de GitHub
- **WHEN** un administrador de un repositorio accede a "Actions > New workflow" en la organización
- **THEN** ve el template "SonarCloud Scan" disponible para seleccionar

#### Scenario: Metadata del template es correcta
- **WHEN** GitHub lee el archivo `sonarcloud-scan.properties.json`
- **THEN** obtiene nombre, descripción, categorías y autor del workflow correctamente formateados según el esquema de GitHub
