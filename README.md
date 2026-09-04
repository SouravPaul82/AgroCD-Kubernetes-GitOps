# AgroCD Kubernetes GitOps

Production-grade GitOps deployment pipeline using Argo CD for automated Kubernetes application delivery, featuring integrated custom dashboards for end-to-end cluster observability and real-time monitoring.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quickstart (kind + Argo CD)](#quickstart-kind--argo-cd)
- [Deploy with Argo CD (example)](#deploy-with-argo-cd-example)
- [Accessing the application & dashboards](#accessing-the-application--dashboards)
- [Repository layout](#repository-layout)
- [Screenshots](#screenshots)
- [Next improvements](#next-improvements)
- [Contributing](#contributing)
- [License & Contact](#license--contact)

## Overview

This repository contains the manifests and configuration to deploy a sample Voting App to a Kubernetes cluster using Argo CD for GitOps-based continuous delivery. The project demonstrates:

- GitOps automation with Argo CD
- Kubernetes manifests and example cluster configuration
- Local/test cluster setup using kind
- Custom observability dashboards (Grafana/Prometheus) for application and cluster metrics

The Voting App is a simple multi-service example (vote + result) used to demonstrate multi-service deployments and automated rollout via Argo CD.

## Features

- Declarative application delivery with Argo CD
- One-click sync & automated rollouts
- Local development support with kind
- Pre-built Grafana dashboards for runtime observability

## Architecture

High level components:

- Argo CD: GitOps continuous delivery controller that watches this repo and applies manifests to the cluster.
- Kubernetes cluster (kind / minikube / cloud): runs the voting app and monitoring stack.
- Voting App: frontend and backend services (vote, result) exposed as ClusterIP/LoadBalancer depending on environment.
- Observability: Prometheus for metrics collection and Grafana for dashboards (custom dashboards included in this repo).

## Prerequisites

- Git
- Docker (for kind)
- kind (https://kind.sigs.k8s.io/)
- kubectl (https://kubernetes.io/docs/tasks/tools/)
- Argo CD (for the GitOps sync) — you can use the CLI (argocd) or the web UI

## Quickstart (kind + Argo CD)

Below is a minimal example to get the stack running locally using kind and Argo CD.

1. Create a kind cluster:

   ```bash
   kind create cluster --name argocd-demo
   kubectl cluster-info --context kind-argocd-demo
   ```

2. Install Argo CD into the cluster:

   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. (Optional) Install a monitoring stack (Prometheus/Grafana) or use the manifests in this repo if provided under `k8s-specifications/`.

4. Port-forward Argo CD server to access the UI:

   ```bash
   kubectl -n argocd port-forward svc/argocd-server 8080:443
   # Open: https://localhost:8080
   ```

5. Retrieve the initial Argo CD admin password and login (CLI):

   ```bash
   # initial password is stored in a secret
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

   # login with argocd CLI (install argocd CLI first)
   argocd login localhost:8080 --insecure
   ```

## Deploy with Argo CD (example)

You can point Argo CD to this repository (root or `k8s-specifications/`) as the application source. Example using the argocd CLI:

```bash
argocd app create voting-app \
  --repo https://github.com/SouravPaul82/AgroCD-Kubernetes-GitOps.git \
  --path k8s-specifications \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Sync the application to deploy
argocd app sync voting-app
```

If you prefer the UI, create a new Application in Argo CD and set:
- Repository URL: https://github.com/SouravPaul82/AgroCD-Kubernetes-GitOps.git
- Path: k8s-specifications (or `/` for root)
- Destination cluster: in-cluster
- Destination namespace: default (or your preferred namespace)

## Accessing the application & dashboards

Find services and port-forward to view the app and Grafana:

```bash
# list services in the namespace where app is deployed
kubectl get svc -n default

# port-forward an example service to localhost (replace <service-name> and <port>)
kubectl -n default port-forward svc/<service-name> 8080:<target-port>

# port-forward Grafana (if installed in monitoring namespace)
kubectl -n monitoring port-forward svc/grafana 3000:3000
# Open http://localhost:3000
```

Note: service names and namespaces depend on the manifests you use. Use `kubectl get all -A` to discover deployed resources.

## Repository layout

- README.md - this file
- k8s-specifications/ - Kubernetes manifests and configuration (Argo CD application manifests live here)
- kind-cluster/ - kind cluster helper files and examples
- *.png - project screenshots (Argo CD UI, app pages, dashboards)

## Screenshots

Argo CD web UI showing application sync and health:

![Argo CD UI](./ArgoCD.png)

Application UI - voting page:

![Voting App - Vote Page](./app_iamge_1_voting.png)

Application UI - results page:

![Voting App - Results Page](./app_image_2_result.png)

Observability dashboards (Grafana / custom dashboards):

![Dashboards](./Dashboard.png)

## Next improvements

- Move screenshots into `docs/images/` and reference them from README for a cleaner root.
- Add step-by-step `kind` + Argo CD setup scripts and a Makefile for automation.
- Add concrete service names and port-forward examples specific to the manifests in `k8s-specifications/`.
- Add badges (build, Argo CD sync status) and a LICENSE file.

## Contributing

Contributions are welcome — open an issue or submit a pull request with improvements.

## License & Contact

Specify a license for the project (for example MIT). Add a LICENSE file to the repo.

Maintainer: @SouravPaul82

