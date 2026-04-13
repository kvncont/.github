## Why

All repositories in the kvncont organization need consistent, maintainable code quality and security scanning via SonarCloud, but today each project must wire up its own SonarCloud workflow from scratch — leading to configuration drift, missed updates, and duplicated effort. A single reusable workflow centralizes this logic so every project gets scanning for free with minimal caller configuration.

## What Changes

- **New reusable workflow** `.github/workflows/reusable-sonarcloud.yml` that accepts `workflow_call` and orchestrates SonarCloud scanning via `SonarSource/sonarcloud-github-action`.
- **Configurable language/scanner support** via an input that selects the appropriate scanner mode (automatic detection, Python, JavaScript/TypeScript, Terraform, or other).
- **Parameterised inputs** for `sonar-project-key`, `sonar-organization`, source directory, and optional extra scanner arguments.
- **SONAR_TOKEN secret** forwarded to the action; no other secrets required.
- **New workflow template stub** `workflow-templates/sonarcloud.yml` + `workflow-templates/sonarcloud.json` so any repo in the org can adopt this workflow from the repository template picker.

## Capabilities

### New Capabilities

- `sonarcloud-scan`: Reusable GitHub Actions workflow that runs SonarCloud analysis for any language. Accepts project key, organization, optional source paths, and scanner configuration; produces a quality gate result posted back to the pull request.
- `sonarcloud-template`: Caller-side workflow template (stub + metadata JSON) that repositories use as the starting point when enabling SonarCloud scanning.

### Modified Capabilities

## Impact

- **New files**: `.github/workflows/reusable-sonarcloud.yml`, `workflow-templates/sonarcloud.yml`, `workflow-templates/sonarcloud.json`
- **No existing files modified**
- **Downstream repos**: Any repo in `kvncont` org can call the reusable workflow; they must supply `SONAR_TOKEN` as a repository secret and provide their SonarCloud project key and organization slug.
- **Dependencies**: `SonarSource/sonarcloud-github-action` (pinned to a specific version tag), `actions/checkout`.
