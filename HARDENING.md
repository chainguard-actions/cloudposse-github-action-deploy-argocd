<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags or version strings instead of full 40-character commit SHAs. A supply-chain attacker who compromises the upstream action repository can push malicious code under the same tag. Unpinned references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v4`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`. (Only `actions/github-script@f28e40c7f34bde8b3046d885e986cb6290c5673b` is correctly pinned.)

Locations:

- `action.yml:96`
- `action.yml:102`
- `action.yml:109`
- `action.yml:133`
- `action.yml:148`
- `action.yml:172`
- `action.yml:228`
- `action.yml:237`
- `action.yml:271`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands, enabling script injection. An attacker-controlled input value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) would be executed by the shell before quoting can protect it.

(a) 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

(a) 'Parse git URL for destination' step (github-script `script:` block): `run("${{ inputs.cluster }}")` — attacker-controlled `inputs.cluster` interpolated directly into the JS script string.

(a) 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs interpolated directly into shell.

(a) 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

(a) 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/...` — multiple inputs interpolated directly.

(a) 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — input interpolated directly.

(a) 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }}` — many inputs interpolated directly.

(a) 'Get Webapp' step: `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.

(a) 'Push to Github' step: `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple github context values and step outputs interpolated directly.

(a) 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — sensitive inputs interpolated directly into shell.

Locations:

- `action.yml:113`
- `action.yml:121`
- `action.yml:163`
- `action.yml:175`
- `action.yml:178`
- `action.yml:192`
- `action.yml:196`
- `action.yml:215`
- `action.yml:237`
- `action.yml:261`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in the value can inject arbitrary key=value pairs into the output file, allowing an attacker to override subsequent step outputs.

(1) 'Resolve kube version' step: `kube_version` is set from `${{ inputs.kube-version }}` (direct interpolation) and then written with `echo "value=$kube_version" >> $GITHUB_OUTPUT` — no sanitization.

(2) 'YQ Platform settings' (metadata) step: `echo "${output}" >> $GITHUB_OUTPUT` inside a loop over yq-parsed external YAML data — the values come from SSM/external sources and are not sanitized before being written.

(3) 'Select GitHub Token for Sync Mode' step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — sensitive PAT/token inputs are interpolated directly and written to GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:163`
- `action.yml:160`
- `action.yml:261`

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

1. **unpinned-uses**: Pinned all 9 unpinned action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ runner.temp }}, ${{ steps.*.outputs.* }}, and ${{ github.* }} expressions out of run: blocks into step env: maps. Shell scripts now reference plain environment variables. The github-script step uses process.env.INPUT_CLUSTER instead of interpolating ${{ inputs.cluster }} into the JS string.

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before all writes to $GITHUB_OUTPUT in: (a) the YQ Platform settings/metadata step, (b) the Resolve kube version step, and (c) the Select GitHub Token for Sync Mode step.

4. **List-type args** (helmfile-args, helm-args): Used xargs-based array tokenization pattern to safely split whitespace-separated argument lists while preserving quoting semantics.

5. **values_file**: Used IFS-based array split (comma/space separated) for the values file list, building --values flags safely.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get Webapp' step (line 270) in hardened/action/action.yml. The WEBAPP_URL extracted from the rendered Kubernetes manifest (which could contain attacker-controlled content) is now sanitized before being written to $GITHUB_OUTPUT. Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and changed the echo to use `${safe}` instead of `${WEBAPP_URL}`. Also properly quoted `"$GITHUB_OUTPUT"` as a best practice.

