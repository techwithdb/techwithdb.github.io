---
title: "Kubernetes Interview Questions & Answers (2026) part 03"
description: "50+ Kubernetes interview questions and answers from basic to advanced — covering Pods, Deployments, Services, Networking, RBAC, Helm, Autoscaling, Security, and real-world troubleshooting scenarios."
date: 2025-01-20
author: "DB"
tags: ["Kubernetes", "K8s", "Interview", "DevOps", "Containers", "CKA", "CKAD"]
tool: "kubernetes"
level: "All Levels"
question_count: 30
draft: false
---

> 30 real-world scenario questions across 14 categories. Every answer explains **what**, **why**, and **how** — the way a senior Kubernetes/DevOps engineer explains it in an actual interview.

---

{{< qa num="1" q="Your CTO asks you to explain why the company should use EKS instead of self-managed Kubernetes on EC2. What do you say?" level="advanced" >}}


**Answer:**

**The problem with self-managed Kubernetes on EC2:**

When you run Kubernetes yourself on EC2 you are responsible for everything: installing and upgrading the control plane (API server, etcd, scheduler, controller manager), patching etcd for security vulnerabilities, ensuring etcd backups, managing control plane HA across multiple EC2 instances, and debugging issues when the API server goes down at 2 AM. A typical self-managed control plane requires 3–5 dedicated EC2 instances just for the control plane, a dedicated team to maintain it, and on-call engineers who deeply understand Kubernetes internals.

**What EKS gives you:**

EKS is AWS's managed Kubernetes service. AWS runs and manages the control plane for you. The Kubernetes API server, etcd, and control plane components run in an AWS-managed account — you never see them, never patch them, never back them up. AWS guarantees 99.95% SLA on the control plane.

**Real-world comparison:**

| Responsibility | Self-Managed on EC2 | EKS |
|---------------|---------------------|-----|
| Kubernetes version upgrades | Your team | One-click in console |
| etcd backup and restore | Your team | AWS handles it |
| Control plane HA | Your team (3+ EC2 instances) | AWS handles it |
| Control plane security patches | Your team | AWS handles it |
| Cost of control plane EC2 | You pay for 3–5 instances | $0.10/hour flat fee |
| Worker nodes | You manage | You manage (but with managed node groups available) |

**Real-world example:**

A fintech startup was spending 40% of their DevOps team's time maintaining a self-managed Kubernetes cluster — patching, backing up etcd, debugging control plane issues. After migrating to EKS, that 40% of time was redirected to building product features. The control plane costs $0.10/hour (~$72/month) which was less than the EC2 cost of their self-managed control plane.

---
{{< /qa >}}

{{< qa num="2" q="Explain the architecture of EKS. What components does AWS manage and what do you manage?" level="advanced" >}}


**Answer:**

**EKS Architecture has two planes:**

**Control Plane (AWS manages entirely):**

The control plane runs in an AWS-owned VPC that is separate from your account. You never access the underlying EC2 instances. AWS manages:
- **kube-apiserver** — the front door to the Kubernetes cluster, receives all kubectl commands and API calls
- **etcd** — the distributed key-value store that holds all cluster state (every pod, service, deployment definition is stored here)
- **kube-scheduler** — decides which node a new pod should run on based on resource requests, affinity rules, and taints/tolerations
- **kube-controller-manager** — runs controllers that watch cluster state and reconcile it toward desired state (e.g., the ReplicaSet controller ensures 3 replicas are always running)

**Data Plane (you manage):**

The data plane runs in your AWS account and your VPC. You manage:
- **Worker nodes** — EC2 instances (or Fargate) where your pods actually run
- **kubelet** — agent running on each worker node, communicates with the control plane, manages pod lifecycle on that node
- **kube-proxy** — handles network routing rules on each node for Service traffic
- **Container runtime** — containerd, which runs the actual containers
- **Add-ons** — CoreDNS, VPC CNI, kube-proxy (you manage the versions and upgrades)

**How they connect:**

Your VPC and the AWS-managed control plane VPC are connected via an AWS-managed endpoint. Your nodes call the API server endpoint (e.g., `https://xxx.gr7.ap-south-1.eks.amazonaws.com`) to register themselves and receive scheduling instructions.

**Real-world implication:**

When a developer runs `kubectl apply -f deployment.yaml`, the request goes to the managed API server → API server stores the desired state in etcd → scheduler picks a node → kubelet on that node pulls the container image and starts the pod. You are responsible for the node being healthy, having the right IAM permissions, and being in the right VPC subnet.

---
{{< /qa >}}

{{< qa num="3" q="Your team is setting up EKS for the first time. Walk me through the complete setup process from scratch." level="advanced" >}}


**Answer:**

**Prerequisites:**
- AWS CLI configured with appropriate IAM permissions
- `kubectl` installed locally
- `eksctl` tool installed (simplifies cluster creation significantly)

**Step 1 — Create the EKS Cluster:**

```bash
eksctl create cluster \
  --name production-cluster \
  --region ap-south-1 \
  --version 1.29 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 6 \
  --managed \
  --with-oidc \
  --ssh-access \
  --ssh-public-key my-key-pair
```

This creates: the EKS control plane, a VPC with public and private subnets across 3 AZs, a Managed Node Group with 3 t3.medium worker nodes, an OIDC provider (required for IAM Roles for Service Accounts), and configures your local kubeconfig.

**Step 2 — Verify connectivity:**

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

**Step 3 — Install essential add-ons:**

```bash
# CoreDNS (usually installed by default)
# VPC CNI for pod networking
eksctl utils update-aws-vpc-cni-chart --cluster production-cluster --region ap-south-1

# Metrics Server (required for HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# AWS Load Balancer Controller (for ALB/NLB Ingress)
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=production-cluster
```

**Step 4 — Configure kubectl access for your team:**

```bash
# Add other IAM users/roles to the cluster
eksctl create iamidentitymapping \
  --cluster production-cluster \
  --region ap-south-1 \
  --arn arn:aws:iam::123456789:role/DevTeamRole \
  --group system:masters \
  --username devteam
```

**Real-world advice:**

Use eksctl or Terraform for cluster creation — never create clusters manually via the console for production. Infrastructure as Code means you can recreate the exact same cluster in a disaster recovery scenario.

{{< /qa >}}


{{< qa num="4" q="What is the difference between Self-Managed Node Groups, Managed Node Groups, and AWS Fargate for EKS? When do you use each?" level="advanced" >}}


**Answer:**

**Self-Managed Node Groups:**

You create the EC2 instances yourself, install the Kubernetes node components (kubelet, kube-proxy, container runtime), register them to the cluster, and manage all patching and upgrades manually. You have complete control over every aspect of the node.

Use when: You need custom AMIs with specific kernel versions, specific security hardening that AWS Managed Node Groups don't support, or GPU nodes with custom driver configurations.

**Managed Node Groups (MNG):**

AWS creates and manages the EC2 instances in your account. You define the instance type, desired capacity, and AMI type (Amazon Linux 2, Bottlerocket, Windows). AWS handles rolling upgrades — when you update the Kubernetes version or AMI, AWS drains nodes one by one, terminates the old node, and launches a new one. Nodes still appear in your account and you pay for the EC2 instances.

Use when: This is the right choice for 90% of workloads. Reduces operational overhead significantly compared to self-managed.

**AWS Fargate:**

Completely serverless. You never see or manage EC2 instances. Each pod runs in its own isolated micro-VM. AWS manages the underlying compute entirely. You pay per pod based on vCPU and memory.

