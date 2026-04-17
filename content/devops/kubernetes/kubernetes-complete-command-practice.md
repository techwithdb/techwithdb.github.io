---
title: "Kubernetes Complete Commands Reference Guide"
description: "A complete, human-friendly kubectl command reference guide covering cluster setup, pods, deployments, services, statefulsets, configmaps, secrets, storage, ingress, HPA, troubleshooting, and output formats — with real examples and explanations."
date: 2026-04-17T10:00:00+05:30
author: "DB"
tags: ["Kubernetes", "kubectl", "DevOps", "K8s", "Containers", "Cheatsheet", "Reference"]
series: "Kubernetes Mastery"
level: "All Levels"
draft: false
---

## Overview

This guide is your go-to reference for every `kubectl` command you will ever need. Rather than just listing commands, each section explains **what the command does**, **when to use it**, and **why it matters** — so you understand the tool, not just copy-paste from a cheatsheet.

Every command uses `ollama-agent` as the example namespace and app name so the examples feel consistent and real.

{{< callout type="tip" title="Golden Rule" >}}
Always include `-n <namespace>` in every `kubectl` command. Without it, kubectl targets the `default` namespace and you will spend time wondering why you cannot find your resources.
{{< /callout >}}

---

## 1. Cluster Setup & Info

Before doing anything in Kubernetes, you need to know the state of your cluster — which version it is running, which context you are in, and which nodes are available. These commands answer those questions.

### kubectl Version & Config

```bash
# Check what version kubectl and the server are running
kubectl version

# Check only the local kubectl client version (no cluster needed)
kubectl version --client
# Show cluster API server address and CoreDNS endpoint
kubectl cluster-info

# Dump full cluster diagnostics — useful for debugging cluster-wide issues
kubectl cluster-info dump
```

### Managing Contexts (Multiple Clusters)

A **context** is a named combination of cluster + user + namespace. If you work with multiple clusters (dev, staging, prod), contexts let you switch between them without changing config files.

```bash
# Show your entire kubeconfig file including all clusters and users
kubectl config view

# List all saved contexts
kubectl config get-contexts

# Show which context is currently active
kubectl config current-context

# Switch to a different cluster/context
kubectl config use-context ollama-agent

# Set the default namespace for your current context
# (so you don't need -n on every command)
kubectl config set-context --current --namespace=ollama-agent

# List all API resource types available in the cluster
kubectl api-resources

# List all API versions the cluster supports
kubectl api-versions
```

### Node Management

Nodes are the worker machines in your cluster. Understanding node status is critical when pods refuse to schedule or when you need to perform maintenance.

```bash
# List all nodes in the cluster
kubectl get nodes

# List nodes with extra info: IP addresses, OS, container runtime
kubectl get nodes -o wide

# Detailed info about a specific node — shows capacity, conditions, and events
kubectl describe node ollama-agent-control-plane

# Show real-time CPU and memory usage per node (requires Metrics Server)
kubectl top nodes

# Mark a node as unschedulable — new pods will not be placed here
kubectl cordon <node>

# Mark a node as schedulable again after maintenance
kubectl uncordon <node>

# Evict all pods from a node safely before maintenance or shutdown
kubectl drain <node> --ignore-daemonsets
```

{{< callout type="tip" title="When to cordon vs drain" >}}
Use `cordon` when you want to stop new pods from landing on a node but keep existing pods running. Use `drain` when you are about to reboot or decommission a node — it evicts existing pods gracefully first.
{{< /callout >}}

### KIND Cluster Commands

KIND (Kubernetes IN Docker) lets you run a full Kubernetes cluster locally inside Docker containers. It is perfect for local development and CI/CD pipelines.

```bash
# Create a default single-node KIND cluster
kind create cluster

# Create a named KIND cluster
kind create cluster --name ollama-agent

# Create a cluster with a custom config file (multi-node, port mappings, etc.)
kind create cluster --name ollama-agent --config kind-config.yaml

# List all KIND clusters on your machine
kind get clusters

# Delete a KIND cluster and all its resources
kind delete cluster --name ollama-agent

# Load a locally built Docker image into KIND
# (KIND cannot pull from localhost — you must load images this way)
kind load docker-image <image> --name <cluster>

# Export the kubeconfig so kubectl can talk to your KIND cluster
kind export kubeconfig --name ollama-agent
```

{{< callout type="info" title="Why kind load docker-image?" >}}
KIND runs inside Docker, so it cannot see images in your local Docker daemon by default. Before running a pod with a locally built image, always run `kind load docker-image` first, otherwise the pod gets an `ImagePullBackOff` error.
{{< /callout >}}

---

## 2. Namespaces

A namespace is a virtual partition inside a Kubernetes cluster. Think of it like separate folders for different teams, environments, or applications — all sharing the same physical cluster but isolated from each other.

```bash
# List all namespaces in the cluster
kubectl get namespaces

# Short form
kubectl get ns

# Create a new namespace
kubectl create namespace ollama-agent

# Delete a namespace AND everything inside it — use with extreme caution
kubectl delete namespace ollama-agent

# Show namespace details including resource quota usage
kubectl describe namespace ollama-agent

# List ALL resource types inside a specific namespace
kubectl get all -n ollama-agent

# List ALL resources across ALL namespaces at once
kubectl get all --all-namespaces

# Short form for --all-namespaces
kubectl get all -A

# Create namespace from a YAML file
kubectl apply -f namespace.yaml
```

