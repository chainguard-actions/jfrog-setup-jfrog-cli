<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli/v4.9.1** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: blocks. In release.yml, `TAG_NAME="${{ github.event.release.tag_name }}"` interpolates a GitHub context value directly into a shell command. In oidc-integration-test.yml, `${{ matrix.audience_id }}`, `${{ matrix.audience_value }}`, `${{ github.run_id }}`, and `${{ github.repository_owner }}` are interpolated directly into curl run: commands, and `${{ steps.setup-jfrog-cli.outputs.oidc-user }}` and `${{ steps.setup-jfrog-cli.outputs.oidc-token }}` are interpolated into test run: commands.

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/oidc-integration-test.yml:47`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/oidc-integration-test.yml:155`
- `.github/workflows/oidc-integration-test.yml:156`

### github-env-injection (severity: high)

In release.yml, the run: block assigns `${{ github.event.release.tag_name }}` to TAG_NAME, then extracts MAJOR and MINOR from it and writes them to $GITHUB_ENV via `echo "MAJOR=$MAJOR" >> $GITHUB_ENV` and `echo "MINOR=$MINOR" >> $GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled tag name could inject arbitrary environment variables.

Locations:

- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:19`

### hardcoded-credentials (severity: high)

Hardcoded credentials found. In auto-build-publish.yml, `JF_PASSWORD: password` is set as a literal plaintext password in the env block of the Setup JFrog CLI step. In test.yml, `JF_ENV_LOCAL` is set to a base64-encoded JFrog configuration JSON that contains a hardcoded password (`APB7DEZRPiHqHDTQokLkx9hhzKT`) embedded in the encoded value.

Locations:

- `.github/workflows/auto-build-publish.yml:57`
- `.github/workflows/test.yml:13`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) repository permissions: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, and test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of immutable full-length SHA commit hashes. Unpinned references: auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v4; release.yml: actions/checkout@v4, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:28`
- `.github/workflows/auto-build-publish.yml:31`
- `.github/workflows/auto-build-publish.yml:34`
- `.github/workflows/auto-build-publish.yml:38`
- `.github/workflows/auto-build-publish.yml:60`
- `.github/workflows/cla.yml:13`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:18`
- `.github/workflows/frogbot-scan-repository.yml:20`
- `.github/workflows/oidc-integration-test.yml:130`
- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:34`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, hardcoded-credentials, missing-permissions, unpinned-uses

**Notes:**

Fixed all findings across 7 workflow files:

1. script-injection: Moved ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, ${{ steps.setup-jfrog-cli.outputs.oidc-token }} from run: blocks to env: blocks in oidc-integration-test.yml. Moved ${{ github.event.release.tag_name }} to env: block in release.yml.

2. github-env-injection: In release.yml, MAJOR and MINOR are now sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. hardcoded-credentials: In auto-build-publish.yml, replaced literal `JF_PASSWORD: password` with `JF_PASSWORD: ${{ secrets.JF_PASSWORD }}`. In test.yml, replaced hardcoded base64 JF_ENV_LOCAL value with `${{ secrets.JF_ENV_LOCAL }}`.

4. missing-permissions: Added `permissions:` blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (contents: read, pull-requests: write), and test.yml (contents: read).

5. unpinned-uses: Pinned all 16 action references to full commit SHAs: actions/checkout@v4->34e114876b0b11c390a56381ad16ebd13914f8d5, jfrog/.github/actions/install-go-with-cache@main->8342bf108acbcb73a0f951cdea804385bf1d16fc, jfrog/.github/actions/install-local-artifactory@main->8342bf108acbcb73a0f951cdea804385bf1d16fc, gacts/run-and-post-run@v1->598d7a875d5620e0457490555b5e18e46082aa47, jsdaniell/create-json@v1.2.3->b8e77fa01397ca39cc4a6198cc29a3be5481afef, jfrog/.github/actions/cla@main->8342bf108acbcb73a0f951cdea804385bf1d16fc, jfrog/frogbot@v2->a2e5edb22925b75bda6847bb46b75e4015cc38db, ad-m/github-push-action@master->881a6320fdb16eb5318c5054f31c218aec2b324c, actions-ecosystem/action-remove-labels@v1->2ce5d41b4b6aa8503e285553f75ed56e0a40bae0, actions/setup-node@v4->49933ea5288caeca8642d1e84afbd3f7d6820020, wei/curl@master->012398a392d02480afa2720780031f8621d5f94c.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. oidc-integration-test.yml: Moved `${{ secrets.JFROG_PLATFORM_URL }}` and `${{ secrets.JFROG_PLATFORM_RT_TOKEN }}` out of `run:` blocks and into `env:` blocks for all three affected steps ('Create OpenID Connect integration', 'Create OIDC Identity Mapping', 'Delete OIDC integration'). Shell scripts now reference `$JFROG_PLATFORM_URL` and `$JFROG_PLATFORM_RT_TOKEN` as plain environment variables.
2. release.yml: Added double-quotes around variable expansions in the 'Update Major and Minor Tags' step: `git tag -f "v$MAJOR"` and `git tag -f "v$MAJOR.$MINOR"`.

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

In .github/workflows/auto-build-publish.yml, replaced the hardcoded literal `--password password` with `--password "$LOCAL_ADMIN_PASSWORD"` and added an `env:` block to the 'Post Step to Test the Auto Build-Publish post step' step that sets `LOCAL_ADMIN_PASSWORD: ${{ secrets.LOCAL_ADMIN_PASSWORD }}`. This removes the hardcoded credential and replaces it with a proper GitHub secret reference.

