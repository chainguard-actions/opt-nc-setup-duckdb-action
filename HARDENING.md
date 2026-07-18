<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.8** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. In `.github/workflows/dependabot-dist.yml` line 24, the command `gh pr checkout ${{ github.event.pull_request.number }}` embeds the pull request number directly into the shell. This workflow is triggered by `pull_request_target` (which runs with base-repo permissions), making this especially dangerous — an attacker could craft a PR number or manipulate the event payload to inject shell metacharacters.

Locations:

- `.github/workflows/dependabot-dist.yml:24`

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable version tags instead of full 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if a tag is moved or a dependency is compromised. Failing references:
- `.github/workflows/codeql-analysis.yml`: `actions/checkout@v6`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`
- `.github/workflows/dependabot-dist.yml`: `actions/checkout@v6`, `actions/setup-node@v6`
- `.github/workflows/release.yml`: `actions/checkout@v6`, `actions/setup-node@v6`, `cycjimmy/semantic-release-action@v6`
- `.github/workflows/test.yml`: `actions/checkout@v6`, `actions/setup-node@v6`

Locations:

- `.github/workflows/codeql-analysis.yml:30`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/dependabot-dist.yml:17`
- `.github/workflows/dependabot-dist.yml:26`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:29`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:10`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be `write-all` depending on repository settings), granting broader access than necessary. `codeql-analysis.yml` correctly defines job-level permissions and is not flagged.

Locations:

- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. **script-injection** (dependabot-dist.yml): Moved `${{ github.event.pull_request.number }}` from the `run:` shell command into the step's `env:` block as `PR_NUMBER`, then referenced it as `"$PR_NUMBER"` in the shell script.

2. **unpinned-uses** (all four workflow files): Pinned all mutable version tags to full 40-character commit SHAs:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - github/codeql-action/init@v4 → @7188fc363630916deb702c7fdcf4e481b751f97a
   - github/codeql-action/autobuild@v4 → @7188fc363630916deb702c7fdcf4e481b751f97a
   - github/codeql-action/analyze@v4 → @7188fc363630916deb702c7fdcf4e481b751f97a
   - cycjimmy/semantic-release-action@v6 → @b12c8f6015dc215fe37bc154d4ad456dd3833c90

3. **missing-permissions** (dependabot-dist.yml, release.yml, test.yml): Added top-level `permissions:` blocks with minimal required permissions:
   - dependabot-dist.yml: contents: write, pull-requests: write
   - release.yml: contents: write, issues: write, pull-requests: write
   - test.yml: contents: read

