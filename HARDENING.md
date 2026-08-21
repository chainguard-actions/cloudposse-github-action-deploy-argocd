<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.1** was hardened automatically. 26 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved. Unpinned references:
- `dcarbone/install-yq-action@v1.3.1`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v6`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times)
- `1arp/create-a-file-action@0.4`
- `nick-fields/retry@v4`

Locations:

- `action.yml:132`
- `action.yml:138`
- `action.yml:144`
- `action.yml:185`
- `action.yml:262`
- `action.yml:333`
- `action.yml:342`
- `action.yml:353`

### script-injection (severity: high)

Multiple `run:` blocks (and the `command:` shell block in the nick-fields/retry step) directly interpolate `${{ ... }}` expressions — including `inputs.*`, `steps.*.outputs.*`, `github.*`, and `runner.*` — into shell commands without routing through env vars. This allows an attacker-controlled value to inject arbitrary shell commands.

Affected steps and offending lines (sub-rule a):

1. **Install git-url-parse**: `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in run:
2. **Read platform context**: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — attacker-controlled inputs injected into shell command
3. **Read platform metadata**: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — attacker-controlled input injected
4. **Resolve kube version**: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` — inputs interpolated directly in run:
5. **Ensure argocd repo structure**: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated in run:
6. **Helmfile render**: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs injected
7. **Build Helm Dependencies**: `helm dependency build ${{ inputs.path }}` — input injected
8. **Helm raw render**: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ...` — multiple inputs injected
9. **Get Webapp**: `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output injected
10. **Push to Github (command:)**: `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple contexts injected
11. **Select GitHub Token**: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input injected directly in run:

Locations:

- `action.yml:150`
- `action.yml:217`
- `action.yml:236`
- `action.yml:252`
- `action.yml:274`
- `action.yml:280`
- `action.yml:295`
- `action.yml:301`
- `action.yml:325`
- `action.yml:363`
- `action.yml:391`

### github-env-injection (severity: high)

Several `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that can set arbitrary output variables or environment variables.

1. **Resolve kube version** step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set directly from `${{ inputs.kube-version }}` (an attacker-controlled input) in the same run block without sanitization.
2. **Select GitHub Token** step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — `inputs.*` values written directly to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:256`
- `action.yml:392`
- `action.yml:394`

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

1. **unpinned-uses**: Pinned all 6 unpinned action references to full 40-character commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, 1arp/create-a-file-action, nick-fields/retry).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ runner.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions out of run: blocks into env: blocks. Referenced them as plain environment variables in shell scripts. For list-type inputs (helmfile-args, helm-args, kube_version), used xargs-based tokenization into bash arrays to preserve argument boundaries.

3. **github-env-injection**: Sanitized all values written to $GITHUB_OUTPUT using `printf '%s' "$VAR" | tr -d '\n\r'` before writing, specifically in the 'Resolve kube version' step (kube_version) and 'Select GitHub Token for Sync Mode' step (both token branches).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 7 findings across action.yml and 6 workflow files:

1. script-injection (action.yml): Moved inputs.cluster from github-script's script: field into env: block as INPUT_CLUSTER, referenced via process.env.INPUT_CLUSTER.

2. github-env-injection (action.yml metadata step): Replaced for-loop with while-read loop using printf/tr sanitization; changed yq expression to emit one entry per line.

3. github-env-injection (action.yml Get Webapp step): Added printf/tr sanitization for WEBAPP_URL before writing to GITHUB_OUTPUT.

4. script-injection (action.yml Helm raw render): Replaced unquoted ${VALUES_STR} string with a proper bash array values_args=(--values "$element").

5. script-injection (test-helmfile-raw.yml): Moved github.sha/run_id/run_attempt into env: block for wait-for-commit step; moved needs.setup.outputs.environment into env: blocks for Get Image, Get Ingress, Get Name, Get ref steps.

6. unpinned-uses: Pinned actions/create-github-app-token@v3 to SHA bcd2ba49218906704ab6c1aa796996da409d3eb1, jiangxin/file-exists-action@v1 to SHA 81a413fe02b9d9c4737c01bf2f566e12735a3285, myrotvorets/set-commit-status-action@master to SHA 596059dae8482156246977ccb271bcfbfede0bec, and both cloudposse/.github shared workflow references to SHA 3911c663309ecdda30d8b8fcbec7bde19d1d6ddb.

7. missing-permissions: Added top-level permissions: contents: read to test-destroy.yml, test-helm-raw.yml, test-helm-raw-default-kube-version.yml, and test-helmfile-raw-default-kube-version.yml.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test-helmfile-raw.yml: moved `${{ needs.setup.outputs.environment }}` out of the `run:` shell command string and into the step's `env:` block as `ENVIRONMENT: ${{ needs.setup.outputs.environment }}`. Both occurrences in the `--name` arguments of the AWS SSM put-parameter commands now reference `$ENVIRONMENT` as a plain shell variable instead of directly interpolating the GitHub Actions expression.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted variable expansion in the `wait-for-commit` step of `.github/workflows/test-helmfile-raw.yml`. Changed `echo ${SEEK_MESSAGE}` to `echo "${SEEK_MESSAGE}"` to prevent word-splitting and glob expansion. The SEEK_MESSAGE variable is constructed from env vars sourced from github context (sha, run_id, run_attempt), and the unquoted echo could allow shell metacharacter interpretation.

