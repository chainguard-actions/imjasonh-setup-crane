<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.5** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are placed straight into the composite step's run: script, so the template is filled in before bash parses it. Offending lines: `case ${{ inputs.version }} in` (line 22, also unquoted), `url="https://github.com/google/go-containerregistry/releases/download/${{ inputs.version }}"` (line 31), `os=${{ runner.os }}` (line 34) and `echo "${{ github.token }}" | crane auth login ...` (line 59). A crafted version input, e.g. one containing `; cmd` or `$(cmd)`, would run arbitrary commands. Fix: map inputs.version, runner.os and github.token into env: variables (or use $RUNNER_OS directly) and always expand them double-quoted, e.g. `case "$VERSION" in`.

Locations:

- `action.yml:22`
- `action.yml:31`
- `action.yml:34`
- `action.yml:59`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:31`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 7436f77..70a5463 100644
--- a/action.yml
+++ b/action.yml
@@ -11,52 +11,73 @@ inputs:
 runs:
   using: "composite"
   steps:
-  - shell: bash
-    run: |
-      set -ex
+    - shell: bash
+      env:
+        INPUT_VERSION: ${{ inputs.version }}
+        RUNNER_OS: ${{ runner.os }}
+        GITHUB_TOKEN: ${{ github.token }}
+      run: |
+        set -ex
 
-      # Install crane:
-      # - if version is "tip", install from tip of main.
-      # - if version is "latest-release", use the latest release URL.
-      # - otherwise, install the specified version.
-      case ${{ inputs.version }} in
-      tip)
-        echo "Installing crane using go get"
-        go install github.com/google/go-containerregistry/cmd/crane@main
-        ;;
-      latest-release)
-        url="https://github.com/google/go-containerregistry/releases/latest/download"
-        ;;
-      *)
-        url="https://github.com/google/go-containerregistry/releases/download/${{ inputs.version }}"
-      esac
+        # Install crane:
+        # - if version is "tip", install from tip of main.
+        # - if version is "latest-release", use the latest release URL.
+        # - otherwise, install the specified version.
+        case "$INPUT_VERSION" in
+        tip)
+          echo "Installing crane using go get"
+          go install github.com/google/go-containerregistry/cmd/crane@main
+          ;;
+        latest-release)
+          url="https://github.com/google/go-containerregistry/releases/latest/download"
+          ;;
+        *)
+          url="https://github.com/google/go-containerregistry/releases/download/$INPUT_VERSION"
+        esac
 
-      os=${{ runner.os }}
-      if [[ $os == "macOS" ]]; then
-        os="Darwin"
-      fi
+        os="$RUNNER_OS"
+        if [[ "$os" == "macOS" ]]; then
+          os="Darwin"
+        fi
 
-      arch=$(uname -m)
-      if [[ "$arch" =~ (aarch64|arm64) ]] ; then
-        arch=arm64
-      fi
+        arch=$(uname -m)
+        if [[ "$arch" =~ (aarch64|arm64) ]] ; then
+          arch=arm64
+        fi
 
-      out=crane
-      if [[ "${os}" == "Windows" ]]; then
-        out=crane.exe
-      fi
+        out=crane
+        if [[ "${os}" == "Windows" ]]; then
+          out=crane.exe
+        fi
 
-      if [[ ! -z ${url} ]]; then
-        echo "Installing crane for ${os} on ${arch}"
-        tmp=$(mktemp -d)
-        cd ${tmp}
-        fname="go-containerregistry_${os}_${arch}.tar.gz"
-        curl -o "$fname" -fsSL --retry 5 --retry-delay 1 --retry-all-errors "${url}/${fname}"
-        tar xzf "$fname" ${out}
-        chmod +x ${tmp}/${out}
-        PATH=${PATH}:${tmp}
-        echo "${tmp}" >> $GITHUB_PATH
-      fi
+        if [[ ! -z ${url} ]]; then
+          echo "Installing crane for ${os} on ${arch}"
+          tmp=$(mktemp -d)
+          cd ${tmp}
+          fname="go-containerregistry_${os}_${arch}.tar.gz"
+          curl -o "$fname" -fsSL --retry 5 --retry-delay 1 --retry-all-errors "${url}/${fname}"
+          tar xzf "$fname" ${out}
+          chmod +x ${tmp}/${out}
+          PATH=${PATH}:${tmp}
+          echo "${tmp}" >> $GITHUB_PATH
+        fi
 
-      # NB: username doesn't seem to matter.
-      echo "${{ github.token }}" | crane auth login ghcr.io --username "dummy" --password-stdin
+        # NB: username doesn't seem to matter.
+        echo "$GITHUB_TOKEN" | crane auth login ghcr.io --username "dummy" --password-stdin

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

Moved inputs.version, runner.os and github.token out of the run: script into env: (INPUT_VERSION, RUNNER_OS_NAME, GH_TOKEN_VALUE), and every use of them is now double-quoted (case "$INPUT_VERSION", the download URL, os="$RUNNER_OS_NAME", and the token piped into crane auth login). Also quoted the tmp/out/url/GITHUB_PATH expansions. Install behavior and arguments are the same as upstream. This re-applies the security changes from the previous harden.

