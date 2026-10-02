<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions into shell commands (sub-rule a), allowing an attacker-controlled value to break out of the intended command context. Affected steps and offending lines:

1. 'Install git-url-parse' (line ~154): `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.
2. 'Read platform context' (line ~234): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — inputs interpolated directly.
3. 'Read platform metadata' (line ~254): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — input interpolated directly.
4. 'Resolve kube version' (lines ~274-276): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.
5. 'Ensure argocd repo structure' (line ~295): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.
6. 'Helmfile render' (lines ~302-308): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.
7. 'Build Helm Dependencies' (line ~318): `helm dependency build ${{ inputs.path }}` — input interpolated directly.
8. 'Helm raw render' (lines ~325-338): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.
9. 'Get Webapp' (line ~347): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly in shell.
10. 'Push to Github' / command: field (lines ~370-391): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `cp -r ./tmp/* ./${{ steps.destination_dir.outputs.name }}/`, `rm -rf .../${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — inputs, step outputs, and github context interpolated directly.
11. 'Select GitHub Token for Sync Mode' (lines ~402-405): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — inputs interpolated directly.

Locations:

- `action.yml:154`
- `action.yml:234`
- `action.yml:254`
- `action.yml:274`
- `action.yml:295`
- `action.yml:302`
- `action.yml:318`
- `action.yml:325`
- `action.yml:347`
- `action.yml:370`
- `action.yml:402`

### github-env-injection (severity: high)

Multiple run: steps write values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. 'Resolve kube version' (line ~278): `echo "value=$kube_version" >> $GITHUB_OUTPUT` — $kube_version is set directly from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` without sanitization. An attacker-controlled newline in the input can inject arbitrary key=value pairs into GITHUB_OUTPUT.

2. 'Select GitHub Token for Sync Mode' (lines ~403, ~405): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — inputs are written directly to GITHUB_OUTPUT without sanitization. A newline in the token value would allow injecting additional output variables.

Locations:

- `action.yml:278`
- `action.yml:403`
- `action.yml:405`

### unpinned-uses (severity: high)

Multiple uses: references in action.yml use mutable version tags instead of immutable 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

- `dcarbone/install-yq-action@v1.3.1` (line 132)
- `mamezou-tech/setup-helmfile@v2.2.0` (line 139)
- `actions/setup-node@v6` (line 147)
- `cloudposse/github-action-yaml-config-query@v1.0.1` (lines 193, 283, 351)
- `actions/checkout@v6` (line 200)
- `1arp/create-a-file-action@0.4` (line 353)
- `nick-fields/retry@v4` (line 361)
- `cloudposse/github-action-wait-commit-status@v0.2.1` (line 408)

Note: `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned to a full SHA.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:200`
- `action.yml:283`
- `action.yml:351`
- `action.yml:353`
- `action.yml:361`
- `action.yml:408`

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

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-character commit SHAs with tag comments for readability.

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks and referenced them as shell variables. Key changes:
   - 'Install git-url-parse': replaced ${{ runner.temp }} with $RUNNER_TEMP (built-in env var)
   - 'Read platform context': moved ssm-path and environment inputs to env vars
   - 'Read platform metadata': moved ssm-path input to env var
   - 'Resolve kube version': moved kube-version, ssm-path, and metadata step output to env vars
   - 'Ensure argocd repo structure': moved config.outputs.tmp to CONFIG_TMP env var
   - 'Helmfile render': moved all inputs and step outputs to env vars
   - 'Build Helm Dependencies': moved path input to env var
   - 'Helm raw render': moved all inputs and step outputs to env vars
   - 'Get Webapp': moved config.outputs.tmp to CONFIG_TMP env var
   - 'Push to Github': moved all step outputs to env vars; github context (GITHUB_REPOSITORY, GITHUB_SHA, GITHUB_RUN_ID, GITHUB_RUN_ATTEMPT) used as built-in env vars
   - 'Select GitHub Token for Sync Mode': moved both token inputs to env vars

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing values to GITHUB_OUTPUT in 'Resolve kube version' and 'Select GitHub Token for Sync Mode' steps.

Note: ${{ }} expressions in with:, if:, and env: blocks are safe and were left as-is. The helmfile-args and helm-args inputs are intentionally left unquoted in the run: block (as env vars) since they are argument lists that need word-splitting.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four security findings in hardened/action/action.yml:

1. github-env-injection (line 213, 'YQ Platform settings' step): Added `safe_output=$(printf '%s' "${output}" | tr -d '\n\r')` before writing loop variable to $GITHUB_OUTPUT to strip newlines from SSM metadata values.

2. script-injection (line 248, 'Helmfile render' step): Replaced unquoted `$INPUT_HELMFILE_ARGS` with xargs-based tokenization into a bash array `helmfile_extra_args`, expanded as `"${helmfile_extra_args[@]}"`.

3. script-injection (lines 285-287, 'Helm raw render' step): Replaced three unquoted expansions (${VALUES_STR}, $INPUT_HELM_ARGS, $ARGUMENTS_KUBE_VERSION) with properly tokenized bash arrays (values_args, helm_extra_args, kube_version_args) using xargs for the argument lists, then expanded as quoted arrays.

4. github-env-injection (line 302, 'Get Webapp' step): Added `safe_webapp_url=$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT to strip newlines from Kubernetes manifest annotation values.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Parse git URL for destination' step of action.yml. Moved `${{ inputs.cluster }}` out of the JavaScript body and into the step's `env:` block as `CLUSTER: ${{ inputs.cluster }}`. Updated the script call from `run("${{ inputs.cluster }}")` to `run(process.env.CLUSTER)` so the user-controlled value is never interpolated into the script source.

