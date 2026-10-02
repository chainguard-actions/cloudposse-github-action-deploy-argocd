<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.1** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags or version strings instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Unpinned references: `dcarbone/install-yq-action@v1.3.1`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used three times), `1arp/create-a-file-action@0.4`, `nick-fields/retry@v4`.

Locations:

- `action.yml:97`
- `action.yml:103`
- `action.yml:110`
- `action.yml:143`
- `action.yml:222`
- `action.yml:296`
- `action.yml:313`
- `action.yml:340`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ inputs.* }}` and `${{ github.* }}` expressions inside shell command strings (sub-rule a). This allows an attacker who controls those inputs to inject arbitrary shell commands. Affected steps and offending lines include:

- 'Parse git URL for destination' step: `run("${{ inputs.cluster }}")` — inputs.cluster interpolated directly into a JS/shell string.
- 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — inputs.ssm-path and inputs.environment interpolated directly.
- 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — inputs.ssm-path interpolated directly.
- 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` — inputs.kube-version and inputs.ssm-path interpolated directly.
- 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — steps output interpolated directly.
- 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path }} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.
- 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — inputs.path interpolated directly.
- 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.
- 'Get Webapp' step: `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.
- 'Push to Github' step: `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"` — github context values interpolated directly.
- 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — inputs interpolated directly.

Locations:

- `action.yml:131`
- `action.yml:175`
- `action.yml:193`
- `action.yml:207`
- `action.yml:225`
- `action.yml:234`
- `action.yml:243`
- `action.yml:253`
- `action.yml:263`
- `action.yml:275`
- `action.yml:285`
- `action.yml:330`
- `action.yml:358`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. 'YQ Platform settings' (metadata) step: writes output from SSM/chamber data (`${output}`) to `$GITHUB_OUTPUT` without sanitization — the SSM data is externally controlled and could contain newlines that inject additional key=value pairs.

2. 'Resolve kube version' step: writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is derived from `${{ inputs.kube-version }}` (interpolated directly) or from SSM metadata — no sanitization applied.

3. 'Select GitHub Token for Sync Mode' step: writes `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — sensitive token values from inputs are written to GITHUB_OUTPUT without sanitization, allowing newline injection.

Locations:

- `action.yml:207`
- `action.yml:220`
- `action.yml:358`
- `action.yml:360`

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

Rewrote action.yml to fix all security findings:

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, 1arp/create-a-file-action, nick-fields/retry).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }} and ${{ github.* }} expressions from run: blocks into env: blocks, then referenced them as plain shell variables. For list-type inputs (helmfile-args, helm-args), used xargs-based tokenization into bash arrays to preserve argument boundaries. For the github-script step, used process.env.INPUT_CLUSTER instead of inline interpolation.

3. **github-env-injection**: Added `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` sanitization before all GITHUB_OUTPUT writes in: YQ Platform settings (metadata loop), Resolve kube version, and Select GitHub Token for Sync Mode steps.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Three fixes applied to hardened/action/action.yml:
1. 'Install git-url-parse' step (lines 154-155): Replaced `${{ runner.temp }}` template expressions inside the run: shell block with `$RUNNER_TEMP` environment variable to eliminate script-injection risk. The `${{ runner.temp }}` in the adjacent env: block (NODE_PATH) was left as-is since env: values are not subject to shell injection.
2. 'Helm raw render' step (lines 356, 370): Quoted `${array[@]}` → `"${array[@]}"` and `${VALUES_STR}` → `"${VALUES_STR}"` to prevent shell metacharacter injection from user-controlled `inputs.values_file`.
3. 'Get Webapp' step (line 389): Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` sanitization step and wrote `safe` to GITHUB_OUTPUT instead of the raw `WEBAPP_URL` value, preventing newline-based injection into GITHUB_OUTPUT.

