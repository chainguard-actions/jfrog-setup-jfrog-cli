<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jfrog--setup-jfrog-cli/v4.8.1** was hardened automatically. 5 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell blocks. (a) release.yml line 16: `TAG_NAME="${{ github.event.release.tag_name }}"` — the release tag name is injected directly into the shell. (b) oidc-integration-test.yml lines ~47–77: `${{ matrix.audience_id }}`, `${{ matrix.audience_value }}`, `${{ github.run_id }}`, `${{ github.repository_owner }}` are interpolated directly into curl run: commands. (c) oidc-integration-test.yml lines ~165–167: `${{ steps.setup-jfrog-cli.outputs.oidc-user }}` and `${{ steps.setup-jfrog-cli.outputs.oidc-token }}` are interpolated directly into run: shell commands. (d) oidc-integration-test.yml cleanup step: `${{ matrix.audience_id }}` and `${{ github.run_id }}` in a curl run: block. All of these allow an attacker to inject arbitrary shell commands via the interpolated values.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/oidc-integration-test.yml:47`
- `.github/workflows/oidc-integration-test.yml:65`
- `.github/workflows/oidc-integration-test.yml:165`
- `.github/workflows/oidc-integration-test.yml:167`
- `.github/workflows/oidc-integration-test.yml:220`

### github-env-injection (severity: high)

In release.yml, the value `${{ github.event.release.tag_name }}` is assigned to the shell variable `TAG_NAME` (line 16) and then `MAJOR` and `MINOR` — derived from `$TAG_NAME` via regex match — are written to `$GITHUB_ENV` (lines 21–22) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A crafted tag name containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### hardcoded-credentials (severity: high)

In auto-build-publish.yml, a literal hardcoded password is assigned: `JF_PASSWORD: password`. Even though this targets a local Artifactory instance, hardcoding credentials in workflow files is a security anti-pattern and violates the hardcoded-credentials check.

Locations:

- `.github/workflows/auto-build-publish.yml:57`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag or branch is updated maliciously. Failing references include: auto-build-publish.yml: `actions/checkout@v4`, `jfrog/.github/actions/install-go-with-cache@main`, `jfrog/.github/actions/install-local-artifactory@main`, `gacts/run-and-post-run@v1`, `jsdaniell/create-json@v1.2.3`; cla.yml: `jfrog/.github/actions/cla@main`; frogbot-scan-pull-request.yml: `jfrog/frogbot@v2`; frogbot-scan-repository.yml: `jfrog/.github/actions/install-go-with-cache@main`, `jfrog/frogbot@v2`; oidc-integration-test.yml: `actions/checkout@v4`; release.yml: `actions/checkout@v4`, `ad-m/github-push-action@master`; remove-label.yml: `actions-ecosystem/action-remove-labels@v1`; test.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `wei/curl@master`.

Locations:

- `.github/workflows/auto-build-publish.yml:27`
- `.github/workflows/auto-build-publish.yml:31`
- `.github/workflows/auto-build-publish.yml:35`
- `.github/workflows/auto-build-publish.yml:43`
- `.github/workflows/auto-build-publish.yml:62`
- `.github/workflows/cla.yml:13`
- `.github/workflows/frogbot-scan-pull-request.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:14`
- `.github/workflows/frogbot-scan-repository.yml:16`
- `.github/workflows/oidc-integration-test.yml:131`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:33`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:40`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` block and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad. Affected files: auto-build-publish.yml, cla.yml, release.yml, remove-label.yml, test.yml.

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

Fixed all 5 findings across 7 workflow files:

1. script-injection: In release.yml, moved `${{ github.event.release.tag_name }}` to an env block (TAG_NAME). In oidc-integration-test.yml, moved all ${{ }} expressions (matrix.audience_id, matrix.audience_value, github.run_id, github.repository_owner, steps outputs) to env blocks and referenced them as plain shell variables.

2. github-env-injection: In release.yml, TAG_NAME is now in an env block; MAJOR and MINOR values derived from it are sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. hardcoded-credentials: In auto-build-publish.yml, replaced `JF_PASSWORD: password` with `JF_PASSWORD: ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}`.

4. unpinned-uses: Pinned all 11 action references to full 40-character commit SHAs with tag comments: actions/checkout@v4, actions/setup-node@v4, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3, jfrog/.github/actions/cla@main, jfrog/frogbot@v2 (×2 files), ad-m/github-push-action@master, actions-ecosystem/action-remove-labels@v1, wei/curl@master.

5. missing-permissions: Added top-level permissions blocks to auto-build-publish.yml (contents: read), cla.yml (contents: read, pull-requests: write, issues: write), release.yml (contents: write), remove-label.yml (contents: read, pull-requests: write, issues: write), and test.yml (contents: read).

### Iteration 1

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal `--password password` in the 'Post Step to Test the Auto Build-Publish post step' with `--password "$LOCAL_ARTIFACTORY_PASSWORD"`. Added an `env:` block to the step that maps `LOCAL_ARTIFACTORY_PASSWORD: ${{ secrets.LOCAL_ARTIFACTORY_PASSWORD }}`, consistent with how the same local Artifactory instance is authenticated in the adjacent 'Setup JFrog CLI' step.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded base64-encoded JFrog CLI configuration blob in .github/workflows/test.yml (line 19) with a GitHub Actions secret reference `${{ secrets.JF_ENV_LOCAL }}`. The original value decoded to a JSON object containing a plaintext admin password, which was committed to source control. The fix requires the repository to have a `JF_ENV_LOCAL` secret configured with the appropriate base64-encoded configuration value.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in the 'Update Major and Minor Tags' step of .github/workflows/release.yml. Changed `git tag -f v$MAJOR` to `git tag -f "v$MAJOR"` and `git tag -f v$MAJOR.$MINOR` to `git tag -f "v$MAJOR.$MINOR"` to properly double-quote the variables sourced transitively from github.event.release.tag_name.

