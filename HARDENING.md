<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.0** was hardened automatically. 26 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable version tags or branch names instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks. Unpinned references: `dcarbone/install-yq-action@v1.3.1`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times), `actions/checkout@v6`, `1arp/create-a-file-action@0.4`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`. Only `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` is correctly pinned.

Locations:

- `action.yml:100`
- `action.yml:106`
- `action.yml:112`
- `action.yml:152`
- `action.yml:163`
- `action.yml:231`
- `action.yml:302`
- `action.yml:311`
- `action.yml:322`
- `action.yml:388`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions inside shell commands, enabling script injection. An attacker-controlled input value can break out of the shell command context and execute arbitrary code.

(a) Direct expression interpolation in run: blocks:
- `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell
- `run("${{ inputs.cluster }}")` inside a github-script `script:` block — inputs.cluster interpolated directly
- `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }}` — inputs interpolated directly in shell
- `chamber --verbose export ${{ inputs.ssm-path }}/_metadata` — inputs interpolated directly
- `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` — inputs interpolated directly
- `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly
- `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly
- `helm dependency build ${{ inputs.path }}` — input interpolated directly
- `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ...` — multiple inputs interpolated directly
- `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"` — github context interpolated directly
- `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly

Locations:

- `action.yml:120`
- `action.yml:148`
- `action.yml:185`
- `action.yml:200`
- `action.yml:218`
- `action.yml:228`
- `action.yml:237`
- `action.yml:248`
- `action.yml:258`
- `action.yml:268`
- `action.yml:345`
- `action.yml:375`

### github-env-injection (severity: high)

The 'Select GitHub Token for Sync Mode' run: block writes `inputs.github-pat` and `inputs.commit-status-github-token` directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled input containing newlines could inject additional key=value pairs into the GitHub output context, potentially overwriting other step outputs.

Failing lines:
  `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT`
  `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT`

Additionally, the 'YQ Platform settings' (metadata) step writes arbitrary yq-parsed output from an external YAML file to `$GITHUB_OUTPUT` without sanitization:
  `echo "${output}" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:375`
- `action.yml:377`
- `action.yml:379`
- `action.yml:222`

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

1. **unpinned-uses**: Pinned all 8 unpinned action references to full SHA digests (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks. Key changes:
   - runner.temp → $RUNNER_TEMP env var in Install git-url-parse step
   - inputs.cluster → INPUT_CLUSTER env var, accessed via process.env.INPUT_CLUSTER in github-script
   - inputs.ssm-path + inputs.environment → env: block in Read platform context
   - inputs.ssm-path → env: block in Read platform metadata
   - inputs.kube-version + inputs.ssm-path → env: block in Resolve kube version
   - steps.config.outputs.tmp → CONFIG_TMP env var in Ensure argocd repo structure
   - All helmfile render inputs → env: block with xargs-based tokenization for list args
   - inputs.path → env: block in Build Helm Dependencies
   - All helm raw render inputs → env: block with xargs-based tokenization for list args
   - inputs.commit-status-github-token + inputs.github-pat → env: block in Select GitHub Token

3. **github-env-injection**: 
   - Select GitHub Token step: both token writes sanitized with `printf '%s' "$VAR" | tr -d '\n\r'`
   - YQ Platform settings (metadata) step: each output sanitized with `printf '%s' "${output}" | tr -d '\n\r'` before writing to GITHUB_OUTPUT

Note: Some ${{ }} expressions remain in with: blocks (action inputs), if: conditions, and env: blocks — these are all safe contexts.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings in hardened/action/action.yml:

1. script-injection (Push to Github step, line 271): Moved all 8 `${{ }}` expressions from the `command:` shell string into an `env:` block on the step. The shell script now references them as plain environment variables: $GIT_DESTINATION_DIR, $GIT_DESTINATION_REF, $INPUT_OPERATION, $CONFIG_PATH, $GITHUB_REPOSITORY_NAME, $GITHUB_SHA_VALUE, $GITHUB_RUN_ID_VALUE, $GITHUB_RUN_ATTEMPT_VALUE.

2. github-env-injection (Resolve kube version step, line 207): Added `safe=$(printf '%s' "$kube_version" | tr -d '\n\r')` and write `$safe` instead of `$kube_version` to $GITHUB_OUTPUT.

3. github-env-injection (Get Webapp step, line 246): Added `safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r')` and write `$safe` instead of `$WEBAPP_URL` to $GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Helm raw render' step of action.yml. Replaced the VALUES_STR string accumulator (which was expanded unquoted as ${VALUES_STR} in the helm template command) with a proper bash array (values_args). The array is built with values_args+=(--values "$element") so each file path is properly quoted, and expanded safely with "${values_args[@]}" in the helm template command. This prevents word splitting, glob expansion, and injection of shell metacharacters via the user-controlled values_file input.