### Namespace Flags

These flags are used with almost every kubectl command — not just namespace commands:

| Flag | Description |
|------|-------------|
| `-n ollama-agent` | Target a specific namespace |
| `--namespace=ollama-agent` | Long form of `-n` |
| `--all-namespaces` or `-A` | Target all namespaces at once |

{{< callout type="warn" title="Deleting a namespace is permanent" >}}
`kubectl delete namespace` removes the namespace and every single resource inside it — pods, services, configmaps, secrets, PVCs. There is no recycle bin. Always double-check before running this command.
{{< /callout >}}

---

## 3. Pods

A pod is the smallest deployable unit in Kubernetes — it wraps one or more containers that share network and storage. Most of your day-to-day work involves inspecting, debugging, and interacting with pods.

### Get & Describe Pods

```bash
# List pods in the default namespace
kubectl get pods

# List pods in a specific namespace
kubectl get pods -n ollama-agent

# List pods across ALL namespaces — useful for cluster-wide debugging
kubectl get pods -A

# List pods with extra detail: node name, pod IP, status conditions
kubectl get pods -o wide

# Watch pods in real time — updates as pods start, stop, or change status
kubectl get pods -w

# Show all labels attached to each pod
kubectl get pods --show-labels

# Filter pods by a specific label
kubectl get pods -l app=ollama-agent

# Full details about a pod: events, container states, resource usage, conditions
kubectl describe pod ollama-backend-xxxxx -n ollama-agent

# Export the full pod spec as YAML (useful for copying and modifying)
kubectl get pod ollama-backend-xxxxx -o yaml

# Export the full pod spec as JSON
kubectl get pod ollama-backend-xxxxx -o json
```

### Pod Logs

Logs are your first stop when something goes wrong. Kubernetes keeps logs for the current and one previous container instance per pod.

```bash
# Get the current logs from a pod
kubectl logs ollama-backend-xxxxx -n ollama-agent

# Stream logs in real time (follow mode — like tail -f)
kubectl logs -f ollama-backend-xxxxx -n ollama-agent

# Get logs from a specific container inside a multi-container pod
kubectl logs ollama-backend-xxxxx -c <container> -n ollama-agent

# Get logs from the PREVIOUS instance of a container — critical for CrashLoopBackOff
kubectl logs ollama-backend-xxxxx --previous -n ollama-agent

# Only show the last 100 lines
kubectl logs ollama-backend-xxxxx --tail=100 -n ollama-agent

# Only show logs from the last 1 hour
kubectl logs ollama-backend-xxxxx --since=1h -n ollama-agent

# Stream logs from all pods in a deployment
kubectl logs -f deployment/ollama-agent -n ollama-agent

# Stream logs from all pods in a StatefulSet
kubectl logs -f statefulset/ollama-agent -n ollama-agent
```

{{< callout type="tip" title="CrashLoopBackOff — always check --previous" >}}
When a pod is in `CrashLoopBackOff`, running `kubectl logs` shows logs from the *current* (already crashed) attempt. Use `--previous` to see what the container printed before it crashed — that is where the actual error is.
{{< /callout >}}

### Pod Exec & Shell

Sometimes you need to get inside a running container to inspect files, test connections, or run commands directly.

```bash
# Open an interactive shell inside a pod (for Alpine/slim images)
kubectl exec -it <pod> -n ollama-agent -- /bin/sh

# Open a bash shell (for full Debian/Ubuntu-based images)
kubectl exec -it <pod> -n ollama-agent -- /bin/bash

# Shell into a specific container in a multi-container pod
kubectl exec -it <pod> -c <container> -n ollama-agent -- /bin/sh

# Run a single command and exit (non-interactive)
kubectl exec <pod> -n ollama-agent -- ls /app

# Print all environment variables set in the pod
kubectl exec <pod> -n ollama-agent -- env

# Print the hosts file from inside the pod
kubectl exec <pod> -n ollama-agent -- cat /etc/hosts

# Open mongosh directly in a MongoDB StatefulSet pod
kubectl exec -it statefulset/ollama-agent -n ollama-agent -- mongosh
```

### Pod Lifecycle

```bash
# Create a standalone pod (not managed by a Deployment)
kubectl run ollama-agent --image=<image>

# Create a temporary debug pod that auto-deletes when you exit
kubectl run ollama-agent --image=<image> --rm -it -- /bin/sh

# Delete a pod — if it belongs to a Deployment, it will be recreated automatically
kubectl delete pod ollama-backend-xxxxx -n ollama-agent

# Force delete a pod that is stuck in Terminating state
kubectl delete pod ollama-backend-xxxxx -n ollama-agent --force

# Immediate deletion with no grace period — use only when absolutely necessary
kubectl delete pod ollama-backend-xxxxx -n ollama-agent --grace-period=0

# Get just the pod phase (Running, Pending, Failed, etc.) programmatically
kubectl get pod ollama-backend-xxxxx -n ollama-agent \
  -o jsonpath='{.status.phase}'
```

