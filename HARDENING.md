<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.33

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.33** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Check the output coverage' step directly interpolates ${{ steps.coverageComment.outputs.* }} expressions inside a run: shell command string. These are steps.*.outputs.* values — a workflow-controllable context — that flow through YAML template substitution before the shell processes them, enabling script injection if the action produces attacker-influenced output. Offending lines include: `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`, `echo "Summary Html - ${{ steps.coverageComment.outputs.summaryHtml }}"`, and similar echo statements for tests, skipped, failures, errors, time, lines, branches, functions, statements, color, and coverageHtml. These should be moved to env: variables and referenced as quoted shell variables (e.g., "$COVERAGE") instead.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:44`

### unpinned-uses (severity: high)

Multiple uses: references use mutable tags or branch names instead of immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Failing references in update-coverage-in-readme.yml: `actions/checkout@v6`, `actions/cache@v5`, `MishaKav/jest-coverage-comment@main` (branch ref), `schneegans/dynamic-badges-action@v1.8.0`. Failing reference in update-main-version.yml: `actions/checkout@v6`.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:15`
- `.github/workflows/update-coverage-in-readme.yml:20`
- `.github/workflows/update-coverage-in-readme.yml:38`
- `.github/workflows/update-coverage-in-readme.yml:65`
- `.github/workflows/update-main-version.yml:18`

### missing-permissions (severity: medium)

The workflow file update-coverage-in-readme.yml has no top-level permissions: key and the single job 'update-coverage-in-readme' also has no job-level permissions: key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to all scopes). A minimal permissions block should be added at the top level or job level.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in update-coverage-in-readme.yml and update-main-version.yml:

1. script-injection: Moved all ${{ steps.coverageComment.outputs.* }} expressions in the 'Check the output coverage' step into an env: block (13 variables: COVERAGE, COLOR, SUMMARY_HTML, TESTS, SKIPPED, FAILURES, ERRORS, TIME, LINES, BRANCHES, FUNCTIONS, STATEMENTS, COVERAGE_HTML). The run: script now references plain shell variables.

2. unpinned-uses: Pinned all 5 action references to full 40-char SHAs with tag comments:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (both files)
   - actions/cache@v5 → @caa296126883cff596d87d8935842f9db880ef25
   - MishaKav/jest-coverage-comment@main → @642ef024cc554a34b7082cea12c7bf63575ae151
   - schneegans/dynamic-badges-action@v1.8.0 → @0e50b8bad39e7e1afd3e4e9c2b7dd145fad07501

3. missing-permissions: Added top-level 'permissions: contents: write' to update-coverage-in-readme.yml (minimum needed for the workflow to function).

