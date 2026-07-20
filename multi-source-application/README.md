# multi-source-application

Demonstrates an Argo CD **multi-source Application**: a Helm chart pulled from
its upstream chart repo, configured with a `values.yaml` that lives in a
**different** git repo. This is the idiomatic way to consume a third-party chart
without forking or vendoring it, while keeping your configuration under version
control.

## How it works

A single `Application` lists two entries under `spec.sources`:

| Source | Role |
|---|---|
| `stefanprodan.github.io/podinfo` (chart) | The upstream Helm chart. |
| `github.com/dhpup/akuity-examples` (`ref: values`) | This repo, attached only to supply the values file. |

The chart source points its `helm.valueFiles` at the other source using the
`$values` reference:

```yaml
sources:
- repoURL: https://stefanprodan.github.io/podinfo
  chart: podinfo
  targetRevision: 6.14.0
  helm:
    valueFiles:
    - $values/multi-source-application/values/podinfo-values.yaml
- repoURL: https://github.com/dhpup/akuity-examples.git
  targetRevision: main
  ref: values          # <-- names this source "values"; used by $values above
```

The `ref` source renders nothing itself (it has no `path` or `chart`); it exists
purely so the chart can read a values file from it. This lets you upgrade the
chart (`targetRevision`) and change your config independently.

## Usage

```sh
kubectl apply -f multi-source-application/application.yaml
```

Deploys podinfo into the `podinfo` namespace with the values from
[`values/podinfo-values.yaml`](values/podinfo-values.yaml).

> **Note:** multi-source Applications require Argo CD v2.6+.
