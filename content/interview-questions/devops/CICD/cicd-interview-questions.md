---
title: "CI/CD & DevOps Interview Questions & Answers (2026)"
description: "40+ CI/CD and DevOps interview questions covering pipelines, Jenkins, GitOps, deployment strategies, monitoring, and SRE — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["cicd", "devops", "jenkins", "gitops", "interview"]
tool: "cicd"
level: "All Levels"
question_count: 40
draft: "false"
---

<div class="qa-list">

{{< qa num="1" q="What is DevOps and why is it important?" level="basic" >}}
**Ans:**

**DevOps** is a culture and set of practices that brings together software Development (Dev) and IT Operations (Ops) to shorten the development lifecycle and deliver high-quality software continuously.

**Core principles:**
- **Collaboration** — Dev and Ops teams work together, not in silos
- **Automation** — CI/CD, IaC, automated testing
- **Continuous feedback** — monitoring, logging, observability
- **Continuous improvement** — retrospectives, blameless post-mortems

**Why it matters:**
- Faster releases (days vs months)
- Higher quality (automated testing catches bugs early)
- Lower failure rate (smaller, more frequent deployments)
- Faster recovery (automation enables quick rollbacks)

```
Traditional: Dev writes code → hands off to Ops → Ops deploys → blame each other
DevOps: Dev + Ops collaborate → automate everything → deploy confidently daily
```
{{< /qa >}}

{{< qa num="2" q="What are CI and CD? How are they different?" level="basic" >}}
**Ans:**

| Term | Full Name | What it does |
|------|-----------|-------------|
| **CI** | Continuous Integration | Automatically builds and tests code when developers push changes |
| **CD** | Continuous Delivery | Automatically deploys to staging; manual gate for production |
| **CD** | Continuous Deployment | Fully automated — every passing build goes to production |

```
Code push
    │
    ▼
CI: Build → Unit Tests → Integration Tests → Static Analysis
    │
    ▼
CD (Delivery): Deploy to Staging → Smoke Tests → [Manual Approval] → Production
                                                       ↑
                                                  (Delivery stops here)
CD (Deployment): Deploy to Staging → Tests → Automatically → Production
```

**Key difference:** Continuous Delivery requires a human to approve production deployment. Continuous Deployment does it automatically.
{{< /qa >}}

{{< qa num="3" q="Explain the DevOps lifecycle." level="basic" >}}
**Ans:**

The DevOps lifecycle is an **infinite loop** of continuous improvement:

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ↑                                                            │
  └────────────────────────────────────────────────────────────┘
```

| Phase | Activities | Tools |
|-------|-----------|-------|
| **Plan** | Sprint planning, backlog | Jira, Trello |
| **Code** | Write code, code review | Git, GitHub, GitLab |
| **Build** | Compile, package | Maven, Gradle, npm |
| **Test** | Unit, integration, security | JUnit, Selenium, SonarQube |
| **Release** | Artifact management | Nexus, JFrog Artifactory |
| **Deploy** | Push to environments | Ansible, Kubernetes, Helm |
| **Operate** | Run and manage infrastructure | Terraform, Kubernetes |
| **Monitor** | Observe, alert, feedback | Prometheus, Grafana, ELK |
{{< /qa >}}

{{< qa num="4" q="How does a CI/CD pipeline work? Explain with a real scenario." level="intermediate" >}}
**Ans:**

**Real scenario: Node.js app deployed to Kubernetes**

```
Developer pushes code to GitHub
          │
          ▼ (webhook triggers)
Jenkins/GitHub Actions
    │
    Stage 1: Checkout
    ├── git clone repo
    │
    Stage 2: Build
    ├── npm install
    ├── npm run build
    │
    Stage 3: Test
    ├── npm run test (unit tests)
    ├── npm run test:integration
    │
    Stage 4: Code Quality
    ├── SonarQube analysis
    ├── ESLint scan
    │
    Stage 5: Security Scan
    ├── trivy image scan
    ├── OWASP dependency check
    │
    Stage 6: Build & Push Docker Image
    ├── docker build -t myapp:${GIT_COMMIT_SHORT} .
    ├── docker push registry/myapp:${GIT_COMMIT_SHORT}
    │
    Stage 7: Deploy to Staging
    ├── helm upgrade --install myapp ./chart --set image.tag=${GIT_COMMIT_SHORT}
    ├── Wait for rollout
    ├── Run smoke tests
    │
    Stage 8: Manual Approval (for prod)
    ├── Notify Slack → team approves
    │
    Stage 9: Deploy to Production
    ├── helm upgrade myapp ./chart -n production --set image.tag=${GIT_COMMIT_SHORT}
    │
    Stage 10: Notify
    └── Slack/email notification with status
