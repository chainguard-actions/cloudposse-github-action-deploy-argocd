<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.1** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Failing references: `dcarbone/install-yq-action@v1.3.1`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times), `1arp/create-a-file-action@0.4`, `nick-fields/retry@v4`.

Locations:

- `action.yml:97`
- `action.yml:103`
- `action.yml:110`
- `action.yml:148`
- `action.yml:218`
- `action.yml:264`
- `action.yml:296`
- `action.yml:318`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions inside shell commands, enabling script injection. Sub-rule (a) violations — ${{ ... }} expressions embedded directly in run: scripts:

1. 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly into shell.

2. 'Parse git URL for destination' step (github-script): `run("${{ inputs.cluster }}")` — attacker-controlled input interpolated into JS script.

3. 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — inputs interpolated directly into shell command.

4. 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — input interpolated directly.

5. 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` — inputs interpolated into shell.

6. 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated unquoted into shell.

7. 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}` — input interpolated directly.

8. 'Helm raw render' step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.

9. 'Push to Github' step (nick-fields/retry command:): `case '${{ inputs.operation }}'`, `pushd ./${{ steps.destination_dir.outputs.name }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — github context and step outputs interpolated directly into shell.

10. 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly into shell condition.

Locations:

- `action.yml:118`
- `action.yml:143`
- `action.yml:175`
- `action.yml:196`
- `action.yml:213`
- `action.yml:228`
- `action.yml:244`
- `action.yml:249`
- `action.yml:270`
- `action.yml:330`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. 'Resolve kube version' step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is set from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` — unsanitized input written to GITHUB_OUTPUT.

2. 'YQ Platform settings' (metadata) step: `echo "${output}" >> $GITHUB_OUTPUT` where `${output}` is derived from yq parsing of SSM-sourced YAML — external/untrusted data written to GITHUB_OUTPUT without sanitization.

3. 'Select GitHub Token for Sync Mode' step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — attacker-controlled inputs written directly to GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:216`
- `action.yml:208`
- `action.yml:332`
- `action.yml:334`

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

1. **unpinned-uses**: Pinned all 6 action references to full 40-char SHAs: dcarbone/install-yq-action@4075b4d, mamezou-tech/setup-helmfile@c04e83e, actions/setup-node@2499707, cloudposse/github-action-yaml-config-query@8178e0c (x3), 1arp/create-a-file-action@d85f6db, nick-fields/retry@ad98453.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }} and ${{ github.* }} expressions from run: blocks to env: blocks. In the github-script step, changed run("${{ inputs.cluster }}") to run(process.env.INPUT_CLUSTER). For helmfile-args and helm-args (list inputs), used xargs-based tokenization into bash arrays. For kube_version arg (also a flag), used xargs tokenization.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all GITHUB_OUTPUT writes in: YQ Platform settings (metadata) step, Resolve kube version step, and Select GitHub Token step.

4. **runner.temp injection**: Replaced ${{ runner.temp }} with the built-in $RUNNER_TEMP environment variable in the Install git-url-parse step.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed four security findings in hardened/action/action.yml:

1. 'Ensure argocd repo structure' (line 218): Moved `${{ steps.config.outputs.tmp }}` to an env var `CONFIG_TMP` and quoted the shell expansion as `"$CONFIG_TMP/manifests"`.

2. 'Helm raw render' (line 261): Replaced unquoted `for element in ${array[@]}` with `for element in "${array[@]}"` and replaced the unsafe `${VALUES_STR}` string concatenation with a proper `values_args` bash array, passing each `--values "$element"` as separate quoted arguments.

3. 'Get Webapp' (line 280): Moved `${{ steps.config.outputs.tmp }}` to an env var `CONFIG_TMP` and quoted the yq file argument as `"$CONFIG_TMP/manifests/resources.yaml"`.

4. 'Get Webapp' (line 284): Added sanitization of `WEBAPP_URL` via `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` before writing to `$GITHUB_OUTPUT`, preventing newline injection attacks.

