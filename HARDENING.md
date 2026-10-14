<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands via crafted input values.

**Install git-url-parse step (line ~118):** `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

**Parse git URL step (line ~148):** `run("${{ inputs.cluster }}")` — `inputs.cluster` interpolated directly into a JavaScript string inside the `script:` block.

**Read platform context step (line ~186):** `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — two inputs interpolated directly into a shell command.

**Read platform metadata step (line ~201):** `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — input interpolated directly into a shell command.

**Resolve kube version step (lines ~215–218):** `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"` — multiple expressions interpolated directly in shell.

**Ensure argocd repo structure step (line ~234):** `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly in shell.

**Helmfile render step (lines ~238–243):** `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }}` — five expressions interpolated directly in shell.

**Build Helm Dependencies step (line ~249):** `helm dependency build ${{ inputs.path }}` — input interpolated directly in shell.

**Helm raw render step (lines ~254–268):** `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} --set image.tag=${{ inputs.image-tag }} --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }}` — eight+ expressions interpolated directly in shell.

**Get Webapp step (line ~280):** `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly in shell.

**Push to Github step (lines ~322–345):** `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `cp -r ./tmp/* ./${{ steps.destination_dir.outputs.name }}/`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple github context and step output expressions interpolated directly in shell.

**Select GitHub Token step (lines ~352–355):** `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly in shell condition.

All of these must be moved to `env:` variables and referenced as `"$VAR"` in the shell script.

Locations:

- `action.yml:118`
- `action.yml:148`
- `action.yml:186`
- `action.yml:201`
- `action.yml:215`
- `action.yml:234`
- `action.yml:238`
- `action.yml:249`
- `action.yml:254`
- `action.yml:280`
- `action.yml:322`
- `action.yml:344`
- `action.yml:352`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`).

**YQ Platform settings/metadata step (line ~209):** `echo "${output}" >> $GITHUB_OUTPUT` — `${output}` is populated by `yq` processing of externally-sourced YAML data (`_metadata.yaml` from SSM/chamber). The value is not sanitized before being written to GITHUB_OUTPUT, allowing newline injection to add arbitrary output variables.

**Resolve kube version step (line ~220):** `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set from `${{ inputs.kube-version }}` (a direct input) or `${{ steps.metadata.outputs.kube_version }}` (a step output from external data). Neither source is sanitized before the write.

**Select GitHub Token step (lines ~353–355):** `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both write `inputs.*` values directly to GITHUB_OUTPUT without sanitization, allowing newline injection to set arbitrary output variables or environment variables in subsequent steps.

Locations:

- `action.yml:209`
- `action.yml:220`
- `action.yml:353`
- `action.yml:355`

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable version tags instead of immutable full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised.

Failing references:
- `uses: dcarbone/install-yq-action@v1.1.0` (line ~100) — tag, not SHA
- `uses: mamezou-tech/setup-helmfile@v2.2.0` (line ~106) — tag, not SHA
- `uses: actions/setup-node@v6` (line ~113) — tag, not SHA
- `uses: cloudposse/github-action-yaml-config-query@v1.0.1` (line ~153, ~226, ~290) — tag, not SHA (used 3 times)
- `uses: actions/checkout@v6` (line ~163) — tag, not SHA
- `uses: 1arp/create-a-file-action@0.2` (line ~298) — tag, not SHA
- `uses: nick-fields/retry@v4` (line ~308) — tag, not SHA
- `uses: cloudposse/github-action-wait-commit-status@v0.2.1` (line ~358) — tag, not SHA

Only `uses: actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` (line ~130) is correctly pinned to a full SHA digest.

Locations:

- `action.yml:100`
- `action.yml:106`
- `action.yml:113`
- `action.yml:153`
- `action.yml:163`
- `action.yml:226`
- `action.yml:290`
- `action.yml:298`
- `action.yml:308`
- `action.yml:358`

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
   - dcarbone/install-yq-action@v1.1.0 → @4d2a79586f70993921358b3d28c596fec4e92ed2
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 occurrences)
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - 1arp/create-a-file-action@0.2 → @ee80edaa4741d9997dbe053e052329d4070c020c
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60
   - cloudposse/github-action-wait-commit-status@v0.2.1 → @2ad2cbd1f6b2a7a7dd1bcc314a6b48f474951202

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: variables. Affected steps: Install git-url-parse (runner.temp → $RUNNER_TEMP), Parse git URL (inputs.cluster → INPUT_CLUSTER env var + process.env.INPUT_CLUSTER in JS), Read platform context (ssm-path, environment), Read platform metadata (ssm-path), Resolve kube version (kube-version, ssm-path, metadata output), Ensure argocd repo structure (config.outputs.tmp), Helmfile render (namespace, environment, path, helmfile-args, kube_version, config.outputs.tmp), Build Helm Dependencies (path), Helm raw render (values_file, application, path, image, image-tag, environment, namespace, helm-args, kube_version, config.outputs.tmp), Get Webapp (config.outputs.tmp), Push to Github (destination_dir, destination.ref, operation, config.path, github.repository, github.sha, github.run_id, github.run_attempt), Select GitHub Token (commit-status-github-token, github-pat).

3. **github-env-injection**: Added sanitization (printf '%s' ... | tr -d '\n\r') before writing to $GITHUB_OUTPUT in: YQ Platform settings/metadata step (for loop output), Resolve kube version step (kube_version), Select GitHub Token step (both token branches).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. **Helmfile render (line 228) - script-injection**: Replaced unquoted `$INPUT_HELMFILE_ARGS` expansion with xargs-based tokenization into a bash array `helmfile_extra_args`, expanded safely as `"${helmfile_extra_args[@]}"`.

2. **Helm raw render (lines 255/264/265/266) - script-injection**: (a) Fixed `for element in ${array[@]}` to `for element in "${array[@]}"`. (b) Replaced the string-concatenation `VALUES_STR` approach with a proper bash array `values_args` built with `values_args+=(--values "$element")`, expanded as `"${values_args[@]}"`.(c) Replaced unquoted `$INPUT_HELM_ARGS` with xargs-based tokenization into `helm_extra_args` array. (d) Replaced unquoted `$KUBE_VERSION_ARG` with `${KUBE_VERSION_ARG:+"$KUBE_VERSION_ARG"}` (it's a single optional flag token).

3. **Get Webapp (line 280) - github-env-injection**: Captured yq output into `raw` variable first, then sanitized with `printf '%s' "$raw" | tr -d '\n\r'` before assigning to `WEBAPP_URL` and writing to `$GITHUB_OUTPUT`.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed two yq query strings in action.yml that had backslashes incorrectly added before double quotes. At line 296, changed `query: .\"${{ steps.resolved_kube_version.outputs.value == '' }}\"` to `query: ."${{ steps.resolved_kube_version.outputs.value == '' }}"`. At line 409, changed `query: .\"${{ inputs.gitref-sha == '' }}\"` to `query: ."${{ inputs.gitref-sha == '' }}"`.