```

```groovy
// Jenkinsfile example
pipeline {
  agent any
  stages {
    stage('Build') { steps { sh 'npm install && npm run build' } }
    stage('Test')  { steps { sh 'npm test' } }
    stage('Docker Build') {
      steps {
        sh "docker build -t myapp:${GIT_COMMIT[0..7]} ."
        sh "docker push myapp:${GIT_COMMIT[0..7]}"
      }
    }
    stage('Deploy Staging') {
      steps { sh "helm upgrade --install myapp ./chart --set image.tag=${GIT_COMMIT[0..7]}" }
    }
    stage('Prod Approval') {
      steps { input message: 'Deploy to production?', ok: 'Deploy' }
    }
    stage('Deploy Prod') {
      steps { sh "helm upgrade myapp ./chart -n prod --set image.tag=${GIT_COMMIT[0..7]}" }
    }
  }
}
```
{{< /qa >}}

{{< qa num="5" q="What is blue-green deployment? When do you use it?" level="intermediate" >}}
**Ans:**

**Blue-Green** runs two identical production environments. Traffic switches instantly from old (blue) to new (green) with zero downtime. Blue stays as instant rollback.

```
                    ┌─── Blue (v1) ←── traffic off
Load Balancer ──────┤
                    └─── Green (v2) ←── all traffic
```

**Flow:**
1. Deploy v2 to Green environment (no user traffic yet)
2. Run smoke tests on Green
3. Switch load balancer: all traffic → Green
4. Green is now "live production"
5. Blue stays running (instant rollback capability)
6. After confidence → decommission Blue or keep for next release

```bash
# Kubernetes Blue-Green with service selector switch
# Blue deployment: label app: myapp-blue
# Green deployment: label app: myapp-green

# Service points to blue:
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'

# After green is ready and tested:
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback (instant):
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
```

**When to use:** Mission-critical apps where downtime is unacceptable. Requires 2x infrastructure cost.
{{< /qa >}}

{{< qa num="6" q="What is canary deployment? When do you use it?" level="intermediate" >}}
**Ans:**

**Canary deployment** gradually shifts a small percentage of traffic to the new version, monitoring for issues before full rollout.

```
v1 (stable) ←── 90% of users
v2 (canary) ←── 10% of users (canary group)

If no issues after 30 min → increase to 50% → 100%
If issues → route 100% back to v1
```

```yaml
# Kubernetes Canary with nginx-ingress
# Stable service:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-stable
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - backend:
          service:
            name: myapp-v1
            port:
              number: 80
---
# Canary ingress:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% traffic
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - backend:
          service:
            name: myapp-v2
            port:
              number: 80
```

**When to use:** Large-scale systems where you want risk-minimized rollouts with real user validation.

**Blue-Green vs Canary:**
- Blue-Green: instant, full switch, high resource cost
- Canary: gradual, lower risk, more complex, lower resource cost
{{< /qa >}}

{{< qa num="7" q="Jenkins pipeline stuck at a stage — how do you troubleshoot?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check the console output
# Jenkins UI → Job → Build Number → Console Output

# Step 2: Look for timeout issues
# Stage hung on: sh script, ssh, API call?
# Add timeouts to stages:
stage('Deploy') {
  options { timeout(time: 10, unit: 'MINUTES') }
  steps { sh './deploy.sh' }
}

# Step 3: Check if it's waiting for input
# Jenkins UI → Pending Input Actions → Check for approval buttons

# Step 4: Check agent/node connectivity
# Jenkins → Manage Jenkins → Nodes → Check if agent is online

# Step 5: Check for hung processes on agent
ssh jenkins-agent
ps aux | grep <process-from-pipeline>
# Kill the hung process if needed

# Step 6: Check Jenkins system log
# Jenkins → Manage Jenkins → System Log → All Jenkins Logs

# Step 7: Check build workspace on agent
ls -la /var/lib/jenkins/workspace/<job-name>/

# Step 8: If waiting on external service (Nexus, K8s, AWS):
# - Check if the service is reachable
curl http://nexus:8081/nexus/
kubectl cluster-info

# Step 9: Kill the build if truly stuck
# Jenkins UI → Build → ⬛ (Stop)
# Or via CLI:
curl -X POST http://jenkins:8080/job/myjob/1/stop \
  --user admin:token
```
{{< /qa >}}

