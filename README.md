# setup-minikube-on-windows
Step by step guide to Setup minikube at your local and explore creating namespaces
## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1 — Enable WSL 2](#step-1--enable-wsl-2)
- [Step 2 — Install Docker Desktop](#step-2--install-docker-desktop)
- [Step 3 — Install minikube](#step-3--install-minikube)
- [Step 4 — Install kubectl](#step-4--install-kubectl)
- [Step 5 — Start Your Cluster](#step-5--start-your-cluster)
- [Step 6 — Verify the Cluster](#step-6--verify-the-cluster)
- [Common kubectl Commands](#-common-kubectl-commands)
- [Troubleshooting](#-troubleshooting)

## ✅ Prerequisites

| Requirement | Notes |
|---|---|
| Windows 10/11 (Home or Pro) | Home edition supported ✅ |
| WSL 2 | Required by Docker Desktop |
| Docker Desktop | Acts as the minikube driver |
| minikube | The local Kubernetes cluster tool |
| kubectl | CLI to interact with your cluster |

⚠️ **Windows Home users:** Hyper-V is **not supported** on Windows Home. This guide uses the **Docker driver**, which works on all Windows editions.

## Step 1 — Enable WSL 2

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

This installs WSL 2 along with Ubuntu by default. **Restart your PC** when prompted.

After restarting, verify WSL 2 is active:

```powershell
wsl --status
```

You should see `Default Version: 2`.

---
## Step 2 — Install Docker Desktop

Download and install Docker Desktop from the official site:

🔗 [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

After installing:

1. Launch **Docker Desktop** from the Start Menu
2. Go to **Settings → General** and make sure **"Use the WSL 2 based engine"** is checked
3. Click **Apply & Restart**
4. Wait until the 🐳 whale icon in the system tray shows **"Docker Desktop is running"**

Verify Docker is working:

```powershell
docker --version
docker info
```

---

## Step 3 — Install minikube

Install minikube using **winget** (built into Windows 10/11):

```powershell
winget install -e --id Kubernetes.minikube
```

Close and reopen PowerShell, then verify:

```powershell
minikube version
```

Set Docker as the default driver so you don't need to specify it every time:

```powershell
minikube config set driver docker
```

---

## Step 4 — Install kubectl

```powershell
winget install -e --id Kubernetes.kubectl
```

Close and reopen PowerShell, then verify:

```powershell
kubectl version --client
```

---

## Step 5 — Start Your Cluster

Make sure Docker Desktop is running, then:

```powershell
minikube start --driver=docker
```

Expected output:

```
🎉  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Optionally, open the Kubernetes Dashboard in your browser:

```powershell
minikube dashboard
```

---

## Step 6 — Verify the Cluster

Run these to confirm everything is healthy:

```powershell
# Overall cluster status
minikube status

# Check the node is Ready
kubectl get nodes

# List all running system pods
kubectl get pods -A
```

You should see a node with status **`Ready`**.

---

## 📟 Common kubectl Commands

```powershell
# List all pods in the default namespace
kubectl get pods

# List all services
kubectl get services

# List all deployments
kubectl get deployments

# Deploy a test nginx app
kubectl create deployment hello-minikube --image=nginx

# Expose it so you can access it
kubectl expose deployment hello-minikube --type=NodePort --port=80

# Open the service in your browser
minikube service hello-minikube

# View logs for a pod
kubectl logs <pod-name>

# Describe a pod (useful for debugging)
kubectl describe pod <pod-name>

# Delete a deployment
kubectl delete deployment hello-minikube

# Stop the cluster (preserves state)
minikube stop

# Delete the cluster entirely
minikube delete
```

---

## 🔧 Troubleshooting

See the full troubleshooting guide: [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)

**Quick reference:**

| Error | Fix |
|---|---|
| `Hyper-V PowerShell Module is not available` | Use `--driver=docker` |
| `Docker is not running` | Start Docker Desktop first |
| `minikube start` hangs or fails | Run `minikube delete --all` then retry |
| `kubectl` not found | Reinstall: `winget install Kubernetes.kubectl` |
| WSL 2 not installed | Run `wsl --install` as Administrator |

---

## 📂 Folder Structure

```
minikube-windows/
├── README.md            ← Step-by-step setup guide (you are here)
├── TROUBLESHOOTING.md   ← Common errors and fixes
└── .gitignore
```

---

