<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 2087b79..a404da4 100644
--- a/action.yml
+++ b/action.yml
@@ -11,56 +11,73 @@ inputs:
 runs:
   using: "composite"
   steps:
-  - shell: bash
-    env:
-      VERSION: ${{ inputs.version }}
-      RUNNER_OS: ${{ runner.os }}
-      GITHUB_TOKEN: ${{ github.token }}
-    run: |
-      set -ex
+    - shell: bash
+      env:
+        VERSION: ${{ inputs.version }}
+        RUNNER_OS: ${{ runner.os }}
+        GITHUB_TOKEN: ${{ github.token }}
+      run: |
+        set -ex
 
-      # Install crane:
-      # - if version is "tip", install from tip of main.
-      # - if version is "latest-release", use the latest release URL.
-      # - otherwise, install the specified version.
-      case "${VERSION}" in
-      tip)
-        echo "Installing crane using go get"
-        go install github.com/google/go-containerregistry/cmd/crane@main
-        ;;
-      latest-release)
-        url="https://github.com/google/go-containerregistry/releases/latest/download"
-        ;;
-      *)
-        url="https://github.com/google/go-containerregistry/releases/download/${VERSION}"
-      esac
+        # Install crane:
+        # - if version is "tip", install from tip of main.
+        # - if version is "latest-release", use the latest release URL.
+        # - otherwise, install the specified version.
+        case "${VERSION}" in
+        tip)
+          echo "Installing crane using go get"
+          go install github.com/google/go-containerregistry/cmd/crane@main
+          ;;
+        latest-release)
+          url="https://github.com/google/go-containerregistry/releases/latest/download"
+          ;;
+        *)
+          url="https://github.com/google/go-containerregistry/releases/download/${VERSION}"
+        esac
 
-      os="${RUNNER_OS}"
-      if [[ "$os" == "macOS" ]]; then
-        os="Darwin"
-      fi
+        os="${RUNNER_OS}"
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
 
-      if [[ ! -z "${url}" ]]; then
-        echo "Installing crane for ${os} on ${arch}"
-        tmp=$(mktemp -d)
-        cd "${tmp}"
-        fname="go-containerregistry_${os}_${arch}.tar.gz"
-        curl -o "$fname" -fsSL --retry 5 --retry-delay 1 --retry-all-errors "${url}/${fname}"
-        tar xzf "$fname" "${out}"
-        chmod +x "${tmp}/${out}"
-        PATH="${PATH}:${tmp}"
-        echo "${tmp}" >> "$GITHUB_PATH"
-      fi
+        if [[ ! -z "${url}" ]]; then
+          echo "Installing crane for ${os} on ${arch}"
+          tmp=$(mktemp -d)
+          cd "${tmp}"
+          fname="go-containerregistry_${os}_${arch}.tar.gz"
+          curl -o "$fname" -fsSL --retry 5 --retry-delay 1 --retry-all-errors "${url}/${fname}"
+          tar xzf "$fname" "${out}"
+          chmod +x "${tmp}/${out}"
+          PATH="${PATH}:${tmp}"
+          echo "${tmp}" >> "$GITHUB_PATH"
+        fi
 
-      # NB: username doesn't seem to matter.
-      echo "${GITHUB_TOKEN}" | crane auth login ghcr.io --username "dummy" --password-stdin
+        # NB: username doesn't seem to matter.
+        echo "${GITHUB_TOKEN}" | crane auth login ghcr.io --username "dummy" --password-stdin

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** prior-fix

**Notes:**

Reviewed the previous patch: it only re-indented the composite step by two spaces. No commands, values or references changed, so there was no security change to re-apply. action.yml already passes inputs.version, runner.os and github.token through env: and uses them only as quoted shell variables. It has no uses: or image references to pin and never pipes a download into a shell. Only an mktemp directory goes to $GITHUB_PATH, and the token goes in via --password-stdin. No edits were needed.

