# .github

Organization-wide GitHub configurations for `kvncont`.

## Workflow Templates

### scan-sonar — SonarCloud Scan & Quality Gate

Reusable workflow to scan repositories with SonarCloud and validate the Quality Gate.

**File:** [`workflow-templates/scan-sonar.yml`](workflow-templates/scan-sonar.yml)

#### Usage

To use this as a reusable workflow, copy `workflow-templates/scan-sonar.yml` to `.github/workflows/scan-sonar.yml` in a repository (or in this `.github` repo), then call it from any other workflow:

```yaml
jobs:
  scan:
    uses: kvncont/.github/.github/workflows/scan-sonar.yml@main
    with:
      project-key: 'my-project-key'
      organization: 'kvncont'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

#### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `project-key` | ✅ | — | SonarCloud project key |
| `organization` | ✅ | — | SonarCloud organization |
| `java-version` | ❌ | `''` | Java version (empty = skip JDK setup) |
| `project-base-dir` | ❌ | `.` | Project base directory |
| `extra-args` | ❌ | `''` | Additional SonarScanner arguments |

#### Secrets

| Secret | Required | Description |
|---|---|---|
| `SONAR_TOKEN` | ✅ | SonarCloud authentication token |