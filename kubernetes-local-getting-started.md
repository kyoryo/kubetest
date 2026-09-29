# Getting Started with Kubernetes Locally

This guide sets up a local Kubernetes environment for learning and development.

The recommended stack is:

- **Docker Desktop** — container runtime
- **kubectl** — Kubernetes CLI
- **kind** — creates Kubernetes clusters using Docker containers
- **Helm** — Kubernetes package manager
- **k9s** — optional terminal UI for Kubernetes

> This guide assumes macOS. The Kubernetes concepts and most commands also apply to Linux and WSL.

---

## 1. Prerequisites

Install Homebrew if you do not already have it:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify:

```bash
brew --version
```

---

## 2. Install Docker Desktop

Install Docker Desktop:

```bash
brew install --cask docker
```

Start Docker Desktop and verify:

```bash
docker version
docker info
```

Kubernetes itself does not require Docker specifically, but `kind` uses a container runtime to run Kubernetes nodes locally.

---

## 3. Install Kubernetes Tools

Install the core tools:

```bash
brew install kubectl kind helm
```

Optional but highly recommended:

```bash
brew install k9s
```

Verify:

```bash
kubectl version --client
kind version
helm version
k9s version
```

You should now have:

```text
kubectl  -> interact with Kubernetes
kind     -> create local Kubernetes clusters
helm     -> install/manage Kubernetes applications
k9s      -> terminal UI for Kubernetes
```

---

# 4. Create Your First Kubernetes Cluster

Create a cluster:

```bash
kind create cluster --name learning
```

Check the cluster:

```bash
kind get clusters
```

Expected:

```text
learning
```

Check Kubernetes nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME                     STATUS   ROLES           AGE   VERSION
learning-control-plane   Ready    control-plane   1m    v1.xx.x
```

Check cluster information:

```bash
kubectl cluster-info
```

---

# 5. Understand the Kubernetes Context

`kubectl` can manage multiple Kubernetes clusters.

List contexts:

```bash
kubectl config get-contexts
```

For a kind cluster you will see something similar to:

```text
kind-learning
```

Switch to it explicitly:

```bash
kubectl config use-context kind-learning
```

Check the current context:

```bash
kubectl config current-context
```

You should get:

```text
kind-learning
```

---

# 6. Run Your First Pod

Create a simple NGINX pod:

```bash
kubectl run nginx \
  --image=nginx \
  --restart=Never
```

Check it:

```bash
kubectl get pods
```

Get more information:

```bash
kubectl describe pod nginx
```

View logs:

```bash
kubectl logs nginx
```

Execute a shell inside it:

```bash
kubectl exec -it nginx -- /bin/bash
```

Exit:

```bash
exit
```

Delete the pod:

```bash
kubectl delete pod nginx
```

---

# 7. Use a Deployment Instead of a Pod

In normal Kubernetes applications, you generally create a **Deployment** rather than manually managing individual Pods.

Create:

```bash
kubectl create deployment nginx \
  --image=nginx
```

Check:

```bash
kubectl get deployments
kubectl get pods
```

Scale it:

```bash
kubectl scale deployment nginx --replicas=3
```

Check:

```bash
kubectl get pods
```

You should now have three Pods.

---

# 8. Create a Kubernetes Manifest

Create `nginx.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 80
```

Apply it:

```bash
kubectl apply -f nginx.yaml
```

Check:

```bash
kubectl get deployment
kubectl get pods
```

Delete it:

```bash
kubectl delete -f nginx.yaml
```

---

# 9. Create a Service

A Deployment creates Pods, but other applications should normally communicate with those Pods through a **Service**.

Create `nginx-service.yaml`:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

Apply:

```bash
kubectl apply -f nginx.yaml
kubectl apply -f nginx-service.yaml
```

Check:

```bash
kubectl get pods
kubectl get services
```

---

# 10. Access the Service Locally

For a local development environment, `kubectl port-forward` is convenient.

```bash
kubectl port-forward service/nginx 8080:80
```

In another terminal:

```bash
curl http://localhost:8080
```

Or open:

```text
http://localhost:8080
```

Stop port forwarding with:

```text
Ctrl+C
```

---

# 11. Understand the Basic Kubernetes Objects

The most important objects to learn first are:

```text
Cluster
│
├── Namespace
│
├── Deployment
│   └── ReplicaSet
│       └── Pod
│           └── Container
│
├── Service
│
├── ConfigMap
│
├── Secret
│
├── Ingress
│
└── PersistentVolume / PersistentVolumeClaim
```

A typical application looks like:

```text
                    ┌──────────────┐
                    │   Ingress    │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │   Service    │
                    └──────┬───────┘
                           │
                ┌──────────┼──────────┐
                │          │          │
             ┌──▼───┐   ┌──▼───┐   ┌──▼───┐
             │ Pod  │   │ Pod  │   │ Pod  │
             └──────┘   └──────┘   └──────┘
