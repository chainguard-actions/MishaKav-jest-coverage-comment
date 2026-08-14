<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.34

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.34** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Check the output coverage' run: block directly interpolates ${{ steps.coverageComment.outputs.* }} expressions inside shell commands (sub-rule a). These steps.*.outputs.* values flow through YAML template substitution before the shell processes them, enabling script injection if any output contains shell metacharacters. Offending lines include: `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`, `echo "Coverage Color - ${{ steps.coverageComment.outputs.color }}"`, and many more similar echo statements.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:43`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or branch names instead of full 40-character commit SHAs. Failing references in update-coverage-in-readme.yml: actions/checkout@v6, actions/cache@v5, MishaKav/jest-coverage-comment@main (branch ref — highest risk), schneegans/dynamic-badges-action@v1.8.0. Failing reference in update-main-version.yml: actions/checkout@v6.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:15`
- `.github/workflows/update-coverage-in-readme.yml:20`
- `.github/workflows/update-coverage-in-readme.yml:33`
- `.github/workflows/update-coverage-in-readme.yml:65`
- `.github/workflows/update-main-version.yml:19`

### missing-permissions (severity: medium)

The workflow file update-coverage-in-readme.yml has no top-level permissions: key and its only job (update-coverage-in-readme) also has no job-level permissions: key. This means the job runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in update-coverage-in-readme.yml and update-main-version.yml:

1. script-injection: Moved all ${{ steps.coverageComment.outputs.* }} expressions in the 'Check the output coverage' step into the step's env: block (13 variables: COVERAGE, COLOR, SUMMARY_HTML, TESTS, SKIPPED, FAILURES, ERRORS, TIME, LINES, BRANCHES, FUNCTIONS, STATEMENTS, COVERAGE_HTML). The run: block now uses plain shell variable references.

2. unpinned-uses: Pinned all four actions in update-coverage-in-readme.yml and one in update-main-version.yml to full 40-character commit SHAs with tag comments for readability: actions/checkout@v6→d23441a48e516b6c34aea4fa41551a30e30af803, actions/cache@v5→caa296126883cff596d87d8935842f9db880ef25, MishaKav/jest-coverage-comment@main→58072a21b8b7f84d36a6e21b5d9a0cad08bc9d75, schneegans/dynamic-badges-action@v1.8.0→0e50b8bad39e7e1afd3e4e9c2b7dd145fad07501.

3. missing-permissions: Added top-level permissions: contents: write and job-level permissions: contents: write to update-coverage-in-readme.yml (write access needed to update README). update-main-version.yml already had job-level permissions.

