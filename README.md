# Local Minikube Setup & Namespace Exploration

This repository contains a comprehensive guide to installing **minikube**, spinning up a local single-node Kubernetes cluster, and mastering Kubernetes **namespaces** for resource isolation.

---

## 📋 Prerequisites

Before beginning, ensure you have a container engine or hypervisor installed on your machine. 
* [Docker Desktop](https://docker.com) is highly recommended for most operating systems.

---

## 🚀 Step 1: Install minikube and kubectl

Execute the appropriate installation command for your operating system terminal:

### macOS (via Homebrew)
```bash
brew install minikube kubectl
```

### Linux (via curl)
```bash
curl -LO https://googleapis.com
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Windows (via PowerShell)
```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType 'directory'
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com'
# Note: Manually add c:\minikube to your System PATH variables
```

---

## 🛠️ Step 2: Start the Cluster

Launch your local cluster. Minikube will automatically pull the required infrastructure images and configure your local `kubectl` context.

```bash
minikube start
```

Verify that your single-node cluster is online and healthy:
```bash
kubectl get nodes
```

---

## 🔍 Step 3: Explore and Create Namespaces

Namespaces provide virtual isolation for groups of resources within a single physical cluster.

### 1. View Existing Namespaces
Kubernetes creates internal system namespaces automatically. List them using:
```bash
kubectl get namespaces
```
*(Standard default spaces include `default`, `kube-system`, `kube-public`, and `kube-node-lease`.)*

### 2. Create a Custom Namespace (Imperative CLI)
To quickly provision a temporary `development` space, use the CLI:
```bash
kubectl create namespace development
```

### 3. Create a Custom Namespace (Declarative Manifest)
For production systems, track configuration states using manifest files. Create a file named `production-ns.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

Apply the file to your cluster:
```bash
kubectl apply -f production-ns.yaml
```

### 4. Deploy a Resource inside a Specific Namespace
By default, resources deploy to the `default` namespace. To isolate an Nginx pod inside your new `development` namespace, append the namespace flag:
```bash
kubectl run test-nginx --image=nginx --namespace=development
```

### 5. Verify the Isolated Resource
Running standard commands will show nothing because `kubectl` defaults to the standard namespace. You must explicitly target your custom space:
```bash
kubectl get pods --namespace=development
```

### 6. Permanently Change Your Default Namespace Context
To avoid typing the namespace flag for every single CLI command, update your current context:
```bash
kubectl config set-context --current --namespace=development
```
Now, standard commands like `kubectl get pods` will target the `development` environment by default.

---

## 🖥️ Step 4: Graphical UI & Cluster Teardown

### Launch the Dashboard
If you prefer a visual web console to audit your cluster and toggle between namespaces, launch the built-in dashboard:
```bash
minikube dashboard
```

### Stop the Cluster
When you are done experimenting, safely shut down your cluster to free up your machine's CPU and memory:
```bash
minikube stop
```
