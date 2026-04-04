# Kubernetes — Pods & ReplicaSets

---

## Pods

### What is a Pod?

A Pod is the **smallest thing you can deploy** in Kubernetes. Think of it as a wrapper around your container(s).

You don't run containers directly in Kubernetes — you run Pods, and inside each Pod, your container lives.

**Simple analogy**: If a container is a person, a Pod is the room they sit in. Most rooms have one person, but sometimes two people share a room if they need to work closely together.

---

### Why Not Just Run Containers Directly?

Kubernetes needs a way to manage networking, storage, and lifecycle for your containers. A Pod provides that layer. Every Pod gets:

- Its **own IP address** (containers inside the same Pod share this IP)
- Its **own network namespace** (containers in the same Pod can talk to each other via `localhost`)
- Shared **storage volumes** (containers in the same Pod can read/write the same files)

---

### Single Container Pod (Most Common — 99% of the time)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: my-app
      image: nginx:1.25
      ports:
        - containerPort: 80
```

One Pod = One Container. Simple. This is what you'll use almost always.

---

### Multi-Container Pod (Sidecar Pattern)

Sometimes you need a helper container alongside your main app — for logging, proxying, or syncing data. They go in the **same Pod**.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-with-sidecar
spec:
  containers:
    - name: my-app
      image: my-app:1.0
      ports:
        - containerPort: 8080

    - name: log-shipper
      image: fluentd:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

  volumes:
    - name: shared-logs
      emptyDir: {}
```

Here `my-app` writes logs to `/var/log/app`, and `log-shipper` reads from the same path and ships them to a logging system. They share the volume because they're in the same Pod.

**Rule of thumb**: If two containers MUST be on the same machine and MUST share storage/network — put them in one Pod. Otherwise, separate Pods.

---

### Pod Lifecycle

A Pod goes through these phases:

| Phase | What's Happening |
|---|---|
| **Pending** | Pod accepted by K8s, but container(s) not running yet. Could be pulling image, waiting for node scheduling. |
| **Running** | At least one container is running. |
| **Succeeded** | All containers exited with code 0. Common for Jobs. |
| **Failed** | At least one container exited with a non-zero code. |
| **Unknown** | K8s can't determine the state. Usually a node communication issue. |

---

### Pod Restart Policies

```yaml
spec:
  restartPolicy: Always    # default — always restart crashed containers
  # restartPolicy: OnFailure  — restart only if exit code != 0
  # restartPolicy: Never       — never restart
```

- `Always` → used for long-running apps (web servers, APIs)
- `OnFailure` → used for Jobs (run once, retry on failure)
- `Never` → used for one-shot tasks

---

### Health Checks (Probes)

Kubernetes doesn't just start your Pod and forget about it. It actively monitors it using probes.

#### Liveness Probe — "Is the container alive?"

If this fails, K8s **kills and restarts** the container.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

#### Readiness Probe — "Is the container ready to receive traffic?"

If this fails, K8s **removes the Pod from the Service** (no traffic sent to it), but does NOT restart it.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```

#### Startup Probe — "Has the container finished starting up?"

For slow-starting apps. Until this passes, liveness and readiness probes are paused.

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

This gives the app up to 300 seconds (30 × 10) to start before K8s considers it dead.

---

### Resource Requests & Limits

Every Pod should declare how much CPU and memory it needs.

```yaml
resources:
  requests:          # guaranteed minimum
    cpu: 100m        # 100 millicores = 0.1 CPU
    memory: 128Mi
  limits:            # maximum allowed
    cpu: 500m
    memory: 256Mi
