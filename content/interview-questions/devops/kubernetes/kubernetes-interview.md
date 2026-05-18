# ☸️ Kubernetes — Complete Scenario-Based Interview Guide

> **80+ scenario-based questions · Real-world answers · Production patterns · YAML manifests**

---

## Kubernetes Interview List

1. Explain Kubernetes architecture to a new team member

---


## 📋 Table of Contents

| # | Category | Questions |
|---|----------|-----------|
| 1 | [🟢 Core Concepts & Architecture](#-core-concepts--architecture) | Q1 – Q8 |
| 2 | [🔵 Pods & Workloads](#-pods--workloads) | Q9 – Q18 |
| 3 | [🟣 Services & Networking](#-services--networking) | Q19 – Q26 |
| 4 | [🟠 Storage & Volumes](#-storage--volumes) | Q27 – Q32 |
| 5 | [🔴 Configuration & Secrets](#-configuration--secrets) | Q33 – Q38 |
| 6 | [🟡 Scheduling & Resource Management](#-scheduling--resource-management) | Q39 – Q45 |
| 7 | [🩵 Security & RBAC](#-security--rbac) | Q46 – Q52 |
| 8 | [🩶 Observability & Troubleshooting](#-observability--troubleshooting) | Q53 – Q60 |
| 9 | [🟤 CI/CD & GitOps](#-cicd--gitops) | Q61 – Q67 |
| 10 | [⚡ Advanced & Production Patterns](#-advanced--production-patterns) | Q68 – Q80 |
| 11 | [📋 Quick Reference Cheatsheet](#-quick-reference-cheatsheet) | — |

---

## 🟢 Core Concepts & Architecture

---

### Q1 — Explain Kubernetes architecture to a new team member

> 🎯 **Scenario:** A developer joining your team asks: *"What is Kubernetes and how does it actually work under the hood?"*

**Answer:**

Kubernetes is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Control Plane (Master) components:**

| Component | Role |
|-----------|------|
| `kube-apiserver` | Front door to K8s — all REST calls go here; validates and persists to etcd |
| `etcd` | Distributed key-value store — single source of truth for all cluster state |
| `kube-scheduler` | Watches for unscheduled pods and assigns them to best-fit nodes |
| `kube-controller-manager` | Runs reconciliation loops (ReplicaSet, Node, Endpoints, etc.) |
| `cloud-controller-manager` | Interacts with cloud provider APIs (AWS, GCP, Azure) for LBs, volumes, routes |

**Worker Node components:**

| Component | Role |
|-----------|------|
| `kubelet` | Agent on each node — ensures containers match the desired spec |
| `kube-proxy` | Maintains iptables/IPVS rules for pod-to-pod and service traffic |
| `Container Runtime` | Runs containers (containerd, CRI-O) |

```
┌──────────────────────────────────────────────────────────┐
│                      CONTROL PLANE                        │
│  ┌────────────┐  ┌──────┐  ┌───────────┐  ┌──────────┐  │
│  │ APIServer  │  │ etcd │  │ Scheduler │  │Controller│  │
│  └────────────┘  └──────┘  └───────────┘  └──────────┘  │
└──────────────────────────────────────────────────────────┘
         │                  │                  │
  ┌──────▼──────┐   ┌───────▼─────┐   ┌───────▼─────┐
  │ Worker Node │   │ Worker Node │   │ Worker Node │
  │  kubelet    │   │  kubelet    │   │  kubelet    │
  │  kube-proxy │   │  kube-proxy │   │  kube-proxy │
  │  [Pods...]  │   │  [Pods...]  │   │  [Pods...]  │
  └─────────────┘   └─────────────┘   └─────────────┘
```

> 💡 **Key insight:** The API server is the **only** component that reads/writes to etcd. All others communicate exclusively through the API server. This makes the API server the single source of authority.

---

### Q2 — What happens step-by-step when you run `kubectl apply -f deployment.yaml`?

> 🎯 **Scenario:** An interviewer asks you to walk through the complete internal flow when applying a manifest.

**Answer:**

```
Step 1: kubectl serializes YAML → HTTP POST to kube-apiserver

Step 2: API Server processing pipeline
  ├── Authentication  (who are you? — cert, bearer token, OIDC)
  ├── Authorization   (are you allowed? — RBAC check)
  ├── Admission Controllers
  │     ├── Mutating  (webhooks that can modify the object)
  │     └── Validating (webhooks that can reject the object)
  └── Persists desired state to etcd

Step 3: Deployment Controller (part of kube-controller-manager)
  → Detects new Deployment in etcd via watch
  → Creates a ReplicaSet object

Step 4: ReplicaSet Controller
  → Detects new ReplicaSet
  → Creates Pod objects (just spec stored in etcd — not running yet)

Step 5: Scheduler
  → Watches for pods with no nodeName assigned
  → Filters nodes: resource fit, taints/tolerations, affinity rules
  → Scores remaining nodes (least requested, node affinity weight, etc.)
  → Writes chosen nodeName to Pod spec in etcd (binding)

Step 6: kubelet on target node
  → Watches API server for pods assigned to its node
  → Pulls container image via container runtime (containerd)
  → Sets up networking (calls CNI plugin: Calico/Cilium/Flannel)
  → Starts containers, runs probes
  → Reports pod status back to API server

Step 7: kube-proxy
  → Detects new pod endpoints
  → Updates iptables/IPVS rules on all nodes
  → Pod is now reachable via Service ClusterIP
```

> ✅ **Admission controllers run BEFORE etcd persistence** — this is where OPA/Gatekeeper, PodSecurity, LimitRanger, and ResourceQuota are enforced. If any validating webhook rejects the object, it never reaches etcd.

---

### Q3 — Difference between Pod, ReplicaSet, Deployment, and StatefulSet

> 🎯 **Scenario:** Your colleague is confused about which workload type to use. Explain with examples.

**Answer:**

| Resource | Purpose | Use When |
|----------|---------|----------|
| **Pod** | Smallest unit; one or more containers | Almost never directly — use higher-level objects |
| **ReplicaSet** | Ensures N copies of a Pod are running | Rarely directly — Deployment manages this |
| **Deployment** | Manages ReplicaSets; rolling updates, rollback | Stateless apps (web servers, APIs) |
| **StatefulSet** | Ordered, stable pod identities + persistent storage per pod | Databases, message queues, Elasticsearch |
| **DaemonSet** | One pod per node (auto) | Log collectors, monitoring agents, CNI plugins |
| **Job** | Run-to-completion task | Batch processing, migrations |
| **CronJob** | Scheduled Job | Backups, reports, cleanup tasks |

```yaml
# Deployment — stateless web app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
```

```yaml
# StatefulSet — PostgreSQL with stable identity
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless   # Required: headless service name
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pg-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  # Each pod gets its own dedicated PVC
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
```

> 💡 **StatefulSet key properties:** Pods get stable DNS names (`postgres-0`, `postgres-1`, `postgres-2`), ordered startup (0 → 1 → 2), ordered shutdown (2 → 1 → 0), and each gets its own PersistentVolumeClaim.

---

### Q4 — What is a Namespace and how do you isolate multiple teams?

> 🎯 **Scenario:** Your company is onboarding 5 teams onto a shared Kubernetes cluster. How do you isolate them properly?

**Answer:**

Namespaces provide **logical isolation** — separate RBAC, resource quotas, and network policies per team or environment.

```yaml
# Namespace per team+environment
apiVersion: v1
kind: Namespace
metadata:
  name: team-alpha-prod
  labels:
    team: alpha
    environment: production
    cost-center: "cc-1234"
```

```yaml
# ResourceQuota — cap total resources per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-alpha-quota
  namespace: team-alpha-prod
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    services: "10"
    persistentvolumeclaims: "20"
    count/deployments.apps: "20"
```

```yaml
# LimitRange — set default container resource requests/limits
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-alpha-prod
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    default:
      cpu: 500m
      memory: 256Mi
    max:
      cpu: "4"
      memory: 4Gi
    min:
      cpu: 50m
      memory: 64Mi
```

```bash
# Set default namespace for current session
kubectl config set-context --current --namespace=team-alpha-prod

# Cross-namespace DNS format
# <service>.<namespace>.svc.cluster.local
curl http://db-service.team-alpha-prod.svc.cluster.local:5432
```

> ✅ **Best practice:** Use `<team>-<environment>` naming (e.g., `alpha-prod`, `alpha-staging`, `alpha-dev`). Never use the `default` namespace for production workloads.

---

### Q5 — Explain etcd and what happens if it goes down

> 🎯 **Scenario:** One node of your production etcd cluster fails. What is the impact and how do you recover?

**Answer:**

etcd is a distributed key-value store using the **Raft consensus algorithm**. It stores ALL cluster state: pods, services, secrets, configmaps, RBAC, and custom resources.

**Impact by failure scenario:**

| Scenario | Impact |
|----------|--------|
| 1 of 3 nodes down | Cluster fully operational — quorum maintained (2 of 3) |
| 2 of 3 nodes down | **Read-only** — no changes possible, existing workloads keep running |
| All nodes down | **Complete outage** — control plane unresponsive |
| Data corruption | Must restore from snapshot backup |

```bash
# Check etcd health
kubectl exec -n kube-system etcd-master -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# Check etcd member list
etcdctl member list --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Take a snapshot backup
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
etcdctl snapshot status /backup/etcd-20240101.db --write-out=table

# Restore from snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-20240101.db \
  --data-dir=/var/lib/etcd-restored \
  --name=master \
  --initial-cluster=master=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380
```

> ⚠️ **Production requirement:** Always run etcd with **3 or 5 nodes** (odd number for quorum). Automate etcd snapshots to S3 every 30 minutes. Test restores regularly.

---

### Q6 — How does Kubernetes self-heal after a node failure?

> 🎯 **Scenario:** A worker node in your cluster suddenly becomes unreachable. Walk through what Kubernetes does automatically.

**Answer:**

```
T+0s     Node stops sending heartbeats to API server
T+40s    Node-monitor-grace-period expires → node marked "NotReady"
         (controlled by --node-monitor-grace-period on controller-manager)
T+5min   pod-eviction-timeout expires → pods on node get "Terminating"
         Scheduler places replacement pods on healthy nodes
T+5min+  New pods start on healthy nodes, pass readiness probes
         Service endpoints updated → traffic routed to new pods
T+...    If node recovers: kubelet reconciles and removes stale pods
```

```yaml
# PodDisruptionBudget — prevents too many pods being evicted at once
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
  namespace: production
spec:
  minAvailable: 2          # Always keep at least 2 pods running
  # OR: maxUnavailable: 1  # Allow at most 1 pod down at a time
  selector:
    matchLabels:
      app: web-app
```

```bash
# Node maintenance workflow
kubectl cordon node-1         # Mark unschedulable — no new pods land here
kubectl drain node-1 \
  --ignore-daemonsets \       # Ignore DaemonSet pods (can't be moved)
  --delete-emptydir-data \    # Delete pods using emptyDir volumes
  --grace-period=60           # Give pods 60s to terminate gracefully
# Do maintenance...
kubectl uncordon node-1       # Mark schedulable again

# Check node conditions
kubectl describe node node-1 | grep -A 20 "Conditions:"
```

> 💡 **PodDisruptionBudgets** prevent Kubernetes from evicting too many pods at once during voluntary disruptions (node drains, cluster upgrades). Always define PDBs for production workloads with replicas > 1.

---

### Q7 — What is the difference between kubectl apply, create, replace, and patch?

> 🎯 **Scenario:** A junior engineer asks when to use which command. Explain with examples.

**Answer:**

| Command | Behavior | Idempotent | Use Case |
|---------|----------|------------|----------|
| `kubectl create` | Creates; **fails** if exists | ❌ | One-time creation |
| `kubectl apply` | Creates OR updates declaratively | ✅ | GitOps, CI/CD pipelines |
| `kubectl replace` | Replaces entire resource; **fails** if missing | ❌ | Force full replacement |
| `kubectl patch` | Partially updates a resource | ✅ | Quick targeted edits |
| `kubectl edit` | Opens resource in editor | ✅ | Manual one-off changes |

```bash
# apply — idempotent, tracks last-applied-configuration annotation
kubectl apply -f deployment.yaml

# create — fails if already exists
kubectl create -f deployment.yaml

# replace — deletes and recreates (loses rollout history)
kubectl replace -f deployment.yaml
kubectl replace --force -f deployment.yaml   # Delete first, then create

# patch — merge patch
kubectl patch deployment web-app \
  -p '{"spec":{"replicas":5}}'

# patch — JSON patch (precise)
kubectl patch deployment web-app \
  --type=json \
  -p='[{"op":"replace","path":"/spec/replicas","value":5}]'

# patch — strategic merge patch (aware of K8s object structure)
kubectl patch deployment web-app \
  --type=strategic \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"web","image":"nginx:1.26"}]}}}}'

# Diff before applying
kubectl diff -f deployment.yaml
```

> ✅ **Always use `kubectl apply`** in production pipelines. It is declarative, idempotent, and enables accurate drift detection via the `kubectl.kubernetes.io/last-applied-configuration` annotation.

---

### Q8 — How do labels and selectors work, and why are they important?

> 🎯 **Scenario:** Your Service is not routing traffic to pods. The pods are running but endpoints are empty. What's happening?

**Answer:**

Labels are key-value metadata on objects. Selectors filter objects by labels. **Services route traffic only to pods whose labels match the Service selector.**

```yaml
# Deployment — pods get these labels
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  selector:
    matchLabels:
      app: web-app        # ReplicaSet selects pods with this label
      tier: frontend
  template:
    metadata:
      labels:
        app: web-app      # Pod label — MUST match selector above
        tier: frontend
        version: v2.0
        env: production
```

```yaml
# Service — routes to pods matching selector
apiVersion: v1
kind: Service
metadata:
  name: web-app-svc
spec:
  selector:
    app: web-app          # Matches pods with this label
    tier: frontend        # AND this label
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Diagnose empty endpoints
kubectl get endpoints web-app-svc
# NAME          ENDPOINTS   AGE
# web-app-svc   <none>      5m  ← PROBLEM

# Check service selector
kubectl get svc web-app-svc -o jsonpath='{.spec.selector}'
# {"app":"web-app","tier":"frontend"}

# Check what labels pods actually have
kubectl get pods --show-labels | grep web-app
# web-app-xxxx  Running  app=web-app,tier=backend  ← MISMATCH!

# Fix: update pod labels to match service selector
kubectl label pod web-app-xxxx tier=frontend --overwrite

# Other label operations
kubectl get pods -l 'app=web-app,env=production'   # Select by multiple labels
kubectl get pods -l 'version in (v1.0, v2.0)'      # Set-based selector
kubectl get pods -l 'env notin (dev, staging)'      # Exclusion selector
kubectl label pod web-app-xxxx version-             # Remove a label
```

---

## 🔵 Pods & Workloads

---

### Q9 — Your pod keeps crashing with CrashLoopBackOff. How do you debug it?

> 🎯 **Scenario:** You deployed a new application and pods show `CrashLoopBackOff`. Production is down. What is your step-by-step debugging process?

**Answer:**

```bash
# Step 1: Get overview
kubectl get pods -n production
# NAME           READY   STATUS             RESTARTS   AGE
# app-xxx-yyy    0/1     CrashLoopBackOff   8          12m

# Step 2: Describe pod — read Events section carefully
kubectl describe pod app-xxx-yyy -n production
# Look for:
#   Exit Code (137=OOM, 1=app error, 139=segfault, 143=SIGTERM)
#   Last State: reason, exitCode, finishedAt
#   Events: Failed to pull image, Failed to mount volume, etc.

# Step 3: Check CURRENT logs
kubectl logs app-xxx-yyy -n production

# Step 4: Check PREVIOUS container logs (before crash — most useful!)
kubectl logs app-xxx-yyy -n production --previous

# Step 5: Check resource pressure
kubectl top pod app-xxx-yyy -n production
kubectl describe pod app-xxx-yyy | grep -A5 "Limits\|Requests"

# Step 6: Override entrypoint to keep container alive for debugging
kubectl run debug \
  --image=your-app-image:tag \
  --restart=Never \
  --command -- sleep 3600
kubectl exec -it debug -- /bin/sh

# Step 7: Check events across namespace
kubectl get events -n production --sort-by='.lastTimestamp' | tail -20
```

**CrashLoopBackOff diagnosis table:**

| Exit Code | Cause | Fix |
|-----------|-------|-----|
| 1 | Application error at startup | Check app logs, fix code |
| 137 | OOMKilled | Increase memory limit |
| 139 | Segmentation fault | Debug application |
| 143 | SIGTERM — graceful shutdown | Check liveness probe aggressiveness |
| 126 | Command not executable | Fix file permissions |
| 127 | Command not found | Fix `command`/`args`, check image |

---

### Q10 — How do you perform a zero-downtime rolling update?

> 🎯 **Scenario:** You need to update a production web app from v1.0 to v2.0 with zero service interruption and immediate rollback capability.

**Answer:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1           # Create 1 extra pod above desired count
      maxUnavailable: 0     # Never go below desired count (zero-downtime)
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: myapp:v2.0
        # Readiness probe gates traffic — CRITICAL for zero-downtime
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        # Liveness probe restarts deadlocked pods
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          periodSeconds: 10
          failureThreshold: 3
        # Give in-flight requests time to complete
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
      # Must be >= preStop sleep time
      terminationGracePeriodSeconds: 30
```

```bash
# Trigger update
kubectl set image deployment/web-app web=myapp:v2.0 -n production

# Watch real-time rollout progress
kubectl rollout status deployment/web-app -n production

# Inspect revision history
kubectl rollout history deployment/web-app -n production

# Instant rollback to previous version
kubectl rollout undo deployment/web-app -n production

# Rollback to specific revision
kubectl rollout undo deployment/web-app --to-revision=3 -n production

# Pause mid-rollout (canary-style manual gate)
kubectl rollout pause deployment/web-app -n production
# Check metrics, error rates...
kubectl rollout resume deployment/web-app -n production
```

> ✅ **Without a readiness probe**, Kubernetes has no way to know if a new pod is actually serving traffic. New pods will receive traffic immediately upon startup — before they're ready — causing errors.

---

### Q11 — What are Init Containers and when do you use them?

> 🎯 **Scenario:** Your app container needs the database to be up and a config file generated before it starts. How do you handle this reliably?

**Answer:**

Init containers run **sequentially, to completion, before** the main container starts. If any init container fails, the pod restarts.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  initContainers:

  # 1. Wait for database to accept connections
  - name: wait-for-postgres
    image: busybox:1.35
    command:
    - sh
    - -c
    - |
      until nc -z postgres-service.db.svc.cluster.local 5432; do
        echo "Waiting for PostgreSQL..."
        sleep 3
      done
      echo "PostgreSQL is ready!"

  # 2. Run database migrations
  - name: run-migrations
    image: myapp:v2.0
    command: ["python", "manage.py", "migrate", "--no-input"]
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: url

  # 3. Fetch runtime config from S3
  - name: fetch-config
    image: amazon/aws-cli:latest
    command:
    - sh
    - -c
    - aws s3 cp s3://my-bucket/config/app.json /config/app.json
    volumeMounts:
    - name: config-vol
      mountPath: /config

  containers:
  - name: app
    image: myapp:v2.0
    volumeMounts:
    - name: config-vol
      mountPath: /app/config
      readOnly: true
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5

  volumes:
  - name: config-vol
    emptyDir: {}
```

> 💡 **Init containers can have different images** from the main container — useful for running tools (aws-cli, curl, migrate) that you don't want in your production image. They share the same network and volumes as the main container.

---

### Q12 — How do you configure liveness, readiness, and startup probes correctly?

> 🎯 **Scenario:** Pods are getting killed unnecessarily during slow startup, and traffic is being sent to pods that aren't ready yet.

**Answer:**

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: myapp:v2.0

    # STARTUP PROBE — Checked first. Disables liveness/readiness until it passes.
    # Use for apps with slow/variable startup times.
    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30   # 30 * 10s = 5 minutes max startup time
      periodSeconds: 10
      # If this fails 30 times, container is killed and restarted

    # LIVENESS PROBE — Is the app alive? Kills and restarts if failing.
    # Don't call slow external dependencies here (DB, external APIs)
    livenessProbe:
      httpGet:
        path: /health/live
        port: 8080
      periodSeconds: 10
      failureThreshold: 3       # Kill after 3 consecutive failures (30s)
      timeoutSeconds: 5
      successThreshold: 1

    # READINESS PROBE — Is the app ready to serve traffic?
    # Removes pod from Service endpoints when failing.
    readinessProbe:
      httpGet:
        path: /health/ready    # Can check DB connectivity here
        port: 8080
      periodSeconds: 5
      failureThreshold: 3      # Remove from LB after 15s of failures
      successThreshold: 2      # Re-add only after 2 consecutive passes
      initialDelaySeconds: 0   # startupProbe handles initial delay

    # TCP socket probe (databases, non-HTTP services)
    # livenessProbe:
    #   tcpSocket:
    #     port: 5432
    #   periodSeconds: 10

    # Exec probe (custom health check command)
    # livenessProbe:
    #   exec:
    #     command: ["redis-cli", "ping"]
    #   periodSeconds: 10
```

| Probe | Purpose | On Failure |
|-------|---------|------------|
| `startupProbe` | App finished initializing | Container killed and restarted |
| `livenessProbe` | App is not deadlocked/broken | Container killed and restarted |
| `readinessProbe` | App is ready to accept traffic | Removed from Service endpoints |

> ⚠️ **Common mistake:** Setting `livenessProbe` without a `startupProbe` on slow-starting apps. The liveness probe fires too soon, kills the app before it finishes starting, and you get a crash loop. Always use `startupProbe` for apps that take more than 30 seconds to start.

---

### Q13 — How do you implement a Canary deployment without a service mesh?

> 🎯 **Scenario:** You want to route 10% of production traffic to v2.0 to validate it, then gradually increase to 100%.

**Answer:**

```yaml
# Stable Deployment — 9 replicas = 90% traffic
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-stable
  namespace: production
spec:
  replicas: 9
  selector:
    matchLabels:
      app: web-app
      track: stable
  template:
    metadata:
      labels:
        app: web-app     # ← shared label
        track: stable
    spec:
      containers:
      - name: web
        image: myapp:v1.0
---
# Canary Deployment — 1 replica = 10% traffic
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-canary
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
      track: canary
  template:
    metadata:
      labels:
        app: web-app     # ← same shared label
        track: canary
    spec:
      containers:
      - name: web
        image: myapp:v2.0
---
# Service selects ALL pods with app=web-app
# Traffic split is proportional to replica count (9:1 = 90%:10%)
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app         # Matches BOTH stable and canary pods
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Step 1: Deploy canary at 10%
kubectl apply -f canary-deployment.yaml

# Step 2: Monitor error rate and latency
kubectl logs -l track=canary --tail=200 -n production
kubectl top pods -l track=canary -n production

# Step 3a: Promote — gradually increase canary, decrease stable
kubectl scale deployment/web-app-canary --replicas=3   # 30%
kubectl scale deployment/web-app-stable --replicas=7   # 70%
# ... eventually
kubectl scale deployment/web-app-canary --replicas=10  # 100%
kubectl scale deployment/web-app-stable --replicas=0

# Step 3b: Rollback if issues found
kubectl scale deployment/web-app-canary --replicas=0
kubectl delete deployment/web-app-canary
```

> 💡 **For header/cookie-based traffic splitting**, use NGINX Ingress canary annotations or a service mesh (Istio, Linkerd). The replica-ratio approach splits randomly which is fine for simple cases.

---

### Q14 — How do DaemonSets work and what are common use cases?

> 🎯 **Scenario:** Your ops team needs a Datadog agent and a log collector running on every node, including new nodes that join automatically.

**Answer:**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: datadog-agent
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: datadog-agent
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: datadog-agent
    spec:
      # Required to run on control-plane nodes too
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node.kubernetes.io/not-ready
        operator: Exists
        effect: NoExecute

      # Only run on Linux nodes
      nodeSelector:
        kubernetes.io/os: linux

      containers:
      - name: agent
        image: datadog/agent:latest
        env:
        - name: DD_API_KEY
          valueFrom:
            secretKeyRef:
              name: datadog-secret
              key: api-key
        - name: DD_KUBERNETES_KUBELET_HOST
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        volumeMounts:
        - name: dockersocket
          mountPath: /var/run/docker.sock
        - name: procdir
          mountPath: /host/proc
          readOnly: true
        - name: cgroups
          mountPath: /host/sys/fs/cgroup
          readOnly: true

      volumes:
      - name: dockersocket
        hostPath:
          path: /var/run/docker.sock
      - name: procdir
        hostPath:
          path: /proc
      - name: cgroups
        hostPath:
          path: /sys/fs/cgroup
```

> 💡 **DaemonSet common use cases:** Log collectors (Fluentd, Promtail), metrics agents (Datadog, node-exporter), network plugins (CNI), storage daemons (Ceph), security scanners, GPU device plugins.

---

### Q15 — How do you run batch Jobs and CronJobs with retry and parallelism?

> 🎯 **Scenario:** You need to run a data processing job that processes 1000 items in parallel chunks, retrying on failure, and a nightly cleanup job.

**Answer:**

```yaml
# Parallel batch Job — process items in parallel
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
  namespace: production
spec:
  completions: 10          # Total tasks to complete
  parallelism: 3           # Run 3 pods simultaneously
  backoffLimit: 4          # Retry up to 4 times per pod
  activeDeadlineSeconds: 3600   # Kill job after 1 hour
  ttlSecondsAfterFinished: 86400  # Auto-delete after 24h
  template:
    spec:
      restartPolicy: OnFailure   # OnFailure or Never (not Always)
      containers:
      - name: processor
        image: data-processor:v1.0
        command:
        - python
        - process.py
        - --chunk-index=$(JOB_COMPLETION_INDEX)
        env:
        - name: JOB_COMPLETION_INDEX
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
```

```yaml
# CronJob — nightly database cleanup at 2 AM UTC
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-cleanup
  namespace: production
spec:
  schedule: "0 2 * * *"          # Cron: minute hour dom month dow
  timeZone: "UTC"                 # K8s 1.27+
  concurrencyPolicy: Forbid       # Skip if previous job still running
  successfulJobsHistoryLimit: 3   # Keep last 3 successful job records
  failedJobsHistoryLimit: 3       # Keep last 3 failed job records
  startingDeadlineSeconds: 300    # If missed window, only retry within 5 min
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 1800
      template:
        spec:
          restartPolicy: OnFailure
          serviceAccountName: cleanup-sa
          containers:
          - name: cleanup
            image: myapp:v2.0
            command: ["python", "cleanup.py", "--days=30"]
            resources:
              requests:
                cpu: 100m
                memory: 256Mi
```

```bash
# Manually trigger a CronJob immediately
kubectl create job --from=cronjob/nightly-cleanup manual-run-$(date +%s)

# Monitor Job progress
kubectl get jobs -n production
kubectl describe job data-processor -n production
kubectl logs -l job-name=data-processor --prefix=true
```

---

### Q16 — How do you use sidecar containers and what are common patterns?

> 🎯 **Scenario:** You need to add request logging, TLS termination, and secrets injection to your app without modifying the application code.

**Answer:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecars
spec:
  # Shared volume for inter-container communication
  volumes:
  - name: shared-logs
    emptyDir: {}
  - name: nginx-config
    configMap:
      name: nginx-config

  containers:
  # Main application (HTTP on localhost:8080)
  - name: app
    image: myapp:v2.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app

  # Sidecar 1: NGINX reverse proxy / TLS terminator
  - name: nginx-proxy
    image: nginx:1.25-alpine
    ports:
    - containerPort: 443
    volumeMounts:
    - name: nginx-config
      mountPath: /etc/nginx/conf.d
    - name: tls-certs
      mountPath: /etc/nginx/certs
      readOnly: true

  # Sidecar 2: Log shipper
  - name: log-shipper
    image: fluent/fluent-bit:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app
      readOnly: true

  # Sidecar 3: Vault Agent (secrets injection)
  - name: vault-agent
    image: hashicorp/vault:latest
    command: ["vault", "agent", "-config=/vault/config/agent.hcl"]
    volumeMounts:
    - name: vault-config
      mountPath: /vault/config
    - name: secrets-vol
      mountPath: /vault/secrets

  initContainers:
  # Init: Set up Vault auth token before main containers start
  - name: vault-init
    image: hashicorp/vault:latest
    command: ["sh", "-c", "vault login -method=kubernetes role=my-app"]

  volumes:
  - name: tls-certs
    secret:
      secretName: app-tls-cert
  - name: vault-config
    configMap:
      name: vault-agent-config
  - name: secrets-vol
    emptyDir:
      medium: Memory  # Store secrets in RAM, not disk
```

> 💡 **Istio service mesh** uses this pattern at scale — automatically injecting an Envoy proxy sidecar into every pod for mTLS, traffic management, and observability without changing application code.

---

### Q17 — How do you handle graceful shutdown of pods in Kubernetes?

> 🎯 **Scenario:** Your pods are being killed abruptly during deployments, causing dropped HTTP connections and failed requests.

**Answer:**

```
Kubernetes pod termination sequence:

1. Pod moves to Terminating state
2. Pod removed from Service endpoints (stops receiving new traffic)
   ← BUT there's a propagation delay! iptables/IPVS rules take ~1-2s to update
3. preStop hook executes (if defined)
4. SIGTERM sent to container process (PID 1)
5. terminationGracePeriodSeconds countdown begins
6. If process still running after grace period → SIGKILL
```

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      # Must be >= preStop sleep + app shutdown time
      terminationGracePeriodSeconds: 60

      containers:
      - name: app
        image: myapp:v2.0

        lifecycle:
          preStop:
            exec:
              # Sleep bridges the gap between endpoint removal
              # and actual traffic drain (~5-15s is typical)
              command: ["/bin/sh", "-c", "sleep 15"]

          # OR: call a graceful shutdown endpoint
          # preStop:
          #   httpGet:
          #     path: /shutdown
          #     port: 8080

        # Your app should handle SIGTERM gracefully:
        # - Stop accepting new connections
        # - Finish processing in-flight requests
        # - Close DB connections cleanly
        # - Exit with code 0
```

```python
# Example: Python app handling SIGTERM
import signal, sys

def handle_sigterm(signum, frame):
    print("Received SIGTERM — finishing in-flight requests...")
    server.shutdown()  # Stop accepting new requests
    # Wait for in-flight requests to complete
    sys.exit(0)

signal.signal(signal.SIGTERM, handle_sigterm)
```

> ⚠️ **Without preStop sleep**, new traffic can still arrive at the pod for 1-2 seconds after endpoint removal starts (due to iptables propagation lag). The preStop sleep ensures the pod stays alive long enough to drain.

---

### Q18 — How do you manage multi-container pods with shared state?

> 🎯 **Scenario:** You have a web server and a metrics exporter that need to share a socket file and log directory.

**Answer:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-exporter
spec:
  volumes:
  # Shared socket directory between containers
  - name: socket-dir
    emptyDir: {}
  # Shared log directory
  - name: log-dir
    emptyDir: {}

  containers:
  # Main app: writes to /tmp/app.sock and /var/log/app/
  - name: web-server
    image: myapp:v2.0
    volumeMounts:
    - name: socket-dir
      mountPath: /tmp
    - name: log-dir
      mountPath: /var/log/app

  # Exporter: reads from the same socket/logs
  - name: metrics-exporter
    image: prom/statsd-exporter:latest
    args:
    - --statsd.listen-unixgram=/tmp/app.sock
    volumeMounts:
    - name: socket-dir
      mountPath: /tmp   # Same path — shares the socket file
    ports:
    - containerPort: 9102
      name: metrics
    readinessProbe:
      httpGet:
        path: /metrics
        port: 9102

  # Log rotator sidecar
  - name: log-rotator
    image: busybox:1.35
    command: ["/bin/sh", "-c"]
    args:
    - |
      while true; do
        find /var/log/app -name "*.log" -mtime +7 -delete
        sleep 3600
      done
    volumeMounts:
    - name: log-dir
      mountPath: /var/log/app
```

> 💡 **Key multi-container rules:** All containers in a pod share the same network namespace (same IP, same localhost), same IPC namespace, and can share volumes. But they have separate filesystem namespaces (different PID 1s).

---

## 🟣 Services & Networking

---

### Q19 — Explain the different Service types and when to use each

> 🎯 **Scenario:** A developer asks when to use ClusterIP vs NodePort vs LoadBalancer.

**Answer:**

```yaml
# ClusterIP (default) — cluster-internal only
apiVersion: v1
kind: Service
metadata:
  name: backend-api
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
  # DNS: backend-api.namespace.svc.cluster.local:8080
```

```yaml
# NodePort — expose on every node's IP:port
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080    # Valid range: 30000-32767
  # Access: <any-node-ip>:30080
```

```yaml
# LoadBalancer — provisions cloud load balancer (AWS NLB/ALB)
apiVersion: v1
kind: Service
metadata:
  name: web-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 443
    targetPort: 8080
    protocol: TCP
  # AWS provisions an NLB; service gets external DNS/IP
```

```yaml
# Headless — no ClusterIP; DNS returns individual pod IPs
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - port: 5432
  # DNS: postgres-0.postgres-headless.ns.svc.cluster.local → pod-0 IP
  #       postgres-1.postgres-headless.ns.svc.cluster.local → pod-1 IP
```

| Type | Scope | Best For |
|------|-------|---------|
| `ClusterIP` | Inside cluster | Microservice communication |
| `NodePort` | Node IPs | Dev, on-prem without cloud LB |
| `LoadBalancer` | Internet | Production external services |
| `Headless` | Direct pod DNS | StatefulSets, Cassandra, Kafka |
| `ExternalName` | DNS alias | Route to external service by name |

---

### Q20 — How does Ingress work and how do you configure HTTPS with cert-manager?

> 🎯 **Scenario:** You have 5 microservices and want them at `api.example.com/users`, `api.example.com/orders`, etc. with auto-renewed HTTPS certificates.

**Answer:**

```bash
# Install NGINX Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.metrics.enabled=true

# Install cert-manager
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set installCRDs=true
```

```yaml
# ClusterIssuer — Let's Encrypt certificate authority
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        ingress:
          class: nginx
```

```yaml
# Ingress — path-based routing + automatic TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.example.com
    secretName: api-example-com-tls   # cert-manager creates this
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: users-service
            port:
              number: 8080
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: orders-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 3000
```

```bash
# Monitor certificate issuance
kubectl get certificate -n production
kubectl describe certificate api-example-com-tls -n production
kubectl get certificaterequest -n production
```

---

### Q21 — How does Kubernetes DNS work and how do you debug DNS failures?

> 🎯 **Scenario:** Pods in your cluster can't resolve service names. Requests fail with "connection refused" or DNS lookup failures.

**Answer:**

Kubernetes runs **CoreDNS** as a cluster DNS server. Every pod's `/etc/resolv.conf` points to the CoreDNS ClusterIP.

```
DNS resolution hierarchy:
  my-service                          → searches: default.svc.cluster.local
  my-service.other-ns                 → searches: svc.cluster.local
  my-service.other-ns.svc            → searches: cluster.local
  my-service.other-ns.svc.cluster.local → full FQDN (resolved directly)
```

```bash
# Launch a debug pod with DNS tools
kubectl run dns-debug \
  --image=registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3 \
  --restart=Never -it -- bash

# Inside the pod:
# Check resolv.conf
cat /etc/resolv.conf
# nameserver 10.96.0.10         ← CoreDNS ClusterIP
# search default.svc.cluster.local svc.cluster.local cluster.local
# options ndots:5

# Test service DNS
nslookup kubernetes.default.svc.cluster.local
nslookup my-service.production.svc.cluster.local

# Test external DNS
nslookup google.com
```

```bash
# Debug CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --prefix

# Check CoreDNS config
kubectl get configmap coredns -n kube-system -o yaml

# Common CoreDNS issues:
# 1. CoreDNS pods OOMKilled → increase memory limits
# 2. ndots:5 causing slow resolution → external lookups try 6 DNS queries
#    Fix: set ndots:1 in pod dnsConfig for external-heavy workloads
# 3. DNS cache poisoning → use separate upstream resolvers

# Override DNS per pod
apiVersion: v1
kind: Pod
spec:
  dnsConfig:
    options:
    - name: ndots
      value: "1"    # Reduces unnecessary search path queries
    nameservers:
    - 8.8.8.8       # Additional custom nameservers
```

---

### Q22 — How do you implement zero-trust networking with NetworkPolicies?

> 🎯 **Scenario:** Security audit requires that only frontend can reach the API, only API can reach the database, and nothing else.

**Answer:**

```yaml
# Step 1: Default-deny ALL ingress AND egress in the namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}     # Applies to ALL pods
  policyTypes:
  - Ingress
  - Egress
```

```yaml
# Step 2: Allow frontend → API (port 8080 only)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api-server
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

```yaml
# Step 3: Allow API → PostgreSQL (port 5432 only)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-postgres
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-server
    ports:
    - protocol: TCP
      port: 5432
```

```yaml
# Step 4: Allow all pods to query CoreDNS (essential!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

```yaml
# Step 5: Allow API egress to external services
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-external-egress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api-server
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: TCP
      port: 443    # HTTPS to external APIs
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```

> ⚠️ **NetworkPolicies require a CNI plugin that supports them** — Calico, Cilium, or Weave Net. The default `flannel` CNI does **not** enforce NetworkPolicies. Policies are additive: multiple policies combine with OR logic.

---

### Q23 — How do you expose a service to multiple clusters or use ExternalName?

> 🎯 **Scenario:** Your app in Kubernetes needs to call an RDS database and a third-party payment API by logical names, not hardcoded endpoints.

**Answer:**

```yaml
# ExternalName — DNS alias to external service
# App uses: postgres-prod.production.svc.cluster.local
# Resolves to: prod-db.abc123.us-east-1.rds.amazonaws.com
apiVersion: v1
kind: Service
metadata:
  name: postgres-prod
  namespace: production
spec:
  type: ExternalName
  externalName: prod-db.abc123.us-east-1.rds.amazonaws.com
  # No selector — no pods. Pure DNS CNAME alias.
```

```yaml
# Endpoints + Service — point to external IP directly
# Use when external service doesn't have a hostname
apiVersion: v1
kind: Service
metadata:
  name: legacy-payment-api
  namespace: production
spec:
  ports:
  - port: 443
    targetPort: 443
---
apiVersion: v1
kind: Endpoints
metadata:
  name: legacy-payment-api   # Must match Service name
subsets:
- addresses:
  - ip: 10.20.30.40          # External server IP
  - ip: 10.20.30.41
  ports:
  - port: 443
```

```yaml
# ServiceEntry (Istio) — fine-grained external service control
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: stripe-api
spec:
  hosts:
  - api.stripe.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
  location: MESH_EXTERNAL
```

---

### Q24 — How does kube-proxy work and what modes does it support?

> 🎯 **Scenario:** Your cluster has high traffic and you suspect kube-proxy iptables rules are becoming a performance bottleneck. What are your options?

**Answer:**

kube-proxy runs on every node and maintains network rules that redirect traffic from Service ClusterIPs to pod IPs.

**Three modes:**

| Mode | How It Works | Performance | Notes |
|------|-------------|-------------|-------|
| `userspace` | Proxy in userspace (old) | Slowest | Deprecated |
| `iptables` | Kernel netfilter rules | Good | Default; O(n) rules for n services |
| `ipvs` | Linux Virtual Server | Best | O(1) lookups; for 1000+ services |

```bash
# Check current kube-proxy mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode

# Switch to IPVS mode
kubectl edit configmap kube-proxy -n kube-system
# Set: mode: "ipvs"

# Restart kube-proxy pods to apply
kubectl rollout restart daemonset/kube-proxy -n kube-system

# Verify IPVS rules
ipvsadm -Ln   # Run on a node

# Cilium — can replace kube-proxy entirely (eBPF)
# No iptables, no kube-proxy, direct eBPF kernel datapath
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=true \
  --set k8sServiceHost=<api-server-ip>
```

> 💡 **For clusters with > 500 services**, switch to IPVS mode or use Cilium's eBPF kube-proxy replacement. iptables rule evaluation is O(n) — performance degrades linearly with service count.

---

### Q25 — How do you implement mutual TLS (mTLS) between services?

> 🎯 **Scenario:** Your compliance team requires all service-to-service communication to be encrypted and mutually authenticated.

**Answer:**

```bash
# Option 1: Istio service mesh — automatic mTLS
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm install istio-base istio/base -n istio-system --create-namespace
helm install istiod istio/istiod -n istio-system

# Label namespace for automatic sidecar injection
kubectl label namespace production istio-injection=enabled

# Enforce strict mTLS across namespace
```

```yaml
# Istio PeerAuthentication — enforce mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: enforce-mtls
  namespace: production
spec:
  mtls:
    mode: STRICT    # STRICT: only mTLS, no plaintext
    # PERMISSIVE: both mTLS and plaintext (migration mode)
    # DISABLE: no mTLS
```

```yaml
# Istio AuthorizationPolicy — service-level access control
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-server-policy
  namespace: production
spec:
  selector:
    matchLabels:
      app: api-server
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        # Only allow calls from frontend service account
        - "cluster.local/ns/production/sa/frontend-sa"
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
```

```bash
# Verify mTLS is working
istioctl authn tls-check api-server-pod-xxx.production

# View certificate details
istioctl proxy-config secret api-server-pod-xxx.production
```

---

### Q26 — How do you configure Service topology and traffic routing policies?

> 🎯 **Scenario:** You have a multi-region cluster and want traffic to prefer pods in the same zone to reduce latency and egress costs.

**Answer:**

```yaml
# Topology Aware Routing (K8s 1.27+)
apiVersion: v1
kind: Service
metadata:
  name: web-app
  annotations:
    service.kubernetes.io/topology-mode: "Auto"
    # "Auto" — route to same-zone endpoints when possible
    # Fallback to cross-zone if no local endpoints are healthy
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
```

```yaml
# TrafficPolicy in Istio (more fine-grained control)
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: web-app
  namespace: production
spec:
  host: web-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: UPGRADE
        idleTimeout: 10s
    loadBalancer:
      localityLbSetting:
        enabled: true
        distribute:
        - from: "us-east-1/us-east-1a/*"
          to:
            "us-east-1/us-east-1a/*": 80   # 80% same-AZ
            "us-east-1/us-east-1b/*": 20   # 20% different AZ
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
```


## 🟠 Storage & Volumes

---

### Q27 — Explain PVs, PVCs, and StorageClasses with dynamic provisioning

> 🎯 **Scenario:** A developer needs persistent storage for their PostgreSQL pod that survives pod restarts and node failures.

**Answer:**

```
Storage abstraction layers:

StorageClass  → defines HOW storage is created (AWS EBS gp3, NFS, local SSD)
      ↓ (dynamic provisioning — auto-creates PV when PVC is created)
PersistentVolume (PV) → the actual storage resource
      ↓ (binding — K8s matches PVC to PV)
PersistentVolumeClaim (PVC) → app's request for storage
      ↓ (mount)
Pod → uses PVC as a volume
```

```yaml
# StorageClass — tells K8s how to provision storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
reclaimPolicy: Retain                    # Keep EBS volume when PVC deleted
allowVolumeExpansion: true               # Allow PVC resize
volumeBindingMode: WaitForFirstConsumer  # Create volume in same AZ as pod
```

```yaml
# PVC — developer requests storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
  namespace: production
spec:
  accessModes:
  - ReadWriteOnce      # RWO: one node read-write
  # ReadWriteMany      # RWX: multiple nodes read-write (NFS, EFS)
  # ReadOnlyMany       # ROX: multiple nodes read-only
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 100Gi
```

```yaml
# Pod using the PVC
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: postgres
    image: postgres:15
    volumeMounts:
    - name: data
      mountPath: /var/lib/postgresql/data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: postgres-data
```

```bash
# Check PV/PVC binding status
kubectl get pv,pvc -n production

# STATUS lifecycle:
# PVC: Pending → Bound → Lost (if PV deleted)
# PV:  Available → Bound → Released → Reclaimed/Retained
```

---

### Q28 — How do you expand a PVC without downtime?

> 🎯 **Scenario:** Your PostgreSQL PVC is 80% full and you need to expand from 100Gi to 200Gi without stopping the database.

**Answer:**

```bash
# Step 1: Verify StorageClass supports expansion
kubectl get storageclass fast-ssd -o jsonpath='{.allowVolumeExpansion}'
# Must return: true

# Step 2: Expand the PVC — just edit the request size
kubectl patch pvc postgres-data -n production \
  -p '{"spec":{"resources":{"requests":{"storage":"200Gi"}}}}'

# Step 3: Monitor expansion
kubectl get pvc postgres-data -n production -w
# Conditions will show:
# FileSystemResizePending → True (volume resized, waiting for fs resize)
# Then PVC capacity updates to 200Gi

# Step 4: For most CSI drivers — online expansion, no restart needed
# Verify inside the pod
kubectl exec -it postgres-pod -n production -- df -h /var/lib/postgresql/data
```

```bash
# If driver requires pod restart (older drivers):
# StatefulSet will automatically recreate the pod after deletion
kubectl delete pod postgres-0 -n production
# kubelet will resize filesystem during pod startup
```

> ⚠️ **Volume expansion is one-directional** — you can only increase, never decrease PVC size. The underlying cloud volume may charge for the new size immediately.

---

### Q29 — How do you back up and restore stateful workloads?

> 🎯 **Scenario:** You need a robust backup strategy for databases running in your cluster.

**Answer:**

```yaml
# Option A: Application-level backup (recommended for databases)
# PostgreSQL nightly backup to S3
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: production
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: backup-sa
          restartPolicy: OnFailure
          containers:
          - name: pg-backup
            image: postgres:15-alpine
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: password
            - name: BACKUP_BUCKET
              value: s3://my-company-backups
            command:
            - /bin/sh
            - -c
            - |
              DATE=$(date +%Y%m%d_%H%M%S)
              pg_dump -h postgres-service -U postgres -d mydb \
                | gzip \
                | aws s3 cp - ${BACKUP_BUCKET}/postgres/${DATE}.sql.gz
              echo "Backup completed: ${DATE}"
```

```bash
# Option B: Velero — Kubernetes-native backup (resources + volumes)

# Install Velero with AWS plugin
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.7.0 \
  --bucket my-velero-backups \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1 \
  --use-restic   # For PVC backup

# Create immediate backup
velero backup create prod-backup-$(date +%Y%m%d) \
  --include-namespaces production \
  --include-resources pods,deployments,statefulsets,pvc,secrets,configmaps

# Schedule automated backups
velero schedule create daily-prod \
  --schedule="0 3 * * *" \
  --include-namespaces production \
  --ttl 720h    # Keep for 30 days

# List backups
velero backup get

# Restore from backup
velero restore create --from-backup prod-backup-20240101

# Restore specific namespace only
velero restore create \
  --from-backup prod-backup-20240101 \
  --include-namespaces production
```

---

### Q30 — What are CSI drivers and how do you use them?

> 🎯 **Scenario:** Your team wants to use AWS EFS for shared storage (ReadWriteMany) across multiple pods.

**Answer:**

CSI (Container Storage Interface) is the standard API for storage plugins in Kubernetes. Cloud providers and storage vendors implement CSI drivers.

```bash
# Install AWS EFS CSI driver
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
helm install aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
  --namespace kube-system \
  --set controller.serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::123456789:role/efs-csi-role
```

```yaml
# StorageClass for EFS (ReadWriteMany)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-0abc123def456     # Your EFS filesystem ID
  directoryPerms: "700"
  gidRangeStart: "1000"
  gidRangeEnd: "2000"
```

```yaml
# PVC using EFS — ReadWriteMany (multiple pods can write simultaneously)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage
  namespace: production
spec:
  accessModes:
  - ReadWriteMany      # Multiple pods across multiple nodes can write
  storageClassName: efs-sc
  resources:
    requests:
      storage: 100Gi
```

```yaml
# Multiple pods sharing the same EFS PVC
apiVersion: apps/v1
kind: Deployment
metadata:
  name: media-processor
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: processor
        image: media-processor:v1
        volumeMounts:
        - name: shared-media
          mountPath: /data/media
      volumes:
      - name: shared-media
        persistentVolumeClaim:
          claimName: shared-storage  # All 5 replicas share this PVC
```

---

### Q31 — How do you handle storage for a StatefulSet across multiple AZs?

> 🎯 **Scenario:** You're running a 3-node Elasticsearch cluster across 3 AZs. How do you ensure each node gets storage in the correct AZ?

**Answer:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: search
spec:
  serviceName: elasticsearch-headless
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      # Spread pods evenly across AZs
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: elasticsearch

      containers:
      - name: elasticsearch
        image: elasticsearch:8.11.0
        env:
        - name: node.name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: cluster.initial_master_nodes
          value: "elasticsearch-0,elasticsearch-1,elasticsearch-2"
        - name: ES_JAVA_OPTS
          value: "-Xms2g -Xmx2g"
        resources:
          requests:
            cpu: "1"
            memory: 4Gi
          limits:
            cpu: "2"
            memory: 4Gi
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data

  # Each pod gets its own PVC — provisioned in the same AZ as the pod
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: gp3-wait-for-consumer  # WaitForFirstConsumer binding!
      resources:
        requests:
          storage: 500Gi
```

```yaml
# Critical: WaitForFirstConsumer binding mode
# Without this, PV is created in a random AZ — pod scheduling fails!
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-wait-for-consumer
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer   # ← Waits for pod to schedule, then creates EBS in same AZ
allowVolumeExpansion: true
reclaimPolicy: Retain
```

---

### Q32 — How do you migrate data from one PVC to another?

> 🎯 **Scenario:** You need to migrate your PostgreSQL data from a gp2 EBS volume to gp3 without any data loss or extended downtime.

**Answer:**

```bash
# Method 1: Snapshot-based migration (preferred for large datasets)

# Step 1: Create a VolumeSnapshot of the source PVC
kubectl apply -f - <<EOF
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: postgres-data-snapshot
  namespace: production
spec:
  volumeSnapshotClassName: csi-aws-vsc
  source:
    persistentVolumeClaimName: postgres-data-old
EOF

# Step 2: Wait for snapshot to be ready
kubectl get volumesnapshot postgres-data-snapshot -n production -w

# Step 3: Create new PVC from snapshot
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data-new
  namespace: production
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd-gp3
  resources:
    requests:
      storage: 100Gi
  dataSource:
    name: postgres-data-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
EOF
```

```bash
# Method 2: rsync migration (for smaller datasets or when snapshots unavailable)

# Step 1: Deploy a migration pod that mounts both PVCs
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pvc-migrator
  namespace: production
spec:
  restartPolicy: Never
  containers:
  - name: migrator
    image: alpine:latest
    command:
    - sh
    - -c
    - |
      apk add --no-cache rsync
      rsync -avz --progress /source/ /destination/
      echo "Migration complete!"
    volumeMounts:
    - name: source
      mountPath: /source
      readOnly: true
    - name: destination
      mountPath: /destination
  volumes:
  - name: source
    persistentVolumeClaim:
      claimName: postgres-data-old
  - name: destination
    persistentVolumeClaim:
      claimName: postgres-data-new
EOF

kubectl logs -f pvc-migrator -n production
```

---

## 🔴 Configuration & Secrets

---

### Q33 — Difference between ConfigMaps and Secrets, and how to use them

> 🎯 **Scenario:** Your app needs both non-sensitive configuration (feature flags, timeouts) and sensitive data (DB credentials, API keys). How do you manage both?

**Answer:**

```yaml
# ConfigMap — non-sensitive configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"
  CACHE_TTL: "300"
  FEATURE_NEW_UI: "true"

  # Multi-line config file
  app.yaml: |
    server:
      port: 8080
      timeout: 30s
    database:
      pool_size: 20
      max_idle: 5

  nginx.conf: |
    upstream backend { server 127.0.0.1:8080; }
    server {
      listen 80;
      location / { proxy_pass http://backend; }
    }
```

```yaml
# Secret — sensitive data (base64-encoded, but NOT encrypted by default)
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: production
type: Opaque
stringData:           # stringData auto-base64-encodes
  DB_PASSWORD: "super-secret-password"
  JWT_SIGNING_KEY: "very-long-random-secret-key"
  tls.crt: |
    -----BEGIN CERTIFICATE-----
    MIID...
    -----END CERTIFICATE-----
```

```yaml
# Use in a Deployment
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:v2.0

        # Inject all ConfigMap keys as env vars
        envFrom:
        - configMapRef:
            name: app-config
        # Inject all Secret keys as env vars
        - secretRef:
            name: app-secrets

        # Or inject individual keys
        env:
        - name: DB_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD

        # Mount ConfigMap as config files
        volumeMounts:
        - name: app-config-vol
          mountPath: /etc/app
          readOnly: true
        - name: secrets-vol
          mountPath: /etc/secrets
          readOnly: true

      volumes:
      - name: app-config-vol
        configMap:
          name: app-config
          items:                     # Mount only specific keys
          - key: app.yaml
            path: config.yaml       # Filename in container
      - name: secrets-vol
        secret:
          secretName: app-secrets
          defaultMode: 0400          # Owner read-only
```

> ⚠️ **Kubernetes Secrets are only base64-encoded, NOT encrypted at rest!** Anyone with RBAC access to read secrets can decode them. For production: enable EncryptionConfiguration on etcd AND use External Secrets Operator with AWS Secrets Manager, GCP Secret Manager, or HashiCorp Vault.

---

### Q34 — How do you use External Secrets Operator with AWS Secrets Manager?

> 🎯 **Scenario:** Your security policy forbids storing secrets in Kubernetes. All secrets must live in AWS Secrets Manager and be synced automatically.

**Answer:**

```bash
# Install External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
  --namespace external-secrets \
  --create-namespace \
  --set installCRDs=true
```

```yaml
# SecretStore — configure AWS Secrets Manager connection
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secretsmanager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa   # Uses IRSA — no static keys!
```

```yaml
# ExternalSecret — sync a specific secret from AWS → K8s
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  refreshInterval: 1h         # Re-sync from AWS every hour
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: database-credentials  # K8s Secret that gets created
    creationPolicy: Owner        # ESO owns this secret
    template:
      type: Opaque
      engineVersion: v2
  data:
  - secretKey: DB_PASSWORD       # Key in K8s Secret
    remoteRef:
      key: prod/myapp/database   # AWS SM secret name
      property: password         # JSON field within secret
  - secretKey: DB_USERNAME
    remoteRef:
      key: prod/myapp/database
      property: username
  # Bulk import all fields from a JSON secret
  dataFrom:
  - extract:
      key: prod/myapp/all-secrets
```

```bash
# Verify sync status
kubectl get externalsecret database-credentials -n production
# STATUS column shows: SecretSynced or error

kubectl describe externalsecret database-credentials -n production
```

---

### Q35 — How do you rotate secrets without downtime?

> 🎯 **Scenario:** Your database password is compromised. You need to rotate it immediately with zero application downtime.

**Answer:**

```bash
# Rotation strategy for env-var injected secrets (requires pod restart):

# Step 1: Update secret in database first (allow both old and new password)
# Step 2: Update the K8s secret
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=new-secure-password-v2 \
  --dry-run=client -o yaml | kubectl apply -f -

# Step 3: Rolling restart — zero-downtime with 4 replicas and maxUnavailable=0
kubectl rollout restart deployment/api-server -n production
kubectl rollout status deployment/api-server -n production

# Step 4: Remove old password from database after all pods are updated
# Step 5: Verify no old-password connections in DB
```

```bash
# For volume-mounted secrets — automatic file update (no restart needed)
# K8s automatically updates mounted secret files within ~1 minute
# App must watch for file changes and reload

# Check file was updated in pod
kubectl exec -it api-server-xxx -- cat /etc/secrets/DB_PASSWORD

# Force immediate update (before kubelet sync)
# Annotate pod to trigger kubelet reconcile
kubectl annotate pod api-server-xxx rotation-timestamp=$(date +%s)
```

```yaml
# Best practice: version your secrets
apiVersion: v1
kind: Secret
metadata:
  name: db-secret-v2          # Include version in name
  namespace: production
  annotations:
    rotation-date: "2024-01-15"
    rotated-by: "security-team"
```

---

### Q36 — How do you manage environment-specific configs with Kustomize?

> 🎯 **Scenario:** Your base application YAML works for dev, but production needs 5 replicas, different resource limits, a different image tag, and production database URLs.

**Answer:**

```
Directory structure:
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── dev-patch.yaml
    ├── staging/
    │   └── kustomization.yaml
    └── production/
        ├── kustomization.yaml
        ├── prod-patch.yaml
        └── prod-configmap.yaml
```

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
commonLabels:
  app: web-api
  managed-by: kustomize
```

```yaml
# base/deployment.yaml (minimal baseline)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: api
        image: myapp:latest
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
```

```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
namespace: production
images:
- name: myapp
  newTag: "v2.1.0-prod"    # Override image tag
patches:
- path: prod-patch.yaml
configMapGenerator:
- name: app-config
  literals:
  - DB_URL=postgresql://prod-db.internal:5432/myapp
  - LOG_LEVEL=warn
  - REPLICAS=5
```

```yaml
# overlays/production/prod-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: api
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: "2"
            memory: 1Gi
```

```bash
# Preview production output
kubectl kustomize overlays/production/

# Apply production config
kubectl apply -k overlays/production/

# Apply dev config
kubectl apply -k overlays/dev/
```

---

### Q37 — How do you enable encryption at rest for Kubernetes Secrets?

> 🎯 **Scenario:** A security audit flags that Kubernetes Secrets are stored in plaintext in etcd. How do you fix this?

**Answer:**

```yaml
# /etc/kubernetes/enc/encryption-config.yaml
# Create this on the control plane node

apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  - configmaps         # Optionally encrypt ConfigMaps too
  providers:
  - aescbc:            # AES-CBC encryption
      keys:
      - name: key1
        secret: <base64-encoded-32-byte-key>  # openssl rand -base64 32
  - identity: {}       # Fallback: allows reading unencrypted secrets
                       # Remove this after re-encrypting all existing secrets
```

```bash
# Generate a 32-byte key
head -c 32 /dev/urandom | base64

# Add to kube-apiserver flags (in /etc/kubernetes/manifests/kube-apiserver.yaml)
# --encryption-provider-config=/etc/kubernetes/enc/encryption-config.yaml

# Mount the config file in the apiserver static pod
# volumeMounts:
# - name: enc
#   mountPath: /etc/kubernetes/enc
#   readOnly: true
# volumes:
# - name: enc
#   hostPath:
#     path: /etc/kubernetes/enc
#     type: DirectoryOrCreate

# After apiserver restarts, re-encrypt all existing secrets
kubectl get secrets -A -o json | kubectl replace -f -

# Verify a secret is encrypted in etcd
ETCDCTL_API=3 etcdctl get /registry/secrets/default/my-secret \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  | hexdump -C | head
# Should show: k8s:enc:aescbc:v1:key1:... (encrypted, not plaintext)
```

---

### Q38 — How do you use Sealed Secrets for GitOps-safe secret management?

> 🎯 **Scenario:** Your team uses GitOps (all config in Git), but Kubernetes Secrets can't be committed to Git as they're only base64-encoded. How do you solve this?

**Answer:**

Sealed Secrets encrypts K8s Secrets with a public key. The encrypted SealedSecret is safe to commit to Git. Only the in-cluster controller can decrypt it.

```bash
# Install Sealed Secrets controller
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets \
  --namespace kube-system

# Install kubeseal CLI
brew install kubeseal

# Fetch the public key (for offline use)
kubeseal --fetch-cert > pub-cert.pem
```

```bash
# Create a SealedSecret from a regular Secret
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=my-super-secret \
  --dry-run=client -o yaml \
  | kubeseal \
  --cert pub-cert.pem \
  --format yaml > sealed-db-secret.yaml

# Now commit sealed-db-secret.yaml to Git — it's safe!
git add sealed-db-secret.yaml
git commit -m "Add sealed DB secret"

# Apply to cluster — controller decrypts and creates regular Secret
kubectl apply -f sealed-db-secret.yaml

# Verify Secret was created
kubectl get secret db-secret
```

```yaml
# sealed-db-secret.yaml (safe to commit to Git)
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: db-secret
  namespace: production
spec:
  encryptedData:
    DB_PASSWORD: AgB7x9... (long base64 encrypted string)
  template:
    metadata:
      name: db-secret
      namespace: production
    type: Opaque
```

---

## 🟡 Scheduling & Resource Management

---

### Q39 — How do resource requests and limits affect scheduling and stability?

> 🎯 **Scenario:** Your cluster nodes are running out of memory and pods are getting OOMKilled randomly. How do you properly configure resources?

**Answer:**

```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: myapp:v2.0
    resources:
      requests:
        cpu: "250m"     # Used for SCHEDULING: node must have 250m CPU free
        memory: "512Mi" # Used for SCHEDULING: node must have 512Mi RAM free
      limits:
        cpu: "1000m"    # THROTTLED if exceeded (never killed for CPU)
        memory: "1Gi"   # KILLED (OOMKill) if exceeded
```

```
CPU behavior:
  Request → Scheduling guarantee + proportional share
  Limit   → Hard throttle (CPU is compressible — pod slows down but doesn't die)

Memory behavior:
  Request → Scheduling guarantee
  Limit   → Hard kill (memory is incompressible — OOMKill if exceeded)
```

**Quality of Service (QoS) classes — affects eviction priority:**

```yaml
# Guaranteed QoS — requests == limits (both set, equal values)
# LAST to be evicted under node pressure
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "500m"      # Same as request
    memory: "512Mi"  # Same as request
```

```yaml
# Burstable QoS — requests < limits
# Middle priority for eviction
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

```yaml
# BestEffort QoS — no requests or limits set at all
# FIRST to be evicted under node pressure
# Never use for production workloads
```

```bash
# Find pods without resource limits (dangerous!)
kubectl get pods -A -o json | jq -r '
  .items[] |
  select(.spec.containers[].resources.limits == null) |
  "\(.metadata.namespace)/\(.metadata.name)"
'

# Check actual resource usage
kubectl top pods -A --sort-by=memory
kubectl top nodes
```

---

### Q40 — How do you use node affinity, taints, and tolerations for workload placement?

> 🎯 **Scenario:** You have GPU nodes for ML and spot nodes for batch workloads. Critical services must run on on-demand nodes only.

**Answer:**

```bash
# Label nodes by type
kubectl label node gpu-node-1 accelerator=nvidia-v100
kubectl label node spot-node-1 lifecycle=spot
kubectl label node ondemand-1 lifecycle=on-demand

# Taint GPU nodes (repel all pods that don't explicitly tolerate it)
kubectl taint node gpu-node-1 gpu=true:NoSchedule

# Taint spot nodes (warn pods — they can be interrupted)
kubectl taint node spot-node-1 spot=true:NoSchedule
```

```yaml
# ML Training Job — targets GPU nodes via toleration + nodeSelector
apiVersion: v1
kind: Pod
metadata:
  name: ml-training
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

  affinity:
    nodeAffinity:
      # HARD requirement — must run on GPU nodes
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values: ["nvidia-v100", "nvidia-a100"]

  containers:
  - name: trainer
    image: tensorflow/tensorflow:latest-gpu
    resources:
      limits:
        nvidia.com/gpu: 2   # Request 2 GPUs
```

```yaml
# Batch Job — prefers spot nodes, tolerates interruption
apiVersion: batch/v1
kind: Job
spec:
  template:
    spec:
      tolerations:
      - key: "spot"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"

      affinity:
        nodeAffinity:
          # SOFT preference — prefer spot but fallback to on-demand
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: lifecycle
                operator: In
                values: ["spot"]
```

```yaml
# Critical API — must run on on-demand, spread across nodes
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 6
  template:
    spec:
      affinity:
        # HARD: must be on on-demand
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: lifecycle
                operator: In
                values: ["on-demand"]
        # HARD: spread across different nodes
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: critical-api
            topologyKey: kubernetes.io/hostname
```

---

### Q41 — How does Horizontal Pod Autoscaler (HPA) work?

> 🎯 **Scenario:** Your API handles traffic spikes. You want it to scale from 2 to 20 pods based on CPU and a custom requests-per-second metric.

**Answer:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  minReplicas: 2
  maxReplicas: 20
  metrics:
  # Built-in: scale on CPU utilization
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Scale when average CPU > 70%
  # Built-in: scale on memory
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 400Mi
  # Custom metric from Prometheus Adapter
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"     # Scale when >1000 RPS per pod
  # External metric (e.g., SQS queue depth)
  - type: External
    external:
      metric:
        name: sqs_queue_depth
        selector:
          matchLabels:
            queue: orders-queue
      target:
        type: Value
        value: "500"

  # Scaling behavior — prevents thrashing
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60      # Wait 60s before scaling up again
      policies:
      - type: Pods
        value: 4                          # Add at most 4 pods per 60s
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300     # Wait 5 min before scaling down
      policies:
      - type: Percent
        value: 10                         # Remove at most 10% per 60s
        periodSeconds: 60
```

```bash
# Check HPA status
kubectl get hpa -n production
# TARGETS: 75%/70%  means current=75%, target=70% → will scale up

kubectl describe hpa api-hpa -n production
# Shows scaling events and decisions

# HPA requires Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

### Q42 — How do you use KEDA for event-driven autoscaling?

> 🎯 **Scenario:** You have a consumer pod that processes SQS messages. You want it to scale based on queue depth, including scaling to zero when the queue is empty.

**Answer:**

```bash
# Install KEDA
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda \
  --namespace keda \
  --create-namespace
```

```yaml
# ScaledObject — scale Deployment based on SQS queue depth
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: sqs-consumer-scaler
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sqs-consumer
  minReplicaCount: 0    # Scale to ZERO when queue is empty!
  maxReplicaCount: 50
  pollingInterval: 15   # Check queue every 15 seconds
  cooldownPeriod: 60    # Wait 60s before scaling to zero
  triggers:
  - type: aws-sqs-queue
    authenticationRef:
      name: keda-aws-auth
    metadata:
      queueURL: https://sqs.us-east-1.amazonaws.com/123456789/orders-queue
      queueLength: "10"       # 1 pod per 10 messages
      awsRegion: us-east-1
      identityOwner: operator
```

```yaml
# ScaledJob — scale Jobs based on Kafka topic lag
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: kafka-processor
  namespace: production
spec:
  jobTargetRef:
    template:
      spec:
        restartPolicy: OnFailure
        containers:
        - name: processor
          image: kafka-processor:v1
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka.kafka.svc.cluster.local:9092
      consumerGroup: my-consumer-group
      topic: orders-topic
      lagThreshold: "100"   # 1 pod per 100 messages lag
```

```yaml
# Cron-based scaling — scale to 0 at night for dev clusters
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: dev-cron-scaler
  namespace: development
spec:
  scaleTargetRef:
    name: dev-api
  minReplicaCount: 0
  maxReplicaCount: 3
  triggers:
  - type: cron
    metadata:
      timezone: "America/New_York"
      start: "0 8 * * 1-5"    # Scale up 8 AM weekdays
      end: "0 20 * * 1-5"     # Scale to 0 at 8 PM weekdays
      desiredReplicas: "3"
```

---

### Q43 — How do you right-size pods using Vertical Pod Autoscaler (VPA)?

> 🎯 **Scenario:** You suspect your pods are over-provisioned (costing money) or under-provisioned (causing OOMKills). How do you determine the right sizing?

**Answer:**

```yaml
# VPA in "Off" mode — recommendations only, don't apply automatically
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  updatePolicy:
    updateMode: "Off"      # Off / Initial / Recreate / Auto
    # Off      = show recommendations only
    # Initial  = apply only when pod is first created
    # Recreate = apply and restart pods when recommendations change significantly
    # Auto     = same as Recreate currently (may drain and recreate in future)
  resourcePolicy:
    containerPolicies:
    - containerName: api
      minAllowed:
        cpu: 50m
        memory: 64Mi
      maxAllowed:
        cpu: "4"
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
```

```bash
# After running VPA in "Off" mode for 24-48 hours, check recommendations
kubectl describe vpa api-vpa -n production

# Output shows:
# Container Recommendations:
#   Container Name: api
#     Lower Bound:
#       Cpu:    100m
#       Memory: 256Mi
#     Target:                  ← USE THIS for requests
#       Cpu:    250m
#       Memory: 512Mi
#     Upper Bound:
#       Cpu:    500m
#       Memory: 1Gi
#     Uncapped Target:
#       Cpu:    250m
#       Memory: 512Mi
```

> ⚠️ **Don't use VPA + HPA on the same CPU/memory metric simultaneously** — they fight each other. Use VPA to right-size, then use HPA based on custom metrics (RPS, queue depth).

---

### Q44 — How do you implement topology spread constraints for HA?

> 🎯 **Scenario:** Your 6-replica deployment must have exactly 2 pods per availability zone for fault tolerance.

**Answer:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ha-web-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: ha-web-app
  template:
    metadata:
      labels:
        app: ha-web-app
    spec:
      topologySpreadConstraints:
      # Spread evenly across AZs
      - maxSkew: 1                           # Max pods difference between zones
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule      # Hard requirement
        labelSelector:
          matchLabels:
            app: ha-web-app
        matchLabelKeys:
        - pod-template-hash                   # K8s 1.27+: only count same deployment
        minDomains: 3                         # Require at least 3 zones to exist

      # Also spread across nodes within each zone
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway     # Soft preference for nodes
        labelSelector:
          matchLabels:
            app: ha-web-app
```

```bash
# Verify pod distribution
kubectl get pods -o wide -l app=ha-web-app

# Check which zone each node is in
kubectl get nodes -L topology.kubernetes.io/zone

# Expected output: 2 pods in each of us-east-1a, us-east-1b, us-east-1c
```

---

### Q45 — What is the Cluster Autoscaler and how does it work?

> 🎯 **Scenario:** Your cluster runs out of nodes during peak hours, and pods sit Pending. But at night there are underutilized nodes wasting money.

**Answer:**

```bash
# Install Cluster Autoscaler for EKS
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm install cluster-autoscaler autoscaler/cluster-autoscaler \
  --namespace kube-system \
  --set autoDiscovery.clusterName=my-cluster \
  --set awsRegion=us-east-1 \
  --set rbac.serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::123456789:role/ClusterAutoscalerRole
```

```yaml
# Annotate node groups for Cluster Autoscaler discovery
# (on the EC2 AutoScaling Group in AWS)
# k8s.io/cluster-autoscaler/enabled: "true"
# k8s.io/cluster-autoscaler/<cluster-name>: "owned"

# Cluster Autoscaler behavior:
# SCALE UP:  Pod is Pending due to insufficient resources → adds nodes
# SCALE DOWN: Node utilization < 50% for 10min → drains and terminates node
#             (only if all pods can be rescheduled elsewhere)
```

```yaml
# Prevent specific pods from being evicted during scale-down
apiVersion: v1
kind: Pod
metadata:
  name: critical-job
  annotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
spec:
  containers:
  - name: critical-job
    image: long-running-job:v1
```

```bash
# Check Cluster Autoscaler status and decisions
kubectl get configmap cluster-autoscaler-status -n kube-system -o yaml

# View scale events
kubectl get events -n kube-system | grep cluster-autoscaler
```

---

## 🩵 Security & RBAC

---

### Q46 — How do you set up RBAC for a developer with read-only access?

> 🎯 **Scenario:** A developer needs to view pods, logs, and deployments in the staging namespace but must not modify anything.

**Answer:**

```yaml
# Role — namespace-scoped permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-readonly
  namespace: staging
rules:
# View pods and their logs
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/status"]
  verbs: ["get", "list", "watch"]
# View services, configmaps, events
- apiGroups: [""]
  resources: ["services", "endpoints", "configmaps",
               "events", "persistentvolumeclaims", "replicationcontrollers"]
  verbs: ["get", "list", "watch"]
# View deployments, replicasets
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets",
               "daemonsets", "replicationcontrollers"]
  verbs: ["get", "list", "watch"]
# View jobs and cronjobs
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch"]
# View HPA
- apiGroups: ["autoscaling"]
  resources: ["horizontalpodautoscalers"]
  verbs: ["get", "list", "watch"]
# Explicitly NO exec, portforward, or secret access
```

```yaml
# RoleBinding — assigns Role to user/group
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-readonly-binding
  namespace: staging
subjects:
- kind: User
  name: jane.doe@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: dev-team                    # All members of this group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-readonly
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Verify permissions
kubectl auth can-i list pods \
  --namespace=staging --as=jane.doe@company.com      # → yes

kubectl auth can-i delete deployments \
  --namespace=staging --as=jane.doe@company.com      # → no

kubectl auth can-i exec pods \
  --namespace=staging --as=jane.doe@company.com      # → no

# List all permissions for a user
kubectl auth can-i --list --namespace=staging \
  --as=jane.doe@company.com
```

---

### Q47 — How do you create a ServiceAccount with minimal AWS permissions using IRSA?

> 🎯 **Scenario:** Your app pod needs to read from an S3 bucket and write to DynamoDB. How do you give it AWS permissions without static credentials?

**Answer:**

IRSA (IAM Roles for Service Accounts) binds a Kubernetes ServiceAccount to an AWS IAM role using OIDC federation. No static AWS keys in pods.

```bash
# Step 1: Create IAM OIDC provider for the EKS cluster
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --region us-east-1 \
  --approve

# Step 2: Create IAM role with trust policy for the ServiceAccount
OIDC_ISSUER=$(aws eks describe-cluster --name my-cluster \
  --query "cluster.identity.oidc.issuer" --output text)

aws iam create-role \
  --role-name MyAppRole \
  --assume-role-policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Effect\": \"Allow\",
      \"Principal\": {\"Federated\": \"arn:aws:iam::123456789:oidc-provider/${OIDC_ISSUER#*//}\"},
      \"Action\": \"sts:AssumeRoleWithWebIdentity\",
      \"Condition\": {
        \"StringEquals\": {
          \"${OIDC_ISSUER#*//}:sub\": \"system:serviceaccount:production:my-app-sa\"
        }
      }
    }]
  }"

# Step 3: Attach minimal IAM policy
aws iam put-role-policy --role-name MyAppRole \
  --policy-name MyAppPolicy \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": ["s3:GetObject", "s3:ListBucket"],
        "Resource": ["arn:aws:s3:::my-app-bucket", "arn:aws:s3:::my-app-bucket/*"]
      },
      {
        "Effect": "Allow",
        "Action": ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:Query"],
        "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/MyTable"
      }
    ]
  }'
```

```yaml
# Step 4: Create annotated ServiceAccount in K8s
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: production
  annotations:
    eks.amazonaws.com/role-arn: "arn:aws:iam::123456789:role/MyAppRole"
    eks.amazonaws.com/token-expiration: "86400"  # 24h token expiry
```

```yaml
# Step 5: Use ServiceAccount in Deployment
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      serviceAccountName: my-app-sa   # Automatically gets AWS credentials
      containers:
      - name: app
        image: myapp:v2.0
        # AWS SDK auto-discovers credentials from the projected volume
        # No AWS_ACCESS_KEY_ID or AWS_SECRET_ACCESS_KEY needed!
```

---

### Q48 — How do you enforce Pod Security Standards?

> 🎯 **Scenario:** Security audit requires that no pods in production run as root, use privileged mode, or mount host paths.

**Answer:**

```yaml
# Enforce Pod Security Standards at namespace level (K8s 1.25+, built-in)
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # enforce: deny pods that violate the policy
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.28
    # warn: allow but show warning (use during migration)
    pod-security.kubernetes.io/warn: restricted
    # audit: record in audit log
    pod-security.kubernetes.io/audit: restricted
```

**Three built-in policy levels:**

| Level | Restrictions |
|-------|-------------|
| `privileged` | No restrictions — for trusted system pods |
| `baseline` | Minimal restrictions — blocks privileged, hostNetwork, hostPID |
| `restricted` | Most secure — non-root, read-only FS, dropped capabilities, seccomp |

```yaml
# Pod that satisfies the "restricted" policy
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
  namespace: production
spec:
  securityContext:
    runAsNonRoot: true           # Must not run as root
    runAsUser: 10000             # Specific non-root UID
    runAsGroup: 10000
    fsGroup: 10000
    seccompProfile:
      type: RuntimeDefault       # Default seccomp profile required

  containers:
  - name: app
    image: myapp:v2.0
    securityContext:
      allowPrivilegeEscalation: false    # Cannot gain more privileges
      readOnlyRootFilesystem: true       # Immutable container filesystem
      runAsNonRoot: true
      capabilities:
        drop: ["ALL"]                    # Drop all Linux capabilities
        add: ["NET_BIND_SERVICE"]        # Only add what's needed

    volumeMounts:
    - name: tmp-dir                      # Writable temp dir (since rootFS is RO)
      mountPath: /tmp
    - name: cache-dir
      mountPath: /app/cache

  volumes:
  - name: tmp-dir
    emptyDir: {}
  - name: cache-dir
    emptyDir: {}
```

---

### Q49 — How do you audit who did what in your cluster?

> 🎯 **Scenario:** Someone deleted a production deployment. How do you trace exactly who did it, when, and from where?

**Answer:**

```yaml
# Audit policy — what to log and at what verbosity
# /etc/kubernetes/audit-policy.yaml (on control plane)
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
- RequestReceived   # Don't log every incoming request
rules:
# Log all modifications to critical resources in detail
- level: RequestResponse
  verbs: ["create", "update", "patch", "delete"]
  resources:
  - group: "apps"
    resources: ["deployments", "statefulsets", "daemonsets"]
  - group: ""
    resources: ["secrets", "configmaps", "serviceaccounts"]
  namespaces: ["production", "staging"]

# Log who accessed secrets (metadata only — don't log secret values)
- level: Metadata
  verbs: ["get", "list", "watch"]
  resources:
  - group: ""
    resources: ["secrets"]

# Log Node/ServiceAccount auth issues
- level: Metadata
  users: ["system:anonymous"]

# Don't log health check noise
- level: None
  nonResourceURLs: ["/healthz*", "/readyz*", "/livez*"]

# Default: log metadata for everything else
- level: Metadata
```

```bash
# Enable in kube-apiserver (add to /etc/kubernetes/manifests/kube-apiserver.yaml)
# --audit-policy-file=/etc/kubernetes/audit-policy.yaml
# --audit-log-path=/var/log/kubernetes/audit.log
# --audit-log-maxage=30
# --audit-log-maxbackup=10
# --audit-log-maxsize=100   # 100MB per file

# Search for who deleted the deployment
cat /var/log/kubernetes/audit.log | \
  jq 'select(.verb=="delete" and .objectRef.resource=="deployments" and .objectRef.name=="web-app")' | \
  jq '{time:.requestReceivedTimestamp, user:.user.username, userAgent:.userAgent, sourceIP:.sourceIPs[0]}'

# Output: {"time":"2024-01-15T14:23:07Z", "user":"john.doe", "userAgent":"kubectl/v1.28.0", "sourceIP":"10.0.1.5"}
```

---

### Q50 — How do you use OPA/Gatekeeper to enforce custom policies?

> 🎯 **Scenario:** You want to enforce that all pods have resource limits set, all images come from your private registry, and all namespaces have a required label.

**Answer:**

```bash
# Install Gatekeeper
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
```

```yaml
# ConstraintTemplate — defines the policy logic in Rego
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlimits
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLimits
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8srequiredlimits

      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        not container.resources.limits.memory
        msg := sprintf("Container '%v' must have memory limits set", [container.name])
      }

      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        not container.resources.limits.cpu
        msg := sprintf("Container '%v' must have CPU limits set", [container.name])
      }
```

```yaml
# Constraint — applies the template as an actual policy
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLimits
metadata:
  name: require-resource-limits
spec:
  enforcementAction: deny    # deny / warn / dryrun
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    namespaces: ["production", "staging"]
    excludedNamespaces: ["kube-system", "monitoring"]
```

```yaml
# Policy: images must come from approved registries
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sallowedrepos
spec:
  crd:
    spec:
      names:
        kind: K8sAllowedRepos
      validation:
        openAPIV3Schema:
          properties:
            repos:
              type: array
              items:
                type: string
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8sallowedrepos

      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        satisfied := [good | repo := input.parameters.repos[_]; good := startswith(container.image, repo)]
        not any(satisfied)
        msg := sprintf("Image '%v' is not from an approved registry", [container.image])
      }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAllowedRepos
