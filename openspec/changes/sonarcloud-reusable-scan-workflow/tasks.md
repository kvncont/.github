## 1. Estructura de archivos y scaffolding

- [ ] 1.1 Crear el archivo `workflow-templates/sonarcloud-scan.yml` con el esqueleto del reusable workflow (`on: workflow_call`)
- [ ] 1.2 Crear el archivo `workflow-templates/sonarcloud-scan.properties.json` con la metadata del template (nombre, descripción, categorías, iconografía)

## 2. Definición de inputs y secrets del workflow

- [ ] 2.1 Declarar inputs requeridos: `sonar-organization`, `sonar-project-key`
- [ ] 2.2 Declarar input `language` con `default: generic` y descripción de valores aceptados (`python`, `javascript`, `terraform`, `generic`)
- [ ] 2.3 Declarar inputs opcionales: `sonar-sources` (default `.`), `sonar-exclusions` (default `''`), `coverage-report-path` (default `''`)
- [ ] 2.4 Declarar inputs opcionales de runtime: `python-version` (default `'3.x'`), `node-version` (default `'20'`)
- [ ] 2.5 Declarar `secrets: inherit` para heredar `SONAR_TOKEN` del repositorio llamador

## 3. Job principal de escaneo

- [ ] 3.1 Definir el job `sonarcloud-scan` que corre en `ubuntu-latest`
- [ ] 3.2 Agregar el paso `actions/checkout@v4` con `fetch-depth: 0` (requerido por SonarCloud para análisis de historial)
- [ ] 3.3 Agregar paso condicional `actions/setup-python` (`if: inputs.language == 'python'`) con `python-version: ${{ inputs.python-version }}`
- [ ] 3.4 Agregar paso condicional `actions/setup-node` (`if: inputs.language == 'javascript'`) con `node-version: ${{ inputs.node-version }}`
- [ ] 3.5 Agregar paso de validación de `language` que falle con mensaje claro si el valor no es uno de los cuatro permitidos

## 4. Integración con SonarCloud

- [ ] 4.1 Agregar el paso `SonarSource/sonarcloud-github-action@v3` (pinear a versión estable, no `@master`)
- [ ] 4.2 Configurar `env.GITHUB_TOKEN` y `env.SONAR_TOKEN` en el paso de escaneo usando los secrets heredados
- [ ] 4.3 Pasar los `args` de Sonar Scanner con los inputs del workflow: `-Dsonar.organization`, `-Dsonar.projectKey`, `-Dsonar.sources`, `-Dsonar.exclusions`
- [ ] 4.4 Agregar argumento condicional `-Dsonar.python.coverage.reportPaths` / `-Dsonar.javascript.lcov.reportPaths` cuando `coverage-report-path` no esté vacío

## 5. Documentación y validación

- [ ] 5.1 Agregar comentarios inline en el YAML explicando cada sección de inputs y secrets
- [ ] 5.2 Agregar un archivo `workflow-templates/README-sonarcloud-scan.md` con ejemplos de uso para cada ecosistema (Python, JS, Terraform, generic)
- [ ] 5.3 Verificar que el workflow sea válido ejecutando `gh workflow list` o un linter de Actions en el repo de prueba
- [ ] 5.4 Probar la invocación desde un repositorio de prueba con `language: python` y `language: javascript`