```

- **Requests** = what the scheduler uses to place the Pod on a node. "I need at least this much."
- **Limits** = the ceiling. If the container exceeds memory limit → it gets **OOMKilled**. If it exceeds CPU limit → it gets **throttled**.

**What happens if you don't set these?**
- Pod can consume unlimited resources on the node
- Can starve other Pods
- Scheduler has no idea where to place it efficiently

---

### Pod Networking — How Pods Talk to Each Other

- Every Pod gets a **unique IP** in the cluster
- Pod A can talk to Pod B using Pod B's IP directly (flat network, no NAT)
- But Pod IPs are **ephemeral** — they change when Pods restart
- That's why you use **Services** (ClusterIP, NodePort, LoadBalancer) to get a stable endpoint

---

### Important: Pods Are Ephemeral

Pods are **disposable**. They can be killed, rescheduled, or replaced at any time. Never treat a Pod as permanent.

- Node goes down → Pods on that node are gone
- Deployment rolls out a new version → old Pods are terminated, new ones created
- Pod crashes → depending on restart policy, it may or may not come back

This is why you **never create Pods directly** in production. You use controllers like ReplicaSets (or Deployments) that manage Pods for you.

---

---

## ReplicaSets

### What is a ReplicaSet?

A ReplicaSet makes sure a **specific number of identical Pods are always running**. If a Pod dies, the ReplicaSet creates a new one. If there are too many, it kills the extras.

**Simple analogy**: You tell a manager "I always need 3 workers on the floor." If one leaves, the manager hires a replacement. If somehow there are 4, the manager sends one home.

---

### How It Works

```
You say: "I want 3 replicas of my-app"

ReplicaSet sees: 0 Pods running
Action: Creates 3 Pods

One Pod crashes.

ReplicaSet sees: 2 Pods running
Action: Creates 1 more Pod

Someone manually creates an extra Pod with matching labels.

ReplicaSet sees: 4 Pods running
Action: Terminates 1 Pod
```

It's a **constant reconciliation loop** — always comparing desired state vs actual state.

---

### ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:1.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
```

Let's break this down:

| Field | What It Does |
|---|---|
| `replicas: 3` | "I want 3 Pods running at all times" |
| `selector.matchLabels` | How the ReplicaSet finds its Pods. It looks for Pods with label `app: my-app` |
| `template` | The Pod blueprint. When ReplicaSet needs to create a new Pod, it uses this template |

---

### The Label Connection (Critical to Understand)

The ReplicaSet doesn't "own" Pods by name. It owns them **by labels**.

```
selector.matchLabels.app: my-app  ←→  template.metadata.labels.app: my-app
```

These **MUST match**. If they don't, K8s will reject the ReplicaSet.

**What this means in practice:**
- If you manually create a Pod with label `app: my-app`, the ReplicaSet will count it as one of its own
- If you remove the label from a Pod, the ReplicaSet will think it lost a Pod and create a new one
- The labeled Pod that you "detached" keeps running — it's just no longer managed by the ReplicaSet

This is actually useful for debugging — you can detach a broken Pod from the ReplicaSet (by changing its label), inspect it, while the ReplicaSet automatically replaces it with a healthy one.

---

### Scaling

#### Manual scaling

```bash
kubectl scale replicaset my-app-rs --replicas=5
```

#### Or edit the YAML

```bash
kubectl edit rs my-app-rs
# change replicas: 3 → replicas: 5
```

#### Scale to zero (stop all Pods without deleting the ReplicaSet)

```bash
kubectl scale replicaset my-app-rs --replicas=0
```

---

### ReplicaSet vs Deployment — Why You Almost Never Create ReplicaSets Directly

Here's the thing — **you should almost never create a ReplicaSet yourself**.

Instead, you create a **Deployment**, and the Deployment creates and manages the ReplicaSet for you.

```
Deployment → manages → ReplicaSet → manages → Pods
```

Why? Because a ReplicaSet alone **cannot do rolling updates**.

| Feature | ReplicaSet | Deployment |
|---|---|---|
| Maintains desired Pod count | ✅ | ✅ |
| Rolling updates | ❌ | ✅ |
| Rollback to previous version | ❌ | ✅ |
| Update history | ❌ | ✅ |
| Pause/Resume rollout | ❌ | ✅ |

