## 1. Repository Setup

- [x] 1.1 Create the `.github/workflows/` directory in the repository root if it does not already exist
- [x] 1.2 Confirm the latest stable release tag for `SonarSource/sonarcloud-github-action` (check https://github.com/SonarSource/sonarcloud-github-action/releases)
- [x] 1.3 Confirm the latest stable release tag for `actions/checkout` (currently `v4`)

## 2. Reusable Workflow — Core File

- [x] 2.1 Create `.github/workflows/reusable-sonarcloud.yml` with `name: Reusable — SonarCloud Scan`
- [x] 2.2 Add `on.workflow_call` trigger block with `inputs` section containing `sonar-project-key` (string, required) and `sonar-organization` (string, required)
- [x] 2.3 Add optional inputs to the `workflow_call.inputs` block: `scanner-type` (string, default `auto`), `source-dir` (string, default `.`), `extra-args` (string, default `""`), `java-version` (string, default `'17'`)
- [x] 2.4 Add `secrets` block under `workflow_call` declaring `SONAR_TOKEN` as required with a clear description
- [x] 2.5 Define the `sonarcloud-scan` job with `runs-on: ubuntu-latest`
- [x] 2.6 Add `permissions: read-all` block at the job level
- [x] 2.7 Add step `actions/checkout@v4` with `fetch-depth: 0` as the first step
- [x] 2.8 Add step `actions/setup-java` (pinned version) to install JDK using `${{ inputs.java-version }}`
- [x] 2.9 Add step `SonarSource/sonarcloud-github-action@<pinned-tag>` with `env.SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}` and `with` block passing `projectBaseDir: ${{ inputs.source-dir }}` and `args` constructed from `sonar-project-key`, `sonar-organization`, and `extra-args`
- [x] 2.10 Add inline comments explaining how `scanner-type` affects the scan (note: for `auto` the scanner auto-detects; callers with Terraform can pass `-Dsonar.sources` via `extra-args`)
- [x] 2.11 Validate YAML syntax locally with `yamllint` or `actionlint` (or equivalent)

## 3. Workflow Template Stub

- [x] 3.1 Create `workflow-templates/sonarcloud.yml` with `name: SonarCloud Code Scanning`
- [x] 3.2 Set `on.push.branches: [ $default-branch ]` and `on.pull_request.branches: [ $default-branch ]`
- [x] 3.3 Add a single job `sonarcloud` that calls `uses: kvncont/.github/.github/workflows/reusable-sonarcloud.yml@main`
- [x] 3.4 Pass `with` block containing `sonar-project-key: YOUR_PROJECT_KEY` and `sonar-organization: kvncont` as placeholder values
- [x] 3.5 Pass `secrets` block with `SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}`
- [x] 3.6 Add instructional comments: where to find the project key, how to add `SONAR_TOKEN` as a repository secret, how to change `scanner-type` for Python/JS/Terraform

## 4. Workflow Template Metadata

- [x] 4.1 Create `workflow-templates/sonarcloud.json` with `name`, `description`, `iconName`, and `categories` fields
- [x] 4.2 Set `iconName` to `"shield-check"` (valid GitHub Octicon)
- [x] 4.3 Set `categories` to `["Python", "JavaScript", "Terraform", "Security"]`
- [x] 4.4 Validate the JSON file is well-formed (`python -m json.tool workflow-templates/sonarcloud.json`)

## 5. Verification

- [x] 5.1 Lint the reusable workflow YAML (`actionlint .github/workflows/reusable-sonarcloud.yml` or equivalent)
- [x] 5.2 Lint the template YAML (`yamllint workflow-templates/sonarcloud.yml`)
- [x] 5.3 Manually verify all required inputs/secrets referenced in the workflow match their declarations in `workflow_call`
- [ ] 5.4 Open a PR to `main` and confirm CI passes (no workflow syntax errors reported by GitHub)
- [ ] 5.5 Test-call the reusable workflow from a sandbox repository using a minimal caller config and confirm SonarCloud analysis completes successfully