{{< qa num="8" q="How do you optimize CI/CD pipeline performance?" level="intermediate" >}}
**Ans:**

```groovy
// 1. Run independent stages in parallel
stage('Parallel Tests') {
  parallel {
    stage('Unit Tests')        { steps { sh 'npm test' } }
    stage('Integration Tests') { steps { sh 'npm run test:integration' } }
    stage('Lint')              { steps { sh 'npm run lint' } }
  }
}

// 2. Cache dependencies
// Jenkins with Docker:
stage('Build') {
  steps {
    sh 'docker build --cache-from myapp:cache -t myapp:latest .'
  }
}

// 3. Shallow git clone (faster for large repos)
git url: 'https://github.com/org/repo.git', depth: 1

// 4. Use incremental builds
// Only rebuild if relevant files changed
when {
  changeset "src/**"
}

// 5. Use smaller/purpose-built agent images
// Agent with only needed tools, not a 3GB full-featured image

// 6. Artifact caching (Maven, npm, pip)
// ~/.m2, node_modules — mount as Docker volume or use cache action

// 7. Fail fast — run unit tests before expensive integration tests

// 8. Use build agents close to deployment targets (same AZ)
```

```bash
# Typical time breakdown:
# Slow pipeline: 45 min
# After parallel stages: 20 min
# After caching + shallow clone: 8 min
```
{{< /qa >}}

{{< qa num="9" q="What is GitOps and how is it different from traditional CI/CD?" level="intermediate" >}}
**Ans:**

**GitOps** is a practice where Git is the single source of truth for both application code AND infrastructure/deployment configuration. Changes to production happen only via Git commits.

```
Traditional CI/CD:            GitOps:
Code → CI → push → CD         Code → CI → Git commit → GitOps Agent → Cluster
(push model)                   (pull model — Flux/ArgoCD watches Git)
```

| Feature | Traditional CI/CD | GitOps |
|---------|------------------|--------|
| Trigger | CI pipeline pushes to cluster | Agent pulls from Git |
| Secrets | In CI/CD system | Sealed Secrets or Vault |
| Rollback | Re-run pipeline | `git revert` |
| Audit | CI logs | Git history |
| Drift | Can drift unnoticed | Agent continuously reconciles |
| Tools | Jenkins, GitLab CI | ArgoCD, Flux |

**GitOps workflow with ArgoCD:**
```bash
# 1. Developer pushes new image tag to values.yaml in gitops-repo
# 2. ArgoCD detects Git change (polls every 3 min or webhook)
# 3. ArgoCD compares Git state to cluster state
# 4. ArgoCD applies difference (deploys new version)
# 5. If someone manually changes cluster, ArgoCD reverts it

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Create application
argocd app create myapp \
  --repo https://github.com/org/gitops-repo \
  --path apps/myapp \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production
```
{{< /qa >}}

{{< qa num="10" q="How do you secure a CI/CD pipeline?" level="intermediate" >}}
**Ans:**

```
Security layers:

1. Source Code Security
   - Branch protection rules (no direct push to main)
   - Required code reviews (at least 2 approvals)
   - Signed commits (GPG)
   - SAST: SonarQube, Semgrep, Checkmarx

2. Build Security
   - Use specific image tags (not :latest) for build agents
   - Pin dependency versions (package-lock.json, requirements.txt)
   - OWASP dependency check
   - Software Bill of Materials (SBOM) generation

3. Container Security
   - Trivy/Snyk scan before push
   - Non-root user in containers
   - Read-only filesystem
   - Sign images (Docker Content Trust / cosign + sigstore)
   - Use private registry (ECR, GCR, Nexus)

4. Secret Management
   - Never hardcode secrets in Jenkinsfile or .gitlab-ci.yml
   - Use Jenkins Credentials, GitHub Secrets, Vault
   - Rotate credentials regularly

5. Deployment Security
   - Least privilege IAM roles for CI/CD pipeline
   - Use OIDC (not long-lived access keys) for AWS
   - Network policies in Kubernetes
   - Audit log all deployments

6. Infrastructure
   - Run CI agents in isolated environments
   - Ephemeral build agents (spin up, use, terminate)
   - Separate deployment IAM roles per environment
```