Use when: You want zero node management. Good for batch jobs, infrequent workloads, or teams without Kubernetes infrastructure expertise. Not suitable for DaemonSets, hostPath volumes, or pods that need node-level access.

**Real-world example:**

A company uses Managed Node Groups for their web tier and API services (predictable, always-on), and Fargate for their nightly data processing jobs (run for 2 hours then terminate — no point managing nodes for intermittent batch work).

{{< /qa >}}

{{< qa num="5" q="Your production EKS cluster has all worker nodes in public subnets. Your security team says this is wrong. What is the issue and how do you fix it?" level="advanced" >}}


**Answer:**

**The security problem:**

Worker nodes in public subnets have public IP addresses assigned to them. This means:
- Each worker node EC2 instance is directly reachable from the internet
- If a container escapes (container breakout exploit), the underlying node is exposed to the internet
- Kubernetes NodePort services on those nodes are potentially reachable from anywhere
- The attack surface is dramatically larger than necessary

**The correct architecture — private worker nodes:**

```
Internet
    |
    v
Internet Gateway
    |
    v
Public Subnets (AZ-a, AZ-b, AZ-c)
    [ALB / NLB Load Balancers only]
    |
    v
Private Subnets (AZ-a, AZ-b, AZ-c)
    [All EKS Worker Nodes]
    [NAT Gateway for outbound internet (to pull images)]
```

Worker nodes should be in private subnets. They have no public IPs. Inbound traffic reaches them only through the load balancer. Outbound traffic (to pull container images from ECR, call AWS APIs) goes through a NAT Gateway.

**How to migrate existing nodes:**

You cannot move existing nodes between subnets. The fix is to create a new node group in private subnets and migrate workloads:

```bash
# Create new node group in private subnets
eksctl create nodegroup \
  --cluster production-cluster \
  --name private-workers \
  --node-private-networking \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 10

# Taint the old public nodes to prevent new pods being scheduled there
kubectl taint nodes <public-node-name> dedicated=old:NoSchedule

# Drain each old node (moves pods to new private nodes)
kubectl drain <public-node-name> --ignore-daemonsets --delete-emptydir-data

# Delete old node group after pods are migrated
eksctl delete nodegroup --cluster production-cluster --name public-workers
```

**EKS API endpoint access:**

Also configure the EKS cluster API endpoint to be private-only for maximum security:
```
EKS Console → Cluster → Networking → Endpoint access → Private
```
This means `kubectl` commands only work from within the VPC (via VPN or bastion host), not from the public internet.

{{< /qa >}}

{{< qa num="6" q="You need to run GPU workloads for ML inference alongside regular web service workloads on the same EKS cluster. How do you architect this?" level="advanced" >}}


**Answer:**

**The challenge:**

