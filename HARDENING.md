<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags or version strings instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks. Unpinned references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`. (Note: `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned.)

Locations:

- `action.yml:103`
- `action.yml:108`
- `action.yml:115`
- `action.yml:131`
- `action.yml:152`
- `action.yml:163`
- `action.yml:232`
- `action.yml:270`
- `action.yml:310`
- `action.yml:338`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ inputs.* }}`, `${{ steps.*.outputs.* }}`, `${{ github.* }}`) into shell commands, enabling script injection. Sub-rule (a) violations include:

1. 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` — runner.temp interpolated directly.
2. 'Parse git URL for destination' (github-script): `run("${{ inputs.cluster }}")` — inputs.cluster interpolated into JS run() call.
3. 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated.
4. 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly into shell.
5. 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — inputs.path interpolated.
6. 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated.
7. 'Get Webapp' step: `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated.
8. 'Push to Github' step (via nick-fields/retry command): `case '${{ inputs.operation }}' in`, `pushd ./${{ steps.destination_dir.outputs.name }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple github context and step outputs interpolated.
9. 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated into shell condition.

Locations:

- `action.yml:120`
- `action.yml:131`
- `action.yml:218`
- `action.yml:240`
- `action.yml:248`
- `action.yml:255`
- `action.yml:270`
- `action.yml:285`
- `action.yml:310`
- `action.yml:338`

### github-env-injection (severity: high)

Multiple `run:` blocks write untrusted input values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. 'Resolve kube version' step: assigns `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` to the shell variable `kube_version`, then writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` without sanitization. An attacker-controlled input containing newlines can inject arbitrary key=value pairs into GITHUB_OUTPUT.

2. 'YQ Platform settings' (metadata) step: writes `echo "${output}" >> $GITHUB_OUTPUT` where `${output}` is derived from external SSM/YAML data without sanitization.

3. 'Select GitHub Token for Sync Mode' step: writes `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both are direct interpolations of caller-supplied inputs into GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:218`
- `action.yml:200`
- `action.yml:338`

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

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-character SHA hashes using lookup_action_sha. All uses: references now include the SHA with a tag comment.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ steps.*.outputs.* }}, and ${{ github.* }} expressions from run: blocks to env: blocks. Key changes:
   - runner.temp → RUNNER_TEMP env var
   - inputs.cluster → INPUT_CLUSTER, used via process.env.INPUT_CLUSTER in JS
   - All inputs in Helmfile render, Build Helm Dependencies, Helm raw render steps moved to env:
   - All github context values in Push to Github moved to env:
   - inputs.commit-status-github-token and inputs.github-pat in Select GitHub Token moved to env:
   - List-type inputs (helmfile-args, helm-args, values_file) tokenized with xargs-based array construction

3. **github-env-injection**: Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to GITHUB_OUTPUT in:
   - YQ Platform settings (metadata) step
   - Resolve kube version step
   - Select GitHub Token for Sync Mode step

Note: ${{ }} expressions in with:, if:, and outputs: blocks were left as-is since those are not shell injection risks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get Webapp' step in action.yml (around line 214) by adding sanitization of the WEBAPP_URL value before writing to $GITHUB_OUTPUT. Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and changed the echo to use `${safe}` instead of `${WEBAPP_URL}`. This prevents newline injection attacks where an attacker-controlled annotation value in the rendered Helm/Helmfile manifest could inject additional key=value pairs into $GITHUB_OUTPUT.

