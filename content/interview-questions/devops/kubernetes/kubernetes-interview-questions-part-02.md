---
title: "Kubernetes Interview Questions & Answers (2026) Part 02"
description: "25+ Kubernetes interview questions and answers covering pods, deployments, services, networking, scaling, debugging, and advanced scenarios — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["kubernetes", "k8s", "interview", "devops", "containers"]
tool: "kubernetes"
level: "All Levels"
question_count: 60
draft: "false"
---

<div class="qa-list">

{{< qa num="1" q="What is Kubernetes and why do we need it?" level="basic" >}}
**Ans:**

**Kubernetes (K8s)** is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Why we need it:**
- **Docker alone** can't: auto-restart failed containers, scale automatically, load-balance traffic, or do rolling updates
- Kubernetes handles all of this and more

**Core capabilities:**

| Feature | What it does |
|---------|-------------|
| Self-healing | Auto-restarts failed containers |
| Auto-scaling | HPA scales pods based on CPU/memory |
| Load balancing | Distributes traffic across pods |
| Rolling updates | Zero-downtime deployments |
| Service discovery | Pods find each other by name |
| Storage orchestration | Dynamic PV provisioning |
| Secret management | Secure secret injection |

```bash
# Check cluster info
kubectl cluster-info

# Get all nodes
kubectl get nodes
```
{{< /qa >}}

{{< qa num="2" q="Explain pods, deployments, and services in Kubernetes." level="basic" >}}
**Ans:**

**Pod** — Smallest deployable unit. Contains one or more containers that share network/storage.
```yaml
# Basic pod
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx:1.25
    ports:
    - containerPort: 80
```

**Deployment** — Manages ReplicaSet + rolling updates + rollback for stateless apps.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: nginx:1.25
```

**Service** — Stable network endpoint for pods. Provides load balancing and service discovery.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```
{{< /qa >}}

{{< qa num="3" q="Difference between ReplicaSet, Deployment, and StatefulSet?" level="basic" >}}
**Ans:**

| Resource | Purpose | Pod Identity | Storage | Use Case |
|----------|---------|--------------|---------|---------|
| **ReplicaSet** | Maintains N replicas | Random names | Shared/none | Rarely used directly |
| **Deployment** | Manages RS + rolling updates | Random names (pod-xxx) | Ephemeral | Stateless apps (APIs, web servers) |
| **StatefulSet** | Ordered, stable pod names | Stable (pod-0, pod-1) | Per-pod PVC | Databases, Kafka, Zookeeper |

```bash
# Deployment pods:
my-deploy-7d4f8b-abc12
my-deploy-7d4f8b-xyz99

# StatefulSet pods (ordered, stable):
my-db-0
my-db-1
my-db-2
```

**StatefulSet guarantees:**
- Pods created in order (0, 1, 2...)
- Deleted in reverse order (2, 1, 0)
- Each pod gets its own PVC (data persists even if pod restarts)
{{< /qa >}}

{{< qa num="4" q="How does Kubernetes schedule a Pod? Walk through the control-plane process." level="intermediate" >}}
**Ans:**

```
kubectl apply -f pod.yaml
         │
         ▼
API Server (validates, stores in etcd)
         │
         ▼
etcd (persists pod spec with no nodeName)
         │
         ▼
Scheduler (watches for unscheduled pods)
    1. Filtering: Removes nodes that can't run pod
       (insufficient CPU/memory, taints, node selector)
    2. Scoring: Ranks remaining nodes
       (least requested resources, affinity, topology)
    3. Binding: Sets pod.spec.nodeName = best node
         │
         ▼
Kubelet on selected node (watches API for pods assigned to it)
    1. Pulls container image (if not cached)
    2. Creates containers via container runtime (containerd)
    3. Starts containers
    4. Reports status back to API Server
         │
         ▼
Pod Running (status updated in etcd via API Server)
```

```bash
# Watch scheduling events
kubectl describe pod my-pod
kubectl get events --sort-by='.lastTimestamp'
```
{{< /qa >}}

