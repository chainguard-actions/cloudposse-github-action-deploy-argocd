<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 29 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell commands, violating sub-rule (a). Affected steps include:
- 'Install git-url-parse': `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`
- 'Setup chamber' (Read platform context): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...`
- 'Read platform metadata': `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`
- 'YQ Platform settings' (metadata): iterates over yq output and writes to GITHUB_OUTPUT
- 'Resolve kube version': `kube_version="${{ inputs.kube-version }}"` and `if [ -z "$kube_version" ] && [ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"`
- 'Ensure argocd repo structure': `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`
- 'Helmfile render': `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml`
- 'Build Helm Dependencies': `helm dependency build ${{ inputs.path }}`
- 'Helm raw render': `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }}`
- 'Get Webapp': `${{ steps.config.outputs.tmp }}/manifests/resources.yaml`
- 'Push to Github' (command: block): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}'`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}`
- 'Select GitHub Token for Sync Mode': `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT`, `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:113`
- `action.yml:114`
- `action.yml:148`
- `action.yml:163`
- `action.yml:175`
- `action.yml:185`
- `action.yml:193`
- `action.yml:200`
- `action.yml:210`
- `action.yml:219`
- `action.yml:228`
- `action.yml:237`
- `action.yml:248`
- `action.yml:258`
- `action.yml:270`
- `action.yml:280`
- `action.yml:295`
- `action.yml:310`
- `action.yml:330`
- `action.yml:340`
- `action.yml:355`
- `action.yml:360`

### script-injection (severity: high)

The 'wait-for-commit' run: block in test-helmfile-raw.yml directly interpolates ${{ github.sha }}, ${{ github.run_id }}, and ${{ github.run_attempt }} inside a shell string assignment, violating sub-rule (a): `SEEK_MESSAGE="Deploy cloudposse/github-action-deploy-argocd SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`

Locations:

- `.github/workflows/test-helmfile-raw.yml:120`

### github-env-injection (severity: high)

Multiple run: blocks write untrusted input values to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'):

1. 'Select GitHub Token for Sync Mode' step: writes ${{ inputs.github-pat }} and ${{ inputs.commit-status-github-token }} directly to $GITHUB_OUTPUT via `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT`. These are caller-controlled inputs.

2. 'YQ Platform settings' (metadata) step: writes SSM-sourced key=value pairs from yq output directly to $GITHUB_OUTPUT via `echo "${output}" >> $GITHUB_OUTPUT` in a loop. The values come from external SSM data and are not sanitized.

