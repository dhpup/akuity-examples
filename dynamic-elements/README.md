# dynamic-elements

An `ApplicationSet` **matrix + list generator** that builds its list elements
*dynamically* from a YAML file in git using `elementsYaml`, rather than hard-coding
them inline. Handy when the set of things to deploy is itself data under version
control.

## How it works

`app-of-apps.yaml` deploys `argocd/dynamic-elements.yaml`. Its matrix combines a
Git **files** generator with a list generator whose elements come from a field
*inside* the file the git generator read:

```yaml
generators:
- matrix:
    generators:
    - git:
        repoURL: https://github.com/argoproj/argo-cd.git
        files:
        - path: applicationset/examples/list-generator/list-elementsYaml-example.yaml
    - list:
        elements: []
        elementsYaml: "{{ .key.components | toJson }}"   # expand a list read from the file
```

Each element in that file's `components` list becomes an Application that
installs a Helm chart (`chart` / `repoUrl` / `version` / `releaseName` from the
data) into the element's namespace. Change the data file and the generated set
changes with it.

## Usage

```sh
kubectl apply -f dynamic-elements/app-of-apps.yaml
```
