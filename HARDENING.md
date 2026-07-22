<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates GitHub Actions expressions inside shell commands (sub-rule a). Three expressions are embedded: (1) `case ${{ inputs.version }} in` — attacker-controlled input interpolated directly into a shell case statement, allowing command injection; (2) `curl -s -u "username:${{ github.token }}"` — github.token interpolated directly into a shell command; (3) `os=${{ runner.os }}` — runner.os interpolated directly into a shell assignment. All ${{ ... }} expressions inside run: blocks are script-injection risks because YAML template substitution happens before the shell ever sees the string, bypassing shell quoting. These should be moved to env: variables and referenced as quoted shell variables (e.g., "$INPUTS_VERSION", "$GITHUB_TOKEN", "$RUNNER_OS").

Locations:

- `action.yml:22`
- `action.yml:26`
- `action.yml:32`
- `action.yml:36`
- `action.yml:46`

### unpinned-uses (severity: high)

The workflow file .github/workflows/use-action.yaml references multiple actions using mutable tags/branches instead of full 40-character commit SHA digests, making the workflow vulnerable to supply-chain attacks if those refs are moved: (1) `uses: actions/setup-go@v2` — mutable version tag; (2) `uses: actions/checkout@v2` — mutable version tag; (3) `uses: imjasonh/setup-crane@main` — mutable branch ref (appears 3 times). All should be pinned to their full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/use-action.yaml:14`
- `.github/workflows/use-action.yaml:16`
- `.github/workflows/use-action.yaml:18`
- `.github/workflows/use-action.yaml:22`
- `.github/workflows/use-action.yaml:27`

### missing-permissions (severity: medium)

The workflow file .github/workflows/use-action.yaml has no top-level `permissions:` key and the single job `use-action` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be write-all), violating the principle of least privilege. A minimal permissions block (e.g., `permissions: {}` or `permissions: contents: read`) should be added at the top level or on each job.

Locations:

- `.github/workflows/use-action.yaml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed action.yml: moved all ${{ inputs.version }}, ${{ github.token }}, and ${{ runner.os }} expressions out of the run: block into an env: block (as INPUTS_VERSION, GITHUB_TOKEN, RUNNER_OS) and updated all shell references to use quoted variables. Fixed .github/workflows/use-action.yaml: added `permissions: {}` at the top level, and pinned all action references to full commit SHAs — actions/setup-go@v2 → bfdd3570ce990073878bf10f6b2d79082de49492, actions/checkout@v2 → 0717577d45739eb3c851188b29f50ed6c0b2194e, imjasonh/setup-crane@main (×3) → feee3b6bb0d4c68370f256a4502498c9227e5c6b.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in action.yml:
1. Line 37: `if [[ $os == "macOS" ]]` → `if [[ "$os" == "macOS" ]]` — quoted $os in the [[ test
2. Line 41: `if [[ ! -z ${tag} ]]` → `if [[ ! -z "${tag}" ]]` — quoted ${tag} in the [[ test
3. Line 43: The curl URL now uses double-quoting around the entire URL string, properly quoting both ${tag} and ${os} expansions to prevent shell metacharacter injection

