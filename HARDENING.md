<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.8.0** was hardened automatically. 37 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

These composite steps in action.yml reference actions by tag instead of a full commit SHA: dcarbone/install-yq-action@v1.1.0 (L132), mamezou-tech/setup-helmfile@v2.2.0 (L139), actions/setup-node@v6 (L147), cloudposse/github-action-yaml-config-query@v1.0.1 (L193, L282, L358), actions/checkout@v6 (L200), 1arp/create-a-file-action@0.2 (L369), nick-fields/retry@v4 (L381), cloudposse/github-action-wait-commit-status@v0.2.1 (L439). Only actions/github-script is pinned to a SHA.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:200`
- `action.yml:282`
- `action.yml:358`
- `action.yml:369`
- `action.yml:381`
- `action.yml:439`

### script-injection (severity: high)

Step 'Install git-url-parse', rule (a): an expression is placed directly in the run script: `mkdir -p ${{ runner.temp }}/action-deps`.

Locations:

- `action.yml:154`

### script-injection (severity: high)

Step 'Read platform context', rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} --format yaml ...`.

Locations:

- `action.yml:238`

### script-injection (severity: high)

Step 'Read platform metadata', rule (a): `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`.

Locations:

- `action.yml:257`

### script-injection (severity: high)

Step 'Resolve kube version', rule (a): `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"`.

Locations:

- `action.yml:273`

### script-injection (severity: high)

Step 'Ensure argocd repo structure', rule (a): `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`.

Locations:

- `action.yml:296`

### script-injection (severity: high)

Step 'Helmfile render', rule (a): `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/...`.

Locations:

- `action.yml:302`

### script-injection (severity: high)

Step 'Build Helm Dependencies', rule (a): `helm dependency build ${{ inputs.path }}`.

Locations:

- `action.yml:317`

### script-injection (severity: high)

Step 'Helm raw render', rule (a): `IFS=', ' read -r -a array <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }}`. Rule (b) also applies: `${array[@]}` and `${VALUES_STR}` are expanded without quotes, and both hold data from inputs.values_file.

Locations:

- `action.yml:323`

### script-injection (severity: high)

Step 'Get Webapp', rule (a): `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` appears directly in the run script.

Locations:

- `action.yml:349`

### script-injection (severity: high)

Step 'Select GitHub Token for Sync Mode', rule (a): `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"` and `echo "token=${{ inputs.commit-status-github-token }}"`.

Locations:

- `action.yml:433`

### github-env-injection (severity: high)

Step 'Resolve kube version' writes `echo "value=$kube_version" >> $GITHUB_OUTPUT` without sanitizing. kube_version comes from inputs.kube-version or steps.metadata.outputs.kube_version, and nothing runs `printf '%s' | tr -d '\n\r'` on it first.

Locations:

- `action.yml:277`

### github-env-injection (severity: high)

Step 'Get Webapp' writes `echo "webapp_url=${WEBAPP_URL}" >> $GITHUB_OUTPUT` without sanitizing. WEBAPP_URL is read from manifests rendered from inputs the caller controls (chart path, values, image), so a newline in it could inject extra outputs.

Locations:

- `action.yml:353`

### github-env-injection (severity: high)

Step 'Select GitHub Token for Sync Mode' writes `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` without the newline-stripping step.

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

All changes are in action.yml. Nothing was run or tested; the fixes were checked only by reading the file.

- **Pinned actions:** I pinned 8 actions (10 `uses:` lines) to full commit SHAs and kept the tag as a trailing comment.
- **Moved expressions:** every `${{ }}` expression in the flagged run steps now sits in the step's `env:` and is used as a quoted variable. This covers the install, chamber, kube-version, mkdir, helmfile, helm dependency build, helm template, get-webapp and token-select steps.
- **Argument lists:** upstream pasted helm-args and helmfile-args straight into the script, so the shell honoured quotes inside them. They are now split the same quote-aware way (the guarded xargs-to-array form) so the commands still get the same arguments.
- **values_file:** still split on commas and spaces as before, but now builds a `--values <file>` array instead of an unquoted string. Empty entries are skipped, as the old unquoted form did.
- **kube_version argument:** passed as `${KUBE_VERSION_ARG:+"$KUBE_VERSION_ARG"}` so it disappears entirely when empty.
- **Output sanitising:** the kube_version, webapp_url and token outputs have newlines stripped before they are written to $GITHUB_OUTPUT.
- **Extra fix (not in the findings):** the github-script step now reads the cluster from `process.env.CLUSTER` instead of having `inputs.cluster` pasted into the JavaScript.
- **Left as is:** the `command:` passed to nick-fields/retry was not flagged and still contains inline expressions.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

I fixed both findings in action.yml. Nothing was run or tested; I only re-read the edited script to confirm no ${{ }} expressions are left in it.

- **github-env-injection** (the 'YQ Platform settings' step with id metadata): the yq output from ./_metadata.yaml is now saved to a variable first. Each entry has '\r' and '\n' stripped with printf | tr before it is written, and "$GITHUB_OUTPUT" is now quoted. The yq call ends in '|| true': upstream ran yq inside the for loop's word list, so a yq failure never stopped the step, and this keeps that. The loop still splits on whitespace as before.
- **script-injection** (the 'Push to Github' step, nick-fields/retry): every ${{ }} expression in the command script now lives in the step's env: block as DEST_DIR, DEST_REF, OPERATION, CONFIG_PATH, REPOSITORY_NAME, COMMIT_SHA, RUN_ID and RUN_ATTEMPT. The script uses them as double-quoted shell variables, and $GITHUB_OUTPUT is quoted there too. This relies on retry passing the step's environment through to the bash it starts, which child processes normally inherit.

I left one upstream quirk alone: the error message for an unknown operation has backticks around `deploy` and `destroy`, so bash tries to run those words as commands. It's a literal in the script, not an expression, so it can't be injected through.