metadata:
  name: allowed-repos
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
  parameters:
    repos:
    - "gcr.io/my-company/"
    - "123456789.dkr.ecr.us-east-1.amazonaws.com/"
```

---

### Q51 — How do you implement image scanning and signing in your pipeline?

> 🎯 **Scenario:** You want to prevent unscanned or unsigned container images from being deployed.

**Answer:**

```yaml
# GitHub Actions: scan image before push
name: Build and Scan
on: [push]
jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Build image
      run: docker build -t myapp:${{ github.sha }} .

    - name: Scan with Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: myapp:${{ github.sha }}
        format: sarif
        output: trivy-results.sarif
        severity: CRITICAL,HIGH
        exit-code: 1          # Fail pipeline on CRITICAL/HIGH vulns
        ignore-unfixed: true  # Skip vulns with no fix available

    - name: Sign with Cosign (keyless)
      uses: sigstore/cosign-installer@v3
    - run: |
        cosign sign --yes myapp:${{ github.sha }}
        # Creates a signature stored in OCI registry
```

```yaml
# Policy Controller / Connaisseur — enforce signature verification at admission
apiVersion: connaisseur.io/v1beta1
kind: ValidationPolicy
metadata:
  name: require-signed-images
spec:
  validators:
  - name: cosign
    type: cosign
    host: https://sigstore.dev
  policy:
  - pattern: "123456789.dkr.ecr.us-east-1.amazonaws.com/*:*"
    validators:
    - name: cosign
      with:
        key: k8s://cosign-keys/cosign-pub-key
