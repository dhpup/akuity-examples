# git-files-generator

An `ApplicationSet` **Git files generator**: instead of scanning directories, it
reads config *files* from git and generates an Application per entry. Uses
`elementsYaml` to expand a list embedded inside each config file.

## How it works

`app-of-apps.yaml` deploys `argocd/git-files-generator.yaml`. It has two matrix
generators — one for prod, one for non-prod — each reading a config file and
fanning out over the `clusters:` list inside it via `elementsYaml`:

```yaml
generators:
- matrix:
    generators:
    - git:
        files:
        - path: git-files-generator/configs/prod-config.yaml
    - list:
        elementsYaml: "{{ .clusters | toJson }}"
```

```
configs/
├── prod-config.yaml       ← clusters: [{clusterName, env: prod}]
└── non-prod-config.yaml   ← clusters: [{clusterName, env: dev}, {…, env: stage}]
```

Each entry produces an Application `git-files-generator-<env>` that deploys the
[`guestbook-deploy`](https://github.com/dhpup/guestbook-deploy) `env/<env>`
overlay into namespace `git-files-generator-guestbook-<env>`. Edit a config file
to change what gets generated — no `ApplicationSet` changes needed.

## Usage

```sh
kubectl apply -f git-files-generator/app-of-apps.yaml
```
