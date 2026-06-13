# Local Minikube Setup & Namespace Exploration

[![Windows](https://img.shields.io/badge/Windows-10%2F11-0078D4?logo=windows)](https://www.microsoft.com/en-us/windows)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Latest-326CE5?logo=kubernetes)](https://kubernetes.io)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

This repository contains a comprehensive guide to installing **minikube**, spinning up a local single-node Kubernetes cluster, and mastering Kubernetes **namespaces** for resource isolation.

---

## 📑 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Enable WSL 2](#-step-1-enable-wsl-2)
- [Step 2: Install Docker Desktop](#-step-2-install-docker-desktop)
- [Step 3: Install minikube](#-step-3-install-minikube)
- [Step 4: Install kubectl](#-step-4-install-kubectl)
- [Step 5: Start the Cluster](#-step-5-start-the-cluster)
- [Step 6: Explore and Create Namespaces](#-step-6-explore-and-create-namespaces)
- [Step 7: Graphical UI & Cluster Teardown](#-step-7-graphical-ui--cluster-teardown)
- [Quick Reference](#-quick-reference)
- [Troubleshooting](#-troubleshooting)
- [Next Steps](#-next-steps)

---

## 📋 Prerequisites

Before starting, ensure your system meets these requirements:

- **Windows 10/11** (build 18362 or higher)
- **Virtualization enabled** in BIOS (VT-x for Intel, AMD-V for AMD)
- **Administrator access** to run PowerShell commands
- **4+ GB RAM** minimum (8+ GB recommended for comfortable operation)
- **20 GB free disk space** minimum
- A supported container engine or hypervisor

---

## 🚀 Step 1: Enable WSL 2

- Before beginning, ensure you have a container engine or hypervisor installed on your machine.
- Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

- This installs WSL 2 along with Ubuntu by default. **Restart your PC when prompted.**
- After restarting, verify WSL 2 is active:

```powershell
wsl --status
```

**Expected Output:**
```
Default Distribution: Ubuntu
Default Version: 2
Windows Subsystem for Linux last updated: ...
```

---

## 🚀 Step 2: Install Docker Desktop

- Download and install: [Docker Desktop](https://www.docker.com/products/docker-desktop) (highly recommended for Windows)
- After installing:
  1. Launch **Docker Desktop** from the Start Menu
  2. Go to **Settings → Resources → WSL integration**
  3. Enable **"Use the WSL 2 based engine"** and select your Ubuntu distribution
  4. Click **Apply & Restart**
  5. Wait until the 🐳 whale icon in the system tray shows **"Docker Desktop is running"**
  6. Verify Docker is working:

```powershell
docker --version
docker info
```

**Expected Output:**
```
Docker version 24.0.x, build xxxxxxx
Client: Docker Engine - Community
```

---

## 🚀 Step 3: Install minikube

### Windows (via PowerShell)

Open **PowerShell as Administrator** and run:

```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```

### Add minikube to PATH

- Make sure to run PowerShell as Administrator:

```powershell
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```

- **Close and reopen PowerShell**, then verify:

```powershell
minikube version
```

**Expected Output:**
```
minikube version: v1.32.x
```

---

## 🚀 Step 4: Install kubectl

### Windows (via PowerShell)

```powershell
winget install -e --id Kubernetes.kubectl
```

- **Close and reopen PowerShell**, then verify:

```powershell
kubectl version --client
```

**Expected Output:**
```
Client Version: v1.28.x
Kustomize Version: v5.0.x
```

---

## 🛠️ Step 5: Start the Cluster

- Launch your local cluster. Minikube will automatically pull the required infrastructure images and configure your local `kubectl` context:

```bash
minikube start
```

> ℹ️ **Tip:** First startup may take 2-5 minutes as it downloads the Kubernetes container images.

- Verify that your single-node cluster is online and healthy:

```bash
kubectl get nodes
```

**Expected Output:**
```
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   2m30s   v1.28.3
```

---

## 🔍 Step 6: Explore and Create Namespaces

**Namespaces** provide virtual isolation for groups of resources within a single physical cluster. They enable multi-team development and environment separation.

### 1. View Existing Namespaces

Kubernetes creates internal system namespaces automatically. List them:

```bash
kubectl get namespaces
```

**Expected Output:**
```
NAME              STATUS   AGE
default           Active   5m
kube-system       Active   5m
kube-public       Active   5m
kube-node-lease   Active   5m
```

*(Standard namespaces include `default`, `kube-system`, `kube-public`, and `kube-node-lease`.)*

### 2. Create a Custom Namespace (Imperative CLI)

To quickly provision a temporary `development` namespace, use the CLI:

```bash
kubectl create namespace development
```

Verify creation:
```bash
kubectl get namespaces
```

### 3. Create a Custom Namespace (Declarative Manifest)

For production systems, track configuration in manifest files. Create `production-ns.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: prod
```

Apply the file to your cluster:

```bash
kubectl apply -f production-ns.yaml
```

### 4. Deploy a Resource inside a Specific Namespace

By default, resources deploy to the `default` namespace. To isolate an Nginx pod in the `development` namespace:

```bash
kubectl run test-nginx --image=nginx --namespace=development
```

Verify deployment:
```bash
kubectl get pods --namespace=development
```

**Expected Output:**
```
NAME         READY   STATUS    RESTARTS   AGE
test-nginx   1/1     Running   0          30s
```

### 5. Verify the Isolated Resource

Standard `kubectl` commands show only the `default` namespace. Explicitly target your namespace:

```bash
kubectl get pods --namespace=development
```

Or use the shorthand:
```bash
kubectl get pods -n development
```

### 6. Permanently Change Your Default Namespace Context

To avoid typing the namespace flag for every command, update your context:

```bash
kubectl config set-context --current --namespace=development
```

Now, standard commands like `kubectl get pods` automatically target the `development` namespace.

To switch back to `default`:
```bash
kubectl config set-context --current --namespace=default
```

---

## 🖥️ Step 7: Graphical UI & Cluster Teardown

### Launch the Dashboard

For a visual web console to audit your cluster and toggle between namespaces:

```bash
minikube dashboard
```

This opens the Kubernetes Dashboard in your default browser at `http://localhost:port`.

### Stop the Cluster

When done experimenting, safely shut down to free up CPU and memory:

```bash
minikube stop
```

### Delete the Cluster (Optional)

To completely remove the minikube cluster and reclaim disk space:

```bash
minikube delete
```

---

## 📝 Quick Reference

| Task | Command |
|------|---------|
| Start cluster | `minikube start` |
| Stop cluster | `minikube stop` |
| Delete cluster | `minikube delete` |
| Cluster status | `minikube status` |
| List namespaces | `kubectl get namespaces` |
| Create namespace | `kubectl create namespace <name>` |
| Switch namespace | `kubectl config set-context --current --namespace=<name>` |
| View pods in namespace | `kubectl get pods -n <namespace>` |
| View all resources | `kubectl get all -n <namespace>` |
| Launch dashboard | `minikube dashboard` |
| View cluster info | `kubectl cluster-info` |
| View node details | `kubectl describe node minikube` |

---

## ❓ Troubleshooting

### WSL 2 Installation Fails

**Error:** `WSL 2 installation is incomplete`

**Solution:**
- Enable virtualization in BIOS (search for VT-x or AMD-V depending on your processor)
- Run: `wsl --update` to update WSL kernel
- Restart your computer

### Docker Desktop Won't Start

**Error:** `Docker daemon failed to start`

**Solution:**
```powershell
wsl --shutdown
docker --version  # Restart Docker
```

If still failing, check that WSL 2 is active:
```powershell
wsl --status
```

### Minikube Fails to Start

**Error:** `Exited with exit code 1`

**Solution:**
```bash
minikube delete  # Remove the old cluster
minikube start --driver=docker  # Start with explicit driver
```

### kubectl Can't Connect to Cluster

**Error:** `The connection to the server was refused`

**Solution:**
- Verify minikube is running: `minikube status`
- Reset kubectl context:
  ```bash
  kubectl config use-context minikube
  ```
- Check cluster info:
  ```bash
  kubectl cluster-info
  ```

### Namespace Not Switching

**Error:** `error: No context exists with the name "minikube"`

**Solution:**
```bash
kubectl config get-contexts  # List all contexts
kubectl config use-context minikube  # Switch to minikube context
kubectl config set-context --current --namespace=development
```

### Not Enough Resources

**Error:** `Insufficient memory` or `Insufficient CPU`

**Solution:**
- Increase minikube resources:
  ```bash
  minikube config set memory 8192
  minikube config set cpus 4
  ```
- Delete and restart:
  ```bash
  minikube delete
  minikube start
  ```

---

## 🚀 Next Steps

Now that you have a working Kubernetes cluster, explore these advanced topics:

- **Deploy Sample Applications:** Use Helm to deploy pre-configured applications
- **Set Up Persistent Volumes:** Learn data persistence across pod restarts
- **Configure Network Policies:** Control traffic between namespaces
- **Service Types:** Explore ClusterIP, NodePort, and LoadBalancer services
- **ConfigMaps & Secrets:** Manage configuration and sensitive data
- **Ingress Controllers:** Route external traffic to services
- **Monitoring:** Set up Prometheus and Grafana for cluster monitoring
- **CI/CD Integration:** Connect your cluster to GitHub Actions or GitLab CI

### Useful Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

**Happy Kubernetes learning! 🎉**