```

---

### Q52 — How do you set up network segmentation between namespaces?

> 🎯 **Scenario:** You have dev, staging, and production namespaces on the same cluster. Dev pods must not be able to talk to production services.

**Answer:**

```yaml
# Block all cross-namespace traffic INTO production
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  # Only allow traffic from WITHIN production namespace
  - from:
    - podSelector: {}     # Any pod in THIS namespace
  # AND from monitoring namespace (for Prometheus scraping)
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
```

```yaml
# Allow Ingress controller (lives in ingress-nginx namespace) to reach production
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-controller
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: frontend          # Only frontend pods accessible externally
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
```

---

## 🩶 Observability & Troubleshooting

---

### Q53 — How do you set up Prometheus + Grafana monitoring?

> 🎯 **Scenario:** Your cluster has no monitoring. Set up CPU, memory, pod health metrics with dashboards and alerts.

**Answer:**

```bash
# Install kube-prometheus-stack (Prometheus + Grafana + Alertmanager + node-exporter)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=changeme \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.storageClassName=fast-ssd \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi
```

```yaml
# ServiceMonitor — tell Prometheus to scrape your app
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-server-monitor
  namespace: production
  labels:
    release: monitoring       # Must match Prometheus serviceMonitorSelector
spec:
  selector:
    matchLabels:
      app: api-server
  endpoints:
  - port: metrics
    path: /metrics
    interval: 15s
    scrapeTimeout: 10s
