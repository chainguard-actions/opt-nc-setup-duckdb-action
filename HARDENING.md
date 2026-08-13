<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.12** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command. In dependabot-dist.yml the step 'Checkout out pull request' runs `gh pr checkout ${{ github.event.pull_request.number }}`, injecting the PR number directly into the shell command string. Although the value is numeric in practice, any ${{ }} interpolation inside a run: block is a script-injection finding. Fix by routing through an env var: `env: PR_NUMBER: ${{ github.event.pull_request.number }}` and then `gh pr checkout "$PR_NUMBER"`.

Locations:

- `.github/workflows/dependabot-dist.yml:22`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of full 40-character commit SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:
- codeql-analysis.yml: actions/checkout@v7, github/codeql-action/init@v4.37.6, github/codeql-action/autobuild@v4.37.6, github/codeql-action/analyze@v4.37.6
- dependabot-dist.yml: actions/checkout@v7, actions/setup-node@v7
- release.yml: actions/checkout@v7, actions/setup-node@v7, cycjimmy/semantic-release-action@v6
- test.yml: actions/checkout@v7, actions/setup-node@v7
All should be pinned to full SHA digests (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4).

Locations:

- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:39`
- `.github/workflows/codeql-analysis.yml:49`
- `.github/workflows/codeql-analysis.yml:53`
- `.github/workflows/dependabot-dist.yml:16`
- `.github/workflows/dependabot-dist.yml:26`
- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:28`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:10`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions (which may be write-all), violating the principle of least privilege. Affected files: dependabot-dist.yml (job: post-update), release.yml (job: build), test.yml (job: test). Each file should declare minimal required permissions at the top level or per job.

Locations:

- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files: (1) script-injection in dependabot-dist.yml: moved PR number expression into env var PR_NUMBER and referenced as "$PR_NUMBER" in shell; (2) unpinned-uses: pinned all action references to full SHA digests (actions/checkout@v7→3d3c42e5..., actions/setup-node@v7→820762786..., github/codeql-action/*@v4.37.6→5595ccaf..., cycjimmy/semantic-release-action@v6→b12c8f60...); (3) missing-permissions: added job-level permissions blocks to dependabot-dist.yml (contents:write, pull-requests:write), release.yml (contents:write, pull-requests:write), and test.yml (contents:read).

