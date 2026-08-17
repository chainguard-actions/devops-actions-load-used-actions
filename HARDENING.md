<!-- markdownlint-disable -->

# Hardening Report: devops-actions--load-used-actions/v1.3.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-actions--load-used-actions/v1.3.7** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In publishing.yml, the step 'run: |' (pwsh) embeds '${{ steps.load-actions.outputs.actions-file }}' directly in Write-Host and variable assignment lines, and a separate step uses 'run: echo ${{ steps.tag.outputs.tag }}'. These step output values flow through YAML template substitution before the shell sees them, enabling script injection.

Locations:

- `.github/workflows/publishing.yml:27`
- `.github/workflows/publishing.yml:28`
- `.github/workflows/publishing.yml:50`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In testing.yml, two separate pwsh run: blocks embed '${{ steps.load-actions.outputs.actions-file }}' directly in Write-Host and Get-Content commands. These step output values flow through YAML template substitution before the shell sees them, enabling script injection.

Locations:

- `.github/workflows/testing.yml:46`
- `.github/workflows/testing.yml:47`
- `.github/workflows/testing.yml:80`
- `.github/workflows/testing.yml:81`

### unpinned-uses (severity: high)

Reusable workflow references use mutable branch refs instead of full 40-character commit SHAs. 'devops-actions/.github/.github/workflows/approve-dependabot-pr.yml@main' and 'devops-actions/.github/.github/workflows/rw-ossf-scorecard.yml@main' are both pinned to the @main branch, which can change at any time and is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/approve-dependabot-pr.yml:12`
- `.github/workflows/ossf-analysis.yml:13`

### missing-permissions (severity: medium)

semver-check.yml has no top-level 'permissions:' key and its only job ('semver') also has no job-level 'permissions:' key. Without explicit permissions, the workflow inherits the default GITHUB_TOKEN permissions, which may be overly broad depending on repository settings.

Locations:

- `.github/workflows/semver-check.yml:1`

### broad-permissions (severity: medium)

Three workflow files set top-level 'permissions: read-all', which grants read access to all available scopes rather than the minimal specific permissions required. This should be replaced with explicit, minimal permission scopes.

Locations:

- `.github/workflows/codeql.yml:9`
- `.github/workflows/ossf-analysis.yml:12`
- `.github/workflows/testing.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, broad-permissions

**Notes:**

Fixed all 5 findings across 6 workflow files:
1. script-injection (publishing.yml): Moved steps.load-actions.outputs.actions-file and steps.tag.outputs.tag into env blocks; referenced via $env:ACTIONS_FILE and $TAG in pwsh/bash scripts.
2. script-injection (testing.yml): Moved steps.load-actions.outputs.actions-file into env blocks in both pwsh run blocks (load-all-used-actions and load-all-used-actions-other-org jobs); referenced via $env:ACTIONS_FILE.
3. unpinned-uses (approve-dependabot-pr.yml, ossf-analysis.yml): Replaced @main branch refs with pinned SHA 78ae1648b575b039d174b55ba1f5aaccab81b458 (with # main comment) for both devops-actions/.github reusable workflows.
4. missing-permissions (semver-check.yml): Added top-level 'permissions: contents: read' block.
5. broad-permissions (codeql.yml, ossf-analysis.yml, testing.yml): Replaced 'permissions: read-all' with specific minimal scopes: codeql.yml gets contents/actions/security-events; ossf-analysis.yml gets contents/actions/security-events/id-token; testing.yml gets contents only.

