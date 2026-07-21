# project-quickstart

A self-service **project onboarding** pattern: a GitHub Action scaffolds a new
project (an `AppProject` + a project-scoped app-of-apps) from templates, so teams
can request an Argo CD project via a form instead of hand-writing manifests.

## How it works

```
templates/                       ← scaffolding, with REPLACE_ME_* placeholders
├── project.yaml                 ← AppProject template
└── application.yaml             ← per-project app-of-apps template
argocd/
├── projects/
│   └── project-quickstart-demo.yaml   ← umbrella AppProject the scaffolds land in
│   └── <new projects rendered here>
└── apps/                        ← generated project app-of-apps land here
app-of-apps.yaml                 ← deploys everything under argocd/
```

1. The [`create-project`](../.github/workflows/create-project.yaml) GitHub Action
   takes the templates in `templates/`, substitutes the `REPLACE_ME_*`
   placeholders (name, repo), and commits the rendered manifests under `argocd/`.
2. `app-of-apps.yaml` recurses `argocd/` and deploys whatever has been scaffolded
   there — the umbrella `project-quickstart-demo` project plus any generated
   projects/apps.

## Usage

```sh
kubectl apply -f project-quickstart/app-of-apps.yaml
```

Then trigger the `create-project` workflow to scaffold a new project. The
bootstrap app uses `prune: true` + `allowEmpty: true` so removing a scaffold from
git cleanly removes it from the cluster.
