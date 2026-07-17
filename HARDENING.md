<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jfrog--setup-jfrog-cli/v4.10.1** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks directly interpolate `${{ ... }}` expressions. In `oidc-integration-test.yml`, the 'Create OpenID Connect integration' step embeds `${{ matrix.audience_id }}`, `${{ matrix.audience_value }}`, and `${{ github.run_id }}` directly in a curl shell command; the 'Create OIDC Identity Mapping' step embeds `${{ matrix.audience_id }}`, `${{ github.run_id }}`, and `${{ github.repository_owner }}`; the 'Validate user output' and 'Validate token output' steps embed `${{ steps.setup-jfrog-cli.outputs.oidc-user }}` and `${{ steps.setup-jfrog-cli.outputs.oidc-token }}` directly in run: commands; the 'Delete OIDC integration' step embeds `${{ matrix.audience_id }}` and `${{ github.run_id }}`. In `release.yml`, the 'Extract Major and Minor Versions from Tag' step embeds `${{ github.event.release.tag_name }}` directly in a shell variable assignment. All of these allow an attacker to inject arbitrary shell commands via the interpolated values before the shell ever sees them.

Locations:

- `.github/workflows/oidc-integration-test.yml:48`
- `.github/workflows/oidc-integration-test.yml:63`
- `.github/workflows/oidc-integration-test.yml:148`
- `.github/workflows/oidc-integration-test.yml:151`
- `.github/workflows/oidc-integration-test.yml:170`
- `.github/workflows/release.yml:16`

### github-env-injection (severity: high)

In `release.yml`, the 'Extract Major and Minor Versions from Tag' step assigns `${{ github.event.release.tag_name }}` to the shell variable `TAG_NAME` (direct expression interpolation), then extracts `MAJOR` and `MINOR` from it via a regex match, and writes them to `$GITHUB_ENV` with `echo "MAJOR=$MAJOR" >> $GITHUB_ENV` and `echo "MINOR=$MINOR" >> $GITHUB_ENV` without applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A crafted release tag containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:20`

### hardcoded-credentials (severity: high)

In `auto-build-publish.yml`, the 'Setup JFrog CLI' step sets `JF_PASSWORD: password` — a literal hardcoded plaintext password. This credential is committed to the repository and visible to anyone with read access. It should be stored in a GitHub Actions secret and referenced as `${{ secrets.JF_PASSWORD }}`.

Locations:

- `.github/workflows/auto-build-publish.yml:65`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags, branch names, or version strings instead of immutable 40-character commit SHAs. This exposes the workflows to supply-chain attacks if the referenced tag or branch is moved to point to malicious code. Affected references:
- auto-build-publish.yml: `actions/checkout@v6`, `jfrog/.github/actions/install-go-with-cache@main`, `jfrog/.github/actions/install-local-artifactory@main`, `gacts/run-and-post-run@v1`, `jsdaniell/create-json@v1.2.3`
- cla.yml: `jfrog/.github/actions/cla@main`
- frogbot-scan-pull-request.yml: `jfrog/frogbot@v2`
- frogbot-scan-repository.yml: `jfrog/.github/actions/install-go-with-cache@main`, `jfrog/frogbot@v2`
- oidc-integration-test.yml: `actions/checkout@v6`
- release.yml: `actions/checkout@v6`, `ad-m/github-push-action@master`
- remove-label.yml: `actions-ecosystem/action-remove-labels@v1`
- test.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `wei/curl@master`

Locations:

- `.github/workflows/auto-build-publish.yml:29`
- `.github/workflows/auto-build-publish.yml:32`
- `.github/workflows/auto-build-publish.yml:35`
- `.github/workflows/auto-build-publish.yml:40`
- `.github/workflows/auto-build-publish.yml:68`
- `.github/workflows/cla.yml:17`
- `.github/workflows/frogbot-scan-pull-request.yml:13`
- `.github/workflows/frogbot-scan-repository.yml:13`
- `.github/workflows/frogbot-scan-repository.yml:16`
- `.github/workflows/oidc-integration-test.yml:130`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:34`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:44`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` block and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often `write-all` for older repositories), granting unnecessarily broad access. Affected files: `auto-build-publish.yml`, `release.yml`, `remove-label.yml`, and `test.yml`.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, hardcoded-credentials, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 7 workflow files:

1. **script-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks in oidc-integration-test.yml (5 steps fixed: Create OpenID Connect integration, Create OIDC Identity Mapping, Validate user output, Validate token output, Delete OIDC integration) and release.yml (Extract Major and Minor Versions from Tag step).

2. **github-env-injection**: Fixed in release.yml by moving ${{ github.event.release.tag_name }} to env: block as TAG_NAME, sanitizing with `printf '%s' "$TAG_NAME" | tr -d '\n\r'` before regex matching, and sanitizing MAJOR/MINOR before writing to $GITHUB_ENV.

3. **hardcoded-credentials**: Replaced `JF_PASSWORD: password` with `JF_PASSWORD: ${{ secrets.JF_PASSWORD }}` in auto-build-publish.yml.

4. **unpinned-uses**: Pinned all 11 mutable action references to full 40-character commit SHAs with tag comments across auto-build-publish.yml, cla.yml, frogbot-scan-pull-request.yml, frogbot-scan-repository.yml, oidc-integration-test.yml, release.yml, remove-label.yml, and test.yml.

5. **missing-permissions**: Added minimal permissions blocks to auto-build-publish.yml (contents: read), release.yml (contents: write for tag pushing), remove-label.yml (pull-requests: write), and test.yml (contents: read).

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

In .github/workflows/auto-build-publish.yml, replaced the hardcoded literal password 'password' in the 'jf c add' command with the environment variable "${JF_PASSWORD}". Added an env: block to the 'Post Step to Test the Auto Build-Publish post step' step that maps JF_PASSWORD to ${{ secrets.JF_PASSWORD }}, consistent with how the main Setup JFrog CLI step already handles the password.

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded base64-encoded Artifactory credential in .github/workflows/test.yml (line 18) with a GitHub secret reference. The JF_ENV_LOCAL environment variable now reads from `${{ secrets.JF_ENV_LOCAL }}` instead of embedding the literal base64 value that decoded to a JSON object containing a plaintext password. The credential must now be stored as a repository secret named JF_ENV_LOCAL.

