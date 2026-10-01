<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.7.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags/versions instead of immutable full-length commit SHAs, making the action vulnerable to supply-chain attacks. Unpinned references:
- `dcarbone/install-yq-action@v1.1.0` (line 108)
- `mamezou-tech/setup-helmfile@v2.2.0` (line 113)
- `actions/setup-node@v6` (line 119)
- `cloudposse/github-action-yaml-config-query@v1.0.1` (lines 133, 191, 196, 253)
- `actions/checkout@v6` (line 138)
- `1arp/create-a-file-action@0.2` (line 258)
- `nick-fields/retry@v4` (line 265)
- `cloudposse/github-action-wait-commit-status@v0.2.1` (line 303)

Locations:

- `action.yml:108`
- `action.yml:113`
- `action.yml:119`
- `action.yml:133`
- `action.yml:138`
- `action.yml:191`
- `action.yml:196`
- `action.yml:253`
- `action.yml:258`
- `action.yml:265`
- `action.yml:303`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml interpolate `${{ }}` expressions directly into shell command strings (rule a), allowing an attacker-controlled value to inject arbitrary shell commands before the shell ever sees the string.

1. **Install git-url-parse** (~line 120): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly.

2. **Read platform context** (~line 155): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml --output-file ./platform.yaml` — attacker-controlled inputs interpolated directly into shell.

3. **Read platform metadata** (~line 168): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata --format yaml --output-file ./_metadata.yaml` — attacker-controlled input interpolated directly.

4. **Resolve kube version** (~line 181): `kube_version="${{ inputs.kube-version }}"` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.

5. **Ensure argocd repo structure** (~line 201): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

6. **Helmfile render** (~line 206): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.

7. **Build Helm Dependencies** (~line 219): `helm dependency build ${{ inputs.path }}` — input interpolated directly.

8. **Helm raw render** (~line 225): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set image.tag=${{ inputs.image-tag }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.

9. **Get Webapp** (~line 247): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly into shell path argument.

10. **Push to Github** (~line 285): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — github context and step outputs interpolated directly.

11. **Select GitHub Token** (~line 300): `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly into shell test expression.

Locations:

- `action.yml:120`
- `action.yml:155`
- `action.yml:168`
- `action.yml:181`
- `action.yml:201`
- `action.yml:206`
- `action.yml:219`
- `action.yml:225`
- `action.yml:247`
- `action.yml:285`
- `action.yml:300`

### github-env-injection (severity: high)

Two `run:` blocks write untrusted input values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. **Resolve kube version** (~line 184): The variable `$kube_version` is set from `${{ inputs.kube-version }}` (and `${{ steps.metadata.outputs.kube_version }}`), then written directly to `$GITHUB_OUTPUT` via `echo "value=$kube_version" >> $GITHUB_OUTPUT` without sanitization. An attacker can inject newlines to poison subsequent GITHUB_OUTPUT entries.

2. **Select GitHub Token** (~line 300-302): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — the raw input values (which may contain newlines) are written directly to `$GITHUB_OUTPUT` without sanitization, allowing GITHUB_OUTPUT injection.

Locations:

- `action.yml:184`
- `action.yml:301`
- `action.yml:303`

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

1. **unpinned-uses**: Pinned all 8 action references to full commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x4, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ steps.* }}, and ${{ runner.* }} expressions out of run: shell blocks into env: maps. Affected steps: Install git-url-parse, Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Get Webapp, Push to Github, Select GitHub Token. The Push to Github step uses built-in GITHUB_REPOSITORY/GITHUB_SHA/GITHUB_RUN_ID/GITHUB_RUN_ATTEMPT env vars for the commit message.

3. **github-env-injection**: Sanitized values written to $GITHUB_OUTPUT using `printf '%s' "$VAR" | tr -d '\n\r'` in the Resolve kube version step and the Select GitHub Token step.

4. **helmfile-args / helm-args / values_file**: These list-style inputs are tokenized using the xargs+while-read-NUL pattern to preserve argument boundaries while preventing injection.

Note: ${{ inputs.cluster }} remains in the actions/github-script `script:` block (JavaScript context, not a shell run: block) — this is the same pattern as the already-pinned github-script step and is not a shell injection vector.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Helm raw render' step in action.yml: replaced the unsafe VALUES_STR string concatenation approach (which used unquoted `${VALUES_STR}` expansion) with a proper bash array `values_args`. Each file from INPUT_VALUES_FILE is now added as `values_args+=(--values "$element")` and expanded as `"${values_args[@]}"` in the helm template command. This ensures each element is properly quoted as a separate shell word, preventing injection of shell metacharacters via the values_file input.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:
1. Script injection (line 131): Moved `inputs.cluster` out of the github-script `script:` block by adding `INPUT_CLUSTER: ${{ inputs.cluster }}` to the step's `env:` block and changing `run("${{ inputs.cluster }}")` to `run(process.env.INPUT_CLUSTER)`. This prevents the attacker-controlled cluster URL from being embedded verbatim into JavaScript source code.
2. GitHub env injection (line 196, metadata step): Added `safe_output=$(printf '%s' "${output}" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, sanitizing newlines from SSM-derived metadata values.
3. GitHub env injection (line 247, Get Webapp step): Added `safe_webapp_url=$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, sanitizing newlines from the Helm/helmfile-rendered webapp URL.