When you update a Deployment (e.g., change the image tag), it:
1. Creates a **new ReplicaSet** with the new Pod template
2. Gradually scales up the new ReplicaSet
3. Gradually scales down the old ReplicaSet
4. Keeps the old ReplicaSet around (with 0 replicas) for rollback

```bash
# See the ReplicaSets managed by a Deployment
kubectl get rs

# Example output:
# NAME                    DESIRED   CURRENT   READY
# my-app-6b7f8d4c5f      3         3         3      ← current (new)
# my-app-5a9e3b2d1c      0         0         0      ← previous (kept for rollback)
```

---

### When Would You Actually Use a ReplicaSet Directly?

Almost never. The only cases:

- You need a set of identical Pods that will **never be updated** (rare)
- You're building a custom controller that manages its own ReplicaSets
- You're learning Kubernetes and want to understand the building blocks

For everything else → use a **Deployment**.

---

### Useful Commands

```bash
# List all Pods managed by a ReplicaSet
kubectl get pods -l app=my-app

# Describe a ReplicaSet (see events, conditions)
kubectl describe rs my-app-rs

# Check ReplicaSet status
kubectl get rs my-app-rs -o wide

# Delete a ReplicaSet (also deletes its Pods)
kubectl delete rs my-app-rs

# Delete a ReplicaSet but keep its Pods running (orphan them)
kubectl delete rs my-app-rs --cascade=orphan
```

---

## Quick Summary

| Concept | One-Liner |
|---|---|
| **Pod** | Smallest deployable unit. Wraps one or more containers. Gets its own IP. Ephemeral. |
| **ReplicaSet** | Ensures N identical Pods are always running. Self-heals by replacing dead Pods. |
| **Deployment** | Manages ReplicaSets. Adds rolling updates, rollbacks, and history. Use this in production. |

```
You create → Deployment → creates → ReplicaSet → creates → Pods
```

That's the hierarchy. Understand this and you understand 80% of how workloads run in Kubernetes.


---

---

## Services

### What is a Service?

A Service is a **stable network endpoint** that gives you a consistent way to reach a set of Pods.

Remember how Pods are ephemeral? They get created, destroyed, rescheduled — and every time they come back, they get a **new IP**. You can't rely on Pod IPs.

A Service solves this. It sits in front of your Pods and provides:
- A **fixed IP** (ClusterIP) that never changes
- A **DNS name** (e.g., `my-app.my-namespace.svc.cluster.local`)
- **Load balancing** across all matching Pods

**Simple analogy**: Pods are like employees who keep changing desks. A Service is the reception desk — you always call the same number, and the receptionist routes you to whoever is available.

---

### How Does a Service Find Its Pods?

Using **label selectors** — same concept as ReplicaSets.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app       # finds all Pods with this label
  ports:
    - port: 80        # port the Service listens on
      targetPort: 8080 # port the container is actually listening on
```

Any Pod with label `app: my-app` becomes a backend for this Service. Pods come and go — the Service automatically updates its list.

Behind the scenes, Kubernetes maintains an **Endpoints** object that tracks the IPs of all matching Pods:

```bash
# See which Pod IPs are behind a Service
kubectl get endpoints my-app-svc

# Output:
# NAME          ENDPOINTS                                   AGE
# my-app-svc    10.0.1.5:8080,10.0.2.8:8080,10.0.3.2:8080  5m
```

---

### Port vs TargetPort vs NodePort — The Confusion Killer

This trips up everyone. Let's clear it up:

```yaml
ports:
  - port: 80          # the port the SERVICE listens on (what other Pods use to connect)
  - targetPort: 8080  # the port your CONTAINER is actually listening on
  - nodePort: 30080   # the port opened on every NODE (only for NodePort/LoadBalancer type)
