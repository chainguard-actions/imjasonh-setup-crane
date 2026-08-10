<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.6** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ github.repository }} expression is directly interpolated inside a `run:` shell command string. This allows the value to be parsed by the shell before quoting can protect it. The offending line is: `crane copy ubuntu ghcr.io/${{ github.repository }}/ubuntu-copy`. While `github.repository` is not directly attacker-controlled via PR, any ${{ }} expression in a run: block is a script-injection risk per the check rules. It should be moved to an `env:` variable and referenced as `"$GITHUB_REPOSITORY"` instead.

Locations:

- `.github/workflows/use-action.yaml:34`

### unpinned-uses (severity: high)

Two `uses:` references in the workflow use mutable tag refs instead of full 40-character commit SHA digests, making the workflow vulnerable to supply-chain attacks if the upstream tag is moved or compromised:
- `uses: actions/setup-go@v5` (line 22) — should be pinned to a full SHA
- `uses: actions/checkout@v4` (line 24) — should be pinned to a full SHA

Locations:

- `.github/workflows/use-action.yaml:22`
- `.github/workflows/use-action.yaml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three findings in .github/workflows/use-action.yaml: (1) Pinned actions/setup-go@v5 to full SHA 40f1582b2485089dde7abd97c1529aa768e1baff; (2) Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262; (3) Moved ${{ github.repository }} expression out of the run: shell string into an env: block as REPO, and referenced it as "$REPO" in the crane copy command to prevent script injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in action.yml that could allow metacharacter injection via a crafted runner.os value: (1) quoted `$os` in the `[[ $os == "macOS" ]]` condition → `[[ "$os" == "macOS" ]]`; (2) quoted `${out}` in `tar xzf "$fname" ${out}` → `tar xzf "$fname" "${out}"`; (3) quoted `${tmp}/${out}` in `chmod +x ${tmp}/${out}` → `chmod +x "${tmp}/${out}"`.

