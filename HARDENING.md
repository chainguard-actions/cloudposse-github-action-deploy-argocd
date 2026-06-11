<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-deploy-argocd/v1.7.0** was hardened automatically. 26 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml use mutable tags or version strings instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if those tags are moved. Unpinned references: `dcarbone/install-yq-action@v1.1.0`, `mamezou-tech/setup-helmfile@v2.2.0`, `actions/setup-node@v6`, `cloudposse/github-action-yaml-config-query@v1.0.1` (×3 occurrences), `actions/checkout@v6`, `1arp/create-a-file-action@0.2`, `nick-fields/retry@v4`, `cloudposse/github-action-wait-commit-status@v0.2.1`. Only `actions/github-script@f28e40c7f34bde8b3046d885e986cb6290c5673b` is correctly pinned.

Locations:

- `action.yml:117`
- `action.yml:122`
- `action.yml:129`
- `action.yml:165`
- `action.yml:175`
- `action.yml:259`
- `action.yml:347`
- `action.yml:357`
- `action.yml:372`
- `action.yml:431`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, violating rule (a). This allows an attacker who controls the inputs to inject arbitrary shell commands.

1. **Parse git URL step** (rule a): `run("${{ inputs.cluster }}")` — the `inputs.cluster` value is interpolated directly into a JS string that is then executed.

2. **Read platform context step** (rule a): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` — `inputs.ssm-path` and `inputs.environment` are interpolated directly into a shell command.

3. **Read platform metadata step** (rule a): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` — same issue.

4. **Resolve kube version step** (rule a): `kube_version="${{ inputs.kube-version }}"` and `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` — inputs and step outputs interpolated directly into shell.

5. **Ensure argocd repo structure step** (rule a): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` — step output interpolated directly.

6. **Helmfile render step** (rule a): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }}` — multiple inputs interpolated directly into shell.

7. **Build Helm Dependencies step** (rule a): `helm dependency build ${{ inputs.path }}` — `inputs.path` interpolated directly.

8. **Helm raw render step** (rule a): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... --set image.tag=${{ inputs.image-tag }} --set environment=${{ inputs.environment }} --namespace ${{ inputs.namespace }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }}` — many inputs interpolated directly.

9. **Push to Github step** (rule a): `git commit -m "Deploy ${{ github.repository }} SHA ${{ github.sha }} RUN ${{ github.run_id }} ATEMPT ${{ github.run_attempt }}"` — github context values interpolated directly into shell.

10. **Select GitHub Token step** (rule a): `if [ -z "${{ inputs.commit-status-github-token }}" ]` — input interpolated directly into shell condition.

Locations:

- `action.yml:158`
- `action.yml:205`
- `action.yml:230`
- `action.yml:244`
- `action.yml:268`
- `action.yml:273`
- `action.yml:292`
- `action.yml:299`
- `action.yml:410`
- `action.yml:418`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. **Resolve kube version step**: `kube_version` is set from `${{ inputs.kube-version }}` and `${{ steps.metadata.outputs.kube_version }}` (both untrusted), then written unsanitized: `echo "value=$kube_version" >> $GITHUB_OUTPUT`. An attacker can inject newlines to poison subsequent GITHUB_OUTPUT entries.

2. **Select GitHub Token for Sync Mode step**: Writes `${{ inputs.github-pat }}` and `${{ inputs.commit-status-github-token }}` directly to GITHUB_OUTPUT without sanitization: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT`. These are direct expression writes with no newline stripping.

3. **YQ Platform settings step (metadata)**: Writes `${output}` (derived from yq parsing of SSM-sourced data, which is externally controlled) to `$GITHUB_OUTPUT` in a loop: `echo "${output}" >> $GITHUB_OUTPUT` — no sanitization applied.

Locations:

- `action.yml:244`
- `action.yml:418`
- `action.yml:230`

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

Fixed all findings in action.yml:

1. **unpinned-uses**: Pinned all 10 unpinned action references to full 40-char SHA hashes with tag comments preserved.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions out of run: blocks into env: blocks. Shell commands now reference plain environment variables ($VAR_NAME). Affected steps: Parse git URL, Read platform context, Read platform metadata, Resolve kube version, Ensure argocd repo structure, Helmfile render, Build Helm Dependencies, Helm raw render, Push to Github, Select GitHub Token for Sync Mode.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all writes to $GITHUB_OUTPUT in: YQ Platform settings (metadata loop), Resolve kube version, and Select GitHub Token for Sync Mode steps.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four security findings in action.yml:
1. (script-injection, line 113) Moved ${{ runner.temp }} out of the run: block into an env: variable RUNNER_TEMP, then referenced it as "$RUNNER_TEMP/action-deps" in the shell script.
2. (script-injection, line 222) In the Helmfile render step, replaced bare $KUBE_VERSION_ARG with ${KUBE_VERSION_ARG:+--args="$KUBE_VERSION_ARG"} and bare $INPUT_HELMFILE_ARGS with ${INPUT_HELMFILE_ARGS:+"$INPUT_HELMFILE_ARGS"} to prevent shell metacharacter injection while handling empty values correctly.
3. (script-injection, line 265) In the Helm raw render step, replaced unquoted ${array[@]} with "${array[@]}", built a proper bash array VALUES_ARGS with individually quoted --values elements, and used ${INPUT_HELM_ARGS:+"$INPUT_HELM_ARGS"} and ${KUBE_VERSION_ARG:+"$KUBE_VERSION_ARG"} for optional arguments.
4. (github-env-injection, line 285) In the Get Webapp step, added sanitization: safe=$(printf '%s' "$WEBAPP_URL" | tr -d '\n\r') and wrote the sanitized value to $GITHUB_OUTPUT instead of the raw WEBAPP_URL.

