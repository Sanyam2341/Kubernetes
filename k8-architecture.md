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


---

## 22. Who Creates Nodes? (IMPORTANT - Common Confusion)

Nodes are **NOT created by Kubernetes**. Nodes are actual servers (physical or virtual machines).

```
On-Prem:  Infra team racks a physical server, installs OS, installs kubelet -> it becomes a node
AWS EC2:  You launch an EC2 instance, install kubelet -> it becomes a node
EKS:      AWS manages node creation (Auto Scaling Group creates EC2 instances)
kind:     Docker containers act as nodes
```

**How does a server become a "node" in Kubernetes?**
```
1. You create a server (EC2, physical, VM, whatever)
2. Install kubelet + container runtime on it
3. kubelet starts and contacts kube-apiserver
4. API server registers it as a node in etcd
5. Now Kubernetes "knows" about this node
6. kubectl get nodes -> shows this node
```

> Nodes are created by YOU (manually or via Terraform, CloudFormation, ASG).
> NOT by controller manager, NOT by scheduler, NOT by any K8s component.

---

## 23. Controller Manager - What It Manages and What It Does NOT

### What Controller Manager DOES:

- Monitors pod count and ensures replicas match desired state
- Monitors node health via heartbeats
- Evicts pods from dead/unhealthy nodes
- Recreates pods on healthy nodes
- Manages deployments, jobs, namespaces, service accounts, endpoints

### What Controller Manager does NOT do:

- Does NOT create new nodes
- Does NOT replace dead nodes
- Does NOT provision servers
- Does NOT manage infrastructure

```
+-------------------------------------------+
|  OUTSIDE KUBERNETES (Infra Level)         |
|                                           |
|  - You / Terraform / ASG create NODES    |
|  - Kubernetes has NO control here         |
+-------------------------------------------+
              |
              v
+-------------------------------------------+
|  INSIDE KUBERNETES (Cluster Level)        |
|                                           |
|  Controller Manager manages:              |
|    - Pod count (replicas)                 |
|    - Node health monitoring               |
|    - Pod eviction from dead nodes         |
|    - Recreating pods on healthy nodes     |
|                                           |
|  Controller Manager does NOT:             |
|    - Create new nodes                     |
|    - Replace dead nodes                   |
|    - Provision servers                    |
+-------------------------------------------+
```

---

## 24. Scenario Walkthroughs - Pod Dies vs Node Dies

### Scenario 1: A POD Dies

```
You deployed: 3 replicas of nginx
Running: Pod-1 (Node-1), Pod-2 (Node-1), Pod-3 (Node-2)

Pod-2 crashes (OOM, app error, etc.)
      |
      v
Replication Controller (inside controller-manager) detects:
  desired = 3, actual = 2
      |
      v
Tells API server: "Create 1 more pod"
      |
      v
Scheduler picks a node for the new pod
      |
      v
kubelet on that node creates the pod
      |
      v
Back to: desired = 3, actual = 3

Controller Manager helped? YES
  -> detected pod count mismatch and triggered new pod creation
```

### Scenario 2: A NODE Dies (Server Crashes)

```
Cluster: Node-1 (healthy), Node-2 (healthy), Node-3 (has problems)

Node-3 stops sending heartbeat to API server
      |
      v
Node Controller (inside controller-manager) detects:

  Every 5 seconds -> "Is Node-3 alive?" -> checking heartbeat
      |
  After 40 seconds of no heartbeat:
      |
      v
  Marks Node-3 as "NotReady"
  (kubectl get nodes -> Node-3 shows NotReady)
      |
      v
  Adds taint: node.kubernetes.io/not-ready on Node-3
      |
      v
  After 5 minutes (tolerationSeconds=300):
      |
      v
  Evicts ALL pods from Node-3
      |
      v
  Replication Controller sees: "desired = 3, actual = 1"
  (because 2 pods were on Node-3 and got evicted)
      |
      v
  Scheduler places new pods on Node-1 and Node-2
      |
      v
  Pods recreated on healthy nodes

Controller Manager helped? YES - but it did NOT create a new node. It only:
  1. Detected node is dead
  2. Marked it NotReady
  3. Evicted pods from dead node
  4. Recreated pods on healthy nodes

The dead node stays dead. Kubernetes does NOT replace it.
```

