<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.10.0** was hardened automatically. 39 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps that reference actions by tag instead of a full 40-character commit SHA: dcarbone/install-yq-action@v1.3.1; mamezou-tech/setup-helmfile@v2.2.0; actions/setup-node@v6; cloudposse/github-action-yaml-config-query@v1.0.1 (3 uses); actions/checkout@v6; 1arp/create-a-file-action@0.4; nick-fields/retry@v4; cloudposse/github-action-wait-commit-status@v0.2.1. Only actions/github-script is pinned to a SHA.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:200`
- `action.yml:282`
- `action.yml:359`
- `action.yml:370`
- `action.yml:383`
- `action.yml:441`

### script-injection (severity: high)

Step 'Install git-url-parse', rule (a): an expression is interpolated directly into run: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`.

Locations:

- `action.yml:154`

### script-injection (severity: high)

Step 'Read platform context', rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml ...` interpolates inputs directly into the shell.

Locations:

- `action.yml:238`

### script-injection (severity: high)

Step 'Read platform metadata', rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` interpolates an input directly into the shell.

Locations:

- `action.yml:257`

### script-injection (severity: high)

Step 'Resolve kube version', rule (a): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` are interpolated directly into run:.

Locations:

- `action.yml:273`

### script-injection (severity: high)

Step 'Ensure argocd repo structure', rule (a): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`. The output is built from inputs.namespace and inputs.application.

Locations:

- `action.yml:296`

### script-injection (severity: high)

Step 'Helmfile render', rule (a): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` interpolates inputs and step outputs directly into the shell.

Locations:

- `action.yml:302`

### script-injection (severity: high)

Step 'Build Helm Dependencies', rule (a): `helm dependency build ${{ inputs.path }}`.

Locations:

- `action.yml:317`

### script-injection (severity: high)

Step 'Helm raw render', rule (a): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }} > ${{ steps.config.outputs.tmp }}/...`. Rule (b): `${array[@]}` and `${VALUES_STR}` hold input-derived data and are expanded without quotes.

Locations:

- `action.yml:323`

### script-injection (severity: high)

Step 'Get Webapp', rule (a): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` is interpolated into run:. The output comes from inputs.namespace and inputs.application.

Locations:

- `action.yml:352`

### script-injection (severity: high)

Step 'Push to Github' (nick-fields/retry `command:`, which is run with shell: bash), rule (a): `pushd ./${{ steps.destination_dir.outputs.name }}`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `case '${{ inputs.operation }}' in`, `rm -rf ./.../${{ steps.config.outputs.path }}` and `git push origin ${{ steps.destination.outputs.ref }}` interpolate inputs and parsed-input outputs into a bash script. steps.destination.outputs come from parsing inputs.cluster.

Locations:

- `action.yml:390`

### script-injection (severity: high)

Step 'Select GitHub Token for Sync Mode', rule (a): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"` and `echo "token=${{ inputs.commit-status-github-token }}"` are interpolated directly into run:.

Locations:

- `action.yml:435`

### github-env-injection (severity: high)

Step 'Resolve kube version': `echo "value=$kube_version" >> $GITHUB_OUTPUT` writes a value taken from inputs.kube-version or steps.metadata.outputs.kube_version without first sanitizing it with printf '%s' | tr -d '\n\r'.

Locations:

- `action.yml:277`

### github-env-injection (severity: high)

Step 'Get Webapp': `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` writes a value read from manifests rendered from inputs (path, image, values, helm args) without first sanitizing it with printf '%s' | tr -d '\n\r'.

Locations:

- `action.yml:354`

### github-env-injection (severity: high)

Step 'Select GitHub Token for Sync Mode': `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` write inputs straight to GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:436`

### github-env-injection (severity: high)

Step 'YQ Platform settings' (id: metadata): `echo "${output}" >> $GITHUB_OUTPUT` writes key=value lines from SSM data fetched via inputs.ssm-path without sanitization. The loop also splits the yq output on unquoted words.

Locations:

- `action.yml:265`

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

Pinned 8 actions (yaml-config-query 3x) to SHAs, keeping the tag as a comment. Moved all inputs and step outputs used in run: blocks, and in the nick-fields/retry command, into step env: vars. github.* values now come from the default GITHUB_* env vars. Single values are double-quoted. helm-args and helmfile-args were inlined unquoted upstream, so they are now split with the quote-aware xargs method into arrays. values_file is still split with IFS=', ', and the result goes into an array of --values arguments. kube_version passes through ${VAR:+"$VAR"} for helm and through --args="$VAR" for helmfile, as upstream did. GITHUB_OUTPUT writes (kube version, webapp url, token, metadata loop) are sanitized with printf | tr -d '\n\r'. Also: github-script now reads inputs.cluster from process.env instead of inlining it in JS. In the retry script, the backticks in the 'invalid operation' error message were replaced with single quotes.

