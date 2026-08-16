<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.7.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit pins, making the action vulnerable to supply-chain attacks if those tags are moved. Unpinned references found:
- `dcarbone/install-yq-action@v1.1.0` (line 111)
- `mamezou-tech/setup-helmfile@v2.2.0` (line 117)
- `actions/setup-node@v6` (line 123)
- `cloudposse/github-action-yaml-config-query@v1.0.1` (lines 162, 232, 294)
- `actions/checkout@v6` (line 169)
- `nick-fields/retry@v4` (line 319)
- `1arp/create-a-file-action@0.2` (line 305)
- `cloudposse/github-action-wait-commit-status@v0.2.1` (line 370)
Note: `actions/github-script@f28e40c7f34bde8b3046d885e986cb6290c5673b` is correctly pinned.

Locations:

- `action.yml:111`
- `action.yml:117`
- `action.yml:123`
- `action.yml:162`
- `action.yml:169`
- `action.yml:232`
- `action.yml:294`
- `action.yml:305`
- `action.yml:319`
- `action.yml:370`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing an attacker who controls those inputs to inject arbitrary shell commands. Affected steps and offending lines:

1. **Install git-url-parse** (line 129): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner context interpolated directly in shell.

2. **Parse git URL for destination** (line 152): `run("${{ inputs.cluster }}")` — attacker-controlled `inputs.cluster` interpolated directly into a JS script string, enabling code injection.

3. **Read platform context** (line 200): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — attacker-controlled inputs interpolated directly into shell command.

4. **Read platform metadata** (line 210): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — attacker-controlled `inputs.ssm-path` interpolated directly.

5. **Resolve kube version** (lines 223–225): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly.

6. **Ensure argocd repo structure** (line 241): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

7. **Helmfile render** (lines 246–251): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple attacker-controlled inputs interpolated directly.

8. **Build Helm Dependencies** (line 259): `helm dependency build ${{ inputs.path }}` — attacker-controlled `inputs.path` interpolated directly.

9. **Helm raw render** (lines 264–278): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }}` — many attacker-controlled inputs interpolated directly.

10. **Get Webapp** (line 287): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — step output interpolated directly in shell.

11. **Push to Github** (command block, lines 330–354): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple github context and step output values interpolated directly into shell.

12. **Select GitHub Token for Sync Mode** (lines 364–367): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"` — attacker-controlled inputs interpolated directly.

Locations:

- `action.yml:129`
- `action.yml:152`
- `action.yml:200`
- `action.yml:210`
- `action.yml:223`
- `action.yml:241`
- `action.yml:246`
- `action.yml:259`
- `action.yml:264`
- `action.yml:287`
- `action.yml:330`
- `action.yml:364`

### github-env-injection (severity: high)

The **Select GitHub Token for Sync Mode** step writes attacker-controlled `inputs.github-pat` and `inputs.commit-status-github-token` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in either token value could inject additional key=value pairs into the output file, poisoning subsequent steps that read from `$GITHUB_OUTPUT`.

Offending lines:
```
echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT
echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT
```

Additionally, the **Resolve kube version** step writes `${{ inputs.kube-version }}` (via the `$kube_version` shell variable, set directly from the expression) to `$GITHUB_OUTPUT` without sanitization:
```
kube_version="${{ inputs.kube-version }}"
...
echo "value=$kube_version" >> $GITHUB_OUTPUT
```

Locations:

- `action.yml:365`
- `action.yml:367`
- `action.yml:227`

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

1. **unpinned-uses**: Pinned all 8 unpinned action references to full 40-character SHA commits with tag comments for readability: dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query (3 occurrences), actions/checkout, nick-fields/retry, 1arp/create-a-file-action, cloudposse/github-action-wait-commit-status.

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: maps for every affected step: Install git-url-parse (runner.temp), Parse git URL (inputs.cluster via process.env.INPUT_CLUSTER in JS), Read platform context (inputs.ssm-path, inputs.environment), Read platform metadata (inputs.ssm-path), Resolve kube version (inputs.kube-version, inputs.ssm-path, steps.metadata.outputs.kube_version), Ensure argocd repo structure (steps.config.outputs.tmp), Helmfile render (all inputs), Build Helm Dependencies (inputs.path), Helm raw render (all inputs), Get Webapp (steps.config.outputs.tmp), Push to Github (step outputs and inputs moved to env vars; github context uses built-in GITHUB_* env vars), Select GitHub Token (inputs.commit-status-github-token, inputs.github-pat).

3. **github-env-injection**: Sanitized all values written to $GITHUB_OUTPUT using printf '%s' | tr -d '\n\r' in the Resolve kube version step and the Select GitHub Token for Sync Mode step.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, github-env-injection

**Notes:**

Fixed all 5 security findings:

1. script-injection (sub-rule a) in test-helmfile-raw.yml: Moved ${{ needs.setup.outputs.environment }}, ${{ github.sha }}, ${{ github.run_id }}, ${{ github.run_attempt }} from run: shell strings to env: blocks.

2. script-injection (sub-rule b) in action.yml: Replaced unquoted $INPUT_HELMFILE_ARGS, ${array[@]}, ${VALUES_STR}, $INPUT_HELM_ARGS, $KUBE_VERSION_ARG with proper bash arrays using read -ra and ${arr[@]+"${arr[@]}"} expansion.

3. unpinned-uses: Pinned all mutable tag/branch references to full SHA digests in all 7 workflow files: actions/checkout@v6→df4cb1c, actions/create-github-app-token@v3→bcd2ba4, nick-fields/assert-action@v2→aa0067e, jiangxin/file-exists-action@v1→81a413f, myrotvorets/set-commit-status-action@master→da7fe59, cloudposse/.github@main→8244c7c.

4. missing-permissions: Added 'permissions: contents: read' to test-destroy.yml, test-helm-raw.yml, test-helm-raw-default-kube-version.yml, and test-helmfile-raw-default-kube-version.yml.

5. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT in the YQ Platform settings (metadata) step and Get Webapp step in action.yml.

