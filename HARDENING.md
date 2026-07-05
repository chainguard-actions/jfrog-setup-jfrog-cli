<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli--/v4.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli--/v4.9.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA digests. Failing references: auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v4; release.yml: actions/checkout@v4, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:30`
- `.github/workflows/cla.yml:12`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:19`
- `.github/workflows/oidc-integration-test.yml:100`
- `.github/workflows/release.yml:10`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:21`

### permissions (severity: medium)

missing-permissions: These workflow files have no top-level permissions: key and no per-job permissions: key, granting the default (potentially write) token permissions to all jobs.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings. In release.yml, ${{ github.event.release.tag_name }} is embedded directly in a shell variable assignment (TAG_NAME="${{ github.event.release.tag_name }}"). In oidc-integration-test.yml, ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} are all interpolated directly into run: shell commands. These bypass shell quoting and allow injection of arbitrary shell metacharacters.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/oidc-integration-test.yml:48`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/oidc-integration-test.yml:131`
- `.github/workflows/oidc-integration-test.yml:134`

### github-env-injection (severity: high)

In release.yml, the run block interpolates ${{ github.event.release.tag_name }} directly into the shell variable TAG_NAME, then writes MAJOR and MINOR (derived from TAG_NAME via regex match) to $GITHUB_ENV without sanitization (no printf '%s' ... | tr -d '\n\r' step). A tag name containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release.yml:16`

### hardcoded-credentials (severity: high)

Hardcoded credentials found: (1) auto-build-publish.yml sets JF_PASSWORD: password as a literal plaintext password in the env block for the Setup JFrog CLI step. (2) test.yml sets JF_ENV_LOCAL to a base64-encoded JSON blob that decodes to a JFrog CLI config containing a hardcoded password (APB7DEZRPiHqHDTQokLkx9hhzKT).

Locations:

- `.github/workflows/auto-build-publish.yml:56`
- `.github/workflows/test.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all findings across 8 workflow files:

1. unpinned-uses: Pinned all mutable action references to full 40-char SHA digests:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - jfrog/.github/actions/install-go-with-cache@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/install-local-artifactory@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/cla@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - gacts/run-and-post-run@v1 → @598d7a875d5620e0457490555b5e18e46082aa47
   - jsdaniell/create-json@v1.2.3 → @b8e77fa01397ca39cc4a6198cc29a3be5481afef
   - jfrog/frogbot@v2 → @8e421a30aaa71d3aecf347cacd7f64d4e3f6ebbc
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c
   - actions-ecosystem/action-remove-labels@v1 → @2ce5d41b4b6aa8503e285553f75ed56e0a40bae0
   - wei/curl@master → @012398a392d02480afa2720780031f8621d5f94c

2. permissions: Added top-level permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (pull-requests: write, contents: read), and test.yml (contents: read).

3. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in oidc-integration-test.yml (matrix.audience_id, matrix.audience_value, github.run_id, github.repository_owner, steps.setup-jfrog-cli.outputs.oidc-user, steps.setup-jfrog-cli.outputs.oidc-token) and release.yml (github.event.release.tag_name).

4. github-env-injection: In release.yml, moved tag_name into env block and sanitized MAJOR/MINOR values with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_ENV.

5. hardcoded-credentials: In auto-build-publish.yml, replaced literal JF_PASSWORD: password with JF_PASSWORD: ${{ secrets.JF_PASSWORD }}. In test.yml, replaced hardcoded base64 JF_ENV_LOCAL value with JF_ENV_LOCAL: ${{ secrets.JF_ENV_LOCAL }}.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal password 'password' in the post: block of the 'Post Step to Test the Auto Build-Publish post step' step with an environment variable reference. Added an `env:` block to the step with `JF_ADMIN_PASSWORD: ${{ secrets.JF_PASSWORD }}`, and updated the jf c add command to use `--password "$JF_ADMIN_PASSWORD"` instead of the hardcoded literal. This is consistent with how the same secret is used in the 'Setup JFrog CLI' step.

