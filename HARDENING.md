<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.1** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions inside shell commands (rule a), allowing shell metacharacter injection before the shell ever quotes the value.

- 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`
- 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...`
- 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`
- 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"`
- 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`
- 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}`
- 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}`
- 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}`
- 'Get Webapp' step: `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` in yq command
- 'Push to Github' command block: `case '${{ inputs.operation }}' in`, `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`
- 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]`

Locations:

- `action.yml:130`
- `action.yml:174`
- `action.yml:190`
- `action.yml:203`
- `action.yml:222`
- `action.yml:227`
- `action.yml:240`
- `action.yml:246`
- `action.yml:269`
- `action.yml:315`
- `action.yml:330`
- `action.yml:344`

### github-env-injection (severity: high)

Multiple run: blocks write unsanitized values to $GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization step:

1. 'YQ Platform settings' (metadata step, line ~196): `echo "${output}" >> $GITHUB_OUTPUT` — `${output}` is derived from yq parsing of SSM-sourced YAML data, which is externally controlled. No sanitization applied.

2. 'Resolve kube version' (line ~207): `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set directly from `${{ inputs.kube-version }}` (user-supplied input) or `${{ steps.metadata.outputs.kube_version }}` (SSM-sourced). No sanitization applied.

3. 'Get Webapp' (line ~270): `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` — `${WEBAPP_URL}` is derived from yq parsing of rendered Helm/Helmfile manifests whose content is influenced by user-supplied inputs. No sanitization applied.

4. 'Select GitHub Token for Sync Mode' (lines ~345-347): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — directly writes user-supplied input values to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:196`
- `action.yml:207`
- `action.yml:270`
- `action.yml:345`

### unpinned-uses (severity: high)

The following uses: references in action.yml use mutable tags or version strings instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

- `uses: dcarbone/install-yq-action@v1.3.1` (line ~113)
- `uses: mamezou-tech/setup-helmfile@v2.2.0` (line ~118)
- `uses: actions/setup-node@v6` (line ~124)
- `uses: cloudposse/github-action-yaml-config-query@v1.0.1` (lines ~148, ~213, ~278)
- `uses: 1arp/create-a-file-action@0.4` (line ~286)
- `uses: nick-fields/retry@v4` (line ~297)

Note: `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1`, `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3`, and `cloudposse/github-action-wait-commit-status@968fc17eddf4d9f4f2203edb4c72814a1d8bc650` are correctly pinned to full SHAs.

Locations:

- `action.yml:113`
- `action.yml:118`
- `action.yml:124`
- `action.yml:148`
- `action.yml:213`
- `action.yml:278`
- `action.yml:286`
- `action.yml:297`

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

1. UNPINNED-USES: Pinned all 6 unpinned action references to full commit SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (4 occurrences)
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60

2. SCRIPT-INJECTION / STATIC-INLINE-INJECTION: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps (Install git-url-parse, Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Get Webapp, Push to Github, Select GitHub Token for Sync Mode). Used xargs-based tokenization for list inputs (helmfile-args, helm-args, values_file).

3. GITHUB-ENV-INJECTION: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all writes to $GITHUB_OUTPUT in: YQ Platform settings (metadata), Resolve kube version, Get Webapp, and Select GitHub Token for Sync Mode steps.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed three issues in hardened/action/action.yml:
1. Line 296: Removed backslash-escaped quotes from yq query for resolved_kube_version step - changed `.\"${{ ... }}\"` to `."${{ ... }}"`
2. Line 371: Fixed values_file input splitting to handle commas AND whitespace (matching upstream behavior) by substituting commas with spaces before xargs: `"${INPUT_VALUES_FILE//,/ }"` instead of `"$INPUT_VALUES_FILE"`
3. Line 421: Removed backslash-escaped quotes from yq query for gitref-sha step - changed `.\"${{ ... }}\"` to `."${{ ... }}"`

