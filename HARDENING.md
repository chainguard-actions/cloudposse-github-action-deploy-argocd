<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.7.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

9 of 11 `uses:` references in action.yml use mutable tags or version strings instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if any of those upstream actions are compromised or their tags are moved. Only `actions/github-script@f28e40c7f34bde8b3046d885e986cb6290c5673b` is properly pinned. Unpinned references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (×3), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`.

Locations:

- `action.yml:103`
- `action.yml:108`
- `action.yml:115`
- `action.yml:148`
- `action.yml:153`
- `action.yml:199`
- `action.yml:252`
- `action.yml:260`
- `action.yml:268`
- `action.yml:318`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands (rule a), allowing an attacker who controls the inputs or referenced context values to inject arbitrary shell commands. Affected steps and offending lines:

(1) 'Install git-url-parse' (line 119): `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly.

(2) 'Parse git URL for destination' github-script (line 143): `run("${{ inputs.cluster }}")` — attacker-controlled `inputs.cluster` interpolated directly into the JS script string.

(3) 'Read platform context' (line 169): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — attacker-controlled inputs interpolated directly into shell.

(4) 'Read platform metadata' (line 181): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — attacker-controlled input interpolated directly.

(5) 'Resolve kube version' (line 193): `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` — attacker-controlled inputs interpolated directly.

(6) 'Ensure argocd repo structure' (line 208): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

(7) 'Helmfile render' (lines 213–219): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple attacker-controlled inputs interpolated directly.

(8) 'Build Helm Dependencies' (line 223): `helm dependency build ${{ inputs.path }}` — attacker-controlled input interpolated directly.

(9) 'Helm raw render' (lines 228–240): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple attacker-controlled inputs interpolated directly.

(10) 'Get Webapp' (line 251): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.

(11) 'Push to Github' command block (lines 280, 290, 302): `pushd ./${{ steps.destination_dir.outputs.name }}`, `case '${{ inputs.operation }}' in`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"` — step outputs and github context interpolated directly into shell.

(12) 'Select GitHub Token for Sync Mode' (lines 312–315): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — attacker-controlled inputs interpolated directly.

Locations:

- `action.yml:119`
- `action.yml:143`
- `action.yml:169`
- `action.yml:181`
- `action.yml:193`
- `action.yml:208`
- `action.yml:213`
- `action.yml:223`
- `action.yml:228`
- `action.yml:251`
- `action.yml:280`
- `action.yml:290`
- `action.yml:302`
- `action.yml:312`

### github-env-injection (severity: high)

Several `run:` blocks write unsanitized values derived from untrusted inputs or external data to `$GITHUB_OUTPUT` without applying the required `printf '%s' ... | tr -d '\n\r'` sanitization:

(1) 'YQ Platform settings' (metadata step, line 186): `echo "${output}" >> $GITHUB_OUTPUT` where `$output` is derived from yq-parsed SSM/external data read from `_metadata.yaml`. The value is not sanitized before being written to GITHUB_OUTPUT, allowing newline injection that could set arbitrary output variables.

(2) 'Resolve kube version' (line 196): `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is set from `${{ inputs.kube-version }}` (an attacker-controlled input interpolated directly into the script at line 193). No sanitization is applied before the write.

(3) 'Select GitHub Token for Sync Mode' (lines 313–315): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both write attacker-supplied input values directly to GITHUB_OUTPUT without sanitization, enabling newline injection.

Locations:

- `action.yml:186`
- `action.yml:196`
- `action.yml:313`
- `action.yml:315`

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

1. **unpinned-uses**: Pinned all 9 unpinned action references to full 40-char SHA digests with tag comments preserved.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ github.* }}, ${{ runner.* }}, and ${{ steps.*.outputs.* }} expressions from run: shell script bodies into env: blocks. Shell scripts now reference only environment variables. The github-script step now uses process.env.INPUT_CLUSTER instead of inline interpolation. helmfile-args and helm-args are tokenized with xargs+while-read-loop pattern for safe list expansion.

3. **github-env-injection**: Added `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` sanitization before all GITHUB_OUTPUT writes in: (a) YQ Platform settings/metadata step, (b) Resolve kube version step, (c) Select GitHub Token for Sync Mode step.

All shell commands now use only environment variables (no inline ${{ }} expressions), preventing shell injection attacks from attacker-controlled inputs.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get Webapp' step (line 279) in hardened/action/action.yml. The WEBAPP_URL value read from the helm/helmfile-rendered manifest file (resources.yaml) is now sanitized before being written to $GITHUB_OUTPUT. Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and changed the echo to use `${safe}` instead of `${WEBAPP_URL}`. This prevents newline injection attacks where user-controlled inputs could inject newlines into the webapp-url annotation value to poison $GITHUB_OUTPUT.

