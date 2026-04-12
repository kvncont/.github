## 1. Creación del Reusable Workflow

- [ ] 1.1 Crear el archivo `.github/workflows/sonarcloud-scan.yml` con el trigger `on: workflow_call`
- [ ] 1.2 Definir los inputs requeridos: `language`, `sonar_organization`, `sonar_project_key`
- [ ] 1.3 Definir el secret requerido: `sonar_token`

## 2. Lógica de Preparación por Lenguaje

- [ ] 2.1 Agregar paso condicional para Python: instalar dependencias con `pip install -r requirements.txt`
- [ ] 2.2 Agregar paso condicional para JavaScript: instalar dependencias con `npm ci`
- [ ] 2.3 Verificar que Terraform y Generic no requieren pasos adicionales

## 3. Integración con SonarCloud

- [ ] 3.1 Agregar el paso de checkout del repositorio (`actions/checkout`)
- [ ] 3.2 Configurar y agregar el paso de análisis con `SonarSource/sonarcloud-github-action`
- [ ] 3.3 Pasar `SONAR_TOKEN`, `sonar.organization` y `sonar.projectKey` al análisis

## 4. Documentación

- [ ] 4.1 Actualizar el `README.md` con instrucciones de uso del reusable workflow y ejemplos por lenguaje
