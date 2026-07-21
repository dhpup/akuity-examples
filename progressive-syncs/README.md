# progressive-syncs

An `ApplicationSet` that rolls a change out **one environment at a time** —
`dev` → `stage` → `prod` — using the `RollingSync` strategy, so a bad change is
caught early instead of hitting every environment at once.

## How it works

`app-of-apps.yaml` deploys `argocd/progressive.yaml`. A list generator produces
one guestbook Application per environment, each labeled `envLabel: <env>`, and a
`RollingSync` strategy orders the waves by that label:

```yaml
strategy:
  type: RollingSync
  rollingSync:
    steps:
    - matchExpressions: [{key: envLabel, operator: In, values: [dev]}]
    - matchExpressions: [{key: envLabel, operator: In, values: [stage]}]
      maxUpdate: 0        # gate: stage waits until you allow it
    - matchExpressions: [{key: envLabel, operator: In, values: [prod]}]
      maxUpdate: 10%
```

Argo CD syncs `dev` first and only advances to the next step once the current
one is Healthy. `maxUpdate` controls how much of each wave updates at a time
(`0` pauses stage as a manual gate; `10%` throttles prod).

Each generated app deploys the
[`guestbook-deploy`](https://github.com/dhpup/guestbook-deploy) `env/<env>`
overlay into namespace `guestbook-<env>` on the local cluster.

## Usage

```sh
kubectl apply -f progressive-syncs/app-of-apps.yaml
```
