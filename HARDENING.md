<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.11** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. In `.github/workflows/dependabot-dist.yml` line 23, the `run:` block contains `gh pr checkout ${{ github.event.pull_request.number }}`. This workflow is triggered by `pull_request_target`, making this especially dangerous — an attacker could craft a PR with a malicious pull request number to inject shell commands. The value must be passed via an `env:` variable and the variable must be double-quoted in the shell command.

Locations:

- `.github/workflows/dependabot-dist.yml:23`

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable tag refs instead of immutable full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if a tag is moved or hijacked. Failing references include: codeql-analysis.yml: `actions/checkout@v7`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`; dependabot-dist.yml: `actions/checkout@v7`, `actions/setup-node@v6`; release.yml: `actions/checkout@v7`, `actions/setup-node@v6`, `cycjimmy/semantic-release-action@v6`; test.yml: `actions/checkout@v7`, `actions/setup-node@v6`. All should be pinned to their full SHA digest.

Locations:

- `.github/workflows/codeql-analysis.yml:31`
- `.github/workflows/dependabot-dist.yml:17`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:9`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may include broad write access). Affected files: `dependabot-dist.yml`, `release.yml`, and `test.yml`. Each should declare minimal required permissions (e.g., `permissions: contents: read`) at the top level or per job.

Locations:

- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files: (1) script-injection in dependabot-dist.yml: moved PR number expression into env var PR_NUMBER and double-quoted it in the shell command; (2) unpinned-uses: pinned all 6 action references to full 40-char SHAs (actions/checkout@v7, actions/setup-node@v6, github/codeql-action/{init,autobuild,analyze}@v4, cycjimmy/semantic-release-action@v6) with tag comments for readability; (3) missing-permissions: added top-level permissions blocks to dependabot-dist.yml (contents: write, pull-requests: write), release.yml (contents: write), and test.yml (contents: read). codeql-analysis.yml already had job-level permissions so no change was needed there.

