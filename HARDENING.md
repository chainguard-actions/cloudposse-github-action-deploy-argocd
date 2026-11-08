<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.11.0** was hardened automatically. 37 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps that reference actions by tag instead of a full 40-character commit SHA: dcarbone/install-yq-action@v1.3.1 (line 136), mamezou-tech/setup-helmfile@v2.2.0 (line 143), actions/setup-node@v6 (line 151), cloudposse/github-action-yaml-config-query@v1.0.1 (lines 197, 286, 363), 1arp/create-a-file-action@0.4 (line 374), nick-fields/retry@v4 (line 396).

Locations:

- `action.yml:136`
- `action.yml:143`
- `action.yml:151`
- `action.yml:197`
- `action.yml:286`
- `action.yml:363`
- `action.yml:374`
- `action.yml:396`

### script-injection (severity: high)

Step 'Install git-url-parse' breaks rule (a): an expression is placed directly in the run: block: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`.

Locations:

- `action.yml:158`

### script-injection (severity: high)

Step 'Read platform context' breaks rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml ...` puts inputs straight into the shell command.

Locations:

- `action.yml:242`

### script-injection (severity: high)

Step 'Read platform metadata' breaks rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` puts an input straight into the shell command.

Locations:

- `action.yml:261`

### script-injection (severity: high)

Step 'Resolve kube version' breaks rule (a): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` put expressions straight into the shell.

Locations:

- `action.yml:277`

### script-injection (severity: high)

Step 'Ensure argocd repo structure' breaks rule (a): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`. This path is built from the inputs namespace and application.

Locations:

- `action.yml:300`

### script-injection (severity: high)

Step 'Helmfile render' breaks rule (a): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml`.

Locations:

- `action.yml:306`

### script-injection (severity: high)

Step 'Build Helm Dependencies' breaks rule (a): `helm dependency build ${{ inputs.path }}`.

Locations:

- `action.yml:321`

### script-injection (severity: high)

Step 'Helm raw render' breaks rule (a): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }} > ${{ steps.config.outputs.tmp }}/...`. It also breaks rule (b): `${array[@]}` and `${VALUES_STR}` are expanded without quotes and contain data from inputs.

Locations:

- `action.yml:327`

### script-injection (severity: high)

Step 'Get Webapp' breaks rule (a): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` is placed directly in the run: script.

Locations:

- `action.yml:356`

### script-injection (severity: high)

Step 'Push to Github' breaks rule (a). Its nick-fields/retry `command:` is a bash script, and it puts expressions in directly: `case '${{ inputs.operation }}' in`, `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}`, `git commit -m "Deploy ${{ github.repository }} ..."`, `git push origin ${{ steps.destination.outputs.ref }}`.

Locations:

- `action.yml:416`

### script-injection (severity: high)

Step 'Select GitHub Token for Sync Mode' breaks rule (a): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"` and `echo "token=${{ inputs.commit-status-github-token }}"` put inputs straight into the shell.

Locations:

- `action.yml:448`

### github-env-injection (severity: high)

Step 'Resolve kube version' writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` without sanitizing it first. $kube_version comes from inputs.kube-version or steps.metadata.outputs.kube_version, and the step does not run `printf '%s' ... | tr -d '\n\r'` before the write.

Locations:

- `action.yml:281`

### github-env-injection (severity: high)

Step 'Select GitHub Token for Sync Mode' writes inputs.github-pat or inputs.commit-status-github-token directly to $GITHUB_OUTPUT (`echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT`) without sanitizing out newlines.

Locations:

- `action.yml:449`

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

Pinned dcarbone/install-yq-action, mamezou-tech/setup-helmfile, actions/setup-node, cloudposse/github-action-yaml-config-query (3x), 1arp/create-a-file-action and nick-fields/retry to SHAs I looked up. Moved all ${{ }} expressions out of the run: scripts and the nick-fields/retry command into step env: (the retry child process inherits the step env), and the github-script step now reads the cluster input from process.env. helm-args and helmfile-args were pasted in as raw text before, so they are now split with the quote-aware xargs loop into arrays. values_file is split with IFS=', ' into an array of '--values file' pairs. The optional kube_version arg uses ${VAR:+"$VAR"}. Resolve kube version and Select GitHub Token now strip newlines with tr -d before writing to GITHUB_OUTPUT. Nothing was run or tested.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, 'YQ Platform settings' (metadata) and 'Get Webapp' (result) now pass each value through printf '%s' | tr -d '\n\r' before writing it to "$GITHUB_OUTPUT", which is now quoted. The yq extraction commands themselves are unchanged.

