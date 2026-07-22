<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jfrog--setup-jfrog-cli/v4.9.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or branch names instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag or branch is overwritten with malicious code. Affected references include: auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v4; release.yml: actions/checkout@v4, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:30`
- `.github/workflows/auto-build-publish.yml:34`
- `.github/workflows/auto-build-publish.yml:38`
- `.github/workflows/auto-build-publish.yml:43`
- `.github/workflows/auto-build-publish.yml:63`
- `.github/workflows/cla.yml:13`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:21`
- `.github/workflows/frogbot-scan-repository.yml:24`
- `.github/workflows/oidc-integration-test.yml:100`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:37`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:42`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions (${{ }}) are interpolated directly inside run: shell command strings, allowing an attacker to inject arbitrary shell commands. In release.yml, `TAG_NAME="${{ github.event.release.tag_name }}"` injects a release tag directly into a bash variable assignment in a run: block. In oidc-integration-test.yml, multiple run: blocks embed ${{ secrets.JFROG_PLATFORM_URL }}, ${{ secrets.JFROG_PLATFORM_RT_TOKEN }}, ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} directly into shell commands. Any of these values flowing through YAML template substitution before the shell sees them can carry shell metacharacters.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/oidc-integration-test.yml:47`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/oidc-integration-test.yml:120`
- `.github/workflows/oidc-integration-test.yml:122`

### github-env-injection (severity: high)

In release.yml, the run: block sets TAG_NAME from the untrusted expression ${{ github.event.release.tag_name }} and then writes MAJOR and MINOR (values extracted from TAG_NAME via regex) to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A crafted release tag containing newline characters could inject arbitrary environment variable definitions into subsequent steps. The offending lines are: `echo "MAJOR=$MAJOR" >> $GITHUB_ENV` and `echo "MINOR=$MINOR" >> $GITHUB_ENV`.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### hardcoded-credentials (severity: high)

Literal credential values are hardcoded in workflow files. (1) auto-build-publish.yml sets `JF_PASSWORD: password` as a plain-text environment variable for the Setup JFrog CLI step, and also passes `--password password` as a CLI argument in the post-step script — both are hardcoded Artifactory passwords. (2) test.yml sets `JF_ENV_LOCAL` to a base64-encoded JSON blob that decodes to a JFrog CLI server configuration containing a hardcoded password (`"password":"APB7DEZRPiHqHDTQokLkx9hhzKT"`). These should be stored in GitHub Actions secrets and referenced via ${{ secrets.* }}.

Locations:

- `.github/workflows/auto-build-publish.yml:44`
- `.github/workflows/auto-build-publish.yml:60`
- `.github/workflows/test.yml:14`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` block and no per-job `permissions:` blocks. Without explicit permissions, workflows run with the repository's default token permissions (which may be read/write for contents), violating the principle of least privilege. Affected files: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials, missing-permissions

**Notes:**

Fixed all 5 findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full SHA commits with tag comments:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - jfrog/.github/actions/install-go-with-cache@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/install-local-artifactory@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/cla@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - gacts/run-and-post-run@v1 → @598d7a875d5620e0457490555b5e18e46082aa47
   - jsdaniell/create-json@v1.2.3 → @b8e77fa01397ca39cc4a6198cc29a3be5481afef
   - jfrog/frogbot@v2 → @17efdfdbb49cb4ea05a9ac58bb29483763a0f5b9
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c
   - actions-ecosystem/action-remove-labels@v1 → @2ce5d41b4b6aa8503e285553f75ed56e0a40bae0
   - wei/curl@master → @012398a392d02480afa2720780031f8621d5f94c

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in release.yml and oidc-integration-test.yml.

3. github-env-injection: In release.yml, sanitized MAJOR and MINOR values with printf '%s' | tr -d '\n\r' before writing to $GITHUB_ENV.

4. hardcoded-credentials: Replaced hardcoded 'password' in auto-build-publish.yml with ${{ secrets.ARTIFACTORY_PASSWORD }}, and replaced hardcoded base64-encoded JSON blob in test.yml with ${{ secrets.JF_ENV_LOCAL }}.

5. missing-permissions: Added minimal permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (pull-requests: write, issues: write), and test.yml (contents: read).

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in .github/workflows/release.yml. In the 'Update Major and Minor Tags' step, changed `git tag -f v$MAJOR` to `git tag -f "v$MAJOR"` and `git tag -f v$MAJOR.$MINOR` to `git tag -f "v$MAJOR.$MINOR"`. Both $MAJOR and $MINOR are sourced from github.event.release.tag_name (workflow-controllable) via GITHUB_ENV, so they must be double-quoted even though they are constrained to digits by the regex validation in the previous step.