GPU instances (p3, g4dn) are expensive — $0.526/hour to $12.24/hour. You don't want your web service pods accidentally scheduled on GPU nodes (wasting money). You don't want ML pods scheduled on regular t3 nodes (they'd fail or run slowly without GPUs).

**Solution: Multiple Node Groups with Taints and Tolerations**

**Step 1 — Create a dedicated GPU node group with a taint:**

```bash
eksctl create nodegroup \
  --cluster production-cluster \
  --name gpu-workers \
  --node-type g4dn.xlarge \
  --nodes 2 \
  --nodes-min 0 \
  --nodes-max 10 \
  --node-labels "workload-type=gpu" \
  --node-taints "nvidia.com/gpu=true:NoSchedule"
```

The taint `nvidia.com/gpu=true:NoSchedule` means: no pod will be scheduled on this node UNLESS that pod explicitly has a matching toleration.

**Step 2 — Install the NVIDIA device plugin:**

```bash
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.0/nvidia-device-plugin.yml
```

This makes GPUs visible to Kubernetes as schedulable resources.

**Step 3 — ML inference pod with toleration and GPU request:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-inference-service
spec:
  replicas: 2
  template:
    spec:
      tolerations:
      - key: "nvidia.com/gpu"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
      nodeSelector:
        workload-type: gpu
      containers:
      - name: inference
        image: my-ml-inference:v1.2
        resources:
          limits:
            nvidia.com/gpu: 1      # Request exactly 1 GPU
          requests:
            memory: "8Gi"
            cpu: "4"
```

**Step 4 — Web service pod with no GPU toleration (cannot land on GPU nodes):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-service
spec:
  replicas: 10
  template:
    spec:
      # No tolerations for GPU taint
      # No nodeSelector for GPU nodes
      containers:
      - name: web
        image: my-web-service:v2.1
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
```

Web service pods cannot be scheduled on GPU nodes (no toleration). ML pods are pinned to GPU nodes (toleration + nodeSelector). GPU resources are never wasted on workloads that don't need them.

{{< /qa >}}

{{< qa num="7" q="Explain how pod networking works in EKS. A developer asks why their pod can directly ping another pod using its IP address." level="advanced" >}}


**Answer:**

**The VPC CNI plugin — EKS's networking foundation:**

EKS uses the Amazon VPC CNI (Container Network Interface) plugin. This is fundamentally different from how most Kubernetes CNI plugins work.

**Most CNI plugins (Flannel, Calico overlay mode):** Create a virtual overlay network. Pods get IPs from a virtual CIDR that is separate from the VPC CIDR. Traffic between pods is encapsulated in packets (VXLAN tunnels) and sent across the real network. The VPC network sees EC2 instance IPs only — pod IPs are "hidden" inside tunnels.

**Amazon VPC CNI:** Every pod gets a REAL VPC IP address. Not a virtual IP — an actual IP from your VPC subnet CIDR. The pod's IP is routable anywhere in the VPC (and by extension, in any VPC that is peered or connected via Transit Gateway).

**How the VPC CNI allocates IPs:**

Each EC2 worker node has multiple Elastic Network Interfaces (ENIs). Each ENI can have multiple private IP addresses. The VPC CNI pre-allocates a pool of IP addresses from ENIs and assigns one to each pod when it starts.

```
Worker Node (t3.large):
  ENI 1: 10.0.1.5 (node's primary IP)
         10.0.1.10 (assigned to pod-1)
         10.0.1.11 (assigned to pod-2)
         10.0.1.12 (assigned to pod-3)
  ENI 2: 10.0.1.20 (assigned to pod-4)
         10.0.1.21 (assigned to pod-5)
```

**Why pod-to-pod pinging works directly:**

Because pod-2 (IP 10.0.1.11) and pod-4 (IP 10.0.1.20) are both real VPC IPs, the VPC routing tables already know how to route between them — no tunnel or overlay needed. It's just VPC routing.

**Real-world implication:**

Your VPC CIDR must be large enough to accommodate all your pod IPs. If you have 100 nodes each running 30 pods, you need at least 3,000 IPs. A /24 subnet only has 256 addresses. Plan your VPC CIDR with pod density in mind — a /16 VPC with multiple /20 subnets per AZ is a common production choice.

{{< /qa >}}

{{< qa num="8" q="What is the difference between ClusterIP, NodePort, LoadBalancer, and Ingress? When do you use each?" level="advanced" >}}


**Answer:**

**ClusterIP (default):**

Creates a virtual IP address that is only reachable from within the cluster. Pods in the cluster can reach the service at this stable IP, even as the underlying pods change.

Use when: Service-to-service communication within the cluster. Your frontend pods talking to your backend API pods. The backend API should never be exposed externally — ClusterIP is the right choice.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-api
spec:
  type: ClusterIP
  selector:
    app: backend-api
  ports:
  - port: 8080
    targetPort: 8080
```

**NodePort:**

Opens a port (30000–32767) on EVERY worker node. Traffic hitting `<any-node-IP>:<nodeport>` is forwarded to the pods. Technically reachable from outside the cluster but requires knowing node IPs.

Use when: Development and testing only. Never for production — it requires knowing node IPs, bypasses the load balancer, and exposes a port on every node unnecessarily.

**LoadBalancer:**

Creates an AWS Load Balancer (NLB or CLB by default) and points it at the pods. One LoadBalancer Service = one AWS Load Balancer = one AWS cost item.

Use when: You need to expose a single TCP/UDP service directly. Good for non-HTTP services like a game server or database proxy. For HTTP services, Ingress is better because one Ingress can route to multiple services, whereas LoadBalancer creates a separate load balancer per service.

**Ingress:**

An Ingress is a Layer 7 (HTTP/HTTPS) routing resource that uses one load balancer to route to multiple services based on hostname and path.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
spec:
  rules:
  - host: api.mycompany.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
  - host: admin.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 80
```

One ALB, three services, two hostnames. Compare this to LoadBalancer type which would create three separate load balancers at 3× the cost.

---
{{< /qa >}}

{{< qa num="9" q="Your Ingress is not routing traffic correctly — some paths return 404. Walk me through your debugging approach." level="advanced" >}}


**Answer:**

**Step 1 — Check if the Ingress resource exists and has an Address:**

```bash
kubectl get ingress -n my-namespace
# If ADDRESS column is empty, the Ingress controller hasn't reconciled it yet
```

If ADDRESS is empty, the AWS Load Balancer Controller (or nginx Ingress controller) is not working. Check:
```bash
kubectl get pods -n kube-system | grep aws-load-balancer
kubectl logs -n kube-system deployment/aws-load-balancer-controller
```

**Step 2 — Describe the Ingress for events:**

```bash
kubectl describe ingress my-ingress -n my-namespace
```

Look at the Events section. Common errors:
- "Failed to build model: couldn't find nodegroup" — node label missing
- "InvalidParameterException" — security group or subnet annotation wrong

**Step 3 — Check the backend Service and Endpoints:**

```bash
# Does the Service exist?
kubectl get svc my-service -n my-namespace

# Does the Service have any Endpoints (pods selected)?
kubectl get endpoints my-service -n my-namespace
# If ENDPOINTS shows <none>, no pods are matching the Service selector
```

A Service with no Endpoints means the label selector in the Service does not match any running pods. Compare:
```bash
kubectl get pods --show-labels -n my-namespace
kubectl describe svc my-service -n my-namespace | grep Selector
```

**Step 4 — Verify the path and pathType:**

A common mistake is using `Exact` pathType when `Prefix` is needed. `/users/123` will NOT match a rule with `path: /users` and `pathType: Exact`. Change to `pathType: Prefix`.

**Step 5 — Check ALB Target Group health:**

In the AWS console, go to EC2 → Target Groups → find the target group associated with your ALB. If targets show "unhealthy", the pods are failing the load balancer health check. The health check path (e.g., `/health`) may not exist in your application.

**Real-world example:**

A team's Ingress worked for `/api` paths but returned 404 for `/api/v2/users`. The issue: the annotation was `path: /api` with `pathType: Exact`. Changing to `pathType: Prefix` fixed it — `/api/v2/users` is a prefix match for `/api`.

{{< /qa >}}

{{< qa num="10" q="Your stateful application (PostgreSQL) on EKS needs persistent storage that survives pod restarts. How do you set this up?" level="advanced" >}}


**Answer:**

**Why pods need external storage:**

Pod storage (the container filesystem) is ephemeral. When a pod restarts, crashes, or is rescheduled to a different node, all data written to the container filesystem is gone. For a database this would mean losing all data on every restart.

**Solution: Persistent Volume Claims (PVC) backed by EBS**

**Step 1 — Install the EBS CSI Driver:**

```bash
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster production-cluster \
  --service-account-role-arn arn:aws:iam::123456789:role/AmazonEKS_EBS_CSI_DriverRole
```

**Step 2 — Create a StorageClass for gp3 volumes:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

`WaitForFirstConsumer` is important — it delays EBS volume creation until a pod is scheduled, so the volume is created in the same AZ as the pod. EBS volumes are AZ-specific.

`reclaimPolicy: Retain` means when the PVC is deleted, the EBS volume is NOT deleted. This protects database data from accidental deletion.

**Step 3 — StatefulSet with PVC:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql
spec:
  serviceName: postgresql
  replicas: 1
  template:
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: gp3
      resources:
        requests:
          storage: 50Gi
```

**What happens when the pod restarts:**

Pod crashes → Kubernetes schedules a new pod → new pod starts on the same node (or a node in the same AZ) → the same EBS volume is detached from the old location and reattached to the new node → PostgreSQL starts and finds all its data intact.

{{< /qa >}}

{{< qa num="11" q="Multiple pods on different nodes need to read and write the same shared files simultaneously. What storage solution do you use?" level="advanced" >}}


**Answer:**

**The problem with EBS for this use case:**

EBS volumes have `ReadWriteOnce` access mode — they can only be mounted to one EC2 instance at a time. If Pod A is on Node 1 and Pod B is on Node 2, they cannot both mount the same EBS volume simultaneously. EBS is the wrong tool here.

**Solution: EFS (Amazon Elastic File System) with the EFS CSI Driver**

EFS supports `ReadWriteMany` — any number of pods on any number of nodes can mount the same filesystem simultaneously.

**Step 1 — Install EFS CSI Driver:**

```bash
eksctl create addon \
  --name aws-efs-csi-driver \
  --cluster production-cluster
```

**Step 2 — Create EFS Filesystem and configure security group:**

The EFS filesystem must have mount targets in every AZ your nodes run in. The EFS security group must allow inbound NFS traffic (port 2049) from your worker node security group.

**Step 3 — Create StorageClass and PersistentVolume:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-0123456789abcdef
```

**Step 4 — Multiple pods sharing the same PVC:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```

All pods reference the same PVC — they all see the same files in real-time.

**Real-world use case:**

A video processing platform has 20 worker pods running in parallel. Each pod reads the same input video segments from the shared EFS and writes its processed output back. All pods see each other's output immediately through the shared filesystem.

{{< /qa >}}

{{< qa num="12" q="Your pod running in EKS needs to access an S3 bucket and read from Secrets Manager. A developer hardcoded AWS credentials. What is the right approach?" level="advanced" >}}


**Answer:**

**Why hardcoded credentials are dangerous:**

Credentials in a container image or environment variable can be extracted from the running pod by anyone with `kubectl exec` access. They also appear in pod specs, CI/CD logs, and potentially in git history. There is no auto-rotation — if leaked, they are valid until manually revoked.

**The correct approach: IAM Roles for Service Accounts (IRSA)**

IRSA uses the Kubernetes Service Account mechanism combined with AWS IAM OIDC federation. The pod receives temporary, auto-rotating AWS credentials that are scoped to exactly the permissions it needs.

**Step 1 — Create an IAM Policy with least-privilege permissions:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["secretsmanager:GetSecretValue"],
      "Resource": "arn:aws:secretsmanager:ap-south-1:123456789:secret:my-app-secret-*"
    }
  ]
}
```

**Step 2 — Create IAM Role for the Service Account:**

```bash
eksctl create iamserviceaccount \
  --cluster production-cluster \
  --namespace my-app \
  --name my-app-sa \
  --attach-policy-arn arn:aws:iam::123456789:policy/MyAppPolicy \
  --approve
```

**Step 3 — Use the Service Account in the pod:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
spec:
  template:
    spec:
      serviceAccountName: my-app-sa   # use the annotated service account
      containers:
      - name: app
        image: my-app:v1.0
        # NO environment variables for AWS credentials
        # The AWS SDK automatically discovers credentials from the token file
```

**How it works at runtime:**

When the pod starts, Kubernetes mounts a JWT token file into the pod at a well-known path. When the AWS SDK makes an API call, it exchanges this JWT token with AWS STS for temporary credentials. The credentials are scoped to exactly the IAM policy you defined. They expire every hour and are automatically refreshed. If the pod is compromised and the token is stolen, it expires soon and can only access the specific resources defined in the policy.

{{< /qa >}}

{{< qa num="13" q="How do you implement RBAC in EKS? A junior developer should be able to view pods but not delete them. How do you set this up?" level="advanced" >}}


**Answer:**

**What is RBAC:**

RBAC (Role-Based Access Control) in Kubernetes defines who can do what to which resources. It has four components: Role (permissions within a namespace), ClusterRole (permissions across all namespaces), RoleBinding (assigns a Role to a user/group), ClusterRoleBinding (assigns a ClusterRole to a user/group).

**Step 1 — Create a Role with view-only permissions:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-viewer
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
  # Intentionally omitting: delete, create, update, patch
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
```

**Step 2 — Create a RoleBinding connecting the Role to the developer:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: junior-dev-pod-viewer
  namespace: production
subjects:
- kind: User
  name: junior-developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-viewer
  apiGroup: rbac.authorization.k8s.io
```

**Step 3 — Map the IAM user to the Kubernetes user:**

In EKS, authentication uses IAM. Authorization uses RBAC. You must map the IAM user to a Kubernetes username:

```bash
eksctl create iamidentitymapping \
  --cluster production-cluster \
  --arn arn:aws:iam::123456789:user/junior-developer \
  --username junior-developer \
  --group pod-readers
```

**Verification:**

```bash
# Test what the user can do
kubectl auth can-i get pods --namespace production --as junior-developer
# Output: yes

kubectl auth can-i delete pods --namespace production --as junior-developer
# Output: no
```

**Real-world RBAC structure for a team:**

| Role | Permissions | Assigned to |
|------|-------------|-------------|
| developer | get, list, watch pods/logs/events | All developers |
| deployer | create, update deployments | CI/CD service account |
| operator | everything in namespace | Senior engineers |
| cluster-admin | everything everywhere | Only DevOps leads, via MFA |


{{< /qa >}}

{{< qa num="14" q="Explain the difference between HPA, VPA, and Cluster Autoscaler. When do you use each?" level="advanced" >}}


**Answer:**

**HPA — Horizontal Pod Autoscaler:**

Scales the number of pod replicas based on metrics (CPU utilization, memory, custom metrics). Works at the pod level — adds or removes identical copies of your pod.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-api
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

When CPU across all pods exceeds 70%, HPA adds more pods. When it drops below, HPA removes pods.

Use when: Stateless workloads that can be horizontally scaled. Web servers, API services, workers. Most web applications.

**VPA — Vertical Pod Autoscaler:**

Adjusts the CPU and memory requests/limits on individual pods based on actual usage. Makes the pod bigger or smaller rather than adding more pods.

Use when: Workloads that cannot be horizontally scaled (single-instance databases, legacy monoliths). Or to right-size resource requests automatically instead of guessing. Note: VPA requires a pod restart to apply new resource settings.

**Cluster Autoscaler:**

Adds or removes worker nodes (EC2 instances) from the cluster. Works at the infrastructure level — when pods cannot be scheduled because all nodes are full, Cluster Autoscaler adds a new node.

Use when: You need the underlying node capacity to grow and shrink with workload. Used alongside HPA — HPA adds pods, Cluster Autoscaler adds nodes when those pods cannot be scheduled.

**How they work together in practice:**

```
Traffic spike hits web API
    → HPA detects CPU > 70%
    → HPA increases replicas from 3 to 10
    → 7 new pods are "Pending" (not enough node capacity)
    → Cluster Autoscaler detects Pending pods
    → Cluster Autoscaler adds 2 new EC2 nodes
    → New nodes join cluster
    → Pending pods are scheduled and start running

Traffic drops
    → HPA reduces replicas from 10 to 3
    → 7 pods are terminated
    → 2 nodes are now underutilized
    → Cluster Autoscaler removes the underutilized nodes
```

{{< /qa >}}


{{< qa num="15" q="What is Karpenter and how is it better than Cluster Autoscaler?" level="advanced" >}}


**Answer:**

**Cluster Autoscaler limitations:**

Cluster Autoscaler works with pre-configured Node Groups. You define a node group with a specific instance type (e.g., t3.large) and min/max count. Cluster Autoscaler can only scale within those pre-defined groups. Problems:

- Slow: It waits for pods to be Pending, then waits for AWS to provision the new EC2, then waits for the node to join the cluster and become Ready. Total time: 3–5 minutes typically.
- Inflexible: You must pre-decide which instance types to use. You cannot easily mix m5.large and m5.xlarge in the same node group.
- Over-provisioning: Because scaling is slow, teams over-provision to avoid running out of capacity.

**Karpenter — direct node provisioning:**

Karpenter is an open-source Kubernetes node autoscaler built by AWS. Instead of managing node groups, Karpenter directly calls the EC2 API to launch exactly the right instance type for your pending pods.

**Key advantages:**

Karpenter looks at the pending pod's resource requests and selects the optimal instance type from a wide range. It can mix Spot and On-Demand, different instance families, and different sizes — all in one "NodePool" configuration.

Karpenter is 3–5× faster than Cluster Autoscaler because it makes EC2 API calls directly, bypassing the Auto Scaling Group polling loop.

Karpenter consolidates nodes automatically — if multiple small nodes are underutilized, it moves pods to fewer nodes and terminates the empty ones.

**Real-world example:**

A team had 3 node groups (small, medium, large) managed by Cluster Autoscaler. Scaling a new deployment took 4 minutes. After migrating to Karpenter with a single NodePool accepting m5, m5a, m4 across all sizes, scaling time dropped to 45 seconds and node costs dropped 22% because Karpenter consistently chose the cheapest available instance type that fit the pending pods.

{{< /qa >}}

{{< qa num="16" q="Your application needs to scale based on the number of messages in an SQS queue, not CPU. How do you implement this?" level="advanced" >}}


**Answer:**

**Why HPA alone cannot do this:**

Standard HPA scales on CPU and memory from the Kubernetes Metrics API. SQS queue depth is an external metric that HPA cannot access natively.

**Solution: KEDA (Kubernetes Event-Driven Autoscaling)**

KEDA extends Kubernetes with the ability to scale based on external event sources — SQS queues, Kafka topics, Redis lists, Prometheus metrics, and 50+ other sources.

**Step 1 — Install KEDA:**

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda --namespace keda --create-namespace
```

**Step 2 — Create a ScaledObject for SQS:**

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: sqs-scaler
  namespace: my-app
spec:
  scaleTargetRef:
    name: payment-processor
  minReplicaCount: 0          # Scale to zero when queue is empty!
  maxReplicaCount: 50
  triggers:
  - type: aws-sqs-queue
    metadata:
      queueURL: https://sqs.ap-south-1.amazonaws.com/123456789/payment-queue
      queueLength: "10"       # One pod per 10 messages
      awsRegion: ap-south-1
    authenticationRef:
      name: sqs-auth
```

**What this does:**

- Queue has 0 messages → 0 pods running (scale to zero — saves 100% of compute cost when idle)
- Queue has 100 messages → 10 pods running
- Queue has 500 messages → 50 pods running (max)
- Queue drains to 0 → pods scale back to 0

**Scale-to-zero** is the killer feature. A batch processing job that runs for 4 hours at midnight costs nothing during the other 20 hours. With CPU-based HPA you cannot scale to zero because some minimum pods must always exist to measure CPU.

**Real-world example:**

A fintech company processes bank statements uploaded to S3 via an SQS trigger. With KEDA SQS scaler, the processor fleet scales from 0 to 40 pods when documents are uploaded, processes everything, then scales back to 0. They eliminated the cost of keeping 3 always-on worker pods — saving $800/month for a workload that only needs to run 2 hours per day.

{{< /qa >}}

{{< qa num="17" q="Your new application version was deployed to EKS and is causing errors. Walk me through how you do a rollback?" level="advanced" >}}


**Answer:**

**Kubernetes deployment history:**

Every time you update a Deployment, Kubernetes keeps a revision history (default 10 revisions). You can see and rollback to any previous revision without re-deploying.

**Step 1 — Identify that something is wrong:**

```bash
# Check pod status
kubectl get pods -n production
# Look for pods in CrashLoopBackOff, Error, or OOMKilled state

# Check recent events
kubectl get events -n production --sort-by='.lastTimestamp' | tail -20

# Check pod logs
kubectl logs deployment/my-app -n production --previous
```

**Step 2 — View deployment history:**

```bash
kubectl rollout history deployment/my-app -n production
# REVISION  CHANGE-CAUSE
# 1         Initial deployment
# 2         Updated to v1.1 - added user authentication
# 3         Updated to v1.2 - refactored payment service (CURRENT - broken)
```

**Step 3 — Rollback to previous revision:**

```bash
# Rollback to the immediately previous version
kubectl rollout undo deployment/my-app -n production

# OR rollback to a specific revision
kubectl rollout undo deployment/my-app -n production --to-revision=2

# Watch the rollback progress
kubectl rollout status deployment/my-app -n production
```

**Step 4 — Verify the rollback succeeded:**

```bash
kubectl get pods -n production
kubectl describe deployment my-app -n production | grep Image
```

**How Kubernetes performs the rollback:**

Kubernetes uses the same rolling update strategy as a forward deployment, just in reverse. It creates pods running the old version and terminates pods running the new version, one by one, respecting `maxUnavailable` and `maxSurge` settings. Users see no downtime.

**Annotation for change-cause tracking (best practice):**

```bash
kubectl annotate deployment/my-app kubernetes.io/change-cause="v1.2 - payment service refactor" -n production
```

This populates the CHANGE-CAUSE column in rollout history, making it easy to identify which revision to roll back to during an incident.

{{< /qa >}}

{{< qa num="18" q=" How do you perform a zero-downtime deployment to EKS? Explain all the settings involved." level="advanced" >}}


**Answer:**

**Zero-downtime deployment requires three things working together:**

1. Rolling update strategy with correct maxUnavailable and maxSurge settings
2. Readiness probes so traffic only goes to ready pods
3. PreStop hooks and terminationGracePeriodSeconds so in-flight requests complete

**Complete deployment configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2           # Allow 2 extra pods during update (7 total temporarily)
      maxUnavailable: 0     # Never reduce below 5 running pods
  template:
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: api
        image: my-api:v2.0
        ports:
        - containerPort: 8080

        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
          successThreshold: 1

        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3

        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 10"]
```

**What each setting does:**

`maxSurge: 2` — during the update, Kubernetes can run 2 extra pods above the desired count. This ensures new pods start before old ones terminate.

`maxUnavailable: 0` — never allow any pod to be unavailable during the update. Combined with `maxSurge: 2`, this means: always 5 or more pods serving traffic at all times.

`readinessProbe` — before a new pod receives any traffic, it must pass the readiness check. If the new version crashes on startup, the readiness probe fails, and no traffic is sent to it. The old pods keep running.

`preStop sleep 10` — when a pod receives the SIGTERM termination signal, it waits 10 seconds before starting shutdown. This gives the load balancer time to remove the pod from its target group and stop routing new requests to it. Without this, there is a brief window where the load balancer routes requests to a terminating pod.

`terminationGracePeriodSeconds: 60` — after SIGTERM is sent, Kubernetes waits up to 60 seconds for the pod to finish handling in-flight requests before force-killing it with SIGKILL.


{{< /qa >}}

{{< qa num="19" q="How do you set up centralized logging for all pods in an EKS cluster? Your security team requires logs to be retained for 90 days." level="advanced" >}}


**Answer:**

**Architecture: Fluent Bit → CloudWatch Logs**

Fluent Bit is a lightweight log processor that runs as a DaemonSet — one pod on every node. It reads logs from all containers on its node and ships them to CloudWatch Logs.

**Step 1 — Create IAM permissions for log shipping:**

```bash
eksctl create iamserviceaccount \
  --name fluent-bit \
  --namespace amazon-cloudwatch \
  --cluster production-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
  --approve
```

**Step 2 — Deploy Fluent Bit using the AWS-provided configuration:**

```bash
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml
```

**Step 3 — Configure CloudWatch log retention:**

```bash
# Set 90-day retention on all EKS log groups
aws logs put-retention-policy \
  --log-group-name /aws/containerinsights/production-cluster/application \
  --retention-in-days 90
```

**Log structure in CloudWatch:**

```
/aws/containerinsights/production-cluster/
  /application     → all container stdout/stderr logs
  /dataplane       → kubelet, kube-proxy system logs
  /host            → EC2 instance system logs
  /performance     → CPU, memory, network metrics per pod
```

**Searching logs:**

```bash
# Use AWS CLI for quick searches
aws logs filter-log-events \
  --log-group-name /aws/containerinsights/production-cluster/application \
  --filter-pattern "ERROR" \
  --start-time $(date -d '1 hour ago' +%s)000
```

Or use CloudWatch Log Insights:
```sql
fields @timestamp, kubernetes.pod_name, log
| filter kubernetes.namespace_name = "production"
| filter log like /ERROR/
| sort @timestamp desc
| limit 100
```

{{< /qa >}}


{{< qa num="20" q="Your EKS cluster has performance issues but you don't know which service is causing the problem. How do you set up monitoring?" level="advanced" >}}


**Answer:**

**Solution: Prometheus + Grafana with kube-state-metrics**

**Architecture:**

```
Each pod exposes /metrics endpoint
    → Prometheus scrapes metrics every 15 seconds
    → Grafana queries Prometheus and displays dashboards
    → AlertManager sends PagerDuty/Slack alerts on threshold breach
```

**Step 1 — Install kube-prometheus-stack:**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=your-secure-password
```

This installs Prometheus, Grafana, AlertManager, kube-state-metrics, and node-exporter in one command.

**Step 2 — Access Grafana:**

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
# Open http://localhost:3000
```

Pre-built dashboards include: Kubernetes cluster overview, node CPU/memory, pod resource usage, namespace resource quotas, and persistent volume usage.

**Step 3 — Set up a critical alert:**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: pod-crash-alert
  namespace: monitoring
spec:
  groups:
  - name: pod-alerts
    rules:
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[5m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} is crash looping"
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has been restarting frequently."
```

**Key metrics to monitor:**

| Metric | What it tells you | Alert threshold |
|--------|------------------|-----------------|
| `container_cpu_usage_seconds_total` | Pod CPU consumption | > 80% of limit |
| `container_memory_working_set_bytes` | Pod memory usage | > 85% of limit |
| `kube_pod_status_phase` | Pod state (Pending, Running, Failed) | Any pod Pending > 5 min |
| `kube_deployment_status_replicas_unavailable` | Unhealthy replicas | > 0 for production |
| `node_disk_io_time_seconds_total` | Node disk saturation | > 80% utilization |

---
{{< /qa >}}

{{< qa num="21" q="Explain how you implement a GitOps workflow for EKS deployments using ArgoCD." level="advanced" >}}


**Answer:**

**What is GitOps:**

GitOps is a practice where the entire desired state of your Kubernetes cluster is stored in a Git repository. ArgoCD continuously reconciles what is in Git with what is actually running in the cluster. If someone manually changes something in the cluster, ArgoCD detects the drift and reverts it. If you change something in Git, ArgoCD automatically applies it to the cluster.

**Architecture:**

```
Developer pushes code
    → GitHub Actions CI pipeline runs:
        - Builds Docker image
        - Runs tests
        - Pushes image to ECR
        - Updates image tag in Git (Kubernetes manifests repo)
    → ArgoCD detects change in Git repo
    → ArgoCD applies new manifests to EKS cluster
    → ArgoCD reports sync status (Synced / OutOfSync / Degraded)
```

**Step 1 — Install ArgoCD:**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Step 2 — Create an Application pointing to your Git repo:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-api-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/mycompany/k8s-manifests
    targetRevision: main
    path: apps/web-api/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true       # Delete resources removed from Git
      selfHeal: true    # Revert any manual changes to the cluster
    syncOptions:
    - CreateNamespace=true
```

**Step 3 — Deployment workflow:**

A developer does not run `kubectl apply` to deploy. They submit a Pull Request changing the image tag in the manifests repository. After review and merge, ArgoCD detects the change and automatically deploys within 3 minutes.

**Benefits realized in production:**

- Complete audit trail: every deployment is a git commit with author, timestamp, and PR description
- Easy rollback: `git revert` the commit, ArgoCD deploys the previous version
- No `kubectl` access needed for deployments: developers interact with Git, not the cluster
- Drift detection: if someone accidentally deletes a ConfigMap manually, ArgoCD restores it from Git automatically

{{< /qa >}}

{{< qa num="22" q="Your EKS cluster costs are much higher than expected. How do you identify and reduce the cost?" level="advanced" >}}


**Answer:**

**Step 1 — Enable Cost Allocation Tags and use AWS Cost Explorer:**

Enable the Kubernetes-specific cost allocation tags:

- **eks:cluster-name** — identifies which cluster the cost belongs to
- **kubernetes.io/service-name** — identifies the service

Use Kubecost (open-source) or AWS Container Cost Allocation for per-namespace and per-pod cost breakdown.

**Step 2 — Identify the biggest cost drivers:**

```bash
# Check what instance types your nodes are (GPU instances = very expensive if underused)
kubectl get nodes -o custom-columns=NAME:.metadata.name,TYPE:.metadata.labels.node\\.kubernetes\\.io/instance-type

# Find pods with no resource requests (they may be over-provisioned)
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.containers[].resources.requests == null) | .metadata.name'
```

**Step 3 — Right-size pod resource requests:**

The single largest cost optimization in EKS is fixing oversized resource requests. Kubernetes reserves the requested CPU and memory on the node for each pod — even if the pod uses 10% of what it requested.

```bash
# Use VPA in recommendation mode to see what pods actually use
kubectl get vpa --all-namespaces
```

If a pod requests 2 CPU cores but only uses 200m (0.2 cores), you are paying for 10× more capacity than needed. Fix requests based on actual usage data from Prometheus.

**Step 4 — Use Spot Instances for non-critical workloads:**

```yaml
# In Karpenter NodePool
spec:
  template:
    spec:
      requirements:
      - key: "karpenter.sh/capacity-type"
        operator: In
        values: ["spot", "on-demand"]   # prefer spot, fallback to on-demand
```

Spot Instances save 70–90% on worker node costs. Use them for stateless web services, batch jobs, and dev/test workloads. Keep production databases and stateful workloads on On-Demand.

**Step 5 — Scale to zero with KEDA for batch workloads:**

Any workload that does not need to run 24/7 should scale to zero when idle. This eliminates the cost of always-on pods for intermittent work.

**Real-world impact:**

A company reduced EKS costs by 58% through: fixing oversized resource requests (saved 25%), migrating 60% of workloads to Spot Instances (saved 20%), and implementing KEDA scale-to-zero for batch jobs (saved 13%).


{{< /qa >}}

{{< qa num="23" q="A pod is stuck in **Pending** state. Walk me through diagnosing the root cause?" level="advanced" >}}


**Answer:**

"Pending" means the Kubernetes scheduler cannot find a suitable node to place the pod. This is always a scheduling problem.

**Step 1 — Describe the pod to see scheduler events:**

```
kubectl describe pod <pod-name> -n <namespace>
```

Scroll to the Events section at the bottom. The message will tell you exactly why scheduling failed. Common messages:

**"0/3 nodes are available: 3 Insufficient cpu"**

All nodes have less free CPU than the pod requests. Either:
- The pod's CPU request is too high (a pod requesting 4 CPU when nodes are 2 vCPU each)
- The nodes are genuinely full (Cluster Autoscaler should be adding a new node)

```bash
# Check available resources on each node
kubectl describe nodes | grep -A 5 "Allocated resources"
```

**"0/3 nodes are available: 3 node(s) had taint that the pod didn't tolerate"**

All nodes have a taint that the pod doesn't have a toleration for. Check node taints:
```bash
kubectl describe nodes | grep Taints
```

**"0/3 nodes are available: 3 node(s) didn't match Pod's node affinity"**

The pod has a nodeAffinity or nodeSelector requiring a label that no node has. Check:
```bash
kubectl get nodes --show-labels
```

**"persistentvolumeclaim 'my-pvc' not found" or "waiting for a volume to be created"**

The PVC either doesn't exist or is not bound to a PV. Check:
```bash
kubectl get pvc -n <namespace>
# STATUS should be "Bound" not "Pending"
```

**"exceeded quota"**

The namespace has a ResourceQuota and the new pod would exceed it:
```bash
kubectl describe resourcequota -n <namespace>
```

**Step 2 — Check Cluster Autoscaler logs (if expecting new nodes):**

```bash
kubectl logs -n kube-system deployment/cluster-autoscaler | tail -50
```

If Cluster Autoscaler is failing to add nodes (e.g., insufficient EC2 quota, max node group size reached), the pending pods will never be scheduled.

---
{{< /qa >}}

{{< qa num="24" q="A pod is in `CrashLoopBackOff`. How do you debug it?" level="advanced" >}}


**Answer:**

CrashLoopBackOff means the container started, crashed (exited with a non-zero code), Kubernetes restarted it, it crashed again — and this cycle is repeating. Kubernetes adds an exponential backoff delay between restarts (hence BackOff).

**Step 1 — Get the exit reason:**

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:
```
Last State:     Terminated
  Reason:       OOMKilled      # Out of Memory - increase memory limit
  # OR
  Reason:       Error          # Application crashed - check logs
  # OR  
  Reason:       ContainerCannotRun  # Image entrypoint issue
```

**Step 2 — Read the logs from the previous crashed container:**

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

The `--previous` flag is critical. Without it you get logs from the current (likely empty or starting) container instance. With `--previous` you get the logs from the last crashed run — which contains the actual error that caused the crash.

**Step 3 — Common causes and fixes:**

**OOMKilled:** Container hit its memory limit and was killed.
```bash
# Check memory limit vs actual usage
kubectl top pod <pod-name> -n <namespace>
# Increase the memory limit in the Deployment spec
```

**Application startup error:** Database connection string wrong, missing environment variable, configuration file not found. The application logs will show the specific error.

**Image pull error that becomes CrashLoopBackOff:** Sometimes confused with ImagePullBackOff. Check:
```bash
kubectl describe pod <pod-name> | grep "Failed to pull"
```

**Readiness probe misconfiguration causing repeated restarts:** If the liveness probe path is wrong, Kubernetes kills the container and restarts it repeatedly. Check:
```bash
kubectl describe pod <pod-name> | grep -A 10 "Liveness:"
```

**Step 4 — Debug interactively when you cannot read enough from logs:**

```bash
# Override the entrypoint to get a shell instead of running the crashing app
kubectl run debug-pod \
  --image=my-crashing-app:v1.0 \
  --restart=Never \
  -it \
  --command -- /bin/sh
# Now you can explore the container environment manually
```

---
{{< /qa >}}

{{< qa num="25" q="Pods are running but application response times suddenly spiked to 10 seconds. What do you investigate?" level="advanced" >}}


**Answer:**

This is a performance degradation scenario. Work through these layers systematically.

**Layer 1 — Is it a pod resource issue?**

```bash
kubectl top pods -n production --sort-by=cpu
kubectl top nodes
```

If a pod shows CPU near its limit, it is being CPU-throttled. Kubernetes enforces CPU limits strictly — when a container hits its CPU limit, the kernel throttles it even if the node has spare CPU.

Fix: Increase the CPU limit or reduce CPU request to get better scheduling.

**Layer 2 — Is it a downstream dependency?**

Use kubectl exec to run a test from inside the pod:
```bash
kubectl exec -it <pod-name> -n production -- /bin/sh
# Time a request to the database
time nc -zv postgres-service 5432

# Time an external API call
time curl -o /dev/null -s -w "%{time_total}" https://api.external-service.com/health
```

If the database call takes 8 seconds, the problem is the database not the application pod.

**Layer 3 — Is it a DNS resolution issue?**

Slow DNS lookups in Kubernetes are a common but overlooked performance problem. Each hostname lookup goes through CoreDNS, and if CoreDNS pods are overwhelmed:

```bash
# Check CoreDNS pods
kubectl get pods -n kube-system | grep coredns

# Check CoreDNS logs for errors
kubectl logs -n kube-system -l k8s-app=kube-dns

# Increase CoreDNS replicas if needed
kubectl scale deployment coredns -n kube-system --replicas=4
```

**Layer 4 — Is it a network issue between pods?**

```bash
# Check if there are packet drops at the node level
kubectl debug node/<node-name> -it --image=nicolaka/netshoot
# Inside the debug container:
netstat -s | grep -i "packet"
```

**Layer 5 — Check if a recent deployment caused the regression:**

```bash
kubectl rollout history deployment/my-app -n production
# If a recent rollout correlates with the spike, rollback
kubectl rollout undo deployment/my-app -n production
```
{{< /qa >}}

{{< qa num="26" q="How do you make an EKS application highly available across multiple Availability Zones?" level="advanced" >}}


**Answer:**

**Multi-AZ HA requires correct configuration at every layer:**

**Layer 1 — Spread worker nodes across AZs:**

```bash
eksctl create nodegroup \
  --cluster production-cluster \
  --name workers-multi-az \
  --nodes 6 \
  --nodes-min 3 \
  --nodes-max 12
  # eksctl automatically spreads nodes across AZs by default
```

Verify:
```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,AZ:.metadata.labels.topology\\.kubernetes\\.io/zone
```

**Layer 2 — Spread pods across AZs using Pod Topology Spread Constraints:**

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 6
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: web-api
```

This ensures pods are distributed as evenly as possible across AZs. `maxSkew: 1` means the difference between the AZ with the most pods and the AZ with the least pods can be at most 1. If AZ-a has 3 pods and AZ-b has 0 pods, no new pod can be scheduled in AZ-a until AZ-b catches up.

**Layer 3 — Pod Anti-Affinity to prevent multiple replicas on the same node:**

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - web-api
      topologyKey: "kubernetes.io/hostname"
```

This prevents two replicas of the same pod from landing on the same node. If a node fails, you lose at most one pod, not multiple replicas.

**Layer 4 — PodDisruptionBudget to maintain availability during node maintenance:**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-api-pdb
spec:
  minAvailable: 2          # At least 2 pods must always be running
  selector:
    matchLabels:
      app: web-api
```

When Cluster Autoscaler drains a node or a Managed Node Group upgrade happens, it respects the PDB. If removing a pod would bring the count below `minAvailable: 2`, the drain operation waits until another pod becomes available elsewhere.

{{< /qa >}}

{{< qa num="27" q="Your microservices architecture has 20 services. You need mTLS between all services, traffic management, and distributed tracing. What do you use and why?" level="advanced" >}}


**Answer:**

**The problem without a service mesh:**

With 20 microservices, each service team would need to implement: TLS certificate management and rotation, retry logic, circuit breakers, timeout handling, and distributed tracing instrumentation. That is 20 codebases all implementing the same cross-cutting concerns differently. A security audit would need to verify each independently.

**Solution: Istio service mesh**

Istio injects a sidecar proxy (Envoy) into every pod. All network traffic between pods goes through these sidecar proxies, not directly between containers. This means security, observability, and traffic management features are handled at the infrastructure layer — application code needs zero changes.

**Install Istio on EKS:**

```bash
istioctl install --set profile=production -y
kubectl label namespace production istio-injection=enabled
```

The label `istio-injection=enabled` tells Istio to automatically inject the Envoy sidecar proxy into every new pod in the production namespace.

**Automatic mTLS between all services:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

This single configuration enforces mutual TLS for all pod-to-pod communication in the production namespace. Every service now authenticates the identity of every other service it communicates with. Istio handles certificate generation and rotation automatically.

**Traffic management example — canary deployment:**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: payment-service
spec:
  hosts:
  - payment-service
  http:
  - route:
    - destination:
        host: payment-service
        subset: v1
      weight: 90
    - destination:
        host: payment-service
        subset: v2
      weight: 10
```

Route 10% of traffic to v2 of the payment service while 90% goes to v1. Gradually shift traffic as confidence grows. No changes to application code or Deployments needed.

{{< /qa >}}

{{< qa num="28" q="Design the EKS architecture for a fintech company handling payment processing. It needs PCI-DSS compliance, zero-downtime deployments, and 99.99% availability." level="advanced" >}}


**Answer:**

**Cluster topology:**

Run three separate EKS clusters: production (payment processing), staging (pre-prod testing), and management (CI/CD, monitoring, ArgoCD). Separation prevents a deployment to staging from impacting production infrastructure.

**Network isolation for PCI-DSS:**

Payment processing pods run in a dedicated namespace with a NetworkPolicy that allows NO ingress or egress except to explicitly whitelisted services. No pod in the payment namespace can make arbitrary external HTTP calls.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-service-isolation
  namespace: payment-processing
spec:
  podSelector:
    matchLabels:
      app: payment-processor
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-gateway
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: payment-database
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - port: 53    # DNS only
```

**Node isolation for PCI scope reduction:**

Payment processing pods run on dedicated nodes with a taint so no other workload can be scheduled there. These nodes have enhanced CloudTrail logging and are the only nodes in-scope for PCI audits.

**Zero-downtime deployments:**

Blue-green deployments using Argo Rollouts:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    blueGreen:
      activeService: payment-service-active
      previewService: payment-service-preview
      autoPromotionEnabled: false   # Require manual promotion
      prePromotionAnalysis:
        templates:
        - templateName: payment-success-rate
        args:
        - name: service-name
          value: payment-service-preview
```

New version deploys to the "preview" service. Automated analysis checks error rate and p99 latency. If metrics are healthy, a human approves promotion. If not, automatic rollback. Zero requests lost because traffic switches atomically between the two services.

**99.99% availability (52 minutes downtime/year maximum):**

Multi-AZ node groups across 3 AZs. PodDisruptionBudget ensuring minimum 2 replicas always running. Pod topology spread constraints ensuring replicas are in different AZs AND different nodes. RDS Aurora Global Database for payment data with failover under 1 second. Route 53 health checks with 10-second TTL for DNS failover.

{{< /qa >}}


{{< qa num="29" q="Your EKS cluster is running 500 pods across 20 nodes. You need to upgrade the Kubernetes version from 1.27 to 1.29. How do you do this safely?" level="advanced" >}}


**Answer:**

**Kubernetes upgrade rules:**

You can only upgrade one minor version at a time (1.27 → 1.28, then 1.28 → 1.29). Skipping versions is not supported. Also: the control plane must be upgraded before the worker nodes.

**Phase 1 — Upgrade EKS Control Plane (1.27 → 1.28):**

```bash
# Review the upgrade guide for any breaking changes
# Update one minor version at a time

eksctl upgrade cluster \
  --name production-cluster \
  --version 1.28 \
  --approve
```

This upgrades the managed control plane (API server, etcd, scheduler). Your worker nodes remain on 1.27 during this step. Kubernetes guarantees backward compatibility — 1.27 nodes work with a 1.28 control plane.

**Phase 2 — Check for deprecated API versions in your manifests:**

A very common upgrade failure cause: your Deployments use API versions that were deprecated and removed in the new Kubernetes version.

```bash
# Install kubent (kube no trouble) to find deprecated APIs
kubent
```

Fix any flagged resources before upgrading nodes.

**Phase 3 — Upgrade worker nodes (1.27 → 1.28):**

For Managed Node Groups, AWS does this with a rolling update:

```bash
eksctl upgrade nodegroup \
  --name standard-workers \
  --cluster production-cluster \
  --kubernetes-version 1.28
```

This creates new nodes with 1.28, cordons old nodes, drains them one by one (respecting PodDisruptionBudgets), and terminates them. Your pods are moved to the new nodes during the drain.

**Phase 4 — Upgrade add-ons:**

After node upgrade, update the add-ons to versions compatible with 1.28:
```bash
eksctl update addon --name vpc-cni --cluster production-cluster
eksctl update addon --name coredns --cluster production-cluster
eksctl update addon --name kube-proxy --cluster production-cluster
```

**Phase 5 — Repeat for 1.28 → 1.29.**

**Real-world timeline:** A 20-node cluster upgrade takes approximately 90 minutes per version, mostly waiting for new nodes to provision and old nodes to drain. Total for two minor version upgrades: 3–4 hours.

{{< /qa >}}


{{< qa num="30" q="How do you implement multi-tenancy in an EKS cluster where 5 different teams share the same cluster but must be isolated from each other?" level="advanced" >}}


**Answer:**

**Multi-tenancy in Kubernetes uses namespaces as the isolation boundary, with four enforcement mechanisms:**

**1. Namespace-per-team:**

```bash
kubectl create namespace team-payments
kubectl create namespace team-identity
kubectl create namespace team-analytics
kubectl create namespace team-notifications
kubectl create namespace team-frontend
```

**2. RBAC — each team can only see and manage their own namespace:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-payments-full-access
  namespace: team-payments
subjects:
- kind: Group
  name: payments-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: admin          # admin role scoped to team-payments namespace only
  apiGroup: rbac.authorization.k8s.io
```

The payments team has admin access in `team-payments` namespace. They can create, update, delete pods, deployments, services in their namespace. They cannot see anything in `team-identity` or any other namespace.

**3. ResourceQuota — prevent one team from consuming all cluster resources:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-payments-quota
  namespace: team-payments
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    count/pods: "100"
    count/services: "20"
```

Even if the payments team has a runaway pod that requests unlimited CPU, it cannot consume more than 40 CPU cores or 80 GiB of memory.

**4. NetworkPolicy — teams cannot access each other's services:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: team-isolation
  namespace: team-payments
spec:
  podSelector: {}          # Apply to all pods in this namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: team-payments    # Only allow traffic from same namespace
    - namespaceSelector:
        matchLabels:
          name: kube-system      # Allow system components (CoreDNS)
```

This prevents the team-analytics namespace from directly calling the team-payments API. Cross-team communication must go through defined API contracts, not direct pod-to-pod networking.

**5. LimitRange — prevent pods without resource requests (which consume unlimited resources):**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-payments
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

If a developer in the payments team deploys a pod without resource requests, the LimitRange automatically applies default requests and limits. This prevents uncontrolled resource consumption.

---

## 📝 Quick Reference Cheat Sheet

### EKS Key Commands

```bash
# Cluster info
kubectl cluster-info
eksctl get cluster

# Node status
kubectl get nodes -o wide
kubectl describe node <node-name>

# Pod debugging
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous
kubectl exec -it <pod> -n <ns> -- /bin/sh

# Scaling
kubectl scale deployment <name> --replicas=5
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>

# Resource usage
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top nodes
```

### Pod Status Quick Reference

| Status | Meaning | Common Cause |
|--------|---------|--------------|
| Pending | Not scheduled | Insufficient resources, taint/toleration mismatch, PVC unbound |
| ContainerCreating | Pulling image or mounting volume | Image pull slow, PVC not bound |
| CrashLoopBackOff | App crashes repeatedly | App error, OOMKill, wrong config |
| OOMKilled | Out of memory | Memory limit too low |
| ImagePullBackOff | Cannot pull container image | Wrong image name, ECR permissions, private repo credentials |
| Terminating (stuck) | Pod not cleaning up | Finalizers not removed, storage unmount issue |
| Evicted | Removed from node | Node ran out of disk or memory |

### Storage Access Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| ReadWriteOnce (RWO) | One node, read-write | Databases, EBS |
| ReadOnlyMany (ROX) | Many nodes, read-only | Configuration, static assets |
| ReadWriteMany (RWX) | Many nodes, read-write | Shared uploads, EFS |

### Service Type Summary

| Type | Reachable from | Use case |
|------|---------------|----------|
| ClusterIP | Inside cluster only | Service-to-service |
| NodePort | Outside via node IP:port | Dev/test only |
| LoadBalancer | Outside via AWS LB | Single TCP/UDP services |
| Ingress | Outside via ALB (HTTP/HTTPS) | Multiple web services |

### EKS Upgrade Order

```
1. Update EKS control plane (eksctl upgrade cluster)
2. Update EKS managed add-ons (vpc-cni, coredns, kube-proxy)
3. Update worker nodes (eksctl upgrade nodegroup)
4. Update third-party add-ons (metrics-server, ALB controller, etc.)
```
{{< /qa >}}

---

*40 scenario-based questions covering every major EKS topic — from associate level through solutions architect professional and DevOps engineer level interviews.*
