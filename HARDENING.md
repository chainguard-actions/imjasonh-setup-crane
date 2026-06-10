<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **imjasonh--setup-crane/v0.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: Multiple unquoted shell variable expansions of workflow-controllable data in the run: block.

1. `${out}` is derived from `$RUNNER_OS` (set via `${{ runner.os }}` in env:) and is used unquoted in two commands:
   - `tar xzf "$fname" ${out}` (line 55)
   - `chmod +x ${tmp}/${out}` (line 56)
   An attacker-controlled `runner.os` value containing shell metacharacters could be exploited.

2. `$os` (also derived from `$RUNNER_OS` / `${{ runner.os }}`) is used unquoted in `if [[ $os == "macOS" ]]; then` (line 36). While `[[` prevents word splitting, this is still an unquoted expansion of a workflow-controllable value.

3. `${url}` (which may embed `${VERSION}` from `${{ inputs.version }}`) is used unquoted in `if [[ ! -z ${url} ]]; then` (line 50).

All of these should use double-quoted expansions: `"${out}"`, `"$os"`, `"${url}"`.

Locations:

- `action.yml:36`
- `action.yml:50`
- `action.yml:55`
- `action.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed four unquoted shell variable expansions in action.yml that could allow script injection via workflow-controllable values:
1. Quoted `$os` in the `[[ $os == "macOS" ]]` condition (line 36) → `[[ "$os" == "macOS" ]]`
2. Quoted `${url}` in the `[[ ! -z ${url} ]]` condition (line 50) → `[[ ! -z "${url}" ]]`
3. Quoted `${out}` in the `tar xzf "$fname" ${out}` command (line 55) → `tar xzf "$fname" "${out}"`
4. Quoted `${tmp}/${out}` in the `chmod +x ${tmp}/${out}` command (line 56) → `chmod +x "${tmp}/${out}"`

All variables are already sourced from the env: block (not directly from ${{ }} expressions in the run: block), so no additional env: block changes were needed — only proper quoting was required.

