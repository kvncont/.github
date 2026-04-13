# sonarcloud-scan Specification

## Purpose
TBD - created by archiving change reusable-sonarcloud-workflow. Update Purpose after archive.
## Requirements
### Requirement: Reusable workflow is callable via workflow_call
The workflow SHALL expose a `workflow_call` trigger so any repository in the kvncont organization (or a permitted external caller) can reference it with `uses: kvncont/.github/.github/workflows/reusable-sonarcloud.yml@main`.

#### Scenario: Caller references the reusable workflow
- **WHEN** a caller workflow defines `uses: kvncont/.github/.github/workflows/reusable-sonarcloud.yml@main` with valid inputs and secrets
- **THEN** GitHub Actions SHALL execute the reusable workflow without errors

### Requirement: Mandatory inputs are declared with descriptions
The workflow SHALL declare the following required inputs under `on.workflow_call.inputs`:
- `sonar-project-key` (string, required): The unique SonarCloud project key.
- `sonar-organization` (string, required): The SonarCloud organization slug.

#### Scenario: Caller omits a required input
- **WHEN** a caller workflow omits `sonar-project-key` or `sonar-organization`
- **THEN** GitHub Actions SHALL reject the workflow run with a validation error before any job starts

### Requirement: Optional inputs with documented defaults
The workflow SHALL declare the following optional inputs with defaults:
- `scanner-type` (string, default `auto`): Controls scanner mode. Accepted values: `auto`, `python`, `javascript`, `terraform`.
- `source-dir` (string, default `.`): Directory to scan, relative to the repository root.
- `extra-args` (string, default `""`): Additional sonar-scanner properties passed verbatim (e.g., `-Dsonar.exclusions=tests/**`).
- `java-version` (string, default `'17'`): JDK version used to run the scanner.

#### Scenario: Caller provides no optional inputs
- **WHEN** a caller workflow supplies only the required inputs
- **THEN** the workflow SHALL run with default values (`auto` scanner, `.` source dir, JDK 17, no extra args)

#### Scenario: Caller overrides scanner type
- **WHEN** a caller sets `scanner-type: python`
- **THEN** the scan job SHALL configure the SonarCloud action for Python analysis

### Requirement: SONAR_TOKEN secret is required and forwarded
The workflow SHALL declare `SONAR_TOKEN` as a required secret under `on.workflow_call.secrets` and pass it to the SonarCloud action step.

#### Scenario: Caller forwards SONAR_TOKEN
- **WHEN** a caller passes `secrets: { SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }} }`
- **THEN** the scan step SHALL authenticate to SonarCloud successfully

#### Scenario: SONAR_TOKEN is not provided
- **WHEN** a caller omits the SONAR_TOKEN secret
- **THEN** GitHub Actions SHALL fail the workflow at validation or the SonarCloud action SHALL exit with an authentication error

### Requirement: Checkout step precedes scan
The workflow's scan job SHALL include an `actions/checkout` step with `fetch-depth: 0` before running the SonarCloud action, to ensure full git history is available for blame information.

#### Scenario: Scan runs after checkout
- **WHEN** the scan job executes
- **THEN** `actions/checkout@v4` with `fetch-depth: 0` SHALL be the first step, and the SonarCloud action SHALL be the subsequent step

### Requirement: SonarCloud action is pinned to a version tag
The workflow SHALL reference `SonarSource/sonarcloud-github-action` pinned to a specific release tag (e.g., `v3`). No floating `@master` or untagged references are permitted.

#### Scenario: Action version is inspectable in workflow YAML
- **WHEN** a maintainer reads `.github/workflows/reusable-sonarcloud.yml`
- **THEN** the SonarCloud action reference SHALL include an explicit version tag (e.g., `SonarSource/sonarcloud-github-action@v3`)

### Requirement: Workflow sets minimum required permissions
The scan job SHALL declare `permissions: read-all` at the job level (or the minimal subset: `contents: read`, `pull-requests: read`) to follow least-privilege principles.

#### Scenario: Permissions block is present
- **WHEN** the workflow YAML is validated
- **THEN** the scan job SHALL contain a `permissions` key that does not grant write access beyond what is strictly needed

### Requirement: Workflow runs on ubuntu-latest
The scan job SHALL use `runs-on: ubuntu-latest`.

#### Scenario: Job runner is ubuntu-latest
- **WHEN** the reusable workflow is triggered
- **THEN** the scan job SHALL be assigned to an ubuntu-latest runner

