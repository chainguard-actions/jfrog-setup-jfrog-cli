<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli--/v4.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli--/v4.8.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags/branches instead of pinned SHA digests. Failing references: auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v4; release.yml: actions/checkout@v4, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:28`
- `.github/workflows/cla.yml:12`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:19`
- `.github/workflows/oidc-integration-test.yml:100`
- `.github/workflows/release.yml:10`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:21`

### missing-permissions (severity: medium)

These workflow files have no top-level 'permissions:' key and no job-level 'permissions:' key on any job, meaning they run with the default (potentially write) token permissions: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### hardcoded-credentials (severity: high)

test.yml sets the env var JF_ENV_LOCAL to a hardcoded base64-encoded value that decodes to a JSON object containing a literal 'password' field ("password":"APB7DEZRPiHqHDTQokLkx9hhzKT"). This is a hardcoded credential embedded directly in the workflow file.

Locations:

- `.github/workflows/test.yml:14`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In release.yml, the step 'Extract Major and Minor Versions from Tag' uses TAG_NAME="${{ github.event.release.tag_name }}" directly in a run: block — an attacker-controlled release tag name is injected into the shell. In oidc-integration-test.yml, multiple run: blocks interpolate ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ secrets.JFROG_PLATFORM_URL }}, ${{ secrets.JFROG_PLATFORM_RT_TOKEN }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} directly into shell commands.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/oidc-integration-test.yml:46`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/oidc-integration-test.yml:113`
- `.github/workflows/oidc-integration-test.yml:116`
- `.github/workflows/oidc-integration-test.yml:152`
- `.github/workflows/oidc-integration-test.yml:157`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, hardcoded-credentials, script-injection

**Notes:**

Fixed all 4 findings across 8 workflow files:

1. unpinned-uses: Pinned all 11 action references to full commit SHAs with tag comments preserved. Files: auto-build-publish.yml, cla.yml, frogbot-scan-pull-request.yml, frogbot-scan-repository.yml, oidc-integration-test.yml, release.yml, remove-label.yml, test.yml.

2. missing-permissions: Added top-level permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (pull-requests: write, issues: write), and test.yml (contents: read).

3. hardcoded-credentials: Replaced the hardcoded base64-encoded JF_ENV_LOCAL value (containing a plaintext password) in test.yml with ${{ secrets.JF_ENV_LOCAL }}.

4. script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks. In release.yml, TAG_NAME is now set via env. In oidc-integration-test.yml, all secrets, matrix values, github context values, and step outputs are now referenced via env variables (JFROG_PLATFORM_URL, JFROG_PLATFORM_RT_TOKEN, AUDIENCE_ID, AUDIENCE_VALUE, RUN_ID, REPOSITORY_OWNER, OIDC_USER, OIDC_TOKEN) in the shell scripts.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced hardcoded literal 'password' credentials in .github/workflows/auto-build-publish.yml with secrets references. In the 'Post Step to Test the Auto Build-Publish post step', added an env block with ARTIFACTORY_PASSWORD: ${{ secrets.ARTIFACTORY_PASSWORD }} and updated the --password argument to use "$ARTIFACTORY_PASSWORD". In the 'Setup JFrog CLI' step, replaced JF_PASSWORD: password with JF_PASSWORD: ${{ secrets.ARTIFACTORY_PASSWORD }}.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/release.yml, added sanitization for MAJOR and MINOR values before writing to $GITHUB_ENV. Introduced safe_major and safe_minor variables using `printf '%s' "$VAR" | tr -d '\n\r'` to strip newline/carriage-return characters. The TAG_NAME was already correctly placed in the step's env block. Also quoted $GITHUB_ENV references for best practice.

