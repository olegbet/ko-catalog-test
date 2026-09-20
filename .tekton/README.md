# Konflux bundle publishing (`.tekton/`)

These Pipelines-as-Code `PipelineRun` files onboard this repository to Konflux so
that every task and pipeline is built and published as a versioned Tekton bundle.

Each catalog artifact this repo **owns** has two `PipelineRun`s — one for pull
requests (`-pull-request.yaml`) and one for merges to `main` (`-push.yaml`):

| Artifact | Source path | Component | Published image |
|----------|-------------|-----------|-----------------|
| `ko-oci-ta` task | `task/ko-oci-ta` | `task-ko-oci-ta` | `.../ko-catalog/ko-oci-ta` |
| `ko-build-oci-ta` pipeline | `pipelines/ko-build-oci-ta` | `pipeline-ko-build-oci-ta` | `.../ko-catalog/ko-build-oci-ta` |

Both use the shared [`pipelines/build-tekton-bundle.yaml`](pipelines/build-tekton-bundle.yaml)
pipeline (the `tkn-bundle-oci-ta` task turns the YAML in `path-context` into a
Tekton bundle image). The `dockerfile: Containerfile` param is inherited from the
generic build template and is ignored for bundle builds — no `Containerfile` is
needed.

> The `{{revision}}`, `{{source_url}}`, `{{git_auth_secret}}` etc. tokens are
> filled in by Pipelines-as-Code at runtime — leave them as-is.

## ⚠️ Placeholders to replace before onboarding

These files are scaffolded with placeholders. After the repo is onboarded to a
Konflux tenant, replace them everywhere under `.tekton/`:

| Placeholder | Replace with | Reference value in `container-build-catalog` |
|-------------|--------------|----------------------------------------------|
| `__ORG__` | GitHub org that hosts the repo (e.g. `openshift-pipelines` or `konflux-ci`) | `konflux-ci` |
| `__TENANT_NAMESPACE__` | Konflux tenant namespace (also the quay path segment) | `rhtap-build-tenant` |

The Konflux **Application** name is set to `ko-catalog` and the **Component**
names are `task-ko-oci-ta` and `pipeline-ko-build-oci-ta`. Change these only if
your onboarding uses different names — the `serviceAccountName`
(`build-pipeline-<component>`) and the quay image path must stay consistent with
the actual Konflux Components.

A quick way to apply the substitutions:

```bash
grep -rl '__ORG__\|__TENANT_NAMESPACE__' .tekton/ | \
  xargs sed -i 's/__ORG__/openshift-pipelines/g; s/__TENANT_NAMESPACE__/<your-tenant>/g'
```

## Onboarding checklist

1. Create the Konflux **Application** (`ko-catalog`) and one **Component** per
   artifact (`task-ko-oci-ta`, `pipeline-ko-build-oci-ta`) in the tenant namespace.
2. Install the Konflux GitHub app / configure Pipelines-as-Code on the repo.
3. Replace the placeholders above.
4. Merge — the `-push.yaml` runs publish the bundles with the commit revision as
   the tag; consumers reference them by digest.
