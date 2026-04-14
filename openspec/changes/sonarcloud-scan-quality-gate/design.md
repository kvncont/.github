## Context

La organización `kvncont` gestiona múltiples repositorios que requieren análisis de calidad de código con SonarCloud. Actualmente no existe un mecanismo centralizado: cada repositorio debería configurar sus propios pasos, lo que genera inconsistencias y duplicación de lógica. GitHub Actions permite definir **reusable workflows** que otros repositorios pueden llamar como un job, centralizando la lógica de escaneo.

SonarCloud requiere:
- `fetch-depth: 0` en el checkout para análisis de blame/historia de commits.
- Un token (`SONAR_TOKEN`) para autenticarse contra la API.
- El `sonar-project.properties` en el repositorio consumidor, o bien parámetros pasados como argumentos al scanner.
- La acción oficial `SonarSource/sonarcloud-github-action` para ejecutar el análisis y validar el Quality Gate.

## Goals / Non-Goals

**Goals:**
- Centralizar en un único workflow reutilizable la lógica de escaneo SonarCloud y validación de Quality Gate.
- Permitir que cualquier repositorio de la organización invoque el workflow con mínima configuración.
- Fallar el job si el Quality Gate no es aprobado, bloqueando merges no conformes.
- Soportar configuración de `organization` y `projectKey` como inputs obligatorios.
- Exponer `args` adicionales como input opcional para personalización avanzada del scanner.

**Non-Goals:**
- Configuración del proyecto en SonarCloud (se asume que ya existe).
- Instalación de lenguajes o build tools específicos (el consumidor es responsable).
- Soporte multi-branch avanzado o decoración de PRs más allá de lo que la acción oficial ofrece.

## Decisions

### Usar `SonarSource/sonarcloud-github-action`

**Decisión**: Utilizar la acción oficial `SonarSource/sonarcloud-github-action` en lugar de invocar el scanner CLI directamente.

**Alternativas consideradas**:
- **Scanner CLI directo**: Mayor control pero requiere instalación manual de Java/scanner y configuración compleja.
- **Acción oficial**: Encapsula instalación, configuración y validación del Quality Gate en un solo paso, menos código a mantener.

**Rationale**: La acción oficial maneja automáticamente la validación del Quality Gate mediante `sonar.qualitygate.wait=true`, simplificando el workflow.

### Inputs vs sonar-project.properties

**Decisión**: Requerir `sonar-organization` y `sonar-project-key` como inputs del workflow, pasándolos como flags `-D` al scanner.

**Alternativas consideradas**:
- **Solo `sonar-project.properties`**: El consumidor controla todo, pero el workflow no puede validar la configuración mínima.
- **Inputs obligatorios**: Garantiza que el workflow siempre tenga los parámetros mínimos necesarios, con validación implícita.

**Rationale**: Los inputs obligatorios hacen el contrato explícito y evitan fallos silenciosos por archivos de propiedades mal configurados.

### Secret `SONAR_TOKEN` como secret declarado

**Decisión**: Declarar `SONAR_TOKEN` como secret requerido en `on.workflow_call.secrets`, permitiendo al repositorio consumidor pasarlo explícitamente o usar `secrets: inherit` desde su side.

**Alternativas consideradas**:
- **Solo `secrets: inherit` en el job del caller**: El workflow no documenta qué secrets necesita, lo cual dificulta el descubrimiento y la validación.
- **Secret declarado en `workflow_call.secrets`**: El contrato queda explícito; el caller puede usar `secrets: inherit` o mapearlo directamente, con plena flexibilidad.

**Rationale**: Declarar el secret explícitamente documenta el contrato del workflow y permite la validación de presencia, manteniendo la simplicidad para el consumidor.

## Risks / Trade-offs

- **[Riesgo] Cambios en la acción oficial de SonarCloud** → Usar una versión fija (e.g., `@master` o tag específico) y actualizar periódicamente con Dependabot.
- **[Trade-off] `secrets: inherit` expone todos los secrets del consumidor al workflow** → Aceptable dado que el workflow es interno a la organización y está bajo control del equipo.
- **[Riesgo] El Quality Gate no está configurado en SonarCloud** → El scanner fallará con un error explícito, lo cual es el comportamiento deseable.