```

```yaml
# PrometheusRule — define alerting rules
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: api-server-alerts
  namespace: production
  labels:
    release: monitoring
spec:
  groups:
  - name: api-server
    rules:
    - alert: HighErrorRate
      expr: |
        (rate(http_requests_total{status=~"5..",job="api-server"}[5m]) /
         rate(http_requests_total{job="api-server"}[5m])) > 0.05
      for: 3m
      labels:
        severity: critical
        team: backend
      annotations:
        summary: "High error rate on {{ $labels.pod }}"
        description: "Error rate {{ $value | humanizePercentage }} for 3+ minutes"
        runbook_url: "https://wiki/runbook/high-error-rate"

    - alert: PodCrashLooping
      expr: increase(kube_pod_container_status_restarts_total[1h]) > 3
      labels:
        severity: warning
      annotations:
        summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping"

    - alert: PodMemoryNearLimit
      expr: |
        container_memory_working_set_bytes / container_spec_memory_limit_bytes > 0.90
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Pod {{ $labels.pod }} memory at {{ $value | humanizePercentage }} of limit"

    - alert: NodeDiskSpaceLow
      expr: |
        (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.10
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Node {{ $labels.instance }} disk is {{ $value | humanizePercentage }} free"
```

---

### Q54 — How do you debug a pod stuck in `Pending` state?

> 🎯 **Scenario:** You deployed a new pod and it's been in Pending state for 10 minutes. How do you diagnose and fix it?

**Answer:**

```bash
# Step 1: Always start with describe — read Events section
kubectl describe pod <pod-name> -n <namespace>
# Focus on the LAST few lines of the Events section

# ─── COMMON PENDING REASONS AND FIXES ───────────────────────────────

# Error 1: "0/3 nodes are available: 3 Insufficient memory"
# → Node doesn't have enough resources
kubectl describe nodes | grep -A 10 "Allocated resources"
kubectl top nodes
# Fix: Reduce pod requests, scale up node group, or delete unused pods

# Error 2: "0/3 nodes are available: 3 node(s) had taint {key: value}"
# → Pod doesn't tolerate node taints
kubectl get nodes -o custom-columns='NAME:.metadata.name,TAINTS:.spec.taints'
# Fix: Add toleration to pod spec

# Error 3: "0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector"
# → Node labels don't match pod's nodeSelector or nodeAffinity
kubectl get nodes --show-labels
# Fix: Correct nodeSelector/affinity or add missing labels to nodes

# Error 4: "pod has unbound immediate PersistentVolumeClaims"
# → PVC not bound to a PV
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>
# Fix: Create StorageClass, create PV, or wait for dynamic provisioning

# Error 5: "persistentvolumeclaim ... not found"
# → PVC doesn't exist at all
# Fix: kubectl apply -f pvc.yaml

# Error 6: "Maximum number of Pods is already running"
# → Node has hit pod limit (default 110 pods/node)
kubectl describe node <node> | grep "pods:"
# Fix: Add more nodes, reduce pods per node, or increase --max-pods kubelet flag
```

```bash
# Check scheduler events specifically
kubectl get events -n <namespace> \
  --field-selector reason=FailedScheduling \
  --sort-by='.lastTimestamp'

# Check if ResourceQuota is blocking
kubectl get resourcequota -n <namespace>
kubectl describe resourcequota -n <namespace>

# Check LimitRange
kubectl get limitrange -n <namespace>
kubectl describe limitrange -n <namespace>
```

---

### Q55 — How do you debug service connectivity issues?

> 🎯 **Scenario:** Pod A cannot connect to Service B in the same namespace. How do you diagnose the root cause?

**Answer:**

```bash
# Step 1: Verify pod and service exist
kubectl get pods -l app=service-b -n production
kubectl get svc service-b -n production

# Step 2: Check endpoints — are any pods backing the service?
kubectl get endpoints service-b -n production
# NAME        ENDPOINTS           AGE
# service-b   <none>              5m  ← PROBLEM: empty endpoints

# Diagnose empty endpoints:
# a) Get service selector
kubectl get svc service-b -o jsonpath='{.spec.selector}' -n production
# {"app":"service-b","version":"v1"}

# b) Check if any pods match ALL selector labels
kubectl get pods -n production -l app=service-b,version=v1 --show-labels
# If no pods found → label mismatch!

# c) Check pod readiness (unready pods are excluded from endpoints)
kubectl get pods -n production -l app=service-b
# If READY=0/1 → pod failing readiness probe

# Step 3: Test DNS resolution from pod-a
kubectl exec -it pod-a -n production -- nslookup service-b
kubectl exec -it pod-a -n production -- \
  nslookup service-b.production.svc.cluster.local

# Step 4: Test port connectivity
kubectl exec -it pod-a -n production -- \
  nc -zv service-b 8080
# or:
kubectl exec -it pod-a -n production -- \
  curl -v http://service-b:8080/health

# Step 5: Check NetworkPolicies blocking traffic
kubectl get networkpolicy -n production
kubectl describe networkpolicy <policy-name> -n production

# Step 6: Debug with ephemeral container (K8s 1.23+)
kubectl debug -it pod-a -n production \
  --image=nicolaka/netshoot \
  --target=app-container

# Inside netshoot: full network debugging toolkit
curl -v http://service-b:8080
tcpdump -i eth0 host service-b
nmap -p 8080 service-b
```

---

### Q56 — How do you investigate high memory or CPU usage?

> 🎯 **Scenario:** Your cluster is slow and nodes are under pressure. How do you identify the culprit pods?

**Answer:**

```bash
# Check node pressure
kubectl get nodes
# If STATUS shows "MemoryPressure" or "DiskPressure" — node is struggling

kubectl describe node <node-name>
# Look at: Conditions, Allocated Resources, Events

# Find the top CPU consumers
kubectl top pods -A --sort-by=cpu | head -20

# Find the top memory consumers
kubectl top pods -A --sort-by=memory | head -20

# Check container-level usage
kubectl top pods -A --containers --sort-by=memory | head -30

# Find pods near their memory limits (risk of OOMKill)
kubectl get pods -A -o json | jq -r '
  .items[] |
  .metadata.namespace + "/" + .metadata.name
' | while read pod; do
  ns=$(echo $pod | cut -d/ -f1)
  name=$(echo $pod | cut -d/ -f2)
  kubectl top pod $name -n $ns --containers 2>/dev/null
done

# PromQL queries for investigation:
# Top memory consumers:
# topk(10, container_memory_working_set_bytes{container!="POD"})

# Memory usage % of limit:
# container_memory_working_set_bytes / container_spec_memory_limit_bytes > 0.8

# OOMKill history:
# increase(kube_pod_container_status_restarts_total[24h]) > 0
# kube_pod_container_status_last_terminated_reason{reason="OOMKilled"} == 1
```

```bash
# Find and clean up resources wasting cluster capacity
# Deployments with 0 replicas
kubectl get deployments -A --field-selector=spec.replicas=0

# Completed/failed pods not cleaned up
kubectl get pods -A --field-selector=status.phase=Succeeded
kubectl get pods -A --field-selector=status.phase=Failed

# Clean up completed pods older than 1 day
kubectl get pods -A --field-selector=status.phase=Succeeded -o json | \
  jq -r '.items[] | select(.status.startTime < "2024-01-14") | .metadata.namespace + "/" + .metadata.name'
```

---

### Q57 — How do you do live debugging with ephemeral containers?

> 🎯 **Scenario:** A distroless production container has no shell or debugging tools. You need to debug it in production without restarting it.

**Answer:**

```bash
# Ephemeral containers (K8s 1.23+ GA) — inject a debug container into a running pod
# The debug container shares the same network, PID namespace as the target container

# Basic debug with busybox
kubectl debug -it <pod-name> \
  --image=busybox:1.35 \
  --target=<container-name>   # --target shares PID namespace

# Debug with netshoot (full network toolkit)
kubectl debug -it <pod-name> \
  --image=nicolaka/netshoot \
  --target=main-app

# Debug with a copy of the pod (with modifications)
kubectl debug <pod-name> \
  -it \
  --copy-to=debug-pod \
  --image=myapp:debug \    # Override with debug-enabled image
  --share-processes        # Share PID namespace to see original app process

# Debug a node (runs privileged container with host filesystem access)
kubectl debug node/<node-name> \
  -it \
  --image=ubuntu:22.04

# Inside node debug pod:
# chroot /host   → access the full node filesystem
# systemctl status kubelet
# journalctl -u kubelet -f
```

```bash
# Alternative: inject a debug sidecar via strategic merge patch
kubectl patch pod <pod-name> -n production \
  --patch '{"spec":{"ephemeralContainers":[{
    "name":"debug",
    "image":"nicolaka/netshoot",
    "stdin":true,
    "tty":true,
    "targetContainerName":"main-app"
  }]}}'

