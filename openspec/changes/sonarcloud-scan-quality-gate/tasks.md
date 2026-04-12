## 1. Crear el reusable workflow

- [ ] 1.1 Crear el archivo `.github/workflows/sonarcloud-scan-quality-gate.yml` en el repositorio `.github` de la organización
- [ ] 1.2 Definir el evento `workflow_call` con los inputs `sonar-organization` (required, string) y `sonar-project-key` (required, string) y `args` (optional, string, default: '')
- [ ] 1.3 Configurar `secrets: inherit` en el workflow para heredar `SONAR_TOKEN` del repositorio consumidor

## 2. Implementar el job de escaneo

- [ ] 2.1 Definir el job `sonarcloud-scan` con `runs-on: ubuntu-latest`
- [ ] 2.2 Agregar el paso `actions/checkout@v4` con `fetch-depth: 0`
- [ ] 2.3 Agregar el paso `SonarSource/sonarcloud-github-action@master` con los args `-Dsonar.organization`, `-Dsonar.projectKey`, `-Dsonar.qualitygate.wait=true` y el input `args` adicional
- [ ] 2.4 Configurar la variable de entorno `GITHUB_TOKEN` y `SONAR_TOKEN` en el paso de SonarCloud usando los secrets heredados

## 3. Validar el workflow

- [ ] 3.1 Verificar la sintaxis del archivo YAML con un linter o el validador de GitHub Actions
- [ ] 3.2 Confirmar que el workflow aparece correctamente en la pestaña Actions del repositorio `.github`
- [ ] 3.3 Invocar el workflow desde un repositorio de prueba para verificar que el escaneo se ejecuta y el Quality Gate bloquea correctamente en caso de fallo