```bash
# GitHub Actions OIDC (no AWS access keys needed)
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
    aws-region: us-east-1
```
{{< /qa >}}

{{< qa num="11" q="Explain how Prometheus and Grafana work together." level="basic" >}}
**Ans:**

```
Application/Service
    │
    │ exposes /metrics endpoint (HTTP)
    ▼
Prometheus (scrapes /metrics every 15s, stores time-series data)
    │
    │ PromQL queries
    ▼
Grafana (connects to Prometheus as data source, visualizes dashboards)
    │
    ▼
Alertmanager (receives alerts from Prometheus rules, sends to Slack/PagerDuty)
```

**Components:**

| Component | Role |
|-----------|------|
| **Prometheus** | Pull-based metrics collector and time-series DB |
| **Exporters** | Expose metrics: node_exporter (OS), kube-state-metrics (K8s), blackbox_exporter (HTTP) |
| **Alertmanager** | Routes alerts to notification channels |
| **Grafana** | Visualization and dashboarding |

```bash
# Default ports:
# Prometheus: 9090
# Grafana: 3000
# Alertmanager: 9093
# Node Exporter: 9100

# Example PromQL query in Grafana:
# CPU usage %:
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# HTTP error rate:
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```
{{< /qa >}}

{{< qa num="12" q="What is SRE and how is it different from DevOps?" level="intermediate" >}}
**Ans:**

| Aspect | DevOps | SRE |
|--------|--------|-----|
| Origin | Cultural movement (collaboration) | Engineering discipline (Google, 2003) |
| Goal | Faster software delivery | Reliability and scalability |
| Focus | Automation, CI/CD | SLOs, SLAs, error budgets |
| Team | Dev + Ops merged | Dedicated reliability engineers |
| Metrics | Deployment frequency, MTTR | SLI/SLO/SLA, error budgets |

**SRE key concepts:**
```
SLI (Service Level Indicator): Measurable metric (e.g., request latency p99)
SLO (Service Level Objective): Target (e.g., 99.9% requests < 500ms)
SLA (Service Level Agreement): Contract with consequences (e.g., 99.5% uptime or money back)
Error Budget: 100% - SLO = how much downtime you can "spend" (e.g., 0.1% = 43.8 min/month)
```

```bash
# SRE principle: if error budget is burned, stop feature releases
# and focus on reliability until budget recovers

# Error budget calculation:
# SLO: 99.9% → 0.1% budget → 43.8 minutes downtime allowed per month
# If we've used 40 minutes → only 3.8 min left → freeze risky deployments
```
{{< /qa >}}

{{< qa num="13" q="What is observability and how is it different from monitoring?" level="intermediate" >}}
**Ans:**

| Aspect | Monitoring | Observability |
|--------|-----------|---------------|
| Focus | Known failure modes | Unknown/unexpected failures |
| Approach | Pre-defined dashboards | Explore and query freely |
| Answer | "Is this component up?" | "Why is this slow for these users?" |
| Data | Metrics | Metrics + Logs + Traces (the 3 pillars) |

**Three Pillars of Observability:**

```
Metrics  → What is happening? (CPU=90%, error_rate=5%)
Logs     → What happened? (ERROR: connection refused to db:5432)
Traces   → Where is the bottleneck? (request took 3s, 2.5s in DB query)
```

**Tools:**

| Pillar | Open Source | Commercial |
|--------|------------|-----------|
| Metrics | Prometheus, Grafana | Datadog, New Relic |
| Logs | ELK Stack, Loki | Splunk, Datadog Logs |
| Traces | Jaeger, Zipkin, Tempo | Datadog APM, Honeycomb |

```bash
# Distributed tracing example with OpenTelemetry
# Each service adds trace context to requests
# Jaeger UI shows: service A → service B (200ms) → service C (2.3s) ← bottleneck
```
{{< /qa >}}

{{< qa num="14" q="How do you handle rollback in production?" level="intermediate" >}}
**Ans:**

**Strategy depends on deployment type:**

