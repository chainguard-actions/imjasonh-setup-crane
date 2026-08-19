<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.5** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml contains multiple direct ${{ }} expression interpolations inside a run: shell block, violating rule (a). (1) Line 22: `case ${{ inputs.version }} in` — inputs.version is attacker-controlled and interpolated directly into a case statement without quoting, also violating rule (b) (unquoted). (2) Line 30: `url="https://github.com/google/go-containerregistry/releases/download/${{ inputs.version }}"` — inputs.version interpolated directly into a URL string. (3) Line 33: `os=${{ runner.os }}` — runner.os interpolated directly and unquoted (rule b: `if [[ $os == ... ]]` uses $os unquoted). (4) Line 61: `echo "${{ github.token }}" | crane auth login ...` — github.token interpolated directly into the shell command. All four are script-injection rule (a) violations; lines 22 and 33 are also rule (b) violations.

Locations:

- `action.yml:22`
- `action.yml:30`
- `action.yml:33`
- `action.yml:61`

### script-injection (severity: high)

The workflow file .github/workflows/use-action.yaml contains a direct ${{ }} expression interpolation inside a run: block, violating rule (a). Line 26: `crane copy ubuntu ghcr.io/${{ github.repository }}/ubuntu-copy` — github.repository is interpolated directly into the shell command string without being routed through an env: variable.

Locations:

- `.github/workflows/use-action.yaml:26`

### unpinned-uses (severity: high)

The workflow file .github/workflows/use-action.yaml references actions using mutable tag refs instead of full 40-character SHA digests: (1) `uses: actions/setup-go@v5` (line 20) — tag ref `v5`; (2) `uses: actions/checkout@v4` (line 22) — tag ref `v4`. These can be silently updated by the action author, enabling supply-chain attacks.

Locations:

- `.github/workflows/use-action.yaml:20`
- `.github/workflows/use-action.yaml:22`

### missing-permissions (severity: medium)

The workflow file .github/workflows/use-action.yaml has no top-level `permissions:` key and no job-level `permissions:` key on the `use-action` job. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

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

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings in action.yml and .github/workflows/use-action.yaml:

1. action.yml script-injection: Added env: block with INPUT_VERSION=${{ inputs.version }}, RUNNER_OS=${{ runner.os }}, GITHUB_TOKEN=${{ github.token }}. Replaced all four inline ${{ }} expressions with plain env var references ($INPUT_VERSION, $RUNNER_OS, $GITHUB_TOKEN). Properly quoted the case statement and os= assignment.

2. use-action.yaml script-injection: Moved ${{ github.repository }} to env: block as GITHUB_REPOSITORY and referenced it as $GITHUB_REPOSITORY in the shell command.

3. use-action.yaml unpinned-uses: Pinned actions/setup-go@v5 to SHA 40f1582b2485089dde7abd97c1529aa768e1baff and actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262.

4. use-action.yaml missing-permissions: Added top-level permissions: {} and job-level permissions: {contents: read, packages: write} (packages: write needed for crane copy to ghcr.io).

