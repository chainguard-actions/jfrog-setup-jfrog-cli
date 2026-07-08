<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli/v4.8.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks. Failing references include: auto-build-publish.yml: actions/checkout@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v4; release.yml: actions/checkout@v4, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v4, actions/setup-node@v4, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:29`
- `.github/workflows/auto-build-publish.yml:33`
- `.github/workflows/auto-build-publish.yml:37`
- `.github/workflows/auto-build-publish.yml:43`
- `.github/workflows/auto-build-publish.yml:62`
- `.github/workflows/cla.yml:13`
- `.github/workflows/frogbot-scan-pull-request.yml:15`
- `.github/workflows/frogbot-scan-repository.yml:18`
- `.github/workflows/frogbot-scan-repository.yml:20`
- `.github/workflows/oidc-integration-test.yml:119`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:33`
- `.github/workflows/remove-label.yml:12`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:38`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: block and no job-level permissions: blocks, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/cla.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings. Sub-rule (a) violations: (1) release.yml line 16: `TAG_NAME="${{ github.event.release.tag_name }}"` — attacker-controlled release tag name is interpolated directly into a shell script. (2) oidc-integration-test.yml lines 47-57: `${{ matrix.audience_id }}`, `${{ github.run_id }}`, `${{ matrix.audience_value }}`, `${{ github.repository_owner }}` interpolated directly in curl run blocks. (3) oidc-integration-test.yml line 139: `run: test -n "${{ steps.setup-jfrog-cli.outputs.oidc-user }}"` — step output interpolated directly. (4) oidc-integration-test.yml line 143: `run: test -n "${{ steps.setup-jfrog-cli.outputs.oidc-token }}"` — step output interpolated directly. (5) oidc-integration-test.yml lines 149-151: cleanup run block with `${{ matrix.audience_id }}`, `${{ github.run_id }}` interpolated directly.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/oidc-integration-test.yml:47`
- `.github/workflows/oidc-integration-test.yml:62`
- `.github/workflows/oidc-integration-test.yml:139`
- `.github/workflows/oidc-integration-test.yml:143`
- `.github/workflows/oidc-integration-test.yml:149`

### github-env-injection (severity: high)

In release.yml, the run: block at line 16 interpolates `${{ github.event.release.tag_name }}` directly into the shell variable TAG_NAME, then writes derived values MAJOR and MINOR to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker who can create a release with a crafted tag name containing newlines could inject arbitrary environment variables.

Locations:

- `.github/workflows/release.yml:16`

### hardcoded-credentials (severity: high)

Hardcoded credentials found in workflow files: (1) test.yml line 14: JF_ENV_LOCAL is set to a base64-encoded JSON blob that decodes to include a hardcoded password ("password":"APB7DEZRPiHqHDTQokLkx9hhzKT") and admin credentials for a local Artifactory instance. (2) auto-build-publish.yml line 57: JF_PASSWORD is set to the literal value 'password' for a local Artifactory instance.

Locations:

- `.github/workflows/test.yml:14`
- `.github/workflows/auto-build-publish.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - jfrog/.github/actions/install-go-with-cache@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/.github/actions/install-local-artifactory@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - gacts/run-and-post-run@v1 → @598d7a875d5620e0457490555b5e18e46082aa47
   - jsdaniell/create-json@v1.2.3 → @b8e77fa01397ca39cc4a6198cc29a3be5481afef
   - jfrog/.github/actions/cla@main → @8342bf108acbcb73a0f951cdea804385bf1d16fc
   - jfrog/frogbot@v2 → @a2e5edb22925b75bda6847bb46b75e4015cc38db
   - ad-m/github-push-action@master → @881a6320fdb16eb5318c5054f31c218aec2b324c
   - actions-ecosystem/action-remove-labels@v1 → @2ce5d41b4b6aa8503e285553f75ed56e0a40bae0
   - wei/curl@master → @012398a392d02480afa2720780031f8621d5f94c

2. missing-permissions: Added top-level permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (pull-requests: write, issues: write), and test.yml (contents: read).

3. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in oidc-integration-test.yml (audience_id, run_id, audience_value, repository_owner, oidc-user output, oidc-token output) and release.yml (tag_name).

4. github-env-injection: In release.yml, moved tag_name into env block and sanitized with printf '%s' | tr -d '\n\r' before using in GITHUB_ENV writes. Used printf format to write MAJOR/MINOR to GITHUB_ENV safely.

5. hardcoded-credentials: In test.yml, replaced the hardcoded base64-encoded JF_ENV_LOCAL value with ${{ secrets.JF_ENV_LOCAL }}. In auto-build-publish.yml, replaced the hardcoded JF_PASSWORD: password with ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}.

### Iteration 1

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal password 'password' in the post: script of the 'Post Step to Test the Auto Build-Publish post step' step in .github/workflows/auto-build-publish.yml. Added an env: block to the step that maps LOCAL_ARTIFACTORY_PASSWORD to ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}, and updated the jf c add command to use --password "$LOCAL_ARTIFACTORY_PASSWORD" instead of --password password. This is consistent with how the same credential is used in the Setup JFrog CLI step.

