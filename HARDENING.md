<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yaml references actions by mutable tags instead of full 40-character commit SHAs: 'uses: actions/checkout@v3' (line 37), 'uses: actions/upload-pages-artifact@v2' (line 48), 'uses: actions/deploy-pages@v3' (line 54).

Locations:

- `action.yaml:37`
- `action.yaml:48`
- `action.yaml:54`

### script-injection (severity: high)

Rule (a) violated: the 'Build' step's run: block puts GitHub Actions expressions straight into the shell script: '${{ inputs.install_command }}' (line 43) and '${{ inputs.build_command }}' (line 44). The values are pasted into the script before bash parses it, so a caller-supplied value becomes shell code. To fix it, move both inputs into env: variables (e.g. INSTALL_COMMAND, BUILD_COMMAND) and run them explicitly, for example with bash -c "$INSTALL_COMMAND". These inputs are meant to hold commands, so the fix makes that execution explicit and stops raw text from being pasted into the script.

Locations:

- `action.yaml:43`
- `action.yaml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:43`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:44`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yaml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yaml b/action.yaml
index 3975d71..2a5c3ff 100644
--- a/action.yaml
+++ b/action.yaml
@@ -25,32 +25,48 @@ outputs:
   page_url:
     description: "The URL of the page"
     value: ${{ steps.deploy.outputs.page_url }}
-
-  
-
 runs:
   using: 'composite'
-
   steps:
     - name: Checkout if required
       if: ${{ inputs.checkout == 'true' }}
       uses: actions/checkout@v3
-
     - name: 'Build'
       shell: bash
+      env:
+        INSTALL_CMD: ${{ inputs.install_command }}
+        BUILD_CMD: ${{ inputs.build_command }}
       run: |
         echo "::group::Build"
-        ${{ inputs.install_command }}
-        ${{ inputs.build_command }}
+        _script=$(mktemp)
+        printf '%s\n' "$INSTALL_CMD" > "$_script"
+        printf '%s\n' "$BUILD_CMD" >> "$_script"
+        bash -eo pipefail "$_script"
+        rm -f "$_script"
         echo "::endgroup::"
-
     - name: 'upload'
       uses: actions/upload-pages-artifact@v2
-      with: 
+      with:
         path: ${{ inputs.path }}
-
     - id: deploy
       name: Deploy to GitHub Pages
       uses: actions/deploy-pages@v3
       with:
-        token: ${{ github.token }}
\ No newline at end of file
+        token: ${{ github.token }}

```

Locations:

- `action.yaml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, prior-fix

**Notes:**

Pinned actions/checkout@v3, actions/upload-pages-artifact@v2 and actions/deploy-pages@v3 to full commit SHAs, with the tag kept as a trailing comment. In the Build step, moved install_command and build_command into env vars (INSTALL_CMD, BUILD_CMD). They are now written to a temp script and run with 'bash -eo pipefail', so the step still stops at the first failing command, as upstream's inline version did. This re-applies the previous harden's patch.

