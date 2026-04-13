## Context

The kvncont organization hosts projects in multiple languages (Python, JavaScript/TypeScript, Terraform). All of them are expected to run SonarCloud analysis on every push and pull request. Currently there is no shared workflow; each project would need to copy-paste configuration. This design introduces a single reusable GitHub Actions workflow in the `.github` organization repository that any caller can reference with two lines of YAML.

The reusable workflow model (`workflow_call`) is the idiomatic GitHub Actions solution: it lives in this repository and is versioned with it, so the organization can patch the scanner version or add steps in one place and every caller benefits on its next run.

## Goals / Non-Goals

**Goals:**
- Provide a `workflow_call`-based reusable workflow at `.github/workflows/reusable-sonarcloud.yml` that runs SonarCloud analysis.
- Support Python, JavaScript/TypeScript, Terraform, and a generic "auto" mode via a single `scanner-type` input.
- Expose the minimum required inputs (`sonar-project-key`, `sonar-organization`) and optional inputs (`source-dir`, `extra-args`, `java-version`) with sane defaults.
- Accept `SONAR_TOKEN` as the only mandatory secret, forwarded from the caller.
- Provide a workflow template stub (`workflow-templates/sonarcloud.yml` + `sonarcloud.json`) so org members can bootstrap from the GitHub UI.

**Non-Goals:**
- Building/compiling the project under test (callers are responsible for their own build steps when needed for Java/Kotlin bytecode analysis).
- Running unit tests or coverage within this workflow.
- Supporting self-hosted SonarQube (on-premises); this workflow targets SonarCloud only.
- Multi-branch project configuration beyond what SonarCloud auto-detects.

## Decisions

### Decision 1 — Single workflow file, scanner-type input (not separate workflows per language)

**Chosen**: One reusable workflow with a `scanner-type` string input (`auto | python | javascript | terraform`).

**Rationale**: Languages share 90% of the workflow steps (checkout, run sonarcloud-github-action). A branching step inside one file is simpler to maintain than N separate reusable workflows. Callers set one extra input and get the right behaviour.

**Alternative considered**: Separate `reusable-sonarcloud-python.yml`, `reusable-sonarcloud-js.yml`, etc. Rejected because it multiplies maintenance surface and forces callers to know which file to call.

### Decision 2 — Use `SonarSource/sonarcloud-github-action` action (not sonar-scanner CLI directly)

**Chosen**: `SonarSource/sonarcloud-github-action@v3` (pinned to a version tag).

**Rationale**: The official action handles authentication, PR decoration, and quality gate status automatically. It wraps the sonar-scanner CLI and sets `sonar.pullrequest.*` properties from GitHub context without manual wiring.

**Alternative considered**: Installing the sonar-scanner CLI manually. Rejected because it requires maintaining the download URL, Java setup, and PR properties manually.

### Decision 3 — Pin to version tag, not SHA

**Chosen**: Pin to a version tag such as `SonarSource/sonarcloud-github-action@v3`.

**Rationale**: SonarSource controls and signs their release tags; version tags are human-readable and semver-communicative. The organisation can do a global find-replace when upgrading. SHA pinning is preferred for third-party community actions but adds noise for first-party vendor actions with a reliable release process.

**Alternative considered**: Floating `@master`. Rejected — can break silently.

### Decision 4 — `SONAR_TOKEN` as inherited secret (not `secrets: inherit`)

**Chosen**: Explicitly declare `SONAR_TOKEN` as a required secret in the `workflow_call` trigger.

**Rationale**: `secrets: inherit` is convenient but passes every caller secret to the reusable workflow, violating least-privilege. Explicit declaration documents exactly what is needed.

### Decision 5 — `java-version` optional input with default `17`

**Chosen**: Expose `java-version` as an optional string input defaulting to `'17'`.

**Rationale**: The SonarCloud scanner requires Java. Some repos may use a different JDK. Making it configurable avoids hard-coding while providing a working default.

## Risks / Trade-offs

- **SonarCloud project must be pre-created**: The reusable workflow cannot create the SonarCloud project; this must be done once by the org admin. → Mitigation: document this prerequisite in the workflow template comments.
- **Quality gate failures do not block merges by default**: The action reports gate status but branch protection must be separately configured to enforce it. → Mitigation: note this in the template comments.
- **`auto` scanner type works only for supported languages**: SonarCloud auto-detect covers most languages but may miss some edge cases. → Mitigation: callers can switch to an explicit `scanner-type` and pass `extra-args`.
- **Version tag pinning**: If SonarSource breaks a release tag (rare), all callers break simultaneously. → Mitigation: monitor SonarSource release notes; use Dependabot on this repo to receive update PRs.

## Migration Plan

1. Add `.github/workflows/reusable-sonarcloud.yml` and `workflow-templates/sonarcloud.yml` + `sonarcloud.json` to this repo via a PR.
2. Verify the workflow is callable from a test repository using a `workflow_call` reference.
3. Announce availability to org members via the standard communication channel.
4. Existing repos that have ad-hoc SonarCloud steps can optionally migrate to the reusable workflow; there is no forced migration.

**Rollback**: Delete or rename the workflow file. Callers will receive a `workflow not found` error on their next run and can revert to their inline configuration.

## Open Questions

- Should the workflow enforce quality gate pass/fail at the workflow level (i.e., fail the job if the quality gate fails)? Currently the action exits 0 regardless. This can be added later as an optional boolean input `fail-on-quality-gate`.
- Is there a need for a `sonar-scanner-version` input to allow callers to pin a specific scanner CLI version? Deferred until a concrete need arises.
