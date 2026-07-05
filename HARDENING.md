<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli--/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli--/v5.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA digests, making them vulnerable to supply-chain attacks. Failing references: auto-build-publish.yml: actions/checkout@v6, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml: jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml: jfrog/frogbot@v2; frogbot-scan-repository.yml: jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml: actions/checkout@v6; release.yml: actions/checkout@v6, ad-m/github-push-action@master; remove-label.yml: actions-ecosystem/action-remove-labels@v1; test.yml: actions/checkout@v6, actions/setup-node@v6, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:29`
- `.github/workflows/auto-build-publish.yml:32`
- `.github/workflows/auto-build-publish.yml:35`
- `.github/workflows/auto-build-publish.yml:39`
- `.github/workflows/auto-build-publish.yml:63`
- `.github/workflows/cla.yml:18`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:19`
- `.github/workflows/frogbot-scan-repository.yml:21`
- `.github/workflows/oidc-integration-test.yml:116`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:40`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:37`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings. Sub-rule (a) violations: (1) release.yml: `TAG_NAME="${{ github.event.release.tag_name }}"` — github.event.release.tag_name is interpolated directly into the shell script. (2) oidc-integration-test.yml: multiple run: blocks interpolate ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} directly into shell commands, allowing shell metacharacter injection.

Locations:

- `.github/workflows/release.yml:18`
- `.github/workflows/oidc-integration-test.yml:48`
- `.github/workflows/oidc-integration-test.yml:57`
- `.github/workflows/oidc-integration-test.yml:130`
- `.github/workflows/oidc-integration-test.yml:133`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: block and no job-level permissions: blocks, meaning they run with the default (potentially broad) token permissions: auto-build-publish.yml, remove-label.yml, test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### hardcoded-credentials (severity: high)

A literal plaintext password is hardcoded in auto-build-publish.yml: `JF_PASSWORD: password`. While this appears to be a local test Artifactory instance credential, hardcoding passwords in workflow files is a security anti-pattern. Additionally, the `post:` inline script in the gacts/run-and-post-run step also contains `--password password` as a literal CLI argument.

Locations:

- `.github/workflows/auto-build-publish.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full commit SHAs:
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

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks:
   - release.yml: TAG_NAME=${{ github.event.release.tag_name }} moved to env block
   - oidc-integration-test.yml: matrix.audience_id, matrix.audience_value, github.run_id, steps.setup-jfrog-cli.outputs.oidc-user, steps.setup-jfrog-cli.outputs.oidc-token all moved to env blocks

3. missing-permissions: Added top-level permissions blocks:
   - auto-build-publish.yml: permissions: contents: read
   - remove-label.yml: permissions: pull-requests: write
   - test.yml: permissions: contents: read

4. hardcoded-credentials: Replaced hardcoded 'password' literal in auto-build-publish.yml with ${{ secrets.JF_LOCAL_PASSWORD }} in both the gacts/run-and-post-run post script and the Setup JFrog CLI step env block.

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two findings in .github/workflows/release.yml:
1. github-env-injection (lines 25-26): Added sanitization of MAJOR and MINOR variables using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV. Also quoted $GITHUB_ENV path.
2. script-injection (lines 36, 38): Double-quoted $MAJOR and $MINOR in the `git tag -f` commands to prevent shell metacharacter injection from these workflow-controlled values.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded base64-encoded JFrog credential in .github/workflows/test.yml (line 18) with a GitHub Actions secret reference: ${{ secrets.JF_ENV_LOCAL }}. The original value decoded to a JSON object containing a plaintext password 'APB7DEZRPiHqHDTQokLkx9hhzKT' for a local Artifactory server. The credential must now be stored as a repository secret named JF_ENV_LOCAL containing the base64-encoded JSON connection configuration.

