# all-examples

A single **app-of-apps** that registers *every* example in this repo as a
child Argo CD `Application`, with **sync disabled**. Load this one app and then
choose which example to deploy — one sync click each — straight from the Argo CD
UI. Ideal for demos where you want the full menu on screen but only light up one
example at a time.

## How it works

```
all-examples (parent, manual sync)
└── all-examples/apps/*.yaml   ← one child Application per example
    ├── example-project-and-app                    (manual)
    ├── example-git-generator                      (manual)
    ├── example-sync-waves-and-hooks               (manual)
    └── … one per example …
```

- **The parent is manual.** Syncing it only *creates* the child `Application`
  objects — it deploys none of the example workloads.
- **Every child is manual too** (no `syncPolicy.automated`). Each shows up
  `OutOfSync`; nothing runs until you sync it.
- Each child points straight at that example's content, so it deploys in a
  **single sync** and cleanly prunes when you delete it.

## Usage

```sh
kubectl apply -f all-examples/app-of-apps.yaml
```

Then in the Argo CD UI:

1. Sync the **`all-examples`** app once → the `example-*` apps appear, all `OutOfSync`.
2. Sync whichever **`example-*`** app you want to demo → it deploys.
3. Delete (or leave un-synced) the ones you don't need.

> **Pick one entrypoint, not both.** Each example also has its own standalone
> `app-of-apps.yaml` (see the [root README](../README.md)) for deploying just
> that example. The `all-examples` children deploy the *same* underlying
> resources, so don't apply both the collection and a standalone entrypoint for
> the same example against one instance.

## Notes per example

| Child app | Deploys | Namespace |
|---|---|---|
| `example-*` (most) | that example's `argocd/` Argo CD resources | `argocd` |
| `example-sync-waves-and-hooks` | the `manifests/` workload | `sync-waves-demo` |
| `example-multi-source-application` | upstream podinfo chart + `$values` | `podinfo` |
| `example-workflows-infrastructure` | the full platform demo (see note) | `argocd` |

> **`example-workflows-infrastructure`** is heavier than the rest and requires
> the Argo CD instance to have `kustomize.buildOptions: --enable-helm` set (see
> the [root README](../README.md)). Syncing it creates the
> `workflows-infrastructure-demo` project and its child apps, which you then
> sync individually.
