# multi-tenant-project-rbac

Two tenants (`project-a`, `project-b`), each a **least-privilege `AppProject`**
with its own scoped RBAC role and its own guestbook `Application`. This is the
reference for how to lock a tenant down to exactly what it needs — the opposite
of a wide-open `'*'` project.

## How it works

`app-of-apps.yaml` recurses `argocd/` and deploys:

```
argocd/
├── projects/
│   ├── project-a.yaml   ← AppProject, tenant A
│   └── project-b.yaml   ← AppProject, tenant B
└── apps/
    ├── guestbook-a.yaml ← Application in project-a → namespace guestbook-a
    └── guestbook-b.yaml ← Application in project-b → namespace guestbook-b
```

Each `AppProject` is scoped down to the minimum the guestbook actually needs:

| Constraint | Value |
|---|---|
| `sourceRepos` | only `guestbook-deploy` |
| `destinations` | only its own namespace (`guestbook-a` / `-b`) on `in-cluster` |
| `clusterResourceWhitelist` | only `Namespace` (for `CreateNamespace=true`) |
| `namespaceResourceWhitelist` | only `Service`, `Deployment`, `Ingress` — exactly what the guestbook renders |
| `roles` | a role that can only `get`/`sync` applications within its own project |

Try to point a tenant app at another repo, cluster, namespace, or resource kind
and Argo CD rejects it — that is the guarantee this example demonstrates.

## Usage

```sh
kubectl apply -f multi-tenant-project-rbac/app-of-apps.yaml
```
