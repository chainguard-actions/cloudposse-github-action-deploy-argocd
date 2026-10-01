<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable version tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved. Failing references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (×3 occurrences), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`.

Locations:

- `action.yml:96`
- `action.yml:100`
- `action.yml:106`
- `action.yml:130`
- `action.yml:155`
- `action.yml:222`
- `action.yml:237`
- `action.yml:302`
- `action.yml:320`
- `action.yml:378`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell command strings (rule a), allowing an attacker who controls those inputs to inject arbitrary shell commands. Affected steps and offending lines include:

- **Install git-url-parse** (run:): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly.
- **Parse git URL for destination** (script:): `run("${{ inputs.cluster }}")` — attacker-controlled `inputs.cluster` interpolated directly into JS.
- **Read platform context** (run:): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — attacker-controlled inputs interpolated directly into shell.
- **Read platform metadata** (run:): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — same issue.
- **Resolve kube version** (run:): `kube_version="${{ inputs.kube-version }}"` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.
- **Ensure argocd repo structure** (run:): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.
- **Helmfile render** (run:): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path }} ... ${{ inputs.helmfile-args }}` — multiple attacker-controlled inputs interpolated directly.
- **Build Helm Dependencies** (run:): `helm dependency build ${{ inputs.path }}` — attacker-controlled input interpolated directly.
- **Helm raw render** (run:): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple attacker-controlled inputs interpolated directly.
- **Get Webapp** (run:): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.
- **Push to Github** (command:): `pushd ./${{ steps.destination_dir.outputs.name }}`, `case '${{ inputs.operation }}'`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple contexts interpolated directly.
- **Select GitHub Token for Sync Mode** (run:): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — sensitive inputs interpolated directly into shell.

Locations:

- `action.yml:113`
- `action.yml:122`
- `action.yml:163`
- `action.yml:175`
- `action.yml:193`
- `action.yml:205`
- `action.yml:215`
- `action.yml:228`
- `action.yml:248`
- `action.yml:258`
- `action.yml:270`
- `action.yml:285`
- `action.yml:330`
- `action.yml:355`
- `action.yml:362`
- `action.yml:366`
- `action.yml:385`
- `action.yml:388`
- `action.yml:390`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs or external data to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. **YQ Platform settings / metadata step**: `echo "${output}" >> $GITHUB_OUTPUT` — `${output}` is derived from yq-parsed SSM/metadata YAML content (external, attacker-influenced data). No newline sanitization is applied before the write.

2. **Resolve kube version step**: `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set directly from `${{ inputs.kube-version }}` (attacker-controlled input) or `${{ steps.metadata.outputs.kube_version }}` (external SSM data). No sanitization before the write.

3. **Select GitHub Token for Sync Mode step**: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — sensitive token inputs are written directly to `$GITHUB_OUTPUT` without sanitization, enabling newline injection that could poison subsequent environment variables.

Locations:

- `action.yml:205`
- `action.yml:215`
- `action.yml:228`
- `action.yml:388`
- `action.yml:390`

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

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 10 action references to full 40-character commit SHAs using lookup_action_sha. Original tags preserved as comments.

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks. Key changes:
   - Install git-url-parse: runner.temp → RUNNER_TEMP env var
   - Parse git URL (JS script): inputs.cluster → INPUT_CLUSTER env var, referenced via process.env.INPUT_CLUSTER
   - Read platform context: inputs.ssm-path + inputs.environment → INPUT_SSM_PATH + INPUT_ENVIRONMENT
   - Read platform metadata: inputs.ssm-path → INPUT_SSM_PATH
   - Resolve kube version: all three expressions → INPUT_KUBE_VERSION, INPUT_SSM_PATH, METADATA_KUBE_VERSION
   - Ensure argocd repo structure: steps.config.outputs.tmp → CONFIG_TMP
   - Helmfile render: all inputs → env vars (INPUT_NAMESPACE, INPUT_ENVIRONMENT, INPUT_PATH, INPUT_HELMFILE_ARGS, KUBE_VERSION_ARG, CONFIG_TMP)
   - Build Helm Dependencies: inputs.path → INPUT_PATH
   - Helm raw render: all inputs → env vars
   - Get Webapp: steps.config.outputs.tmp → CONFIG_TMP
   - Push to Github: all context values → DESTINATION_DIR, DESTINATION_REF, INPUT_OPERATION, CONFIG_PATH, GIT_COMMIT_MSG env vars
   - Select GitHub Token: both token inputs → INPUT_COMMIT_STATUS_GITHUB_TOKEN, INPUT_GITHUB_PAT

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before all $GITHUB_OUTPUT writes that use attacker-influenced values:
   - YQ Platform settings (metadata): each loop output sanitized
   - Resolve kube version: kube_version sanitized before write
   - Select GitHub Token: both token values sanitized before write

Note: helmfile-args and helm-args are list-style inputs used unquoted (as upstream intended for word-splitting), which is correct behavior for these argument list inputs.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. **Helmfile render (script-injection)**: Replaced unquoted `$INPUT_HELMFILE_ARGS` expansion with xargs-based tokenization into a bash array `helmfile_args`, expanded as `"${helmfile_args[@]}"`. This prevents shell metacharacter injection while preserving argument boundaries.

2. **Helm raw render (script-injection)**: Replaced three unquoted expansions:
   - `${VALUES_STR}` (built from comma-separated `$INPUT_VALUES_FILE`): now uses a proper bash array `values_args` with each file path quoted as `--values "$element"`
   - `$INPUT_HELM_ARGS`: tokenized via xargs into `helm_args` array, expanded as `"${helm_args[@]}"`
   - `$KUBE_VERSION_ARG`: placed in conditional `kube_version_args` array (single value, either empty or `--kube-version=X`), expanded as `"${kube_version_args[@]}"`

3. **Get Webapp (github-env-injection)**: Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` sanitization step before writing to `$GITHUB_OUTPUT`, preventing newline injection that could override subsequent output variables.

