<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.28

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.28** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow references four actions using mutable tag or branch refs instead of pinned 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved or the branch is updated:
- `actions/checkout@v3` (line 16)
- `actions/cache@v3` (line 20)
- `MishaKav/jest-coverage-comment@main` (line 41) — uses a branch ref
- `schneegans/dynamic-badges-action@v1.6.0` (line 71)
All should be pinned to full commit SHAs.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:16`
- `.github/workflows/update-coverage-in-readme.yml:20`
- `.github/workflows/update-coverage-in-readme.yml:41`
- `.github/workflows/update-coverage-in-readme.yml:71`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `update-coverage-in-readme` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal permissions block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

### script-injection (severity: high)

Sub-rule (a): The 'Check the output coverage' run: block directly interpolates `${{ steps.coverageComment.outputs.* }}` expressions inside shell commands (e.g. `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`). The `steps.*.outputs.*` context is workflow-controllable — if the upstream action (MishaKav/jest-coverage-comment@main) produces output containing shell metacharacters, they will be interpreted by the shell before quoting can protect them. All such expressions should be moved to an `env:` block and referenced as quoted shell variables (e.g. `"$COVERAGE"`) instead.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/update-coverage-in-readme.yml:
1. unpinned-uses: Pinned all four actions to full SHA digests — actions/checkout@v3 → a37ce9120846195fa4ece8f58b268e6043cb2f26, actions/cache@v3 → 6f8efc29b200d32929f49075959781ed54ec270c, MishaKav/jest-coverage-comment@main → 642ef024cc554a34b7082cea12c7bf63575ae151, schneegans/dynamic-badges-action@v1.6.0 → 5d424ad4060f866e4d1dab8f8da0456e6b1c4f56. Original tags/branch preserved as inline comments.
2. missing-permissions: Added top-level `permissions: contents: read` block.
3. script-injection: Moved all 13 `${{ steps.coverageComment.outputs.* }}` expressions from the 'Check the output coverage' run block into a step-level `env:` block, and replaced them with plain shell variable references ($COVERAGE, $COLOR, $SUMMARY_HTML, $TESTS, $SKIPPED, $FAILURES, $ERRORS, $TIME, $LINES, $BRANCHES, $FUNCTIONS, $STATEMENTS, $COVERAGE_HTML).

