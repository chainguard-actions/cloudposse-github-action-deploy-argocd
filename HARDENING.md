<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.11.0** was hardened automatically. 26 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or overwritten. Failing references:
- `dcarbone/install-yq-action@v1.3.1` (Setup yq step)
- `mamezou-tech/setup-helmfile@v2.2.0` (Setup helmfile step)
- `actions/setup-node@v6` (Setup node step)
- `cloudposse/github-action-yaml-config-query@v1.0.1` (Config, Context, and Deplicated Ref steps)
- `nick-fields/retry@v4` (Push to Github step)
- `1arp/create-a-file-action@0.4` (Config render step)

Locations:

- `action.yml:116`
- `action.yml:122`
- `action.yml:129`
- `action.yml:155`
- `action.yml:196`
- `action.yml:248`
- `action.yml:278`
- `action.yml:310`
- `action.yml:340`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell commands. These expressions are expanded by the GitHub Actions template engine before the shell ever sees them, allowing an attacker who controls the input values to inject arbitrary shell commands. Violations include:

(a) 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

(a) 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — attacker-controlled inputs interpolated directly.

(a) 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — attacker-controlled input interpolated directly.

(a) 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.

(a) 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

(a) 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple attacker-controlled inputs interpolated directly.

(a) 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — attacker-controlled input interpolated directly.

(a) 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple attacker-controlled inputs interpolated directly.

(a) 'Get Webapp' step: `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.

(a) 'Push to Github' step: `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, and `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"` — inputs and github context interpolated directly.

(a) 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — attacker-controlled inputs interpolated directly.

Locations:

- `action.yml:136`
- `action.yml:163`
- `action.yml:175`
- `action.yml:185`
- `action.yml:200`
- `action.yml:210`
- `action.yml:222`
- `action.yml:232`
- `action.yml:248`
- `action.yml:270`
- `action.yml:285`
- `action.yml:300`
- `action.yml:320`
- `action.yml:345`
- `action.yml:360`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into these values to poison subsequent steps or exfiltrate data.

1. 'Resolve kube version' step writes `${{ inputs.kube-version }}` (and transitively `${{ steps.metadata.outputs.kube_version }}`, which itself comes from SSM data read via `${{ inputs.ssm-path }}`) to `$GITHUB_OUTPUT` via `echo "value=$kube_version" >> $GITHUB_OUTPUT` with no sanitization.

2. 'YQ Platform settings' (metadata) step writes arbitrary key=value pairs from `_metadata.yaml` (fetched via the attacker-controlled `${{ inputs.ssm-path }}` path) to `$GITHUB_OUTPUT` via `echo "${output}" >> $GITHUB_OUTPUT` with no sanitization.

3. 'Select GitHub Token for Sync Mode' step writes `${{ inputs.github-pat }}` and `${{ inputs.commit-status-github-token }}` directly to `$GITHUB_OUTPUT` via `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` with no sanitization.

Locations:

- `action.yml:200`
- `action.yml:185`
- `action.yml:345`

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

1. **unpinned-uses**: Pinned all 6 action references to full commit SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 occurrences)
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps: Install git-url-parse, Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Get Webapp, Push to Github, Select GitHub Token for Sync Mode. List-type inputs (helmfile-args, helm-args) are tokenized with xargs for proper argument splitting.

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before all writes to $GITHUB_OUTPUT in: YQ Platform settings (metadata loop), Resolve kube version, and Select GitHub Token for Sync Mode.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection in the 'Parse git URL for destination' step of action.yml. Moved `${{ inputs.cluster }}` from being directly interpolated into the JavaScript string literal `run("${{ inputs.cluster }}")` to an env variable `CLUSTER: ${{ inputs.cluster }}` in the step's env block. The script now reads the value safely via `run(process.env.CLUSTER)` instead of inline template substitution, preventing a malicious cluster value from breaking out of the string literal and executing arbitrary JavaScript.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get Webapp' step in action.yml (around line 214) by adding sanitization of WEBAPP_URL before writing to $GITHUB_OUTPUT. Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and changed the echo to use `${safe}` instead of `${WEBAPP_URL}`. This prevents newline injection attacks where user-controlled inputs (image, image-tag, namespace, application, etc.) could embed newlines in the rendered manifest, which yq would then extract and write unsanitized to $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two query lines in action.yml that had spurious backslashes before double quotes. Line 301: changed `query: .\"${{ steps.resolved_kube_version.outputs.value == '' }}\"` to `query: ."${{ steps.resolved_kube_version.outputs.value == '' }}"`. Line 422: changed `query: .\"${{ inputs.gitref-sha == '' }}\"` to `query: ."${{ inputs.gitref-sha == '' }}"`.

