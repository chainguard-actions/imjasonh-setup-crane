<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.7** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.repository }}` is interpolated directly inside a `run:` shell command string. This allows the value to be injected into the shell command before the shell ever sees it. The offending line is: `crane copy ubuntu ghcr.io/${{ github.repository }}/ubuntu-copy`. Even though `github.repository` is not directly attacker-controlled via PR, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules.

Locations:

- `.github/workflows/use-action.yaml:35`

### unpinned-uses (severity: high)

Two `uses:` references in the workflow file are pinned to mutable version tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved or the upstream repository is compromised. Failing references: `actions/setup-go@v5` (line 25) and `actions/checkout@v4` (line 26).

Locations:

- `.github/workflows/use-action.yaml:25`
- `.github/workflows/use-action.yaml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in .github/workflows/use-action.yaml: (1) Pinned actions/setup-go@v5 to full SHA 40f1582b2485089dde7abd97c1529aa768e1baff, (2) Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262, (3) Moved ${{ github.repository }} expression out of the run: shell string into an env: block as REPOSITORY, referencing it as $REPOSITORY in the shell script to prevent script injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in .github/workflows/use-action.yaml line 37. The `$REPOSITORY` variable (holding `${{ github.repository }}`) was used unquoted in `crane copy ubuntu ghcr.io/$REPOSITORY/ubuntu-copy`. Changed to `crane copy ubuntu "ghcr.io/$REPOSITORY/ubuntu-copy"` to prevent shell metacharacters from being interpreted. The variable was already correctly placed in the step's `env:` block; only the quoting in the `run:` command needed to be fixed.

