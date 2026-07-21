# git-directory-multicluster-generator

A Git **directory** generator that fans one app out across a
`configs/<app>/<cluster>/` folder hierarchy — the pattern you'd use to drive
**per-app, per-cluster** deployments from directory structure alone.

## How it works

`app-of-apps.yaml` deploys `argocd/appset.yaml`, an `ApplicationSet` that globs
two directory levels:

```yaml
generators:
- git:
    directories:
    - path: git-directory-multicluster-generator/configs/*/*
```

```
configs/
├── app1/cluster-1/configmap.yaml
├── app2/cluster-1/configmap.yaml
├── app2/cluster-2/configmap.yaml
├── app3/cluster-1/configmap.yaml
└── app3/cluster-2/configmap.yaml
```

Each `<app>/<cluster>` leaf becomes one Application, deployed into a namespace
named `<cluster>-<app>` so the fan-out is visible at a glance.

## Runs anywhere, scales to real multi-cluster

Out of the box every generated app targets the **local cluster** (`in-cluster`),
so the example runs on a single demo cluster with no cluster registration. The
folder names (`cluster-1`, `cluster-2`) are logical labels.

To make it a **true multi-cluster** deployment:

1. Register a cluster in Argo CD / Akuity named for each `<cluster>` folder.
2. In `argocd/appset.yaml`, switch the destination from `name: in-cluster` to
   `name: '{{.path.basename}}'` (the cluster folder name).
3. Add matching `destinations` entries to `argocd/project.yaml`.

## Usage

```sh
kubectl apply -f git-directory-multicluster-generator/app-of-apps.yaml
```