### Scenario 3: Who Replaces the Dead Node Then?

```
Node-3 is dead. Kubernetes will NOT create a new node.

WHO DOES?

Option A: You manually create a new server and join it to cluster
          (kubeadm join, install kubelet, etc.)

Option B: Cloud Auto Scaling (AWS EKS example)
          - EKS Node Group has Auto Scaling Group (ASG)
          - ASG detects: desired = 3 nodes, actual = 2
          - ASG launches new EC2 instance automatically
          - kubelet starts, joins cluster
          - New node appears in kubectl get nodes

Option C: Cluster Autoscaler (Kubernetes add-on)
          - Sees pods are "Pending" (no node has capacity)
          - Tells cloud provider: "Launch more nodes"
          - Cloud provider creates new server
          - New node joins cluster

IMPORTANT DISTINCTION:
  Controller Manager -> manages PODS (recreates pods, not nodes)
  Auto Scaling Group -> manages NODES (recreates nodes, cloud level)
  Cluster Autoscaler -> bridge between K8s and cloud (requests new nodes)
```

### Scenario 4: You Delete a Pod Manually

```
kubectl delete pod nginx-pod-1
      |
      v
API server deletes the pod
      |
      v
Replication Controller sees: desired = 3, actual = 2
      |
      v
Creates new pod automatically
```

### Scenario 5: You Delete a Node (Drain + Remove)

```
kubectl drain node-3 --ignore-daemonsets
      |
      v
All pods on node-3 are gracefully moved to other nodes
      |
      v
kubectl delete node node-3
      |
      v
Node removed from cluster
      |
      v
Kubernetes does NOT create a replacement node
You have to do it yourself (or ASG does it in cloud)
```

---

## 25. Summary Table - What Happens When Things Die

| What Died | Who Detects | Who Fixes | What Happens |
|---|---|---|---|
| **Pod dies** | Controller Manager (Replication Controller) | Controller Manager + Scheduler | New pod created on healthy node |
| **Node dies** | Controller Manager (Node Controller) | Controller Manager moves PODS only | Pods evicted and recreated. Node stays dead |
| **Node replacement** | NOT Kubernetes | You / ASG / Cluster Autoscaler | New server created outside K8s |
| **Pod deleted manually** | Controller Manager | Controller Manager + Scheduler | New pod created if replica count not met |
| **Node drained manually** | You triggered it | Kubernetes moves pods | Pods moved, node stays until you delete it |


---

## 26. How 3 Control Planes Work Together

### etcd Sync (RAFT Consensus Protocol):

```
One etcd node = LEADER, other 2 = FOLLOWERS

kubectl apply -f pod.yaml
      |
API Server writes to etcd LEADER
      |
Leader sends data to both FOLLOWERS
      |
Followers confirm back
      |
Leader: "2 out of 3 confirmed (majority)" -> COMMIT
      |
Data is now on ALL 3 etcd nodes
```

### Scheduler and Controller Manager - Only 1 Active:

```
API Server:         ALL 3 run simultaneously (load balanced)
etcd:               ALL 3 run simultaneously (RAFT sync)
Scheduler:          Only 1 ACTIVE (leader election), other 2 STANDBY
Controller Manager: Only 1 ACTIVE (leader election), other 2 STANDBY

Why only 1 active?
  If all 3 schedulers active -> might schedule same pod 3 times
  If all 3 controller managers active -> might create 9 pods instead of 3

If active one dies -> leader election in ~2 seconds -> standby takes over
```

### Example:

```
CP-1: API Server [ON]  etcd [ON]  Scheduler [ACTIVE]   Controller Mgr [STANDBY]
CP-2: API Server [ON]  etcd [ON]  Scheduler [STANDBY]  Controller Mgr [ACTIVE]
CP-3: API Server [ON]  etcd [ON]  Scheduler [STANDBY]  Controller Mgr [STANDBY]

CP-2 dies:
  API Server: 2 still running, load balancer routes around it
  etcd: 2 out of 3 alive, quorum intact
  Scheduler: was on CP-1, no impact
  Controller Mgr: was on CP-2, leader election -> CP-1 or CP-3 takes over in ~2s
```

