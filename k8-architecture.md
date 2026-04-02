# Kubernetes Architecture – Complete Guide 

---

## 🧠 1. Overview

Kubernetes has **2 main parts**:

- **Control Plane** (Old: Master) → Manages cluster
- **Worker Nodes** → Run applications (Pods)

---

## 🧩 2. High-Level Architecture

```
┌────────────────────────────┐
│       CONTROL PLANE        │
│  (Manages entire cluster)  │
└────────────────────────────┘
              |
              |
    -------------------------
    |                       |
┌──────────────────────┐  ┌──────────────────────┐
│    WORKER NODE 1     │  │    WORKER NODE 2     │
│  (Runs applications) │  │  (Runs applications) │
└──────────────────────┘  └──────────────────────┘
```

---

## 🧠 3. Detailed Architecture

```
        👨‍💻 DEV / DEVOPS
              |
           kubectl
              |
              v
    ┌────────────────────────┐
    │     kube-apiserver     │
    │   (ENTRY POINT / API)  │
    └────────────────────────┘
          /    |    \
         /     |     \
        v      v      v
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  Scheduler   │ │  Controller  │ │     etcd     │
│              │ │  Manager     │ │ (Cluster DB) │
└──────────────┘ └──────────────┘ └──────────────┘

                (CONTROL PLANE)

    WORKER NODE 1                      WORKER NODE 2
┌──────────────────────────┐  ┌──────────────────────────┐
│ kubelet (node agent)     │  │ kubelet (node agent)     │
├──────────────────────────┤  ├──────────────────────────┤
│ kube-proxy (networking)  │  │ kube-proxy (networking)  │
├──────────────────────────┤  ├──────────────────────────┤
│ Container Runtime        │  │ Container Runtime        │
│ (containerd via CRI)     │  │ (containerd via CRI)     │
├──────────────────────────┤  ├──────────────────────────┤
│        PODS              │  │        PODS              │
│   (Your Applications)    │  │   (Your Applications)    │
└──────────────────────────┘  └──────────────────────────┘
```

---

## 🔄 4. Dev / DevOps Flow

```
kubectl
   |
   v
kube-apiserver
   |
   v
Scheduler → selects node
   |
   v
kubelet (on worker node)
   |
   v
Container Runtime (via CRI)
   |
   v
POD CREATED ✅
```

---

## 🌐 5. End User Flow (IMPORTANT)

```
👤 User (Browser / App)
        |
        v
Load Balancer (External)
        |
        v
Kubernetes Service
        |
        v
kube-proxy (routes traffic)
        |
        v
Pod (Application)
```

---

## 6. Internal Communication Flow

```
kube-apiserver ←→ kubelet (on each worker node)

Scheduler / Controller Manager
            ↓
   (via API server only)
```

> **All communication goes through kube-apiserver**

---

## 🌐 7. Networking (CNI)

```
Pod (Node 1) ←──────────────→ Pod (Node 2)
         (Handled by CNI plugins)
```

---

## 8. Runtime Flow (CRI)

```
kubelet ←────────→ Container Runtime (containerd / Docker)
```

---

## 9. Full End-to-End Flow

| Step   | Flow                                          |
|--------|-----------------------------------------------|
| STEP 1 | Dev → `kubectl apply`                         |
| STEP 2 | kubectl → kube-apiserver                      |
| STEP 3 | kube-apiserver → etcd (store desired state)   |
| STEP 4 | Scheduler → assigns node                      |
| STEP 5 | kubelet → checks API server                   |
| STEP 6 | kubelet → container runtime (via CRI)         |
| STEP 7 | Pod starts running                            |
| STEP 8 | kubelet → updates status to API server        |
| STEP 9 | User → Load Balancer → Service → kube-proxy → Pod |

---

## 10. Components (Simple Explanation)

### 🔹 Control Plane

- **kube-apiserver** → Main entry point; all communication happens through it
- **etcd** → Stores cluster data/state
- **kube-scheduler** → Decides where pods will run
- **controller-manager** → Keeps system in desired state

### 🔹 Worker Node

- **kubelet** → Talks to API server and creates pods
- **kube-proxy** → Handles network routing to pods
- **Container Runtime** → Runs containers (containerd/Docker)
- **Pods** → Your actual applications

---

## 11. Key Abbreviations

