<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The run: block in dependabot-dist.yml directly interpolates the GitHub Actions expression `${{ github.event.pull_request.number }}` into a shell command. This allows an attacker to inject arbitrary shell commands via a crafted pull request number. The offending line is: `gh pr checkout ${{ github.event.pull_request.number }}`

Locations:

- `.github/workflows/dependabot-dist.yml:22`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: actions/checkout@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, actions/setup-node@v6, cycjimmy/semantic-release-action@v6.

Locations:

- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/dependabot-dist.yml:16`
- `.github/workflows/dependabot-dist.yml:24`
- `.github/workflows/release.yml:8`
- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:24`
- `.github/workflows/test.yml:8`
- `.github/workflows/test.yml:9`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Each file should declare minimal required permissions.

Locations:

- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files: (1) script-injection in dependabot-dist.yml: moved `${{ github.event.pull_request.number }}` into the step env block as PR_NUMBER and referenced it as "$PR_NUMBER" in the shell command; (2) unpinned-uses: pinned all six action references to full 40-char commit SHAs (actions/checkout@df4cb1c, actions/setup-node@249970729, github/codeql-action/*@7188fc3, cycjimmy/semantic-release-action@b12c8f6) with tag comments for readability; (3) missing-permissions: added top-level permissions blocks to dependabot-dist.yml (contents: write, pull-requests: write), release.yml (contents: write), and test.yml (contents: read).

