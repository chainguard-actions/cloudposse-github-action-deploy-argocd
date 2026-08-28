<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.11.0** was hardened automatically. 26 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable version tags instead of full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `dcarbone/install-yq-action@v1.3.1`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v6`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times)
- `1arp/create-a-file-action@0.4`
- `nick-fields/retry@v4`

Only `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1` and `actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3` and `cloudposse/github-action-wait-commit-status@968fc17eddf4d9f4f2203edb4c72814a1d8bc650` are correctly pinned.

Locations:

- `action.yml:100`
- `action.yml:105`
- `action.yml:112`
- `action.yml:152`
- `action.yml:232`
- `action.yml:285`
- `action.yml:308`
- `action.yml:323`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell command strings. These values are substituted by the template engine before the shell parses them, allowing an attacker-controlled value to inject arbitrary shell commands.

Violations (sub-rule a — direct expression interpolation):

1. **Install git-url-parse** step: `mkdir -p ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

2. **Parse git URL for destination** (github-script `run(...)` call): `run("${{ inputs.cluster }}")` — attacker-controlled `inputs.cluster` interpolated directly into the script string.

3. **Read platform context** step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — inputs interpolated directly into shell command.

4. **Read platform metadata** step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — input interpolated directly.

5. **Resolve kube version** step: `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.

6. **Ensure argocd repo structure** step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

7. **Helmfile render** step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly.

8. **Build Helm Dependencies** step: `helm dependency build ${{ inputs.path }}` — input interpolated directly.

9. **Helm raw render** step: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }}` — multiple inputs interpolated directly.

10. **Get Webapp** step: `yq ... ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly.

11. **Push to Github** (nick-fields/retry `command:`) : `pushd ./${{ steps.destination_dir.outputs.name }}`, `case '${{ inputs.operation }}'`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple github context values and step outputs interpolated directly.

12. **Select GitHub Token for Sync Mode** step: `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly in shell test.

Locations:

- `action.yml:119`
- `action.yml:131`
- `action.yml:185`
- `action.yml:200`
- `action.yml:218`
- `action.yml:243`
- `action.yml:253`
- `action.yml:270`
- `action.yml:277`
- `action.yml:295`
- `action.yml:335`
- `action.yml:374`

### github-env-injection (severity: high)

Several `run:` steps write untrusted or externally-derived values to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`), enabling newline injection that could allow an attacker to inject arbitrary output variables:

1. **YQ Platform settings** (metadata step): Iterates over yq-parsed SSM metadata content and writes each key=value pair directly to `$GITHUB_OUTPUT` with `echo "${output}" >> $GITHUB_OUTPUT`. The `${output}` variable is derived from external SSM data and is not sanitized before the write.

2. **Resolve kube version** step: Writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` where `$kube_version` is derived directly from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` (both untrusted). No sanitization is applied.

3. **Select GitHub Token for Sync Mode** step: Writes `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — both `inputs.github-pat` and `inputs.commit-status-github-token` are caller-controlled and written without sanitization.

Locations:

- `action.yml:218`
- `action.yml:232`
- `action.yml:374`

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

1. **unpinned-uses**: Pinned all 6 unpinned action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, 1arp/create-a-file-action, nick-fields/retry).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks to env: blocks across all affected steps: Install git-url-parse ($RUNNER_TEMP), Parse git URL (INPUT_CLUSTER via process.env), Read platform context (SSM_PATH, ENVIRONMENT), Read platform metadata (SSM_PATH), Resolve kube version (INPUT_KUBE_VERSION, INPUT_SSM_PATH, METADATA_KUBE_VERSION), Ensure argocd repo structure (CONFIG_TMP), Helmfile render (all inputs as env vars, helmfile-args and kube_version tokenized with xargs array pattern), Build Helm Dependencies (INPUT_PATH), Helm raw render (all inputs as env vars, helm-args and kube_version tokenized with xargs), Get Webapp (CONFIG_TMP), Push to Github (DESTINATION_DIR, DESTINATION_REF, CONFIG_PATH, OPERATION, GIT_COMMIT_MSG), Select GitHub Token (COMMIT_STATUS_GITHUB_TOKEN, GITHUB_PAT).

3. **github-env-injection**: Added `printf '%s' ... | tr -d '\n\r'` sanitization before all GITHUB_OUTPUT writes that use externally-derived values: YQ Platform settings metadata loop, Resolve kube version output, and both branches of Select GitHub Token.

### Iteration 2

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 6 workflow files:

1. unpinned-uses: Pinned all mutable references to full SHAs:
   - cloudposse/.github@main → @3911c663309ecdda30d8b8fcbec7bde19d1d6ddb (branch.yml, release.yml)
   - actions/create-github-app-token@v3 → @bcd2ba49218906704ab6c1aa796996da409d3eb1 (8 occurrences across 5 files)
   - jiangxin/file-exists-action@v1 → @81a413fe02b9d9c4737c01bf2f566e12735a3285 (test-destroy.yml)
   - myrotvorets/set-commit-status-action@master → @15a9c2cb2a71e7f728b2eb965f90aeace9188f0f (test-helmfile-raw.yml)

2. missing-permissions: Added 'permissions: contents: read' top-level block to test-destroy.yml, test-helm-raw.yml, test-helm-raw-default-kube-version.yml, and test-helmfile-raw-default-kube-version.yml.

3. script-injection: In test-helmfile-raw.yml, moved all ${{ }} expressions out of run: shell strings into env: blocks: needs.setup.outputs.environment (test job run block + 4 assert job run blocks), github.sha/run_id/run_attempt (argocd job wait-for-commit run block).

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Helm raw render' step of action.yml. Replaced the unsafe string-concatenation approach (VALUES_STR with unquoted expansions) with a proper bash array (values_args). Each file path from the comma/space-separated INPUT_VALUES_FILE is now safely added to the array with proper quoting: `values_args+=(--values "$element")`. The array is expanded safely with `"${values_args[@]}"` in the helm template command, preventing shell metacharacters in the values_file input from causing command injection.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansion in the 'wait-for-commit' step of .github/workflows/test-helmfile-raw.yml. Changed `echo ${SEEK_MESSAGE}` to `echo "${SEEK_MESSAGE}"` to prevent shell metacharacter interpretation of the workflow-controllable data in the SEEK_MESSAGE variable (which is constructed from github.sha, github.run_id, and github.run_attempt).

### Iteration 5

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get Webapp' step in action.yml (line ~280): added newline sanitization before writing WEBAPP_URL to $GITHUB_OUTPUT. The WEBAPP_URL value (read from yq-parsed Kubernetes manifests that could contain attacker-controlled content) is now sanitized with `safe=$(printf '%s' "${WEBAPP_URL}" | tr -d '\n\r')` before being written as `echo "webapp_url=${safe}" >> "$GITHUB_OUTPUT"`. This prevents injection of arbitrary key=value pairs into $GITHUB_OUTPUT via embedded newline characters.