{{< callout type="tip" title="Temporary debug pods are extremely useful" >}}
`kubectl run debug --image=busybox --rm -it -- /bin/sh` spins up a disposable container inside your cluster. Use it to test DNS resolution, curl internal services, or check network policies — without touching your running application pods.
{{< /callout >}}

---

## 4. Deployments

A Deployment is how you run stateless applications in Kubernetes. It manages a ReplicaSet which in turn manages your pods — ensuring the desired number of replicas are always running and handling rolling updates safely.

### Create & Manage Deployments

```bash
# List all deployments in a namespace
kubectl get deployments -n ollama-agent

# Short form
kubectl get deploy -n ollama-agent

# Detailed info: pod template, strategy, replica count, conditions
kubectl describe deployment ollama-agent -n ollama-agent

# Create or update a deployment from a YAML file
kubectl apply -f deployment.yaml

# Delete a deployment (and all its pods)
kubectl delete deployment ollama-agent -n ollama-agent

# Create a deployment imperatively (quick, no YAML needed)
kubectl create deployment ollama-agent --image=<image>

# Export the current deployment spec as YAML — useful for creating a base template
kubectl get deployment ollama-agent -o yaml -n ollama-agent
```

### Scaling & Rolling Updates

One of Kubernetes' biggest strengths is zero-downtime updates. Rolling updates gradually replace old pods with new ones, ensuring your app stays available throughout.

```bash
# Scale a deployment to 3 replicas
kubectl scale deployment ollama-agent --replicas=3 -n ollama-agent

# Watch the progress of a rolling update in real time
kubectl rollout status deployment/ollama-agent -n ollama-agent

# View the history of all previous rollouts
kubectl rollout history deployment/ollama-agent -n ollama-agent

# Roll back to the previous version (emergency rollback)
kubectl rollout undo deployment/ollama-agent -n ollama-agent

# Roll back to a specific revision number from history
kubectl rollout undo deployment/ollama-agent --to-revision=2 -n ollama-agent

# Restart all pods in a deployment with a rolling restart (no downtime)
kubectl rollout restart deployment/ollama-agent -n ollama-agent

# Pause a rolling update mid-way — inspect the state before continuing
kubectl rollout pause deployment/ollama-agent -n ollama-agent

# Resume a paused rolling update
kubectl rollout resume deployment/ollama-agent -n ollama-agent

# Update the container image in a deployment
kubectl set image deployment/ollama-agent <container>=<image>:<tag> -n ollama-agent
```

{{< callout type="tip" title="Always check rollout status after applying changes" >}}
After running `kubectl apply -f deployment.yaml`, immediately run `kubectl rollout status deployment/ollama-agent -n ollama-agent`. This command blocks and waits until the rollout completes (or fails), giving you confirmation that your change actually landed successfully.
{{< /callout >}}

{{< callout type="info" title="How rollout history works" >}}
Kubernetes keeps a configurable number of previous ReplicaSets (default: 10). Each `kubectl apply` that changes the pod template creates a new revision. You can see all revisions with `rollout history` and jump back to any of them with `rollout undo --to-revision`.
{{< /callout >}}

---

## 5. Services

A Service gives your pods a stable network address. Pods come and go (they are ephemeral), but the Service IP and DNS name stay constant. Services also act as internal load balancers, spreading traffic across all healthy pods that match a label selector.

### Get & Describe Services

```bash
# List all services in a namespace
kubectl get services -n ollama-agent

# Short form
kubectl get svc -n ollama-agent

# List services across all namespaces
kubectl get svc -A

# List services with extra columns: selector labels and port details
kubectl get svc -o wide -n ollama-agent

# Detailed service info: selector, endpoints, port mappings
kubectl describe svc ollama-agent -n ollama-agent

# List endpoints — these are the actual pod IPs behind a service
kubectl get endpoints -n ollama-agent

# Check endpoints for a specific service (tells you if any pods are ready)
kubectl get ep ollama-agent -n ollama-agent

# Create or update a service from a YAML file
kubectl apply -f service.yaml

# Delete a service (pods continue running, just lose external access)
kubectl delete svc ollama-agent -n ollama-agent

# Export service spec as YAML
kubectl get svc ollama-agent -o yaml -n ollama-agent
```

### Service Types Explained

Choosing the wrong service type is a common mistake. Here is exactly when to use each one:

| Type | Accessible From | Use Case |
|------|----------------|----------|
| `ClusterIP` (default) | Inside the cluster only | Internal microservice-to-microservice communication |
| `NodePort` | `NodeIP:30000–32767` from outside | Development access, on-premise without a cloud LB |
| `LoadBalancer` | External internet via cloud LB | Production on AWS/GCP/Azure — creates an ELB automatically |
| `ExternalName` | Inside the cluster | Aliasing an external database DNS name (no proxying) |
| `Headless` (`clusterIP: None`) | Pod IPs returned directly via DNS | StatefulSets where each pod needs its own stable DNS |

{{< callout type="warn" title="LoadBalancer does not work in KIND" >}}
`LoadBalancer` type services require a cloud provider to provision an actual load balancer. In KIND (local development), the external IP will stay `<pending>` forever. Use `kubectl port-forward` instead for local access.
{{< /callout >}}

### Port Forwarding

Port forwarding creates a temporary tunnel from your laptop directly to a pod or service — perfect for testing services without exposing them externally.

