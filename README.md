# Local Minikube Setup & Namespace Exploration

[![Windows](https://img.shields.io/badge/Windows-10%2F11-0078D4?logo=windows)](https://www.microsoft.com/en-us/windows)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Latest-326CE5?logo=kubernetes)](https://kubernetes.io)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

This repository contains a comprehensive guide to installing **minikube**, spinning up a local single-node Kubernetes cluster, and mastering Kubernetes **namespaces** for resource isolation.

---

## 📑 Table of Contents

- [Prerequisites](#-prerequisites)
- [Pre-Setup Verification](#-pre-setup-verification)
- [Step 1: Enable WSL 2](#-step-1-enable-wsl-2)
- [Step 2: Install Docker Desktop](#-step-2-install-docker-desktop)
- [Step 3: Install minikube](#-step-3-install-minikube)
- [Step 4: Install kubectl](#-step-4-install-kubectl)
- [Step 5: Start the Cluster](#-step-5-start-the-cluster)
- [Step 6: Explore and Create Namespaces](#-step-6-explore-and-create-namespaces)
- [Step 7: Graphical UI & Cluster Teardown](#-step-7-graphical-ui--cluster-teardown)
- [Verification Checklist](#-verification-checklist)
- [Quick Reference](#-quick-reference)
- [Troubleshooting](#-troubleshooting)
- [Next Steps](#-next-steps)

---

## 📋 Prerequisites

Before starting, ensure your system meets these requirements:

- **Windows 10/11** (build 18362 or higher)
- **Virtualization enabled in BIOS** (VT-x for Intel, AMD-V for AMD) — *Required for WSL 2*
- **Administrator access** to run PowerShell commands
- **4+ GB RAM** minimum (8+ GB recommended for comfortable operation)
- **20 GB free disk space** minimum
- **WSL 2 required** for Docker Desktop integration

### ⚠️ Important: BIOS Virtualization

Virtualization must be enabled in your BIOS before proceeding. This varies by manufacturer:

- **Intel systems:** Search for "VT-x" or "Intel Virtualization Technology"
- **AMD systems:** Search for "AMD-V" or "SVM Mode"
- **Hyper-V:** If enabled, minikube can use it as a hypervisor

Restart your computer after enabling virtualization for changes to take effect.

---

## 🔍 Pre-Setup Verification

Before beginning the installation, verify your system is ready:

### 1. Check Windows Version
```powershell
[System.Environment]::OSVersion.Version
```
**Expected:** Version 10.0.18362 or higher

### 2. Check Virtualization Status (Intel)
```powershell
Get-WmiObject Win32_Processor | Select-Object Name, @{Name="Virtualization Enabled"; Expression={$_.VirtualizationFirmwareEnabled}}
```

### 3. Check Hyper-V Capability (Optional)
```powershell
systeminfo | findstr /I "Hyper-V"
```

---

## 🚀 Step 1: Enable WSL 2

**WSL 2 is required for Docker Desktop and provides better performance than WSL 1.**

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
Windows Subsystem for Linux last updated: [date]
Kernel version: [version]
WSLg version: [version]
```

✅ **Verification:** If "Default Version: 2" is shown, WSL 2 is correctly configured.

---

## 🚀 Step 2: Install Docker Desktop

**Docker Desktop is required for minikube to work properly on Windows.**

- Download and install: [Docker Desktop](https://www.docker.com/products/docker-desktop) (latest version recommended)
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

✅ **Verification:** Both commands return version info without errors.

---

## 🚀 Step 3: Install minikube

### Windows (via PowerShell)

Open **PowerShell as Administrator** and run:

```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```

### Add minikube to PATH

Make sure to run PowerShell as Administrator:

```powershell
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```

**Close and reopen PowerShell**, then verify:

```powershell
minikube version
```

**Expected Output:**
```
minikube version: v1.32.x
```

✅ **Verification:** Version command returns without "command not found" error.

---

## 🚀 Step 4: Install kubectl

**kubectl** is the command-line tool to interact with your Kubernetes cluster.

### Windows (via PowerShell)

```powershell
winget install -e --id Kubernetes.kubectl
```

**Close and reopen PowerShell**, then verify:

```powershell
kubectl version --client
```

**Expected Output:**
```
Client Version: v1.28.x
Kustomize Version: v5.0.x
```

✅ **Verification:** Version command returns without errors.

---

## 🛠️ Step 5: Start the Cluster

### Launch Your Local Cluster

Minikube will automatically pull the required infrastructure images and configure your local `kubectl` context:

```bash
minikube start --driver=docker
```

> ℹ️ **Tip:** First startup may take 2-5 minutes as it downloads the Kubernetes container images (~700 MB).

> 💡 **Note:** If you prefer to use Hyper-V instead, use: `minikube start --driver=hyperv`

### Verify Cluster Health

```bash
kubectl get nodes
```

**Expected Output:**
```
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   2m30s   v1.28.3
```

### Check Cluster Status

```bash
minikube status
```

**Expected Output:**
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
```

✅ **Verification:** Node shows "Ready" status and all components show "Running".

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
    team: backend
```

Apply the file to your cluster:

```bash
kubectl apply -f production-ns.yaml
```

Verify:
```bash
kubectl get namespaces --show-labels
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

### 5. Compare Namespace Isolation

Deploy the same pod to the `default` namespace:

```bash
kubectl run test-nginx-default --image=nginx --namespace=default
```

**See the difference:**
```bash
# Pods in development namespace
kubectl get pods -n development

# Pods in default namespace
kubectl get pods -n default

# Pods are isolated — each namespace has its own test-nginx
```

### 6. Verify Namespace Isolation

Standard `kubectl` commands show only the `default` namespace:

```bash
kubectl get pods
```

To see pods in other namespaces, explicitly target them:

```bash
kubectl get pods --namespace=development
# or use the shorthand:
kubectl get pods -n development
```

### 7. Permanently Change Your Default Namespace Context

To avoid typing the namespace flag for every command, update your context:

```bash
kubectl config set-context --current --namespace=development
```

Now, standard commands like `kubectl get pods` automatically target the `development` namespace:

```bash
kubectl get pods  # Shows only pods in development namespace
```

To switch back to `default`:
```bash
kubectl config set-context --current --namespace=default
```

### 8. View Your Current Context

```bash
kubectl config current-context
kubectl config view --minify
```

---

## 🖥️ Step 7: Graphical UI & Cluster Teardown

### Launch the Kubernetes Dashboard

For a visual web console to audit your cluster and toggle between namespaces:

```bash
minikube dashboard
```

This opens the Kubernetes Dashboard in your default browser at `http://localhost:[port]`.

**Dashboard Features:**
- View all pods, services, and deployments across namespaces
- Switch namespaces using the dropdown menu
- Monitor resource usage and pod logs
- Create and delete resources through the UI

### Stop the Cluster

When done experimenting, safely shut down to free up CPU and memory:

```bash
minikube stop
```

**Result:** Cluster saves its state; restart with `minikube start`.

### Delete the Cluster (Optional)

⚠️ **WARNING:** This action is irreversible and deletes all cluster data.

To completely remove the minikube cluster and reclaim disk space:

```bash
minikube delete
```

**After deletion:**
- All pods, services, and custom namespaces are removed
- VM/container is destroyed
- ~2 GB of disk space is freed
- Next `minikube start` will create a fresh cluster

---

## ✅ Verification Checklist

Use this checklist to verify your setup is complete:

- [ ] **WSL 2 installed** — `wsl --status` shows "Default Version: 2"
- [ ] **Docker Desktop running** — `docker --version` returns without errors
- [ ] **Docker WSL 2 integration enabled** — Settings → Resources → WSL integration ✓
- [ ] **minikube in PATH** — `minikube version` works in any PowerShell window
- [ ] **kubectl in PATH** — `kubectl version --client` works in any PowerShell window
- [ ] **Cluster starting** — `minikube start --driver=docker` completes successfully
- [ ] **Cluster health** — `kubectl get nodes` shows minikube node as "Ready"
- [ ] **Namespaces created** — `kubectl get namespaces` shows development namespace
- [ ] **Pod deployment** — `kubectl run test-nginx -n development --image=nginx` succeeds
- [ ] **Dashboard accessible** — `minikube dashboard` opens in browser
- [ ] **Context switching** — `kubectl config set-context --current --namespace=development` works

---

## 📝 Quick Reference

| Task | Command |
|------|---------|
| Start cluster | `minikube start --driver=docker` |
| Check cluster status | `minikube status` |
| Stop cluster | `minikube stop` |
| Delete cluster | `minikube delete` |
| View all nodes | `kubectl get nodes` |
| **Namespace Operations** | |
| List namespaces | `kubectl get namespaces` |
| Create namespace | `kubectl create namespace <name>` |
| Delete namespace | `kubectl delete namespace <name>` |
| Switch default namespace | `kubectl config set-context --current --namespace=<name>` |
| View current context | `kubectl config current-context` |
| **Pod Operations** | |
| View pods (all namespaces) | `kubectl get pods --all-namespaces` |
| View pods in namespace | `kubectl get pods -n <namespace>` |
| Create pod in namespace | `kubectl run <pod-name> --image=<image> -n <namespace>` |
| Delete pod | `kubectl delete pod <pod-name> -n <namespace>` |
| View pod logs | `kubectl logs <pod-name> -n <namespace>` |
| **Resource Queries** | |
| View all resources in namespace | `kubectl get all -n <namespace>` |
| Describe resource | `kubectl describe pod <pod-name> -n <namespace>` |
| View cluster info | `kubectl cluster-info` |
| View node details | `kubectl describe node minikube` |
| **Dashboard & UI** | |
| Launch dashboard | `minikube dashboard` |
| SSH into minikube VM | `minikube ssh` |
| View minikube logs | `minikube logs` |

---

## ❓ Troubleshooting

### WSL 2 Installation Fails

**Error:** `WSL 2 installation is incomplete` or `The system cannot find the file specified`

**Solution:**
1. Enable virtualization in BIOS (VT-x or AMD-V) and restart
2. Update WSL kernel:
   ```powershell
   wsl --update
   ```
3. Reinstall WSL 2:
   ```powershell
   wsl --install -d Ubuntu
   ```
4. Restart your computer

---

### Docker Desktop Won't Start

**Error:** `Docker daemon failed to start` or `failed to start service`

**Solution:**
1. Shutdown WSL and restart Docker:
   ```powershell
   wsl --shutdown
   ```
2. Launch Docker Desktop again from Start Menu
3. Wait 30 seconds for the whale icon to show "Docker Desktop is running"

**If still failing:**
```powershell
wsl --status
```
Verify WSL 2 is active (not WSL 1).

---

### Minikube Fails to Start

**Error:** `Exited with exit code 1` or `driver not found`

**Solution:**
1. Delete the corrupted cluster:
   ```bash
   minikube delete
   ```
2. Start with explicit Docker driver:
   ```bash
   minikube start --driver=docker
   ```
3. If Docker driver is unavailable, try Hyper-V:
   ```bash
   minikube start --driver=hyperv
   ```

**If still failing, check minikube logs:**
```bash
minikube logs
```

---

### kubectl Can't Connect to Cluster

**Error:** `The connection to the server was refused` or `Unable to connect to the server`

**Solution:**
1. Verify minikube is running:
   ```bash
   minikube status
   ```
   Expected: All components show "Running"

2. Reset kubectl context:
   ```bash
   kubectl config use-context minikube
   ```

3. Check cluster connectivity:
   ```bash
   kubectl cluster-info
   ```

4. Verify the minikube context exists:
   ```bash
   kubectl config get-contexts
   ```

---

### Namespace Not Switching

**Error:** `error: No context exists with the name "minikube"` or namespace changes don't persist

**Solution:**
1. List all available contexts:
   ```bash
   kubectl config get-contexts
   ```
   Expected: Shows "minikube" context

2. Switch to minikube context:
   ```bash
   kubectl config use-context minikube
   ```

3. Set namespace for current context:
   ```bash
   kubectl config set-context --current --namespace=development
   ```

4. Verify the change:
   ```bash
   kubectl config view --minify
   ```

---

### Not Enough Resources

**Error:** `Insufficient memory`, `Insufficient CPU`, or `Pod stuck in Pending state`

**Solution:**
1. Increase minikube resources:
   ```bash
   minikube config set memory 8192
   minikube config set cpus 4
   ```

2. Delete and restart:
   ```bash
   minikube delete
   minikube start --driver=docker
   ```

3. Verify allocated resources:
   ```bash
   minikube config view
   ```

**Note:** Resource allocation requires a restart of minikube after configuration.

---

### Port Conflicts or Networking Issues

**Error:** `port is already allocated` or `connection refused`

**Solution:**
1. Check if minikube port conflicts with another service:
   ```bash
   minikube ssh
   netstat -tulpn | grep LISTEN
   ```

2. Stop conflicting services and restart minikube:
   ```bash
   minikube stop
   minikube start
   ```

3. Access minikube services via the correct IP:
   ```bash
   minikube ip
   ```

---

### Pods Stuck in ImagePullBackOff

**Error:** `Failed to pull image` or `ImagePullBackOff` status

**Solution:**
1. Verify internet connectivity is available
2. Check the image name for typos:
   ```bash
   kubectl describe pod <pod-name> -n <namespace>
   ```
3. Use images from Docker Hub's official library:
   ```bash
   # Good
   kubectl run nginx --image=nginx
   
   # Good (with version tag)
   kubectl run nginx --image=nginx:1.24
   
   # Avoid private images without credentials
   ```

---

## 🚀 Next Steps

Now that you have a working Kubernetes cluster, explore these advanced topics:

### Basic Kubernetes Concepts
- **Deploy Sample Applications:** Use `kubectl apply` to deploy pre-built manifests
- **Services:** Expose pods using ClusterIP, NodePort, and LoadBalancer services
- **ConfigMaps & Secrets:** Manage configuration and sensitive data

### Intermediate Topics
- **Persistent Volumes:** Learn data persistence across pod restarts
- **Network Policies:** Control traffic between namespaces and pods
- **Deployments & StatefulSets:** Manage replicas and stateful applications
- **Ingress Controllers:** Route external traffic to services

### Advanced Topics
- **Helm:** Package manager for Kubernetes applications
- **Monitoring:** Set up Prometheus and Grafana for cluster monitoring
- **CI/CD Integration:** Connect your cluster to GitHub Actions or GitLab CI
- **Security:** RBAC (Role-Based Access Control) and Pod Security Policies

### Useful Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes Tutorials](https://kubernetes.io/docs/tutorials/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Interactive Tutorials](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

---

**Happy Kubernetes learning! 🎉**
