# Local Minikube Setup & Namespace Exploration

This repository contains a comprehensive guide to installing **minikube**, spinning up a local single-node Kubernetes cluster, and mastering Kubernetes **namespaces** for resource isolation.

---

## 🚀 Step 1: Enable WSL 2
  -  Before beginning, ensure you have a container engine or hypervisor installed on your machine.
  -  Open PowerShell as Administrator and run:
```powershell
wsl --install
```
  - This installs WSL 2 along with Ubuntu by default. Restart your PC when prompted.
  - After restarting, verify WSL 2 is active:

```powershell
wsl --status
```

  ## 🚀 Step 2: Install Docker Desktop
 - Download and install: [Docker Desktop](https://docker.com) is highly recommended for most operating systems.
 - After installing:
    1. Launch Docker Desktop from the Start Menu
    2. Go to Settings → General and make sure "Use the WSL 2 based engine" is checked
    3. Click Apply & Restart
    4. Wait until the 🐳 whale icon in the system tray shows "Docker Desktop is running"
    5. Verify Docker is working:
```powershell
docker --version
docker info
```

## 🚀 Step 3: Install minikube

### Windows (via PowerShell)
```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```
### Add the minikube.exe binary to your PATH.
 - Make sure to run PowerShell as Administrator.
```powershell
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```
## 🚀 Step 4: Install kubectl
### Windows (via PowerShell)
```powershell
winget install -e --id Kubernetes.kubectl
```
 - Close and reopen PowerShell, then verify:

```powershell
kubectl version --client
```

## 🛠️ Step 5: Start the Cluster

 - Launch your local cluster. Minikube will automatically pull the required infrastructure images and configure your local `kubectl` context.

```bash
minikube start
```

 - Verify that your single-node cluster is online and healthy:
```bash
kubectl get nodes
```

---

## 🔍 Step 6: Explore and Create Namespaces

 - Namespaces provide virtual isolation for groups of resources within a single physical cluster.

### 1. View Existing Namespaces
 - Kubernetes creates internal system namespaces automatically. List them using:
```bash
kubectl get namespaces
```
*(Standard default spaces include `default`, `kube-system`, `kube-public`, and `kube-node-lease`.)*

### 2. Create a Custom Namespace (Imperative CLI)
 - To quickly provision a temporary `development` space, use the CLI:
```bash
kubectl create namespace development
```

### 3. Create a Custom Namespace (Declarative Manifest)
 - For production systems, track configuration states using manifest files. Create a file named `production-ns.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

 - Apply the file to your cluster:
```bash
kubectl apply -f production-ns.yaml
```

### 4. Deploy a Resource inside a Specific Namespace
 - By default, resources deploy to the `default` namespace. To isolate an Nginx pod inside your new `development` namespace, append the namespace flag:
```bash
kubectl run test-nginx --image=nginx --namespace=development
```

### 5. Verify the Isolated Resource
 - Running standard commands will show nothing because `kubectl` defaults to the standard namespace. You must explicitly target your custom space:
```bash
kubectl get pods --namespace=development
```

### 6. Permanently Change Your Default Namespace Context
 - To avoid typing the namespace flag for every single CLI command, update your current context:
```bash
kubectl config set-context --current --namespace=development
```
 - Now, standard commands like `kubectl get pods` will target the `development` environment by default.

---

## 🖥️ Step 7: Graphical UI & Cluster Teardown

### Launch the Dashboard
 - If you prefer a visual web console to audit your cluster and toggle between namespaces, launch the built-in dashboard:
```bash
minikube dashboard
```

### Stop the Cluster
 - When you are done experimenting, safely shut down your cluster to free up your machine's CPU and memory:
```bash
minikube stop
```
