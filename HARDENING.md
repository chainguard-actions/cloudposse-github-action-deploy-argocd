<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags or version strings instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if any of those upstream actions are compromised or their tags are moved. Failing references:
- `dcarbone/install-yq-action@v1.1.0`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v4`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times)
- `actions/checkout@v6`
- `1arp/create-a-file-action@0.2`
- `nick-fields/retry@v4`
- `cloudposse/github-action-wait-commit-status@v0.2.1`

Locations:

- `action.yml:131`
- `action.yml:136`
- `action.yml:142`
- `action.yml:155`
- `action.yml:163`
- `action.yml:200`
- `action.yml:235`
- `action.yml:278`
- `action.yml:282`
- `action.yml:315`

### script-injection (severity: high)

Numerous `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions — including attacker-controllable `inputs.*`, `github.*`, and step/runner context values — inside shell (bash) commands. YAML template substitution happens before the shell ever sees the string, so an attacker can inject arbitrary shell metacharacters. Sub-rule (a) violations:

1. **Install git-url-parse** (line ~131): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — `${{ runner.temp }}` interpolated directly in shell.

2. **Parse git URL for destination** (line ~148): `run("${{ inputs.cluster }}")` — `inputs.cluster` interpolated directly into a JavaScript string in a `github-script` run block.

3. **Read platform context** (line ~178): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — both inputs interpolated directly in shell.

4. **Read platform metadata** (line ~192): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — input interpolated directly in shell.

5. **Resolve kube version** (line ~202): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — all interpolated directly in shell.

6. **Ensure argocd repo structure** (line ~218): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

7. **Helmfile render** (line ~224): `${{ inputs.namespace }}`, `${{ inputs.environment }}`, `${{ inputs.path }}`, `${{ steps.arguments.outputs.kube_version }}`, `${{ inputs.helmfile-args }}`, `${{ steps.config.outputs.tmp }}` all interpolated directly in shell.

8. **Build Helm Dependencies** (line ~235): `helm dependency build ${{ inputs.path }}` — input interpolated directly.

9. **Helm raw render** (line ~241): `${{ inputs.values_file }}`, `${{ inputs.application }}`, `${{ inputs.path }}`, `${{ inputs.image }}`, `${{ inputs.image-tag }}`, `${{ inputs.environment }}`, `${{ inputs.namespace }}`, `${{ inputs.helm-args }}`, `${{ steps.arguments.outputs.kube_version }}`, `${{ steps.config.outputs.tmp }}` all interpolated directly in shell.

10. **Get Webapp** (line ~261): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` interpolated directly in shell.

11. **Push to Github** (line ~282): `${{ steps.destination_dir.outputs.name }}`, `${{ steps.destination.outputs.ref }}`, `${{ inputs.operation }}` (inside a `case` statement), `${{ steps.config.outputs.path }}`, `${{ github.repository }}`, `${{ github.sha }}`, `${{ github.run_id }}`, `${{ github.run_attempt }}` all interpolated directly in shell.

12. **Select GitHub Token for Sync Mode** (line ~309): `${{ inputs.commit-status-github-token }}` and `${{ inputs.github-pat }}` interpolated directly in shell conditionals.

Locations:

- `action.yml:131`
- `action.yml:148`
- `action.yml:178`
- `action.yml:192`
- `action.yml:202`
- `action.yml:218`
- `action.yml:224`
- `action.yml:235`
- `action.yml:241`
- `action.yml:261`
- `action.yml:282`
- `action.yml:309`

### github-env-injection (severity: high)

Multiple `run:` blocks write untrusted or externally-influenced values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`), enabling newline injection that could poison subsequent steps' environment or outputs.

1. **YQ Platform settings / metadata step** (line ~196): Iterates over raw yq output from SSM-sourced `_metadata.yaml` and writes each `key=value` pair directly to `$GITHUB_OUTPUT` with `echo "${output}" >> $GITHUB_OUTPUT`. The values come from SSM (external/untrusted) and are not sanitized.

