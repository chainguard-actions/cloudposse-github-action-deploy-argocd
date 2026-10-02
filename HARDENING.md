<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if any of these upstream actions are compromised or their tags are moved. Failing references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v4`, `cloudposse/github-action-yaml-config-query@v1.0.1` (used twice), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`.

Locations:

- `action.yml:131`
- `action.yml:136`
- `action.yml:142`
- `action.yml:182`
- `action.yml:189`
- `action.yml:267`
- `action.yml:350`
- `action.yml:361`
- `action.yml:415`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a). Before the shell executes the command, GitHub Actions substitutes these expressions as raw text, allowing an attacker who controls the input values to inject arbitrary shell metacharacters. Affected steps and offending expressions:

1. **Install git-url-parse** (~line 148): `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` — runner.temp is a workflow-controllable context.

2. **Read platform context** (~line 224): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — inputs.ssm-path and inputs.environment are attacker-controlled.

3. **Read platform metadata** (~line 242): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — inputs.ssm-path is attacker-controlled.

4. **Resolve kube version** (~line 258): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"` — all attacker-controllable.

5. **Ensure argocd repo structure** (~line 278): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — steps output is workflow-controllable.

6. **Helmfile render** (~line 284): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — multiple attacker-controlled inputs.

7. **Build Helm Dependencies** (~line 299): `helm dependency build ${{ inputs.path }}` — inputs.path is attacker-controlled.

8. **Helm raw render** (~line 306): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set image.tag=${{ inputs.image-tag }} ... --set environment=${{ inputs.environment }} ... --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` — numerous attacker-controlled inputs.

9. **Get Webapp** (~line 332): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` used as a shell argument — workflow-controllable.

10. **Push to Github** (command: block, ~line 375): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"`, `git push origin ${{ steps.destination.outputs.ref }}` — multiple attacker-controllable values.

11. **Select GitHub Token for Sync Mode** (~line 409): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT`, `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — sensitive token values interpolated directly into shell.

Locations:

- `action.yml:148`
- `action.yml:224`
- `action.yml:242`
- `action.yml:258`
- `action.yml:278`
- `action.yml:284`
- `action.yml:299`
- `action.yml:306`
- `action.yml:332`
- `action.yml:375`
- `action.yml:409`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into these values to poison subsequent step outputs or environment variables.

1. **Resolve kube version** (~line 262): `echo "value=$kube_version" >> $GITHUB_OUTPUT` — `$kube_version` is set directly from `${{ inputs.kube-version }}` (and `${{ steps.metadata.outputs.kube_version }}`), both attacker-controllable, with no newline sanitization before the write.

2. **YQ Platform settings / metadata** (~line 250): `echo "${output}" >> $GITHUB_OUTPUT` inside a loop — `${output}` comes from parsing a YAML file whose path is derived from `${{ inputs.ssm-path }}`. The file content is external and unsanitized.

3. **Select GitHub Token for Sync Mode** (~line 410–412): `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` — token values from inputs are written directly to $GITHUB_OUTPUT without sanitization. Even though these are secrets, a newline in the value would corrupt the output file format.

4. **Get Webapp** (~line 334): `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` — `WEBAPP_URL` is extracted by yq from a file at a path derived from `${{ steps.config.outputs.tmp }}` (workflow-controllable). The yq output is unsanitized before the write.

Locations:

- `action.yml:262`
- `action.yml:250`
- `action.yml:410`
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

Rewrote hardened/action/action.yml with all security fixes:

1. **unpinned-uses**: Pinned all 9 action references to full 40-char commit SHAs (dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query x3, actions/checkout, 1arp/create-a-file-action, nick-fields/retry, cloudposse/github-action-wait-commit-status).

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ runner.temp }}, ${{ steps.*.outputs.* }}, and ${{ github.* }} expressions out of run: shell blocks into env: maps. Shell scripts now reference only plain $VAR_NAME environment variables.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before every write to $GITHUB_OUTPUT in: Resolve kube version, YQ Platform settings (metadata loop), Select GitHub Token for Sync Mode, and Get Webapp steps.

4. **List-type inputs** (helmfile-args, helm-args, values_file): Tokenized using the xargs+while-read-NUL pattern with guards to properly handle whitespace-separated argument lists without collapsing them into a single word.

The actions/github-script step was already pinned to a SHA and was left unchanged.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yml:

1. (a) 'Parse git URL for destination' step: Moved `inputs.cluster` out of the `actions/github-script` script body and into the step's `env:` block as `CLUSTER: ${{ inputs.cluster }}`. The JavaScript now reads it safely via `run(process.env.CLUSTER)` instead of the previously injected `run("${{ inputs.cluster }}")` which allowed arbitrary JavaScript injection.

2. (b) 'Helm raw render' step: Replaced the `VALUES_STR` string variable (built by concatenating `--values <file>` strings and then expanded unquoted as `${VALUES_STR}`) with a proper bash array `values_arr`. Each `--values` flag and its file argument are now added as separate array elements via `values_arr+=(--values "$t")`, and expanded safely as `"${values_arr[@]}"` in the helm template command. The `# shellcheck disable=SC2086` comment was also removed since it's no longer needed.