kubectl attach <pod-name> -c debug -it
```

---

### Q58 — How do you trace a slow request across multiple microservices?

> 🎯 **Scenario:** Your API response time is 3 seconds. You have 5 microservices in the call chain. How do you find where the slowness is?

**Answer:**

```bash
# Install Jaeger (distributed tracing)
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm install jaeger jaegertracing/jaeger \
  --namespace tracing \
  --create-namespace \
  --set allInOne.enabled=true \
  --set provisionDataStore.cassandra=false
```

```yaml
# OpenTelemetry Collector — collects traces from all services
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
  namespace: tracing
spec:
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
    processors:
      batch: {}
      memory_limiter:
        limit_mib: 400
    exporters:
      jaeger:
        endpoint: jaeger-collector:14250
        tls:
          insecure: true
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, batch]
          exporters: [jaeger]
```

```python
# Python app — instrument with OpenTelemetry
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

provider = TracerProvider()
provider.add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="otel-collector:4317"))
)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("my-service")

@app.route("/orders")
def get_orders():
    with tracer.start_as_current_span("get-orders") as span:
        span.set_attribute("user.id", user_id)
        # Call downstream services — trace ID propagated automatically
        result = call_inventory_service()
        return result
```

---

### Q59 — How do you implement log aggregation with Loki?

> 🎯 **Scenario:** Your team wants to search and correlate logs across hundreds of pods in Grafana (same tool you use for metrics).

**Answer:**

```bash
# Install Loki + Promtail + Grafana (Loki Stack)
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki-stack grafana/loki-stack \
  --namespace logging \
  --create-namespace \
  --set loki.enabled=true \
  --set promtail.enabled=true \
  --set grafana.enabled=true \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=50Gi
