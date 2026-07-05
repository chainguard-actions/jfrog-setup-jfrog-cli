<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli--/v4.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli--/v4.8.1** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags/branches instead of pinned full-length SHA commits, making them vulnerable to supply-chain attacks.

auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3
cla.yml: jfrog/.github/actions/cla@main
frogbot-scan-pull-request.yml: jfrog/frogbot@v2
frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2
oidc-integration-test.yml: actions/checkout@v4
release.yml: actions/checkout@v4, ad-m/github-push-action@master
remove-label.yml: actions-ecosystem/action-remove-labels@v1
test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master

Locations:

- `.github/workflows/auto-build-publish.yml:30`
- `.github/workflows/auto-build-publish.yml:33`
- `.github/workflows/auto-build-publish.yml:36`
- `.github/workflows/auto-build-publish.yml:40`
- `.github/workflows/auto-build-publish.yml:65`
- `.github/workflows/cla.yml:13`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:20`
- `.github/workflows/frogbot-scan-repository.yml:22`
- `.github/workflows/oidc-integration-test.yml:113`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:36`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:26`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:47`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: block and no job-level permissions: blocks, meaning they run with the default (potentially broad) token permissions: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings.

release.yml: TAG_NAME="${{ github.event.release.tag_name }}" — the release tag name is interpolated directly into a shell variable assignment inside a run: block.

oidc-integration-test.yml: Multiple ${{ }} expressions are interpolated directly in run: blocks, including ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }}. These appear in curl commands and test assertions.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/oidc-integration-test.yml:47`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/oidc-integration-test.yml:131`
- `.github/workflows/oidc-integration-test.yml:135`
- `.github/workflows/oidc-integration-test.yml:155`
- `.github/workflows/oidc-integration-test.yml:159`

### github-env-injection (severity: high)

release.yml: The run: block sets TAG_NAME from ${{ github.event.release.tag_name }} (an untrusted input), extracts MAJOR and MINOR from it via regex, and then writes them to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Although the regex match provides some filtering, the values are written directly: echo "MAJOR=$MAJOR" >> $GITHUB_ENV and echo "MINOR=$MINOR" >> $GITHUB_ENV.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:20`

### hardcoded-credentials (severity: high)

test.yml: The env variable JF_ENV_LOCAL is assigned a hardcoded base64-encoded value (eyJ2ZXJzaW9uIjoxLCJ1cmwiOiJodHRwOi8vMTI3LjAuMC4xOjgwODEvYXJ0aWZhY3RvcnkvIiwidXNlciI6ImFkbWluIiwicGFzc3dvcmQiOiJBUEI3REVaUlBpSHFIRFRRb2tMa3g5aGh6S1QiLCJzZXJ2ZXJJZCI6ImxvY2FsIn0=) which decodes to a JSON object containing a password field with a literal credential value. This matches the hardcoded-credentials pattern (password: <literal>).

Locations:

- `.github/workflows/test.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all findings across 8 workflow files: (1) unpinned-uses: Pinned all action references to full commit SHAs with tag comments in auto-build-publish.yml, cla.yml, frogbot-scan-pull-request.yml, frogbot-scan-repository.yml, oidc-integration-test.yml, release.yml, remove-label.yml, and test.yml. (2) missing-permissions: Added minimal permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (pull-requests: write, issues: write), and test.yml (contents: read). (3) script-injection: Moved all ${{ }} expressions in run: blocks to env: blocks in oidc-integration-test.yml (matrix.audience_id, matrix.audience_value, github.run_id, steps.setup-jfrog-cli.outputs.oidc-user, steps.setup-jfrog-cli.outputs.oidc-token) and release.yml (github.event.release.tag_name). (4) github-env-injection: In release.yml, TAG_NAME moved to env block, and MAJOR/MINOR values are sanitized with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_ENV. (5) hardcoded-credentials: Replaced the hardcoded base64 JF_ENV_LOCAL value in test.yml with ${{ secrets.JF_ENV_LOCAL }}.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced both hardcoded 'password' literals in .github/workflows/auto-build-publish.yml with references to ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}. (1) In the 'Post Step to Test the Auto Build-Publish post step', added an env block with LOCAL_ARTIFACTORY_PASSWORD: ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }} and updated the shell command to use "$LOCAL_ARTIFACTORY_PASSWORD" instead of the literal 'password'. (2) In the 'Setup JFrog CLI' step, changed JF_PASSWORD from the hardcoded literal 'password' to ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in the 'Update Major and Minor Tags' step of .github/workflows/release.yml. Changed `git tag -f v$MAJOR` to `git tag -f "v$MAJOR"` and `git tag -f v$MAJOR.$MINOR` to `git tag -f "v$MAJOR.$MINOR"`. The variables $MAJOR and $MINOR are sourced from $GITHUB_ENV (populated from github.event.release.tag_name), and while they are validated as numeric by a regex in the previous step, double-quoting them prevents any potential shell metacharacter injection if validation were bypassed or the env var overridden.

