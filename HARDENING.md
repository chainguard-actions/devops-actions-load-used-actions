<!-- markdownlint-disable -->

# Hardening Report: devops-actions--load-used-actions/v1.3.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-actions--load-used-actions/v1.3.8** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In publishing.yml, the step 'Use tag' runs `echo ${{ steps.tag.outputs.tag }}` — the steps context value is interpolated directly into the shell command before the shell sees it, enabling command injection. Additionally, the test-local-action job's run block interpolates `${{ steps.load-actions.outputs.actions-file }}` directly inside Write-Host and Get-Content shell commands.

Locations:

- `.github/workflows/publishing.yml:27`
- `.github/workflows/publishing.yml:28`
- `.github/workflows/publishing.yml:51`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In testing.yml, two separate job steps (load-all-used-actions and load-all-used-actions-other-org) each contain run: blocks that interpolate `${{ steps.load-actions.outputs.actions-file }}` directly inside Write-Host and Get-Content PowerShell commands, enabling command injection via a maliciously crafted output value.

Locations:

- `.github/workflows/testing.yml:47`
- `.github/workflows/testing.yml:48`
- `.github/workflows/testing.yml:80`
- `.github/workflows/testing.yml:81`

### unpinned-uses (severity: high)

Workflow files reference reusable workflows by mutable branch name (@main) instead of a pinned 40-character commit SHA. This means the referenced workflow can be silently changed by the owner of the referenced repository at any time, creating a supply-chain risk. Failing references: `devops-actions/.github/.github/workflows/approve-dependabot-pr.yml@main` in approve-dependabot-pr.yml and `devops-actions/.github/.github/workflows/rw-ossf-scorecard.yml@main` in ossf-analysis.yml.

Locations:

- `.github/workflows/approve-dependabot-pr.yml:12`
- `.github/workflows/ossf-analysis.yml:16`

### broad-permissions (severity: medium)

Three workflow files declare `permissions: read-all` at the top level. This grants read access to all available GitHub token scopes rather than the minimal set required, violating the principle of least privilege. Affected files: codeql.yml (line 14), ossf-analysis.yml (line 13), testing.yml (line 9).

Locations:

- `.github/workflows/codeql.yml:14`
- `.github/workflows/ossf-analysis.yml:13`
- `.github/workflows/testing.yml:9`

### missing-permissions (severity: medium)

semver-check.yml has no top-level `permissions:` key and its only job (`semver`) also has no job-level `permissions:` key. Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be overly broad (write-all in many configurations).

Locations:

- `.github/workflows/semver-check.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions

**Notes:**

Fixed all 5 findings: (1) Script injection in publishing.yml - moved steps.load-actions.outputs.actions-file and steps.tag.outputs.tag into env blocks; (2) Script injection in testing.yml - moved steps.load-actions.outputs.actions-file into env blocks in both affected jobs; (3) Unpinned reusable workflow references in approve-dependabot-pr.yml and ossf-analysis.yml pinned to SHA a634e5b033926a6683519b3ffc7b556312ca7e00 (main); (4) Broad permissions (read-all) replaced with specific minimal permissions in codeql.yml, ossf-analysis.yml, and testing.yml; (5) Added missing permissions block (contents: read) to semver-check.yml.