```

---

# 12. Namespaces

Namespaces separate resources inside a cluster.

List namespaces:

```bash
kubectl get namespaces
```

Create one:

```bash
kubectl create namespace dev
```

Create a resource inside it:

```bash
kubectl -n dev create deployment nginx \
  --image=nginx
```

List resources:

```bash
kubectl -n dev get pods
```

Delete the namespace:

```bash
kubectl delete namespace dev
```

---

# 13. Useful kubectl Commands

## Resources

```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get ingress
kubectl get configmaps
kubectl get secrets
```

Everything in the current namespace:

```bash
kubectl get all
```

Everything across namespaces:

```bash
kubectl get all -A
```

---

## Detailed Information

```bash
kubectl describe pod <pod>
kubectl describe deployment <deployment>
kubectl describe service <service>
```

---

## Logs

```bash
kubectl logs <pod>
```

Follow logs:

```bash
kubectl logs -f <pod>
```

Logs from a deployment:

```bash
kubectl logs deployment/<deployment>
```

---

## Execute Commands

```bash
kubectl exec -it <pod> -- /bin/sh
```

If the container has bash:

```bash
kubectl exec -it <pod> -- /bin/bash
```

---

## Apply and Delete Manifests

```bash
kubectl apply -f .
kubectl delete -f .
```

---

# 14. Use k9s

Start k9s:

```bash
k9s
```

Useful keys:

```text
:pod       Pods
:deploy    Deployments
:svc       Services
:ns        Namespaces
:ing       Ingresses
:cm        ConfigMaps
:secret    Secrets
```

Press:

```text
? 
```

inside k9s to see available commands.

k9s is particularly useful while learning because it lets you see Pods, logs, events, deployments, and services without repeatedly typing `kubectl get ...`.

---

# 15. Install an Application with Helm

Add a Helm repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update repositories:

```bash
helm repo update
```

Search:

```bash
helm search repo nginx
```

Install:

```bash
helm install nginx bitnami/nginx
```

Check:

```bash
helm list
kubectl get pods
kubectl get services
```

Uninstall:

```bash
helm uninstall nginx
```

---

# 16. Inspect Cluster Events

When something is not working, events are often one of the first places to look.

```bash
kubectl get events
```

Sort by creation time:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

For all namespaces:

```bash
kubectl get events -A \
  --sort-by=.metadata.creationTimestamp
```

---

# 17. Debugging a Pod

If a Pod is not running:

```bash
kubectl get pods
```

Look at its status:

```bash
kubectl describe pod <pod>
```

Check logs:

```bash
kubectl logs <pod>
```

If the Pod has multiple containers:

```bash
kubectl logs <pod> -c <container>
```

Check previous container logs after a crash:

```bash
kubectl logs <pod> --previous
```

A useful debugging sequence is:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 18. Build a Local Application

A useful next step is to deploy an application you built yourself.

For example:

```text
Go application
     │
     ▼
Docker image
     │
     ▼
Kind cluster
     │
     ├── Deployment
     │      └── Pods
     │
     └── Service
            │
            ▼
       localhost
```

Build your image:

```bash
docker build -t myapp:dev .
```

Because kind runs Kubernetes nodes as Docker containers, load the image into the cluster:

```bash
kind load docker-image myapp:dev --name learning
```

Then use:

```yaml
containers:
  - name: myapp
    image: myapp:dev
    imagePullPolicy: IfNotPresent
```

Deploy:

```bash
kubectl apply -f deployment.yaml
```

---

# 19. Recommended Local Project Structure

A simple project can start with:

```text
myapp/
├── Dockerfile
├── cmd/
├── internal/
│
└── k8s/
    ├── deployment.yaml
    ├── service.yaml
    └── configmap.yaml
