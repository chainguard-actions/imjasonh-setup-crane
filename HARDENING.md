<!-- markdownlint-disable -->

# Hardening Report: imjasonh--setup-crane/v0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **imjasonh--setup-crane/v0.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): two shell variables holding workflow-controlled data are expanded without double quotes in the composite step's run: block. Line 53 has `if [[ ! -z ${url} ]]; then`. `url` is built from the `VERSION` env var, which comes from `inputs.version`. Line 39 has `if [[ $os == "macOS" ]]; then`. `os` comes from `RUNNER_OS`. Both sit inside `[[ ]]`, where bash does not split words, so the real risk is small. The check still requires them quoted: use `"${url}"` and `"$os"`.

Locations:

- `action.yml:53`
- `action.yml:39`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 672982a..9066110 100644
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
-      if [[ $os == "macOS" ]]; then
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
+          tar xzf "$fname" "${out}"
+          chmod +x "${tmp}/${out}"
+          PATH=${PATH}:${tmp}
+          echo "${tmp}" >> $GITHUB_PATH
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

**Fixes applied:** script-injection, prior-fix

**Notes:**

I quoted the shell variables in the run: block of action.yml: "${url}" (line 53) and "$os" (line 39). I also re-applied the previous harden's quoting: "${out}" in tar and "${tmp}/${out}" in chmod. Beyond that I quoted cd "${tmp}", the PATH assignment and "$GITHUB_PATH". Nothing was run or tested. Behaviour should not change: when version is 'tip', url is never set and the empty-string check still skips the download. I kept the script's indentation as it was. The previous patch also re-indented the steps, but that is only formatting.

