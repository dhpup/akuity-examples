# sync-waves-and-hooks

Demonstrates the two mechanisms Argo CD gives you to control **ordering** during
a sync: **sync waves** (ordering *within* the Sync phase) and **resource hooks**
(work that runs in dedicated phases *around* the sync).

## What happens on sync

```
PreSync           Sync (by wave)                         PostSync
──────────    ─────────────────────────────────────    ──────────
db-migration  wave -1: app-config (ConfigMap)           smoke-test
  (Job)   ──▶ wave  0: whoami Deployment + Service  ──▶   (Job)
```

1. **PreSync hook** — `db-migration` Job runs to completion first (simulating a
   schema migration). If it fails, the sync stops and nothing else is applied.
2. **Sync, wave -1** — `app-config` ConfigMap is applied before anything consumes it.
3. **Sync, wave 0** — the `whoami` Deployment and Service are applied and must
   become Healthy.
4. **PostSync hook** — `smoke-test` Job runs only once everything is synced and
   healthy (simulating a post-deploy probe).

## Key annotations

| Annotation | Purpose |
|---|---|
| `argocd.argoproj.io/sync-wave: "<n>"` | Order within the Sync phase; lower waves apply first (can be negative). |
| `argocd.argoproj.io/hook: PreSync\|Sync\|PostSync\|SyncFail` | Run a resource in a specific phase. `SyncFail` runs only when a sync fails — useful for rollback/cleanup. |
| `argocd.argoproj.io/hook-delete-policy: HookSucceeded` | Garbage-collect the hook resource after it succeeds (also `BeforeHookCreation`, `HookFailed`). |

## Usage

```sh
kubectl apply -f sync-waves-and-hooks/app-of-apps.yaml
```

Then watch the resources appear in wave/hook order in the Argo CD UI. Everything
deploys into the `sync-waves-demo` namespace.
