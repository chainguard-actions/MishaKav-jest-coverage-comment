<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.32

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.32** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- `actions/checkout@v6` (both workflow files)
- `actions/cache@v5` (update-coverage-in-readme.yml)
- `MishaKav/jest-coverage-comment@main` (update-coverage-in-readme.yml)
- `schneegans/dynamic-badges-action@v1.7.0` (update-coverage-in-readme.yml)

Locations:

- `.github/workflows/update-coverage-in-readme.yml:16`
- `.github/workflows/update-coverage-in-readme.yml:20`
- `.github/workflows/update-coverage-in-readme.yml:38`
- `.github/workflows/update-coverage-in-readme.yml:68`
- `.github/workflows/update-main-version.yml:20`

### missing-permissions (severity: medium)

The workflow file `update-coverage-in-readme.yml` has no top-level `permissions:` key, and its only job (`update-coverage-in-readme`) also has no job-level `permissions:` key. This means the workflow runs with the default (broad) repository permissions.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

### script-injection (severity: high)

Rule (a) violation: The 'Check the output coverage' run: block in update-coverage-in-readme.yml directly interpolates `${{ steps.coverageComment.outputs.* }}` expressions inside shell echo commands. These are `steps.*.outputs.*` values — a workflow-controllable context — and are expanded by the template engine before the shell sees them, allowing an attacker who can influence the action's outputs to inject arbitrary shell commands. Offending lines include:
  echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"
  echo "Summary Html - ${{ steps.coverageComment.outputs.summaryHtml }}"
  (and 14 similar lines in the same step)

Locations:

- `.github/workflows/update-coverage-in-readme.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in update-coverage-in-readme.yml and update-main-version.yml:

1. unpinned-uses: Pinned all four action references to full 40-char SHAs with tag comments: actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/cache@v5 → caa296126883cff596d87d8935842f9db880ef25, MishaKav/jest-coverage-comment@main → 642ef024cc554a34b7082cea12c7bf63575ae151, schneegans/dynamic-badges-action@v1.7.0 → e9a478b16159b4d31420099ba146cdc50f134483. Both workflow files updated.

2. missing-permissions: Added top-level 'permissions: contents: read' to update-coverage-in-readme.yml (minimum needed for checkout).

3. script-injection: Moved all 13 ${{ steps.coverageComment.outputs.* }} expressions from the 'Check the output coverage' run: block into a step-level env: block. Shell script now uses plain $VAR references, eliminating template injection risk.

