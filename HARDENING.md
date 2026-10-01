<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.11.0** was hardened automatically. 26 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved. Failing references:
- `dcarbone/install-yq-action@v1.3.1`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v6`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used in 'Config', 'Context', and 'Deplicated Ref' steps)
- `1arp/create-a-file-action@0.4`
- `nick-fields/retry@v4`

Locations:

- `action.yml:130`
- `action.yml:136`
- `action.yml:143`
- `action.yml:163`
- `action.yml:200`
- `action.yml:232`
- `action.yml:248`
- `action.yml:263`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a). This allows an attacker who controls the inputs or referenced context values to inject arbitrary shell commands. Affected steps and offending expressions:

1. **Install git-url-parse** (`run:`): `${{ runner.temp }}` used unquoted in `mkdir -p ${{ runner.temp }}/action-deps`.

2. **Read platform context** (`run:`): `${{ inputs.ssm-path }}` and `${{ inputs.environment }}` interpolated directly into `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}`.

3. **Read platform metadata** (`run:`): `${{ inputs.ssm-path }}` interpolated directly into `chamber --verbose export ${{ inputs.ssm-path }}/_metadata`.

4. **Resolve kube version** (`run:`): `${{ inputs.kube-version }}`, `${{ inputs.ssm-path }}`, and `${{ steps.metadata.outputs.kube_version }}` interpolated directly into shell variable assignments and conditionals.

5. **Ensure argocd repo structure** (`run:`): `${{ steps.config.outputs.tmp }}` interpolated directly into `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`.

6. **Helmfile render** (`run:`): `${{ inputs.namespace }}`, `${{ inputs.environment }}`, `${{ inputs.path }}`, `${{ steps.arguments.outputs.kube_version }}`, `${{ inputs.helmfile-args }}`, and `${{ steps.config.outputs.tmp }}` all interpolated directly into the helmfile command.

7. **Build Helm Dependencies** (`run:`): `${{ inputs.path }}` interpolated directly into `helm dependency build ${{ inputs.path }}`.

8. **Helm raw render** (`run:`): `${{ inputs.values_file }}`, `${{ inputs.application }}`, `${{ inputs.path }}`, `${{ inputs.image }}`, `${{ inputs.image-tag }}`, `${{ inputs.environment }}`, `${{ inputs.namespace }}`, `${{ inputs.helm-args }}`, `${{ steps.arguments.outputs.kube_version }}`, and `${{ steps.config.outputs.tmp }}` all interpolated directly into the helm template command.

9. **Get Webapp** (`run:`): `${{ steps.config.outputs.tmp }}` interpolated directly into the yq command.

10. **Push to Github** (`command:` passed to nick-fields/retry): `${{ steps.destination_dir.outputs.name }}`, `${{ steps.destination.outputs.ref }}`, `${{ inputs.operation }}`, `${{ steps.config.outputs.path }}`, `${{ github.repository }}`, `${{ github.sha }}`, `${{ github.run_id }}`, and `${{ github.run_attempt }}` all interpolated directly into shell commands.

11. **Select GitHub Token for Sync Mode** (`run:`): `${{ inputs.commit-status-github-token }}` and `${{ inputs.github-pat }}` interpolated directly into shell conditionals and echo commands.

Locations:

- `action.yml:148`
- `action.yml:175`
- `action.yml:188`
- `action.yml:196`
- `action.yml:210`
- `action.yml:218`
- `action.yml:228`
- `action.yml:237`
- `action.yml:244`
- `action.yml:275`
- `action.yml:310`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs or external data to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows newline injection that can poison subsequent steps' outputs or environment.

1. **YQ Platform settings** (id: metadata): The loop `for output in $(yq ... ./_metadata.yaml); do echo "${output}" >> $GITHUB_OUTPUT; done` writes raw yq-parsed values from an external YAML file directly to `$GITHUB_OUTPUT` without sanitization. The content of `_metadata.yaml` is fetched from SSM and could contain newlines.

2. **Resolve kube version**: `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is set from `${{ inputs.kube-version }}` (direct input interpolation) or `${{ steps.metadata.outputs.kube_version }}` (step output from external data) — no sanitization applied.

3. **Select GitHub Token for Sync Mode**: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` write raw input values (PAT tokens) directly to `$GITHUB_OUTPUT` without sanitization.

4. **Get Webapp**: `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` where `WEBAPP_URL` is parsed from rendered Kubernetes manifests via yq — external data written to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `action.yml:196`
- `action.yml:205`
- `action.yml:244`
- `action.yml:310`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ssm-path }}" appears directly in run: block of step "Read platform context"; move to env: map

Locations:

- `action.yml:242`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.environment }}" appears directly in run: block of step "Read platform context"; move to env: map

Locations:

- `action.yml:242`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ssm-path }}" appears directly in run: block of step "Read platform metadata"; move to env: map

Locations:

- `action.yml:261`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.kube-version }}" appears directly in run: block of step "Resolve kube version"; move to env: map

Locations:

- `action.yml:277`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.ssm-path }}" appears directly in run: block of step "Resolve kube version"; move to env: map

Locations:

- `action.yml:278`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.namespace }}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:306`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.environment }}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:307`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path}}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:308`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.helmfile-args }}" appears directly in run: block of step "Helmfile render"; move to env: map

Locations:

- `action.yml:312`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Build Helm Dependencies"; move to env: map

Locations:

- `action.yml:321`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.values_file }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:327`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.application }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:330`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:330`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:331`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:332`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image-tag }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:333`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image-tag }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:334`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.environment }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:335`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.namespace }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:337`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.helm-args }}" appears directly in run: block of step "Helm raw render"; move to env: map

Locations:

- `action.yml:341`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.commit-status-github-token }}" appears directly in run: block of step "Select GitHub Token for Sync Mode"; move to env: map

Locations:

- `action.yml:447`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.github-pat }}" appears directly in run: block of step "Select GitHub Token for Sync Mode"; move to env: map

Locations:

- `action.yml:448`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.commit-status-github-token }}" appears directly in run: block of step "Select GitHub Token for Sync Mode"; move to env: map

Locations:

- `action.yml:450`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 6 unpinned action references to their full commit SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 occurrences)
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps: Install git-url-parse (runner.temp → $RUNNER_TEMP), Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Get Webapp, Push to Github, Select GitHub Token for Sync Mode.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all GITHUB_OUTPUT writes in: YQ Platform settings (metadata loop), Resolve kube version, Get Webapp, and Select GitHub Token for Sync Mode.

4. **helmfile-args and helm-args**: These list-style inputs are properly tokenized using the xargs/while-read-NUL pattern to preserve argument boundaries while preventing injection.