| Component | Full Form                      | Purpose               |
|-----------|--------------------------------|-----------------------|
| **CNI**   | Container Network Interface    | Pod networking        |
| **CRI**   | Container Runtime Interface    | Runtime communication |

---

## 12. Key Points to Remember

- **kube-apiserver** = brain
- **kubelet** = executor
- **Pods** = applications
- **Service** = exposes pods
- **Load Balancer** = entry point for users

---

## 🧠 13. Simple Memory Trick

```
Control Plane → gives instruction
kubelet       → executes
Pods          → run apps
User          → accesses via Service
```

---

## ⭐ 14. One-Line Summary

> **Dev → API Server → Scheduler → kubelet → Pod → User via Load Balancer & Service**

---

## 🔥 15. Interview Answer (Quick)

> Developer sends request using **kubectl** → **kube-apiserver** processes it → **scheduler** assigns node → **kubelet** creates pod → user accesses application via **load balancer** and **service**.


---

## 16. etcd - The Key-Value Store (Cluster Database)

- etcd stores **everything** about the cluster - nodes, pods, configmaps, secrets, namespaces, roles, etc.
- It is a **key-value store**, not a relational DB
- Only **kube-apiserver** talks to etcd directly - no other component can
- If etcd dies, cluster loses all state - **that is why etcd backup is critical**

```
Example: What etcd stores

Key                                    Value
---                                    -----
/registry/pods/default/nginx-pod       {pod spec, status, metadata...}
/registry/nodes/worker-1               {node info, capacity, conditions...}
/registry/services/default/my-service  {service spec, clusterIP, ports...}
/registry/secrets/default/db-password  {encrypted secret data...}
```

```
Flow:
kubectl apply -f pod.yaml
        |
kube-apiserver receives request
        |
Writes desired state to etcd
        |
etcd stores: "user wants 3 nginx pods"
        |
Controller manager reads from API server
        |
Ensures actual state = desired state
```

---

## 17. kube-apiserver - The Gateway / Entry Point

- **Every single request** goes through API server - kubectl, scheduler, controller manager, kubelet - ALL of them
- It is the **only component** that talks to etcd
- Handles **authentication, authorization, and admission control**
- It is a REST API - everything is an API call internally

```
                    +------------------+
  kubectl ----------|                  |
  scheduler --------|  kube-apiserver  |<---> etcd
  controller-mgr ---|                  |
  kubelet ----------|  (SINGLE ENTRY)  |
  kube-proxy -------|                  |
                    +------------------+
```

**What happens when you run kubectl get pods:**
```
kubectl get pods
      |
kubectl sends GET request to kube-apiserver
      |
API server authenticates (who are you?)
      |
API server authorizes (are you allowed?)
      |
API server reads from etcd
      |
Returns pod list to kubectl
```

---

## 18. kube-controller-manager - The Brain That Maintains Desired State

- Runs multiple **controllers** as a single process
- Each controller watches a specific resource and ensures **actual state = desired state**

### Controllers inside controller-manager:

| Controller | What It Does |
|---|---|
| **Node Controller** | Monitors node health, marks unhealthy nodes |
| **Replication Controller** | Ensures correct number of pod replicas |
| **Deployment Controller** | Manages rollouts and rollbacks |
| **Service Account Controller** | Creates default service accounts |
| **Endpoint Controller** | Populates endpoint objects (connects Services to Pods) |
| **Job Controller** | Manages Job resources (run-to-completion) |
| **Namespace Controller** | Cleans up resources when namespace is deleted |

### Node Controller - Default Timings (IMPORTANT):

```
+-------------------------------------------------------------+
|  Node Controller Flow (Default Values)                      |
+-------------------------------------------------------------+
|                                                             |
|  Every 5 seconds -> checks node heartbeat                   |
|       (--node-monitor-period=5s)                            |
|                                                             |
|  After 40 seconds of no heartbeat -> marks node "NotReady"  |
|       (--node-monitor-grace-period=40s)                     |
|                                                             |
|  After 5 minutes of NotReady -> evicts pods to healthy node |
|       (controlled by pod toleration: tolerationSeconds=300) |
|                                                             |
+-------------------------------------------------------------+
```

**NOTE:** These values are **defaults hardcoded in the binary**. They do NOT appear in the manifest YAML unless explicitly overridden. To override, add these flags:

