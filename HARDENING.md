<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.7.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or overwritten. Failing references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3×), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`. (The `actions/github-script` reference is correctly pinned to a SHA and is not flagged.)

Locations:

- `action.yml:97`
- `action.yml:102`
- `action.yml:109`
- `action.yml:143`
- `action.yml:160`
- `action.yml:232`
- `action.yml:316`
- `action.yml:340`
- `action.yml:393`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions (inputs, steps outputs, github context) into shell commands via YAML template substitution. This allows an attacker-controlled value to inject arbitrary shell commands before the shell ever parses the string. Specific violations (sub-rule a):

- 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`
- 'Parse git URL for destination' script block: `run("${{ inputs.cluster }}")`
- 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}`
- 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata`
- 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"`
- 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`
- 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path }} ... ${{ inputs.helmfile-args }}`
- 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}`
- 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}`
- 'Get Webapp' step: `${{ steps.config.outputs.tmp }}/manifests/resources.yaml`
- 'Push to Github' (nick-fields/retry command): `pushd ./${{ steps.destination_dir.outputs.name }}`, `case '${{ inputs.operation }}'`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}`
- 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]`

Locations:

- `action.yml:115`
- `action.yml:130`
- `action.yml:183`
- `action.yml:200`
- `action.yml:218`
- `action.yml:243`
- `action.yml:249`
- `action.yml:261`
- `action.yml:271`
- `action.yml:278`
- `action.yml:295`
- `action.yml:355`
- `action.yml:383`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline in the value can inject arbitrary key=value pairs into the output file, poisoning downstream steps.

1. 'Resolve kube version' step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` — where `$kube_version` is set from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` without sanitization.

2. 'Select GitHub Token for Sync Mode' step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both write unsanitized `inputs.*` values directly to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:228`
- `action.yml:384`
- `action.yml:386`

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

Rewrote action.yml with all security fixes:

1. **unpinned-uses**: Pinned all 9 action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps: Install git-url-parse (runner.temp), Parse git URL (inputs.cluster), Read platform context (inputs.ssm-path, inputs.environment), Read platform metadata (inputs.ssm-path), Resolve kube version (inputs.kube-version, inputs.ssm-path, steps.metadata.outputs.kube_version), Ensure argocd repo structure (steps.config.outputs.tmp), Helmfile render (all inputs.* and steps.*), Build Helm Dependencies (inputs.path), Helm raw render (all inputs.*), Get Webapp (steps.config.outputs.tmp), Push to Github (steps.destination_dir.outputs.name, steps.destination.outputs.ref, inputs.operation, steps.config.outputs.path, github.* replaced with built-in GITHUB_* env vars), Select GitHub Token (inputs.commit-status-github-token, inputs.github-pat). List-type inputs (helmfile-args, helm-args) are tokenized with xargs for correct argument splitting.

3. **github-env-injection**: Sanitized all values written to $GITHUB_OUTPUT using printf '%s' | tr -d '\n\r' in Resolve kube version and Select GitHub Token steps.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. github-env-injection (metadata step, ~line 196): Added sanitization of each `output` value before writing to $GITHUB_OUTPUT using `safe_output="$(printf '%s' "${output}" | tr -d '\n\r')"` to strip newlines that could inject additional key=value pairs.

2. script-injection (Helm raw render step, ~line 283): Replaced unquoted `${array[@]}` expansion with properly quoted `"${array[@]}"` in the for loop, and built a `values_args` bash array with properly quoted elements (`values_args+=(--values "$element")`), then used `"${values_args[@]}"` in the helm template command instead of the old unquoted `${VALUES_STR}`.

3. github-env-injection (Get Webapp step, ~line 301): Added sanitization of `WEBAPP_URL` before writing to $GITHUB_OUTPUT using `safe_webapp_url="$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')"` to strip newlines that could inject additional entries.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two locations in action.yml where backslashes were incorrectly added before double quotes in yq query strings. Line ~296: restored `query: ."${{ steps.resolved_kube_version.outputs.value == '' }}"` to `query: ."${{ steps.resolved_kube_version.outputs.value == '' }}"`. Line ~419: restored `query: ."${{ inputs.gitref-sha == '' }}"` to `query: ."${{ inputs.gitref-sha == '' }}"`. The backslashes were literal text that caused yq to fail with "Can't handle character".

