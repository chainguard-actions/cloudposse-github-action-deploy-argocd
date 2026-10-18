<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 26 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions into shell commands, violating sub-rule (a). This allows an attacker who controls the inputs or referenced context values to inject arbitrary shell commands.

1. 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly in run:.
2. 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml --output-file ./platform.yaml` — inputs interpolated directly in run:.
3. 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata --format yaml --output-file ./_metadata.yaml` — inputs interpolated directly in run:.
4. 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly in run:.
5. 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly in run:.
6. 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — multiple inputs and step outputs interpolated directly in run:.
7. 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — input interpolated directly in run:.
8. 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — multiple inputs and step outputs interpolated directly in run:.
9. 'Get Webapp' step: `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly in run:.
10. 'Push to Github' command: `pushd ./${{ steps.destination_dir.outputs.name }}`, `origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}'`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple inputs, step outputs, and github context values interpolated directly in the shell command.
11. 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly in run:.

Locations:

- `action.yml:147`
- `action.yml:196`
- `action.yml:210`
- `action.yml:220`
- `action.yml:237`
- `action.yml:243`
- `action.yml:256`
- `action.yml:263`
- `action.yml:283`
- `action.yml:320`
- `action.yml:345`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'Resolve kube version' step: sets shell variable `kube_version` from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` (both untrusted), then writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` without sanitization. A newline-containing input value could inject additional output variables.

2. 'Select GitHub Token for Sync Mode' step: directly writes `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both are direct writes of input values to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:224`
- `action.yml:346`
- `action.yml:348`

### unpinned-uses (severity: high)

Multiple uses: references in action.yml are pinned to mutable tags or version strings instead of immutable full 40-character commit SHAs. This exposes the action to supply-chain attacks where a tag is moved to point to malicious code.

Unpinned references found:
- uses: dcarbone/install-yq-action@v1.1.0
- uses: mamezou-tech/setup-helmfile@v2.2.0
- uses: actions/setup-node@v6
- uses: cloudposse/github-action-yaml-config-query@v1.0.1 (appears 3 times)
- uses: actions/checkout@v6
- uses: 1arp/create-a-file-action@0.2
- uses: nick-fields/retry@v4
- uses: cloudposse/github-action-wait-commit-status@v0.2.1

Only actions/github-script is correctly pinned to a full SHA: actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3

Locations:

- `action.yml:128`
- `action.yml:133`
- `action.yml:140`
- `action.yml:174`
- `action.yml:179`
- `action.yml:228`
- `action.yml:295`
- `action.yml:305`
- `action.yml:313`
- `action.yml:356`

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

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-character commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks for all affected steps: Install git-url-parse (runner.temp), Read platform context (ssm-path, environment), Read platform metadata (ssm-path), Resolve kube version (kube-version, ssm-path, metadata output), Ensure argocd repo structure (config.outputs.tmp), Helmfile render (namespace, environment, path, helmfile-args, kube_version, config.outputs.tmp), Build Helm Dependencies (path), Helm raw render (values_file, application, path, image, image-tag, environment, namespace, helm-args, kube_version, config.outputs.tmp), Get Webapp (config.outputs.tmp), Push to Github (destination_dir.outputs.name, destination.outputs.ref, operation, config.outputs.path; github context replaced with built-in GITHUB_* env vars), Select GitHub Token (commit-status-github-token, github-pat). List-type inputs (helmfile-args, helm-args) are tokenized with the xargs idiom into bash arrays.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to $GITHUB_OUTPUT in: Resolve kube version step (kube_version value) and Select GitHub Token step (both token values).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'YQ Platform settings' step (id: metadata, ~line 200): Added sanitization of the `output` loop variable (sourced from SSM/_metadata.yaml) before writing to $GITHUB_OUTPUT. Now uses `safe_output="$(printf '%s' "${output}" | tr -d '\n\r')"` and writes `${safe_output}` instead of `${output}`.

2. 'Get Webapp' step (~line 265): Added sanitization of `WEBAPP_URL` (extracted from rendered Kubernetes manifests) before writing to $GITHUB_OUTPUT. Now uses `safe_webapp_url="$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')"` and writes `${safe_webapp_url}` instead of `${WEBAPP_URL}`.

Both fixes follow the prescribed pattern: capture raw value, sanitize with printf+tr at the write site (not nested inside another command substitution), then write the sanitized value.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:

1. 'Parse git URL for destination' step (line ~130): Moved `inputs.cluster` from inline JavaScript interpolation `run("${{ inputs.cluster }}")` to an env var `CLUSTER_URL: ${{ inputs.cluster }}` and referenced it safely as `process.env.CLUSTER_URL` in the script body.

2. 'Helm raw render' step (line ~272): Replaced the unquoted array expansion `for element in ${array[@]}` and unquoted `VALUES_STR+="--values $element "` with a proper bash array `values_args=()` that uses quoted expansion `"${array[@]}"` and quoted element insertion `values_args+=(--values "$element")`. The `helm template` call now uses `"${values_args[@]}"` instead of the unquoted `${VALUES_STR}`.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two query lines in action.yml where backslashes had been incorrectly added before double quotes. Line 297 (resolved_kube_version query) and line 422 (gitref-sha query) both had `.\"...\"` instead of the correct `."..."`. Removed the backslashes to restore upstream's original quoting, which is required for yq to correctly parse the query expressions.

