# gitops-production-cluster
"A production-grade GitOps pipeline using Kubernetes and Argo CD. This repository serves as the absolute single source of truth for both infrastructure configuration and application state, enforcing strict deployment immutability."

# ☸️ Autonomous GitOps: Zero-Touch Production Architecture

Welcome to my GitOps portfolio project. In modern cloud environments, relying on manual deployments or simple push pipelines leaves systems vulnerable to configuration drift and human error. 

This repository solves that by implementing a **pull-based reconciliation loop** using Kubernetes and Argo CD. This repository doesn't just store code—it acts as the absolute, immutable source of truth for the entire cluster state. If a configuration isn't documented here in Git, it does not exist in production.

## 🧠 The Engineering Philosophy

I built this to demonstrate enterprise-grade reliability and zero-touch deployment practices. By utilizing Argo CD's **App of Apps** pattern, this architecture actively monitors the live Kubernetes cluster and enforces the desired state defined in this repository. 

### Core Capabilities:
* **Instant Self-Healing:** If a compromised script or rogue administrator manually alters the cluster (e.g., deleting pods, changing replica counts, or modifying ports), the controller detects the drift and instantly overwrites the manual changes to match this repository.
* **Automated Pruning:** Resources removed from this Git repository are automatically deleted from the live cluster, preventing zombie workloads and silent resource leaks.
* **Zero-Touch Deployments:** 100% of the application state is declarative. No engineer ever needs to run `kubectl apply` in the production environment. 

## 🏗️ Architecture Blueprint

```
gitops-production-cluster/
├── apps/
│   └── frontend-web/
│       ├── deployment.yaml      # The workload definition (3 Replicas)
│       └── service.yaml         # Internal cluster networking
├── argo-config/
│   └── application.yaml         # The declarative sync instruction for Argo CD
└── README.md

```

If you want to pull this down and see the self-healing in action, you can run this entirely locally on Docker Desktop or Minikube.

1. Spin up the Environment
Ensure your local Kubernetes cluster is running, then install the Argo CD controller:

Bash
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

2. Access the Command Center
Forward the port to access the Argo CD visual dashboard:

Bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
URL: https://localhost:8080

Username: admin
Password: Retrieve your secure local password by running:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

3. Deploy the GitOps Controller
Apply the root configuration file. Argo CD will immediately take over, read this GitHub repository, and build the cluster:

Bash
kubectl apply -f argo-config/application.yaml

4. Test the Self-Healing (The Fun Part)
Open your terminal and attempt to manually break the production environment by scaling the deployment down to zero:

Bash
kubectl scale deployment frontend-webapp --replicas=0

Instantly check the Argo CD dashboard. You will see it detect the unauthorized state change (Configuration Drift) and automatically spin the pods back up to the required 3 replicas defined in this repository.
