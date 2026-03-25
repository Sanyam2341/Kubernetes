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
* `kubectl cluster-info dump` - Dump full cluster debug info for troubleshooting.
* `kubectl get pods` - List pods in the current namespace.
* `kubectl get pods -A` - View all pods across all namespaces.
* `kubectl get pods -w` - Watch pods in real-time (live updates on status changes).
* `kubectl get pods -n <ns> -o wide` - Show pods with extra details (IP, Node).
* `kubectl describe node <node-name>` - Deep dive into a specific node's health.
* `kubectl describe pod <pod-name> -n <ns>` - Full details of a pod (events, status, IP).

### 🚢 Creating & Deleting Resources
* `kubectl run <name> --image=<image>` - Quickly create a pod from the command line.
* `kubectl run <name> --image=<image> -n <ns>` - Create a pod in a specific namespace.
* `kubectl create ns <name>` - Shorthand to create a namespace quickly.
* `kubectl apply -f <file.yml>` - Create or update resources from a YAML file.
* `kubectl delete pod <name> -n <ns>` - Delete a specific pod.
* `kubectl delete deployment <name> -n <ns>` - Delete a deployment and all its pods.
* `kubectl delete ns <name>` - Delete an entire namespace and everything inside it.

### 📈 Deployments & Scaling
* `kubectl get deployments` - List all deployments in the current namespace.
* `kubectl get deployments -n <ns>` - List deployments in a specific namespace.
* `kubectl get daemonsets -n <ns>` - List all DaemonSets (e.g., `kubectl get daemonsets -n kube-system`).
* `kubectl scale deployment/<name> -n <ns> --replicas=<count>` - Scale pods up or down (e.g., `--replicas=10`).
* `kubectl set image deployment/<name> -n <ns> <container>=<image:tag>` - Rolling update to a new image version (e.g., `nginx=nginx:1.26.3`).

### 🔧 Debugging & Access
* `kubectl exec -it pod/<name> -n <ns> -- bash` - Open a shell inside a running pod.

### 🔄 Context & Configuration
*Used to switch between different clusters (e.g., switching from Kind to Minikube).*

* **List Contexts:** `kubectl config get-contexts`
* **Current Context:** `kubectl config current-context`
* **Switch Active Cluster:** `kubectl config use-context <context-name>`

---

## 📦 4. Namespaces
*Namespaces are like separate "offices" inside your cluster — they keep teams, environments, and apps isolated from each other. A single cluster can have many namespaces, and each namespace can hold multiple pods.*

| Command | Description |
| :--- | :--- |
| `kubectl get ns` | List all namespaces in the cluster. |
| `kubectl create ns <name>` | Create a new namespace (`ns` is shorthand for `namespace`). |
| `kubectl get pods -n <namespace>` | View pods inside a specific namespace. |
| `kubectl run <pod> --image=<img> -n <ns>` | Launch a pod in a specific namespace. |
| `kubectl delete ns <name>` | Delete a namespace and all its resources. |

---

## 🧩 5. Workload Types (Deployment vs ReplicaSet vs StatefulSet)

* 🛡️ **ReplicaSet** — Ensures a fixed number of pod copies are always running. If one dies, it replaces it. But it can't handle version upgrades smoothly — use Deployments instead.
* 🤖 **Deployment** — The smart boss that manages ReplicaSets. Handles rolling updates (upgrade pods one-by-one with zero downtime) and rollbacks. Use this 90% of the time.
* 🔄 **Rolling Update** — When you change the image version, the Deployment kills old pods and starts new ones gradually so your app never goes fully down.
* 🏷️ **Labels** — Key-value tags you attach to resources (e.g., `app: nginx`). They are how you organize and identify your pods, nodes, or any K8s object.
* 🎯 **Selectors** — Filters that match labels to connect resources together. A Deployment uses `selector.matchLabels` to know which pods belong to it.
* 📚 **StatefulSet** — For apps that need identity & storage (like databases). Each pod gets a fixed name (`db-0`, `db-1`) and its own persistent volume. Starts/stops in order.
* 👾 **DaemonSet** — Runs exactly one pod on every node automatically. No `replicas` field — the node count IS the replica count. Used for system-level stuff like log collectors, monitoring agents, and CNI plugins (`kindnet`, `kube-proxy`).
* ⏳ **Job** — Runs a pod that does its task and exits. Retries on failure. Use `completions` and `parallelism` to run in batches (e.g., 10 tasks, 3 at a time).
* ⏰ **CronJob** — A Job that runs on a schedule (like Linux cron). Example: `schedule: "0 2 * * *"` = every night at 2 AM. Schedule format: `minute hour day-of-month month day-of-week`. Common YAML mistakes: `labels` must be inside `metadata`, `restartPolicy` and `volumes` go at pod-level (outside `containers`).
* 💾 **PersistentVolume (PV)** — A pre-provisioned piece of storage (like a parking spot). With `hostPath`, the `capacity.storage` is just a label, NOT a real disk limit — the pod can use the entire host disk.
* 📎 **PersistentVolumeClaim (PVC)** — A request for storage by a pod (like a parking ticket that claims a spot). Binds to a matching PV.
* 📂 **Volumes** — Give storage to containers. `hostPath` maps a folder from the worker node into the container. Folders are created on the **worker node**, not the master. Use `DirectoryOrCreate` to auto-create folders.

