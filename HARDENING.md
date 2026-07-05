<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli--/v4.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli--/v4.10.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned references found:
- auto-build-publish.yml: actions/checkout@v6, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3
- cla.yml: jfrog/.github/actions/cla@main
- frogbot-scan-pull-request.yml: jfrog/frogbot@v2
- frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2
- oidc-integration-test.yml: actions/checkout@v6
- release.yml: actions/checkout@v6, ad-m/github-push-action@master
- remove-label.yml: actions-ecosystem/action-remove-labels@v1
- test.yml: actions/checkout@v6, actions/setup-node@v6, wei/curl@master

Locations:

- `.github/workflows/auto-build-publish.yml:29`
- `.github/workflows/auto-build-publish.yml:33`
- `.github/workflows/auto-build-publish.yml:37`
- `.github/workflows/auto-build-publish.yml:42`
- `.github/workflows/auto-build-publish.yml:67`
- `.github/workflows/cla.yml:18`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:19`
- `.github/workflows/frogbot-scan-repository.yml:21`
- `.github/workflows/oidc-integration-test.yml:115`
- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:36`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:40`

### missing-permissions (severity: medium)

The following workflow files have no top-level 'permissions:' key and no job-level 'permissions:' keys, meaning they run with the default (potentially broad) token permissions:
- auto-build-publish.yml: triggered by pull_request_target with no permissions restriction
- release.yml: triggered by release events with no permissions restriction
- remove-label.yml: triggered by pull_request_target with no permissions restriction
- test.yml: triggered by push/pull_request with no permissions restriction

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell command strings. In release.yml, 'github.event.release.tag_name' is interpolated directly into a shell variable assignment: TAG_NAME="${{ github.event.release.tag_name }}". In oidc-integration-test.yml, matrix and github context values (${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, ${{ steps.setup-jfrog-cli.outputs.oidc-token }}) are interpolated directly into run: shell commands.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/oidc-integration-test.yml:48`
- `.github/workflows/oidc-integration-test.yml:60`
- `.github/workflows/oidc-integration-test.yml:130`
- `.github/workflows/oidc-integration-test.yml:133`

### github-env-injection (severity: high)

In release.yml, the value of 'github.event.release.tag_name' is interpolated directly into the shell via ${{ github.event.release.tag_name }}, assigned to TAG_NAME, then regex-extracted into MAJOR and MINOR variables which are written to $GITHUB_ENV without sanitization (no 'printf | tr -d newlines' step). An attacker who can create a release with a crafted tag name containing newlines could inject arbitrary environment variables.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all findings across 7 workflow files:

1. **unpinned-uses**: Pinned all mutable references to full 40-char SHAs:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
   - actions/setup-node@v6 → @48b55a011bda9f5d6aeb4c2d9c7362e8dae4041e
   - jfrog/.github/actions/install-go-with-cache@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/install-local-artifactory@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/cla@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - gacts/run-and-post-run@v1 → @598d7a875d5620e0457490555b5e18e46082aa47
   - jsdaniell/create-json@v1.2.3 → @b8e77fa01397ca39cc4a6198cc29a3be5481afef
   - jfrog/frogbot@v2 → @8e421a30aaa71d3aecf347cacd7f64d4e3f6ebbc
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c
   - actions-ecosystem/action-remove-labels@v1 → @2ce5d41b4b6aa8503e285553f75ed56e0a40bae0
   - wei/curl@master → @012398a392d02480afa2720780031f8621d5f94c

2. **missing-permissions**: Added top-level permissions blocks to auto-build-publish.yml (contents: read), release.yml (contents: write), remove-label.yml (pull-requests: write, issues: write), and test.yml (contents: read).

3. **script-injection**: Moved all ${{ }} expressions out of run: shell strings into env: blocks in release.yml and oidc-integration-test.yml.

4. **github-env-injection**: In release.yml, sanitized the tag name with `printf '%s' "$TAG_NAME" | tr -d '\n\r'` before regex extraction, preventing newline injection into GITHUB_ENV.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced both hardcoded literal 'password' values in .github/workflows/auto-build-publish.yml with ${{ secrets.ARTIFACTORY_LOCAL_PASSWORD }}. Fixed locations: (1) the JF_PASSWORD env var in the 'Setup JFrog CLI' step (was 'JF_PASSWORD: password'), and (2) the --password flag in the post-step shell script passed to gacts/run-and-post-run (was '--password password'). Both now reference the ARTIFACTORY_LOCAL_PASSWORD secret instead of a hardcoded literal value.

