<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.10** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks.

.github/workflows/codeql-analysis.yml:
  - uses: actions/checkout@v7
  - uses: github/codeql-action/init@v4
  - uses: github/codeql-action/autobuild@v4
  - uses: github/codeql-action/analyze@v4

.github/workflows/dependabot-dist.yml:
  - uses: actions/checkout@v7
  - uses: actions/setup-node@v6

.github/workflows/release.yml:
  - uses: actions/checkout@v7
  - uses: actions/setup-node@v6
  - uses: cycjimmy/semantic-release-action@v6

.github/workflows/test.yml:
  - uses: actions/checkout@v7
  - uses: actions/setup-node@v6

All of these should be pinned to a full 40-character commit SHA (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4).

Locations:

- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:44`
- `.github/workflows/codeql-analysis.yml:57`
- `.github/workflows/dependabot-dist.yml:17`
- `.github/workflows/dependabot-dist.yml:25`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:24`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:10`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in a run: block. In .github/workflows/dependabot-dist.yml, the step 'Checkout out pull request' interpolates `${{ github.event.pull_request.number }}` directly into a shell command:

  run: |
    gh pr checkout ${{ github.event.pull_request.number }}

This workflow is triggered by `pull_request_target`, which runs with write permissions in the context of the base repository. Although `pull_request.number` is an integer and lower risk than string fields, any `${{ ... }}` expression inside a run: block is a script-injection finding per the check rules. The value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting. An attacker-controlled PR number field could be manipulated in edge cases. All expressions must be moved to env: vars and double-quoted in the shell.

Locations:

- `.github/workflows/dependabot-dist.yml:23`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning the GITHUB_TOKEN is granted its default (broad) permissions.

- .github/workflows/dependabot-dist.yml: triggered by pull_request_target (elevated risk) with no permissions restriction.
- .github/workflows/release.yml: no permissions block at top-level or job level.
- .github/workflows/test.yml: no permissions block at top-level or job level.

Each file should declare a top-level `permissions:` block with the minimal scopes required (e.g. `contents: read`), or add job-level `permissions:` to every job.

Locations:

- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 4 workflow files: (1) Pinned all 6 action references to full 40-char SHAs using lookup_action_sha (actions/checkout@v7→3d3c42e, github/codeql-action/*@v4→e4fba86, actions/setup-node@v6→249970, cycjimmy/semantic-release-action@v6→b12c8f6); (2) Fixed script injection in dependabot-dist.yml by moving `${{ github.event.pull_request.number }}` into an env var PR_NUMBER and referencing it as "$PR_NUMBER" in the shell; (3) Added top-level permissions blocks to dependabot-dist.yml (contents: write, pull-requests: write), release.yml (contents: write), and test.yml (contents: read).

