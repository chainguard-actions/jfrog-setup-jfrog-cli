<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli--/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli--/v5.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use action references pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned references found:
- auto-build-publish.yml: actions/checkout@v6, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3
- cla.yml: jfrog/.github/actions/cla@main
- frogbot-scan-pull-request.yml: jfrog/frogbot@v2
- frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2
- oidc-integration-test.yml: actions/checkout@v6
- release.yml: actions/checkout@v6, ad-m/github-push-action@master
- remove-label.yml: actions-ecosystem/action-remove-labels@v1
- test.yml: actions/checkout@v6, actions/setup-node@v6, wei/curl@master

Locations:

- `.github/workflows/auto-build-publish.yml:30`
- `.github/workflows/auto-build-publish.yml:33`
- `.github/workflows/auto-build-publish.yml:36`
- `.github/workflows/auto-build-publish.yml:41`
- `.github/workflows/auto-build-publish.yml:62`
- `.github/workflows/cla.yml:18`
- `.github/workflows/frogbot-scan-pull-request.yml:13`
- `.github/workflows/frogbot-scan-repository.yml:20`
- `.github/workflows/frogbot-scan-repository.yml:22`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:38`
- `.github/workflows/remove-label.yml:13`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:42`

### missing-permissions (severity: medium)

The following workflow files have no top-level 'permissions:' key and no job-level 'permissions:' key on any job, meaning they run with the default (potentially broad) token permissions:
- auto-build-publish.yml
- test.yml
- remove-label.yml

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/remove-label.yml:1`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings, violating sub-rule (a). An attacker or malicious input can inject shell metacharacters before the shell ever sees the value.

- release.yml 'Extract Major and Minor Versions from Tag' step: TAG_NAME="${{ github.event.release.tag_name }}" — github.event.release.tag_name is interpolated directly into the shell script.
- oidc-integration-test.yml 'Create OpenID Connect integration' step: ${{ matrix.audience_id }}, ${{ github.run_id }}, ${{ matrix.audience_value }}, ${{ secrets.JFROG_PLATFORM_URL }}, ${{ secrets.JFROG_PLATFORM_RT_TOKEN }} interpolated directly in curl run block.
- oidc-integration-test.yml 'Create OIDC Identity Mapping' step: same pattern with ${{ github.repository_owner }}, ${{ matrix.audience_id }}, ${{ github.run_id }}.
- oidc-integration-test.yml 'Validate user output' step: test -n "${{ steps.setup-jfrog-cli.outputs.oidc-user }}" — steps output interpolated directly.
- oidc-integration-test.yml 'Validate token output' step: test -n "${{ steps.setup-jfrog-cli.outputs.oidc-token }}" — steps output interpolated directly.
- oidc-integration-test.yml 'Delete OIDC integration' step: ${{ secrets.JFROG_PLATFORM_URL }}, ${{ matrix.audience_id }}, ${{ github.run_id }} interpolated directly.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/oidc-integration-test.yml:48`
- `.github/workflows/oidc-integration-test.yml:62`
- `.github/workflows/oidc-integration-test.yml:84`
- `.github/workflows/oidc-integration-test.yml:87`
- `.github/workflows/oidc-integration-test.yml:131`

### github-env-injection (severity: high)

In release.yml, the 'Extract Major and Minor Versions from Tag' step interpolates ${{ github.event.release.tag_name }} directly into the shell variable TAG_NAME, then writes MAJOR and MINOR (values derived from TAG_NAME via regex match) to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A crafted tag name containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release.yml:16`

### hardcoded-credentials (severity: high)

In test.yml, the env variable JF_ENV_LOCAL is set to a hardcoded base64-encoded value that decodes to a JFrog CLI server configuration JSON containing a hardcoded password: {"version":1,"url":"http://127.0.0.1:8081/artifactory/","user":"admin","password":"APB7DEZRPiHqHDTQokLkx9hhzKT","serverId":"local"}. Even if this is a local test credential, hardcoding passwords in workflow files is a security risk.

Locations:

- `.github/workflows/test.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all findings across 8 workflow files: (1) unpinned-uses: pinned all action references to full commit SHAs with tag comments in auto-build-publish.yml, cla.yml, frogbot-scan-pull-request.yml, frogbot-scan-repository.yml, oidc-integration-test.yml, release.yml, remove-label.yml, and test.yml; (2) missing-permissions: added 'permissions: contents: read' to auto-build-publish.yml and test.yml, and 'permissions: pull-requests: write' to remove-label.yml; (3) script-injection: moved all ${{ }} expressions in run: blocks to env: blocks in release.yml (TAG_NAME) and oidc-integration-test.yml (JFROG_PLATFORM_URL, JFROG_PLATFORM_RT_TOKEN, AUDIENCE_ID, RUN_ID, AUDIENCE_VALUE, REPOSITORY_OWNER, OIDC_USER, OIDC_TOKEN); (4) github-env-injection: in release.yml, moved tag_name to env block and sanitized with printf/tr before writing to GITHUB_ENV; (5) hardcoded-credentials: replaced the hardcoded base64-encoded JFrog credentials in test.yml with a secret reference ${{ secrets.JF_ENV_LOCAL }}.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced both hardcoded 'password' literals in .github/workflows/auto-build-publish.yml with GitHub Actions secret references (${{ secrets.JF_PASSWORD }}). (1) The JF_PASSWORD env var in the 'Setup JFrog CLI' step now uses ${{ secrets.JF_PASSWORD }}. (2) The --password flag in the gacts/run-and-post-run post-step command now references the $JF_PASSWORD environment variable, which is injected via a new env: block on that step using ${{ secrets.JF_PASSWORD }}.

