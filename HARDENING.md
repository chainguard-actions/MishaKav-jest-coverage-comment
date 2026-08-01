<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.35

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.35** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if those tags/branches are moved or compromised.

In `.github/workflows/update-coverage-in-readme.yml`:
- `uses: actions/checkout@v7` (tag)
- `uses: actions/cache@v6` (tag)
- `uses: MishaKav/jest-coverage-comment@main` (branch — especially dangerous)
- `uses: schneegans/dynamic-badges-action@v1.9.0` (tag)

In `.github/workflows/update-main-version.yml`:
- `uses: actions/checkout@v7` (tag)

All should be pinned to full 40-character hex commit SHAs, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:16`
- `.github/workflows/update-coverage-in-readme.yml:21`
- `.github/workflows/update-coverage-in-readme.yml:38`
- `.github/workflows/update-coverage-in-readme.yml:57`
- `.github/workflows/update-main-version.yml:20`

### script-injection (severity: high)

Sub-rule (a): The 'Check the output coverage' `run:` block in `update-coverage-in-readme.yml` directly interpolates `${{ steps.coverageComment.outputs.* }}` expressions inside shell commands. These `steps.*.outputs.*` values are workflow-controllable and flow through YAML template substitution before the shell sees them, allowing an attacker who can influence the action's outputs (e.g. via a malicious coverage file or a compromised `MishaKav/jest-coverage-comment@main` action) to inject arbitrary shell commands.

Offending lines include:
  `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`
  `echo "Summary Html - ${{ steps.coverageComment.outputs.summaryHtml }}"`
  (and 13 more similar lines)

Fix: move each output into an `env:` variable and reference it as a quoted `"$VAR"` in the shell script, with no `${{ }}` inside the `run:` block.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:44`

### missing-permissions (severity: medium)

The workflow file `update-coverage-in-readme.yml` has no top-level `permissions:` key and its only job (`update-coverage-in-readme`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. `write` on all scopes for private repositories or repositories with permissive defaults). A minimal explicit `permissions:` block should be added — for example `permissions: contents: read` — and any additional scopes granted only as needed.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in update-coverage-in-readme.yml and update-main-version.yml:

1. unpinned-uses: Pinned all 5 action references to full 40-char SHAs:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 (both files)
   - actions/cache@v6 → @55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6
   - MishaKav/jest-coverage-comment@main → @30a48d70e2a76ff9cf0e0b348e487c865bfea4dc # main
   - schneegans/dynamic-badges-action@v1.9.0 → @28b0fa8bdeb46170ac397105ece0c1fe58f68910 # v1.9.0

2. script-injection: Moved all 13 ${{ steps.coverageComment.outputs.* }} expressions from the 'Check the output coverage' run: block into an env: block (COVERAGE, COLOR, SUMMARY_HTML, TESTS, SKIPPED, FAILURES, ERRORS, TIME, LINES, BRANCHES, FUNCTIONS, STATEMENTS, COVERAGE_HTML). Shell script now references plain $VAR names.

3. missing-permissions: Added top-level `permissions: contents: write` to update-coverage-in-readme.yml (write needed to update README). The update-main-version.yml already had job-level permissions: contents: write.

