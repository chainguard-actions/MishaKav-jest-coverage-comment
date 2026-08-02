<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.36

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.36** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag or branch refs instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved or compromised.

In `.github/workflows/update-coverage-in-readme.yml`:
- `uses: actions/checkout@v7` (line 16)
- `uses: actions/cache@v6` (line 20)
- `uses: MishaKav/jest-coverage-comment@main` (line 38)
- `uses: schneegans/dynamic-badges-action@v1.9.0` (line 64)

In `.github/workflows/update-main-version.yml`:
- `uses: actions/checkout@v7` (line 18)

Locations:

- `.github/workflows/update-coverage-in-readme.yml:16`
- `.github/workflows/update-coverage-in-readme.yml:20`
- `.github/workflows/update-coverage-in-readme.yml:38`
- `.github/workflows/update-coverage-in-readme.yml:64`
- `.github/workflows/update-main-version.yml:18`

### script-injection (severity: high)

Sub-rule (a): The 'Check the output coverage' run: block in update-coverage-in-readme.yml directly interpolates multiple `${{ steps.coverageComment.outputs.* }}` expressions inside shell commands. These step output values flow through YAML template substitution before the shell sees them, meaning any special characters (semicolons, backticks, etc.) in the output values could be interpreted as shell metacharacters. Affected lines include:
- `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`
- `echo "Coverage Color - ${{ steps.coverageComment.outputs.color }}"`
- `echo "Summary Html - ${{ steps.coverageComment.outputs.summaryHtml }}"`
- and several more `${{ steps.coverageComment.outputs.* }}` interpolations in the same step.
These should be routed through env: variables and the env vars double-quoted in the shell.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:47`

### missing-permissions (severity: medium)

The workflow file `update-coverage-in-readme.yml` has no top-level `permissions:` key and the single job `update-coverage-in-readme` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents and pull requests). A minimal explicit permissions block should be added.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in update-coverage-in-readme.yml and update-main-version.yml:

1. unpinned-uses: Pinned all 5 action references to full 40-char commit SHAs with tag comments preserved:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (both files)
   - actions/cache@v6 → @55cc8345863c7cc4c66a329aec7e433d2d1c52a9
   - MishaKav/jest-coverage-comment@main → @642ef024cc554a34b7082cea12c7bf63575ae151
   - schneegans/dynamic-badges-action@v1.9.0 → @28b0fa8bdeb46170ac397105ece0c1fe58f68910

2. script-injection: Moved all ${{ steps.coverageComment.outputs.* }} expressions in the 'Check the output coverage' step into an env: block (COVERAGE, COLOR, SUMMARY_HTML, TESTS, SKIPPED, FAILURES, ERRORS, TIME, LINES, BRANCHES, FUNCTIONS, STATEMENTS, COVERAGE_HTML) and referenced them as plain shell variables.

3. missing-permissions: Added top-level 'permissions: contents: write' to update-coverage-in-readme.yml (write access needed to update README content).

