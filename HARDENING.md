<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.6.0** was hardened automatically. 37 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps whose uses: lines point to tags instead of full 40-character commit SHAs: dcarbone/install-yq-action@v1.1.0, mamezou-tech/setup-helmfile@v2.2.0, actions/setup-node@v4, cloudposse/github-action-yaml-config-query@v1.0.1 (3 occurrences), actions/checkout@v6, 1arp/create-a-file-action@0.2, nick-fields/retry@v4, cloudposse/github-action-wait-commit-status@v0.2.1.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:200`
- `action.yml:282`
- `action.yml:357`
- `action.yml:368`
- `action.yml:381`
- `action.yml:439`

### script-injection (severity: high)

Step 'Install git-url-parse' breaks rule (a): an expression is placed straight into the run script: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps`.

Locations:

- `action.yml:154`

### script-injection (severity: high)

Step 'Read platform context' breaks rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml ...` places inputs straight into the shell script.

Locations:

- `action.yml:238`

### script-injection (severity: high)

Step 'Read platform metadata' breaks rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...` places an input straight into the shell script.

Locations:

- `action.yml:257`

### script-injection (severity: high)

Step 'Resolve kube version' breaks rule (a): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"` place expressions straight into the shell script.

Locations:

- `action.yml:273`

### script-injection (severity: high)

Step 'Ensure argocd repo structure' breaks rule (a): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests` places a step output (built from inputs) straight into the shell script.

Locations:

- `action.yml:296`

### script-injection (severity: high)

Step 'Helmfile render' breaks rule (a): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/manifests/resources.yaml` places expressions straight into the shell script.

Locations:

- `action.yml:302`

### script-injection (severity: high)

Step 'Build Helm Dependencies' breaks rule (a): `helm dependency build ${{ inputs.path }}`.

Locations:

- `action.yml:316`

### script-injection (severity: high)

Step 'Helm raw render' breaks rules (a) and (b). Rule (a): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }} > ${{ steps.config.outputs.tmp }}/...` place expressions straight into the script. Rule (b): `${array[@]}` and `${VALUES_STR}`, which hold input-derived data, are expanded without quotes.

Locations:

- `action.yml:322`

### script-injection (severity: high)

Step 'Get Webapp' breaks rule (a): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` is placed straight into the shell script.

Locations:

- `action.yml:350`

### script-injection (severity: high)

Step 'Select GitHub Token for Sync Mode' breaks rule (a): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"` and `echo "token=${{ inputs.commit-status-github-token }}"` place inputs straight into the shell script.

Locations:

- `action.yml:433`

### github-env-injection (severity: high)

Step 'Resolve kube version' writes `echo "value=$kube_version" >> $GITHUB_OUTPUT`, where kube_version comes from inputs.kube-version or steps.metadata.outputs.kube_version, without first sanitizing it with printf '%s' | tr -d '\n\r'.

Locations:

- `action.yml:277`

### github-env-injection (severity: high)

Step 'Get Webapp' writes `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` without sanitizing it with printf '%s' | tr -d '\n\r'. WEBAPP_URL is read from manifests rendered from the caller-supplied chart or helmfile (inputs.path).

Locations:

- `action.yml:352`

### github-env-injection (severity: high)

Step 'Select GitHub Token for Sync Mode' writes inputs.github-pat or inputs.commit-status-github-token directly to $GITHUB_OUTPUT (`echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT`) without sanitizing it with printf '%s' | tr -d '\n\r'.

Locations:

- `action.yml:434`
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

Pinned all 10 composite uses: refs to full SHAs, with the tag kept as a comment. Moved every ${{ }} expression in the flagged run: steps into env: and quoted single values (path, namespace, environment, application, image, tag, ssm-path, config tmp). helm-args and helmfile-args were pasted inline upstream, so they are split with the quote-aware xargs method (with the unmatched-quote check and an empty-value guard) and passed as a bash array. values_file still splits on commas and spaces with IFS=', ' read -a, but now reads from an env var and becomes a --values array. kube_version is passed as an optional single argument with ${VAR:+"$VAR"}, and as --args="$KUBE_VERSION_ARG" for helmfile. The kube_version, webapp_url and token outputs are cleaned with tr -d '\n\r' before they are written. The token step now reads from env. I also moved inputs.cluster in the github-script step to process.env. I left the nick-fields/retry command (a with: input, not flagged) unchanged.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Metadata loop now strips CR/LF from each value with printf|tr before writing it to "$GITHUB_OUTPUT" (redirect now quoted). In the nick-fields/retry 'Push to Github' step, every ${{ }} expression moved into the step's env: (DEST_DIR, DEST_REF, INPUT_OPERATION, CONFIG_PATH, GH_* context) and is used as double-quoted shell variables. nick-fields/retry passes process.env to the command it runs, so the variables reach the script. Upstream's backticked echo text was left as it was.

