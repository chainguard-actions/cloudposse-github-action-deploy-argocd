<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions (rule a), allowing script injection. Affected steps and offending lines:

1. **Install git-url-parse** (run:): `mkdir -p ${{ runner.temp }}/action-deps` — expression interpolated directly in shell command.

2. **Read platform context** (run:): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml` — attacker-controlled inputs interpolated directly into shell command.

3. **Read platform metadata** (run:): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata --format yaml` — attacker-controlled input interpolated directly.

4. **Resolve kube version** (run:): `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.

5. **Ensure argocd repo structure** (run:): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

6. **Helmfile render** (run:): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — multiple inputs interpolated directly.

7. **Build Helm Dependencies** (run:): `helm dependency build ${{ inputs.path }}` — input interpolated directly.

8. **Helm raw render** (run:): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — many inputs interpolated directly.

9. **Get Webapp** (run:): `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.

10. **Push to Github** (command:): `pushd ./${{ steps.destination_dir.outputs.name }}`, `origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple inputs and github context values interpolated directly into shell commands.

11. **Select GitHub Token for Sync Mode** (run:): `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly.

Locations:

- `action.yml:123`
- `action.yml:179`
- `action.yml:192`
- `action.yml:202`
- `action.yml:218`
- `action.yml:223`
- `action.yml:234`
- `action.yml:240`
- `action.yml:258`
- `action.yml:295`
- `action.yml:319`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs or step outputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. **YQ Platform settings / metadata step**: `echo "${output}" >> $GITHUB_OUTPUT` — `${output}` is derived from yq parsing of SSM/platform data (external data source), written without sanitization.

2. **Resolve kube version**: `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set from `${{ inputs.kube-version }}` (attacker-controlled input) and `${{ steps.metadata.outputs.kube_version }}` (step output), written without sanitization.

3. **Get Webapp**: `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` — `${WEBAPP_URL}` is derived from yq parsing of rendered manifest files (which themselves incorporate attacker-controlled inputs), written without sanitization.

4. **Select GitHub Token for Sync Mode**: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — attacker-controlled inputs written directly to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `action.yml:199`
- `action.yml:207`
- `action.yml:260`
- `action.yml:321`
- `action.yml:323`

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:

- `uses: dcarbone/install-yq-action@v1.3.1` (tag)
- `uses: mamezou-tech/setup-helmfile@v2.2.0` (tag)
- `uses: actions/setup-node@v6` (tag)
- `uses: cloudposse/github-action-yaml-config-query@v1.0.1` (tag, used 3 times)
- `uses: actions/checkout@v6` (tag)
- `uses: 1arp/create-a-file-action@0.4` (tag)
- `uses: nick-fields/retry@v4` (tag)
- `uses: cloudposse/github-action-wait-commit-status@v0.2.1` (tag)

Only `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned to a SHA.

Locations:

- `action.yml:104`
- `action.yml:110`
- `action.yml:117`
- `action.yml:150`
- `action.yml:156`
- `action.yml:210`
- `action.yml:265`
- `action.yml:272`
- `action.yml:282`
- `action.yml:329`

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

1. **unpinned-uses**: Pinned all 8 action references to full commit SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 uses)
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60
   - cloudposse/github-action-wait-commit-status@v0.2.1 → @2ad2cbd1f6b2a7a7dd1bcc314a6b48f474951202

2. **script-injection/static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps: Install git-url-parse (uses $RUNNER_TEMP env var), Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Get Webapp, Push to Github, Select GitHub Token for Sync Mode.

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before all $GITHUB_OUTPUT writes in: YQ Platform settings/metadata step, Resolve kube version, Get Webapp, Select GitHub Token for Sync Mode.

4. **helmfile-args and helm-args** (list inputs) are properly tokenized using xargs+read loop into bash arrays to preserve argument boundaries.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yml:

1. 'Parse git URL for destination' step (line ~158): Moved `inputs.cluster` from direct interpolation in the JavaScript `run("${{ inputs.cluster }}")` call to an env var `INPUT_CLUSTER: ${{ inputs.cluster }}`, then referenced it safely as `run(process.env.INPUT_CLUSTER)` to prevent JavaScript injection.

2. 'Helm raw render' step (line ~270): Replaced the unquoted `${VALUES_STR}` string expansion (which allowed shell metacharacter injection via `inputs.values_file`) with a properly quoted bash array `values_arr`. Each `--values` flag and its file argument are stored as separate array elements via `values_arr+=(--values "$element")`, then expanded safely with `"${values_arr[@]}"`.

