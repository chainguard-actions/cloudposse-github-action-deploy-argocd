<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags or version strings instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten. Failing references:
- `dcarbone/install-yq-action@v1.1.0`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v4`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times)
- `actions/checkout@v6`
- `1arp/create-a-file-action@0.2`
- `nick-fields/retry@v4`
- `cloudposse/github-action-wait-commit-status@v0.2.1`

Locations:

- `action.yml:131`
- `action.yml:137`
- `action.yml:144`
- `action.yml:185`
- `action.yml:192`
- `action.yml:240`
- `action.yml:265`
- `action.yml:317`
- `action.yml:325`
- `action.yml:334`
- `action.yml:378`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). The Actions runner substitutes these expressions before the shell parses the command, allowing an attacker who controls the input values to inject arbitrary shell commands via metacharacters.

Affected steps and offending lines:

1. **Install git-url-parse** (sub-rule a): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

2. **Parse git URL for destination** (sub-rule a): `run("${{ inputs.cluster }}")` — attacker-controlled `inputs.cluster` interpolated directly into a JS script string.

3. **Read platform context** (sub-rule a): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — inputs interpolated directly into shell command.

4. **Read platform metadata** (sub-rule a): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — input interpolated directly.

5. **Resolve kube version** (sub-rule a): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs/step outputs interpolated directly.

6. **Ensure argocd repo structure** (sub-rule a): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

7. **Helmfile render** (sub-rule a): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.

8. **Build Helm Dependencies** (sub-rule a): `helm dependency build ${{ inputs.path }}` — input interpolated directly.

9. **Helm raw render** (sub-rule a): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.

10. **Get Webapp** (sub-rule a): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly in shell.

11. **Push to Github** (sub-rule a): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple inputs and github context values interpolated directly.

12. **Select GitHub Token for Sync Mode** (sub-rule a): `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly in shell condition.

Locations:

- `action.yml:151`
- `action.yml:175`
- `action.yml:218`
- `action.yml:232`
- `action.yml:248`
- `action.yml:265`
- `action.yml:270`
- `action.yml:283`
- `action.yml:289`
- `action.yml:308`
- `action.yml:347`
- `action.yml:370`

### github-env-injection (severity: high)

Multiple `run:` blocks write untrusted input values to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that could allow an attacker to set arbitrary output variables or environment values.

1. **Select GitHub Token for Sync Mode**: Writes `${{ inputs.github-pat }}` and `${{ inputs.commit-status-github-token }}` directly to `$GITHUB_OUTPUT`:
   ```
   echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT
   echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT
   ```
   These are caller-controlled inputs written without sanitization.

2. **Resolve kube version**: Writes `$kube_version` to `$GITHUB_OUTPUT` where `$kube_version` is set directly from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` (both untrusted) without sanitization:
   ```
   kube_version="${{ inputs.kube-version }}"
   ...
   echo "value=$kube_version" >> $GITHUB_OUTPUT
   ```

3. **YQ Platform settings (metadata)**: Writes `${output}` derived from yq parsing of SSM-sourced YAML data to `$GITHUB_OUTPUT` without sanitization:
   ```
   for output in $(yq ... ./_metadata.yaml); do
     echo "${output}" >> $GITHUB_OUTPUT
   done
   ```
   The SSM data is externally controlled and could contain newlines.

Locations:

- `action.yml:370`
- `action.yml:248`
- `action.yml:240`

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

1. **unpinned-uses**: Pinned all 8 action references to full 40-character commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: maps. Key changes:
   - runner.temp → RUNNER_TEMP env var in Install git-url-parse
   - inputs.cluster → INPUT_CLUSTER env var, accessed via process.env.INPUT_CLUSTER in JS script
   - inputs.ssm-path, inputs.environment → env vars in Read platform context and Read platform metadata
   - inputs.kube-version, inputs.ssm-path, steps.metadata.outputs.kube_version → env vars in Resolve kube version
   - steps.config.outputs.tmp → CONFIG_TMP env var in Ensure argocd repo structure and Get Webapp
   - All inputs in Helmfile render, Build Helm Dependencies, Helm raw render moved to env vars
   - helmfile-args, helm-args, kube_version_arg tokenized with xargs+while-read-NUL loop for safe argument splitting
   - All step outputs and github context in Push to Github moved to env vars
   - inputs.commit-status-github-token, inputs.github-pat moved to env vars in Select GitHub Token

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to GITHUB_OUTPUT in:
   - Select GitHub Token (both token paths)
   - Resolve kube version
   - YQ Platform settings (metadata) loop

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security issues in hardened/action/action.yml:
1. script-injection (line ~248): Quoted the bash array expansion in the 'Helm raw render' step: changed `for element in ${array[@]}` to `for element in "${array[@]}"`. Without double-quoting, shell metacharacters in the `values_file` input could be interpreted as shell commands.
2. github-env-injection (line ~278): Added newline sanitization in the 'Get Webapp' step before writing WEBAPP_URL to $GITHUB_OUTPUT. Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and changed the echo to use `${safe}` instead of `${WEBAPP_URL}`, preventing newline injection into the GitHub output file.

