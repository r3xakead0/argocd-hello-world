# Argo CD Hello World with Minikube

This repository contains a minimal GitOps example that deploys the
`nginxdemos/hello` container to a local Minikube cluster using Argo CD.

Argo CD watches the Kubernetes manifests in `manifests/` and deploys them to
the `nginxdemos` namespace. The application is exposed with a `NodePort`
service on port `30080`.

## Repository Layout

```text
.
|-- application.yaml          # Argo CD Application definition
`-- manifests/
    |-- deployment.yaml       # Hello World Deployment
    `-- service.yaml          # NodePort Service
```

## Requirements

- Docker
- Minikube
- kubectl
- A GitHub repository reachable by Argo CD
- Optional: Argo CD CLI, if you want to manage Argo CD from the command line

## Start Minikube

Start a local Kubernetes cluster with the Docker driver:

```bash
minikube start --driver=docker
```

Confirm the cluster is ready:

```bash
kubectl get nodes
```

## Install Argo CD

Create the Argo CD namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD into the cluster:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for the Argo CD workloads to become ready:

```bash
kubectl get pods -n argocd
```

## Access the Argo CD UI

Forward the Argo CD API server to your local machine:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open the UI:

```text
https://localhost:8080
```

The default username is:

```text
admin
```

Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

## Deploy the Application

Apply the Argo CD `Application` resource:

```bash
kubectl apply -f application.yaml
```

The application definition points Argo CD to:

- Repository: `https://github.com/r3xakead0/argocd-hello-world.git`
- Revision: `main`
- Path: `manifests`
- Destination namespace: `nginxdemos`

The `CreateNamespace=true` sync option lets Argo CD create the `nginxdemos`
namespace automatically.

## Verify the Deployment

Check the Argo CD application:

```bash
kubectl get applications -n argocd
```

Inspect the application status:

```bash
kubectl describe application hello-world-app -n argocd
```

Check the Kubernetes resources created by Argo CD:

```bash
kubectl get all -n nginxdemos
```

If automated sync is working, the application should converge to a healthy and
synced state.

## Access the Hello World App

Open the service with Minikube:

```bash
minikube service hello-world-service -n nginxdemos
```

You can also access the configured NodePort directly:

```bash
minikube ip
```

Then open:

```text
http://<minikube-ip>:30080
```

## Stop Minikube

Stop the local cluster:

```bash
minikube stop
```

## Troubleshooting

If the Argo CD UI is not available, confirm the port-forward command is still
running and that the `argocd-server` service exists:

```bash
kubectl get svc -n argocd
```

If the application does not sync, check the application events:

```bash
kubectl describe application hello-world-app -n argocd
```

If the app namespace or resources are missing, verify that Argo CD can reach the
repository configured in `application.yaml` and that the `targetRevision` branch
exists.

If the service does not open, confirm the app pods are running:

```bash
kubectl get pods -n nginxdemos
```