| Feature | Deployment | ReplicaSet | StatefulSet | DaemonSet |
| :--- | :--- | :--- | :--- | :--- |
| Main Use | Web Servers (Nginx) | (Internal use only) | Databases (MySQL, MongoDB) | Logging, Monitoring, CNI |
| Pod Names | Random (`web-a1b2`) | Random | Fixed (`db-0`, `db-1`) | One per node |
| Replicas | You set the count | You set the count | You set the count | Auto (1 per node) |
| Storage | Shared/Temporary | Temporary | Permanent & Individual | Temporary |
| Updates | Automatic & Smooth | Manual/Hard | Careful & Ordered | Rolling |

---

## 🗄️ 6. Volumes & Storage
*Volumes give persistent storage to containers. Without them, data is lost when a pod dies.*

| Command | Description |
| :--- | :--- |
| `kubectl get pv` | List all PersistentVolumes in the cluster. |
| `kubectl get pvc -n <ns>` | List PersistentVolumeClaims in a namespace. |
| `kubectl describe pv <name>` | See details of a PV (capacity, status, reclaim policy). |
| `kubectl describe pvc <name> -n <ns>` | See PVC details and which PV it's bound to. |
| `kubectl get cronjob -n <ns>` | List all CronJobs in a namespace. |
| `kubectl get jobs -n <ns>` | List all Jobs created by CronJobs. |
| `docker exec -it <worker-node> ls /path` | Check hostPath volumes inside KIND worker nodes. |
| `docker exec -it <worker-node> bash` | Shell into a KIND worker node to inspect files. |

---

## 🔗 7. Git & GitHub
*Commands used to push your local K8s work to GitHub.*

* `git init` - Turn a local folder into a Git repository.
* `git remote add origin <url>` - Connect local repo to your GitHub repo.
* `git remote -v` - Verify the remote URL is set correctly.
* `git add .` - Stage all changed files for commit.
* `git commit -m "message"` - Save a snapshot with a descriptive note.
* `git branch -M main` - Rename the default branch to `main`.
* `git push origin main` - Upload your commits to GitHub.
* `git config --global credential.helper osxkeychain` - Save your GitHub token in Mac Keychain.

---

## 🩺 8. System & Troubleshooting (EC2/Linux)
*Infrastructure-level commands to ensure the host machine is healthy.*

* **SSH into EC2:** `ssh -i "key.pem" ubuntu@<ip>` (Connect to your practice instance).
* **Resource Monitor:** `htop` (Monitor RAM/CPU usage in real-time).
* **Storage Check:** `lsblk` or `df -h` (Verify EBS volume space).
* **Virtualization Check:** `egrep -c '(vmx|svm)' /proc/cpuinfo` (Check for nested VT support).
* **Partition Growth:** `sudo growpart /dev/xvda 1` followed by `sudo resize2fs /dev/xvda1`.

---

## 📝 9. Tips & Best Practices
* **Aliasing:** Add `alias k='kubectl'` to your `.bashrc` or `.zshrc` to save time.
* **Watch Mode:** Add `-w` to the end of any `get` command (e.g., `kubectl get pods -w`) to see updates in real-time.
* **Namespace:** Use `-n <namespace>` to look at specific parts of the cluster.