3. 'Resolve kube version' step: writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` where $kube_version is set from `${{ inputs.kube-version }}` or `${{ steps.metadata.outputs.kube_version }}` — both untrusted sources — without sanitization.

Locations:

- `action.yml:200`
- `action.yml:210`
- `action.yml:219`
- `action.yml:355`
- `action.yml:360`

### unpinned-uses (severity: high)

action.yml references multiple actions by mutable version tags instead of full 40-character commit SHAs:
- `dcarbone/install-yq-action@v1.1.0`
- `mamezou-tech/setup-helmfile@v2.2.0`
- `actions/setup-node@v4`
- `cloudposse/github-action-yaml-config-query@v1.0.1` (used 3 times)
- `actions/checkout@v6`
- `1arp/create-a-file-action@0.2`
- `nick-fields/retry@v4`
- `cloudposse/github-action-wait-commit-status@v0.2.1`
Only `actions/github-script@f28e40c7f34bde8b3046d885e986cb6290c5673b` is correctly pinned to a SHA.

Locations:

- `action.yml:96`
- `action.yml:101`
- `action.yml:107`
- `action.yml:130`
- `action.yml:155`
- `action.yml:160`
- `action.yml:205`
- `action.yml:285`
- `action.yml:320`
- `action.yml:365`

### unpinned-uses (severity: high)

Workflow files reference actions by mutable tags or branch names instead of full 40-character commit SHAs:

branch.yml:
- `cloudposse/.github/.github/workflows/shared-github-action.yml@main`

release.yml:
- `cloudposse/.github/.github/workflows/shared-release-branches.yml@main`

test-destroy.yml:
- `actions/checkout@v6`
- `actions/create-github-app-token@v3`
- `nick-fields/assert-action@v2`
- `jiangxin/file-exists-action@v1`

test-helm-raw.yml:
- `actions/checkout@v6`
- `actions/create-github-app-token@v3`
- `nick-fields/assert-action@v2`

test-helm-raw-default-kube-version.yml:
- `actions/checkout@v6`
- `actions/create-github-app-token@v3`
- `nick-fields/assert-action@v2`

test-helmfile-raw-default-kube-version.yml:
- `actions/checkout@v6`
- `actions/create-github-app-token@v3`
- `nick-fields/assert-action@v2`

test-helmfile-raw.yml:
- `actions/checkout@v6`
- `actions/create-github-app-token@v3`
- `myrotvorets/set-commit-status-action@master`
- `nick-fields/assert-action@v2`

Locations:

- `.github/workflows/branch.yml:18`
- `.github/workflows/release.yml:9`
- `.github/workflows/test-destroy.yml:35`
- `.github/workflows/test-destroy.yml:75`
- `.github/workflows/test-destroy.yml:107`
- `.github/workflows/test-destroy.yml:120`
- `.github/workflows/test-destroy.yml:125`
- `.github/workflows/test-helm-raw.yml:34`
- `.github/workflows/test-helm-raw.yml:62`
- `.github/workflows/test-helm-raw.yml:97`
- `.github/workflows/test-helm-raw.yml:102`
- `.github/workflows/test-helm-raw-default-kube-version.yml:34`
- `.github/workflows/test-helm-raw-default-kube-version.yml:57`
- `.github/workflows/test-helm-raw-default-kube-version.yml:88`
- `.github/workflows/test-helmfile-raw-default-kube-version.yml:34`
- `.github/workflows/test-helmfile-raw-default-kube-version.yml:57`
- `.github/workflows/test-helmfile-raw-default-kube-version.yml:88`
- `.github/workflows/test-helmfile-raw.yml:38`
- `.github/workflows/test-helmfile-raw.yml:66`
- `.github/workflows/test-helmfile-raw.yml:107`
- `.github/workflows/test-helmfile-raw.yml:148`
- `.github/workflows/test-helmfile-raw.yml:160`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions:
- test-destroy.yml
- test-helm-raw.yml
- test-helm-raw-default-kube-version.yml
- test-helmfile-raw-default-kube-version.yml

Locations:

- `.github/workflows/test-destroy.yml:1`
- `.github/workflows/test-helm-raw.yml:1`
- `.github/workflows/test-helm-raw-default-kube-version.yml:1`
- `.github/workflows/test-helmfile-raw-default-kube-version.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all security findings in action.yml and .github/workflows/ files:

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: blocks in action.yml. Key steps fixed: Install git-url-parse (runner.temp), Read platform context (ssm-path, environment), Read platform metadata (ssm-path), Resolve kube version (kube-version, ssm-path, metadata output), Ensure argocd repo structure (config.tmp), Helmfile render (namespace, environment, path, helmfile-args, kube_version), Build Helm Dependencies (path), Helm raw render (values_file, application, path, image, image-tag, environment, namespace, helm-args, kube_version), Get Webapp (config.tmp), Push to Github (destination_dir, destination.ref, operation, config.path, github.repository, github.sha, github.run_id, github.run_attempt), Select GitHub Token (commit-status-github-token, github-pat). Also fixed test-helmfile-raw.yml wait-for-commit step.

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before all GITHUB_OUTPUT writes in: YQ Platform settings (metadata loop), Resolve kube version, Select GitHub Token for Sync Mode.

3. unpinned-uses (action.yml): Pinned all 8 unpinned action references to full 40-char SHAs.

4. unpinned-uses (workflows): Pinned all unpinned action references in branch.yml, release.yml, test-destroy.yml, test-helm-raw.yml, test-helm-raw-default-kube-version.yml, test-helmfile-raw-default-kube-version.yml, test-helmfile-raw.yml.

5. missing-permissions: Added 'permissions: contents: read' to test-destroy.yml, test-helm-raw.yml, test-helm-raw-default-kube-version.yml, test-helmfile-raw-default-kube-version.yml.

List/args inputs (helmfile-args, helm-args) are tokenized with xargs to preserve argument boundaries. Single-value inputs use quoted $VAR expansion.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection findings:

1. action.yml 'Helm raw render' step: Replaced unquoted `${VALUES_STR}` string expansion with a proper bash array `values_args`. Each values file is now added as `--values "$element"` to the array, which is then expanded as `"${values_args[@]}"` in the helm template command. This prevents word-splitting and glob expansion of attacker-controlled `values_file` input.

2. test-helmfile-raw.yml SSM put-parameter step (lines 60/65): Moved `${{ needs.setup.outputs.environment }}` into the step's env: block as `SETUP_ENVIRONMENT`, replacing direct expression interpolation in the shell run: block.

3. test-helmfile-raw.yml wait-for-commit step (line 168): Changed `echo ${SEEK_MESSAGE}` to `echo "${SEEK_MESSAGE}"` to prevent word-splitting and glob expansion.

4. test-helmfile-raw.yml assert job steps (lines 229, 240, 251, 262): Moved `${{ needs.setup.outputs.environment }}` into env: blocks as `SETUP_ENVIRONMENT` for all four yq steps (Get Image, Get Ingress, Get Name, Get ref), replacing direct expression interpolation in file path arguments.