```bash
# 1. Kubernetes rollback (fastest)
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=3
kubectl rollout status deployment/myapp   # Monitor

# 2. Helm rollback
helm rollback myapp 3      # Roll back to revision 3
helm history myapp         # Show release history

# 3. Docker Swarm rollback
docker service rollback myapp

# 4. Blue-Green rollback
# Switch load balancer back to blue environment
aws elbv2 modify-listener \
  --listener-arn <arn> \
  --default-actions Type=forward,TargetGroupArn=<blue-tg-arn>

# 5. Database rollback (most complex)
# Run down-migration scripts
flyway repair
flyway migrate -target=V2__baseline
# OR restore from pre-deployment snapshot

# 6. Pipeline-level rollback
# Trigger previous successful deployment pipeline
# Jenkins: Build with parameters → pass old image tag

# 7. GitOps rollback
git revert <commit-hash>
git push   # ArgoCD auto-deploys previous version
```

**Golden rule:** Always test rollback procedure before it's needed in a real incident.
{{< /qa >}}

{{< qa num="15" q="How do you perform disaster recovery and backups in DevOps?" level="advanced" >}}
**Ans:**

**RTO and RPO:**
```
RPO (Recovery Point Objective): How much data loss is acceptable? (e.g., max 1 hour)
RTO (Recovery Time Objective): How fast must you recover? (e.g., < 4 hours)
```

**Backup strategy:**

```bash
# Databases
# RDS automated backups (daily + transaction logs → point-in-time recovery)
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00"

# Manual snapshot before any major change
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier pre-migration-snapshot

# EBS snapshots (automated with DLM)
aws dlm create-lifecycle-policy ...

# Application files → S3 (with versioning + cross-region replication)
aws s3api put-bucket-versioning --bucket myapp-data --versioning-configuration Status=Enabled
```

**Disaster Recovery Tiers:**

| Tier | Strategy | RTO | RPO | Cost |
|------|---------|-----|-----|------|
| Backup & Restore | Restore from S3/snapshots | Hours | Hours | $ |
| Pilot Light | Core services running in DR region | ~1 hour | Minutes | $$ |
| Warm Standby | Scaled-down replica always running | Minutes | Seconds | $$$ |
| Multi-site Active-Active | Full capacity in 2+ regions | Seconds | Near-zero | $$$$ |

```bash
# Test DR regularly:
# 1. Terminate primary DB → confirm failover to read replica
# 2. Restore snapshot to new instance
# 3. Failover Route 53 to DR region
# Document recovery runbooks and drill quarterly
```
{{< /qa >}}

{{< qa num="16" q="How do you design logging and monitoring for a microservices architecture?" level="advanced" >}}
**Ans:**

```
Each microservice → structured JSON logs → centralized collector → storage → query/alert

Architecture:
Microservices (stdout logs)
    │
Fluentd/Fluent Bit (DaemonSet in K8s) ← collects from all pods
    │
    ├──→ Elasticsearch/OpenSearch (search & store logs)
    │           ↓
    │        Kibana (visualize, search logs)
    │
    └──→ Loki (cost-efficient, Grafana-native)
                ↓
            Grafana (query with LogQL)
```

**Structured logging (JSON) — crucial for filtering:**
```json
{
  "timestamp": "2024-01-01T10:00:00Z",
  "level": "ERROR",
  "service": "payment-api",
  "trace_id": "abc123",
  "user_id": "u-456",
  "message": "Payment gateway timeout",
  "duration_ms": 5001
}
```

**Correlation across services (distributed tracing):**
```bash
# Each request gets a unique trace_id
# Passed in HTTP headers: X-Trace-ID
# All services log it → you can search all logs for a single user request

# Jaeger/Zipkin shows the full call chain:
# API Gateway → Auth Service → Payment Service → Database
```

**Alerting rules:**
```yaml
# Prometheus alert for high error rate
groups:
- name: microservices
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.01
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "{{ $labels.service }} error rate > 1%"
```
{{< /qa >}}

{{< qa num="17" q="How do you reduce MTTR (Mean Time To Recovery) in production systems?" level="advanced" >}}
**Ans:**

**MTTR = Detection time + Diagnosis time + Fix time + Verification time**

