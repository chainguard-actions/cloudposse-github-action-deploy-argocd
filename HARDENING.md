<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-deploy-argocd/v1.10.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks. Unpinned references:
- `dcarbone/install-yq-action@v1.3.1` (line 101)
- `mamezou-tech/setup-helmfile@v2.2.0` (line 106)
- `actions/setup-node@v6` (line 113)
- `cloudposse/github-action-yaml-config-query@v1.0.1` (lines 148, 208, 261)
- `actions/checkout@v6` (line 152)
- `1arp/create-a-file-action@0.4` (line 268)
- `nick-fields/retry@v4` (line 278)
- `cloudposse/github-action-wait-commit-status@v0.2.1` (line 325)
Only `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned.

Locations:

- `action.yml:101`
- `action.yml:106`
- `action.yml:113`
- `action.yml:148`
- `action.yml:152`
- `action.yml:208`
- `action.yml:261`
- `action.yml:268`
- `action.yml:278`
- `action.yml:325`

### script-injection (severity: high)

Numerous `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands, enabling script injection. Any caller-controlled input can inject arbitrary shell commands. Specific violations (sub-rule a):

1. **Install git-url-parse** (line ~117): `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

2. **Parse git URL for destination** (line ~137): `run("${{ inputs.cluster }}")` — `inputs.cluster` interpolated directly into a JavaScript string inside a github-script block, allowing injection into the script.

3. **Read platform context** (line ~177): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — two untrusted inputs interpolated directly into shell command.

4. **Read platform metadata** (line ~190): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — untrusted input interpolated directly.

5. **Resolve kube version** (line ~201): `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — multiple untrusted expressions interpolated into shell.

6. **Helmfile render** (line ~219): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple unquoted inputs interpolated directly.

7. **Build Helm Dependencies** (line ~230): `helm dependency build ${{ inputs.path }}` — unquoted input interpolated directly.

8. **Helm raw render** (line ~235): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — many unquoted inputs interpolated directly.

9. **Get Webapp** (line ~256): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated into shell path.

10. **Push to Github / command** (lines ~295–322): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple untrusted expressions interpolated into shell.

11. **Select GitHub Token** (line ~331): `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly into shell condition.

Locations:

- `action.yml:117`
- `action.yml:137`
- `action.yml:177`
- `action.yml:190`
- `action.yml:201`
- `action.yml:219`
- `action.yml:230`
- `action.yml:235`
- `action.yml:256`
- `action.yml:295`
- `action.yml:331`

### github-env-injection (severity: high)

Multiple `run:` steps write untrusted input values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that can poison subsequent steps' environment or outputs.

1. **YQ Platform settings (metadata)** (line ~197): `echo "${output}" >> $GITHUB_OUTPUT` — `${output}` is derived from SSM/external YAML data parsed by yq. An attacker who controls SSM content (or a misconfigured SSM path) can inject newlines to add arbitrary key=value pairs to GITHUB_OUTPUT.

2. **Resolve kube version** (line ~205): `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set from `${{ inputs.kube-version }}` or `${{ steps.metadata.outputs.kube_version }}` (both untrusted), without sanitization before the write.

3. **Select GitHub Token for Sync Mode** (lines ~332–334): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — PAT/token inputs are written directly to GITHUB_OUTPUT without sanitization. A newline in the token value would allow injecting additional output variables.

Locations:

- `action.yml:197`
- `action.yml:205`
- `action.yml:332`

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

Fixed all security findings in action.yml:

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-char commit SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @48b55a011bda9f5d6aeb4c2d9c7362e8dae4041e
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 occurrences)
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60
   - cloudposse/github-action-wait-commit-status@v0.2.1 → @2ad2cbd1f6b2a7a7dd1bcc314a6b48f474951202

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks for all affected steps: Install git-url-parse, Parse git URL (also changed JS to use process.env.INPUT_CLUSTER), Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Get Webapp, Push to Github, Select GitHub Token for Sync Mode.

3. **github-env-injection**: Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT in: YQ Platform settings (metadata loop), Resolve kube version, and both branches of Select GitHub Token for Sync Mode.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings in action.yml:

1. Helmfile render (line 232): Replaced bare unquoted `$INPUT_HELMFILE_ARGS` expansion with `read -ra helmfile_args_array <<< "$INPUT_HELMFILE_ARGS"` and `"${helmfile_args_array[@]}"` to safely expand caller-controlled helmfile arguments.

2. Helm raw render (lines 268-270): (a) Replaced the `VALUES_STR` string concatenation loop with a proper bash array `values_args` using `"--values" "$element"` pairs expanded as `"${values_args[@]}"`. (b) Replaced bare `$INPUT_HELM_ARGS` with `read -ra helm_args_array` + `"${helm_args_array[@]}"`. (c) Replaced bare `$KUBE_VERSION_ARG` with `read -ra kube_version_arg_array` + `"${kube_version_arg_array[@]}"`. All three were unquoted expansions of caller-controlled values.

3. Get Webapp (line 285): Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` to strip newlines before writing to $GITHUB_OUTPUT, preventing newline-injection attacks that could poison subsequent steps' environment variables or outputs.

