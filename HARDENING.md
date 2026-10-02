<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.11.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags/versions rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or overwritten. Failing references: `dcarbone/install-yq-action@v1.3.1`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times), `1arp/create-a-file-action@0.4`, `nick-fields/retry@v4`.

Locations:

- `action.yml:118`
- `action.yml:123`
- `action.yml:129`
- `action.yml:157`
- `action.yml:232`
- `action.yml:291`
- `action.yml:299`
- `action.yml:314`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions (rule a), allowing an attacker-controlled value to be injected into the shell command string before the shell ever parses it. Affected steps and offending lines include:
- 'Install git-url-parse': `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`
- 'Parse git URL for destination' (github-script): `run("${{ inputs.cluster }}")` — inputs.cluster interpolated directly into JS
- 'Read platform context': `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...`
- 'Read platform metadata': `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`
- 'Resolve kube version': `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"`
- 'Ensure argocd repo structure': `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`
- 'Helmfile render': `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}`
- 'Build Helm Dependencies': `helm dependency build ${{ inputs.path }}`
- 'Helm raw render': `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ...` and multiple other inputs
- 'Get Webapp': `${{ steps.config.outputs.tmp }}/manifests/resources.yaml`
- 'Push to Github': `pushd ./${{ steps.destination_dir.outputs.name }}`, `origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}'`, `${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`
- 'Select GitHub Token for Sync Mode': `if [ -z "${{ inputs.commit-status-github-token }}" ]`

Locations:

- `action.yml:134`
- `action.yml:148`
- `action.yml:200`
- `action.yml:210`
- `action.yml:222`
- `action.yml:240`
- `action.yml:245`
- `action.yml:256`
- `action.yml:261`
- `action.yml:281`
- `action.yml:322`
- `action.yml:341`

### github-env-injection (severity: high)

Multiple `run:` steps write unsanitized untrusted values to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization step:

1. 'YQ Platform settings' (metadata step, line ~217): writes raw yq-parsed key=value pairs from SSM (external data source) directly to $GITHUB_OUTPUT via `echo "${output}" >> $GITHUB_OUTPUT` — no newline sanitization.

2. 'Resolve kube version' (line ~227): assigns `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` to a shell variable then writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` — the inputs value is interpolated directly into the run: block and then forwarded to GITHUB_OUTPUT without sanitization.

3. 'Get Webapp' (line ~283): writes `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` where WEBAPP_URL is derived from yq parsing of a file path constructed from `${{ steps.config.outputs.tmp }}` (itself derived from inputs) — no sanitization.

4. 'Select GitHub Token for Sync Mode' (lines ~342-345): writes `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — direct interpolation of sensitive input values into GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:217`
- `action.yml:227`
- `action.yml:283`
- `action.yml:342`

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

1. **unpinned-uses**: Pinned all 8 action references to full 40-char commit SHAs: dcarbone/install-yq-action@v1.3.1, mamezou-tech/setup-helmfile@v2.2.0, actions/setup-node@v6, cloudposse/github-action-yaml-config-query@v1.0.1 (3x), 1arp/create-a-file-action@0.4, nick-fields/retry@v4.

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks. Key changes: runner.temp→RUNNER_TEMP, inputs.cluster→INPUT_CLUSTER (used via process.env in JS), inputs.ssm-path/environment→env vars for chamber commands, all helm/helmfile inputs→env vars. List-type inputs (helmfile-args, helm-args, values_file) are tokenized with xargs+while-read-NUL loop to preserve argument boundaries.

3. **github-env-injection**: All values written to $GITHUB_OUTPUT are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing: metadata outputs, kube_version, webapp_url, and github tokens.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Helm raw render' step in action.yml: replaced the unsafe string accumulator `VALUES_STR` (which concatenated `--values $element` without quoting and expanded unquoted as `${VALUES_STR}`) with a properly-quoted bash array `values_flags_arr`. The xargs tokenization loop now directly populates the array with `--values` and `"$t"` as separate elements, and the helm template command uses `"${values_flags_arr[@]}"` for safe expansion. No other findings were present.