```bash
# Reduce Detection Time:
# - Alerting thresholds tight enough to catch issues before users notice
# - Synthetic monitoring (ping endpoints every 30s from multiple regions)
# - Real User Monitoring (RUM) — detect impact immediately
# - On-call rotation with <5 min response SLA

# Reduce Diagnosis Time:
# - Runbooks for every alert (link in alert itself)
# - Pre-built dashboards per service
# - Distributed tracing (find bottleneck in seconds, not hours)
# - Structured logs with correlation IDs (grep single trace_id)

# Reduce Fix Time:
# - One-click rollback scripts ready
# - Feature flags (disable bad feature without redeploy)
# - Circuit breakers (auto-isolate failing service)
# - Pre-approved emergency changes

# Reduce Verification Time:
# - Smoke tests that run in < 2 minutes
# - Synthetic traffic patterns

# Cultural practices:
# - Blameless post-mortems → learn without fear
# - GameDays / Chaos Engineering (simulate failures safely)
# - Runbooks kept up-to-date and reviewed quarterly
```
{{< /qa >}}

{{< qa num="18" q="Explain the Maven lifecycle." level="basic" >}}
**Ans:**

Maven has three built-in lifecycles. The most important is the **default (build) lifecycle**:

| Phase | What happens |
|-------|-------------|
| `validate` | Validates project is correct and all info is available |
| `compile` | Compiles source code (`src/main/java`) |
| `test` | Runs unit tests (JUnit, TestNG) |
| `package` | Packages compiled code into JAR/WAR |
| `verify` | Runs integration tests, checks package quality |
| `install` | Installs package into local Maven repository (`~/.m2`) |
| `deploy` | Deploys to remote repository (Nexus, Artifactory) |

```bash
# Run up to package phase:
mvn package

# Run up to install:
mvn install

# Skip tests:
mvn package -DskipTests

# Run specific phase:
mvn compile

# Clean previous build before build:
mvn clean package
```

Other lifecycles:
- `clean` — removes `target/` directory
- `site` — generates project documentation
{{< /qa >}}

{{< qa num="19" q="What is immutable vs mutable infrastructure?" level="intermediate" >}}
**Ans:**

| Aspect | Mutable Infrastructure | Immutable Infrastructure |
|--------|----------------------|-------------------------|
| Updates | SSH in and patch the server | Replace old server with new pre-built image |
| Drift | Common (servers diverge over time) | None (every server from same image) |
| Debugging | Easier (server is long-lived) | Harder (must reproduce in new image) |
| Rollback | Manual, risky | Fast (launch old AMI/image) |
| Snowflakes | Common | Prevented |
| Tools | Ansible, Chef, Puppet | Packer + Terraform, Docker + K8s |

**Immutable approach:**
```bash
# 1. Build AMI with Packer (bake everything in)
packer build web-server.json
# Creates: ami-1234567890abcdef0

# 2. Terraform deploys instances from that AMI
resource "aws_launch_template" "web" {
  image_id      = "ami-1234567890abcdef0"
  instance_type = "t3.micro"
}

# 3. New version? Build new AMI, update Terraform, roll out
# Old AMI stays for rollback
```

**Containers are inherently immutable** — you never patch a running container; you build a new image and redeploy.
{{< /qa >}}

{{< qa num="20" q="If there is an unexpected configuration change in AWS, how do you detect who made the change and when?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: AWS CloudTrail (primary audit tool)
# CloudTrail records every API call with: who, what, when, from where

# Search CloudTrail for recent changes
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceName,AttributeValue=i-1234567890abcdef0 \
  --start-time 2024-01-01T00:00:00Z

# Query via CloudWatch Logs Insights (if CloudTrail → CloudWatch enabled):
fields userIdentity.arn, eventName, eventTime, requestParameters
| filter eventSource = "ec2.amazonaws.com"
| filter eventName = "ModifyInstanceAttribute"
| sort eventTime desc
| limit 20

# Method 2: AWS Config
# Tracks resource configuration history
aws configservice get-resource-config-history \
  --resource-type AWS::EC2::Instance \
  --resource-id i-1234567890abcdef0

# Method 3: AWS Config Rules + SNS
# Set up Config rule to detect unauthorized changes
# Alert via SNS → email/Slack

# Method 4: EventBridge (CloudWatch Events) → Lambda → Slack
# Rule: "When EC2 SecurityGroup is modified → trigger Lambda → post to Slack"

# Prevention:
# - Enable CloudTrail across all regions + all accounts (org trail)
# - Store CloudTrail logs in central immutable S3 bucket (MFA Delete enabled)
# - Set up Config conformance packs for compliance monitoring
```
{{< /qa >}}

</div>
