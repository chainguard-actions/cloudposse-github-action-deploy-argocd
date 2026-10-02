<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.11.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags/versions rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved: `dcarbone/install-yq-action@v1.3.1`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used three times), `1arp/create-a-file-action@0.4`, `nick-fields/retry@v4`. All should be pinned to a full 40-char SHA digest.

Locations:

- `action.yml:100`
- `action.yml:106`
- `action.yml:113`
- `action.yml:148`
- `action.yml:196`
- `action.yml:247`
- `action.yml:268`
- `action.yml:285`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ inputs.* }}`, `${{ steps.*.outputs.* }}`, and `${{ github.* }}` expressions into shell command strings (sub-rule a), allowing an attacker who controls those inputs to inject arbitrary shell commands. Affected steps:

- **Install git-url-parse**: `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly into shell.
- **Read platform context**: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — attacker-controlled inputs in shell command.
- **Read platform metadata**: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — same issue.
- **Resolve kube version**: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]`.
- **Ensure argocd repo structure**: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`.
- **Helmfile render**: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}`.
- **Build Helm Dependencies**: `helm dependency build ${{ inputs.path }}`.
- **Helm raw render**: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} --set image.tag=${{ inputs.image-tag }} --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}`.
- **Get Webapp**: `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml`.
- **Push to Github (command)**: `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}`.
- **Select GitHub Token**: `if [ -z "${{ inputs.commit-status-github-token }}" ]`.

All expressions must be moved to `env:` variables and referenced as `"$VAR"` (double-quoted) in the shell script.

Locations:

- `action.yml:121`
- `action.yml:163`
- `action.yml:181`
- `action.yml:196`
- `action.yml:213`
- `action.yml:220`
- `action.yml:232`
- `action.yml:241`
- `action.yml:258`
- `action.yml:295`
- `action.yml:325`

### github-env-injection (severity: high)

Two `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

**Select GitHub Token for Sync Mode**: Writes `${{ inputs.github-pat }}` and `${{ inputs.commit-status-github-token }}` directly to `$GITHUB_OUTPUT` via template interpolation — a newline in the PAT value could inject additional key=value pairs:
```
echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT
echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT
```

**Resolve kube version**: Assigns `kube_version` directly from `${{ inputs.kube-version }}` via template interpolation, then writes it to `$GITHUB_OUTPUT` without sanitization:
```
kube_version="${{ inputs.kube-version }}"
...
echo "value=$kube_version" >> $GITHUB_OUTPUT
```

Fix: use `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before every write to a special environment file.

Locations:

- `action.yml:197`
- `action.yml:204`
- `action.yml:326`
- `action.yml:328`

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

1. **unpinned-uses**: Pinned all 6 unpinned action references to full 40-char SHAs:
   - dcarbone/install-yq-action@v1.3.1 → @4075b4dca348d74bd83f2bf82d30f25d7c54539b
   - mamezou-tech/setup-helmfile@v2.2.0 → @c04e83ec7650bf2ec910864bcb409479cf56d8e6
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - cloudposse/github-action-yaml-config-query@v1.0.1 → @8178e0c0d186f53de40f6bcf8e039f1e6a9aefc5 (3 occurrences)
   - 1arp/create-a-file-action@0.4 → @d85f6db32e7029404a7cf9edcefe475675c82ec2
   - nick-fields/retry@v4 → @ad984534de44a9489a53aefd81eb77f87c70dc60

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ github.* }}, ${{ runner.temp }}, and ${{ steps.*.outputs.* }} expressions from run: blocks to env: blocks. Shell scripts now reference $VAR_NAME instead.

3. **github-env-injection**: Added `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` sanitization before all writes to $GITHUB_OUTPUT in the 'Resolve kube version' and 'Select GitHub Token for Sync Mode' steps.

4. **List inputs** (helmfile-args, helm-args, values_file): Used xargs-based tokenization with bash arrays to properly handle whitespace-separated argument lists without shell injection.

5. **Install git-url-parse step**: Replaced ${{ runner.temp }} with $RUNNER_TEMP environment variable.

6. **Parse git URL step**: Moved ${{ inputs.cluster }} to env: block as INPUT_CLUSTER, referenced via process.env.INPUT_CLUSTER in the JavaScript.

### Iteration 2

**Fixes applied:** github-env-injection, github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'YQ Platform settings' step (id: metadata, ~line 208): Added `safe=$(printf '%s' "${output}" | tr -d '\n\r')` before writing each metadata key=value pair to $GITHUB_OUTPUT, replacing the direct `echo "${output}" >> $GITHUB_OUTPUT`.
2. 'Get Webapp' step (id: result, ~line 295): Added `safe=$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')` before writing the webapp URL to $GITHUB_OUTPUT, replacing the direct `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT`.