2. **Resolve kube version** (line ~202): `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is derived from `${{ inputs.kube-version }}` (caller-controlled) or `${{ steps.metadata.outputs.kube_version }}` (SSM-sourced). No sanitization applied.

3. **Get Webapp** (line ~261): `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` where `WEBAPP_URL` is extracted from a rendered Kubernetes manifest via yq. The manifest content is influenced by caller-supplied inputs. No sanitization applied.

4. **Select GitHub Token for Sync Mode** (line ~309): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — caller-supplied token values written directly to `$GITHUB_OUTPUT` without sanitization. A newline embedded in the token value could inject additional output variables.

Locations:

- `action.yml:196`
- `action.yml:207`
- `action.yml:268`
- `action.yml:311`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ssm-path }}" appears directly in run: block of step "Read platform context"; move to env: map

Locations:

- `action.yml:238`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.environment }}" appears directly in run: block of step "Read platform context"; move to env: map

Locations:

- `action.yml:238`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ssm-path }}" appears directly in run: block of step "Read platform metadata"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.kube-version }}" appears directly in run: block of step "Resolve kube version"; move to env: map

Locations:

- `action.yml:273`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ssm-path }}" appears directly in run: block of step "Resolve kube version"; move to env: map

Locations:

- `action.yml:274`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.namespace }}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:302`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.environment }}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:303`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path}}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:304`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.helmfile-args }}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:308`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Build Helm Dependencies"; move to env: map

Locations:

- `action.yml:317`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.values_file }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:323`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.application }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:326`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:326`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:327`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:328`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image-tag }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:329`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image-tag }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:330`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.environment }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:331`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.namespace }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:333`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.helm-args }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:337`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.commit-status-github-token }}" appears directly in run: block of step "Select GitHub Token for Sync Mode"; move to env: map

Locations:

- `action.yml:434`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.github-pat }}" appears directly in run: block of step "Select GitHub Token for Sync Mode"; move to env: map

Locations:

- `action.yml:435`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.commit-status-github-token }}" appears directly in run: block of step "Select GitHub Token for Sync Mode"; move to env: map

Locations:

- `action.yml:437`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 8 action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ runner.* }}, ${{ steps.*.outputs.* }} expressions from run: blocks into env: blocks. The github-script step now reads inputs.cluster via process.env.INPUT_CLUSTER. The Push to Github step uses built-in GITHUB_REPOSITORY, GITHUB_SHA, GITHUB_RUN_ID, GITHUB_RUN_ATTEMPT env vars (which are always available) and moves step outputs to env vars.

3. **github-env-injection**: All values written to $GITHUB_OUTPUT are now sanitized with `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing — covering the metadata loop, kube version, webapp URL, and GitHub token outputs.

4. **helmfile-args and helm-args**: These list-style inputs are tokenized with the xargs/while-read-NUL idiom to preserve argument boundaries safely.

5. **values_file**: Preserved upstream's IFS=', ' read -r -a array splitting (comma+space separated list) since that matches upstream's intent.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two locations in hardened/action/action.yml where backslashes had been incorrectly added before double quotes in yq query strings. At line 297, restored `query: ."${{ steps.resolved_kube_version.outputs.value == '' }}"` to `query: ."${{ steps.resolved_kube_version.outputs.value == '' }}"` (removing backslashes). At line 418, similarly restored `query: ."${{ inputs.gitref-sha == '' }}"` to `query: ."${{ inputs.gitref-sha == '' }}"` (removing backslashes). The backslashes were causing yq to fail with 'Can\'t handle character' errors.

### Iteration 2

**Fixes applied:** publish-gate

**Notes:**

Fixed YAML parsing error at line 418 in action.yml. The 'query:' and 'config:' fields in the 'Deplicated Ref' step's 'with:' block were incorrectly placed on the same line ('query: ."${{ inputs.gitref-sha == '' }}"       config: |-'). Split them onto separate lines with proper indentation so the YAML parses correctly.

