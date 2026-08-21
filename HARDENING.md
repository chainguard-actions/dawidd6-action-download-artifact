<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-download-artifact/v24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-download-artifact/v24** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in download.yml directly interpolate `${{ steps.download.outputs.found_artifact }}` inside shell commands. The `steps.*.outputs.*` context is a workflow-controllable source and must never appear directly inside a `run:` block. Offending lines:
  - `run: test ${{ steps.download.outputs.found_artifact }} == true`
  - `run: test ${{ steps.download.outputs.found_artifact }} == false`
These should be moved to an `env:` variable and the shell variable double-quoted.

Locations:

- `.github/workflows/download.yml:213`
- `.github/workflows/download.yml:228`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks:

**.github/workflows/download.yml**: 18 occurrences of `uses: actions/checkout@v7` (tag `v7`).

**.github/workflows/upload.yml**: 4 occurrences of `uses: actions/upload-artifact@v7` (tag `v7`).

**.github/workflows/npm-updates.yml**: `uses: dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master` (branch `master`).

All should be pinned to full SHA digests, e.g. `actions/checkout@<40-char-sha> # v7`.

Locations:

- `.github/workflows/download.yml:27`
- `.github/workflows/upload.yml:20`
- `.github/workflows/npm-updates.yml:7`

### missing-permissions (severity: medium)

`.github/workflows/npm-updates.yml` has no top-level `permissions:` key and its single job (`npm-updates`) also has no job-level `permissions:` key. Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions:` block (e.g. `permissions: {}` or only the scopes actually needed) should be added at the top level.

Locations:

- `.github/workflows/npm-updates.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) Script injection in download.yml: moved `${{ steps.download.outputs.found_artifact }}` out of two `run:` blocks into `env:` variables (`FOUND_ARTIFACT`) and double-quoted the shell references. (2) Unpinned uses: pinned `actions/checkout@v7` → SHA `3d3c42e5aac5ba805825da76410c181273ba90b1` (18 occurrences in download.yml), `actions/upload-artifact@v7` → SHA `043fb46d1a93c77aae656e7c1c64a875d1fc6a0a` (4 occurrences in upload.yml), and `dawidd6/reusable-workflows@master` → SHA `eef24d408f08a926601a42fd4051807bcf3d3569` in npm-updates.yml. (3) Missing permissions: added `permissions: {}` top-level block to npm-updates.yml.

