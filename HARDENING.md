<!-- markdownlint-disable -->

# Hardening Report: devops-actions--load-used-actions/v1.3.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-actions--load-used-actions/v1.3.6** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### broad-permissions (severity: medium)

The workflow uses top-level 'permissions: read-all', which grants overly broad read access to all scopes. Replace with specific minimal permissions.

Locations:

- `.github/workflows/codeql.yml:14`
- `.github/workflows/ossf-analysis.yml:12`
- `.github/workflows/publishing.yml:11`
- `.github/workflows/testing.yml:9`

### missing-permissions (severity: medium)

The workflow file 'semver-check.yml' has no top-level 'permissions:' key and the single job 'semver' also has no job-level 'permissions:' key. This defaults to the repository's maximum token permissions.

Locations:

- `.github/workflows/semver-check.yml:1`

### unpinned-uses (severity: high)

The reusable workflow reference 'devops-actions/.github/.github/workflows/rw-ossf-scorecard.yml@main' uses a mutable branch ref (@main) instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the referenced branch is compromised.

Locations:

- `.github/workflows/ossf-analysis.yml:16`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In publishing.yml, the step 'Use tag' runs: 'echo ${{ steps.tag.outputs.tag }}' — a step output is injected directly into the shell. In the same file, a pwsh run: block interpolates '${{ steps.load-actions.outputs.actions-file }}' directly into Write-Host and a variable assignment. These allow an attacker who controls the output value to inject arbitrary shell commands.

Locations:

- `.github/workflows/publishing.yml:25`
- `.github/workflows/publishing.yml:26`
- `.github/workflows/publishing.yml:27`
- `.github/workflows/publishing.yml:52`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In testing.yml, two pwsh run: blocks (in jobs 'load-all-used-actions' and 'load-all-used-actions-other-org') interpolate '${{ steps.load-actions.outputs.actions-file }}' directly into Write-Host calls and Get-Content -Path arguments. An attacker who controls the action output value could inject arbitrary PowerShell commands.

Locations:

- `.github/workflows/testing.yml:47`
- `.github/workflows/testing.yml:48`
- `.github/workflows/testing.yml:79`
- `.github/workflows/testing.yml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** broad-permissions, missing-permissions, unpinned-uses, script-injection

**Notes:**

Fixed all 5 findings across 5 workflow files:
1. broad-permissions: Replaced 'permissions: read-all' with minimal specific permissions in codeql.yml ({}), ossf-analysis.yml ({}), publishing.yml ({}), and testing.yml (contents: read).
2. missing-permissions: Added 'permissions: contents: read' to semver-check.yml.
3. unpinned-uses: Pinned devops-actions/.github reusable workflow from '@main' to '@a634e5b033926a6683519b3ffc7b556312ca7e00 # main' in ossf-analysis.yml.
4. script-injection in publishing.yml: Moved steps.load-actions.outputs.actions-file to env var ACTIONS_FILE in the pwsh block; moved steps.tag.outputs.tag to env var TAG in the 'Use tag' step.
5. script-injection in testing.yml: Moved steps.load-actions.outputs.actions-file to env var ACTIONS_FILE in both pwsh blocks (load-all-used-actions and load-all-used-actions-other-org jobs).

