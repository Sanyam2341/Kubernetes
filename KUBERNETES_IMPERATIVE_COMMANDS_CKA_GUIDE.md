# Kubernetes Imperative Commands - CKA Exam Guide

## Table of Contents
- [What are Imperative Commands?](#what-are-imperative-commands)
- [Essential Commands](#essential-commands)
  - [Pod Management](#1-pod-management)
  - [Deployment Management](#2-deployment-management)
  - [Service Management](#3-service-management)
  - [Namespace Management](#4-namespace-management)
  - [ConfigMap & Secret](#5-configmap--secret)
  - [ServiceAccount & RBAC](#6-serviceaccount--rbac)
  - [Job & CronJob](#7-job--cronjob)
  - [Ingress](#8-ingress)
- [Critical Exam Tips](#critical-exam-tips)
- [Time-Saving Aliases](#time-saving-aliases)
- [Practice Scenarios](#practice-scenarios)
- [Key Points for Exam Success](#key-points-for-exam-success)

---

## What are Imperative Commands?

Imperative commands let you create/modify Kubernetes resources directly via `kubectl` without writing YAML files. They're faster for the exam and real-world quick tasks.

**Benefits:**
- ⚡ Faster than writing YAML from scratch
- 🎯 Perfect for CKA exam time constraints
- 🔧 Great for quick troubleshooting and testing
- 📝 Can generate YAML templates for further customization

---

## Essential Commands

### **1. Pod Management**

#### Create Basic Pods
```bash
# Create a simple pod
kubectl run nginx --image=nginx

# Create pod with specific port
kubectl run nginx --image=nginx --port=80

# Create pod with labels
kubectl run nginx --image=nginx --labels="app=web,env=prod"

# Create pod with environment variables
kubectl run nginx --image=nginx --env="DB_HOST=mysql" --env="DB_PORT=3306"

# Create pod in specific namespace
kubectl run nginx --image=nginx -n development

# Create pod and expose it (creates pod + service)
kubectl run nginx --image=nginx --port=80 --expose
```

#### Generate Pod YAML
```bash
# Dry run - generate YAML without creating
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Create and save YAML to file
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Generate with multiple containers (edit after generation)
kubectl run multi-pod --image=nginx --dry-run=client -o yaml > pod.yaml
```

#### Pod Operations
```bash
# Delete pod
kubectl delete pod nginx

# Delete pod forcefully
kubectl delete pod nginx --force --grace-period=0

# Get pod details
kubectl get pod nginx -o wide
kubectl get pod nginx -o yaml

# Describe pod
kubectl describe pod nginx

# View logs
kubectl logs nginx
kubectl logs nginx -f                    # Follow logs
kubectl logs nginx --previous            # Previous container logs
kubectl logs nginx -c container-name     # Specific container in multi-container pod

# Execute commands in pod
kubectl exec -it nginx -- /bin/bash
kubectl exec nginx -- ls /usr/share/nginx/html

# Copy files to/from pod
kubectl cp nginx:/tmp/file.txt ./file.txt
kubectl cp ./file.txt nginx:/tmp/file.txt
```

---

### **2. Deployment Management**

#### Create Deployments
```bash
# Create basic deployment
kubectl create deployment nginx --image=nginx

# Create deployment with replicas
kubectl create deployment nginx --image=nginx --replicas=3

# Create deployment with port
kubectl create deployment nginx --image=nginx --port=80

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Create deployment with multiple replicas and save to file
kubectl create deployment webapp --image=nginx --replicas=5 --dry-run=client -o yaml > webapp-deploy.yaml
```

#### Manage Deployments
```bash
# Scale deployment
kubectl scale deployment nginx --replicas=5

# Autoscale deployment
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Update image (rolling update)
kubectl set image deployment/nginx nginx=nginx:1.21
kubectl set image deployment/nginx nginx=nginx:1.21 --record

# Rollout operations
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=2
kubectl rollout restart deployment/nginx

# Edit deployment
kubectl edit deployment nginx

# Delete deployment
kubectl delete deployment nginx
```

---

### **3. Service Management**

#### Create Services
```bash
# Expose pod as ClusterIP service (default)
kubectl expose pod nginx --port=80 --target-port=80

# Expose deployment as ClusterIP
kubectl expose deployment nginx --port=80 --target-port=80

# Expose deployment as NodePort
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort

# Expose deployment as LoadBalancer
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Expose with specific name
kubectl expose deployment nginx --port=80 --name=nginx-service

# Create ClusterIP service imperatively
kubectl create service clusterip nginx --tcp=80:80

# Create NodePort service with specific node port
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080

# Create LoadBalancer service
kubectl create service loadbalancer nginx --tcp=80:80
```

#### Generate Service YAML
```bash
# Generate service YAML from pod
kubectl expose pod nginx --port=80 --dry-run=client -o yaml > service.yaml

# Generate service YAML from deployment
kubectl expose deployment nginx --port=80 --type=NodePort --dry-run=client -o yaml > service.yaml
```

#### Service Operations
```bash
# Get services
kubectl get svc
kubectl get svc -o wide

# Describe service
kubectl describe svc nginx

# Delete service
kubectl delete svc nginx

# Edit service
kubectl edit svc nginx
```

---

### **4. Namespace Management**

```bash
# Create namespace
kubectl create namespace development
kubectl create namespace production
kubectl create ns staging

# Generate namespace YAML
kubectl create namespace dev --dry-run=client -o yaml > namespace.yaml

# List all namespaces
kubectl get namespaces
kubectl get ns

# Set default namespace for current context
kubectl config set-context --current --namespace=development

# View current context
kubectl config view --minify | grep namespace

# Delete namespace (deletes all resources in it)
kubectl delete namespace development

# Get resources in specific namespace
kubectl get pods -n development
kubectl get all -n development

# Get resources across all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A
```

---

### **5. ConfigMap & Secret**

#### ConfigMap Commands
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config --from-literal=DB_HOST=mysql --from-literal=DB_PORT=3306

# Create ConfigMap from multiple literals
kubectl create configmap app-config \
  --from-literal=DB_HOST=mysql \
  --from-literal=DB_PORT=3306 \
  --from-literal=DB_NAME=myapp

# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.properties

# Create ConfigMap from directory
kubectl create configmap app-config --from-file=config-dir/

# Create ConfigMap from env file
kubectl create configmap app-config --from-env-file=app.env

# Generate ConfigMap YAML
kubectl create configmap app-config --from-literal=KEY=value --dry-run=client -o yaml > configmap.yaml

# View ConfigMap
kubectl get configmap app-config
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Delete ConfigMap
kubectl delete configmap app-config
```

#### Secret Commands
```bash
# Create generic Secret from literal values
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Create Secret from file
kubectl create secret generic db-secret --from-file=username.txt --from-file=password.txt

# Create TLS Secret
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/key.key

# Create Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# Generate Secret YAML (note: values will be base64 encoded)
kubectl create secret generic db-secret \
  --from-literal=password=secret123 \
  --dry-run=client -o yaml > secret.yaml

# View Secret (values are base64 encoded)
kubectl get secret db-secret
kubectl describe secret db-secret
kubectl get secret db-secret -o yaml

# Decode secret value
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode

# Delete Secret
kubectl delete secret db-secret
```

---

### **6. ServiceAccount & RBAC**

#### ServiceAccount Commands
```bash
# Create ServiceAccount
kubectl create serviceaccount my-sa
kubectl create sa app-sa

# Generate ServiceAccount YAML
kubectl create serviceaccount my-sa --dry-run=client -o yaml > sa.yaml

# View ServiceAccount
kubectl get serviceaccount
kubectl get sa
kubectl describe sa my-sa

# Delete ServiceAccount
kubectl delete serviceaccount my-sa
```

#### RBAC Commands

**ClusterRole (Cluster-wide permissions)**
```bash
# Create ClusterRole with specific verbs and resources
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# Create ClusterRole for multiple resources
kubectl create clusterrole resource-reader \
  --verb=get,list,watch \
  --resource=pods,deployments,services

# Create ClusterRole with all verbs
kubectl create clusterrole admin-role --verb=* --resource=*

# Generate ClusterRole YAML
kubectl create clusterrole pod-reader --verb=get,list --resource=pods --dry-run=client -o yaml > clusterrole.yaml
```

**Role (Namespace-scoped permissions)**
```bash
# Create Role in specific namespace
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n development

# Create Role with multiple resources
kubectl create role app-role \
  --verb=get,list,create,update \
  --resource=pods,services \
  -n development

# Generate Role YAML
kubectl create role pod-reader --verb=get,list --resource=pods -n dev --dry-run=client -o yaml > role.yaml
```

**ClusterRoleBinding (Bind ClusterRole)**
```bash
# Bind ClusterRole to ServiceAccount
kubectl create clusterrolebinding pod-reader-binding \
  --clusterrole=pod-reader \
  --serviceaccount=default:my-sa

# Bind ClusterRole to User
kubectl create clusterrolebinding admin-binding \
  --clusterrole=cluster-admin \
  --user=john@example.com

# Bind ClusterRole to Group
kubectl create clusterrolebinding dev-binding \
  --clusterrole=edit \
  --group=developers

# Generate ClusterRoleBinding YAML
kubectl create clusterrolebinding pod-reader-binding \
  --clusterrole=pod-reader \
  --serviceaccount=default:my-sa \
  --dry-run=client -o yaml > clusterrolebinding.yaml
```

**RoleBinding (Bind Role in namespace)**
```bash
# Bind Role to ServiceAccount in namespace
kubectl create rolebinding pod-reader-binding \
  --role=pod-reader \
  --serviceaccount=default:my-sa \
  -n development

# Bind Role to User
kubectl create rolebinding dev-binding \
  --role=developer \
  --user=jane@example.com \
  -n development

# Generate RoleBinding YAML
kubectl create rolebinding pod-reader-binding \
  --role=pod-reader \
  --serviceaccount=default:my-sa \
  -n dev \
  --dry-run=client -o yaml > rolebinding.yaml
```

#### Check Permissions
```bash
# Check if you can perform action
kubectl auth can-i create pods
kubectl auth can-i delete deployments
kubectl auth can-i get pods --as=system:serviceaccount:default:my-sa
kubectl auth can-i list secrets -n kube-system

# Check all permissions for current user
kubectl auth can-i --list

# Check permissions for specific user
kubectl auth can-i --list --as=john@example.com
```

---

### **7. Job & CronJob**

#### Job Commands
```bash
# Create simple Job
kubectl create job hello --image=busybox -- echo "Hello World"

# Create Job with command
kubectl create job pi --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'

# Create Job from CronJob
kubectl create job test-job --from=cronjob/my-cronjob

# Generate Job YAML
kubectl create job hello --image=busybox --dry-run=client -o yaml > job.yaml

# View Jobs
kubectl get jobs
kubectl describe job hello

# View Job logs
kubectl logs job/hello

# Delete Job
kubectl delete job hello

# Delete Job and its pods
kubectl delete job hello --cascade=foreground
```

#### CronJob Commands
```bash
# Create CronJob (runs every 5 minutes)
kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" -- echo "Hello"

# Create CronJob (runs daily at midnight)
kubectl create cronjob backup --image=backup-tool --schedule="0 0 * * *" -- /backup.sh

# Create CronJob (runs every hour)
kubectl create cronjob hourly-task --image=busybox --schedule="0 * * * *" -- date

# Generate CronJob YAML
kubectl create cronjob hello --image=busybox --schedule="*/5 * * * *" --dry-run=client -o yaml > cronjob.yaml

# View CronJobs
kubectl get cronjobs
kubectl get cj
kubectl describe cronjob hello

# Suspend CronJob
kubectl patch cronjob hello -p '{"spec":{"suspend":true}}'

# Resume CronJob
kubectl patch cronjob hello -p '{"spec":{"suspend":false}}'

# Delete CronJob
kubectl delete cronjob hello
```

**Cron Schedule Format:**
```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │
# * * * * *

Examples:
*/5 * * * *      # Every 5 minutes
0 * * * *        # Every hour
0 0 * * *        # Daily at midnight
0 0 * * 0        # Weekly on Sunday at midnight
0 0 1 * *        # Monthly on 1st at midnight
```

---

### **8. Ingress**

```bash
# Create basic Ingress
kubectl create ingress my-ingress --rule="example.com/=service:80"

# Create Ingress with path
kubectl create ingress my-ingress --rule="example.com/path=service:80"

# Create Ingress with multiple rules
kubectl create ingress my-ingress \
  --rule="example.com/app1=app1-service:80" \
  --rule="example.com/app2=app2-service:80"

# Create Ingress with TLS
kubectl create ingress my-ingress \
  --rule="example.com/=service:80,tls=my-tls-secret"

# Create Ingress with default backend
kubectl create ingress my-ingress \
  --default-backend=default-service:80 \
  --rule="example.com/=service:80"

# Generate Ingress YAML
kubectl create ingress my-ingress \
  --rule="example.com/=service:80" \
  --dry-run=client -o yaml > ingress.yaml

# View Ingress
kubectl get ingress
kubectl get ing
kubectl describe ingress my-ingress

# Edit Ingress
kubectl edit ingress my-ingress

# Delete Ingress
kubectl delete ingress my-ingress
```

---

## Critical Exam Tips

### **The Golden Pattern for CKA**

This is the most important workflow for the exam:

```bash
# Step 1: Generate YAML with dry-run
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml

# Step 2: Edit the file to add/modify specifications
vi deploy.yaml

# Step 3: Apply it
kubectl apply -f deploy.yaml

# Step 4: Verify
kubectl get deployment nginx
kubectl describe deployment nginx
```

### **Quick Editing Commands**

```bash
# Edit existing resource directly
kubectl edit deployment nginx
kubectl edit pod nginx
kubectl edit service nginx

# Replace resource from file (deletes and recreates)
kubectl replace -f deployment.yaml

# Replace with force (for immutable fields)
kubectl replace -f deployment.yaml --force

# Patch resource (partial update)
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'
kubectl patch pod nginx -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.21"}]}}'

# Apply changes from file
kubectl apply -f deployment.yaml
```

### **Useful Output Formats**

```bash
# Wide output (more columns)
kubectl get pods -o wide
kubectl get nodes -o wide

# YAML output
kubectl get pod nginx -o yaml
kubectl get deployment nginx -o yaml

# JSON output
kubectl get pod nginx -o json

# JSONPath (extract specific fields)
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{.items[*].status.podIP}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP

# Show labels
kubectl get pods --show-labels
kubectl get pods -L app,env
```

### **Filtering and Sorting**

```bash
# Filter by label
kubectl get pods -l app=nginx
kubectl get pods -l 'env in (prod,staging)'
kubectl get pods -l app=nginx,env=prod

# Filter by field
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.namespace=default

# Sort by
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by=.status.startTime
```

### **Explain Command (Your Best Friend)**

```bash
# Explain resource structure
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy

# Recursive explain
kubectl explain pod --recursive
kubectl explain deployment.spec --recursive

# Specific field
kubectl explain pod.spec.containers.resources
kubectl explain service.spec.type
```

---

## Time-Saving Aliases

Set these up at the beginning of your exam:

```bash
# Basic aliases
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kgn='kubectl get nodes'
alias kga='kubectl get all'

# Describe aliases
alias kdp='kubectl describe pod'
alias kds='kubectl describe service'
alias kdd='kubectl describe deployment'
alias kdn='kubectl describe node'

# Delete aliases
alias kd='kubectl delete'
alias kdf='kubectl delete -f'
alias kdp='kubectl delete pod'

# Apply/Create aliases
alias ka='kubectl apply -f'
alias kc='kubectl create'

# Logs aliases
alias kl='kubectl logs'
alias klf='kubectl logs -f'

# Exec alias
alias kx='kubectl exec -it'

# Namespace aliases
alias kn='kubectl config set-context --current --namespace'

# Dry-run aliases
alias kdr='kubectl --dry-run=client -o yaml'

# Enable kubectl autocomplete
source <(kubectl completion bash)
complete -F __start_kubectl k

# Add to .bashrc for persistence
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

### **Using Aliases in Exam**

```bash
# Instead of:
kubectl get pods

# Use:
k get pods
# or
kgp

# Instead of:
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml

# Use:
k create deploy nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
```

---

## Practice Scenarios

### **Scenario 1: Create and Expose Application**
```bash
# Create deployment with 3 replicas
kubectl create deployment webapp --image=nginx --replicas=3

# Expose as NodePort service
kubectl expose deployment webapp --port=80 --type=NodePort

# Verify
kubectl get deployment webapp
kubectl get svc webapp
kubectl get pods -l app=webapp
```

### **Scenario 2: ConfigMap and Secret in Pod**
```bash
# Create ConfigMap
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

# Create Secret
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Generate pod YAML
kubectl run myapp --image=nginx --dry-run=client -o yaml > pod.yaml

# Edit pod.yaml to add ConfigMap and Secret
vi pod.yaml

# Apply
kubectl apply -f pod.yaml

# Verify
kubectl describe pod myapp
```

### **Scenario 3: RBAC Setup**
```bash
# Create namespace
kubectl create namespace dev

# Create ServiceAccount
kubectl create serviceaccount dev-sa -n dev

# Create Role
kubectl create role pod-manager \
  --verb=get,list,create,delete \
  --resource=pods \
  -n dev

# Create RoleBinding
kubectl create rolebinding dev-sa-binding \
  --role=pod-manager \
  --serviceaccount=dev:dev-sa \
  -n dev

# Test permissions
kubectl auth can-i create pods --as=system:serviceaccount:dev:dev-sa -n dev
kubectl auth can-i delete deployments --as=system:serviceaccount:dev:dev-sa -n dev
```

### **Scenario 4: Multi-Container Pod**
```bash
# Generate base pod YAML
kubectl run multi-pod --image=nginx --dry-run=client -o yaml > multi-pod.yaml

# Edit to add sidecar container
vi multi-pod.yaml
```

Add second container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Sidecar running"; sleep 10; done']
```

```bash
# Apply
kubectl apply -f multi-pod.yaml

# Verify
kubectl get pod multi-pod
kubectl logs multi-pod -c nginx
kubectl logs multi-pod -c sidecar
```

### **Scenario 5: Job and CronJob**
```bash
# Create Job that runs once
kubectl create job backup --image=backup-tool -- /backup.sh

# Create CronJob that runs daily
kubectl create cronjob daily-backup \
  --image=backup-tool \
  --schedule="0 2 * * *" \
  -- /backup.sh

# Manually trigger job from cronjob
kubectl create job manual-backup --from=cronjob/daily-backup

# Verify
kubectl get jobs
kubectl get cronjobs
kubectl logs job/backup
```

### **Scenario 6: Resource Updates**
```bash
# Create deployment
kubectl create deployment app --image=nginx:1.19 --replicas=3

# Update image
kubectl set image deployment/app nginx=nginx:1.21 --record

# Check rollout status
kubectl rollout status deployment/app

# View rollout history
kubectl rollout history deployment/app

# Rollback to previous version
kubectl rollout undo deployment/app

# Scale deployment
kubectl scale deployment app --replicas=5

# Autoscale
kubectl autoscale deployment app --min=3 --max=10 --cpu-percent=80
```

### **Scenario 7: Troubleshooting Pod**
```bash
# Create pod
kubectl run debug-pod --image=nginx

# Check pod status
kubectl get pod debug-pod
kubectl describe pod debug-pod

# View logs
kubectl logs debug-pod
kubectl logs debug-pod --previous

# Execute commands
kubectl exec debug-pod -- nginx -v
kubectl exec -it debug-pod -- /bin/bash

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Delete and recreate
kubectl delete pod debug-pod --force --grace-period=0
kubectl run debug-pod --image=nginx
```

---

## Key Points for Exam Success

### **1. Master the Dry-Run Pattern**
```bash
# Always use dry-run to generate YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml > service.yaml
```

### **2. Use kubectl explain**
```bash
# When you forget field names or structure
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy
kubectl explain service.spec
```

### **3. Practice Speed**
- Set up aliases immediately
- Use tab completion
- Practice typing commands quickly
- Know when to use imperative vs declarative

### **4. Know the Documentation**
During the exam, you can access:
- https://kubernetes.io/docs/
- https://kubernetes.io/blog/

Bookmark these pages:
- kubectl Cheat Sheet
- Pod documentation
- Deployment documentation
- Service documentation

### **5. Time Management**
- Easy questions first (imperative commands)
- Skip and return to complex questions
- Use dry-run to save time
- Don't spend too long on one question

### **6. Verification is Key**
Always verify your work:
```bash
kubectl get <resource>
kubectl describe <resource>
kubectl logs <pod>
```

### **7. Common Mistakes to Avoid**
- Forgetting to specify namespace with `-n`
- Not using `--dry-run=client` (using server instead)
- Forgetting to expose ports when creating services
- Not checking if resource already exists before creating
- Forgetting to verify after creation

### **8. Essential Commands to Memorize**
```bash
# Create resources
kubectl run
kubectl create deployment
kubectl expose
kubectl create configmap
kubectl create secret

# Manage resources
kubectl scale
kubectl set image
kubectl rollout
kubectl edit
kubectl patch

# View resources
kubectl get
kubectl describe
kubectl logs
kubectl explain

# Debug
kubectl exec
kubectl port-forward
kubectl cp
```

### **9. Practice These Daily**
```bash
# Create pod with labels and env vars
kubectl run nginx --image=nginx --labels=app=web,env=prod --env=DB_HOST=mysql

# Create deployment and expose
kubectl create deployment app --image=nginx --replicas=3
kubectl expose deployment app --port=80 --type=NodePort

# Create configmap and secret
kubectl create configmap config --from-literal=key=value
kubectl create secret generic secret --from-literal=password=pass

# RBAC setup
kubectl create serviceaccount sa
kubectl create role role --verb=get,list --resource=pods
kubectl create rolebinding binding --role=role --serviceaccount=default:sa
```

### **10. Exam Day Checklist**
- [ ] Set up aliases and autocomplete
- [ ] Test kubectl connection
- [ ] Bookmark kubernetes.io docs
- [ ] Have kubectl cheat sheet ready
- [ ] Practice dry-run commands
- [ ] Know how to use kubectl explain
- [ ] Understand time limits per question
- [ ] Plan to verify all answers

---

## Additional Resources

### **Official Documentation**
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [kubectl Commands Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)

### **Practice Commands**
```bash
# Generate all resource types for practice
kubectl create deployment test --image=nginx --dry-run=client -o yaml
kubectl create service clusterip test --tcp=80:80 --dry-run=client -o yaml
kubectl create configmap test --from-literal=key=value --dry-run=client -o yaml
kubectl create secret generic test --from-literal=key=value --dry-run=client -o yaml
kubectl create serviceaccount test --dry-run=client -o yaml
kubectl create role test --verb=get --resource=pods --dry-run=client -o yaml
kubectl create rolebinding test --role=test --serviceaccount=default:test --dry-run=client -o yaml
kubectl create job test --image=busybox --dry-run=client -o yaml -- echo "test"
kubectl create cronjob test --image=busybox --schedule="* * * * *" --dry-run=client -o yaml -- echo "test"
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Create pod | `kubectl run nginx --image=nginx` |
| Create deployment | `kubectl create deployment nginx --image=nginx` |
| Expose deployment | `kubectl expose deployment nginx --port=80` |
| Scale deployment | `kubectl scale deployment nginx --replicas=3` |
| Update image | `kubectl set image deployment/nginx nginx=nginx:1.21` |
| Create namespace | `kubectl create namespace dev` |
| Create configmap | `kubectl create configmap config --from-literal=key=value` |
| Create secret | `kubectl create secret generic secret --from-literal=pass=123` |
| Create serviceaccount | `kubectl create serviceaccount sa` |
| Create role | `kubectl create role role --verb=get --resource=pods` |
| Create rolebinding | `kubectl create rolebinding rb --role=role --serviceaccount=default:sa` |
| Create job | `kubectl create job job --image=busybox -- echo "hi"` |
| Create cronjob | `kubectl create cronjob cj --image=busybox --schedule="* * * * *" -- echo "hi"` |
| Generate YAML | `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml` |
| Explain resource | `kubectl explain pod.spec` |
| Get resources | `kubectl get pods -o wide` |
| Describe resource | `kubectl describe pod nginx` |
| View logs | `kubectl logs nginx -f` |
| Execute command | `kubectl exec -it nginx -- bash` |
| Edit resource | `kubectl edit deployment nginx` |
| Delete resource | `kubectl delete pod nginx` |

---

## Good Luck with Your CKA Exam! 🚀

Remember:
- **Practice makes perfect** - Run these commands daily
- **Speed matters** - Use aliases and autocomplete
- **Verify everything** - Always check your work
- **Stay calm** - You can skip and return to questions
- **Use the docs** - kubernetes.io is your friend

**You've got this!** 💪