```yaml
# In kube-controller-manager.yaml
- --node-monitor-period=5s
- --node-monitor-grace-period=40s
```

For pod eviction (v1.13+), it is controlled by **Taint-Based Eviction** - the pod toleration decides when to evict:
```yaml
# Every pod automatically gets this toleration:
tolerations:
- effect: NoExecute
  key: node.kubernetes.io/not-ready
  tolerationSeconds: 300    # 5 minutes
```

### Example: Controller Manager in Action

```
You say: "I want 3 replicas of nginx"
      |
API server stores in etcd: desired = 3 replicas
      |
Replication Controller watches API server
      |
Sees: desired = 3, actual = 0
      |
Creates 3 pods via API server
      |
Now: desired = 3, actual = 3

If 1 pod dies:
      |
Controller sees: desired = 3, actual = 2
      |
Creates 1 more pod automatically
      |
Back to: desired = 3, actual = 3
```

---

## 19. File Locations on Master Node (Control Plane)

After SSH/exec into the master node, here is where everything lives:

### Static Pod Manifests (Most Important)

```bash
/etc/kubernetes/manifests/
  etcd.yaml                      # etcd configuration
  kube-apiserver.yaml            # API server configuration
  kube-controller-manager.yaml   # Controller manager configuration
  kube-scheduler.yaml            # Scheduler configuration
```

> kubelet watches this directory. If you edit any file here, the component **auto-restarts**.

### Certificates and Keys (PKI)

```bash
/etc/kubernetes/pki/
  ca.crt                  # Cluster CA certificate
  ca.key                  # Cluster CA private key
  apiserver.crt           # API server certificate
  apiserver.key           # API server private key
  apiserver-kubelet-client.crt
  apiserver-kubelet-client.key
  front-proxy-ca.crt
  front-proxy-ca.key
  sa.key                  # Service account signing key
  sa.pub                  # Service account public key
  etcd/
    ca.crt              # etcd CA certificate
    server.crt          # etcd server certificate
    server.key          # etcd server key
```

### Kubeconfig Files

```bash
/etc/kubernetes/
  admin.conf                    # Admin kubeconfig (used by kubectl)
  controller-manager.conf       # Controller manager kubeconfig
  scheduler.conf                # Scheduler kubeconfig
  kubelet.conf                  # Kubelet kubeconfig
```

### How to Access These Files

```bash
# For kind cluster:
docker exec -it <control-plane-container> bash
# Example:
docker exec -it sanyam-kind-cluster-control-plane bash

# For kubeadm cluster:
ssh user@master-node-ip

# Then navigate:
cd /etc/kubernetes/manifests/
cat kube-apiserver.yaml
cat etcd.yaml
cat kube-controller-manager.yaml
cat kube-scheduler.yaml
```

### Quick Commands to Inspect

```bash
# See all static pod manifests
ls -la /etc/kubernetes/manifests/

# Check etcd data directory
ls -la /var/lib/etcd/

# Check certificates
ls -la /etc/kubernetes/pki/

# Check kubelet config
cat /var/lib/kubelet/config.yaml

# See running control plane processes
ps aux | grep kube
```

---

## 20. Quick Reference - Who Talks to Whom

```
+------------------------------------------------------+
|                                                      |
|  kubectl -------> API Server <-----> etcd            |
|                      ^                               |
|  Scheduler ----------+                               |
|  Controller Manager -+                               |
|  kubelet ------------+                               |
|  kube-proxy ---------+                               |
|                                                      |
|  RULE: Everything goes through API Server            |
|  RULE: Only API Server talks to etcd                 |
|                                                      |
+------------------------------------------------------+
```

---

## 21. Interview Quick Answers

**Q: What is etcd?**
> Key-value store that holds the entire cluster state. Only API server communicates with it.

**Q: What does controller manager do?**
> Runs controllers that ensure actual state matches desired state. Example: if you want 3 replicas and 1 pod dies, it creates a new one.

**Q: How does Kubernetes detect a node failure?**
> Controller manager checks node heartbeat every 5s. If no heartbeat for 40s, marks node NotReady. After 5 minutes, pods are evicted to healthy nodes.

**Q: Where are control plane configs stored?**
> /etc/kubernetes/manifests/ - static pod YAMLs for apiserver, etcd, controller-manager, scheduler.
