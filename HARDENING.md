<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags or version strings instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Unpinned references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v4`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`. (Note: `actions/github-script@f28e40c7f34bde8b3046d885e986cb6290c5673b` is correctly pinned.)

Locations:

- `action.yml:116`
- `action.yml:122`
- `action.yml:129`
- `action.yml:163`
- `action.yml:172`
- `action.yml:222`
- `action.yml:300`
- `action.yml:308`
- `action.yml:318`
- `action.yml:355`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, enabling script injection. Sub-rule (a) violations — direct expression interpolation in shell:

1. **Install git-url-parse** step: `mkdir -p ${{ runner.temp }}/action-deps` — `runner.temp` is interpolated directly into the shell command.

2. **Parse git URL for destination** step (github-script): `run("${{ inputs.cluster }}")` — `inputs.cluster` is interpolated directly into the JavaScript/shell context.

3. **Read platform context** step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — both `inputs.ssm-path` and `inputs.environment` are interpolated directly.

4. **Read platform metadata** step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — `inputs.ssm-path` interpolated directly.

5. **Resolve kube version** step: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` — inputs interpolated directly into shell.

6. **Helmfile render** step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.

7. **Build Helm Dependencies** step: `helm dependency build ${{ inputs.path }}` — `inputs.path` interpolated directly.

8. **Helm raw render** step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.

9. **Push to Github** step (nick-fields/retry command): `case '${{ inputs.operation }}' in`, `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple expressions interpolated directly.

10. **Select GitHub Token for Sync Mode** step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly.

Locations:

- `action.yml:135`
- `action.yml:155`
- `action.yml:196`
- `action.yml:204`
- `action.yml:215`
- `action.yml:248`
- `action.yml:261`
- `action.yml:266`
- `action.yml:325`
- `action.yml:345`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. **YQ Platform settings (metadata)** step: Writes SSM-derived key=value pairs from `_metadata.yaml` directly to `$GITHUB_OUTPUT` via `echo "${output}" >> $GITHUB_OUTPUT` in a loop. The metadata values come from an external SSM store and are not sanitized — a newline in any value could inject arbitrary outputs.

2. **Resolve kube version** step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `kube_version` is set from `${{ inputs.kube-version }}` (direct input interpolation) or from `${{ steps.metadata.outputs.kube_version }}` (SSM-derived). Neither path sanitizes the value before writing to `$GITHUB_OUTPUT`.

3. **Select GitHub Token for Sync Mode** step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both write unsanitized input values directly to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:207`
- `action.yml:215`
- `action.yml:345`

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

Fixed all security findings in action.yml:

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-character SHA hashes with tag comments preserved: dcarbone/install-yq-action@4d2a79..., mamezou-tech/setup-helmfile@c04e83..., actions/setup-node@49933e..., cloudposse/github-action-yaml-config-query@8178e0... (3x), actions/checkout@df4cb1..., 1arp/create-a-file-action@ee80ed..., nick-fields/retry@ad9845..., cloudposse/github-action-wait-commit-status@2ad2cb...

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ runner.temp }} expressions out of run: shell blocks into env: maps. Shell commands now reference plain environment variables ($VAR_NAME). Optional args (helmfile-args, helm-args) use ${VAR:+"$VAR"} form to avoid passing empty arguments.

3. **github-env-injection**: Added printf '%s' ... | tr -d '\n\r' sanitization before all writes to $GITHUB_OUTPUT in: (a) YQ Platform settings/metadata loop, (b) Resolve kube version step, (c) Select GitHub Token for Sync Mode step.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings in action.yml:

1. 'Ensure argocd repo structure' step: Moved `${{ steps.config.outputs.tmp }}` to `CONFIG_TMP` env var; used `"${CONFIG_TMP}/manifests"` with proper quoting.

2. 'Helmfile render' step: Moved `${{ steps.arguments.outputs.kube_version }}` to `KUBE_VERSION_ARG` env var and `${{ steps.config.outputs.tmp }}` to `CONFIG_TMP` env var; used them properly quoted in the shell script.

3. 'Helm raw render' step: Moved `${{ steps.arguments.outputs.kube_version }}` to `KUBE_VERSION_ARG` and `${{ steps.config.outputs.tmp }}` to `CONFIG_TMP` env vars; replaced unquoted `${array[@]}` and string-concatenated `${VALUES_STR}` with a properly quoted bash array `values_args`; used `${KUBE_VERSION_ARG:+"$KUBE_VERSION_ARG"}` for the optional positional argument.

4. 'Get Webapp' step: Moved `${{ steps.config.outputs.tmp }}` to `CONFIG_TMP` env var; added sanitization `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` before writing to `$GITHUB_OUTPUT` to prevent newline injection.

