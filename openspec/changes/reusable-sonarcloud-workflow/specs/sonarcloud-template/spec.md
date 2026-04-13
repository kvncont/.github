## ADDED Requirements

### Requirement: Workflow template YAML stub exists in workflow-templates/
A caller-ready YAML stub SHALL be placed at `workflow-templates/sonarcloud.yml`. It SHALL demonstrate how to call the reusable workflow with all required inputs and the SONAR_TOKEN secret, and SHALL include comments guiding the user on what to replace.

#### Scenario: User opens the template from the GitHub repository template picker
- **WHEN** a user navigates to the Actions tab of a repository in the kvncont org and selects "New workflow"
- **THEN** the SonarCloud workflow template SHALL appear as an option and, when selected, pre-fill the editor with the contents of `workflow-templates/sonarcloud.yml`

#### Scenario: Template contains all required inputs
- **WHEN** a developer reads `workflow-templates/sonarcloud.yml`
- **THEN** they SHALL see placeholder values for `sonar-project-key` and `sonar-organization`, and a reference to the SONAR_TOKEN secret

### Requirement: Workflow template metadata JSON file exists
A metadata file SHALL be placed at `workflow-templates/sonarcloud.json` following the GitHub workflow template schema. It SHALL include:
- `name`: Human-readable template name (e.g., "SonarCloud Code Scanning")
- `description`: One-sentence description of what the template does
- `iconName`: A valid GitHub Octicon name (e.g., `"shield-check"`)
- `categories`: An array listing relevant language/topic categories (e.g., `["Python", "JavaScript", "Terraform", "Security"]`)

#### Scenario: Metadata file is valid JSON
- **WHEN** the file `workflow-templates/sonarcloud.json` is parsed
- **THEN** it SHALL be valid JSON with non-empty `name`, `description`, `iconName`, and `categories` fields

#### Scenario: Categories reflect supported languages
- **WHEN** the categories array is inspected
- **THEN** it SHALL include at least `"Python"`, `"JavaScript"`, and `"Security"`

### Requirement: Template triggers on push and pull_request to default branch
The template YAML stub SHALL configure `on.push.branches` and `on.pull_request.branches` using the `$default-branch` template variable so GitHub substitutes the actual default branch name when a user adopts the template.

#### Scenario: Template uses $default-branch placeholder
- **WHEN** a developer reads the template YAML
- **THEN** the branch filters SHALL contain `$default-branch` rather than a hard-coded branch name like `main`

### Requirement: Template includes instructional comments
The template YAML SHALL include inline comments that explain:
- How to set the `SONAR_TOKEN` secret in the repository settings
- Where to find the SonarCloud project key and organization slug
- How to change the `scanner-type` for different languages

#### Scenario: Developer can adopt the template without external documentation
- **WHEN** a developer reads `workflow-templates/sonarcloud.yml` with no prior SonarCloud knowledge
- **THEN** the inline comments SHALL provide enough guidance to correctly fill in the required values
