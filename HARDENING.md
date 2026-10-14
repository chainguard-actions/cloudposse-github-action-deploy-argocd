<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions inside shell commands (rule a), allowing script injection. Affected steps and offending lines:

1. 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly into shell.

2. 'Parse git URL for destination' step (github-script): `run("${{ inputs.cluster }}")` — attacker-controlled input interpolated directly into JavaScript executed by github-script.

3. 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — inputs interpolated directly into shell command.

4. 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — input interpolated directly into shell command.

5. 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly into shell.

6. 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly into shell.

7. 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly into shell.

8. 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — input interpolated directly into shell.

9. 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly into shell.

10. 'Get Webapp' step: `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly into shell.

11. 'Push to Github' step (nick-fields/retry command:): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `cp -r ./tmp/* ./${{ steps.destination_dir.outputs.name }}/`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple inputs and github context values interpolated directly into shell.

12. 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly into shell condition.

Locations:

- `action.yml:124`
- `action.yml:125`
- `action.yml:148`
- `action.yml:193`
- `action.yml:212`
- `action.yml:229`
- `action.yml:230`
- `action.yml:231`
- `action.yml:251`
- `action.yml:257`
- `action.yml:258`
- `action.yml:259`
- `action.yml:262`
- `action.yml:263`
- `action.yml:274`
- `action.yml:280`
- `action.yml:283`
- `action.yml:284`
- `action.yml:285`
- `action.yml:286`
- `action.yml:287`
- `action.yml:288`
- `action.yml:290`
- `action.yml:294`
- `action.yml:295`
- `action.yml:307`
- `action.yml:347`
- `action.yml:349`
- `action.yml:353`
- `action.yml:356`
- `action.yml:360`
- `action.yml:372`
- `action.yml:373`
- `action.yml:384`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'):

1. 'Resolve kube version' step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` where $kube_version was set directly from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` without sanitization. An attacker-controlled newline in the input can inject arbitrary key=value pairs into GITHUB_OUTPUT.

2. 'YQ Platform settings' (metadata) step: `echo "${output}" >> $GITHUB_OUTPUT` where ${output} is derived from yq parsing of SSM-sourced data (externally controlled). No sanitization is applied before the write.

3. 'Select GitHub Token for Sync Mode' step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — sensitive token inputs written directly to GITHUB_OUTPUT without sanitization. A newline in the token value would inject additional output variables.

Locations:

- `action.yml:221`
- `action.yml:233`
- `action.yml:385`
- `action.yml:387`

### unpinned-uses (severity: high)

Multiple action references in action.yml use mutable tags or version strings instead of immutable full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

- `dcarbone/install-yq-action@v1.3.1` (tag)
- `mamezou-tech/setup-helmfile@v2.2.0` (tag)
- `actions/setup-node@v6` (tag)
- `cloudposse/github-action-yaml-config-query@v1.0.1` (tag, used 3 times)
- `actions/checkout@v6` (tag)
- `1arp/create-a-file-action@0.4` (tag)
- `nick-fields/retry@v4` (tag)
- `cloudposse/github-action-wait-commit-status@v0.2.1` (tag)

Note: `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned to a full SHA.

Locations:

- `action.yml:105`
- `action.yml:111`
- `action.yml:118`
- `action.yml:157`
- `action.yml:164`
- `action.yml:239`
- `action.yml:314`
- `action.yml:324`
- `action.yml:334`
- `action.yml:390`

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

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks across all affected steps: Install git-url-parse ($RUNNER_TEMP), Parse git URL (INPUT_CLUSTER via process.env), Read platform context (INPUT_SSM_PATH/INPUT_ENVIRONMENT), Read platform metadata (INPUT_SSM_PATH), Resolve kube version (INPUT_KUBE_VERSION/INPUT_SSM_PATH/METADATA_KUBE_VERSION), Ensure argocd repo structure (CONFIG_TMP), Helmfile render (all inputs), Build Helm Dependencies (INPUT_PATH), Helm raw render (all inputs), Get Webapp (CONFIG_TMP), Push to Github (DESTINATION_DIR_NAME/DESTINATION_REF/INPUT_OPERATION/CONFIG_PATH/GH_REPOSITORY/GH_SHA/GH_RUN_ID/GH_RUN_ATTEMPT), Select GitHub Token (INPUT_COMMIT_STATUS_GITHUB_TOKEN/INPUT_GITHUB_PAT).

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to GITHUB_OUTPUT in Resolve kube version and Select GitHub Token steps.

4. For list inputs (helmfile-args, helm-args, values_file), used proper xargs-based tokenization with guarded arrays to preserve argument boundaries while preventing injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'YQ Platform settings' step (id: metadata, ~line 219): Added sanitization of each `${output}` value using `safe_output="$(printf '%s' "${output}" | tr -d '\n\r')"` before writing to $GITHUB_OUTPUT. SSM-sourced metadata values could contain embedded newlines that would inject additional key=value pairs.
2. 'Get Webapp' step (~line 280): Added sanitization of `${WEBAPP_URL}` using `safe_webapp_url="$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')"` before writing to $GITHUB_OUTPUT. The URL is derived from rendered Kubernetes manifests incorporating user-controlled inputs (image name/tag) and could contain embedded newlines.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed three issues in action.yml: (1) Removed erroneous backslash-escaping of double quotes in the yq query at line 296 (steps.resolved_kube_version.outputs.value == '' check); (2) Fixed values_file tokenization at line 366 to split on commas AND whitespace by substituting commas with spaces before passing to xargs (${INPUT_VALUES_FILE//,/ }), matching upstream's behavior; (3) Removed erroneous backslash-escaping of double quotes in the yq query at line 413 (inputs.gitref-sha == '' check).

