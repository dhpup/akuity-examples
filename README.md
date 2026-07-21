# akuity-examples

A collection of Argo CD / ApplicationSet examples for Akuity Platform demos. Each
top-level directory is a self-contained example with an `app-of-apps.yaml` that you
point Argo CD at; the `argocd/` (and `configs/`) subdirectories hold the manifests it
deploys.

> Most examples self-reference this repo (`github.com/dhpup/akuity-examples`) as their
> source, and pull sample workloads from `github.com/dhpup/guestbook-deploy` (which has
> `env/dev`, `env/stage`, and `env/prod` overlays).

> Every example targets the built-in local cluster (`in-cluster`) so it runs on any
> Argo CD / Akuity instance without registering extra clusters. Where an example is
> *about* multiple clusters (e.g. [`git-directory-multicluster-generator`](git-directory-multicluster-generator/)),
> its README shows the one-line switch to a real multi-cluster setup.

## Deploy everything, pick in the UI

The [`all-examples`](all-examples/) app-of-apps registers **every** example
below as a child `Application` with **sync disabled**. Load it once, then choose
which example to deploy — one sync click each — from the Argo CD UI. This is the
easiest way to run a demo: the full menu is on screen, but nothing deploys until
you pick it.

```sh
kubectl apply -f all-examples/app-of-apps.yaml
```

See [`all-examples/README.md`](all-examples/) for how it works. To deploy just a
single example instead, use that example's own `app-of-apps.yaml` (below) — but
don't apply both entrypoints for the same example against one instance.

## Examples

Each directory below has its own `README.md` with a full walkthrough.

| Directory | What it demonstrates |
|---|---|
| [`project-and-app`](project-and-app/) | The basics: a single `AppProject` plus a guestbook `Application`, tied together by a parent app-of-apps. |
| [`project-quickstart`](project-quickstart/) | Templates (`templates/project.yaml`, `templates/application.yaml`) scaffolded into new projects by the [`create-project`](.github/workflows/create-project.yaml) GitHub Action. |
| [`multi-tenant-project-rbac`](multi-tenant-project-rbac/) | Two `AppProject`s (`project-a`, `project-b`) with scoped RBAC roles, each with its own guestbook `Application`. |
| [`git-generator`](git-generator/) | `ApplicationSet` matrix generator using a Git **directory** generator over `configs/*`. |
| [`git-directory-multicluster-generator`](git-directory-multicluster-generator/) | Git directory generator (`configs/*/*`) that fans one app out across multiple clusters. |
| [`git-files-generator`](git-files-generator/) | `ApplicationSet` Git **files** generator driven by `prod-config.yaml` / `non-prod-config.yaml`. |
| [`pr-generator`](pr-generator/) | `ApplicationSet` pull-request generator that spins up preview apps for PRs labeled `preview`. |
| [`dynamic-elements`](dynamic-elements/) | `ApplicationSet` matrix + list generator using `elementsYaml` to build elements dynamically from a Git file. |
| [`progressive-syncs`](progressive-syncs/) | `ApplicationSet` with progressive sync rollout ordering across `dev` → `stage` → `prod`. |
| [`sync-waves-and-hooks`](sync-waves-and-hooks/) | Ordering a sync with **sync waves** plus **PreSync/PostSync resource hooks** (migration + smoke-test Jobs). |
| [`multi-source-application`](multi-source-application/) | A **multi-source** `Application`: an upstream Helm chart configured by a `values.yaml` from a separate git repo via `$values`. |
| [`demos/workflows-infrastructure`](demos/workflows-infrastructure/) | Full platform demo: installs Argo Events, Argo Workflows, Argo Rollouts, External Secrets, and Reloader, plus a cluster-upgrader and a post-sync workflow-validation example. See its [readme](demos/workflows-infrastructure/readme.md). |

## Usage

Point an Argo CD `Application` at an example's `app-of-apps.yaml`, or apply it directly:

```sh
kubectl apply -f <example>/app-of-apps.yaml
```

The `demos/workflows-infrastructure` example has its own bootstrap (`k3d.sh`) and
one-liners documented in its readme.

## Installing tooling: the Helm + Kustomize pattern

The `demos/workflows-infrastructure` example installs every third-party tool
(Argo Workflows, Argo Rollouts, Argo Events, External Secrets, Reloader) with a
single consistent pattern: **Helm for packaging/templating, Kustomize for
lifecycle and last-mile changes** — the approach from
[akuity/helm-charts](https://github.com/akuity/helm-charts). Nothing is vendored.

Each tool lives in its own `configs/<tool>/` directory:

```
configs/workflows/
├── kustomization.yaml   # helmCharts: pins the upstream chart + version
├── values.yaml          # chart configuration (what you'd pass to `helm --values`)
└── rbac.yaml            # your own resources, layered on via resources:
```

```yaml
# configs/workflows/kustomization.yaml
helmCharts:
- name: argo-workflows
  repo: https://argoproj.github.io/argo-helm
  version: 1.0.20
  releaseName: argo-workflows
  namespace: argo
  includeCRDs: true
  valuesFile: values.yaml
resources:
- rbac.yaml
```

The Argo CD `Application` then simply points its source `path:` at that
directory. To upgrade a tool, bump the `version:` in one `kustomization.yaml`.
Your own custom resources (Argo Events `EventSource`/`Sensor`/`EventBus`, the
External Secrets `ClusterSecretStore`, etc.) stay as readable files, added via
`resources:`.

> **Prerequisite:** Kustomize's `helmCharts` field requires `kustomize build
> --enable-helm`. In Argo CD this means the instance must have
> `kustomize.buildOptions: --enable-helm` set (in the `argocd-cm` ConfigMap, or
> the equivalent Akuity Platform instance setting). Without it these apps will
> not render.

## Pinned component versions

Pinned in `demos/workflows-infrastructure/configs/<tool>/kustomization.yaml`:

| Component | Helm chart | Chart version | App version |
|---|---|---|---|
| Argo Rollouts | `argo/argo-rollouts` | `2.41.1` | `v1.9.1` |
| Argo Workflows | `argo/argo-workflows` | `1.0.20` | `v4.0.7` |
| Argo Events | `argo/argo-events` | `2.4.23` | `v1.9.11` |
| External Secrets | `external-secrets/external-secrets` | `2.8.0` | `v2.8.0` (API `external-secrets.io/v1`) |
| Reloader | `stakater/reloader` | `2.2.14` | `v1.4.19` |