---

## 27. What If ALL 3 Control Planes Die?

```
All 3 Control Planes dead:
  API Server: GONE -> kubectl stops working
  etcd: GONE -> all cluster state lost
  Scheduler: GONE -> no new pods
  Controller Manager: GONE -> no self-healing

  BUT: Existing pods on worker nodes KEEP RUNNING
  kubelet runs independently, kube-proxy keeps routing
  App still serves users, you just cannot MANAGE anything

Recovery:
  Option A: etcd backup exists
    -> Create new control plane nodes
    -> Restore etcd from backup -> cluster comes back
    -> Downtime: 15-30 minutes

  Option B: No etcd backup (WORST CASE)
    -> New control plane has empty etcd
    -> Redeploy everything from YAML files / Git repo
    -> Downtime: hours
```

### etcd Backup Command:

```bash
# Take snapshot
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Restore from snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restored
```

---

## 28. Node Dies - What Happens to the Data Inside Pods?

```
STATELESS Apps (nginx, APIs, microservices):
  No important data inside pod
  New pod starts fresh -> works perfectly
  NO DATA LOSS

STATEFUL Apps (databases, file storage):
  Data on pod local disk (emptyDir/hostPath) -> NODE DIES -> DATA LOST
  Data on EBS PersistentVolume -> NODE DIES -> DATA SAFE (EBS is separate)
  New pod attaches same EBS volume -> data intact
```

### What etcd Stores vs What It Does NOT:

```
etcd STORES (configuration):
  Pod definitions, replica count, image name
  Services, Secrets, ConfigMaps
  Node registrations, RBAC rules
  PersistentVolume claims
  = "What should the cluster look like"

etcd does NOT store (application data):
  Database records, user uploads, files
  Container images (stored in registry)
  = Your actual business data

EBS / PersistentVolume stores:
  Your actual application data, database files, uploads

Git Repository stores:
  Your YAML files -> safety net if etcd is also lost
```

---

## 29. Ideal Number of Nodes

### Control Plane - Always ODD (for quorum):

```
2 control planes: quorum = 2, if 1 dies -> no majority -> BROKEN
3 control planes: quorum = 2, if 1 dies -> 2 remain -> WORKS
5 control planes: quorum = 3, if 2 die -> 3 remain -> WORKS
```

### Worker Nodes - Even or Odd, both fine (no quorum rule):

| Environment | Control Plane | Worker Nodes |
|---|---|---|
| Learning/Dev | 1 | 1-2 |
| Staging/QA | 1 | 2-3 |
| Production (Small) | 3 | 3-5 |
| Production (Medium) | 3 | 5-20 |
| Production (Large) | 3-5 | 20-100+ |

Rule of thumb: **N+1 workers** (N = minimum needed, +1 for failover)

---

## 30. Why ASG Alone Is Not Enough for Production

### Worker Node Dies + ASG:

```
0:00  Worker dies -> ALL pods down (if only 1 worker)
0:02  ASG detects unhealthy instance
0:03  ASG launches new EC2
0:07  EC2 boots, kubelet joins cluster
0:09  Pods scheduled, images pulled
0:10  App is UP

DOWNTIME: ~8-10 minutes
```

### With 3 Workers (no waiting for ASG):

```
0:00  Worker-1 dies
0:05  Pods evicted, moved to Worker-2 and Worker-3 immediately
0:06  App is UP

DOWNTIME: ~5-6 min (only affected pods), other pods: ZERO downtime
ASG replaces dead node in background
```

### Control Plane Dies + ASG:

```
ASG creates new EC2 -> but it is a BLANK machine
No etcd data, no certificates, no configs
YOU HAVE TO REBUILD or RESTORE from etcd backup
DOWNTIME: 30 min to hours

With 3 Control Planes:
  1 dies -> other 2 handle everything -> ZERO downtime
```

### Recommended Production Setup:

```
EKS (most companies on AWS):
  Control Plane: Managed by AWS (HA guaranteed, you dont worry)
  Workers: 3-5+ nodes in ASG across multiple AZs

Self-managed (kubeadm):
  3 Control Planes + 3 Workers + ASG for both
```
