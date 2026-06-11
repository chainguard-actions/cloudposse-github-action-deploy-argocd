<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell commands (rule a), enabling script injection. Affected steps include: 'Install git-url-parse' uses ${{ runner.temp }} in run:; 'Parse git URL for destination' interpolates ${{ inputs.cluster }} directly into the github-script string; 'Read platform context' uses ${{ inputs.ssm-path }}/${{ inputs.environment }} in a shell command; 'Read platform metadata' uses ${{ inputs.ssm-path }}; 'Resolve kube version' uses ${{ inputs.kube-version }}, ${{ inputs.ssm-path }}, ${{ steps.metadata.outputs.kube_version }}; 'Ensure argocd repo structure' uses ${{ steps.config.outputs.tmp }}; 'Helmfile render' uses ${{ inputs.namespace }}, ${{ inputs.environment }}, ${{ inputs.path }}, ${{ inputs.helmfile-args }}; 'Build Helm Dependencies' uses ${{ inputs.path }}; 'Helm raw render' uses ${{ inputs.values_file }}, ${{ inputs.application }}, ${{ inputs.path }}, ${{ inputs.image }}, ${{ inputs.image-tag }}, ${{ inputs.environment }}, ${{ inputs.namespace }}, ${{ inputs.helm-args }}; 'Get Webapp' uses ${{ steps.config.outputs.tmp }}; 'Push to Github' command uses ${{ inputs.operation }}, ${{ github.repository }}, ${{ github.sha }}, ${{ github.run_id }}, ${{ github.run_attempt }}; 'Select GitHub Token for Sync Mode' uses ${{ inputs.commit-status-github-token }}, ${{ inputs.github-pat }}.

Locations:

- `action.yml:120`
- `action.yml:138`
- `action.yml:175`
- `action.yml:185`
- `action.yml:196`
- `action.yml:213`
- `action.yml:219`
- `action.yml:230`
- `action.yml:236`
- `action.yml:257`
- `action.yml:290`
- `action.yml:316`

### github-env-injection (severity: high)

The 'Select GitHub Token for Sync Mode' step writes ${{ inputs.github-pat }} and ${{ inputs.commit-status-github-token }} directly to $GITHUB_OUTPUT without sanitization (no printf '%s' ... | tr -d newlines step). An attacker-controlled token value containing newlines could inject arbitrary key=value pairs into the output environment. Additionally, the 'YQ Platform settings' (metadata) step writes yq-parsed output from an external YAML file directly to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:316`
- `action.yml:192`

### unpinned-uses (severity: high)

Multiple uses: references in action.yml use mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks. Unpinned references: dcarbone/install-yq-action@v1.1.0, mamezou-tech/setup-helmfile@v2.2.0, actions/setup-node@v6, cloudposse/github-action-yaml-config-query@v1.0.1 (used 3 times), actions/checkout@v6, 1arp/create-a-file-action@0.2, nick-fields/retry@v4, cloudposse/github-action-wait-commit-status@v0.2.1. Only actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3 is correctly pinned to a SHA.

Locations:

- `action.yml:113`
- `action.yml:117`
- `action.yml:124`
- `action.yml:155`
- `action.yml:163`
- `action.yml:218`
- `action.yml:263`
- `action.yml:278`
- `action.yml:300`
- `action.yml:320`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all security findings in action.yml:

1. **unpinned-uses**: Pinned all 8 unpinned action references to their full 40-character commit SHAs with tag comments for readability.

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks for every affected step:
   - Install git-url-parse: runner.temp → RUNNER_TEMP
   - Parse git URL: inputs.cluster → INPUT_CLUSTER (used via process.env.INPUT_CLUSTER in github-script)
   - Read platform context: inputs.ssm-path, inputs.environment → SSM_PATH, ENVIRONMENT
   - Read platform metadata: inputs.ssm-path → SSM_PATH
   - Resolve kube version: inputs.kube-version, inputs.ssm-path, steps.metadata.outputs.kube_version → KUBE_VERSION, SSM_PATH, METADATA_KUBE_VERSION
   - Ensure argocd repo structure: steps.config.outputs.tmp → CONFIG_TMP
   - Helmfile render: all inputs → INPUT_NAMESPACE, INPUT_ENVIRONMENT, INPUT_PATH, INPUT_HELMFILE_ARGS, CONFIG_TMP, KUBE_VERSION_ARG
   - Build Helm Dependencies: inputs.path → INPUT_PATH
   - Helm raw render: all inputs → INPUT_VALUES_FILE, INPUT_APPLICATION, INPUT_PATH, INPUT_IMAGE, INPUT_IMAGE_TAG, INPUT_ENVIRONMENT, INPUT_NAMESPACE, INPUT_HELM_ARGS, CONFIG_TMP, KUBE_VERSION_ARG
   - Get Webapp: steps.config.outputs.tmp → CONFIG_TMP
   - Push to Github: inputs.operation, github.repository, github.sha, github.run_id, github.run_attempt → INPUT_OPERATION, GITHUB_REPOSITORY_NAME, GITHUB_SHA_VALUE, GITHUB_RUN_ID_VALUE, GITHUB_RUN_ATTEMPT_VALUE
   - Select GitHub Token: inputs.commit-status-github-token, inputs.github-pat → COMMIT_STATUS_GITHUB_TOKEN, GITHUB_PAT

3. **github-env-injection**: Sanitized all values written to $GITHUB_OUTPUT using `printf '%s' "$VAR" | tr -d '\n\r'` in the Select GitHub Token step and YQ Platform settings (metadata) step.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four security findings in action.yml:

1. **github-env-injection ('Resolve kube version', line 218)**: Sanitized `kube_version` with `printf '%s' "$kube_version" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

2. **script-injection ('Helmfile render', line 233)**: Replaced unquoted `$INPUT_HELMFILE_ARGS` with a bash array (`read -r -a helmfile_args_array <<< "$INPUT_HELMFILE_ARGS"`) and expanded it as `"${helmfile_args_array[@]}"`.

3. **script-injection ('Helm raw render', line 258)**: Replaced unquoted `${VALUES_STR}`, `$INPUT_HELM_ARGS`, and `$KUBE_VERSION_ARG` with properly quoted bash arrays. Values file entries are now added to a `values_args` array with individual `--values element` pairs; helm-args and kube-version-arg are read into arrays and expanded with `"${array[@]}"`.

4. **github-env-injection ('Get Webapp', line 274)**: Sanitized `WEBAPP_URL` with `printf '%s' "$WEBAPP_URL" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