```

For a larger application:

```text
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
│
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    │
    └── prod/
        └── kustomization.yaml
```

You do not need Kustomize immediately. Learn the basic Kubernetes objects first.

---

# 20. Create a Multi-Node Kind Cluster

Once the basic concepts are comfortable, create multiple Kubernetes nodes.

Create `kind-config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane

  - role: worker

  - role: worker
```

Create the cluster:

```bash
kind create cluster \
  --name learning \
  --config kind-config.yaml
```

Check:

```bash
kubectl get nodes
```

You should have:

```text
learning-control-plane
learning-worker
learning-worker2
```

This is useful for learning scheduling and node-related behavior.

---

# 21. Clean Up

Delete the entire kind cluster:

```bash
kind delete cluster --name learning
```

Check:

```bash
kind get clusters
```

You can recreate it whenever you need:

```bash
kind create cluster --name learning
```

---

# 22. Suggested Learning Path

Do not try to learn every Kubernetes feature at once.

A good progression is:

### Stage 1 — Basics

Learn:

- Pod
- Deployment
- ReplicaSet
- Service
- Namespace
- Labels and selectors

Practice:

```bash
kubectl get
kubectl describe
kubectl logs
kubectl exec
kubectl apply
kubectl delete
```

### Stage 2 — Configuration

Learn:

- ConfigMap
- Secret
- Environment variables
- Volumes
- PersistentVolume
- PersistentVolumeClaim

### Stage 3 — Networking

Learn:

- ClusterIP
- NodePort
- LoadBalancer
- Ingress
- DNS
- NetworkPolicy

### Stage 4 — Scheduling

Learn:

- Node selectors
- Labels
- Taints and tolerations
- Affinity / anti-affinity
- Requests and limits

### Stage 5 — Packaging

Learn:

- Helm
- Kustomize

### Stage 6 — Production Concepts

Learn:

- StatefulSet
- DaemonSet
- Jobs
- CronJobs
- Probes
- Horizontal Pod Autoscaler
- RBAC
- ServiceAccounts
- Persistent storage
- Observability

---

# 23. Useful Daily Commands

Keep this small cheat sheet handy:

```bash
# Cluster
kubectl cluster-info
kubectl get nodes

# Everything
kubectl get all

# Pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs -f <pod>

# Deployment
kubectl get deployment
kubectl rollout status deployment/<name>
kubectl rollout restart deployment/<name>

# Services
kubectl get svc

# Debug
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl exec -it <pod> -- /bin/sh

# Manifests
kubectl apply -f .
kubectl delete -f .

# Context
kubectl config get-contexts
kubectl config current-context
kubectl config use-context kind-learning

# Local access
kubectl port-forward service/<service> 8080:80

# Kind
kind get clusters
kind get nodes --name learning
kind load docker-image <image>:<tag> --name learning
kind delete cluster --name learning

# Helm
helm list
helm repo update
helm install <release> <chart>
helm uninstall <release>

# k9s
k9s
```

---

# 24. First Exercise

Build the following locally:

```text
                 localhost:8080
                        │
                        ▼
                 ┌─────────────┐
                 │   Service   │
                 └──────┬──────┘
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           ┌─────┐   ┌─────┐   ┌─────┐
           │Pod 1│   │Pod 2│   │Pod 3│
           └─────┘   └─────┘   └─────┘
                 ▲
                 │
            Deployment
```

Requirements:

1. Create a Deployment.
2. Run 3 replicas.
3. Expose it with a ClusterIP Service.
4. Port-forward the Service to `localhost:8080`.
5. Verify it with `curl`.
6. Scale it to 5 replicas.
7. Kill one Pod.
8. Observe Kubernetes automatically creating a replacement.
9. Inspect the events.
10. View everything using k9s.

Commands you should eventually be comfortable with:

```bash
kubectl apply -f .
kubectl get pods
kubectl get deployment
kubectl get service
kubectl scale deployment <name> --replicas=5
kubectl delete pod <pod>
kubectl get events --sort-by=.metadata.creationTimestamp
k9s
```

The important concept is that Kubernetes is **declarative**: you describe the desired state, and Kubernetes continuously works to make the actual state match it.
