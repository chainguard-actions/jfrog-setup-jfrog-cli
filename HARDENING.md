<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jfrog--setup-jfrog-cli/v5.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ ... }} expression interpolation inside run: shell commands. In release.yml, the run: block sets TAG_NAME="${{ github.event.release.tag_name }}" — a GitHub-controlled value injected directly into the shell before quoting can protect it. In oidc-integration-test.yml, three separate run: blocks interpolate ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} directly into shell command strings. Any of these values could contain shell metacharacters that execute arbitrary commands.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/oidc-integration-test.yml:47`
- `.github/workflows/oidc-integration-test.yml:60`
- `.github/workflows/oidc-integration-test.yml:152`
- `.github/workflows/oidc-integration-test.yml:155`
- `.github/workflows/oidc-integration-test.yml:159`
- `.github/workflows/oidc-integration-test.yml:163`

### github-env-injection (severity: high)

In release.yml, the run: block writes MAJOR and MINOR values (derived from ${{ github.event.release.tag_name }}, an attacker-influenced value via a crafted release tag) to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A tag name containing a newline could inject arbitrary environment variables into subsequent steps. The offending lines are: `echo "MAJOR=$MAJOR" >> $GITHUB_ENV` and `echo "MINOR=$MINOR" >> $GITHUB_ENV`.

Locations:

- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:23`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tag or branch refs instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Unpinned references found:
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
- `.github/workflows/auto-build-publish.yml:32`
- `.github/workflows/auto-build-publish.yml:35`
- `.github/workflows/auto-build-publish.yml:39`
- `.github/workflows/auto-build-publish.yml:62`
- `.github/workflows/cla.yml:19`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:16`
- `.github/workflows/frogbot-scan-repository.yml:18`
- `.github/workflows/oidc-integration-test.yml:138`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:34`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:36`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions: block and no job-level permissions: blocks on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (which may be write-all), granting broader access than necessary.
- auto-build-publish.yml: no permissions block at top level or job level
- test.yml: no permissions block at top level or job level
- remove-label.yml: no permissions block at top level or job level

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/remove-label.yml:1`

### hardcoded-credentials (severity: high)

In auto-build-publish.yml, the env: block for the 'Setup JFrog CLI' step contains a literal hardcoded password: `JF_PASSWORD: password`. This is a non-expression literal value assigned to a password field. While this appears to be a local test Artifactory instance credential, hardcoding credentials in workflow files is a security risk as it exposes the value in the repository history and to anyone with read access.

Locations:

- `.github/workflows/auto-build-publish.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings across 7 workflow files:

1. **script-injection** (release.yml, oidc-integration-test.yml): Moved all ${{ }} expressions from run: shell strings into step env: blocks. In release.yml, TAG_NAME is now set via env. In oidc-integration-test.yml, AUDIENCE_ID, AUDIENCE_VALUE, RUN_ID, REPOSITORY_OWNER, OIDC_USER, and OIDC_TOKEN are all moved to env blocks and referenced as plain shell variables.

2. **github-env-injection** (release.yml): Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing MAJOR and MINOR to $GITHUB_ENV.

3. **unpinned-uses**: Pinned all action references to full 40-character SHA digests:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - jfrog/.github/actions/install-go-with-cache@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/install-local-artifactory@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/cla@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - gacts/run-and-post-run@v1 → @598d7a875d5620e0457490555b5e18e46082aa47
   - jsdaniell/create-json@v1.2.3 → @b8e77fa01397ca39cc4a6198cc29a3be5481afef
   - jfrog/frogbot@v2 → @a2e5edb22925b75bda6847bb46b75e4015cc38db
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c
   - actions-ecosystem/action-remove-labels@v1 → @2ce5d41b4b6aa8503e285553f75ed56e0a40bae0
   - wei/curl@master → @012398a392d02480afa2720780031f8621d5f94c

4. **missing-permissions**: Added `permissions: contents: read` to auto-build-publish.yml and test.yml; added `permissions: pull-requests: write` to remove-label.yml.

5. **hardcoded-credentials**: Replaced literal `JF_PASSWORD: password` with `JF_PASSWORD: ${{ secrets.JF_LOCAL_PASSWORD }}` in auto-build-publish.yml.

