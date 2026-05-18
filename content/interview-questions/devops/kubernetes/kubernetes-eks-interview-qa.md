# Kubernetes & EKS Interview Questions & Answers for AWS Cloud and AWS DevOps Engineer
> A comprehensive guide covering Basic, Intermediate, and Advanced topics for Kubernetes and Amazon EKS interviews.

---

## 📚 Table of Contents

### 🟢 Basic Questions

#### Kubernetes Basics
1. [What is Kubernetes?](#1-what-is-kubernetes)
2. [What are the main components of the Kubernetes architecture?](#2-what-are-the-main-components-of-the-kubernetes-architecture)
3. [What is a Pod in Kubernetes?](#3-what-is-a-pod-in-kubernetes)
4. [What is a Node in Kubernetes?](#4-what-is-a-node-in-kubernetes)
5. [What is a Namespace in Kubernetes?](#5-what-is-a-namespace-in-kubernetes)
6. [What is a Deployment in Kubernetes?](#6-what-is-a-deployment-in-kubernetes)
7. [What is a ReplicaSet?](#7-what-is-a-replicaset)
8. [What is a Service in Kubernetes?](#8-what-is-a-service-in-kubernetes)
9. [What are the types of Kubernetes Services?](#9-what-are-the-types-of-kubernetes-services)
10. [What is a ConfigMap?](#10-what-is-a-configmap)
11. [What is a Secret in Kubernetes?](#11-what-is-a-secret-in-kubernetes)
12. [What is kubectl?](#12-what-is-kubectl)
13. [What is a DaemonSet?](#13-what-is-a-daemonset)
14. [What is a StatefulSet?](#14-what-is-a-statefulset)
15. [What is a Job and a CronJob in Kubernetes?](#15-what-is-a-job-and-a-cronjob-in-kubernetes)

#### EKS Basics
16. [What is Amazon EKS?](#16-what-is-amazon-eks)
17. [What is the difference between EKS and self-managed Kubernetes?](#17-what-is-the-difference-between-eks-and-self-managed-kubernetes)
18. [What are EKS Node Groups?](#18-what-are-eks-node-groups)
19. [What is AWS Fargate in the context of EKS?](#19-what-is-aws-fargate-in-the-context-of-eks)
20. [How do you authenticate to an EKS cluster?](#20-how-do-you-authenticate-to-an-eks-cluster)

---

### 🟡 Intermediate Questions

#### Kubernetes Intermediate
21. [What is the difference between a Deployment and a StatefulSet?](#21-what-is-the-difference-between-a-deployment-and-a-statefulset)
22. [What is a PersistentVolume (PV) and PersistentVolumeClaim (PVC)?](#22-what-is-a-persistentvolume-pv-and-persistentvolumeclaim-pvc)
23. [What is a StorageClass in Kubernetes?](#23-what-is-a-storageclass-in-kubernetes)
24. [How does Kubernetes handle rolling updates?](#24-how-does-kubernetes-handle-rolling-updates)
25. [What is a Liveness Probe and Readiness Probe?](#25-what-is-a-liveness-probe-and-readiness-probe)
26. [What is a Horizontal Pod Autoscaler (HPA)?](#26-what-is-a-horizontal-pod-autoscaler-hpa)
27. [What is the Kubernetes Scheduler and how does it work?](#27-what-is-the-kubernetes-scheduler-and-how-does-it-work)
28. [What are Taints and Tolerations?](#28-what-are-taints-and-tolerations)
29. [What are Node Affinity and Pod Affinity?](#29-what-are-node-affinity-and-pod-affinity)
30. [What is RBAC in Kubernetes?](#30-what-is-rbac-in-kubernetes)
31. [What is an Ingress Controller in Kubernetes?](#31-what-is-an-ingress-controller-in-kubernetes)
32. [What is etcd and what role does it play?](#32-what-is-etcd-and-what-role-does-it-play)
33. [What is a Kubernetes Operator?](#33-what-is-a-kubernetes-operator)
34. [What is the difference between a ClusterRole and a Role?](#34-what-is-the-difference-between-a-clusterrole-and-a-role)
35. [How does Kubernetes DNS work?](#35-how-does-kubernetes-dns-work)

#### EKS Intermediate
36. [What is the EKS Control Plane and how is it managed?](#36-what-is-the-eks-control-plane-and-how-is-it-managed)
37. [How does IAM integrate with EKS?](#37-how-does-iam-integrate-with-eks)
38. [What is the aws-auth ConfigMap?](#38-what-is-the-aws-auth-configmap)
39. [What is the Amazon VPC CNI plugin?](#39-what-is-the-amazon-vpc-cni-plugin)
40. [How do you scale nodes in EKS?](#40-how-do-you-scale-nodes-in-eks)
41. [What is Karpenter in EKS?](#41-what-is-karpenter-in-eks)
42. [What is EKS Anywhere?](#42-what-is-eks-anywhere)
43. [How does EKS integrate with AWS Load Balancer Controller?](#43-how-does-eks-integrate-with-aws-load-balancer-controller)
44. [What are EKS Add-ons?](#44-what-are-eks-add-ons)
45. [How do you manage secrets in EKS?](#45-how-do-you-manage-secrets-in-eks)

---

### 🔴 Advanced Questions

#### Kubernetes Advanced
46. [How does the Kubernetes networking model work?](#46-how-does-the-kubernetes-networking-model-work)
47. [What is a Custom Resource Definition (CRD)?](#47-what-is-a-custom-resource-definition-crd)
48. [What is a Vertical Pod Autoscaler (VPA)?](#48-what-is-a-vertical-pod-autoscaler-vpa)
49. [What is a Pod Disruption Budget (PDB)?](#49-what-is-a-pod-disruption-budget-pdb)
50. [How does Kubernetes implement service discovery?](#50-how-does-kubernetes-implement-service-discovery)
51. [What is a Service Mesh and how does Istio work with Kubernetes?](#51-what-is-a-service-mesh-and-how-does-istio-work-with-kubernetes)
52. [What is the Kubernetes API server admission controller?](#52-what-is-the-kubernetes-api-server-admission-controller)
53. [What are Kubernetes Network Policies?](#53-what-are-kubernetes-network-policies)
54. [How does Kubernetes handle multi-tenancy?](#54-how-does-kubernetes-handle-multi-tenancy)
55. [What is the difference between kube-proxy modes (iptables vs IPVS)?](#55-what-is-the-difference-between-kube-proxy-modes-iptables-vs-ipvs)
56. [How does Kubernetes garbage collection work?](#56-how-does-kubernetes-garbage-collection-work)
57. [What is the Cluster Autoscaler and how does it work?](#57-what-is-the-cluster-autoscaler-and-how-does-it-work)
58. [What is GitOps and how is it implemented with Kubernetes?](#58-what-is-gitops-and-how-is-it-implemented-with-kubernetes)
59. [How do you troubleshoot a CrashLoopBackOff error?](#59-how-do-you-troubleshoot-a-crashloopbackoff-error)
60. [What is OPA Gatekeeper and how is it used in Kubernetes?](#60-what-is-opa-gatekeeper-and-how-is-it-used-in-kubernetes)

#### EKS Advanced
61. [How does EKS use IRSA (IAM Roles for Service Accounts)?](#61-how-does-eks-use-irsa-iam-roles-for-service-accounts)
62. [What is EKS Pod Identity and how does it differ from IRSA?](#62-what-is-eks-pod-identity-and-how-does-it-differ-from-irsa)
63. [How do you implement multi-cluster EKS architectures?](#63-how-do-you-implement-multi-cluster-eks-architectures)
64. [How do you optimize costs in an EKS environment?](#64-how-do-you-optimize-costs-in-an-eks-environment)
65. [What is the EKS Distro (EKS-D)?](#65-what-is-the-eks-distro-eks-d)
66. [How do you implement blue/green deployments in EKS?](#66-how-do-you-implement-bluegreen-deployments-in-eks)
67. [How does EKS handle etcd backups and disaster recovery?](#67-how-does-eks-handle-etcd-backups-and-disaster-recovery)
68. [What are EKS security best practices?](#68-what-are-eks-security-best-practices)
69. [How do you monitor and observe an EKS cluster?](#69-how-do-you-monitor-and-observe-an-eks-cluster)
70. [How do you upgrade an EKS cluster with zero downtime?](#70-how-do-you-upgrade-an-eks-cluster-with-zero-downtime)

---

## 🟢 Basic Questions

---

### 1. What is Kubernetes?

**Answer:**

Kubernetes (K8s) is an open-source container orchestration platform originally developed by Google and now maintained by the CNCF (Cloud Native Computing Foundation). It automates the deployment, scaling, and management of containerized applications.

**Key capabilities:**
- **Automated rollouts and rollbacks** — deploy changes and roll them back if something goes wrong
- **Service discovery and load balancing** — expose containers using DNS names or IP addresses
- **Storage orchestration** — automatically mount storage systems (local, cloud, NFS, etc.)
- **Self-healing** — restarts failed containers, replaces containers, kills containers that don't respond to health checks
- **Secret and config management** — manage sensitive information without rebuilding images
- **Horizontal scaling** — scale applications up or down with a command or automatically

```bash
# Check Kubernetes version
kubectl version --short

# Get cluster info
kubectl cluster-info
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 2. What are the main components of the Kubernetes architecture?

**Answer:**

Kubernetes follows a master-worker architecture:

**Control Plane (Master) Components:**

| Component | Role |
|-----------|------|
| **API Server** (`kube-apiserver`) | Frontend for Kubernetes control plane; all communication goes through it |
| **etcd** | Consistent and highly-available key-value store for all cluster data |
| **Scheduler** (`kube-scheduler`) | Watches for newly created Pods and assigns them to nodes |
| **Controller Manager** (`kube-controller-manager`) | Runs controller processes (Node, Replication, Endpoint controllers, etc.) |
| **Cloud Controller Manager** | Links cluster to cloud provider APIs |

**Worker Node Components:**

| Component | Role |
|-----------|------|
| **kubelet** | Agent on each node that ensures containers are running in a Pod |
| **kube-proxy** | Maintains network rules on nodes for Pod communication |
| **Container Runtime** | Software to run containers (containerd, CRI-O, Docker) |

```
┌──────────────────────────────────────────────┐
│              Control Plane                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │API Server│ │Scheduler │ │  Controller  │  │
│  └──────────┘ └──────────┘ │  Manager     │  │
│        │                   └──────────────┘  │
│  ┌─────▼──────────────────────────────────┐  │
│  │                  etcd                  │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
           │
┌──────────▼───────────┐
│     Worker Node      │
│  ┌────────────────┐  │
│  │    kubelet     │  │
│  ├────────────────┤  │
│  │   kube-proxy   │  │
│  ├────────────────┤  │
│  │Container Runtime│ │
│  └────────────────┘  │
└──────────────────────┘
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 3. What is a Pod in Kubernetes?

**Answer:**

A **Pod** is the smallest and most basic deployable unit in Kubernetes. It represents a single instance of a running process and can contain one or more tightly coupled containers that share the same network namespace, IP address, and storage.

**Key characteristics:**
- Each Pod gets a unique IP address within the cluster
- Containers inside a Pod share `localhost` networking
- Pods are ephemeral — they are not self-healing by themselves
- They are typically managed by higher-level controllers (Deployments, StatefulSets)

```yaml
# Example Pod manifest
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app
spec:
  containers:
  - name: app-container
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

```bash
# Get all pods in all namespaces
kubectl get pods -A

# Describe a pod
kubectl describe pod my-app-pod

# Get pod logs
kubectl logs my-app-pod
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 4. What is a Node in Kubernetes?

**Answer:**

A **Node** is a physical or virtual machine that runs workloads (Pods) in a Kubernetes cluster. Every node is managed by the control plane and contains the services needed to run Pods.

**Node components:**
- `kubelet` — communicates with the API server and manages Pod lifecycle
- `kube-proxy` — handles networking rules for Service routing
- **Container runtime** — runs the containers (e.g., containerd)

**Node types:**
- **Master Node** — runs control plane components (in older setups)
- **Worker Node** — runs application workloads

```bash
# List all nodes
kubectl get nodes

# Get node details
kubectl describe node <node-name>

# Check node resource usage
kubectl top nodes
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 5. What is a Namespace in Kubernetes?

**Answer:**

**Namespaces** provide a mechanism to isolate groups of resources within a single cluster. They are ideal for multi-team or multi-project environments where resource quotas and access control need to be separated.

**Default namespaces:**
- `default` — for objects with no other namespace
- `kube-system` — for system objects created by Kubernetes
- `kube-public` — readable by all users; used for public cluster info
- `kube-node-lease` — holds Lease objects for node heartbeats

```bash
# List namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace my-team

# Get pods in a specific namespace
kubectl get pods -n my-team

# Set default namespace for kubectl
kubectl config set-context --current --namespace=my-team
```

```yaml
# ResourceQuota for a namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: my-team
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 6. What is a Deployment in Kubernetes?

**Answer:**

A **Deployment** is a higher-level abstraction that manages a ReplicaSet and provides declarative updates for Pods. It ensures the desired number of Pod replicas are running and handles rollouts and rollbacks.

**Features:**
- Declarative updates — you describe the desired state
- Rolling updates — updates Pods gradually to avoid downtime
- Rollback — easily revert to previous versions
- Pause and resume deployments

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:v2
        ports:
        - containerPort: 8080
```

```bash
# Create deployment
kubectl apply -f deployment.yaml

# Check rollout status
kubectl rollout status deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# View rollout history
kubectl rollout history deployment/my-app
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 7. What is a ReplicaSet?

**Answer:**

A **ReplicaSet** ensures that a specified number of Pod replicas are running at any given time. It replaces Pods that fail, are deleted, or are terminated. Deployments manage ReplicaSets, so in practice you rarely create a ReplicaSet directly.

**How it works:**
1. ReplicaSet defines a label selector to identify which Pods it manages
2. It continuously monitors Pod counts against the desired count
3. It creates or deletes Pods to match the desired state

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
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
        image: nginx
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 8. What is a Service in Kubernetes?

**Answer:**

A **Service** is an abstraction that defines a logical set of Pods and a policy to access them. Since Pods are ephemeral and their IPs change, a Service provides a stable IP address and DNS name to access them.

**How it works:**
- Services use **label selectors** to find matching Pods
- `kube-proxy` maintains network rules to route traffic
- Each Service gets a ClusterIP and a DNS entry (e.g., `my-service.default.svc.cluster.local`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 9. What are the types of Kubernetes Services?

**Answer:**

| Type | Description | Use Case |
|------|-------------|----------|
| **ClusterIP** | Default; exposes Service on internal cluster IP | Internal microservice communication |
| **NodePort** | Exposes Service on each Node's IP at a static port (30000-32767) | External access in development |
| **LoadBalancer** | Exposes Service externally using a cloud load balancer | Production external access |
| **ExternalName** | Maps Service to a DNS name (e.g., external DB) | Connecting to external services |

```yaml
# LoadBalancer Service Example (EKS)
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 10. What is a ConfigMap?

**Answer:**

A **ConfigMap** stores non-confidential configuration data as key-value pairs, decoupling configuration from container images. This allows you to change application behavior without rebuilding images.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_PORT: "8080"
  config.json: |
    {
      "logLevel": "info",
      "retries": 3
    }
```

**Using ConfigMap in a Pod:**

```yaml
spec:
  containers:
  - name: app
    image: my-app
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 11. What is a Secret in Kubernetes?

**Answer:**

A **Secret** stores sensitive data such as passwords, tokens, and SSH keys. Data is stored base64-encoded (not encrypted by default, but can be encrypted at rest with KMS).

```bash
# Create a secret from literal values
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=s3cr3t
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=      # base64("admin")
  password: czNjcjN0      # base64("s3cr3t")
```

```yaml
# Using secrets as environment variables
spec:
  containers:
  - name: app
    image: my-app
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 12. What is kubectl?

**Answer:**

`kubectl` is the command-line tool for interacting with Kubernetes clusters. It communicates with the Kubernetes API server to create, update, delete, and inspect resources.

**Common commands:**

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Pod management
kubectl get pods -n <namespace>
kubectl describe pod <pod-name>
kubectl logs <pod-name> -f
kubectl exec -it <pod-name> -- /bin/bash

# Apply/delete manifests
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml

# Scale deployment
kubectl scale deployment my-app --replicas=5

# Port forwarding
kubectl port-forward pod/my-pod 8080:80

# Resource usage
kubectl top pods
kubectl top nodes
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 13. What is a DaemonSet?

**Answer:**

A **DaemonSet** ensures that a copy of a Pod runs on all (or some selected) nodes. When a new node joins the cluster, the DaemonSet controller automatically adds a Pod to it.

**Common use cases:**
- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter, Datadog)
- Network plugins (Calico, Cilium)
- Storage daemons

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      name: node-exporter
  template:
    metadata:
      labels:
        name: node-exporter
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 14. What is a StatefulSet?

**Answer:**

A **StatefulSet** manages stateful applications that require stable, unique network identifiers, persistent storage, and ordered deployment/scaling/deletion. Unlike Deployments, StatefulSets give each Pod a unique, stable hostname.

**When to use:**
- Databases (MySQL, PostgreSQL, Cassandra)
- Message brokers (Kafka, RabbitMQ)
- Any app needing stable network identity

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
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
        image: postgres:14
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

Pods are named with ordinal suffixes: `postgres-0`, `postgres-1`, `postgres-2`.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 15. What is a Job and a CronJob in Kubernetes?

**Answer:**

**Job:** Creates one or more Pods and ensures they complete successfully. It is used for batch processing or one-time tasks.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
spec:
  completions: 1
  parallelism: 1
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: migrate
        image: my-migration-tool
        command: ["./migrate.sh"]
```

**CronJob:** Creates Jobs on a recurring schedule using cron syntax.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"   # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: my-backup-tool
            command: ["./backup.sh"]
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 16. What is Amazon EKS?

**Answer:**

**Amazon Elastic Kubernetes Service (EKS)** is a fully managed Kubernetes service from AWS that simplifies running Kubernetes by handling the control plane infrastructure, upgrades, patches, and high availability.

**Key benefits:**
- AWS manages the Kubernetes control plane across multiple Availability Zones
- Certified Kubernetes conformant — compatible with standard K8s tooling
- Integrates natively with AWS services (IAM, VPC, ALB, ECR, CloudWatch)
- Supports EC2 nodes, Fargate, and EKS Anywhere
- Automated Kubernetes version upgrades

```bash
# Create EKS cluster using eksctl
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-nodes \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 5 \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 17. What is the difference between EKS and self-managed Kubernetes?

**Answer:**

| Feature | EKS | Self-Managed Kubernetes |
|---------|-----|------------------------|
| **Control Plane** | Fully managed by AWS | You manage it |
| **Upgrades** | Simplified with one-click | Manual and complex |
| **HA** | Multi-AZ by default | Must configure manually |
| **etcd backup** | Managed by AWS | Your responsibility |
| **Cost** | $0.10/hour per cluster + node costs | Only node costs |
| **AWS Integration** | Native (IAM, VPC, ALB) | Manual configuration |
| **Flexibility** | Less control over control plane | Full control |

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 18. What are EKS Node Groups?

**Answer:**

**Node Groups** in EKS are collections of EC2 instances (worker nodes) that share the same configuration. There are two types:

**Managed Node Groups:**
- AWS provisions, registers, and terminates nodes automatically
- Supports EC2 Auto Scaling Groups
- Handles AMI updates and node lifecycle
- Easier to maintain; recommended for most cases

**Self-Managed Node Groups:**
- You manage the EC2 instances manually
- More control over AMIs and configurations
- Required for specialized hardware (GPU, custom kernel)

```bash
# Create managed node group
eksctl create nodegroup \
  --cluster my-cluster \
  --name gpu-nodes \
  --node-type p3.2xlarge \
  --nodes 2 \
  --managed

# List node groups
eksctl get nodegroup --cluster my-cluster
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 19. What is AWS Fargate in the context of EKS?

**Answer:**

**AWS Fargate** is a serverless compute engine for containers. With EKS + Fargate, you can run Pods without provisioning or managing EC2 instances — AWS manages the underlying infrastructure automatically.

**Key points:**
- No node management; pay per Pod CPU/memory
- Uses **Fargate Profiles** to define which Pods run on Fargate
- Each Pod gets its own isolated micro-VM (enhanced security)
- DaemonSets are NOT supported on Fargate

```yaml
# Fargate Profile via eksctl
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: my-cluster
  region: us-east-1
fargateProfiles:
- name: default
  selectors:
  - namespace: default
  - namespace: kube-system
```

```bash
# Create fargate profile
eksctl create fargateprofile \
  --cluster my-cluster \
  --name my-profile \
  --namespace my-namespace
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 20. How do you authenticate to an EKS cluster?

**Answer:**

EKS uses **AWS IAM** for authentication and **Kubernetes RBAC** for authorization.

**Authentication flow:**
1. `kubectl` calls the AWS CLI/SDK to get a pre-signed token via STS
2. The token is passed to the Kubernetes API server
3. The EKS cluster verifies the token with AWS IAM
4. Kubernetes RBAC is checked for authorization

```bash
# Configure kubectl for EKS
aws eks update-kubeconfig \
  --name my-cluster \
  --region us-east-1 \
  --role-arn arn:aws:iam::123456789:role/eks-admin-role

# Verify access
kubectl auth can-i get pods

# Check kubeconfig
kubectl config view
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

## 🟡 Intermediate Questions

---

### 21. What is the difference between a Deployment and a StatefulSet?

**Answer:**

| Feature | Deployment | StatefulSet |
|---------|------------|-------------|
| **Pod identity** | Interchangeable (random names) | Stable, unique (pod-0, pod-1) |
| **Storage** | Shared or ephemeral | Dedicated PVC per Pod |
| **Scaling** | Any order | Ordered (0, 1, 2...) |
| **DNS** | Single service endpoint | Individual headless DNS per Pod |
| **Use case** | Stateless apps | Stateful apps (DBs, queues) |
| **Rolling update** | Parallel or rolling | Sequential (n-1 to 0) |

```bash
# StatefulSet pod DNS format:
# <pod-name>.<service-name>.<namespace>.svc.cluster.local
# Example: postgres-0.postgres.default.svc.cluster.local
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 22. What is a PersistentVolume (PV) and PersistentVolumeClaim (PVC)?

**Answer:**

**PersistentVolume (PV):** A piece of storage in the cluster provisioned by an admin or dynamically via StorageClass. It exists independently of Pods.

**PersistentVolumeClaim (PVC):** A request for storage by a user. It binds to a PV that matches its requirements (size, access mode, StorageClass).

**Access Modes:**
- `ReadWriteOnce (RWO)` — can be mounted by a single node
- `ReadOnlyMany (ROX)` — can be mounted by many nodes in read-only mode
- `ReadWriteMany (RWX)` — can be mounted by many nodes in read/write mode

```yaml
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: gp2
```

```yaml
# Using PVC in a Pod
spec:
  containers:
  - name: app
    volumeMounts:
    - mountPath: /data
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 23. What is a StorageClass in Kubernetes?

**Answer:**

A **StorageClass** defines the "class" of storage (e.g., SSD, HDD, network storage) and enables dynamic provisioning of PersistentVolumes when a PVC is created.

```yaml
# EKS GP3 StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

**Reclaim policies:**
- `Delete` — automatically delete PV when PVC is deleted
- `Retain` — keep PV after PVC deletion for manual reclamation

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 24. How does Kubernetes handle rolling updates?

**Answer:**

A **rolling update** gradually replaces old Pod instances with new ones to ensure zero downtime.

**Parameters:**
- `maxSurge` — max Pods above desired count during update
- `maxUnavailable` — max Pods that can be unavailable during update

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Allow 1 extra pod
      maxUnavailable: 0    # No downtime; always keep all pods running
```

**Process:**
1. Create a new ReplicaSet with the updated Pod template
2. Scale up new ReplicaSet by `maxSurge` Pods
3. Scale down old ReplicaSet by `maxUnavailable` Pods
4. Repeat until all Pods are updated

```bash
# Update image (triggers rolling update)
kubectl set image deployment/my-app my-app=my-app:v2

# Monitor rollout
kubectl rollout status deployment/my-app

# Pause and resume
kubectl rollout pause deployment/my-app
kubectl rollout resume deployment/my-app
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 25. What is a Liveness Probe and Readiness Probe?

**Answer:**

**Liveness Probe:** Determines if a container is running. If it fails, the kubelet restarts the container.

**Readiness Probe:** Determines if a container is ready to accept traffic. If it fails, the Pod is removed from Service endpoints (no traffic sent to it).

**Startup Probe:** Checks if an application has started. Useful for slow-starting containers; disables liveness/readiness until startup succeeds.

```yaml
spec:
  containers:
  - name: app
    image: my-app
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

**Probe types:** `httpGet`, `tcpSocket`, `exec` (command)

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 26. What is a Horizontal Pod Autoscaler (HPA)?

**Answer:**

**HPA** automatically scales the number of Pods in a Deployment or StatefulSet based on observed CPU/memory utilization or custom metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

```bash
# Create HPA
kubectl autoscale deployment my-app --cpu-percent=70 --min=2 --max=10

# Check HPA status
kubectl get hpa
```

**Note:** HPA requires the `metrics-server` to be installed in the cluster.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 27. What is the Kubernetes Scheduler and how does it work?

**Answer:**

The **kube-scheduler** is a control plane component that watches for newly created Pods with no assigned node and selects a node for them to run on.

**Scheduling process (two phases):**

1. **Filtering (Predicates):** Finds nodes that are feasible for the Pod (resource availability, taints/tolerations, node selectors, etc.)
2. **Scoring (Priorities):** Ranks feasible nodes based on scoring functions (resource balance, affinity rules, etc.)

**Factors considered:**
- Resource requests and limits
- Node labels and selectors
- Taints and tolerations
- Affinity and anti-affinity rules
- Pod topology spread constraints
- Node pressure (DiskPressure, MemoryPressure)

```yaml
# Force Pod to a specific node
spec:
  nodeSelector:
    kubernetes.io/hostname: ip-192-168-1-100

# Or use nodeName directly
spec:
  nodeName: ip-192-168-1-100
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 28. What are Taints and Tolerations?

**Answer:**

**Taints** are applied to nodes to repel Pods that don't explicitly tolerate the taint. **Tolerations** are applied to Pods to allow them to be scheduled on tainted nodes.

**Taint effects:**
- `NoSchedule` — Pods without toleration are not scheduled on the node
- `PreferNoSchedule` — Kubernetes tries to avoid scheduling Pods without toleration
- `NoExecute` — Existing Pods without toleration are evicted

```bash
# Add a taint to a node
kubectl taint nodes node1 dedicated=gpu:NoSchedule

# Remove a taint
kubectl taint nodes node1 dedicated=gpu:NoSchedule-
```

```yaml
# Pod with toleration
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

**Use cases:**
- Dedicated nodes for specific workloads (GPU nodes, prod nodes)
- Preventing non-system Pods on control plane nodes

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 29. What are Node Affinity and Pod Affinity?

**Answer:**

**Node Affinity:** Constrains which nodes a Pod can be scheduled on based on node labels. More expressive than `nodeSelector`.

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:  # hard rule
        nodeSelectorTerms:
        - matchExpressions:
          - key: instance-type
            operator: In
            values: ["m5.xlarge", "m5.2xlarge"]
      preferredDuringSchedulingIgnoredDuringExecution:  # soft rule
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values: ["us-east-1a"]
```

**Pod Affinity / Anti-Affinity:** Schedules Pods relative to other Pods.

```yaml
# Anti-affinity: spread pods across nodes
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: my-app
        topologyKey: kubernetes.io/hostname
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 30. What is RBAC in Kubernetes?

**Answer:**

**Role-Based Access Control (RBAC)** regulates access to Kubernetes resources based on the roles of users or service accounts.

**Key objects:**

| Object | Scope | Purpose |
|--------|-------|---------|
| `Role` | Namespace | Grants permissions within a namespace |
| `ClusterRole` | Cluster-wide | Grants permissions across all namespaces |
| `RoleBinding` | Namespace | Binds Role to user/group/service account |
| `ClusterRoleBinding` | Cluster-wide | Binds ClusterRole cluster-wide |

```yaml
# Role — allows reading pods in default namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 31. What is an Ingress Controller in Kubernetes?

**Answer:**

An **Ingress** resource defines HTTP/HTTPS routing rules to Services. An **Ingress Controller** implements those rules (e.g., NGINX, Traefik, AWS ALB Ingress Controller).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
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
```

**In EKS**, the AWS Load Balancer Controller creates ALBs automatically from Ingress resources using annotations.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 32. What is etcd and what role does it play?

**Answer:**

**etcd** is a distributed, consistent key-value store used as Kubernetes' backing store for all cluster data. Every API object (Pods, Services, ConfigMaps, Secrets, etc.) is stored in etcd.

**Key properties:**
- **Consistency:** Uses Raft consensus algorithm for leader election and data replication
- **High availability:** Typically run as a 3 or 5-node cluster (odd number for quorum)
- **Watch API:** Enables Kubernetes controllers to watch for changes

**Important facts:**
- All communication with etcd goes through the API server
- Backing up etcd is critical for disaster recovery
- In EKS, etcd is fully managed by AWS

```bash
# In a self-managed cluster — backup etcd
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 33. What is a Kubernetes Operator?

**Answer:**

A **Kubernetes Operator** is a method of packaging, deploying, and managing a Kubernetes application using custom controllers and CRDs. Operators encode operational knowledge (how to deploy, scale, upgrade, backup) into software.

**Operator pattern:**
1. Define a **CRD** (e.g., `PostgreSQLCluster`)
2. Implement a **Controller** that watches the CRD
3. Controller reconciles the actual state with the desired state

**Popular Operators:**
- Prometheus Operator
- PostgreSQL Operator (Zalando or CrunchyData)
- Cert-Manager
- ArgoCD

```yaml
# Example: Using the Prometheus Operator CRD
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
spec:
  replicas: 2
  retention: 30d
  storage:
    volumeClaimTemplate:
      spec:
        resources:
          requests:
            storage: 50Gi
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 34. What is the difference between a ClusterRole and a Role?

**Answer:**

| Feature | Role | ClusterRole |
|---------|------|-------------|
| **Scope** | Single namespace | Cluster-wide |
| **Use case** | Namespace-scoped resources | Cluster-scoped resources (Nodes, PVs) or all namespaces |
| **Bound by** | RoleBinding | ClusterRoleBinding (or RoleBinding for namespace scope) |

A **ClusterRole** can be bound in two ways:
1. **ClusterRoleBinding** → grants access across all namespaces
2. **RoleBinding** → grants ClusterRole access within a specific namespace only

This is useful when you want to reuse a ClusterRole definition across multiple namespaces.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 35. How does Kubernetes DNS work?

**Answer:**

Kubernetes uses **CoreDNS** as the cluster DNS server. Every Pod gets a `/etc/resolv.conf` pointing to the CoreDNS service IP. Services and Pods are accessible via DNS.

**DNS naming format:**
```
# Service
<service-name>.<namespace>.svc.cluster.local

# Pod
<pod-ip-dashes>.<namespace>.pod.cluster.local
# Example: 10-244-1-5.default.pod.cluster.local

# Headless service Pods (StatefulSet)
<pod-name>.<service-name>.<namespace>.svc.cluster.local
# Example: mysql-0.mysql.default.svc.cluster.local
```

```bash
# Test DNS from inside a Pod
kubectl exec -it my-pod -- nslookup kubernetes.default
kubectl exec -it my-pod -- curl http://my-service.my-namespace.svc.cluster.local
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 36. What is the EKS Control Plane and how is it managed?

**Answer:**

The **EKS Control Plane** includes the Kubernetes API server, scheduler, controller manager, and etcd. AWS:
- Runs the control plane across **3 Availability Zones** for HA
- Manages **etcd backups** automatically
- Handles **security patches** and control plane upgrades
- Provides a dedicated API server endpoint for each cluster
- Monitors and auto-replaces unhealthy control plane nodes

**Customers are responsible for:**
- Worker nodes and node groups
- Application deployments
- Kubernetes version upgrades (with AWS assistance)

```bash
# Check EKS cluster status
aws eks describe-cluster --name my-cluster --region us-east-1

# List EKS clusters
aws eks list-clusters --region us-east-1
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 37. How does IAM integrate with EKS?

**Answer:**

EKS uses a **webhook token authenticator** that verifies AWS IAM identities:

1. `kubectl` requests a pre-signed STS token via `aws eks get-token`
2. The token is sent to the Kubernetes API server
3. The API server passes it to the **AWS IAM Authenticator** webhook
4. The webhook calls STS to validate the token and returns the IAM identity
5. Kubernetes maps the IAM identity to a Kubernetes RBAC user/group via the `aws-auth` ConfigMap (or EKS Access Entries)

```bash
# Get token manually (for debugging)
aws eks get-token --cluster-name my-cluster

# Check what identity kubectl uses
kubectl auth whoami
aws sts get-caller-identity
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 38. What is the aws-auth ConfigMap?

**Answer:**

The **`aws-auth` ConfigMap** in the `kube-system` namespace maps AWS IAM principals (users, roles) to Kubernetes RBAC users and groups. This controls who can access the cluster and with what permissions.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789:role/eks-node-group-role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
    - rolearn: arn:aws:iam::123456789:role/eks-admin-role
      username: admin
      groups:
        - system:masters
  mapUsers: |
    - userarn: arn:aws:iam::123456789:user/john
      username: john
      groups:
        - developers
```

> **Note:** AWS now recommends **EKS Access Entries** (API-based) as the preferred alternative to the aws-auth ConfigMap.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 39. What is the Amazon VPC CNI plugin?

**Answer:**

The **Amazon VPC CNI (Container Network Interface)** plugin is the default networking plugin for EKS. It assigns **real VPC IP addresses** to Pods from the node's subnet, enabling direct communication between Pods and other AWS resources.

**Key features:**
- Each Pod gets a real VPC IP address (not a virtual overlay network)
- Pods can communicate directly with RDS, ElastiCache, and other AWS services
- Security Groups can be applied directly to Pods (`SecurityGroupPolicy`)
- Supports IPv4 and IPv6

```bash
# Check VPC CNI version
kubectl describe daemonset aws-node -n kube-system | grep Image

# Check IP address allocation per node
kubectl get nodes -o custom-columns=\
  'NAME:.metadata.name,MAX_PODS:.status.capacity.pods'
```

**IP address calculation:**
Each EC2 instance type has a limit on ENIs and IPs per ENI. Max Pods = `(ENIs × (IPs per ENI - 1)) + 2`

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 40. How do you scale nodes in EKS?

**Answer:**

**Methods to scale EKS nodes:**

1. **Cluster Autoscaler (CA):** Automatically adjusts the number of nodes in an Auto Scaling Group based on pending Pods.

2. **Karpenter:** AWS-native node autoscaler that provisions optimal EC2 instances on demand (faster and more flexible than CA).

3. **Manual scaling:** Update the desired count in the ASG or via eksctl.

```bash
# Manual scaling with eksctl
eksctl scale nodegroup \
  --cluster my-cluster \
  --name standard-nodes \
  --nodes 5 \
  --nodes-min 2 \
  --nodes-max 10
```

```yaml
# Cluster Autoscaler deployment annotation
spec:
  template:
    spec:
      containers:
      - name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 41. What is Karpenter in EKS?

**Answer:**

**Karpenter** is an open-source, high-performance node autoscaler for Kubernetes, originally built by AWS. It provisions the right EC2 instance types for your workloads in seconds, rather than minutes.

**Advantages over Cluster Autoscaler:**
- Provisions nodes directly (no ASG required)
- Supports diverse instance types automatically (spot + on-demand mix)
- Consolidates underutilized nodes automatically
- Node provisioning in ~60 seconds vs 2-5 minutes for CA

```yaml
# NodePool (Karpenter v0.30+)
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
      - key: karpenter.sh/capacity-type
        operator: In
        values: ["spot", "on-demand"]
      - key: kubernetes.io/arch
        operator: In
        values: ["amd64"]
      - key: karpenter.k8s.aws/instance-category
        operator: In
        values: ["c", "m", "r"]
  disruption:
    consolidationPolicy: WhenUnderutilized
  limits:
    cpu: 1000
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 42. What is EKS Anywhere?

**Answer:**

**EKS Anywhere** allows you to create and manage Kubernetes clusters on your own infrastructure (on-premises, VMware vSphere, bare metal, or other cloud providers) using the same EKS tools and configurations used in AWS.

**Use cases:**
- Regulatory requirements that prevent cloud usage
- Data sovereignty requirements
- Hybrid cloud architectures
- Air-gapped environments

**Key features:**
- Uses same EKS configuration API
- Supports curated packages (CoreDNS, Cilium, etc.)
- Optionally connect to AWS via EKS Connector for management in the AWS Console

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 43. How does EKS integrate with AWS Load Balancer Controller?

**Answer:**

The **AWS Load Balancer Controller** is a controller that manages AWS Elastic Load Balancers for Kubernetes clusters. It provisions:
- **Application Load Balancers (ALBs)** for Ingress resources
- **Network Load Balancers (NLBs)** for Service type LoadBalancer

```yaml
# ALB Ingress via AWS Load Balancer Controller
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

```bash
# Install AWS Load Balancer Controller via Helm
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 44. What are EKS Add-ons?

**Answer:**

**EKS Add-ons** are operational software components that extend the functionality of Kubernetes. AWS manages their lifecycle (installation, updates, conflict resolution).

**Available add-ons:**
- `kube-proxy` — network proxy
- `coredns` — cluster DNS
- `vpc-cni` — Pod networking
- `aws-ebs-csi-driver` — EBS storage
- `aws-efs-csi-driver` — EFS storage
- `adot` — AWS Distro for OpenTelemetry
- `amazon-cloudwatch-observability` — CloudWatch monitoring
- `eks-pod-identity-agent` — Pod identity

```bash
# List available add-ons
aws eks describe-addon-versions --kubernetes-version 1.29

# Install an add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name aws-ebs-csi-driver \
  --service-account-role-arn arn:aws:iam::123456789:role/ebs-csi-role

# List installed add-ons
aws eks list-addons --cluster-name my-cluster
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 45. How do you manage secrets in EKS?

**Answer:**

**Options for secret management in EKS:**

1. **Kubernetes Secrets** (base64 encoded; encrypt with AWS KMS for security)
2. **AWS Secrets Manager + Secrets Store CSI Driver** (mount secrets as volumes)
3. **AWS Systems Manager Parameter Store** (same CSI driver)
4. **External Secrets Operator** (sync external secrets to Kubernetes Secrets)

```yaml
# Using Secrets Store CSI Driver with AWS Secrets Manager
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: aws-secrets
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "prod/myapp/db-password"
        objectType: secretsmanager
        objectAlias: db-password

---
spec:
  containers:
  - name: app
    volumeMounts:
    - name: secrets
      mountPath: /mnt/secrets
      readOnly: true
  volumes:
  - name: secrets
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: aws-secrets
```

```bash
# Enable EKS secrets encryption with KMS
aws eks create-cluster \
  --name my-cluster \
  --encryption-config "resources=[secrets],provider={keyArn=arn:aws:kms:...}"
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

## 🔴 Advanced Questions

---

### 46. How does the Kubernetes networking model work?

**Answer:**

Kubernetes enforces a flat networking model with these requirements:
- All Pods can communicate with each other without NAT
- All Nodes can communicate with all Pods without NAT
- The IP a Pod sees for itself is the same IP others see

**Network layers:**
1. **Pod-to-Pod** — via CNI plugin (VPC CNI, Calico, Cilium, Flannel)
2. **Pod-to-Service** — via kube-proxy (iptables or IPVS rules)
3. **External-to-Service** — via NodePort, LoadBalancer, or Ingress

**CNI Plugin responsibilities:**
- Assign IP addresses to Pods
- Set up routing rules
- Handle network policy enforcement (Calico, Cilium)

```bash
# Inspect CNI config on a node
cat /etc/cni/net.d/10-aws.conflist

# Trace network path
kubectl exec -it my-pod -- traceroute 10.100.0.1
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 47. What is a Custom Resource Definition (CRD)?

**Answer:**

A **CRD** extends the Kubernetes API by defining new resource types. Once a CRD is registered, you can create instances of it using `kubectl` like any built-in resource.

```yaml
# Define a CRD
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.io
spec:
  group: mycompany.io
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: [postgres, mysql]
              replicas:
                type: integer
                minimum: 1
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

```yaml
# Use the custom resource
apiVersion: mycompany.io/v1
kind: Database
metadata:
  name: my-db
spec:
  engine: postgres
  replicas: 3
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 48. What is a Vertical Pod Autoscaler (VPA)?

**Answer:**

**VPA** automatically adjusts the CPU and memory requests/limits of containers based on actual usage. Unlike HPA (which scales replicas), VPA scales resources per container.

**VPA modes:**
- `Off` — only provides recommendations (no changes)
- `Initial` — sets resources only at Pod creation
- `Auto` — updates resources and evicts/restarts Pods to apply changes

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: my-app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 2Gi
```

> **Note:** VPA and HPA should not be used together on the same metric (e.g., both on CPU). HPA + VPA on different metrics (e.g., HPA on custom metrics, VPA on CPU/memory) can work together.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 49. What is a Pod Disruption Budget (PDB)?

**Answer:**

A **PDB** limits the number of Pods of a replicated application that are down simultaneously during voluntary disruptions (node drains, cluster upgrades, evictions).

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  selector:
    matchLabels:
      app: my-app
  minAvailable: 2      # OR use maxUnavailable
  # maxUnavailable: 1  # Maximum 1 Pod can be unavailable
```

**Use cases:**
- Ensures minimum replicas during node drain for upgrades
- Protects stateful applications from data loss during disruptions
- Works with the Cluster Autoscaler and Karpenter

```bash
# Check PDB status
kubectl get pdb

# During cluster upgrade — drain blocks if PDB would be violated
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 50. How does Kubernetes implement service discovery?

**Answer:**

Kubernetes implements service discovery in two ways:

**1. DNS-based (recommended):**
- CoreDNS resolves Service names to ClusterIPs
- `<service>.<namespace>.svc.cluster.local`

**2. Environment variables:**
- At Pod start, Kubernetes injects env vars for all Services in the namespace
- e.g., `MY_SERVICE_SERVICE_HOST`, `MY_SERVICE_SERVICE_PORT`
- Limitation: only works for Services created before the Pod

**Headless Services (for StatefulSets):**
- Set `clusterIP: None`
- DNS returns individual Pod IPs instead of a single VIP
- Enables direct Pod addressing

```yaml
# Headless service
apiVersion: v1
kind: Service
metadata:
  name: my-stateful-svc
spec:
  clusterIP: None
  selector:
    app: my-stateful-app
  ports:
  - port: 5432
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 51. What is a Service Mesh and how does Istio work with Kubernetes?

**Answer:**

A **Service Mesh** is a dedicated infrastructure layer that manages service-to-service communication (traffic management, observability, security) using sidecar proxies without changing application code.

**Istio architecture:**

- **Data Plane:** Envoy sidecar proxies (injected automatically into Pods) handle all traffic
- **Control Plane (Istiod):** Manages proxy configuration, certificate lifecycle, and traffic policies

**Key Istio features:**
- **Traffic management:** canary deployments, circuit breaking, retries, timeouts
- **mTLS:** automatic mutual TLS between services
- **Observability:** distributed tracing (Jaeger), metrics (Prometheus), logging
- **Authorization policies:** fine-grained L7 access control

```yaml
# VirtualService — traffic splitting (canary)
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
  - my-app
  http:
  - route:
    - destination:
        host: my-app
        subset: v1
      weight: 90
    - destination:
        host: my-app
        subset: v2
      weight: 10
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 52. What is the Kubernetes API server admission controller?

**Answer:**

**Admission Controllers** are plugins that intercept API server requests **after authentication and authorization** but before persisting objects to etcd. They can validate or mutate requests.

**Two types:**
- **Mutating Admission Webhooks:** Modify the request (e.g., inject sidecar, add labels/defaults)
- **Validating Admission Webhooks:** Allow or reject the request (e.g., enforce policies)

**Built-in admission controllers:**
- `LimitRanger` — enforces resource limits
- `ResourceQuota` — enforces namespace quotas
- `PodSecurity` — enforces Pod security standards
- `MutatingAdmissionWebhook` — calls external webhook for mutations
- `ValidatingAdmissionWebhook` — calls external webhook for validation

```yaml
# Validating Webhook configuration
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: my-policy-webhook
webhooks:
- name: validate.mycompany.io
  clientConfig:
    service:
      name: policy-service
      namespace: kube-system
      path: /validate
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["apps"]
    resources: ["deployments"]
  admissionReviewVersions: ["v1"]
  sideEffects: None
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 53. What are Kubernetes Network Policies?

**Answer:**

**Network Policies** are Kubernetes resources that control traffic flow at the IP/port level between Pods, namespaces, and external endpoints. They require a CNI plugin that supports them (Calico, Cilium, Weave).

```yaml
# Deny all ingress, allow only from specific namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
  namespace: backend
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
      podSelector:
        matchLabels:
          role: api
    ports:
    - protocol: TCP
      port: 5432
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/8
```

> **Default behavior:** Without a NetworkPolicy, all traffic is allowed. Once a NetworkPolicy selects a Pod, that Pod follows the policy's rules.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 54. How does Kubernetes handle multi-tenancy?

**Answer:**

Kubernetes is not inherently multi-tenant but can be made so using multiple isolation mechanisms:

**Soft multi-tenancy (shared cluster):**
- **Namespaces** — logical isolation
- **RBAC** — access control per team/namespace
- **ResourceQuotas** — limit resource consumption per namespace
- **LimitRanges** — default/max resources per Pod in a namespace
- **Network Policies** — isolate network traffic between namespaces
- **Pod Security Standards** — enforce security contexts

**Hard multi-tenancy (strong isolation):**
- Separate clusters per tenant (VCluster, separate EKS clusters)
- **VCluster** — virtual Kubernetes clusters inside a namespace
- **Capsule / HNC** — multi-tenancy frameworks

```yaml
# ResourceQuota per tenant namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-a-quota
  namespace: tenant-a
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    services: "10"
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 55. What is the difference between kube-proxy modes (iptables vs IPVS)?

**Answer:**

`kube-proxy` manages network rules on nodes for Service routing. It supports three modes:

| Feature | iptables | IPVS |
|---------|----------|------|
| **Routing** | Sequential rule matching | Hash table lookup |
| **Performance** | Degrades at scale (O(n)) | Constant time (O(1)) |
| **Load balancing algorithms** | Round-robin only | RR, least connections, source hash, etc. |
| **Scale** | Good up to ~1000 Services | Scales to 10,000+ Services |
| **Health checking** | Limited | Built-in |

```bash
# Check current kube-proxy mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode

# Switch to IPVS mode (via configmap)
kubectl edit configmap kube-proxy -n kube-system
# Set: mode: "ipvs"
```

For large clusters (>1000 Services), **IPVS** mode is strongly recommended. EKS also supports IPVS mode.

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 56. How does Kubernetes garbage collection work?

**Answer:**

Kubernetes garbage collection automatically removes objects that are no longer needed:

**1. Owner References & Cascading Deletion:**
- Resources have `ownerReferences` pointing to their owner (e.g., Pod → ReplicaSet → Deployment)
- When an owner is deleted, dependents are deleted via **Foreground** or **Background** cascading deletion

```bash
# Delete with cascade (default: background)
kubectl delete deployment my-app

# Orphan dependents (don't delete ReplicaSet/Pods)
kubectl delete deployment my-app --cascade=orphan
```

**2. Image Garbage Collection:**
- `kubelet` removes unused container images when disk usage exceeds `imageGCHighThresholdPercent` (default 85%)

**3. Container Garbage Collection:**
- Removes terminated containers based on `MaxContainerCount` and `MaxDeadContainerAge`

**4. API Resource GC:**
- Removes completed Jobs, finished Pods (based on `ttlSecondsAfterFinished`)

```yaml
# Auto-delete Job after 60 seconds
spec:
  ttlSecondsAfterFinished: 60
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 57. What is the Cluster Autoscaler and how does it work?

**Answer:**

**Cluster Autoscaler (CA)** automatically adjusts the size of a Kubernetes cluster by adding or removing nodes based on Pod scheduling needs.

**Scale-up logic:**
1. Pod is **Pending** because no node has enough resources
2. CA simulates which node group could schedule the Pod
3. CA increases the ASG desired count
4. New node joins, Pod is scheduled

**Scale-down logic:**
1. CA checks if a node's utilization drops below 50% for 10+ minutes
2. Verifies all Pods can be rescheduled on other nodes
3. Drains the node and terminates the EC2 instance

```yaml
# EKS — ASG tags required for CA
Tags:
  k8s.io/cluster-autoscaler/enabled: "true"
  k8s.io/cluster-autoscaler/<cluster-name>: "owned"
```

**Key CA flags:**
```
--scale-down-delay-after-add=10m       # Wait after scale-up before scale-down check
--scale-down-unneeded-time=10m         # Node must be unneeded for this long
--skip-nodes-with-local-storage=false  # Allow scale-down of nodes with local storage
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 58. What is GitOps and how is it implemented with Kubernetes?

**Answer:**

**GitOps** is an operational framework where Git is the single source of truth for infrastructure and application configuration. Changes are made via Git commits/PRs, and automated agents reconcile the cluster state to match.

**GitOps tools for Kubernetes:**
- **ArgoCD** — declarative, Git-based continuous delivery
- **Flux CD** — lightweight, GitOps toolkit for Kubernetes

**ArgoCD workflow:**
1. Developer commits Kubernetes manifests to Git
2. ArgoCD detects the diff between Git and cluster state
3. ArgoCD syncs the cluster (applies manifests)
4. Health status is reported back

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-app
    targetRevision: main
    path: k8s/
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 59. How do you troubleshoot a CrashLoopBackOff error?

**Answer:**

`CrashLoopBackOff` means a container is repeatedly crashing and Kubernetes is backing off before restarting it.

**Step-by-step troubleshooting:**

```bash
# 1. Check Pod status and events
kubectl get pods
kubectl describe pod <pod-name>

# 2. Check current logs
kubectl logs <pod-name>

# 3. Check previous container logs (if container already crashed)
kubectl logs <pod-name> --previous

# 4. Check exit code (tells you why the container exited)
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.exitCode}'

# 5. Debug with a shell (if container has bash)
kubectl debug -it <pod-name> --image=busybox --target=<container-name>
```

**Common exit codes:**

| Exit Code | Meaning |
|-----------|---------|
| `0` | Success (no crash — liveness probe failing?) |
| `1` | Application error |
| `137` | OOMKilled (out of memory) |
| `139` | Segmentation fault |
| `143` | Graceful termination (SIGTERM) |

**Common root causes:**
- Wrong command or entrypoint in the container
- Missing environment variables or secrets
- Application fails healthcheck (liveness probe)
- OOMKilled — increase memory limits
- Bad image or missing dependencies

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 60. What is OPA Gatekeeper and how is it used in Kubernetes?

**Answer:**

**OPA Gatekeeper** is a policy engine for Kubernetes built on Open Policy Agent (OPA). It uses validating admission webhooks and CRDs to enforce custom policies.

**Key concepts:**
- **ConstraintTemplate** — defines the policy logic in Rego
- **Constraint** — an instance of a ConstraintTemplate with specific parameters

```yaml
# ConstraintTemplate — enforce required labels
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: requirelabels
spec:
  crd:
    spec:
      names:
        kind: RequireLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package requirelabels
      violation[{"msg": msg}] {
        provided := {label | input.review.object.metadata.labels[label]}
        required := {label | label := input.parameters.labels[_]}
        missing := required - provided
        count(missing) > 0
        msg := sprintf("Missing required labels: %v", [missing])
      }

---
# Constraint — apply the policy
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequireLabels
metadata:
  name: must-have-env-label
spec:
  match:
    kinds:
    - apiGroups: ["apps"]
      kinds: ["Deployment"]
  parameters:
    labels: ["env", "team", "app"]
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 61. How does EKS use IRSA (IAM Roles for Service Accounts)?

**Answer:**

**IRSA (IAM Roles for Service Accounts)** allows Kubernetes Pods to assume AWS IAM roles using Kubernetes Service Accounts. This replaces the old pattern of assigning IAM roles to EC2 nodes.

**How it works:**
1. EKS cluster has an OIDC provider configured
2. IAM role has a trust policy allowing the OIDC provider and specific service account
3. Pod uses a service account annotated with the IAM role ARN
4. The EKS Pod Identity Webhook injects AWS credential env vars into the Pod
5. AWS SDK in the Pod automatically fetches temporary credentials via OIDC token

```bash
# Create OIDC provider for EKS cluster
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve

# Create IAM role for service account
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --namespace my-namespace \
  --name my-service-account \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

```yaml
# Service Account with IRSA annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: my-namespace
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/my-pod-role
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 62. What is EKS Pod Identity and how does it differ from IRSA?

**Answer:**

**EKS Pod Identity** is a newer, simpler mechanism for granting AWS permissions to Pods. It uses a dedicated Pod Identity Agent DaemonSet instead of OIDC webhooks.

**Comparison:**

| Feature | IRSA | EKS Pod Identity |
|---------|------|-----------------|
| **Mechanism** | OIDC + webhook | Pod Identity Agent DaemonSet |
| **IAM trust policy** | Complex (OIDC condition) | Simple (pods.eks.amazonaws.com) |
| **Cross-account** | Supported | Supported |
| **Cluster config** | OIDC provider required | Agent add-on required |
| **Simplicity** | More complex setup | Simpler setup |

```bash
# Enable Pod Identity add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name eks-pod-identity-agent

# Create Pod Identity association
aws eks create-pod-identity-association \
  --cluster-name my-cluster \
  --namespace my-namespace \
  --service-account my-service-account \
  --role-arn arn:aws:iam::123456789:role/my-pod-role
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 63. How do you implement multi-cluster EKS architectures?

**Answer:**

Multi-cluster architectures improve availability, separate concerns, and meet compliance requirements.

**Common patterns:**

**1. Active-Active (Global Load Balancing):**
- Multiple EKS clusters in different regions
- Route 53 latency/geolocation routing between clusters
- Data synchronization via CRDTs or database replication

**2. Active-Passive (Disaster Recovery):**
- Primary cluster in one region, standby in another
- Velero for backup/restore
- Route 53 failover routing

**3. Hub-Spoke (Management Cluster):**
- Central management cluster running ArgoCD/Flux
- Spoke clusters receive workloads from the hub

**Tools for multi-cluster:**
- **ArgoCD** — multi-cluster GitOps
- **Cluster API (CAPI)** — manage cluster lifecycle
- **AWS App Mesh** — cross-cluster service mesh
- **Velero** — backup and DR

```bash
# Register multiple clusters in ArgoCD
argocd cluster add --kubeconfig ./cluster2-kubeconfig arn:aws:eks:us-west-2:123456789:cluster/cluster2
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 64. How do you optimize costs in an EKS environment?

**Answer:**

**Cost optimization strategies:**

**1. Right-size workloads:**
- Use VPA recommendations to set appropriate resource requests
- Avoid over-provisioning CPU/memory

**2. Spot Instances:**
- Use Karpenter or CA with mixed instance types and Spot
- Design apps to handle interruptions gracefully (2-minute notice)

**3. Node consolidation:**
- Enable Karpenter's `consolidation` policy to bin-pack Pods

**4. Fargate for variable workloads:**
- Only pay for actual Pod CPU/memory

**5. Cluster Autoscaler / Karpenter:**
- Scale down idle nodes automatically

**6. Savings Plans & Reserved Instances:**
- Commit to 1 or 3 years for baseline workloads

```yaml
# Karpenter — prefer Spot, fall back to On-Demand
spec:
  template:
    spec:
      requirements:
      - key: karpenter.sh/capacity-type
        operator: In
        values: ["spot", "on-demand"]
  disruption:
    consolidationPolicy: WhenUnderutilized
    consolidateAfter: 30s
```

```bash
# Monitor costs with Kubecost
helm install kubecost cost-analyzer \
  --repo https://kubecost.github.io/cost-analyzer/ \
  --namespace kubecost --create-namespace
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 65. What is the EKS Distro (EKS-D)?

**Answer:**

**Amazon EKS Distro (EKS-D)** is the same Kubernetes distribution that powers Amazon EKS, made available for you to use anywhere. It is a free, open-source distribution of Kubernetes that includes:

- The same Kubernetes version and patches used in EKS
- Extended support timelines for older K8s versions
- Same versions of dependencies: etcd, CoreDNS, metrics-server, etc.
- Amazon-tested and signed binaries

**Use cases:**
- Run the same Kubernetes distribution on-premises as in EKS
- Consistent behavior across hybrid environments
- Foundation for EKS Anywhere

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 66. How do you implement blue/green deployments in EKS?

**Answer:**

**Blue/Green deployment** runs two identical environments (blue = current, green = new) and switches traffic instantaneously.

**Method 1: Kubernetes Services + Label Switching**

```bash
# Switch traffic from blue to green by updating service selector
kubectl patch service my-service \
  -p '{"spec":{"selector":{"version":"green"}}}'
```

**Method 2: AWS ALB Weighted Target Groups**

```yaml
# Ingress with traffic splitting (AWS Load Balancer Controller)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    alb.ingress.kubernetes.io/actions.blue-green: |
      {
        "type": "forward",
        "forwardConfig": {
          "targetGroups": [
            {"serviceName": "blue-service", "servicePort": 80, "weight": 0},
            {"serviceName": "green-service", "servicePort": 80, "weight": 100}
          ]
        }
      }
```

**Method 3: ArgoCD Rollouts (Argo Rollouts)**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    blueGreen:
      activeService: my-active-service
      previewService: my-preview-service
      autoPromotionEnabled: false
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 67. How does EKS handle etcd backups and disaster recovery?

**Answer:**

In EKS, **etcd is fully managed by AWS**. You do not have direct access to etcd. AWS automatically handles:
- etcd backups (multiple times per day)
- Multi-AZ replication for etcd
- Automatic etcd recovery

**For application-level DR:**
- **Velero** — backs up Kubernetes resources and PersistentVolumes to S3

```bash
# Install Velero with AWS S3 backend
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket my-velero-bucket \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1 \
  --secret-file ./credentials-velero

# Backup all resources in a namespace
velero backup create my-backup --include-namespaces production

# Schedule daily backups
velero schedule create daily-backup \
  --schedule="0 1 * * *" \
  --include-namespaces production

# Restore from backup
velero restore create --from-backup my-backup
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 68. What are EKS security best practices?

**Answer:**

**1. IAM and RBAC:**
- Use IRSA or Pod Identity instead of node-level IAM roles
- Apply least-privilege IAM policies
- Use EKS Access Entries instead of aws-auth ConfigMap
- Regularly audit RBAC bindings

**2. Network Security:**
- Enable private API server endpoint
- Use Security Groups for Pods
- Implement Network Policies (Calico or Cilium)
- Use VPC endpoints for AWS service traffic

**3. Secrets Management:**
- Encrypt Kubernetes Secrets with KMS at rest
- Use AWS Secrets Manager via CSI driver or External Secrets Operator

**4. Pod Security:**
- Enforce Pod Security Standards (`Restricted` profile)
- Disable privilege escalation: `allowPrivilegeEscalation: false`
- Run containers as non-root users
- Use read-only root filesystems

**5. Runtime Security:**
- Enable Amazon GuardDuty for EKS (runtime threat detection)
- Use Falco for real-time runtime security

**6. Image Security:**
- Scan images with Amazon ECR image scanning (or Trivy/Snyk)
- Use immutable image tags
- Sign images with Notary/Cosign

```yaml
# Restricted Pod Security Standard
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 69. How do you monitor and observe an EKS cluster?

**Answer:**

**Three pillars of observability: Metrics, Logs, Traces**

**Metrics:**
- **Amazon CloudWatch Container Insights** — native AWS monitoring for EKS
- **Prometheus + Grafana** — open-source, highly flexible
- **Datadog / New Relic** — enterprise observability platforms

**Logs:**
- **Fluent Bit** (DaemonSet) → CloudWatch Logs / OpenSearch
- **Fluentd** — more plugins, slightly heavier
- EKS control plane logging: enable in AWS Console/CLI

**Traces:**
- **AWS X-Ray** — native AWS distributed tracing
- **OpenTelemetry (ADOT)** — standard collection pipeline
- **Jaeger / Tempo** — open-source tracing

```bash
# Enable EKS Control Plane logging
aws eks update-cluster-config \
  --name my-cluster \
  --logging '{"clusterLogging":[{"types":["api","audit","authenticator","controllerManager","scheduler"],"enabled":true}]}'

# Install Prometheus stack via Helm
helm install kube-prometheus-stack \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  --set grafana.adminPassword=admin123
```

```yaml
# CloudWatch agent as a DaemonSet (Container Insights)
# Install via add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name amazon-cloudwatch-observability
```

[🔝 Back to Table of Contents](#-table-of-contents)

---

### 70. How do you upgrade an EKS cluster with zero downtime?

**Answer:**

**EKS upgrade process (recommended steps):**

**Phase 1: Preparation**
```bash
# 1. Review EKS release notes and deprecated APIs
# 2. Test upgrade in lower environments first
# 3. Backup with Velero

# Check current version
aws eks describe-cluster --name my-cluster --query cluster.version

# Check deprecated API usage
kubectl convert --help
# Use Pluto to detect deprecated APIs
pluto detect-all-in-cluster --target-versions k8s=v1.29.0
```

**Phase 2: Upgrade the Control Plane**
```bash
# Upgrade control plane (15-25 min, no downtime)
aws eks update-cluster-version \
  --name my-cluster \
  --kubernetes-version 1.29

# Wait for completion
aws eks wait cluster-active --name my-cluster
```

**Phase 3: Upgrade Add-ons**
```bash
# Update EKS add-ons (vpc-cni, coredns, kube-proxy)
aws eks update-addon \
  --cluster-name my-cluster \
  --addon-name vpc-cni \
  --resolve-conflicts OVERWRITE
```

**Phase 4: Upgrade Node Groups**
```bash
# For Managed Node Groups
aws eks update-nodegroup-version \
  --cluster-name my-cluster \
  --nodegroup-name standard-nodes

# The process: new nodes → cordon old nodes → drain → terminate
# PodDisruptionBudgets are respected during drain
```

**Phase 5: Validate**
```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A | grep Warning
```

> **Key tip:** Upgrade one minor version at a time (e.g., 1.27 → 1.28 → 1.29). Skipping versions is not supported.

[🔝 Back to Table of Contents](#-table-of-contents)

---

## 📌 Quick Reference

### Kubectl Cheat Sheet

```bash
# Context management
kubectl config get-contexts
kubectl config use-context <context>

# Resource management
kubectl get all -n <namespace>
kubectl apply -f <file>
kubectl delete -f <file>
kubectl edit <resource> <name>

# Debugging
kubectl describe <resource> <name>
kubectl logs <pod> -c <container> -f --previous
kubectl exec -it <pod> -- /bin/sh
kubectl top pods --sort-by=memory

# Port forwarding
kubectl port-forward svc/<service> 8080:80

# Rollout management
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=2
```

---

*Happy interviewing! 🚀 This guide covers the most commonly asked Kubernetes and EKS interview questions across all experience levels.*
