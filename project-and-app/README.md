# project-and-app

The basics, and the smallest complete example in the repo: a single
**`AppProject`** plus a single **`Application`**, tied together by a parent
app-of-apps.

## How it works

`parent-app.yaml` is an `Application` that points at this directory's `argocd/`
folder and deploys the two resources it finds there:

| File | Resource | Purpose |
|---|---|---|
| `argocd/project.yaml` | `AppProject` `example-project` | The project the app below lives in. |
| `argocd/guestbook.yaml` | `Application` `guestbook` | Deploys the upstream [argocd-example-apps](https://github.com/argoproj/argocd-example-apps) guestbook into the `guestbook` namespace. |

This is the canonical **app-of-apps** shape: one bootstrap `Application` that
lays down a project and the apps that belong to it, so the whole unit is created
(and deleted) together.

## Usage

```sh
kubectl apply -f project-and-app/parent-app.yaml
```

Deploys onto the local cluster (`in-cluster`). For a tighter, least-privilege
take on `AppProject` scoping, see
[`multi-tenant-project-rbac`](../multi-tenant-project-rbac/).