```

```yaml
# Promtail ConfigMap — scrape pod logs with metadata enrichment
apiVersion: v1
kind: ConfigMap
metadata:
  name: promtail-config
  namespace: logging
data:
  promtail.yaml: |
    server:
      http_listen_port: 3101
    clients:
    - url: http://loki:3100/loki/api/v1/push
    scrape_configs:
    - job_name: kubernetes-pods
      kubernetes_sd_configs:
      - role: pod
      pipeline_stages:
      - cri: {}
      - labeldrop:
        - filename
      relabel_configs:
      # Enrich logs with K8s metadata
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_label_app]
        target_label: app
      - source_labels: [__meta_kubernetes_pod_container_name]
        target_label: container
```

```
# LogQL queries (Loki Query Language) in Grafana:

# View all logs from production namespace
{namespace="production"}

# View error logs from specific app
{app="api-server", namespace="production"} |= "ERROR"

# Count error rate
rate({app="api-server"} |= "ERROR" [5m])

# Parse JSON logs and filter by field
{app="api-server"} | json | status_code >= 500

# View logs from specific pod
{pod="api-server-7d9f4b-xxxx"}

# Correlate with trace ID
{namespace="production"} | json | traceID = "abc123def456"
```

---

### Q60 — How do you troubleshoot node-level issues?

> 🎯 **Scenario:** A node is in NotReady state and pods on it are being evicted. How do you investigate?

**Answer:**

```bash
# Step 1: Check node status
kubectl get nodes
# NAME        STATUS     ROLES    AGE
# node-1      NotReady   <none>   30d  ← Problem

# Step 2: Describe the node — check Conditions
kubectl describe node node-1
# Conditions:
#   MemoryPressure    False   (if True: node is OOM)
#   DiskPressure      False   (if True: disk almost full)
#   PIDPressure       False   (if True: too many processes)
#   Ready             False   ← PROBLEM
#
# Events:
#   Warning NodeNotReady  kubelet stopped posting node status

# Step 3: SSH to the node and check kubelet
ssh node-1
sudo systemctl status kubelet
sudo journalctl -u kubelet -f --since "1 hour ago"

# Common kubelet issues:
# "Unable to connect to the server" → networking issue
# "certificate has expired" → TLS certs expired
# "failed to get node info" → API server unreachable
# "PLEG is not healthy" → container runtime stuck

# Step 4: Check container runtime
sudo systemctl status containerd
sudo crictl ps   # List running containers via CRI

# Step 5: Check disk space
df -h
du -sh /var/lib/containerd   # Check container storage

# Step 6: Check memory
free -h
sudo dmesg | grep -i "oom\|killed"   # OOM kill events

# Step 7: Check network connectivity to control plane
nc -zv <api-server-ip> 6443

# Step 8: Restart kubelet if needed
sudo systemctl restart kubelet
sudo systemctl restart containerd
```

---

## 🟤 CI/CD & GitOps

---

### Q61 — How do you implement GitOps with ArgoCD?

> 🎯 **Scenario:** Your team wants every change to Kubernetes to go through Git — no manual `kubectl apply` in production.

**Answer:**

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get initial admin password
argocd admin initial-password -n argocd

# Expose UI via LoadBalancer
kubectl patch svc argocd-server -n argocd \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

```yaml
# ArgoCD Application — watches Git and syncs to cluster
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app-production
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io   # Cascade delete
spec:
  project: production-apps
  source:
    repoURL: https://github.com/my-org/k8s-manifests.git
    targetRevision: main
    path: apps/web-app/overlays/production
    # For Helm:
    # chart: web-app
    # helm:
    #   releaseName: web-app
    #   valueFiles: [values-prod.yaml]
    #   parameters:
    #   - name: image.tag
    #     value: v2.1.0
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true         # Delete resources removed from Git
      selfHeal: true      # Revert manual changes back to Git state
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - ApplyOutOfSyncOnly=true   # Only sync changed resources
    retry:
      limit: 5
      backoff:
        duration: 10s
        factor: 2
        maxDuration: 5m
```

```yaml
# ArgoCD AppProject — group apps and restrict permissions
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production-apps
  namespace: argocd
spec:
  description: Production applications
  # Which repos this project can use
  sourceRepos:
  - https://github.com/my-org/k8s-manifests.git
  # Which clusters/namespaces this project can deploy to
  destinations:
  - namespace: production
    server: https://kubernetes.default.svc
  # Which K8s resources this project can create
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  namespaceResourceWhitelist:
  - group: apps
    kind: Deployment
  - group: ''
    kind: Service
  # Prevent deletion in production
  orphanedResources:
    warn: true
```

---

### Q62 — How do you implement image promotion across environments?

> 🎯 **Scenario:** After an image passes tests in staging, you want to automatically promote it to production without changing the manifest files.

**Answer:**

```yaml
# CI/CD image promotion workflow

# .github/workflows/promote.yml
name: Promote to Production
on:
  workflow_dispatch:
    inputs:
      image_tag:
        description: 'Image tag to promote to production'
        required: true
  # Or trigger automatically when staging tests pass:
  # workflow_run:
  #   workflows: ["Staging Tests"]
  #   types: [completed]
  #   branches: [main]

jobs:
  promote:
    runs-on: ubuntu-latest
    if: github.event.workflow_run.conclusion == 'success'
    steps:
    - uses: actions/checkout@v4

    - name: Install Kustomize
      run: curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

    - name: Update production image tag
      run: |
        cd overlays/production
        kustomize edit set image myapp=myapp:${{ inputs.image_tag }}

    - name: Create Pull Request
      uses: peter-evans/create-pull-request@v5
      with:
        title: "Promote ${{ inputs.image_tag }} to production"
        body: |
          Promoting image tag `${{ inputs.image_tag }}` to production.
          Tested and validated in staging.
        branch: promote/${{ inputs.image_tag }}
        base: main
        labels: ["promotion", "production"]
```

```bash
# ArgoCD Image Updater — automatic image promotion based on tag policy
helm install argocd-image-updater \
  argo/argocd-image-updater \
  --namespace argocd
```

```yaml
# ArgoCD Application with Image Updater annotations
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app-staging
  annotations:
    argocd-image-updater.argoproj.io/image-list: "myapp=123456.dkr.ecr.us-east-1.amazonaws.com/myapp"
    argocd-image-updater.argoproj.io/myapp.update-strategy: semver
    argocd-image-updater.argoproj.io/myapp.allow-tags: "~1.x.x"   # Only 1.x.x tags
    argocd-image-updater.argoproj.io/write-back-method: git        # Write tag to Git
```

---

### Q63 — How do you set up a complete Terraform + Kubernetes CI/CD pipeline?

> 🎯 **Scenario:** Your team manages both AWS infrastructure (Terraform) and Kubernetes workloads (Helm charts) in the same repository. How do you automate deployments safely?

**Answer:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  id-token: write        # OIDC
  contents: read
  pull-requests: write

jobs:
  # ─── TERRAFORM ──────────────────────────────────────────
  terraform:
    name: Terraform Plan/Apply
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./infrastructure
    steps:
    - uses: actions/checkout@v4

    - name: Configure AWS credentials (OIDC)
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::123456789:role/github-actions-role
        aws-region: us-east-1

    - uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: 1.7.0

    - run: terraform init
    - run: terraform validate
    - run: terraform fmt -check

    - name: Terraform Plan
      id: plan
      run: terraform plan -out=tfplan -no-color 2>&1 | tee plan.txt

    - name: Comment Plan on PR
      if: github.event_name == 'pull_request'
      uses: actions/github-script@v7
      with:
        script: |
          const plan = require('fs').readFileSync('./infrastructure/plan.txt','utf8')
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: '## Terraform Plan\n```\n' + plan.slice(0,60000) + '\n```'
          })

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve tfplan

  # ─── HELM DEPLOY ────────────────────────────────────────
  helm-deploy:
    name: Deploy to Kubernetes
    runs-on: ubuntu-latest
    needs: terraform
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
    - uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::123456789:role/github-actions-role
        aws-region: us-east-1

    - name: Update kubeconfig
      run: aws eks update-kubeconfig --name my-cluster --region us-east-1

    - name: Build and push image
      run: |
        aws ecr get-login-password | docker login --username AWS \
          --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
        docker build -t myapp:${{ github.sha }} .
        docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:${{ github.sha }}

    - name: Helm upgrade
      run: |
        helm upgrade --install web-app ./charts/web-app \
          --namespace production \
          --set image.tag=${{ github.sha }} \
          --set image.repository=123456789.dkr.ecr.us-east-1.amazonaws.com/myapp \
          --values charts/web-app/values-prod.yaml \
          --wait \
          --timeout 10m \
          --atomic   # Auto-rollback if upgrade fails

    - name: Verify deployment
      run: |
        kubectl rollout status deployment/web-app -n production
        kubectl get pods -n production -l app=web-app
```

---

### Q64 — How do you implement blue-green deployments?

> 🎯 **Scenario:** You need an instant traffic switch to a new version with instant rollback capability — rolling update is too slow.

**Answer:**

