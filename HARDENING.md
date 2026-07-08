<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli/v4.8.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: blocks. (a) release.yml line 16: `TAG_NAME="${{ github.event.release.tag_name }}"` — attacker-controlled release tag name is interpolated directly into a shell command. (a) oidc-integration-test.yml: Multiple ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} expressions are interpolated directly inside run: shell commands.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/oidc-integration-test.yml:55`
- `.github/workflows/oidc-integration-test.yml:66`
- `.github/workflows/oidc-integration-test.yml:155`
- `.github/workflows/oidc-integration-test.yml:158`
- `.github/workflows/oidc-integration-test.yml:175`
- `.github/workflows/oidc-integration-test.yml:178`

### github-env-injection (severity: high)

In release.yml, the run: block interpolates ${{ github.event.release.tag_name }} into TAG_NAME, then writes MAJOR and MINOR (derived from TAG_NAME) to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Lines `echo "MAJOR=$MAJOR" >> $GITHUB_ENV` and `echo "MINOR=$MINOR" >> $GITHUB_ENV` write values derived from an untrusted github context expression.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### hardcoded-credentials (severity: high)

Hardcoded literal password found in auto-build-publish.yml: `JF_PASSWORD: password` is set as a literal string value (not a secrets expression) in the env block of the Setup JFrog CLI step. While this targets a local test Artifactory instance, it is still a hardcoded credential matching the pattern.

Locations:

- `.github/workflows/auto-build-publish.yml:57`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tag or branch names instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references include: auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v4; release.yml: actions/checkout@v4, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:30`
- `.github/workflows/auto-build-publish.yml:33`
- `.github/workflows/auto-build-publish.yml:37`
- `.github/workflows/auto-build-publish.yml:42`
- `.github/workflows/auto-build-publish.yml:64`
- `.github/workflows/cla.yml:12`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:18`
- `.github/workflows/frogbot-scan-repository.yml:20`
- `.github/workflows/oidc-integration-test.yml:130`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:35`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:38`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, and test.yml. Only frogbot-scan-pull-request.yml, frogbot-scan-repository.yml, and oidc-integration-test.yml have permissions blocks.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, hardcoded-credentials, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 8 workflow files:

1. script-injection: In release.yml, moved ${{ github.event.release.tag_name }} to env block as TAG_NAME. In oidc-integration-test.yml, moved all ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} expressions to env blocks.

2. github-env-injection: In release.yml, added printf '%s' ... | tr -d '\n\r' sanitization before writing MAJOR and MINOR to $GITHUB_ENV.

3. hardcoded-credentials: In auto-build-publish.yml, replaced literal JF_PASSWORD: password with JF_PASSWORD: ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}.

4. unpinned-uses: Pinned all actions to full commit SHAs: actions/checkout@v4 -> 34e114876b0b11c390a56381ad16ebd13914f8d5, actions/setup-node@v4 -> 49933ea5288caeca8642d1e84afbd3f7d6820020, jfrog/.github/actions/install-go-with-cache@main -> 8342bf108acbcb73a0f951cdea804385bf1d16fc, jfrog/.github/actions/install-local-artifactory@main -> 8342bf108acbcb73a0f951cdea804385bf1d16fc, gacts/run-and-post-run@v1 -> 598d7a875d5620e0457490555b5e18e46082aa47, jsdaniell/create-json@v1.2.3 -> b8e77fa01397ca39cc4a6198cc29a3be5481afef, jfrog/.github/actions/cla@main -> 8342bf108acbcb73a0f951cdea804385bf1d16fc, jfrog/frogbot@v2 -> a2e5edb22925b75bda6847bb46b75e4015cc38db, ad-m/github-push-action@master -> 881a6320fdb16eb5318c5054f31c218aec2b324c, actions-ecosystem/action-remove-labels@v1 -> 2ce5d41b4b6aa8503e285553f75ed56e0a40bae0, wei/curl@master -> 012398a392d02480afa2720780031f8621d5f94c.

5. missing-permissions: Added permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (pull-requests: write, issues: write), and test.yml (contents: read).

### Iteration 1

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal `password` in the `jf c add --password password` CLI command with an environment variable reference `"$LOCAL_ARTIFACTORY_PASSWORD"`. Added an `env:` block to the `Post Step to Test the Auto Build-Publish post step` step that maps `LOCAL_ARTIFACTORY_PASSWORD: ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}`, consistent with how the same secret is used in the `Setup JFrog CLI` step.

