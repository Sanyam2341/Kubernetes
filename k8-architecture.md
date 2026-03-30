# Kubernetes Architecture – Complete Guide (Easy + Diagrams)

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