```yaml
# Blue deployment — current production (receives all traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
      slot: blue
  template:
    metadata:
      labels:
        app: web-app
        slot: blue
        version: v1.0
    spec:
      containers:
      - name: web
        image: myapp:v1.0
---
# Green deployment — new version (deployed, but receives no traffic yet)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
      slot: green
  template:
    metadata:
      labels:
        app: web-app
        slot: green
        version: v2.0
    spec:
      containers:
      - name: web
        image: myapp:v2.0
---
# Service — currently pointing to blue
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
    slot: blue       # ← Change to "green" to switch all traffic instantly
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Deploy green alongside blue (no traffic yet)
kubectl apply -f green-deployment.yaml

# Test green directly before switching
kubectl port-forward deployment/web-app-green 8081:8080
curl http://localhost:8081/health
curl http://localhost:8081/api/test

# Run smoke tests against green
# ...all good?

# SWITCH TRAFFIC — instant, takes effect in <1 second
kubectl patch service web-app \
  -p '{"spec":{"selector":{"slot":"green"}}}'

# Verify switch took effect
kubectl get endpoints web-app   # Should show green pod IPs

# Monitor error rate for 15 minutes...

# If issues: INSTANT ROLLBACK (1 command)
kubectl patch service web-app \
  -p '{"spec":{"selector":{"slot":"blue"}}}'

# After successful validation: clean up blue
kubectl delete deployment web-app-blue
```

---

### Q65 — How do you use Helm with multiple environments and secrets?

> 🎯 **Scenario:** You need to manage Helm deployments across dev, staging, and production with different values and encrypted secrets in each.

**Answer:**

```
Chart structure:
my-app/
├── Chart.yaml
├── values.yaml            # Defaults
├── values-dev.yaml        # Dev overrides
├── values-staging.yaml    # Staging overrides
├── values-prod.yaml       # Production overrides
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── hpa.yaml
```

```yaml
# values.yaml (base defaults)
replicaCount: 1
image:
  repository: myapp
  tag: latest
  pullPolicy: IfNotPresent
resources:
  requests:
    cpu: 100m
    memory: 128Mi
autoscaling:
  enabled: false
ingress:
  enabled: false
```

```yaml
# values-prod.yaml (production overrides)
replicaCount: 5
image:
  repository: 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp
  tag: "v2.1.0"
  pullPolicy: Always
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: "2"
    memory: 2Gi
autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
ingress:
  enabled: true
  host: api.example.com
  tlsEnabled: true
podDisruptionBudget:
  enabled: true
  minAvailable: 3
```

```bash
# Deploy to production
helm upgrade --install web-app ./my-app \
  --namespace production \
  --values values.yaml \
  --values values-prod.yaml \
  --set image.tag=$IMAGE_TAG \
  --atomic \
  --wait \
  --timeout 10m

# Helm Secrets plugin — encrypt secret values with SOPS
helm secrets enc secrets-prod.yaml   # Encrypt
helm secrets dec secrets-prod.yaml   # Decrypt

helm upgrade --install web-app ./my-app \
  --values values-prod.yaml \
  --values secrets-prod.yaml          # Auto-decrypted by helm-secrets plugin

# Helmfile — manage multiple Helm releases declaratively
helmfile sync                         # Apply all releases
helmfile diff                         # Show pending changes
helmfile apply --selector app=web-app # Apply specific release
```

---

### Q66 — How do you run database migrations safely in Kubernetes?

> 🎯 **Scenario:** Every deployment requires running database migrations. How do you ensure migrations run exactly once, in the right order, and deployments don't start if migrations fail?

**Answer:**

```yaml
# Helm hook — runs before deployment upgrade
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration-{{ .Release.Revision }}
  namespace: {{ .Release.Namespace }}
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"           # Run early (lower = first)
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  backoffLimit: 2
  activeDeadlineSeconds: 300
  template:
    spec:
      restartPolicy: Never
      serviceAccountName: migration-sa
      initContainers:
      # Wait for database to be ready before migrating
      - name: wait-for-db
        image: busybox:1.35
        command: ['sh', '-c',
          'until nc -z postgres-service 5432; do sleep 2; done']
      containers:
      - name: migrate
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        command: ["python", "manage.py", "migrate", "--no-input"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        - name: MIGRATION_LOCK_TIMEOUT
          value: "60s"          # Prevent deadlock if another migration is running
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
```

```bash
# Helm deployment order with hooks:
# 1. pre-install / pre-upgrade → migration job runs
# 2. If migration succeeds → deployment proceeds
# 3. If migration fails → deployment is aborted (with --atomic)

helm upgrade --install web-app ./my-app \
  --atomic \                    # Rollback if any hook fails
  --timeout 15m \
  --wait

# Check migration job logs
kubectl logs -l job-name=db-migration -n production

# For ArgoCD: use Resource Hooks
# annotations:
#   argocd.argoproj.io/hook: PreSync
#   argocd.argoproj.io/hook-delete-policy: HookSucceeded
```

---

### Q67 — How do you test Kubernetes manifests in CI before deploying?

> 🎯 **Scenario:** Broken YAML or misconfigured manifests reach production, causing deployment failures. How do you catch these in CI?

**Answer:**

```yaml
# GitHub Actions — comprehensive manifest validation
name: Validate K8s Manifests
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    # 1. YAML syntax validation
    - name: Validate YAML
      run: |
        pip install yamllint
        yamllint -d relaxed k8s/

    # 2. Kubernetes schema validation with kubeval
    - name: Kubeval
      run: |
        wget https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz
        tar xf kubeval-linux-amd64.tar.gz
        ./kubeval --kubernetes-version=1.28.0 k8s/**/*.yaml

    # 3. Advanced validation with kubeconform
    - name: Kubeconform
      uses: docker://ghcr.io/yannh/kubeconform:latest
      with:
        args: "-strict -summary -kubernetes-version 1.28.0 k8s/"

    # 4. Security scanning with Trivy
    - name: Trivy K8s scan
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: config
        scan-ref: k8s/
        severity: CRITICAL,HIGH
        exit-code: 1

    # 5. Policy compliance with Checkov
    - name: Checkov K8s policies
      uses: bridgecrewio/checkov-action@master
      with:
        directory: k8s/
        framework: kubernetes
        soft_fail: false

    # 6. Kustomize build verification
    - name: Kustomize build
      run: |
        for env in dev staging production; do
          echo "Building $env..."
          kubectl kustomize k8s/overlays/$env > /dev/null
          echo "$env: OK"
        done

    # 7. Helm chart linting
    - name: Helm lint
      run: |
        helm lint charts/web-app/ \
          --values charts/web-app/values-prod.yaml
```

---

## ⚡ Advanced & Production Patterns

---

### Q68 — How do you implement pod topology spread with custom labels?

> 🎯 **Scenario:** You have 12 pods and 4 nodes across 2 AZs. You want at most 2 pods per node and even distribution across AZs.

**Answer:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: critical-api
spec:
  replicas: 12
  selector:
    matchLabels:
      app: critical-api
  template:
    metadata:
      labels:
        app: critical-api
    spec:
      topologySpreadConstraints:
      # Constraint 1: Spread across AZs — max 1 pod difference between zones
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: critical-api

      # Constraint 2: Spread across nodes — max 2 pods per node
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: critical-api

      # ANTI-AFFINITY: Soft preference to avoid same node as DB
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: postgres
              topologyKey: kubernetes.io/hostname
```

---

### Q69 — How do you handle stateful applications with the Operator pattern?

> 🎯 **Scenario:** Your team wants to run Kafka in Kubernetes with automatic partition rebalancing, rolling upgrades, and self-healing. Should you write a StatefulSet or use an Operator?

**Answer:**

Operators encode human operational knowledge into code — they extend Kubernetes with domain-specific controllers.

```bash
# Install Strimzi Kafka Operator
helm repo add strimzi https://strimzi.io/charts
helm install strimzi-kafka-operator strimzi/strimzi-kafka-operator \
  --namespace kafka --create-namespace
```

```yaml
# Kafka cluster managed by Strimzi Operator
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: production-kafka
  namespace: kafka
spec:
  kafka:
    version: 3.6.0
    replicas: 3
    listeners:
    - name: plain
      port: 9092
      type: internal
      tls: false
    - name: tls
      port: 9093
      type: internal
      tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      default.replication.factor: 3
      min.insync.replicas: 2
    storage:
      type: persistent-claim
      size: 100Gi
      class: fast-ssd
    resources:
      requests:
        cpu: "1"
        memory: 4Gi
      limits:
        cpu: "2"
        memory: 4Gi
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
      class: fast-ssd
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

```yaml
# Strimzi manages Topic creation declaratively
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: orders-topic
  namespace: kafka
  labels:
    strimzi.io/cluster: production-kafka
spec:
  partitions: 12
  replicas: 3
  config:
    retention.ms: 604800000     # 7 days
    segment.bytes: 1073741824   # 1GB segments
```

> 💡 **When to use an Operator:** For complex stateful applications (Kafka, PostgreSQL, Elasticsearch, Redis) where the operational runbook has many steps. Check OperatorHub.io before writing your own.

---

### Q70 — How do you implement multi-tenancy in Kubernetes?

> 🎯 **Scenario:** You're building a SaaS platform where each customer gets an isolated environment on your Kubernetes cluster.

**Answer:**

```
Multi-tenancy approaches:

1. Namespace-per-tenant (soft isolation)
   ├── Pro: Simple, low overhead
   ├── Con: Shared kernel, shared cluster DNS
   └── Use: Internal teams, trusted tenants

2. Cluster-per-tenant (hard isolation)
   ├── Pro: Complete isolation
   ├── Con: Expensive, high operational overhead
   └── Use: Highly regulated industries, untrusted code

3. vCluster (virtual clusters)
   ├── Pro: Near-complete isolation, cheaper than full clusters
   ├── Con: Complexity of nested Kubernetes
   └── Use: Best balance for SaaS multi-tenancy
```

```bash
# vCluster — virtual Kubernetes clusters inside namespaces
helm repo add loft-sh https://charts.loft.sh
helm install vcluster-tenant-a vcluster \
  --repo https://charts.loft.sh \
  --namespace tenant-a \
  --create-namespace \
  --values - <<EOF
sync:
  ingresses:
    enabled: true
storage:
  size: 5Gi
isolation:
  enabled: true
  podSecurityStandard: baseline
  resourceQuota:
    enabled: true
    quota:
      requests.cpu: "10"
      requests.memory: 20Gi
      pods: "50"
EOF

# Connect to vCluster as tenant-a admin
vcluster connect vcluster-tenant-a -n tenant-a
kubectl get pods   # Connected to tenant's virtual cluster
```

```yaml
# HNC (Hierarchical Namespace Controller) — namespace trees
# parent namespace propagates RBAC and policies to children
apiVersion: hnc.x-k8s.io/v1alpha2
kind: HierarchyConfiguration
metadata:
  name: hierarchy
  namespace: tenant-a-prod
spec:
  parent: tenant-a   # Inherits RBAC from tenant-a namespace
```

---

### Q71 — How do you implement cost optimization in Kubernetes?

> 🎯 **Scenario:** Your AWS EKS bill is $60K/month. How do you reduce it by 30% without impacting production performance?

**Answer:**

```bash
# 1. Install Kubecost for cost visibility
helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm install kubecost kubecost/cost-analyzer \
  --namespace kubecost --create-namespace \
  --set global.prometheus.enabled=true

# Access Kubecost UI to see cost breakdown per namespace/workload
kubectl port-forward svc/kubecost-cost-analyzer 9090:9090 -n kubecost
```

```yaml
# 2. Use Spot instances for non-critical workloads
# Node group with mixed instances (EKS Managed Node Group)
# 0% on-demand base, 100% Spot above base
# Multiple instance types for Spot diversity

# Deployment that tolerates spot interruption
spec:
  template:
    spec:
      tolerations:
      - key: "eks.amazonaws.com/capacityType"
        operator: "Equal"
        value: "SPOT"
        effect: "NoSchedule"
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: eks.amazonaws.com/capacityType
                operator: In
                values: ["SPOT"]
```

```yaml
# 3. Scale to zero dev/staging at night with KEDA
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: dev-scale-to-zero
  namespace: development
spec:
  scaleTargetRef:
    name: dev-api
  minReplicaCount: 0         # Scales to ZERO
  maxReplicaCount: 5
  triggers:
  - type: cron
    metadata:
      timezone: America/New_York
      start: "0 9 * * 1-5"   # Up at 9 AM weekdays
      end: "0 19 * * 1-5"    # Down at 7 PM weekdays
      desiredReplicas: "3"
```

```bash
# 4. Right-size with VPA recommendations
kubectl get vpa -A -o json | jq '.items[] | {
  name: .metadata.name,
  namespace: .metadata.namespace,
  current: .spec.resourcePolicy,
  recommended: .status.recommendation.containerRecommendations
}'

# 5. Remove unused resources
# Identify unused PVCs (no pod mounting them)
kubectl get pvc -A -o json | jq -r '
  .items[] |
  select(.status.phase == "Bound") |
  select(.metadata.annotations["pv.kubernetes.io/bind-completed"] == "yes") |
  .metadata.namespace + "/" + .metadata.name
'

# 6. Use Descheduler to rebalance pods after scale-down
helm install descheduler kubernetes-sigs/descheduler \
  --namespace kube-system \
  --set cronJobApiVersion=batch/v1 \
  --set schedule="0 */2 * * *"
```

**Cost saving levers:**

| Action | Expected Savings |
|--------|-----------------|
| Right-size over-provisioned pods via VPA | 20–40% |
| Spot instances for dev/batch workloads | 60–80% on those nodes |
| Scale dev/staging to zero at night | 50–70% on those envs |
| Cluster Autoscaler aggressive scale-down | 15–25% |
| Spot + Cluster Autoscaler for production batch | 40–60% |
| Remove unused PVs and idle LoadBalancers | 5–10% |

---

### Q72 — How do you implement a service mesh with Istio?

> 🎯 **Scenario:** Your microservices need automatic mTLS, circuit breaking, retry logic, and distributed tracing without changing application code.

**Answer:**

```bash
# Install Istio
istioctl install --set profile=production

# Enable automatic sidecar injection for production namespace
kubectl label namespace production istio-injection=enabled

# All new pods in "production" now automatically get an Envoy proxy sidecar
kubectl rollout restart deployment -n production
```

```yaml
# Traffic management — canary with header-based routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web-app
  namespace: production
spec:
  hosts:
  - web-app
  http:
  # Header-based routing for internal testers
  - match:
    - headers:
        x-canary:
          exact: "true"
    route:
    - destination:
        host: web-app
        subset: v2
  # Weight-based traffic split (10% canary)
  - route:
    - destination:
        host: web-app
        subset: v1
      weight: 90
    - destination:
        host: web-app
        subset: v2
      weight: 10
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: "gateway-error,connect-failure,retriable-4xx"
    timeout: 10s
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: web-app
spec:
  host: web-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    outlierDetection:              # Circuit breaker
      consecutive5xxErrors: 5
      interval: 10s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
  - name: v1
    labels:
      version: v1.0
  - name: v2
    labels:
      version: v2.0
```

---

### Q73 — How do you manage cluster upgrades safely?

> 🎯 **Scenario:** Your EKS cluster is on K8s 1.26 and needs to upgrade to 1.28. How do you do it with zero downtime?

**Answer:**

```bash
# Pre-upgrade checklist:

# 1. Check deprecated APIs being used
# Install pluto
helm install pluto --repo https://fairwinds.github.io/pluto pluto
kubectl pluto detect-helm    # Check Helm releases
pluto detect-files -d ./k8s  # Check manifest files

# Common API removals in recent versions:
# K8s 1.25: PodSecurityPolicy removed
# K8s 1.26: FlowSchema v1beta1 removed
# K8s 1.27: CSIStorageCapacity v1beta1 removed

# 2. Check addon compatibility (CoreDNS, kube-proxy, CSI drivers)
kubectl get pods -n kube-system -o wide

# 3. Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/pre-upgrade.db ...

# 4. Ensure PodDisruptionBudgets exist for critical workloads
kubectl get pdb -A

# 5. Test upgrade in staging first!
```

```bash
# EKS upgrade process:

# Step 1: Upgrade control plane (AWS manages this)
aws eks update-cluster-version \
  --name my-cluster \
  --kubernetes-version 1.28

# Wait for control plane upgrade
aws eks wait cluster-active --name my-cluster

# Step 2: Update managed addons
aws eks update-addon --cluster-name my-cluster --addon-name kube-proxy \
  --addon-version v1.28.0-eksbuild.1
aws eks update-addon --cluster-name my-cluster --addon-name coredns \
  --addon-version v1.10.1-eksbuild.1
aws eks update-addon --cluster-name my-cluster --addon-name aws-ebs-csi-driver \
  --addon-version v1.25.0-eksbuild.1

# Step 3: Upgrade node groups (rolling replacement of nodes)
aws eks update-nodegroup-version \
  --cluster-name my-cluster \
  --nodegroup-name main \
  --kubernetes-version 1.28

# Monitor node group update
aws eks wait nodegroup-active --cluster-name my-cluster --nodegroup-name main

# Verify cluster is healthy
kubectl get nodes
kubectl get pods -A | grep -v Running | grep -v Completed
```

---

### Q74 — How do you implement chaos engineering in Kubernetes?

> 🎯 **Scenario:** You want to test your system's resilience by intentionally causing failures and verifying your application handles them gracefully.

**Answer:**

```bash
# Install Chaos Mesh
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh \
  --namespace chaos-mesh \
  --create-namespace \
  --set chaosDaemon.runtime=containerd \
  --set chaosDaemon.socketPath=/run/containerd/containerd.sock
```

