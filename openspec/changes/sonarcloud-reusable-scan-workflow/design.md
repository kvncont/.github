## Context

La organización gestiona múltiples repositorios con stacks heterogéneos (Python, JavaScript/TypeScript, Terraform, y proyectos genéricos). Actualmente no existe una forma centralizada de integrar SonarCloud, por lo que cada equipo copia manualmente la configuración o no la adopta. GitHub Actions soporta **reusable workflows** (`workflow_call`) que permiten centralizar la lógica en un único archivo YAML dentro del repositorio `.github`.

## Goals / Non-Goals

**Goals:**
- Diseñar un único archivo `workflow-templates/sonarcloud-scan.yml` invocable desde cualquier repositorio de la organización.
- Soportar los ecosistemas: Python, JavaScript/TypeScript, Terraform y un modo `generic` (para cualquier otro lenguaje soportado por SonarCloud sin configuración especial).
- Parametrizar completamente: organización Sonar, project key, directorio de fuentes, exclusiones, cobertura.
- Publicar el workflow como template de la organización con su `properties.json`.

**Non-Goals:**
- No se implementará un servidor SonarQube self-hosted (solo SonarCloud).
- No se gestionará la creación automática de proyectos en SonarCloud.
- No se cubrirá la generación de reportes de cobertura (solo su consumo si ya existe el archivo).

## Decisions

### D1: Mecanismo de detección del ecosistema → Input explícito

**Decisión**: El ecosistema se declara mediante un input `language` (`python | javascript | terraform | generic`), en lugar de detección automática por archivos presentes en el repo.

**Alternativas consideradas**:
- Detección automática (buscar `requirements.txt`, `package.json`, `*.tf`): más conveniente, pero frágil en repos mixtos y añade complejidad de scripting.
- Un workflow por lenguaje: duplicación excesiva de YAML.

**Rationale**: La declaración explícita es predecible, auditable y no añade pasos de shell complejos.

---

### D2: Setup de runtime → Condicional por input

**Decisión**: Usar `actions/setup-python` o `actions/setup-node` de forma condicional con `if: inputs.language == 'python'` / `if: inputs.language == 'javascript'`. Terraform no requiere setup de runtime para el escáner. El modo `generic` omite el setup.

**Rationale**: Mantiene el workflow en un solo archivo sin matrices (matrix añadiría jobs paralelos innecesarios para un escaneo único).

---

### D3: Acción de escaneo → `SonarSource/sonarcloud-github-action`

**Decisión**: Usar la acción oficial `SonarSource/sonarcloud-github-action@master` que internamente invoca el Sonar Scanner CLI.

**Alternativas consideradas**:
- Instalar `sonar-scanner` CLI manualmente: más control de versión, pero requiere mantenimiento del script de descarga.

**Rationale**: La acción oficial gestiona automáticamente PR decoration y análisis de ramas; es el camino recomendado por SonarCloud.

---

### D4: Secrets y configuración → Secrets heredados + inputs opcionales

**Decisión**: `SONAR_TOKEN` se pasa como `secrets: inherit` para simplificar la adopción. El `projectKey` y `organization` se declaran como inputs requeridos. Las demás opciones (exclusiones, directorio fuentes, ruta cobertura) son inputs opcionales con defaults razonables.

**Rationale**: `secrets: inherit` elimina la necesidad de declarar y mapear manualmente `SONAR_TOKEN` en cada workflow llamador.

---

### D5: Ubicación del archivo → `workflow-templates/`

**Decisión**: El archivo vive en `workflow-templates/sonarcloud-scan.yml` junto con `sonarcloud-scan.properties.json` para integrarse con el catálogo de templates de la organización.

**Rationale**: Sigue la convención de la organización (existe `python-ci.yml` en esa carpeta) y permite descubrir el template desde la UI de GitHub.

## Risks / Trade-offs

- **[Riesgo] `secrets: inherit` expone todos los secrets del repo llamador al workflow reutilizable** → Mitigación: documentar claramente que los repositorios solo deben definir `SONAR_TOKEN` en sus secrets; el workflow no usa ningún otro secret.
- **[Riesgo] Pin a `@master` de la acción de SonarSource puede introducir cambios no controlados** → Mitigación: versionar a un SHA específico en el primer release; actualizar periódicamente.
- **[Trade-off] Input explícito de `language` puede olvidarse** → Los llamadores recibirán un error claro de validación de inputs si lo omiten (GitHub lo reporta automáticamente cuando el input es `required: true`).
- **[Riesgo] Repositorios sin `sonar-project.properties` deben pasar project key manualmente** → Mitigación: documentar ambas modalidades (archivo properties vs. inputs).