```

```
Client → port (80) → Service → targetPort (8080) → Container
```

If `targetPort` is not specified, it defaults to the same value as `port`.

---

---

### Service Types

There are 4 types. Each one builds on top of the previous.

---

### 1. ClusterIP (Default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  type: ClusterIP       # this is the default, you can omit it
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

**What it does:**
- Creates a virtual IP (ClusterIP) that's only reachable **inside the cluster**
- Other Pods can reach it via `my-app-svc.my-namespace.svc.cluster.local` or just `my-app-svc` (if in the same namespace)
- No external access at all

**When to use:**
- Internal communication between microservices
- Backend APIs that only other Pods need to talk to
- Databases, caches, message queues

```
Pod A → my-app-svc:80 → (load balanced) → Pod B, Pod C, Pod D
```

**DNS resolution inside the cluster:**

```bash
# From any Pod in the same namespace
curl http://my-app-svc:80

# From a Pod in a different namespace
curl http://my-app-svc.my-namespace.svc.cluster.local:80
```

---

### 2. NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080     # optional, K8s picks one from 30000-32767 if omitted
```

**What it does:**
- Everything ClusterIP does, PLUS
- Opens a specific port (30000–32767) on **every node** in the cluster
- External traffic can reach the Service via `<any-node-ip>:30080`

**When to use:**
- Quick external access for testing/debugging
- When you don't have a load balancer set up yet

```
External Client → NodeIP:30080 → Service:80 → Pod:8080
```

**Downsides:**
- Ugly port numbers (30000+)
- You need to know a node's IP
- If a node goes down, that entry point is gone
- Not production-friendly on its own

---

### 3. LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"    # use NLB on AWS
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

**What it does:**
- Everything NodePort does, PLUS
- Provisions an **actual cloud load balancer** (AWS ELB/NLB, GCP LB, Azure LB)
- Gives you an external IP or DNS name that routes traffic to your Pods

**When to use:**
- Exposing a single service directly to the internet
- Simple setups where you don't need path-based routing

```
Internet → AWS NLB (external IP) → Service:80 → Pod:8080
```

**On EKS specifically:**
- By default, it creates a **Classic Load Balancer** (old)
- Add the annotation above to get an **NLB** (recommended)
- If you have the AWS Load Balancer Controller installed, you get more control

**Downsides:**
- **One LoadBalancer per Service** = one cloud LB per service = **expensive**
- If you have 20 services, that's 20 load balancers and 20 bills
- No path-based routing (`/api` → service A, `/web` → service B)
- That's why most people use **Ingress** instead (covered later)

---

### 4. ExternalName

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: my-database.abc123.us-east-1.rds.amazonaws.com
```

**What it does:**
- No proxying, no ClusterIP, no selectors
- Just creates a **DNS CNAME record** inside the cluster
- When a Pod resolves `external-db`, it gets back `my-database.abc123.us-east-1.rds.amazonaws.com`

**When to use:**
- Pointing to external services (RDS, ElastiCache, third-party APIs) using a Kubernetes-native DNS name
- Makes it easy to swap the external endpoint later without changing app code

```
Pod → external-db → DNS resolves to → my-database.abc123.us-east-1.rds.amazonaws.com
```

---

### Quick Comparison

| Type | Accessible From | Creates Cloud LB | Use Case |
|---|---|---|---|
| **ClusterIP** | Inside cluster only | No | Internal service-to-service |
| **NodePort** | Inside cluster + Node IP:Port | No | Dev/testing external access |
| **LoadBalancer** | Inside cluster + External IP | Yes ($$) | Single service exposed to internet |
| **ExternalName** | Inside cluster (DNS alias) | No | Pointing to external resources |

---

---

### Headless Service (ClusterIP: None)

Sometimes you don't want load balancing. You want to talk to **specific Pods** directly.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-db-headless
spec:
  clusterIP: None        # this makes it headless
  selector:
    app: my-db
  ports:
    - port: 5432
      targetPort: 5432
```

**What it does:**
- No ClusterIP is assigned
- DNS lookup returns the **individual Pod IPs** instead of a single virtual IP
- No load balancing — the client decides which Pod to connect to

