# simple-argocd-playground

This repository is a small playground to experiment with Argo CD using either a Helm chart or plain Kubernetes manifests.

Purpose
- Quick experiments with Helm deployments (chart is in `helloworld_chart/`).
- Simple tests using plain Kubernetes manifests (folder `kube_manifests/`).
- Explore GitOps workflows by syncing this repository with Argo CD (UI or CLI).

Repository structure
- `helloworld_chart/`: minimal Helm chart (`Chart.yaml`, `templates/`).
- `kube_manifests/`: plain Kubernetes manifests (`deployment.yml`, `service.yml`).
- `app.yml`: example Argo CD Application manifest (currently points to the Helm chart).

Quick local deploy

- Using Helm (local, without Argo CD):
```bash
helm install helloworld ./helloworld_chart -n default --create-namespace
```

- Using plain manifests (local):
```bash
kubectl apply -f kube_manifests/
```

Using Argo CD

1) From the Argo CD UI
- Open the Argo CD UI (usually `https://<ARGOCD_SERVER>` or via port-forwarding: `kubectl port-forward svc/argocd-server -n argocd 8080:443`).
- Click **New App** (or **Create Application**).
- Fill in the fields:
	- `Application Name`: e.g. `simple-argocd-playground`.
	- `Project`: `default` (or your chosen Argo CD project).
	- `Repository URL`: the Git repo URL (e.g. `https://github.com/Ryad3/simple-argocd-playground.git`).
	- `Revision`: `HEAD` (or a specific branch/tag).
	- `Path`:
		- To deploy the Helm chart: `helloworld_chart`
		- To deploy plain manifests (no Helm): `kube_manifests`
	- `Destination`: `Cluster` = `https://kubernetes.default.svc`, `Namespace` = `default` (or your target namespace).
- Create the application and click **Sync** to apply resources to the cluster.

2) From the Argo CD CLI
- Create an application that points to the Helm chart:
```bash
argocd app create simple-argocd-helm \
	--repo https://github.com/Ryad3/simple-argocd-playground.git \
	--path helloworld_chart \
	--dest-server https://kubernetes.default.svc \
	--dest-namespace default

argocd app sync simple-argocd-helm
```

- Create an application that points directly to `kube_manifests` (deploy without Helm):
```bash
argocd app create simple-argocd-manifests \
	--repo https://github.com/Ryad3/simple-argocd-playground.git \
	--path kube_manifests \
	--dest-server https://kubernetes.default.svc \
	--dest-namespace default

argocd app sync simple-argocd-manifests
```

3) Declare the Argo CD Application as a manifest (optional)
- The file `app.yml` at the repository root is an example Argo CD `Application` manifest (it currently points to `helloworld_chart`). Apply it directly with:
```bash
kubectl apply -f app.yml -n argocd
```
- To use `kube_manifests` instead of the chart, edit `app.yml` and change `path: helloworld_chart` to `path: kube_manifests`, and update `metadata.name` if desired.

Notes & tips
- The Helm chart is configurable via `helloworld_chart/values.yaml` (image, replicas, `podLabels`, etc.).
- Chart helper templates and labels are defined in `helloworld_chart/templates/_helpers.tpl`.
- If Argo CD cannot access your repository, verify repository access (SSH/HTTPS) and that the Argo CD server has network access to the Git host.

Contributing
- Suggestions and fixes: open an issue or send a pull request.

---
Enjoy experimenting with Argo CD and Helm!

