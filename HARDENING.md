<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **imjasonh--setup-crane/v0.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The action.yml run: block directly interpolates ${{ ... }} expressions into shell commands (sub-rule a), allowing script injection. Offending lines:
- Line 22: `case ${{ inputs.version }} in` — attacker-controlled input interpolated directly into a case statement
- Line 28: `tag=$(curl -s -u "username:${{ github.token }}" ...)` — github.token interpolated into curl command
- Line 30: `tag="${{ inputs.version }}"` — attacker-controlled input interpolated into shell variable assignment
- Line 33: `os=${{ runner.os }}` — runner.os interpolated directly into shell
- Line 44: `echo "${{ github.token }}" | crane auth login ...` — github.token interpolated into echo/pipe
All of these should be moved to env: variables and then referenced as quoted shell variables (e.g., "$INPUTS_VERSION").

Locations:

- `action.yml:22`
- `action.yml:28`
- `action.yml:30`
- `action.yml:33`
- `action.yml:44`

### unpinned-uses (severity: high)

The workflow file .github/workflows/use-action.yaml references actions by mutable tags/branches instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved:
- Line 16: `actions/setup-go@v2`
- Line 18: `actions/checkout@v2`
- Line 20: `imjasonh/setup-crane@main`
- Line 24: `imjasonh/setup-crane@main`
- Line 31: `imjasonh/setup-crane@main`
All uses: references should be pinned to a full SHA (e.g., `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`).

Locations:

- `.github/workflows/use-action.yaml:16`
- `.github/workflows/use-action.yaml:18`
- `.github/workflows/use-action.yaml:20`
- `.github/workflows/use-action.yaml:24`
- `.github/workflows/use-action.yaml:31`

### missing-permissions (severity: medium)

The workflow file .github/workflows/use-action.yaml has no top-level `permissions:` key and the single job `use-action` also has no `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be write-all), violating the principle of least privilege. A minimal permissions block (e.g., `permissions: {}` or specific scopes like `contents: read`) should be added.

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

Fixed three categories of findings:
1. script-injection / static-inline-injection (action.yml): Moved all ${{ inputs.version }}, ${{ github.token }}, and ${{ runner.os }} expressions out of the run: shell block into an env: map (INPUTS_VERSION, GITHUB_TOKEN, RUNNER_OS). All shell references now use plain quoted environment variables (e.g., "$INPUTS_VERSION", "$GITHUB_TOKEN", "$RUNNER_OS").
2. unpinned-uses (.github/workflows/use-action.yaml): Pinned actions/setup-go@v2 → @bfdd3570ce990073878bf10f6b2d79082de49492, actions/checkout@v2 → @ee0669bd1cc54295c223e0bb666b733df41de1c5, and all three imjasonh/setup-crane@main → @feee3b6bb0d4c68370f256a4502498c9227e5c6b, with original tag/branch preserved as comments.
3. missing-permissions (.github/workflows/use-action.yaml): Added top-level `permissions: {}` to enforce least privilege.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in action.yml's run block: (1) quoted `$os` in the `[[ $os == "macOS" ]]` conditional → `[[ "$os" == "macOS" ]]`; (2) quoted `${tag}` in the `[[ ! -z ${tag} ]]` conditional → `[[ ! -z "${tag}" ]]`; (3) wrapped the entire curl URL in double quotes so both `${tag}` and `${os}` are properly quoted, preventing shell metacharacter injection.

