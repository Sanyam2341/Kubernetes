# ☸️ Kubernetes Learning Journey: Command Reference

This repository serves as a personal "Second Brain" for my Kubernetes (K8s) transition. It includes setup scripts, configuration files, and a categorized list of essential commands.

---

## 🛠 1. Cluster Orchestration (Kind)
*Kind (Kubernetes in Docker) is the primary tool used for multi-node local clusters.*

| Command | Description |
| :--- | :--- |
| `kind create cluster --config kind-config.yml` | Initialize cluster using a YAML config file. |
| `kind get clusters` | List all active Kind clusters. |
| `kind delete cluster --name <name>` | Completely remove a cluster and its nodes. |
| `docker ps` | View the Kind nodes running as Docker containers. |

---

## 🚀 2. Cluster Management (Minikube)
*Used for single-node local testing with specific drivers.*

- **Start with Docker Driver:** `minikube start --driver=docker --memory=2048mb --cpus=2`
- **Check Status:** `minikube status`
- **Stop Cluster:** `minikube stop`
- **Delete All Traces:** `minikube delete`

---

## 🎮 3. The Control Center (kubectl)
*The daily-driver CLI for interacting with any Kubernetes cluster.*

### 🔍 Discovery & Status
* `kubectl get nodes` - Check if Control-plane and Workers are `Ready`.
* `kubectl cluster-info` - Verify the API server connection.
* `kubectl get pods -A` - View all pods across all namespaces.
* `kubectl describe node <node-name>` - Deep dive into a specific node's health.

### 🔄 Context & Configuration
*Used to switch between different clusters (e.g., switching from Kind to Minikube).*

* **List Contexts:** `kubectl config get-contexts`
* **Current Context:** `kubectl config current-context`
* **Switch Active Cluster:** `kubectl config use-context <context-name>`

---

## 🩺 4. System & Troubleshooting (EC2/Linux)
*Infrastructure-level commands to ensure the host machine is healthy.*

* **Resource Monitor:** `htop` (Monitor RAM/CPU usage in real-time).
* **Storage Check:** `lsblk` or `df -h` (Verify EBS volume space).
* **Virtualization Check:** `egrep -c '(vmx|svm)' /proc/cpuinfo` (Check for nested VT support).
* **Partition Growth:** `sudo growpart /dev/xvda 1` followed by `sudo resize2fs /dev/xvda1`.

---

## 📝 5. Tips & Best Practices
* **Aliasing:** Add `alias k='kubectl'` to your `.bashrc` or `.zshrc` to save time.
* **Watch Mode:** Add `-w` to the end of any `get` command (e.g., `kubectl get pods -w`) to see updates in real-time.
* **Namespace:** Use `-n <namespace>` to look at specific parts of the cluster.