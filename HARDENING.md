<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.1** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell command strings (rule a), allowing script injection. Affected steps and offending lines:

- 'Install git-url-parse' (line ~154): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`
- 'Read platform context' (line ~243): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...`
- 'Read platform metadata' (line ~263): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`
- 'Resolve kube version' (line ~280): `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"`
- 'Ensure argocd repo structure' (line ~300): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`
- 'Helmfile render' (line ~306): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}`
- 'Build Helm Dependencies' (line ~320): `helm dependency build ${{ inputs.path }}`
- 'Helm raw render' (line ~327): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and multiple `${{ inputs.* }}` in helm template command
- 'Get Webapp' (line ~352): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` in yq command
- 'Push to Github' command: (line ~415): `case '${{ inputs.operation }}' in`, `pushd ./${{ steps.destination_dir.outputs.name }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`
- 'Select GitHub Token for Sync Mode' (line ~443): `if [ -z "${{ inputs.commit-status-github-token }}" ]`

All of these allow an attacker-controlled value to be interpreted as shell syntax before the shell ever sees it.

Locations:

- `action.yml:154`
- `action.yml:243`
- `action.yml:263`
- `action.yml:280`
- `action.yml:300`
- `action.yml:306`
- `action.yml:320`
- `action.yml:327`
- `action.yml:352`
- `action.yml:415`
- `action.yml:443`

### github-env-injection (severity: high)

Multiple run: blocks write unsanitized values derived from untrusted inputs to $GITHUB_OUTPUT without applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. 'YQ Platform settings' (metadata step, line ~272): `echo "${output}" >> $GITHUB_OUTPUT` where `${output}` is derived from yq parsing of SSM data — an externally-controlled source. No sanitization applied.

2. 'Resolve kube version' (line ~284): `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is set from `${{ inputs.kube-version }}` (direct input) or `${{ steps.metadata.outputs.kube_version }}` (step output). No sanitization applied.

3. 'Select GitHub Token for Sync Mode' (line ~445): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — direct interpolation of input values into $GITHUB_OUTPUT without sanitization. A newline in the token value could inject arbitrary key=value pairs into the output file.

Locations:

- `action.yml:272`
- `action.yml:284`
- `action.yml:445`

### unpinned-uses (severity: high)

The following uses: references in action.yml use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

- `uses: dcarbone/install-yq-action@v1.3.1` (line ~132)
- `uses: mamezou-tech/setup-helmfile@v2.2.0` (line ~139)
- `uses: actions/setup-node@v6` (line ~147)
- `uses: cloudposse/github-action-yaml-config-query@v1.0.1` (lines ~193, ~292, ~367)
- `uses: 1arp/create-a-file-action@0.4` (line ~378)
- `uses: nick-fields/retry@v4` (line ~388)

Pinned references (PASS): `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3`, `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1`, `cloudposse/github-action-wait-commit-status@968fc17eddf4d9f4f2203edb4c72814a1d8bc650`.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:292`
- `action.yml:367`
- `action.yml:378`
- `action.yml:388`

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

1. **unpinned-uses**: Pinned all 6 unpinned action references to full 40-char commit SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 occurrences)
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: shell strings into env: blocks across all affected steps: Install git-url-parse (runner.temp), Read platform context (ssm-path, environment), Read platform metadata (ssm-path), Resolve kube version (kube-version, ssm-path, metadata.kube_version), Ensure argocd repo structure (config.outputs.tmp), Helmfile render (namespace, environment, path, helmfile-args, kube_version, config.outputs.tmp), Build Helm Dependencies (path), Helm raw render (values_file, application, path, image, image-tag, environment, namespace, helm-args, kube_version, config.outputs.tmp), Get Webapp (config.outputs.tmp), Push to Github (operation, destination_dir.outputs.name, destination.outputs.ref, config.outputs.path, github.repository, github.sha, github.run_id, github.run_attempt), Select GitHub Token (commit-status-github-token, github-pat).

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before all $GITHUB_OUTPUT writes: YQ Platform settings metadata loop, Resolve kube version, Select GitHub Token for Sync Mode.

4. List-type inputs (helmfile-args, helm-args) are tokenized with the xargs/while-read-NUL pattern to preserve argument boundaries without injection risk.

5. Fixed a duplicate env: block in the Install git-url-parse step (merged NPM_CONFIG_USERCONFIG and RUNNER_TEMP into a single env: block).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. script-injection (line 133, 'Parse git URL for destination'): Moved `${{ inputs.cluster }}` from inline JavaScript string interpolation into the step's `env:` block as `INPUT_CLUSTER`, then changed `run("${{ inputs.cluster }}")` to `run(process.env.INPUT_CLUSTER)` to prevent arbitrary JavaScript injection.

2. script-injection (lines 262, 285, 'Helm raw render'): (a) Fixed unquoted array expansion `for element in ${array[@]}` → `for element in "${array[@]}"`. (b) Replaced the `VALUES_STR` string variable (expanded unquoted in the helm command) with a proper bash array `values_args` that accumulates `--values "$element"` pairs, then expanded safely as `"${values_args[@]}"` in the helm template invocation.

3. github-env-injection (line 310, 'Get Webapp'): Added newline sanitization before writing to GITHUB_OUTPUT: `safe=$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')` and then `echo "webapp_url=${safe}" >> "$GITHUB_OUTPUT"`.