```bash
# Forward your local port 8080 to service port 80
kubectl port-forward svc/ollama-agent 8080:80 -n ollama-agent

# Forward directly to a specific pod
kubectl port-forward pod/ollama-agent 8080:80 -n ollama-agent

# Expose on all interfaces — allows access from other machines on your network
# (useful when developing on a remote EC2 instance)
kubectl port-forward svc/ollama-agent 8080:80 -n ollama-agent --address 0.0.0.0

# Run port-forward in the background so your terminal stays free
kubectl port-forward svc/ollama-agent 8080:80 -n ollama-agent &

# Forward the Nginx Ingress controller for local Ingress testing
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
```

---

## 6. StatefulSets

StatefulSets are used for stateful applications that need stable identity — databases, message queues, caches. Unlike Deployments, each pod gets a predictable, stable name (`mongodb-0`, `mongodb-1`) and its own dedicated PersistentVolume.

```bash
# List all StatefulSets
kubectl get statefulsets -n ollama-agent

# Short form
kubectl get sts -n ollama-agent

# Detailed StatefulSet info: pod template, update strategy, volume claims
kubectl describe sts ollama-agent -n ollama-agent

# Create or update a StatefulSet from file
kubectl apply -f statefulset.yaml

# Delete a StatefulSet (pods are also deleted, but PVCs are NOT — data is preserved)
kubectl delete sts ollama-agent -n ollama-agent

# Scale a StatefulSet (scales up from last index, scales down from last index)
kubectl scale sts ollama-agent --replicas=3 -n ollama-agent

# Watch the StatefulSet rollout — it rolls one pod at a time by default
kubectl rollout status sts/ollama-agent -n ollama-agent

# Restart all StatefulSet pods one at a time with a rolling restart
kubectl rollout restart sts/ollama-agent -n ollama-agent

# Open a shell in the first pod of a StatefulSet (index always starts at 0)
kubectl exec -it ollama-mongodb-0 -n ollama-agent -- /bin/sh

# View logs from the first StatefulSet pod
kubectl logs ollama-mongodb-0 -n ollama-agent
```

{{< callout type="info" title="StatefulSet pods are always indexed from 0" >}}
StatefulSet pod names follow the pattern `<statefulset-name>-<ordinal>`. Pods are created in order (0 → 1 → 2) and deleted in reverse order (2 → 1 → 0). The first pod is always index 0, so shell access starts with `kubectl exec -it myapp-0`.
{{< /callout >}}

{{< callout type="tip" title="Deleting a StatefulSet does not delete its PVCs" >}}
This is intentional. When you delete a StatefulSet, its PersistentVolumeClaims stay behind to protect your data. You must delete PVCs manually if you want to reclaim the storage.
{{< /callout >}}

---

## 7. ConfigMaps & Secrets

ConfigMaps and Secrets are how you inject configuration into pods without hardcoding values in your Docker image or Dockerfile.

- **ConfigMap** — for non-sensitive config: feature flags, URLs, log levels, config files
- **Secret** — for sensitive data: passwords, API keys, TLS certificates, tokens

### ConfigMaps

```bash
# List all ConfigMaps in a namespace
kubectl get configmaps -n ollama-agent

# Short form
kubectl get cm -n ollama-agent

# Show the full contents of a ConfigMap
kubectl describe cm ollama-agent -n ollama-agent

# Create or update a ConfigMap from a YAML file
kubectl apply -f configmap.yaml

# Delete a ConfigMap (pods using it continue running with the old value until restarted)
kubectl delete cm ollama-agent -n ollama-agent

# Create a ConfigMap quickly from a key-value pair
kubectl create configmap ollama-agent \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info \
  -n ollama-agent

# Create a ConfigMap from an entire file (the filename becomes the key)
kubectl create configmap ollama-agent --from-file=app.properties -n ollama-agent

# Export ConfigMap as YAML — useful for saving to version control
kubectl get cm ollama-agent -o yaml -n ollama-agent
```

### Secrets

```bash
# List all Secrets (shows type and age but NOT the values)
kubectl get secrets -n ollama-agent

# Show Secret metadata — Kubernetes never shows the actual values here
kubectl describe secret ollama-agent -n ollama-agent

# Create or update a Secret from a YAML file
kubectl apply -f secret.yaml

# Delete a Secret
kubectl delete secret ollama-agent -n ollama-agent

# Export the Secret as YAML — values are base64 encoded, not plain text
kubectl get secret ollama-agent -o yaml -n ollama-agent

# Decode a specific secret value directly from the cluster
kubectl get secret ollama-agent \
  -o jsonpath='{.data.KEY}' \
  -n ollama-agent | base64 --decode

# Create a Secret quickly from a key-value pair
kubectl create secret generic ollama-agent \
  --from-literal=DB_PASSWORD=supersecret \
  -n ollama-agent

# Manually encode a value for use in a secret.yaml file
echo -n 'my-password' | base64

# Manually decode a base64 value from a secret.yaml file
echo 'bXktcGFzc3dvcmQ=' | base64 --decode
```

{{< callout type="danger" title="Never commit secret.yaml with real values to Git" >}}
Even though Secret values are base64-encoded, base64 is NOT encryption — it is just encoding. Anyone who sees the YAML can decode it in seconds. Add `secret.yaml` to your `.gitignore` and use a proper secrets manager (AWS Secrets Manager, HashiCorp Vault, Sealed Secrets) for production.
{{< /callout >}}

