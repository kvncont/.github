## Context

Actualmente los repositorios de la organización no tienen una forma centralizada de ejecutar análisis de calidad de código con SonarCloud. Cada proyecto que quiera integrarlo debe configurar manualmente los pasos de escaneo. Un reusable workflow en el repositorio `.github` resuelve esto, actuando como plantilla centralizada reutilizable.

## Goals / Non-Goals

**Goals:**
- Proveer un reusable workflow de GitHub Actions que encapsule toda la lógica de escaneo de SonarCloud
- Soportar múltiples lenguajes: Python, JavaScript/TypeScript, Terraform y proyectos genéricos
- Parametrizar el workflow con inputs para lenguaje, organización y clave de proyecto SonarCloud
- Manejar la instalación de dependencias específicas por lenguaje (e.g., pip install para Python)
- Exponer un secret input requerido para `SONAR_TOKEN`

**Non-Goals:**
- No gestiona la creación de proyectos en SonarCloud (eso es responsabilidad del usuario)
- No instala herramientas de análisis estático adicionales más allá del scanner de SonarCloud
- No fuerza una configuración de calidad gate específica

## Decisions

### D1: Usar `workflow_call` como trigger
El workflow usa el evento `on: workflow_call` para ser invocable desde otros repositorios y workflows. Es el mecanismo nativo de GitHub Actions para reusable workflows, sin necesidad de herramientas externas.

Alternativa considerada: Crear una action compuesta (`action.yml`). Descartada porque el reusable workflow ofrece mayor flexibilidad (permite jobs, runners propios, secrets) y es el patrón recomendado para lógica de CI multi-step.

### D2: Usar `SonarSource/sonarcloud-github-action`
Se usa la action oficial de SonarCloud en lugar del scanner CLI manual. Simplifica la configuración, maneja automáticamente el reporte de PR decoration y está mantenida por SonarSource.

### D3: Parámetro `language` para bifurcar la instalación de dependencias
Un único input `language` (con valores: `python`, `javascript`, `terraform`, `generic`) controla qué pasos de preparación se ejecutan antes del análisis. Esto permite que el mismo workflow maneje múltiples ecosistemas sin complejidad de matrices.

Alternativa considerada: Un workflow por lenguaje. Descartada porque aumenta la superficie de mantenimiento.

### D4: Pinning de actions por SHA
Las actions externas (checkout, sonarcloud) se pinearán por SHA de commit para garantizar reproducibilidad y seguridad, siguiendo mejores prácticas de supply chain.

## Risks / Trade-offs

- **[Riesgo] El token SONAR_TOKEN debe configurarse en cada repositorio consumidor** → Mitigación: Documentar en README el setup necesario.
- **[Riesgo] Cambios en la action de SonarCloud pueden romper el workflow** → Mitigación: Pinear por SHA de versión estable.
- **[Trade-off] Un único workflow parametrizado vs. múltiples workflows especializados** → El enfoque unificado facilita el mantenimiento pero puede crecer en complejidad si los lenguajes requieren lógica muy diferente.