{{< qa num="5" q="Pod stuck in CrashLoopBackOff - how do you debug it?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check pod status and events
kubectl describe pod <pod-name> -n <namespace>
# Look at: Events section, Last State, Exit Code

# Step 2: Check logs (current and previous container)
kubectl logs <pod-name>
kubectl logs <pod-name> --previous    # Logs from crashed container

# Step 3: Check exit code
# Exit 0 = success (but CMD finished unexpectedly)
# Exit 1 = application error
# Exit 137 = OOMKilled
# Exit 139 = segfault

# Step 4: Get into the container temporarily
kubectl exec -it <pod-name> -- sh

# Step 5: If container exits too fast to exec, override CMD
kubectl run debug --image=myimage --command -- sleep 3600
kubectl exec -it debug -- sh

# Step 6: Check resource limits
kubectl describe pod <pod-name> | grep -A5 Limits

# Step 7: Check liveness probe config
kubectl describe pod <pod-name> | grep -A10 Liveness

# Common causes:
# - Wrong command/entrypoint
# - Missing env vars or config
# - OOMKilled (need more memory)
# - App crashes on startup
# - Liveness probe too aggressive
```
{{< /qa >}}

{{< qa num="6" q="What is Ingress and how do you use it?" level="intermediate" >}}
**Ans:**

**Ingress** is a Kubernetes resource that manages external HTTP/HTTPS access to services, providing routing, SSL termination, and name-based virtual hosting.

```
Internet → Ingress Controller (nginx/traefik) → Ingress Rules → Services → Pods
```

**Ingress vs Service:**
- `Service type: LoadBalancer` = one cloud LB per service (expensive)
- `Ingress` = one LB for all services with path/host routing

```yaml
# Ingress resource example
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
```

```bash
# Install nginx ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Check ingress
kubectl get ingress
kubectl describe ingress app-ingress
```
{{< /qa >}}

{{< qa num="7" q="Service not reachable inside the cluster in Kubernetes. How do you debug?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Verify service exists and has correct port
kubectl get service my-service
kubectl describe service my-service

# Step 2: Check endpoints (are pods registered?)
kubectl get endpoints my-service
# If NO endpoints → selector mismatch or pods not Running

# Step 3: Verify pod labels match service selector
kubectl get pods --show-labels
kubectl describe service my-service | grep Selector
# Both must match!

# Step 4: Test from within the cluster (using a debug pod)
kubectl run test --image=busybox --rm -it -- sh
wget -O- http://my-service:80
nslookup my-service

# Step 5: Test DNS resolution
nslookup my-service.default.svc.cluster.local

# Step 6: Check pod is Running and ready
kubectl get pods -l app=myapp
# STATUS must be Running, READY must be 1/1

# Step 7: Check NetworkPolicies blocking traffic
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <policy-name>

# Step 8: Test directly to pod IP (bypass service)
POD_IP=$(kubectl get pod <pod-name> -o jsonpath='{.status.podIP}')
kubectl run test --image=busybox --rm -it -- wget -O- http://$POD_IP:8080
```
{{< /qa >}}

{{< qa num="8" q="Liveness Probe vs Readiness Probe in Kubernetes." level="basic" >}}
**Ans:**

| Probe | Purpose | Action on failure |
|-------|---------|-------------------|
| **Liveness** | Is the container alive/healthy? | **Restarts** the container |
| **Readiness** | Is the container ready to receive traffic? | **Removes from Service** (stops sending traffic) |
| **Startup** | Is the container done starting up? | Delays liveness/readiness checks during slow starts |

```yaml
spec:
  containers:
  - name: app
    image: myapp
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15    # Wait before first check
      periodSeconds: 10           # Check every 10s
      failureThreshold: 3         # Restart after 3 failures

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3         # Remove from service after 3 failures

    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30        # Allow 5 minutes to start (30 * 10s)
      periodSeconds: 10
```
{{< /qa >}}

{{< qa num="9" q="What is HPA (Horizontal Pod Autoscaler) and how does it work?" level="intermediate" >}}
**Ans:**

**HPA** automatically scales the number of pods in a deployment based on CPU/memory/custom metrics.

```
HPA controller → queries metrics-server → compares with target
     │
     └── if CPU > threshold → increase replicas
     └── if CPU < threshold → decrease replicas (min replicas floor)
```

```yaml
# HPA manifest
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70   # Scale if CPU > 70%
```

```bash
# Check HPA status
kubectl get hpa
kubectl describe hpa my-hpa

# Quick create via CLI
kubectl autoscale deployment my-deployment --cpu-percent=70 --min=2 --max=10
```
{{< /qa >}}

{{< qa num="10" q="HPA is not scaling even though CPU is above threshold. How do you debug?" level="advanced" >}}
**Ans:**

```bash
# Step 1: Check HPA status
kubectl describe hpa my-hpa
# Look for: current metrics, conditions, events

# Step 2: Check if metrics-server is running
kubectl get pods -n kube-system | grep metrics-server
kubectl top pods    # If this fails → metrics-server issue

# Step 3: Check that pod has resource requests defined
kubectl describe pod <pod-name> | grep -A5 Requests
# HPA REQUIRES resource requests to calculate utilization %
# If requests not set → HPA can't calculate utilization → won't scale

# Step 4: Check if min/max replicas are hit
kubectl get hpa my-hpa
# If REPLICAS = MAXREPLICAS → HPA is at max, can't scale more

# Step 5: Check cooldown periods
# HPA has a stabilization window (default 5 min scale-down, 3 min scale-up)
# Check scaleUp.stabilizationWindowSeconds in HPA spec

# Step 6: Check API server for RBAC issues
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# Fix: Add resource requests to pod spec
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```
{{< /qa >}}

{{< qa num="11" q="What is kubeconfig and how does it work?" level="basic" >}}
**Ans:**

**kubeconfig** is a YAML file (`~/.kube/config` by default) that stores cluster connection details, credentials, and contexts for `kubectl`.

```yaml
# ~/.kube/config structure
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    server: https://my-k8s-api:6443
    certificate-authority-data: BASE64...
users:
- name: my-user
  user:
    client-certificate-data: BASE64...
    client-key-data: BASE64...
contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: my-user
    namespace: default
current-context: my-context
```

```bash
# View current config
kubectl config view

# List contexts
kubectl config get-contexts

# Switch context
kubectl config use-context my-cluster

# Set default namespace
kubectl config set-context --current --namespace=production

# Merge multiple kubeconfigs
KUBECONFIG=~/.kube/config:~/.kube/cluster2-config kubectl config view --merge --flatten > ~/.kube/merged-config
```
{{< /qa >}}

{{< qa num="12" q="How do you handle secrets in Kubernetes?" level="intermediate" >}}
**Ans:**

```bash
# Create secret
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=mysecret

# Create from file
kubectl create secret generic tls-certs \
  --from-file=tls.crt=./cert.crt \
  --from-file=tls.key=./cert.key

# View (base64 encoded)
kubectl get secret db-secret -o yaml
echo "bXlzZWNyZXQ=" | base64 -d   # decode
```

```yaml
# Use in pod - as environment variable
spec:
  containers:
  - name: app
    envFrom:
    - secretRef:
        name: db-secret

# Use as individual env var
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password

# Mount as file (more secure - not in process env)
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```

**Production best practice:** Use **External Secrets Operator** to sync from AWS Secrets Manager / Vault into K8s secrets.
{{< /qa >}}

{{< qa num="13" q="How do you fetch secrets from Azure Key Vault and use them in Kubernetes?" level="advanced" >}}
**Ans:**

**Using CSI Secrets Store Driver + Azure Key Vault Provider:**

```bash
# Step 1: Install CSI driver
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver

# Step 2: Install Azure Key Vault provider
helm repo add csi-secrets-store-provider-azure https://azure.github.io/secrets-store-csi-driver-provider-azure/charts
helm install azure-csi csi-secrets-store-provider-azure/csi-secrets-store-provider-azure
```

```yaml
# Step 3: Create SecretProviderClass
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-keyvault-secrets
spec:
  provider: azure
  parameters:
    vaultName: my-keyvault
    clientID: <managed-identity-client-id>
    tenantID: <azure-tenant-id>
    objects: |
      array:
        - |
          objectName: db-password
          objectType: secret

# Step 4: Mount in pod
spec:
  containers:
  - name: app
    volumeMounts:
    - name: secrets-store
      mountPath: /mnt/secrets-store
      readOnly: true
  volumes:
  - name: secrets-store
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: azure-keyvault-secrets
```
{{< /qa >}}

{{< qa num="14" q="How do you manage environment variables in Kubernetes?" level="basic" >}}
**Ans:**

```yaml
# Method 1: Direct env in pod spec
spec:
  containers:
  - name: app
    env:
    - name: ENV
      value: "production"
    - name: LOG_LEVEL
      value: "info"

# Method 2: From ConfigMap
# Create ConfigMap:
# kubectl create configmap app-config --from-literal=LOG_LEVEL=info

    envFrom:
    - configMapRef:
        name: app-config

# Method 3: From Secret
    envFrom:
    - secretRef:
        name: db-secret

# Method 4: From field reference (downward API)
    env:
    - name: MY_POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: MY_POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
```
{{< /qa >}}

{{< qa num="15" q="Default namespaces in Kubernetes." level="basic" >}}
**Ans:**

| Namespace | Purpose |
|-----------|---------|
| `default` | Where resources go if no namespace is specified |
| `kube-system` | Kubernetes system components (CoreDNS, kube-proxy, metrics-server, etc.) |
| `kube-public` | Publicly accessible data, readable by all users |
| `kube-node-lease` | Node heartbeat leases for leader election |

```bash
# List all namespaces
kubectl get namespaces

# View resources in kube-system
kubectl get pods -n kube-system

# Create a namespace
kubectl create namespace staging

# Set default namespace for your context
kubectl config set-context --current --namespace=staging
```
{{< /qa >}}

{{< qa num="16" q="What is ClusterIP vs NodePort vs LoadBalancer service type?" level="basic" >}}
**Ans:**

| Type | Access | Use Case |
|------|--------|---------|
| **ClusterIP** | Only inside cluster | Internal service-to-service communication |
| **NodePort** | `<NodeIP>:<30000-32767>` from outside | Development/testing |
| **LoadBalancer** | Cloud LB public IP | Production external access |
| **ExternalName** | DNS CNAME alias | Point to external DNS |

```yaml
# ClusterIP (default)
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080

# NodePort
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080    # Optional, auto-assigned if omitted

# LoadBalancer (creates cloud LB)
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

```bash
kubectl get service my-service
# EXTERNAL-IP column shows: <none> for ClusterIP, node IPs for NodePort, real IP for LB
```
{{< /qa >}}

{{< qa num="17" q="NodePort range in Kubernetes." level="basic" >}}
**Ans:**

The default NodePort range is **30000–32767**.

```bash
# Check configured range on API server
kube-apiserver --help | grep node-port-range
# or check /etc/kubernetes/manifests/kube-apiserver.yaml
grep -i nodeport /etc/kubernetes/manifests/kube-apiserver.yaml

# Custom range (set in kube-apiserver startup args):
--service-node-port-range=30000-32767
```

If you try to assign a port outside this range, the API server will reject it.
{{< /qa >}}

{{< qa num="18" q="Which Kubernetes resource is used to store tokens, secrets, and passwords?" level="basic" >}}
**Ans:**

**Secrets** — Kubernetes objects that store sensitive data like passwords, API tokens, TLS certificates.

Types of secrets:
- `Opaque` — Generic key-value data (most common)
- `kubernetes.io/service-account-token` — Service account JWT tokens
- `kubernetes.io/tls` — TLS certificate and key
- `kubernetes.io/dockerconfigjson` — Docker registry credentials

```bash
# List secrets
kubectl get secrets

# Create opaque secret
kubectl create secret generic my-secret \
  --from-literal=token=abc123

# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mypass

# View secret (base64 encoded)
kubectl get secret my-secret -o jsonpath='{.data.token}' | base64 -d
```

> **Note:** K8s Secrets are base64-encoded, NOT encrypted by default. Use `etcd` encryption at rest + Vault/AWS Secrets Manager for production.
{{< /qa >}}

{{< qa num="19" q="How do you implement zero-downtime deployments in Kubernetes?" level="intermediate" >}}
**Ans:**

```yaml
# Method 1: Rolling Update (default)
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0      # Never have 0 available pods
      maxSurge: 1            # Allow 1 extra pod during update

# Method 2: Add readiness probes (critical!)
  containers:
  - name: app
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
# K8s only routes traffic to pods that pass readiness probe
```

```bash
# Perform rolling update
kubectl set image deployment/my-deploy app=myimage:2.0

# Monitor rollout
kubectl rollout status deployment/my-deploy

# Rollback if issues
kubectl rollout undo deployment/my-deploy

# Method 3: Blue/Green deployment
# Deploy new version as separate deployment
# Switch service selector from app: blue → app: green

# Method 4: Canary with Ingress annotations
# Route 10% traffic to canary, 90% to stable
# nginx.ingress.kubernetes.io/canary: "true"
# nginx.ingress.kubernetes.io/canary-weight: "10"
```

**Also use PodDisruptionBudget to prevent too many pods being unavailable:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-pdb
spec:
  minAvailable: 2    # Always keep at least 2 pods
  selector:
    matchLabels:
      app: myapp
```
{{< /qa >}}

{{< qa num="20" q="How do you restrict communication between namespaces using NetworkPolicy?" level="advanced" >}}
**Ans:**

```yaml
# Default deny all ingress in 'production' namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}     # Applies to all pods
  policyTypes:
  - Ingress
  - Egress
---
# Allow specific namespace to reach production
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-staging
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: staging
    ports:
    - port: 8080
```

```bash
# Apply the policy
kubectl apply -f netpol.yaml

# Test: pod in staging can reach production
kubectl exec -it <staging-pod> -- curl http://myapp.production:8080

# Test: pod in default namespace cannot reach production
kubectl exec -it <default-pod> -- curl http://myapp.production:8080
# Should fail/timeout
```

> **Note:** NetworkPolicy requires a CNI plugin that supports it (Calico, Cilium, Weave). Flannel alone does NOT enforce NetworkPolicies.
{{< /qa >}}

{{< qa num="21" q="A deployment rollout caused a production outage. How would you roll back quickly?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Immediately roll back (fastest)
kubectl rollout undo deployment/my-deployment -n production

# Step 2: Verify rollback is in progress
kubectl rollout status deployment/my-deployment -n production

# Step 3: Confirm old pods are running
kubectl get pods -n production

# Step 4: Check rollout history
kubectl rollout history deployment/my-deployment

# Step 5: Roll back to a specific revision
kubectl rollout undo deployment/my-deployment --to-revision=3

# Step 6: If rollback is slow, force a faster restart
kubectl rollout restart deployment/my-deployment

# Step 7: Verify application is responding
kubectl get svc my-service
curl http://<service-ip>/health

# Prevention for next time:
# - Set minReadySeconds to delay traffic routing
# - Use readiness probes
# - Use PodDisruptionBudget
# - Add deployment annotation: kubernetes.io/change-cause: "version 2.1 - added feature X"
```
{{< /qa >}}

{{< qa num="22" q="Your CI/CD pipeline successfully builds and deploys, but Kubernetes keeps pulling the old image version. What could be causing this?" level="intermediate" >}}
**Ans:**

**Most common causes:**

```bash
# Cause 1: Using 'latest' tag (most common)
# K8s only pulls 'latest' if imagePullPolicy: Always
# Fix: Always use specific version tags
image: myapp:v1.2.3   # Never: myapp:latest in production

# Cause 2: imagePullPolicy not set correctly
# Default behavior:
# - :latest tag → Always pull
# - other tags → IfNotPresent (uses cached image!)
spec:
  containers:
  - name: app
    image: myapp:v1.2.3
    imagePullPolicy: Always   # Force pull on every start

# Cause 3: Image not pushed to registry before deploy
# Ensure CI pipeline order:
# 1. docker build
# 2. docker push ← must complete before step 3
# 3. kubectl set image

# Cause 4: Wrong image name/tag in deployment
kubectl describe deployment my-deployment | grep Image

# Cause 5: Old ReplicaSet running (deployment not updated)
kubectl get replicasets
kubectl rollout status deployment/my-deployment

# Cause 6: Cached image on node — delete pod to force fresh pull
kubectl delete pod <pod-name>   # New pod will pull fresh
# Or: kubectl rollout restart deployment/my-deployment
```
{{< /qa >}}

{{< qa num="23" q="New worker nodes are joining EKS cluster but pods remain stuck in Pending. What could be wrong?" level="advanced" >}}
**Ans:**

```bash
# Step 1: Describe the pending pod for reasons
kubectl describe pod <pod-name>
# Look in Events section for: Insufficient cpu, Insufficient memory, 
# node(s) had untolerated taint, no matching node selector

# Step 2: Check node resources
kubectl describe nodes
kubectl top nodes   # Requires metrics-server

# Step 3: Check if nodes joined correctly
kubectl get nodes
# STATUS must be Ready

# Step 4: Check node labels (if using nodeSelector)
kubectl get node <node-name> --show-labels
kubectl describe pod <pod-name> | grep "Node-Selectors"

# Step 5: Check taints (new nodes may have taints)
kubectl describe node <node-name> | grep -i taint
# If node has taint: node.kubernetes.io/not-ready → node still initializing

# Step 6: Check for missing tolerations in pod spec
kubectl describe pod <pod-name> | grep -i toleration

# Step 7: EKS-specific: Check node group IAM role permissions
# Node needs: AmazonEKSWorkerNodePolicy, AmazonEC2ContainerRegistryReadOnly

# Step 8: Check if EKS add-ons are healthy (VPC CNI, kube-proxy, CoreDNS)
kubectl get pods -n kube-system

# Step 9: Check if PVC is Pending (if pod requires PVC)
kubectl get pvc
# StorageClass must exist and dynamic provisioning must work
```
{{< /qa >}}

{{< qa num="24" q="InitContainer fails with restartPolicy: Never — what happens?" level="advanced" >}}
**Ans:**

If an **InitContainer** fails and the Pod's `restartPolicy` is `Never`, the pod **fails permanently** — it enters `Failed` state and is NOT restarted.

```yaml
# Example
spec:
  restartPolicy: Never
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'exit 1']   # Always fails
  containers:
  - name: app
    image: myapp
```

```bash
# Pod will show:
kubectl get pod my-pod
# STATUS: Init:Error → then → Failed

# Debug
kubectl describe pod my-pod
kubectl logs my-pod -c init-db

# With restartPolicy: OnFailure or Always:
# Pod would retry the initContainer
```

**Key rules:**
- InitContainers must all succeed before the main containers start
- They run sequentially (one at a time)
- If `restartPolicy: Always` → pod retries (CrashLoopBackOff after repeated failures)
- If `restartPolicy: Never` → pod fails permanently on first failure
{{< /qa >}}

{{< qa num="25" q="What is a DaemonSet and does it automatically tolerate master NoSchedule taints?" level="advanced" >}}
**Ans:**

A **DaemonSet** ensures one pod runs on every node (or a subset of nodes). Used for: log collectors (Fluentd), monitoring agents (Prometheus Node Exporter), network plugins.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```

**DaemonSet and master taints:**

Yes — DaemonSets automatically add tolerations for:
- `node.kubernetes.io/not-ready`
- `node.kubernetes.io/unschedulable`
- `node-role.kubernetes.io/control-plane:NoSchedule`
- `node-role.kubernetes.io/master:NoSchedule`

This means DaemonSet pods CAN run on control-plane nodes by default, unlike regular pods.

```bash
kubectl get daemonset -n kube-system
# kube-proxy, calico-node etc. run on all nodes including masters
```
{{< /qa >}}

</div>
