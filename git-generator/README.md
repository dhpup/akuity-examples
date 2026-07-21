# git-generator

An `ApplicationSet` **matrix generator** that combines a Git **directory**
generator with a **list** generator: one Application per `configs/*` directory,
multiplied by each entry in a small inline list.

## How it works

`app-of-apps.yaml` deploys `argocd/git-generator.yaml`, an `ApplicationSet` whose
generator is a matrix of two:

```yaml
generators:
- matrix:
    generators:
    - git:                          # one element per configs/* directory
        directories:
        - path: git-generator/configs/*
    - list:                         # multiplied by this list
        elements:
        - clusterName: in-cluster
          branch: main
```

Each `configs/<name>/` directory (`example-a`, `example-b`) becomes an
Application named `git-generator-<name>-in-cluster`, deploying that directory's
manifests into namespace `<name>-main`. Add a directory under `configs/` and a
new app appears automatically — no manifest edits.

## Usage

```sh
kubectl apply -f git-generator/app-of-apps.yaml
```