{{< callout type="info" title="How to use ConfigMaps and Secrets in a pod" >}}
Both can be injected as environment variables (`envFrom` or `env.valueFrom`) or mounted as files in a volume (`volumeMounts`). File mounting is preferred for large configs and for secrets — it avoids exposing values in process environment listings.
{{< /callout >}}

---

## 8. Persistent Volumes & Storage

Kubernetes manages storage through three objects: **StorageClass** (defines how to provision storage), **PersistentVolume** (the actual storage — like an EBS volume), and **PersistentVolumeClaim** (your pod's request for storage).

In most cloud environments, you only ever create PVCs — Kubernetes automatically provisions the PV using the StorageClass.

```bash
# List PersistentVolumeClaims in a namespace (what your pods requested)
kubectl get pvc -n ollama-agent

# List PersistentVolumes — these are cluster-wide, not namespace-scoped
kubectl get pv

# Full details about a PVC: status, bound volume, access mode, capacity
kubectl describe pvc ollama-agent -n ollama-agent

# Full details about a PV: reclaim policy, storage class, and bound claim
kubectl describe pv ollama-agent

# List all StorageClasses available in the cluster
kubectl get storageclass

# Details about a StorageClass: provisioner, parameters, reclaim policy
kubectl describe storageclass ollama-agent

# Create a PVC from a YAML file
kubectl apply -f pvc.yaml

# Delete a PVC — WARNING: this may delete the underlying data
kubectl delete pvc ollama-agent -n ollama-agent

# List PVCs with extra info: storage class, capacity, access mode
kubectl get pvc -n ollama-agent -o wide
```

### PVC Status Values

| Status | Meaning |
|--------|---------|
| `Pending` | Waiting for a PV to be provisioned or found |
| `Bound` | Successfully connected to a PV — ready to use |
| `Released` | PV was freed but not yet reclaimed/available |
| `Failed` | Automatic provisioning failed |

{{< callout type="danger" title="Deleting a PVC deletes your data" >}}
When you delete a PVC, the underlying PV may also be deleted depending on the `reclaimPolicy` of the StorageClass (`Delete` vs `Retain`). If the policy is `Delete` (default in most cloud environments), your data is gone permanently. Always back up before deleting PVCs.
{{< /callout >}}

---

## 9. Ingress

An Ingress is an HTTP router that sits in front of multiple services, directing requests to the right service based on the URL path or hostname. Instead of creating one LoadBalancer per service (expensive), you create one Ingress controller and route everything through it.

```bash
# List all Ingress resources in a namespace
kubectl get ingress -n ollama-agent

# Short form
kubectl get ing -n ollama-agent

# List Ingress resources across all namespaces
kubectl get ingress -A

# Full Ingress details: rules, TLS config, backend services, events
kubectl describe ingress ollama-agent -n ollama-agent

# Create or update an Ingress from a YAML file
kubectl apply -f ingress.yaml

# Delete an Ingress
kubectl delete ingress ollama-agent -n ollama-agent
```

### Checking the Ingress Controller

The Ingress resource is just a configuration — it does nothing on its own. You also need an **Ingress Controller** (like Nginx, Traefik, or AWS ALB Controller) running in the cluster to actually handle the traffic.

```bash
# Check if the Nginx Ingress Controller pods are running
kubectl get pods -n ingress-nginx

# View the Ingress controller's logs — useful when traffic is not routing correctly
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller

# Check the Ingress controller's service (how external traffic reaches it)
kubectl get svc -n ingress-nginx
```

{{< callout type="tip" title="Test Ingress locally with port-forward" >}}
In KIND or local clusters without a real load balancer, forward the Nginx Ingress controller port to test your Ingress rules:
```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
```
Then access your app at `http://localhost:8080` with the `Host` header set to your Ingress hostname.
{{< /callout >}}

---

## 10. Horizontal Pod Autoscaler (HPA)

The HPA automatically scales your Deployment or StatefulSet up or down based on observed CPU usage, memory usage, or custom metrics. Instead of fixing replicas at 3, you set a range and let Kubernetes decide.

```bash
# List all HPAs and their current vs target metric values
kubectl get hpa -n ollama-agent

# Detailed HPA info: current replicas, metrics, conditions, recent events
kubectl describe hpa ollama-agent -n ollama-agent

# Create or update an HPA from a YAML file
kubectl apply -f hpa.yaml

# Delete an HPA (the Deployment goes back to its fixed replica count)
kubectl delete hpa ollama-agent -n ollama-agent

# Create an HPA imperatively — scale between 2 and 10 pods based on 70% CPU
kubectl autoscale deployment ollama-agent \
  --min=2 --max=10 --cpu-percent=70 \
  -n ollama-agent

# Check current CPU and memory usage per pod (required for HPA decisions)
kubectl top pods -n ollama-agent

# Show resource breakdown per container inside each pod
kubectl top pods -n ollama-agent --containers

# Check resource usage at the node level
kubectl top nodes
```

{{< callout type="warn" title="HPA requires Metrics Server" >}}
If `kubectl get hpa` shows `<unknown>` for the current metric value, the Metrics Server is not installed. Without it, HPA cannot read CPU/memory metrics and will not scale anything.

Install it with:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
{{< /callout >}}

---

## 11. Apply, Delete & Dry Run

These are the core commands for managing resources declaratively — the recommended way to work with Kubernetes.

```bash
# Create or update a resource from a YAML file (idempotent — safe to re-run)
kubectl apply -f deployment.yaml

# Apply all YAML files in a directory at once
kubectl apply -f ./manifests/

# Preview what would change WITHOUT actually applying it (client-side validation)
kubectl apply -f deployment.yaml --dry-run=client

# Preview with server-side validation — catches more errors (webhook validation, etc.)
kubectl apply -f deployment.yaml --dry-run=server

# Show a diff between what is live in the cluster and what is in your file
kubectl diff -f deployment.yaml

# Delete a resource defined in a file
kubectl delete -f deployment.yaml

# Delete all resources defined in a directory
kubectl delete -f ./manifests/

# Delete all pods, services, and deployments in a namespace at once
kubectl delete pod,svc,deploy --all -n ollama-agent

# Create a resource — fails if it already exists (not idempotent, unlike apply)
kubectl create -f deployment.yaml

# Replace an existing resource entirely — resource must already exist
kubectl replace -f deployment.yaml

# Apply a JSON patch to a live resource
kubectl patch deployment ollama-agent -p '{"spec":{"replicas":5}}' -n ollama-agent

# Open a resource in your editor and save to apply changes immediately
kubectl edit deployment ollama-agent -n ollama-agent
```

{{< callout type="tip" title="Prefer apply over create and replace" >}}
`kubectl apply` is idempotent — you can run it multiple times safely. `kubectl create` fails if the resource exists, and `kubectl replace` fails if it does not. In CI/CD pipelines, always use `kubectl apply`.
{{< /callout >}}

{{< callout type="info" title="Use --dry-run=server before applying to production" >}}
`--dry-run=client` only validates YAML syntax locally. `--dry-run=server` sends the request to the API server which runs webhook validations and admission controllers — catching errors that client-side validation misses.
{{< /callout >}}

---

## 12. Labels & Selectors

Labels are key-value pairs attached to any Kubernetes resource. They are the mechanism that Services use to find their pods, that Deployments use to manage their ReplicaSets, and that you use to filter and query resources.

```bash
# Show all labels on every pod
kubectl get pods --show-labels -n ollama-agent

# Filter pods where the label app=backend
kubectl get pods -l app=backend -n ollama-agent

# Filter pods matching multiple labels (AND logic)
kubectl get pods -l app=backend,env=prod -n ollama-agent

# Add a new label to a running pod
kubectl label pod ollama-backend-xxxxx env=prod -n ollama-agent

# Remove a label from a pod (note the trailing dash — that is intentional)
kubectl label pod ollama-backend-xxxxx env- -n ollama-agent

# Get all resource types that share a specific label
kubectl get all -l app=ollama-agent -n ollama-agent

# Filter using set-based selectors (IN operator)
kubectl get pods -l 'app in (frontend,backend)' -n ollama-agent
```

{{< callout type="tip" title="Labels are the glue of Kubernetes" >}}
Never underestimate labels. A Service routes to pods with matching labels. An HPA targets a Deployment by label. `kubectl delete` can target groups by label. Designing a consistent labelling scheme upfront (`app`, `tier`, `env`, `version`) makes everything easier to manage.
{{< /callout >}}

---

## 13. Events & Monitoring

Events tell you what Kubernetes has done recently — scheduling decisions, image pulls, health check failures, OOM kills. They are the first place to look when something is wrong.

```bash
# List all events in a namespace (ordered by object name by default)
kubectl get events -n ollama-agent

# Sort events by time — most recent events appear at the bottom
kubectl get events -n ollama-agent --sort-by='.lastTimestamp'

# Sort by creation timestamp instead of lastTimestamp
kubectl get events -n ollama-agent --sort-by='.metadata.creationTimestamp'

# Events across all namespaces — useful for cluster-wide debugging
kubectl get events -A

# Filter to show only failure events
kubectl get events -n ollama-agent --field-selector reason=Failed

# CPU and memory per pod (requires Metrics Server)
kubectl top pods -n ollama-agent

# CPU and memory broken down per container inside each pod
kubectl top pods -n ollama-agent --containers

# CPU and memory per node
kubectl top nodes
```

{{< callout type="tip" title="Events expire after 1 hour by default" >}}
Kubernetes events are stored in etcd for 1 hour by default. If your pod crashed in the morning and you are investigating in the afternoon, the events may already be gone. Check `kubectl logs --previous` instead for the container output.
{{< /callout >}}

---

## 14. Troubleshooting A to Z

This section covers the most common failure modes you will encounter and exactly how to diagnose and fix each one.

### Pod Not Starting / CrashLoopBackOff

A pod in `CrashLoopBackOff` means the container starts, crashes, Kubernetes restarts it — and the cycle repeats with increasing backoff delays.

```bash
# Step 1 — See why the pod is unhappy
kubectl describe pod <pod-name> -n ollama-agent
# Look at the Events section at the bottom

# Step 2 — Check what the application printed before crashing
kubectl logs <pod-name> --previous -n ollama-agent

# Step 3 — Try running the container interactively to reproduce the error
kubectl run debug --image=<same-image> --rm -it -- /bin/sh
```

### ImagePullBackOff / ErrImagePull

The pod cannot pull the container image. Common causes: wrong image name or tag, private registry with no credentials, or a typo.

```bash
# Check the exact error message
kubectl describe pod <pod-name> -n ollama-agent
# Look for "Failed to pull image" in Events

# For private registries, check if the imagePullSecret is attached
kubectl get pod <pod-name> -o yaml -n ollama-agent | grep imagePullSecret

# Verify the secret exists
kubectl get secret <registry-secret> -n ollama-agent
```

### Pod Stuck in Pending

A `Pending` pod means the scheduler cannot find a suitable node. The reason is always in the Events section.

```bash
# Check why the pod cannot be scheduled
kubectl describe pod <pod-name> -n ollama-agent
# Common reasons: Insufficient CPU/memory, no nodes match nodeSelector/affinity,
# PVC not bound, taint not tolerated

# Check node resource availability
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check if all nodes are schedulable
kubectl get nodes
```

### Service Not Routing Traffic

If you can reach the Service but get no response, the problem is usually the selector not matching any pod labels.

```bash
# Step 1 — Check if any endpoints exist (empty = no pods matched the selector)
kubectl get endpoints ollama-agent -n ollama-agent

# Step 2 — Compare service selector with pod labels
kubectl get svc ollama-agent -o yaml -n ollama-agent | grep selector -A 5
kubectl get pods --show-labels -n ollama-agent

# Step 3 — Check pod readiness (not-ready pods are removed from endpoints)
kubectl get pods -n ollama-agent
```

### Ingress Not Working

```bash
# Step 1 — Check the Ingress resource is configured correctly
kubectl describe ingress ollama-agent -n ollama-agent

# Step 2 — Verify the backend service and port exist
kubectl get svc -n ollama-agent

# Step 3 — Check Ingress controller logs for routing errors
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller --tail=50

# Step 4 — Verify the Ingress controller is running
kubectl get pods -n ingress-nginx
```

### PVC Stuck in Pending

```bash
# Check what the PVC is waiting for
kubectl describe pvc <pvc-name> -n ollama-agent
# Common causes: no StorageClass defined, StorageClass does not exist,
# no available PVs for static provisioning

# Check available storage classes
kubectl get storageclass

# Check if a matching PV exists (static provisioning)
kubectl get pv
```

### HPA Showing `<unknown>` Metrics

```bash
# Confirm Metrics Server is installed and running
kubectl get pods -n kube-system | grep metrics-server

# Check that pods have resource requests defined
kubectl get pod <pod> -o yaml -n ollama-agent | grep -A 5 resources

# Describe the HPA for more detail on why metrics are unavailable
kubectl describe hpa ollama-agent -n ollama-agent
```

### StatefulSet Pod Not Starting

```bash
# Check the pod status (StatefulSets wait for pod N before creating pod N+1)
kubectl get pods -n ollama-agent

# Check if the PVC for this pod is bound
kubectl get pvc -n ollama-agent

# Check the pod events
kubectl describe pod <statefulset>-0 -n ollama-agent
```

### Node Not Ready

```bash
# Check node conditions
kubectl describe node <node-name>
# Look for: MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable

# Check node resource usage
kubectl top node <node-name>

# Check if kubelet is running on the node (SSH into node)
# systemctl status kubelet
```

### Force Delete Stuck Resources

Sometimes resources get stuck in `Terminating` state and refuse to delete because of stuck **finalizers**.

```bash
# Force delete a stuck pod
kubectl delete pod <pod-name> -n ollama-agent --force --grace-period=0

# Remove a finalizer from a stuck resource (replace with your resource type)
kubectl patch pvc <pvc-name> -n ollama-agent \
  -p '{"metadata":{"finalizers":[]}}' \
  --type=merge
```

{{< callout type="danger" title="Force deleting and removing finalizers can cause data inconsistency" >}}
Finalizers exist to allow resources to clean up before being deleted — for example, a PVC finalizer ensures the storage provider can detach and clean up the volume. Removing finalizers forcibly skips this cleanup. Only do this as a last resort when you are certain the resource is safe to remove.
{{< /callout >}}

---

## 15. Output Formats & JSONPath

kubectl supports multiple output formats so you can get exactly the data you need — for human reading, scripts, or piping into other tools.

### Output Format Flags

```bash
# Output as YAML — perfect for saving or modifying a resource spec
kubectl get pods -o yaml -n ollama-agent

# Output as JSON — useful for scripting and jq processing
kubectl get pods -o json -n ollama-agent

# Wide output — adds extra columns like node name and pod IP
kubectl get pods -o wide -n ollama-agent

# Output only the resource names — great for piping into delete commands
kubectl get pods -o name -n ollama-agent
```

### JSONPath — Extract Specific Fields

JSONPath lets you pull a single field out of the JSON output without parsing the full spec.

```bash
# Get the IP address of a specific pod
kubectl get pod ollama-agent -o jsonpath='{.status.podIP}' -n ollama-agent

# Get all container names in a pod
kubectl get pod ollama-agent \
  -o jsonpath='{.spec.containers[*].name}' \
  -n ollama-agent

# Get all node names in the cluster
kubectl get nodes -o jsonpath='{.items[*].metadata.name}'

# Get the name of the first pod matching a label selector
kubectl get pods -n ollama-agent -l app=backend \
  -o jsonpath='{.items[0].metadata.name}'

# Get all ClusterIP addresses of services in a namespace
kubectl get svc -n ollama-agent \
  -o jsonpath='{.items[*].spec.clusterIP}'

# Get image tag of first container in a deployment
kubectl get deployment ollama-agent -n ollama-agent \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

{{< callout type="tip" title="Combine JSONPath with shell loops for bulk operations" >}}
```bash
# Delete all pods with a specific label (using jsonpath to get names)
kubectl get pods -l app=backend -n ollama-agent \
  -o jsonpath='{.items[*].metadata.name}' | \
  xargs kubectl delete pod -n ollama-agent
```
{{< /callout >}}

---

## 16. Quick Reference Cheatsheet

A condensed summary of the most frequently used commands — print this and keep it on your desk.

### Cluster & Context

```bash
kubectl config get-contexts                          # List all contexts
kubectl config use-context <name>                    # Switch cluster
kubectl config set-context --current --namespace=<ns> # Set default namespace
kubectl get nodes -o wide                            # Node status with IPs
kubectl top nodes                                    # Node resource usage
```

### Namespace

```bash
kubectl get ns                                       # List namespaces
kubectl create ns <name>                             # Create namespace
kubectl get all -n <ns>                              # Everything in a namespace
```

### Pods

```bash
kubectl get pods -n <ns> -o wide                     # Pods with IPs and nodes
kubectl get pods -n <ns> -w                          # Watch pods live
kubectl describe pod <pod> -n <ns>                   # Full pod detail + events
kubectl logs <pod> -n <ns> --previous                # Logs before crash
kubectl logs -f <pod> -n <ns>                        # Stream live logs
kubectl exec -it <pod> -n <ns> -- /bin/sh            # Shell into pod
kubectl run debug --image=busybox --rm -it -- /bin/sh # Temp debug pod
kubectl delete pod <pod> -n <ns> --force             # Force delete stuck pod
```

### Deployments

```bash
kubectl apply -f deployment.yaml                     # Create/update deployment
kubectl rollout status deploy/<name> -n <ns>         # Watch rollout
kubectl rollout undo deploy/<name> -n <ns>           # Rollback
kubectl rollout restart deploy/<name> -n <ns>        # Rolling restart
kubectl scale deploy/<name> --replicas=3 -n <ns>     # Scale
kubectl set image deploy/<name> <c>=<img> -n <ns>    # Update image
```

### Services & Networking

```bash
kubectl get svc -n <ns>                              # List services
kubectl get ep -n <ns>                               # List endpoints (pod IPs)
kubectl port-forward svc/<name> 8080:80 -n <ns>      # Local tunnel to service
kubectl describe svc <name> -n <ns>                  # Service detail + endpoints
```

### Config & Secrets

```bash
kubectl get cm -n <ns>                               # List ConfigMaps
kubectl get secrets -n <ns>                          # List secrets
kubectl create secret generic <n> --from-literal=K=V -n <ns>  # Quick secret
kubectl get secret <n> -o jsonpath='{.data.KEY}' -n <ns> | base64 -d  # Decode
```

### Storage

```bash
kubectl get pvc -n <ns>                              # List PVCs
kubectl get pv                                       # List PVs (cluster-wide)
kubectl get storageclass                             # List StorageClasses
kubectl describe pvc <name> -n <ns>                  # PVC details + events
```

### Troubleshooting

```bash
kubectl get events -n <ns> --sort-by='.lastTimestamp'  # Sorted events
kubectl get events -n <ns> --field-selector reason=Failed  # Failures only
kubectl top pods -n <ns>                             # Pod resource usage
kubectl top nodes                                    # Node resource usage
kubectl describe pod <pod> -n <ns>                   # Events and conditions
kubectl logs <pod> --previous -n <ns>                # Pre-crash logs
kubectl get endpoints -n <ns>                        # Check service routing
```

### Cleanup

```bash
kubectl delete -f <file.yaml>                        # Delete from file
kubectl delete pod,svc,deploy --all -n <ns>          # Delete everything in namespace
kubectl delete ns <name>                             # Delete namespace + all contents
```

---

{{< callout type="tip" title="Build muscle memory with these 10 commands" >}}
If you memorise only ten commands, make them these:

1. `kubectl get pods -n <ns> -o wide` — always start here
2. `kubectl describe pod <pod> -n <ns>` — events tell the story
3. `kubectl logs <pod> --previous -n <ns>` — see what killed it
4. `kubectl exec -it <pod> -n <ns> -- /bin/sh` — get inside
5. `kubectl get events -n <ns> --sort-by='.lastTimestamp'` — cluster timeline
6. `kubectl rollout status deploy/<name> -n <ns>` — confirm deployments landed
7. `kubectl rollout undo deploy/<name> -n <ns>` — emergency rollback
8. `kubectl apply -f <file> --dry-run=server` — preview before applying
9. `kubectl get endpoints -n <ns>` — debug service routing
10. `kubectl top pods -n <ns>` — spot resource pressure
{{< /callout >}}