```yaml
# PodChaos — randomly kill pods (simulates node failures)
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure-test
  namespace: production
spec:
  action: pod-kill             # pod-kill / pod-failure / container-kill
  mode: random-max-percent
  value: "30"                  # Kill up to 30% of matching pods
  selector:
    namespaces:
    - production
    labelSelectors:
      app: web-app
  scheduler:
    cron: "@every 10m"         # Run every 10 minutes
```

```yaml
# NetworkChaos — simulate network latency (test timeout handling)
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay-test
spec:
  action: delay
  mode: all
  selector:
    namespaces:
    - production
    labelSelectors:
      app: api-server
  delay:
    latency: "200ms"          # Add 200ms latency
    correlation: "25"
    jitter: "50ms"
  direction: to               # Affects outgoing traffic
  duration: "5m"
```

```yaml
# StressChaos — memory pressure test (verify OOMKill behavior)
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: memory-stress-test
spec:
  mode: one
  selector:
    namespaces: [production]
    labelSelectors:
      app: api-server
  stressors:
    memory:
      workers: 1
      size: "512MB"       # Allocate 512MB in the target container
  duration: "1m"
```

---

### Q75 — How do you implement GitOps with Flux?

> 🎯 **Scenario:** Your team wants a lightweight GitOps solution that automatically syncs Git to the cluster without a UI dependency.

**Answer:**

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Bootstrap Flux — installs controllers + configures Git repo
flux bootstrap github \
  --owner=my-org \
  --repository=k8s-manifests \
  --branch=main \
  --path=clusters/production \
  --personal
```

```yaml
# GitRepository — tells Flux where to pull manifests from
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: k8s-manifests
  namespace: flux-system
spec:
  interval: 1m           # Check for changes every minute
  url: https://github.com/my-org/k8s-manifests
  ref:
    branch: main
  secretRef:
    name: github-token
```

```yaml
# Kustomization — applies manifests from the GitRepository
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: production-apps
  namespace: flux-system
spec:
  interval: 5m
  path: ./apps/production
  prune: true              # Delete resources removed from Git
  sourceRef:
    kind: GitRepository
    name: k8s-manifests
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: web-app
    namespace: production
  timeout: 5m
```

```yaml
# HelmRelease — manage Helm charts through Flux
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: web-app
  namespace: production
spec:
  interval: 10m
  chart:
    spec:
      chart: web-app
      version: ">=1.0.0 <2.0.0"
      sourceRef:
        kind: HelmRepository
        name: my-charts
        namespace: flux-system
  values:
    replicaCount: 5
    image:
      tag: v2.1.0
  upgrade:
    remediation:
      retries: 3
  rollback:
    timeout: 5m
```

---

### Q76 — How do you implement zero-downtime PostgreSQL operations in Kubernetes?

> 🎯 **Scenario:** Your PostgreSQL StatefulSet needs maintenance (failover, upgrade, resize) without application downtime.

**Answer:**

```yaml
# Use CloudNativePG operator — the production-grade PostgreSQL Operator
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: postgres-ha
  namespace: production
spec:
  instances: 3              # 1 primary + 2 replicas
  primaryUpdateStrategy: unsupervised   # Auto-failover
  postgresql:
    parameters:
      max_connections: "200"
      shared_buffers: "512MB"
      wal_level: logical
  bootstrap:
    initdb:
      database: myapp
      owner: appuser
      secret:
        name: app-db-credentials
  storage:
    storageClass: fast-ssd
    size: 100Gi
  backup:
    retentionPolicy: "30d"
    barmanObjectStore:
      destinationPath: s3://my-pg-backups
      s3Credentials:
        inheritFromIAMRole: true
  monitoring:
    enabled: true
  # Automatically failover within 30s if primary fails
  failoverDelay: 0
```

```bash
# CloudNativePG operations
kubectl cnpg status postgres-ha -n production
kubectl cnpg promote postgres-ha-2 -n production   # Manual failover to replica 2
kubectl cnpg backup postgres-ha -n production       # On-demand backup
kubectl cnpg restart postgres-ha -n production      # Rolling restart

# Check replication lag
kubectl exec -it postgres-ha-1 -n production -- \
  psql -c "SELECT * FROM pg_stat_replication;"
```

---

### Q77 — How do you implement Pod Disruption Budgets for production stability?

> 🎯 **Scenario:** During a cluster upgrade, Kubernetes drained 3 out of 4 pods of your API simultaneously, causing a brief outage.

**Answer:**

```yaml
# PodDisruptionBudget — protects against voluntary disruptions
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-api-pdb
  namespace: production
spec:
  # Choose ONE of:
  minAvailable: 3          # Always keep at least 3 pods running
  # maxUnavailable: 1      # Allow at most 1 pod down at a time
  # minAvailable: "75%"    # Keep at least 75% of pods running

  selector:
    matchLabels:
      app: web-api
```

```yaml
# PDB for StatefulSet (database cluster — at most 1 unavailable)
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: postgres-pdb
  namespace: production
spec:
  maxUnavailable: 1       # Only 1 DB pod can be down at a time
  selector:
    matchLabels:
      app: postgres
```

```bash
# Check PDB status
kubectl get pdb -n production
# NAME          MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
# web-api-pdb   3               N/A               1                     5d
# postgres-pdb  N/A             1                 1                     5d

# ALLOWED DISRUPTIONS shows how many pods can currently be voluntarily disrupted

# PDB prevents kubectl drain from draining too many pods:
kubectl drain node-1 --ignore-daemonsets
# error: Cannot evict pod as it would violate the pod's disruption budget
# ← This is expected! PDB is protecting the service
```

> ✅ **Always create PDBs before cluster upgrades.** Node drain respects PDBs — if you can't drain without violating a PDB, the drain blocks until conditions are met (e.g., a pod is rescheduled on another node first).

---

### Q78 — How do you handle secrets rotation with zero downtime using Vault?

> 🎯 **Scenario:** Your app needs dynamic database credentials from HashiCorp Vault, rotated every hour, without pod restarts.

**Answer:**

```bash
# Install Vault with HA in Kubernetes
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault \
  --namespace vault \
  --create-namespace \
  --set server.ha.enabled=true \
  --set server.ha.replicas=3 \
  --set server.auditStorage.enabled=true
```

```yaml
# Vault Agent Injector — sidecars that inject and renew secrets
# Annotate pods to get automatic secret injection

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  template:
    metadata:
      annotations:
        # Enable Vault Agent sidecar injection
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "web-app"
        # Inject DB credentials at /vault/secrets/db-creds
        vault.hashicorp.com/agent-inject-secret-db-creds: "database/creds/web-app"
        # Template to format the secret as a .env file
        vault.hashicorp.com/agent-inject-template-db-creds: |
          {{- with secret "database/creds/web-app" -}}
          export DB_USERNAME="{{ .Data.username }}"
          export DB_PASSWORD="{{ .Data.password }}"
          {{- end }}
        # Renew lease — app gets new creds before they expire
        vault.hashicorp.com/agent-pre-populate-only: "false"
    spec:
      serviceAccountName: web-app-sa  # Vault uses K8s SA for auth
      containers:
      - name: app
        image: myapp:v2.0
        command:
        - /bin/sh
        - -c
        - |
          source /vault/secrets/db-creds  # Load dynamic credentials
          exec python app.py
```

```bash
# Configure Vault database secrets engine
vault secrets enable database
vault write database/config/postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="web-app" \
  connection_url="postgresql://{{username}}:{{password}}@postgres:5432/mydb?sslmode=disable" \
  username="vault-admin" \
  password="vault-admin-password"

vault write database/roles/web-app \
  db_name=postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"
```

---

### Q79 — How do you implement resource quotas with priority classes?

> 🎯 **Scenario:** During a resource crunch, you want critical services to always get resources, and best-effort workloads to be evicted first.

**Answer:**

```yaml
# PriorityClass — defines scheduling and eviction priority
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-production
value: 1000000        # Higher value = higher priority
globalDefault: false
description: "Critical production services — never evict"
preemptionPolicy: PreemptLowerPriority   # Can preempt lower-priority pods
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: standard-production
value: 100000
globalDefault: true
description: "Standard production workloads"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: batch-workload
value: 1000
description: "Batch jobs — evict first during pressure"
preemptionPolicy: Never
```

```yaml
# Critical service uses high-priority class
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  template:
    spec:
      priorityClassName: critical-production   # High priority
      containers:
      - name: payment
        resources:
          requests:
            cpu: "1"
            memory: 1Gi
          limits:
            cpu: "1"       # Guaranteed QoS (requests == limits)
            memory: 1Gi
```

```yaml
# Batch job uses low-priority class
apiVersion: batch/v1
kind: Job
spec:
  template:
    spec:
      priorityClassName: batch-workload       # Low priority — evicted first
      containers:
      - name: batch
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
```

```yaml
# ResourceQuota per PriorityClass per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: critical-quota
  namespace: production
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
  scopeSelector:
    matchExpressions:
    - scopeName: PriorityClass
      operator: In
      values:
      - critical-production
```

---

### Q80 — How do you design a production-ready Kubernetes cluster checklist?

> 🎯 **Scenario:** You're launching a new production Kubernetes cluster. What does your production-readiness checklist include?

**Answer:**

```
☐ CLUSTER SETUP
  ☐ Multi-master control plane (3+ masters for HA)
  ☐ etcd with 3+ nodes, automated backups to S3 every 15 min
  ☐ Worker nodes across 3+ availability zones
  ☐ Cluster Autoscaler configured for node scaling
  ☐ Managed K8s (EKS/GKE/AKS) preferred over self-managed

☐ NETWORKING
  ☐ CNI plugin with NetworkPolicy support (Calico/Cilium)
  ☐ NGINX or AWS Load Balancer Ingress controller
  ☐ cert-manager for automatic TLS certificate management
  ☐ CoreDNS with anti-affinity (spread across nodes)
  ☐ Default-deny NetworkPolicies per namespace

☐ SECURITY
  ☐ RBAC configured — least privilege for all service accounts
  ☐ Pod Security Standards enforced (restricted level for production)
  ☐ Secrets encryption at rest (EncryptionConfiguration or KMS)
  ☐ External Secrets Operator with AWS Secrets Manager
  ☐ Image vulnerability scanning in CI (Trivy, Grype)
  ☐ OPA/Gatekeeper policies (resource limits required, allowed registries)
  ☐ Audit logging enabled with 30-day retention
  ☐ No hostNetwork, hostPID, privileged containers in production
  ☐ Container images built FROM non-root base images

☐ WORKLOADS
  ☐ All Deployments have resource requests AND limits
  ☐ All Deployments have liveness + readiness probes
  ☐ All Deployments have PodDisruptionBudgets (minAvailable: 2+)
  ☐ maxUnavailable: 0 in rolling update strategy
  ☐ terminationGracePeriodSeconds >= 30 + preStop hook
  ☐ Pods spread across AZs with topologySpreadConstraints
  ☐ No bare pods (use Deployment/StatefulSet/DaemonSet)

☐ STORAGE
  ☐ StorageClass with WaitForFirstConsumer binding mode
  ☐ allowVolumeExpansion: true on StorageClasses
  ☐ reclaimPolicy: Retain for production databases
  ☐ Regular PVC snapshots / application-level backups
  ☐ Backup restore tested regularly

☐ OBSERVABILITY
  ☐ Prometheus + Alertmanager + Grafana (kube-prometheus-stack)
  ☐ Loki or EFK for centralized log aggregation
  ☐ Jaeger or Tempo for distributed tracing
  ☐ Alerts: PodCrashLooping, HighErrorRate, NodeNotReady, DiskPressure
  ☐ Grafana dashboards for SLIs (latency, error rate, saturation)
  ☐ SLO tracking and error budget monitoring

☐ CI/CD
  ☐ GitOps (ArgoCD/Flux) — all changes via Git
  ☐ Manifest validation in CI (kubeval, kubeconform, Checkov)
  ☐ Image scanning before push
  ☐ Helm or Kustomize for environment-specific configs
  ☐ Automated rollback on health check failure

☐ CAPACITY & COST
  ☐ VPA in recommendation mode — review monthly
  ☐ HPA on all customer-facing deployments
  ☐ Spot instances for dev/staging/batch
  ☐ Kubecost or AWS Cost Explorer for K8s cost allocation
  ☐ ResourceQuotas per namespace to prevent runaway costs
  ☐ Descheduler for pod bin-packing
```

---

## 📋 Quick Reference Cheatsheet

### Pod Debugging

```bash
kubectl get pods -A -o wide                              # All pods + node/IP info
kubectl describe pod <pod> -n <ns>                       # Full details + events
kubectl logs <pod> --previous -c <container>             # Pre-crash logs
kubectl logs -l app=web --all-containers --prefix -f     # Tail multi-pod
kubectl exec -it <pod> -c <container> -- bash            # Shell into container
kubectl debug -it <pod> --image=nicolaka/netshoot        # Inject debug container
kubectl debug node/<node> -it --image=ubuntu:22.04       # Debug a node
kubectl port-forward pod/<pod> 8080:8080                 # Local port forwarding
kubectl cp <pod>:/path/file ./local                      # Copy from pod
kubectl get events --sort-by='.lastTimestamp' -n <ns>    # Recent events
kubectl get events --field-selector reason=Failed        # Only failures
```

### Deployments & Rollouts

```bash
kubectl apply -f manifest.yaml --dry-run=server          # Preview apply
kubectl diff -f manifest.yaml                            # Show pending changes
kubectl rollout status deployment/<name>                 # Watch rollout
kubectl rollout history deployment/<name>                # Revision history
kubectl rollout undo deployment/<name>                   # Rollback
kubectl rollout undo deployment/<name> --to-revision=3   # Specific revision
kubectl rollout restart deployment/<name>                # Force restart all pods
kubectl scale deployment/<name> --replicas=5             # Manual scale
kubectl set image deployment/<name> app=myapp:v2.0       # Update image
kubectl autoscale deployment/<name> --min=2 --max=10     # Quick HPA
```

### Nodes & Cluster

```bash
kubectl get nodes -o wide                                # Node status
kubectl describe node <node>                             # Node details + capacity
kubectl drain <node> --ignore-daemonsets \
  --delete-emptydir-data                                 # Drain for maintenance
kubectl cordon <node>                                    # Stop scheduling
kubectl uncordon <node>                                  # Resume scheduling
kubectl top nodes                                        # CPU/memory usage
kubectl top pods --containers --sort-by=memory           # Container metrics
kubectl get componentstatuses                            # Control plane health
kubectl cluster-info                                     # Cluster endpoints
```

### Resources & Config

```bash
kubectl get all -n <namespace>                           # All resources in ns
kubectl get pv,pvc -A                                    # Storage overview
kubectl get networkpolicy -A                             # All network policies
kubectl get ingress -A                                   # All ingresses
kubectl get hpa,vpa,scaledobject -A                      # All autoscalers
kubectl api-resources                                    # All resource types
kubectl explain deployment.spec.strategy               # Inline docs
kubectl get quota -A                                     # Resource quotas
kubectl get limitrange -A                                # Limit ranges
```

### Security & RBAC

```bash
kubectl auth can-i create pods -n prod --as=user@co.com  # Check permissions
kubectl auth can-i --list -n prod --as=user@co.com       # List all permissions
kubectl get rolebindings,clusterrolebindings -A          # All RBAC bindings
kubectl get serviceaccounts -A                           # All service accounts
kubectl get secrets -A --field-selector type=Opaque      # Opaque secrets
```

### State & Troubleshooting

```bash
kubectl get pods --field-selector=status.phase=Pending   # Pending pods
kubectl get pods --field-selector=status.phase=Failed    # Failed pods
KUBECONFIG=~/.kube/config kubectl config get-contexts    # List clusters
kubectl config use-context <context>                     # Switch cluster
kubectl config set-context --current --namespace=<ns>    # Set default ns
kubectl delete pod <pod> --grace-period=0 --force        # Force delete
TF_LOG=DEBUG kubectl apply -f file.yaml                  # Verbose kubectl
```

---

## 🎯 Interview Tips

| Topic | What Interviewers Want to Hear |
|-------|-------------------------------|
| **Architecture** | Control plane vs data plane, API server is the hub, etcd quorum |
| **Scheduling** | Filters → Scores → Binds; taints/tolerations; affinity |
| **Probes** | startup → liveness → readiness; consequences of each failing |
| **Services** | ClusterIP/NodePort/LoadBalancer; headless for StatefulSets |
| **Networking** | CNI; CoreDNS; NetworkPolicy requires compatible CNI |
| **Storage** | PV/PVC/StorageClass abstraction; WaitForFirstConsumer for multi-AZ |
| **Security** | RBAC least privilege; PSS; non-root; secrets encryption; IRSA |
| **Scaling** | HPA (stateless) + VPA (right-size) + KEDA (event-driven) + CA (nodes) |
| **GitOps** | ArgoCD/Flux; declarative; self-healing; drift detection |
| **Resilience** | PDBs; maxUnavailable=0; multi-AZ spread; graceful shutdown |
| **Observability** | Prometheus/Grafana metrics; Loki logs; Jaeger traces |
| **Troubleshooting** | describe → logs --previous → events → exec → network debug |
| **Cost** | Spot instances; scale-to-zero; right-sizing; Kubecost |
| **Upgrades** | Check deprecated APIs; PDBs first; control plane then nodes |

---

*Good luck with your Kubernetes interviews! ☸️*
