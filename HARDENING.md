<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a), allowing an attacker who controls input values to inject arbitrary shell commands. Affected steps and examples:

- **Install git-url-parse** (line ~116): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.
- **Read platform context** (line ~196): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — attacker-controlled inputs interpolated directly.
- **Read platform metadata** (line ~218): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — same issue.
- **Resolve kube version** (line ~228): `kube_version="${{ inputs.kube-version }}"` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.
- **Ensure argocd repo structure** (line ~258): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.
- **Helmfile render** (line ~264): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.
- **Build Helm Dependencies** (line ~285): `helm dependency build ${{ inputs.path }}` — input interpolated directly.
- **Helm raw render** (line ~291): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ...` — many inputs interpolated directly.
- **Get Webapp** (line ~322): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.
- **Push to Github** (line ~358): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple contexts interpolated directly.
- **Select GitHub Token** (line ~393): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — inputs interpolated directly in shell conditionals and echo commands.

Locations:

- `action.yml:116`
- `action.yml:196`
- `action.yml:218`
- `action.yml:228`
- `action.yml:258`
- `action.yml:264`
- `action.yml:285`
- `action.yml:291`
- `action.yml:322`
- `action.yml:358`
- `action.yml:393`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

- **Resolve kube version** step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is set directly from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` — an attacker-controlled newline in the value can inject additional key=value pairs into GITHUB_OUTPUT.
- **YQ Platform settings / metadata** step: `echo "${output}" >> $GITHUB_OUTPUT` where `${output}` is derived from SSM-sourced YAML parsed by yq — the value is externally influenced and not sanitized before writing.
- **Select GitHub Token** step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — token inputs are written directly to GITHUB_OUTPUT without sanitization, allowing newline injection.

Locations:

- `action.yml:237`
- `action.yml:222`
- `action.yml:395`
- `action.yml:397`

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable version tags or short version strings instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised. Unpinned references found:

- `dcarbone/install-yq-action@v1.1.0`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v6`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used 4 times)
- `actions/checkout@v6`
- `1arp/create-a-file-action@0.2`
- `nick-fields/retry@v4`
- `cloudposse/github-action-wait-commit-status@v0.2.1`

Only `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned to a full SHA.

Locations:

- `action.yml:97`
- `action.yml:103`
- `action.yml:110`
- `action.yml:148`
- `action.yml:163`
- `action.yml:243`
- `action.yml:335`
- `action.yml:401`

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

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-character commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x4, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ steps.*.outputs.* }}, and ${{ runner.* }} expressions from run: blocks into env: blocks. Referenced them as plain environment variables ($VAR_NAME) in shell scripts. For list-type inputs (helmfile-args, helm-args, kube_version_arg), used the xargs-based tokenization pattern to preserve argument boundaries. For the Push to Github step, used built-in GITHUB_* environment variables for github context values.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing values to $GITHUB_OUTPUT in: YQ Platform settings (metadata loop), Resolve kube version step, and Select GitHub Token step (both branches).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Three fixes applied to hardened/action/action.yml:
1. (script-injection, line 108) Moved `inputs.cluster` from inline JavaScript string interpolation `run("${{ inputs.cluster }}")` to an env var `CLUSTER_URL: ${{ inputs.cluster }}` and updated the script to use `run(process.env.CLUSTER_URL)`, preventing attacker-controlled input from being injected as JavaScript code.
2. (script-injection, line 233) Quoted the bash array expansion from `for element in ${array[@]}` to `for element in "${array[@]}"` to prevent shell metacharacter injection from values_file input.
3. (github-env-injection, line 265) Added sanitization of the yq-derived WEBAPP_URL before writing to GITHUB_OUTPUT: `safe_webapp_url=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and updated the echo to use `${safe_webapp_url}` with a quoted `"$GITHUB_OUTPUT"` to prevent newline injection.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted ${VALUES_STR} expansion in the 'Helm raw render' step. Replaced the string-concatenation approach (VALUES_STR+="--values $element ") with a proper bash array (values_args+=(--values "$element")). Each element from INPUT_VALUES_FILE is now double-quoted when added to the array, and the array is expanded safely with "${values_args[@]}" in the helm template command. This prevents shell metacharacters in the caller-controlled inputs.values_file input from being interpreted by the shell.

