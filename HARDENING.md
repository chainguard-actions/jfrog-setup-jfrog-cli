<!-- markdownlint-disable -->

# Hardening Report: jfrog--setup-jfrog-cli/v4.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **jfrog--setup-jfrog-cli/v4.10.1** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags, branch names, or version strings instead of pinned 40-character SHA digests. Affected references: auto-build-publish.yml uses actions/checkout@v6, jfrog/.github/actions/install-go-with-cache@main, jfrog/.github/actions/install-local-artifactory@main, gacts/run-and-post-run@v1, jsdaniell/create-json@v1.2.3; cla.yml uses jfrog/.github/actions/cla@main; frogbot-scan-pull-request.yml uses jfrog/frogbot@v2; frogbot-scan-repository.yml uses jfrog/.github/actions/install-go-with-cache@main, jfrog/frogbot@v2; oidc-integration-test.yml uses actions/checkout@v6; release.yml uses actions/checkout@v6, ad-m/github-push-action@master; remove-label.yml uses actions-ecosystem/action-remove-labels@v1; test.yml uses actions/checkout@v6, actions/setup-node@v6, wei/curl@master.

Locations:

- `.github/workflows/auto-build-publish.yml:28`
- `.github/workflows/cla.yml:17`
- `.github/workflows/frogbot-scan-pull-request.yml:13`
- `.github/workflows/frogbot-scan-repository.yml:19`
- `.github/workflows/oidc-integration-test.yml:130`
- `.github/workflows/release.yml:9`
- `.github/workflows/remove-label.yml:14`
- `.github/workflows/test.yml:22`

### missing-permissions (severity: medium)

These workflow files have no top-level permissions: key and no job-level permissions: keys, meaning they run with the default (potentially broad) token permissions: auto-build-publish.yml, release.yml, remove-label.yml, and test.yml.

Locations:

- `.github/workflows/auto-build-publish.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/remove-label.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings (sub-rule a). In release.yml, the run block assigns TAG_NAME="${{ github.event.release.tag_name }}" — a github context value is interpolated directly into the shell script before the shell parses it, enabling injection if the tag name contains shell metacharacters. In oidc-integration-test.yml, multiple run: blocks interpolate ${{ matrix.audience_id }}, ${{ matrix.audience_value }}, ${{ github.run_id }}, ${{ github.repository_owner }}, ${{ steps.setup-jfrog-cli.outputs.oidc-user }}, and ${{ steps.setup-jfrog-cli.outputs.oidc-token }} directly into shell commands.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/oidc-integration-test.yml:52`
- `.github/workflows/oidc-integration-test.yml:63`
- `.github/workflows/oidc-integration-test.yml:138`
- `.github/workflows/oidc-integration-test.yml:141`

### github-env-injection (severity: high)

In release.yml, the run: block extracts MAJOR and MINOR from TAG_NAME (which is set to ${{ github.event.release.tag_name }}) and writes them to $GITHUB_ENV without sanitization: `echo "MAJOR=$MAJOR" >> $GITHUB_ENV` and `echo "MINOR=$MINOR" >> $GITHUB_ENV`. Although a regex check is applied, the values are not passed through `printf '%s' ... | tr -d '\n\r'` before the write, leaving the GITHUB_ENV write unsanitized against newline injection.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### hardcoded-credentials (severity: high)

In auto-build-publish.yml, the env block for the Setup JFrog CLI step contains a literal hardcoded password: `JF_PASSWORD: password`. This is a non-expression literal value assigned to a key named JF_PASSWORD (matching the 'password' pattern), not a GitHub Actions secret expression.

Locations:

- `.github/workflows/auto-build-publish.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all findings across 8 workflow files: (1) unpinned-uses: pinned all action references to full SHA digests with tag comments in auto-build-publish.yml, cla.yml, frogbot-scan-pull-request.yml, frogbot-scan-repository.yml, oidc-integration-test.yml, release.yml, remove-label.yml, and test.yml; (2) missing-permissions: added top-level permissions blocks to auto-build-publish.yml (contents: read), release.yml (contents: write), remove-label.yml (pull-requests: write), and test.yml (contents: read); (3) script-injection: moved all ${{ }} expressions in run: blocks to env: blocks in release.yml (TAG_NAME) and oidc-integration-test.yml (AUDIENCE_ID, AUDIENCE_VALUE, RUN_ID, REPOSITORY_OWNER, OIDC_USER, OIDC_TOKEN); (4) github-env-injection: sanitized MAJOR and MINOR values with printf | tr -d before writing to $GITHUB_ENV in release.yml; (5) hardcoded-credentials: replaced literal 'password' value for JF_PASSWORD with ${{ secrets.ARTIFACTORY_PASSWORD }} in auto-build-publish.yml.

### Iteration 1

**Fixes applied:** hardcoded-credentials

**Notes:**

Fixed two hardcoded credential findings: (1) In .github/workflows/auto-build-publish.yml, replaced the literal '--password password' in the jf c add command with '--password "$ARTIFACTORY_PASSWORD"' and added an env block mapping ARTIFACTORY_PASSWORD to ${{ secrets.ARTIFACTORY_PASSWORD }}. (2) In .github/workflows/test.yml, replaced the hardcoded base64-encoded JSON blob (containing a plaintext password) in JF_ENV_LOCAL with ${{ secrets.JF_ENV_LOCAL }} so the credential is stored as a GitHub secret.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in the 'Update Major and Minor Tags' step of .github/workflows/release.yml. Changed `git tag -f v$MAJOR` to `git tag -f "v$MAJOR"` and `git tag -f v$MAJOR.$MINOR` to `git tag -f "v$MAJOR.$MINOR"`. These variables are sourced from github.event.release.tag_name via GITHUB_ENV, so quoting them prevents potential script injection from unquoted expansion of workflow-controllable environment variables.

