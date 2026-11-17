<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.3** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Build' step puts GitHub Actions expressions straight into a run: script: `${{ inputs.install_command }}` (line 43) and `${{ inputs.build_command }}` (line 44). The input text is pasted into the bash script before the shell runs, so any caller-supplied value runs as code. Move each value into an env: variable such as INSTALL_COMMAND and run it from there. The input is meant to hold a command, so an explicit, documented eval is acceptable, for example `eval "$INSTALL_COMMAND"`. What must go is the template substitution.

Locations:

- `action.yaml:43`
- `action.yaml:44`

### unpinned-uses (severity: high)

Three composite steps use a version tag instead of a full 40-character commit SHA: `uses: actions/checkout@v4` (line 37), `uses: actions/upload-pages-artifact@v3` (line 48) and `uses: actions/deploy-pages@v4` (line 54).

Locations:

- `action.yaml:37`
- `action.yaml:48`
- `action.yaml:54`

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
index b2ebd15..9f54739 100644
--- a/action.yaml
+++ b/action.yaml
@@ -25,36 +25,52 @@ outputs:
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
       uses: actions/checkout@v4
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
+        _script="$(mktemp)"
+        printf '%s\n' "$INSTALL_COMMAND" > "$_script"
+        bash -eo pipefail "$_script"
+        printf '%s\n' "$BUILD_COMMAND" > "$_script"
+        bash -eo pipefail "$_script"
+        rm -f "$_script"
         echo "::endgroup::"
-
     - name: 'upload'
       uses: actions/upload-pages-artifact@v3
-      with: 
+      with:
         path: ${{ inputs.path }}
-
     - id: deploy
       name: Deploy to GitHub Pages
       uses: actions/deploy-pages@v4
       with:
         token: ${{ github.token }}
-
     - name: Print footer
       shell: bash
       run: |
@@ -64,4 +80,3 @@ runs:
         echo "Check the rest of our actions in the [GitHub Marketplace](https://github.com/marketplace?category=&type=actions&verification=&query=bitovi)!"  >> $GITHUB_STEP_SUMMARY
         echo ""  >> $GITHUB_STEP_SUMMARY
         echo "You can get help or ask questions on our [Discord Channel](https://discord.gg/zAHn4JBVcX), or set up a free consultation on our [platform engineering website](https://www.bitovi.com/services/devops-consulting/platform-engineering)." >> $GITHUB_STEP_SUMMARY
-   

```

Locations:

- `action.yaml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, prior-fix

**Notes:**

I fixed all four findings in action.yaml.

- **Script injection (also covers the prior-fix finding):** In the Build step, `install_command` and `build_command` no longer get pasted into the script as `${{ }}` expressions. They are now passed in through `env:` as INSTALL_COMMAND and BUILD_COMMAND. The step writes both commands into one temp script and runs it with `bash -eo pipefail`, so it still stops at the first failing line, as upstream's inline version did.
- **One difference from the previous harden's patch:** that patch ran the two commands in separate shells. I run them in a single shell, as upstream did, so a `cd` or exported variable in the install command still carries over to the build command.
- **Unpinned actions:** each `uses:` is now pinned to the full commit SHA that `lookup_action_sha` returned for its tag, with the tag kept as a trailing comment: actions/checkout v4 (11d5960a326750d5838078e36cf38b85af677262), actions/upload-pages-artifact v3 (56afc609e74202658d3ffba0e8f6dda462b719fa) and actions/deploy-pages v4 (d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e).

The action hasn't been run since these edits.