```bash
# Normal Service DNS lookup
nslookup my-app-svc → 10.96.0.15 (single ClusterIP)

# Headless Service DNS lookup
nslookup my-db-headless → 10.0.1.5, 10.0.2.8, 10.0.3.2 (all Pod IPs)
```

**When to use:**
- **StatefulSets** — databases like PostgreSQL, MongoDB, Kafka where each Pod has a unique identity
- When your app needs to discover and connect to specific Pods
- Service meshes that handle their own load balancing

With StatefulSets + Headless Service, each Pod gets a **stable DNS name**:

```
my-db-0.my-db-headless.my-namespace.svc.cluster.local
my-db-1.my-db-headless.my-namespace.svc.cluster.local
my-db-2.my-db-headless.my-namespace.svc.cluster.local
```

These DNS names persist even if the Pod is rescheduled.

---

---

### Session Affinity (Sticky Sessions)

By default, Services load balance randomly across Pods. If you need the same client to always hit the same Pod:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 1800    # sticky for 30 minutes
  ports:
    - port: 80
      targetPort: 8080
```

**When to use:**
- Apps that store session data locally (not recommended, but sometimes unavoidable)
- WebSocket connections that need to stay on the same Pod

**Better approach:** Make your app stateless and store sessions in Redis/DynamoDB instead.

---

---

### Multi-Port Services

If your app exposes multiple ports (e.g., HTTP + metrics):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
  ports:
    - name: http           # name is REQUIRED when you have multiple ports
      port: 80
      targetPort: 8080
    - name: metrics
      port: 9090
      targetPort: 9090
```

---

---

### Services Without Selectors

Sometimes you want a Service that points to something that's NOT a Pod in the cluster — like an external IP or a VM.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service    # must match the Service name
subsets:
  - addresses:
      - ip: 192.168.1.100   # external server IP
      - ip: 192.168.1.101
    ports:
      - port: 80
```

**When to use:**
- Migrating from VMs to Kubernetes gradually — some backends are still on VMs
- Pointing to services in another cluster
- Any external resource that doesn't have a DNS name (otherwise use ExternalName)

---

---

### How Traffic Actually Flows (Under the Hood)

When you create a Service, Kubernetes doesn't spin up a real load balancer inside the cluster. Here's what actually happens:

#### kube-proxy

Every node runs `kube-proxy`, which watches for Service changes and sets up network rules.

**3 modes of kube-proxy:**

| Mode | How It Works | Performance |
|---|---|---|
| **iptables** (default) | Creates iptables rules for each Service. Random Pod selection. | Good for most clusters |
| **IPVS** | Uses Linux IPVS kernel module. Real load balancing algorithms (round-robin, least connections). | Better for large clusters (1000+ Services) |
| **userspace** | Old mode. Proxies through kube-proxy process itself. | Slow. Don't use. |

For EKS, iptables mode is the default and works fine for most cases. If you have hundreds of Services, consider switching to IPVS.

```
Client Pod → ClusterIP → iptables/IPVS rules on the node → randomly selected backend Pod
```

The ClusterIP is a **virtual IP** — it doesn't exist on any network interface. It only exists in the iptables/IPVS rules.

---

---

### Service DNS — How It All Connects

Kubernetes runs an internal DNS server (CoreDNS). Every Service gets a DNS entry automatically.

| DNS Format | Resolves To |
|---|---|
| `my-svc` | ClusterIP (same namespace only) |
| `my-svc.my-namespace` | ClusterIP (cross-namespace) |
| `my-svc.my-namespace.svc.cluster.local` | ClusterIP (fully qualified) |

```bash
# From a Pod, all of these work (if in the same namespace):
curl http://my-app-svc
curl http://my-app-svc:80
curl http://my-app-svc.default.svc.cluster.local
```

**Debugging DNS issues:**

```bash
# Run a debug Pod
kubectl run dns-test --image=busybox:1.36 --rm -it -- sh

