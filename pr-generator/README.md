# pr-generator

An `ApplicationSet` **pull-request generator** that spins up an ephemeral
**preview app** for every open PR carrying the `preview` label, and tears it down
when the PR closes.

## How it works

`app-of-apps.yaml` deploys `argocd/project.yaml` (the `pr-generator-demo`
project) and `argocd/appset.yaml`, whose generator polls GitHub:

```yaml
generators:
- pullRequest:
    github:
      owner: dhpup
      repo: akuity-examples
      tokenRef:
        secretName: application-set-secret
        key: github-token
      labels:
      - preview            # only PRs with this label get a preview app
    requeueAfterSeconds: 15
```

Each matching PR yields an Application `pr-generator-<branch>-<number>` that
deploys `pr-generator/configs` at that PR's `head_sha` into the `pr-generator`
namespace. Close the PR (or remove the label) and the app is pruned.

## Prerequisites

Create the token secret the generator reads from:

```sh
kubectl create secret generic application-set-secret \
  -n argocd \
  --from-literal=github-token=<a GitHub token with repo/PR read access>
```

## Usage

```sh
kubectl apply -f pr-generator/app-of-apps.yaml
```

Then open a PR against the repo and add the `preview` label.
