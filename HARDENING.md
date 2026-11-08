<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.1** was hardened automatically. 38 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps that reference actions by mutable tag instead of a 40-char commit SHA: line 132 `dcarbone/install-yq-action@v1.3.1`; line 139 `mamezou-tech/setup-helmfile@v2.2.0`; line 147 `actions/setup-node@v6`; lines 193, 282, 359 `cloudposse/github-action-yaml-config-query@v1.0.1`; line 370 `1arp/create-a-file-action@0.4`; line 383 `nick-fields/retry@v4`.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:282`
- `action.yml:359`
- `action.yml:370`
- `action.yml:383`

### script-injection (severity: high)

(a) Step 'Install git-url-parse' puts an expression straight into run: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`.

Locations:

- `action.yml:154`

### script-injection (severity: high)

(a) Step 'Read platform context' puts inputs straight into run: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...`.

Locations:

- `action.yml:238`

### script-injection (severity: high)

(a) Step 'Read platform metadata' puts an input straight into run: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`.

Locations:

- `action.yml:257`

### script-injection (severity: high)

(a) Step 'Resolve kube version' puts expressions straight into run: `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]`, `kube_version="${{ steps.metadata.outputs.kube_version }}"`.

Locations:

- `action.yml:273`

### script-injection (severity: high)

(a) Step 'Ensure argocd repo structure' puts a step output straight into run: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`.

Locations:

- `action.yml:296`

### script-injection (severity: high)

(a) Step 'Helmfile render' puts expressions straight into run, unquoted: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml`.

Locations:

- `action.yml:302`

### script-injection (severity: high)

(a) Step 'Build Helm Dependencies' puts an input straight into run: `helm dependency build ${{ inputs.path }}`.

Locations:

- `action.yml:317`

### script-injection (severity: high)

(a) Step 'Helm raw render' puts many expressions straight into run: `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"`, `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }} > ${{ steps.config.outputs.tmp }}/...`. (b) It also expands `${array[@]}` and `${VALUES_STR}` unquoted, and both are built from inputs.values_file.

Locations:

- `action.yml:323`

### script-injection (severity: high)

(a) Step 'Get Webapp' puts a step output straight into run: `${{ steps.config.outputs.tmp }}/manifests/resources.yaml`.

Locations:

- `action.yml:352`

### script-injection (severity: high)

(a) Step 'Select GitHub Token for Sync Mode' puts inputs straight into run: `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"`, `echo "token=${{ inputs.commit-status-github-token }}"`.

Locations:

- `action.yml:435`

### github-env-injection (severity: high)

Step 'YQ Platform settings' (id: metadata) writes values taken from SSM-exported metadata (the SSM path comes from inputs.ssm-path) to $GITHUB_OUTPUT with no sanitization: `echo "${output}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:265`

### github-env-injection (severity: high)

Step 'Resolve kube version' writes a value derived from inputs.kube-version / steps.metadata.outputs.kube_version to $GITHUB_OUTPUT without `printf '%s' | tr -d '\n\r'`: `echo "value=$kube_version" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:277`

### github-env-injection (severity: high)

Step 'Get Webapp' writes a value read from manifests rendered from input-controlled charts and values to $GITHUB_OUTPUT with no sanitization: `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:354`

### github-env-injection (severity: high)

Step 'Select GitHub Token for Sync Mode' writes inputs straight to $GITHUB_OUTPUT with no sanitization: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:436`

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

Every listed finding is fixed in action.yml. Nothing was run to test the changes.

- **Unpinned actions:** All 8 tag-based `uses:` references are now pinned to the full commit SHA returned by lookup_action_sha, with the tag kept as a comment. That covers install-yq-action, setup-helmfile, setup-node, the three yaml-config-query steps, create-a-file-action and retry.
- **Script injection:** Each `${{ }}` expression in the listed `run:` steps moved to the step's `env:` block and is referenced as a quoted shell variable.
  - `helmfile-args` and `helm-args` are lists that upstream pasted in directly, so the shell honoured quotes in them. They are now split with the quote-aware xargs method into an array, guarded so an empty value adds no argument.
  - `values_file` is still split on commas and spaces the way upstream did, but each file becomes its own `--values <file>` pair instead of an unquoted string.
  - The optional `kube_version` flag uses `${KUBE_VERSION_ARG:+"$KUBE_VERSION_ARG"}` for helm, so an empty value adds no argument. For helmfile it stays inside `--args="..."` as upstream had it.
- **Output injection:** The values written to `$GITHUB_OUTPUT` (the metadata key=value lines, the kube version, `webapp_url` and the selected token) now have newlines stripped with `printf '%s' | tr -d '\n\r'` first.
- **Extra fix, not in the findings:** `inputs.cluster` was pasted straight into the github-script JavaScript (`run("${{ inputs.cluster }}")`). It now goes through an env var and is read with `process.env.INPUT_CLUSTER_URL`.
- **Left as is:** The `nick-fields/retry` `command:` block still has inline expressions. It wasn't in the findings and I left it unchanged.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In the 'Push to Github' step (nick-fields/retry, shell: bash), the expressions for destination_dir name, destination ref, inputs.operation, config path, github.repository/sha/run_id/run_attempt now come in through the step's env: block (DEST_DIR_NAME, DEST_REF, DEPLOY_OPERATION, CONFIG_PATH, DEPLOY_*). The command script uses them as double-quoted variables, so pushd, reset, case, cp, rm, commit and push work the same but can't be injected into. $GITHUB_OUTPUT is quoted too. The step's env: is inherited by the child shell that retry spawns. Upstream's literal backticks in the error echo were left unchanged.