# Inside the Pod
nslookup my-app-svc
nslookup my-app-svc.my-namespace.svc.cluster.local

# Check if CoreDNS is running
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

---

---

### Common Problems & Fixes

| Problem | Cause | Fix |
|---|---|---|
| Service has no endpoints | Selector labels don't match Pod labels | `kubectl get endpoints my-svc` — if empty, check labels with `kubectl get pods --show-labels` |
| Can't reach Service from outside | Using ClusterIP type | Switch to NodePort or LoadBalancer, or use Ingress |
| Connection refused | Pod is running but container isn't listening on targetPort | Verify with `kubectl exec -it pod-name -- curl localhost:8080` |
| DNS not resolving | CoreDNS is down or misconfigured | `kubectl get pods -n kube-system -l k8s-app=kube-dns` and check logs |
| LoadBalancer stuck in `<pending>` | No cloud controller / wrong permissions | On EKS, check if AWS Load Balancer Controller is installed and IAM roles are correct |
| Traffic not balanced evenly | Session affinity enabled, or too few connections | Check `sessionAffinity` setting. For long-lived connections (gRPC), consider client-side load balancing |
| Intermittent timeouts | Pod failing readiness probe, getting removed from endpoints | Check readiness probe config and Pod logs |

---

---

### Useful Commands

```bash
# List all Services
kubectl get svc

# Get details of a Service
kubectl describe svc my-app-svc

# See which Pods are behind a Service (endpoints)
kubectl get endpoints my-app-svc

# Test connectivity from inside the cluster
kubectl run curl-test --image=curlimages/curl --rm -it -- curl http://my-app-svc:80

# Check what port a LoadBalancer got
kubectl get svc my-app-svc -o wide

# Get the external IP/hostname of a LoadBalancer Service
kubectl get svc my-app-svc -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Debug: check iptables rules for a Service (on a node)
sudo iptables -t nat -L KUBE-SERVICES | grep my-app-svc
```

---

---

### Services vs Ingress — When to Use What

| | Service (LoadBalancer) | Ingress |
|---|---|---|
| **What it does** | Exposes ONE service externally | Exposes MULTIPLE services via routing rules |
| **Routing** | No path/host-based routing | Path-based (`/api`, `/web`) and host-based (`api.example.com`) |
| **Cloud LB** | One per Service (expensive) | One LB for all services (cost-effective) |
| **TLS** | Configured per Service | Centralized TLS termination |
| **Use when** | Single service, simple setup | Multiple services, production setup |

```
Without Ingress:
  app-a → LoadBalancer ($$$)
  app-b → LoadBalancer ($$$)
  app-c → LoadBalancer ($$$)

With Ingress:
  Single ALB → /api → app-a
              → /web → app-b
              → /admin → app-c
```

Ingress needs an **Ingress Controller** (like AWS Load Balancer Controller, nginx-ingress, or Traefik) to actually work. The Ingress resource alone does nothing without a controller.

---

## Quick Summary — Services

| Concept | One-Liner |
|---|---|
| **ClusterIP** | Internal-only stable IP. Default. Use for service-to-service communication. |
| **NodePort** | Opens a port on every node. Quick external access for dev/testing. |
| **LoadBalancer** | Provisions a real cloud LB. One per service. Simple but expensive at scale. |
| **ExternalName** | DNS alias to an external resource. No proxying. |
| **Headless** | No ClusterIP. Returns individual Pod IPs. Used with StatefulSets. |
| **Ingress** | Not a Service type, but the production way to expose multiple services behind one LB with routing rules. |

```
Internal traffic:  Pod → ClusterIP Service → Backend Pods
External traffic:  Internet → Ingress (ALB) → ClusterIP Service → Backend Pods
```

That's Services in Kubernetes. The ClusterIP + Ingress combo is what you'll use 90% of the time in production on EKS.
