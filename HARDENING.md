<!-- markdownlint-disable -->

# Hardening Report: MishaKav--jest-coverage-comment/v1.0.30

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **MishaKav--jest-coverage-comment/v1.0.30** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file references four Actions using mutable tags/branches instead of full 40-character commit SHA pins. These can be silently updated by the upstream maintainer to inject malicious code:
- `actions/checkout@v6` (line 17)
- `actions/cache@v5` (line 21)
- `MishaKav/jest-coverage-comment@main` (line 42) — uses a branch name, highest risk
- `schneegans/dynamic-badges-action@v1.7.0` (line 65)
All should be pinned to a full SHA, e.g. `actions/checkout@<40-hex-sha> # v6`.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:17`
- `.github/workflows/update-coverage-in-readme.yml:21`
- `.github/workflows/update-coverage-in-readme.yml:42`
- `.github/workflows/update-coverage-in-readme.yml:65`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `update-coverage-in-readme` also has no `permissions:` key. Without explicit permissions, the job inherits the repository's default token permissions (which may be `write-all`). A minimal permissions block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:1`

### script-injection (severity: high)

The 'Check the output coverage' run: block (rule a) directly interpolates `${{ steps.coverageComment.outputs.* }}` expressions inside shell command strings. These step output values flow through YAML template substitution before the shell processes them, meaning any special characters (semicolons, backticks, `$(...)`, etc.) embedded in the output values would be interpreted by the shell. For example:
  `echo "Coverage Percentage - ${{ steps.coverageComment.outputs.coverage }}"`
  `echo "Summary Html - ${{ steps.coverageComment.outputs.summaryHtml }}"`
All `${{ ... }}` expressions must be moved to an `env:` block and referenced as quoted shell variables (e.g. `"$COVERAGE"`) instead.

Locations:

- `.github/workflows/update-coverage-in-readme.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/update-coverage-in-readme.yml:
1. unpinned-uses: Pinned all four actions to full SHAs — actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 (v6), actions/cache@caa296126883cff596d87d8935842f9db880ef25 (v5), MishaKav/jest-coverage-comment@642ef024cc554a34b7082cea12c7bf63575ae151 (main), schneegans/dynamic-badges-action@e9a478b16159b4d31420099ba146cdc50f134483 (v1.7.0).
2. missing-permissions: Added top-level `permissions: contents: read` block.
3. script-injection: Moved all ${{ steps.coverageComment.outputs.* }} expressions from the 'Check the output coverage' run block into an env: block, referencing them as plain shell variables ($COVERAGE, $COLOR, $SUMMARY_HTML, etc.).

