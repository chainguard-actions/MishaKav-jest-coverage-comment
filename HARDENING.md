<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.29

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.29** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow references four actions using mutable tags or branch names instead of pinned full-length SHA digests, making the workflow vulnerable to supply-chain attacks if those refs are moved:
- `actions/checkout@v5` (line 16)
- `actions/cache@v4` (line 20)
- `MishaKav/jest-coverage-comment@main` (line 40) — branch ref, especially dangerous
- `schneegans/dynamic-badges-action@v1.7.0` (line 72)
All should be pinned to a full 40-character commit SHA.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:16`
- `.github/workflows/update-coverage-in-readme.yml:20`
- `.github/workflows/update-coverage-in-readme.yml:40`
- `.github/workflows/update-coverage-in-readme.yml:72`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `update-coverage-in-readme` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents, pull-requests, etc.).

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

### script-injection (severity: high)

Sub-rule (a): The 'Check the output coverage' run: block directly interpolates `${{ steps.coverageComment.outputs.* }}` expressions inside shell echo commands (lines 49–67). These are `steps.*.outputs.*` values — a workflow-controllable context — that are expanded by the template engine before the shell ever sees them. If the upstream action `MishaKav/jest-coverage-comment@main` (itself pinned to a mutable branch) were compromised or produced malicious output containing shell metacharacters, this would allow arbitrary command execution. Offending lines include:
  `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`
  `echo "Summary Html - ${{ steps.coverageComment.outputs.summaryHtml }}"`
  (and 13 more similar lines)
Fix: move each output into an env: variable and reference it as a quoted shell variable, e.g. `echo "Coverage Percentage - $COVERAGE"`.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/update-coverage-in-readme.yml:
1. unpinned-uses: Pinned all four actions to full SHAs — actions/checkout@v5 → fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09, actions/cache@v4 → 0057852bfaa89a56745cba8c7296529d2fc39830, MishaKav/jest-coverage-comment@main → 642ef024cc554a34b7082cea12c7bf63575ae151, schneegans/dynamic-badges-action@v1.7.0 → e9a478b16159b4d31420099ba146cdc50f134483.
2. missing-permissions: Added top-level `permissions: contents: read` (minimum needed for checkout and running tests).
3. script-injection: Moved all 13 `${{ steps.coverageComment.outputs.* }}` expressions from the run: shell block into an env: block, then referenced them as plain shell variables ($COVERAGE, $COLOR, $SUMMARY_HTML, etc.) to prevent shell metacharacter injection.

