<!-- markdownlint-disable -->

# Hardening Report: opt-nc--setup-duckdb-action/v1.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opt-nc--setup-duckdb-action/v1.2.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. In .github/workflows/dependabot-dist.yml, the step 'Checkout out pull request' runs `gh pr checkout ${{ github.event.pull_request.number }}`, embedding the PR number directly into the shell command. This workflow is triggered by pull_request_target, making it especially dangerous — an attacker could craft a PR number value to inject shell metacharacters. The value should be passed via an env: variable and quoted: `env: PR_NUMBER: ${{ github.event.pull_request.number }}` then `gh pr checkout "$PR_NUMBER"`.

Locations:

- `.github/workflows/dependabot-dist.yml:19`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs instead of immutable 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the action repository is compromised. Failing references:
- codeql-analysis.yml: actions/checkout@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4
- dependabot-dist.yml: actions/checkout@v6, actions/setup-node@v6
- release.yml: actions/checkout@v6, actions/setup-node@v6, cycjimmy/semantic-release-action@v6
- test.yml: actions/checkout@v6, actions/setup-node@v6
All should be pinned to full SHA digests, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/dependabot-dist.yml:14`
- `.github/workflows/release.yml:8`
- `.github/workflows/test.yml:8`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions: key and no per-job permissions: key, meaning they run with the default (potentially broad) token permissions. release.yml, dependabot-dist.yml, and test.yml all lack any permissions declaration. Each workflow should declare minimal required permissions explicitly (e.g. `permissions: contents: read`) to follow the principle of least privilege. Note: codeql-analysis.yml correctly declares job-level permissions and passes.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/dependabot-dist.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files: (1) script-injection in dependabot-dist.yml: moved PR number from inline ${{ }} expression in run: to env: block as PR_NUMBER, referenced as "$PR_NUMBER" in shell; (2) unpinned-uses: pinned all 8 action references across codeql-analysis.yml, dependabot-dist.yml, release.yml, and test.yml to full 40-char SHA digests with tag comments; (3) missing-permissions: added top-level permissions blocks to release.yml (contents: write), dependabot-dist.yml (contents: write, pull-requests: write), and test.yml (contents: read). codeql-analysis.yml already had job-level permissions and was not modified for that finding.

