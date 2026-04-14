## ADDED Requirements

### Requirement: Workflow is reusable via workflow_call
The reusable workflow SHALL be triggerable via the `workflow_call` event, allowing any repository in the organization to invoke it as a job.

#### Scenario: Another workflow calls the reusable workflow
- **WHEN** a caller workflow references the reusable workflow using `uses: kvncont/.github/.github/workflows/sonarcloud-scan-quality-gate.yml@main`
- **THEN** the reusable workflow executes in the context of the caller repository

### Requirement: Required inputs for SonarCloud configuration
The workflow SHALL require two inputs: `sonar-organization` (the SonarCloud organization key) and `sonar-project-key` (the SonarCloud project key). Both MUST be provided by the caller.

#### Scenario: Caller provides both required inputs
- **WHEN** the caller passes `sonar-organization` and `sonar-project-key` as inputs
- **THEN** the workflow uses those values to configure the SonarCloud scanner

#### Scenario: Caller omits a required input
- **WHEN** the caller does not provide `sonar-organization` or `sonar-project-key`
- **THEN** GitHub Actions rejects the workflow invocation with a missing required input error

### Requirement: Optional args input for scanner customization
The workflow SHALL accept an optional `args` input (string) to pass additional `-D` flags to the SonarCloud scanner.

#### Scenario: Caller provides additional args
- **WHEN** the caller passes `args: "-Dsonar.sources=src"`
- **THEN** those args are appended to the scanner invocation

#### Scenario: Caller omits args
- **WHEN** the caller does not provide the `args` input
- **THEN** the scanner runs with only the required parameters (organization, project key, quality gate wait)

### Requirement: SONAR_TOKEN is provided via secret inheritance
The workflow SHALL use `secrets: inherit` so the caller repository's `SONAR_TOKEN` secret is available to the workflow without explicit mapping.

#### Scenario: Caller repository has SONAR_TOKEN configured
- **WHEN** the caller repository has a secret named `SONAR_TOKEN` at repository or organization level
- **THEN** the reusable workflow can authenticate against SonarCloud using that token

### Requirement: Full git history checkout
The workflow SHALL perform a full git history checkout (`fetch-depth: 0`) to enable SonarCloud blame analysis.

#### Scenario: Code checkout is performed
- **WHEN** the workflow starts its scan job
- **THEN** `actions/checkout` is called with `fetch-depth: 0`

### Requirement: Quality Gate validation blocks the workflow
The workflow SHALL configure the SonarCloud scanner with `sonar.qualitygate.wait=true` so the job fails if the Quality Gate is not passed.

#### Scenario: Quality Gate passes
- **WHEN** the SonarCloud analysis completes and the Quality Gate status is PASSED
- **THEN** the workflow job succeeds

#### Scenario: Quality Gate fails
- **WHEN** the SonarCloud analysis completes and the Quality Gate status is FAILED
- **THEN** the workflow job fails with a non-zero exit code, blocking any dependent jobs in the caller
