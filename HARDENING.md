<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.9** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved.

- codeql-analysis.yml: `actions/checkout@v6`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`
- dependabot-dist.yml: `actions/checkout@v6`, `actions/setup-node@v6`
- release.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `cycjimmy/semantic-release-action@v6`
- test.yml: `actions/checkout@v6`, `actions/setup-node@v6`

Locations:

- `.github/workflows/codeql-analysis.yml:39`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/codeql-analysis.yml:63`
- `.github/workflows/dependabot-dist.yml:16`
- `.github/workflows/dependabot-dist.yml:24`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:29`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:10`

### script-injection (severity: high)

Sub-rule (a): In `dependabot-dist.yml`, the `run:` block on line 22 directly interpolates a GitHub Actions expression into a shell command: `gh pr checkout ${{ github.event.pull_request.number }}`. Although the job has an `if: github.actor == 'dependabot[bot]'` guard, the expression is still expanded by the Actions template engine before the shell executes it, meaning a crafted PR number value could inject shell metacharacters. The value should be passed via an `env:` variable and double-quoted in the shell command instead.

Locations:

- `.github/workflows/dependabot-dist.yml:22`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. `write` on `contents` by default for many repos). Each workflow should declare the minimal permissions required.

- `dependabot-dist.yml`: triggered on `pull_request_target` (a privileged trigger) with no permissions declared — particularly risky.
- `release.yml`: no permissions declared.
- `test.yml`: no permissions declared.

Locations:

- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across four workflow files:

1. **unpinned-uses**: Pinned all 6 action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6 → df4cb1c069e1874edd31b4311f1884172cec0e10
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38
   - github/codeql-action/{init,autobuild,analyze}@v4 → 7188fc363630916deb702c7fdcf4e481b751f97a
   - cycjimmy/semantic-release-action@v6 → b12c8f6015dc215fe37bc154d4ad456dd3833c90

2. **script-injection**: In dependabot-dist.yml line 22, moved `${{ github.event.pull_request.number }}` into an env: variable (PR_NUMBER) and referenced it as double-quoted `"$PR_NUMBER"` in the shell command.

3. **missing-permissions**: Added top-level permissions blocks to all three affected workflows:
   - dependabot-dist.yml: `contents: write, pull-requests: write` (needs to push commits and interact with PRs)
   - release.yml: `contents: write, issues: write, pull-requests: write` (needed by semantic-release and git push)
   - test.yml: `contents: read` (only needs to read the repository)

