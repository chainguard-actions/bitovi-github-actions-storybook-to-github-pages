<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) is violated in the composite step 'Build'. Two input expressions are placed straight into the run: script, so the shell runs whatever the caller passes: `${{ inputs.install_command }}` and `${{ inputs.build_command }}`. These inputs are meant to hold commands, but because the values are substituted into the script text, any shell syntax in them is parsed and executed. To fix this, pass them through env: (e.g. INSTALL_COMMAND/BUILD_COMMAND) and run them deliberately, for example with bash -c "$INSTALL_COMMAND". The README should also state that these inputs must come only from trusted sources.

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
index 3fe4c57..515e747 100644
--- a/action.yaml
+++ b/action.yaml
@@ -25,36 +25,54 @@ outputs:
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
       uses: actions/checkout@08c6903cd8c0fde910a37f88322edcfb5dd907a8
-
     - name: 'Build'
       shell: bash
+      env:
+        INSTALL_COMMAND: ${{ inputs.install_command }}
+        BUILD_COMMAND: ${{ inputs.build_command }}
       run: |
         echo "::group::Build"
-        ${{ inputs.install_command }}
-        ${{ inputs.build_command }}
+        _install_script=$(mktemp)
+        printf '%s\n' "$INSTALL_COMMAND" > "$_install_script"
+        bash -eo pipefail "$_install_script"
+        rm -f "$_install_script"
+        _build_script=$(mktemp)
+        printf '%s\n' "$BUILD_COMMAND" > "$_build_script"
+        bash -eo pipefail "$_build_script"
+        rm -f "$_build_script"
         echo "::endgroup::"
-
     - name: 'upload'
       uses: actions/upload-pages-artifact@7b1f4a764d45c48632c6b24a0339c27f5614fb0b
-      with: 
+      with:
         path: ${{ inputs.path }}
-
     - id: deploy
       name: Deploy to GitHub Pages
       uses: actions/deploy-pages@d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e
       with:
         token: ${{ github.token }}
-
     - name: Print footer
       shell: bash
       run: |
@@ -64,4 +82,3 @@ runs:
         echo "Check the rest of our actions in the [GitHub Marketplace](https://github.com/marketplace?category=&type=actions&verification=&query=bitovi)!"  >> $GITHUB_STEP_SUMMARY
         echo ""  >> $GITHUB_STEP_SUMMARY
         echo "You can get help or ask questions on our [Discord Channel](https://discord.gg/zAHn4JBVcX), or set up a free consultation on our [platform engineering website](https://www.bitovi.com/services/devops-consulting/platform-engineering)." >> $GITHUB_STEP_SUMMARY
-   

```

Locations:

- `action.yaml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

Moved inputs.install_command and inputs.build_command into the Build step's env: block (INSTALL_COMMAND and BUILD_COMMAND). Both commands are now written to a temp script and run with 'bash -eo pipefail', which keeps upstream's errexit behavior. They go into one script so the build command still sees any directory change or variables the install command sets, as it did when upstream inlined them in a single shell. The other steps were already pinned to SHAs.

