<!-- markdownlint-disable -->

# Hardening Report: devops-actions--load-used-actions/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-actions--load-used-actions/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference reusable workflows or actions using mutable branch (@main) or tag (@v4) refs instead of pinned 40-character SHA commits. This exposes the workflow to supply-chain attacks if the referenced ref is updated maliciously.

Failing references:
- actions-dependencies.yml: `devops-actions/.github/.github/workflows/actions-dependencies.yml@main`
- approve-dependabot-pr.yml: `devops-actions/.github/.github/workflows/approve-dependabot-pr.yml@main`
- dependency-review.yml: `devops-actions/.github/.github/workflows/dependency-review.yml@main`
- issue-pr-tag.yml: `devops-actions/.github/.github/workflows/issue-pr-tag.yml@main`
- ossf-analysis.yml: `devops-actions/.github/.github/workflows/rw-ossf-scorecard.yml@main`
- validate-pr.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/upload-artifact@v4`

Locations:

- `.github/workflows/actions-dependencies.yml:6`
- `.github/workflows/approve-dependabot-pr.yml:10`
- `.github/workflows/dependency-review.yml:17`
- `.github/workflows/issue-pr-tag.yml:14`
- `.github/workflows/ossf-analysis.yml:14`
- `.github/workflows/validate-pr.yml:18`
- `.github/workflows/validate-pr.yml:20`
- `.github/workflows/validate-pr.yml:63`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions inside shell commands (rule a). The values are substituted by the Actions template engine before the shell parses the command, allowing an attacker-controlled value to inject arbitrary shell commands.

Affected steps and offending lines:

.github/workflows/publishing.yml (test-local-action job):
  - `Write-Host "Found actions [${{ steps.load-actions.outputs.actions-file }}]"` (line 27)
  - `$content = Get-Content "${{ steps.load-actions.outputs.actions-file }}"` (line 29)

.github/workflows/publishing.yml (publish job):
  - `run: echo ${{ steps.tag.outputs.tag }}` (line 52) — unquoted and directly interpolated

.github/workflows/testing.yml (load-all-used-actions job):
  - `Write-Host "Got actions file location here [${{ steps.load-actions.outputs.actions-file }}]"` (line 48)
  - `$content = Get-Content -Path "${{ steps.load-actions.outputs.actions-file }}"` (line 49)

.github/workflows/testing.yml (load-all-used-actions-other-org job):
  - Same pattern repeated (lines 82–83)

.github/workflows/validate-pr.yml (run-local-action job):
  - `echo "Actions file: ${{ steps.load-actions.outputs.actions-file }}"` (line 44)
  - `if [ ! -f "${{ steps.load-actions.outputs.actions-file }}" ]` (line 45)
  - `count=$(jq '. | length' "${{ steps.load-actions.outputs.actions-file }}")` (line 48)

Fix: move the value into an env: variable and reference it as a quoted shell variable (e.g., `"$ACTIONS_FILE"`) instead of interpolating the expression directly.

Locations:

- `.github/workflows/publishing.yml:27`
- `.github/workflows/publishing.yml:29`
- `.github/workflows/publishing.yml:52`
- `.github/workflows/testing.yml:48`
- `.github/workflows/testing.yml:49`
- `.github/workflows/testing.yml:82`
- `.github/workflows/testing.yml:83`
- `.github/workflows/validate-pr.yml:44`
- `.github/workflows/validate-pr.yml:45`
- `.github/workflows/validate-pr.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned-uses by resolving full 40-char SHAs: devops-actions/.github@main→78ae1648b575b039d174b55ba1f5aaccab81b458, actions/checkout@v4→11d5960a326750d5838078e36cf38b85af677262, actions/setup-node@v4→49933ea5288caeca8642d1e84afbd3f7d6820020, actions/upload-artifact@v4→ea165f8d65b6e75b540449e92b4886f43607fa02. Fixed all script-injection issues in publishing.yml (lines 27,29,52), testing.yml (lines 48,49,82,83), and validate-pr.yml (lines 44,45,48) by moving ${{ steps.load-actions.outputs.actions-file }} and ${{ steps.tag.outputs.tag }} expressions into step env: blocks and referencing them as $env:ACTIONS_FILE (PowerShell) or $ACTIONS_FILE (bash) and $TAG respectively.

