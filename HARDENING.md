<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-deploy-argocd/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-deploy-argocd/v1.7.0** was hardened automatically. 37 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps that point to tags instead of 40-character commit SHAs: dcarbone/install-yq-action@v1.1.0, mamezou-tech/setup-helmfile@v2.2.0, actions/setup-node@v6, cloudposse/github-action-yaml-config-query@v1.0.1 (used 3 times), actions/checkout@v6, 1arp/create-a-file-action@0.2, nick-fields/retry@v4, cloudposse/github-action-wait-commit-status@v0.2.1.

Locations:

- `action.yml:132`
- `action.yml:139`
- `action.yml:147`
- `action.yml:193`
- `action.yml:200`
- `action.yml:282`
- `action.yml:357`
- `action.yml:368`
- `action.yml:380`
- `action.yml:438`

### script-injection (severity: high)

Rule (a), 'Install git-url-parse' step: `mkdir -p ${{ runner.temp }}/action-deps` and `cd ${{ runner.temp }}/action-deps` put an expression directly in run:.

Locations:

- `action.yml:153`

### script-injection (severity: high)

Rule (a), 'Read platform context' step: `chamber --verbose export ${{ inputs.ssm-path }}/${{ inputs.environment }} ...` puts inputs directly into the shell command.

Locations:

- `action.yml:237`

### script-injection (severity: high)

Rule (a), 'Read platform metadata' step: `chamber --verbose export ${{ inputs.ssm-path }}/_metadata ...`.

Locations:

- `action.yml:256`

### script-injection (severity: high)

Rule (a), 'Resolve kube version' step: `kube_version="${{ inputs.kube-version }}"`, `[ -n "${{ inputs.ssm-path }}" ]` and `kube_version="${{ steps.metadata.outputs.kube_version }}"`.

Locations:

- `action.yml:272`

### script-injection (severity: high)

Rule (a), 'Ensure argocd repo structure' step: `mkdir -p ${{ steps.config.outputs.tmp }}/manifests`. The output is built from inputs.namespace and inputs.application.

Locations:

- `action.yml:295`

### script-injection (severity: high)

Rule (a), 'Helmfile render' step: `helmfile --namespace ${{ inputs.namespace }} --environment ${{ inputs.environment }} --file ${{ inputs.path}} ... --args="${{ steps.arguments.outputs.kube_version }}" ${{ inputs.helmfile-args }} > ${{ steps.config.outputs.tmp }}/...`.

Locations:

- `action.yml:301`

### script-injection (severity: high)

Rule (a), 'Build Helm Dependencies' step: `helm dependency build ${{ inputs.path }}`.

Locations:

- `action.yml:316`

### script-injection (severity: high)

Rule (a), 'Helm raw render' step: `read ... <<< "${{ inputs.values_file }}"` and `helm template ${{ inputs.application }} ${{ inputs.path }} --set image.repository=${{ inputs.image }} ... ${{ inputs.helm-args }} ${{ steps.arguments.outputs.kube_version }}`. `${array[@]}` and `${VALUES_STR}` are also expanded unquoted.

Locations:

- `action.yml:322`

### script-injection (severity: high)

Rule (a), 'Get Webapp' step: the yq file argument `${{ steps.config.outputs.tmp }}/manifests/resources.yaml` is placed directly in the script.

Locations:

- `action.yml:347`

### script-injection (severity: high)

Rule (a), 'Push to Github' step: this is the nick-fields/retry `command`, which runs as a bash script. It interpolates `case '${{ inputs.operation }}' in`, `git reset --hard origin/${{ steps.destination.outputs.ref }}`, `rm -rf ./${{ steps.destination_dir.outputs.name }}/${{ steps.config.outputs.path }}` and `git commit -m "Deploy ${{ github.repository }} ..."` directly into the shell.

Locations:

- `action.yml:387`

### script-injection (severity: high)

Rule (a), 'Select GitHub Token for Sync Mode' step: `if [ -z "${{ inputs.commit-status-github-token }}" ]`, `echo "token=${{ inputs.github-pat }}"` and `echo "token=${{ inputs.commit-status-github-token }}"`.

Locations:

- `action.yml:431`

### github-env-injection (severity: high)

'Resolve kube version' step: `echo "value=$kube_version" >> $GITHUB_OUTPUT` writes a value taken from inputs.kube-version or steps.metadata.outputs.kube_version without the `printf '%s' | tr -d '\n\r'` step.

Locations:

- `action.yml:277`

### github-env-injection (severity: high)

'Select GitHub Token for Sync Mode' step: `echo "token=${{ inputs.github-pat }}" >> $GITHUB_OUTPUT` and `echo "token=${{ inputs.commit-status-github-token }}" >> $GITHUB_OUTPUT` write inputs without stripping newlines.

Locations:

- `action.yml:433`
- `action.yml:435`

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

Pinned 8 actions (10 uses: lines) to full commit SHAs, with the tag kept as a trailing comment. Moved every inline ${{ }} expression out of the run: scripts and the nick-fields/retry command and into step env vars, which are now referenced quoted. Upstream pasted helm-args and helmfile-args straight into the script, so the shell honoured quotes in them; they are now split into a bash array with the quote-aware xargs method (it includes the unmatched-quote check), so quoting is unchanged. values_file is still split on comma and space with IFS, then turned into an array of --values arguments. The optional kube_version flag uses ${VAR:+"$VAR"}. runner.temp is replaced with $RUNNER_TEMP. The kube version, webapp URL and selected token have \n and \r stripped before they are written to GITHUB_OUTPUT. I also passed inputs.cluster to the github-script step through process.env, so it is no longer pasted into the JS code.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'YQ Platform settings' step (id: metadata) in action.yml. Before, each key=value entry from _metadata.yaml went straight into $GITHUB_OUTPUT. Now each entry has newlines and carriage returns stripped (printf '%s' | tr -d '\n\r') first, is written with printf into a quoted "$GITHUB_OUTPUT", and is skipped if it contains '<<'. That stops an entry from starting a multi-line (heredoc-style) output block. The loop still splits the yq output on whitespace, as upstream does, so normal entries such as kube_version=... are written the same as before. Not tested by running the action.

