## Prerequisites

This demo installs tooling with the Helm + Kustomize pattern (kustomize
`helmCharts`), so the target Argo CD instance must have
`kustomize.buildOptions: --enable-helm` enabled. On Akuity Platform, set this in
the instance's Argo CD configuration before syncing. Without it the
`workflows`, `rollouts`, `events`, `external-secrets`, and `reloader` apps will
fail to render.

## One-Liners
curl https://raw.githubusercontent.com/dhpup/akuity-examples/main/demos/workflows-infrastructure/k3d.sh > k3d.sh && curl https://raw.githubusercontent.com/dhpup/akuity-examples/main/demos/workflows-infrastructure/app-of-apps.yaml > app-of-apps.yaml && chmod +x k3d.sh && ./k3d.sh akuity INSTANCE_NAME mac1

kubectl create ns external-secrets && gcloud secrets versions access latest --secret=gcpsm-secret | kubectl create secret generic gcpsm-secret -n external-secrets --from-file=secret-access-credentials=/dev/stdin

## Links
[Workflows UI](https://localhost:30000)

[Working App UI](http://localhost:30001)

[Erroring App UI](http://localhost:30002)
