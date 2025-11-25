# Argo CD GitOps Demo – NGINX Application


### Overview

This repository demonstrates a simple GitOps workflow using **Argo CD** to deploy and manage an **NGINX application** on a Kubernetes cluster created with kind (Kubernetes in Docker).
Argo CD continuously monitors this Git repository and ensures the Kubernetes cluster always reflects the **desired state** stored in Git.


### Architecture Overview

```
Git Repository (K8 Manifests)
        ↓
Argo CD (GitOps Controller)
        ↓
Kubernetes Cluster (kind)
        ↓
Deployment → 2 Pods
Service    → ClusterIP
```

**Project Structure**

```
argocd-nginx-demo/
 ├── deployment.yaml      # NGINX Deployment (2 replicas)
 └── service.yaml         # ClusterIP Service
```

### Getting Started

#### 1. Create a local Kubernetes cluster using kind

```bash
kind create cluster --name argocd-demo
kubectl get nodes
```


#### 2. Install Argo CD

Create namespace:
```bash
kubectl create namespace argocd
```

Install Argo CD components:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check readiness:
```bash
kubectl get pods -n argocd
```
<img width="816" height="219" alt="Screenshot 2025-11-25 210233" src="https://github.com/user-attachments/assets/79471008-32c7-4062-9945-8aba8d7865f3" />


#### 3. Access the Argo CD UI

Port-forward Argo CD API server:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open UI: http://localhost:8080


#### 4. Connect Git Repository in Argo CD

In Argo CD UI:
    Settings -> Repositories -> Connect Repo
    Fill in:
      - Type: Git
      - URL: `https://github.com/<your-user>/argocd-nginx-demo.git`
      - Leave username/password empty for public repo
    Click **CONNECT**


#### 5. Create the Argo CD Application

Navigate to Applications -> **NEW APP**

**General**
    - Application Name: `nginx-demo`
    - Project: `default`
    - Sync Policy: `Manual` (or Auto later)

**Source**
    - Repository URL: choose your repo
    - Revision: `HEAD`
    - Path: / or the folder containing your manifests

**Destination**
    - Cluster: `https://kubernetes.default.svc`
    - Namespace: `default`

Click CREATE, then **SYNC**.

<img width="1830" height="680" alt="Screenshot 2025-11-25 223051" src="https://github.com/user-attachments/assets/17a5869c-1e9d-4fd4-9301-d1440fe50f1d" />


### Drift Detection

Argo CD continuously monitors the Kubernetes cluster and compares it against the **desired state** stored in Git.
If something in the cluster changes outside of Git (a manual edit, deleted pod, modified config), Argo CD identifies this as **configuration drift**.

To simulate drift:
```bash
kubectl delete pod <pod-name>
```

When the live cluster no longer matches the Git-defined state, Argo CD will:
    - Mark the application state as **OutOfSync**
    - Highlight the affected resource in red on the Argo CD UI
    - Record comparison events in the controller logs showing the detected difference
    - Automatically recreate the missing resource (if auto-sync is enabled), or prompt for a manual SYNC action

**Example Argo CD log**
<img width="946" height="68" alt="image" src="https://github.com/user-attachments/assets/331bc998-9183-46ae-8d1b-dfa80430718c" />
<img width="953" height="164" alt="image" src="https://github.com/user-attachments/assets/903a0289-542a-4ad6-bdf3-043c1aa87e38" />


### Learning Outcomes
        - Gained a solid understanding of GitOps principles and how they streamline Kubernetes application delivery.
        - Learned how Argo CD continuously reconciles the desired state in Git with the live state in the cluster.
        - Observed drift detection in action and saw how Argo CD restores configuration consistency through automated reconciliation.
        - Gained hands-on experience deploying workloads through the Argo CD UI and managing changes through Git-driven workflows.

