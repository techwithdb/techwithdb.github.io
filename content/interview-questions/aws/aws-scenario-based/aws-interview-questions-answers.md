---
title: "AWS Interview Questions & Answers (2026)"
description: "50+ AWS interview questions and answers covering EC2, S3, IAM, VPC, ALB, RDS, Lambda, EBS, EKS, and more — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["aws", "cloud", "interview", "devops"]
tool: "aws"
level: "All Levels"
question_count: 50
draft: "false"
---

<div class="qa-list">

{{< qa num="1" q="What is cloud computing? Explain IaaS, PaaS, and SaaS." level="basic" >}}
**Ans:**

**Cloud computing** delivers computing resources (servers, storage, networking, software) over the internet on-demand with pay-as-you-go pricing.

| Model | Full Form | Who manages infra? | Example |
|-------|-----------|-------------------|---------|
| **IaaS** | Infrastructure as a Service | You manage OS, apps; Provider manages hardware | AWS EC2, Azure VMs |
| **PaaS** | Platform as a Service | Provider manages OS + runtime; You manage apps | AWS Elastic Beanstalk, Heroku |
| **SaaS** | Software as a Service | Provider manages everything | Gmail, Salesforce, Office 365 |

```
IaaS → Rent servers (you install everything)
PaaS → Rent a platform (you just deploy code)
SaaS → Use ready-made software (nothing to manage)
```
{{< /qa >}}

{{< qa num="2" q="What is the difference between availability zone and region?" level="basic" >}}
**Ans:**

| Concept | Definition |
|---------|-----------|
| **Region** | A geographic area (e.g., `us-east-1`, `ap-south-1`). Contains 2–6 AZs. |
| **Availability Zone (AZ)** | An isolated data center within a region (e.g., `us-east-1a`, `us-east-1b`). Has independent power, networking, cooling. |

**Why it matters:**
- Deploy across multiple AZs → survive data center failure
- Deploy across multiple Regions → survive regional disaster

```
Region: us-east-1 (N. Virginia)
  ├── AZ: us-east-1a
  ├── AZ: us-east-1b
  └── AZ: us-east-1c
```
{{< /qa >}}

{{< qa num="3" q="What is the difference between Security Groups and Network ACLs in AWS?" level="basic" >}}
**Ans:**

| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| Level | Instance (EC2) level | Subnet level |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow inbound AND outbound explicitly) |
| Rules | Allow only | Allow and Deny |
| Rule evaluation | All rules evaluated | Rules evaluated in order (lowest number first) |
| Applies to | Specific EC2 instances | All resources in the subnet |

**Example:**
| ----------------| ---------------------------------------------------------------------------|
| Security Group: |  Allow port 80 inbound → response traffic on port 80 auto-allowed outbound | 
| NACL: | Allow port 80 inbound → must also explicitly allow ephemeral ports (1024-65535) outbound |

{{< /qa >}}

{{< qa num="4" q="What are IAM Roles vs IAM Policies? When should you use each?" level="basic" >}}
**Ans:**

| Concept | Definition | When to use |
|---------|-----------|-------------|
| **IAM Policy** | JSON document defining permissions (Allow/Deny actions on resources) | Attach to users, groups, or roles |
| **IAM Role** | An identity that can be assumed by a service, user, or application. Has policies attached. | When an AWS service (EC2, Lambda) needs to access other AWS services |

**Example flow:**
```
EC2 instance → assumes IAM Role "S3ReadRole"
IAM Role has policy: { "Effect": "Allow", "Action": "s3:GetObject", "Resource": "*" }
EC2 can now read S3 without hardcoded credentials
```

```bash
# Check what role an EC2 has
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
{{< /qa >}}

{{< qa num="5" q="Explain the difference between S3 Standard, S3 IA, and Glacier." level="basic" >}}
**Ans:**

| Class | Use Case | Access | Cost |
|-------|---------|--------|------|
| **S3 Standard** | Frequently accessed data | Instant | Highest storage cost |
| **S3 Standard-IA** (Infrequent Access) | Accessed monthly, but needs fast retrieval | Instant | Lower storage, retrieval fee |
| **S3 One Zone-IA** | Infrequent, non-critical data | Instant | Cheapest IA, single AZ |
| **S3 Glacier Instant** | Archives accessed a few times a year | Milliseconds | Very low storage |
| **S3 Glacier Flexible** | Long-term archive | Minutes to hours | Very low |
| **S3 Glacier Deep Archive** | Regulatory/compliance archive | Up to 12 hours | Lowest cost |

```bash
# Move objects between classes using lifecycle rules
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration file://lifecycle.json
```
{{< /qa >}}

{{< qa num="6" q="How does Auto Scaling work with Application Load Balancer? Explain a real scenario." level="intermediate" >}}
**Ans:**

**Architecture:**
```
Internet → ALB → Target Group → Auto Scaling Group → EC2 instances
```

**How it works:**
1. ALB distributes traffic across healthy EC2 instances in the Target Group
2. Auto Scaling Group monitors CloudWatch metrics (CPU, request count)
3. When CPU > 70% for 5 minutes → Scale Out: launch new EC2s, register them with ALB
4. When CPU < 30% for 10 minutes → Scale In: terminate EC2s, deregister from ALB

**Real Scenario:**
```
E-commerce site during a flash sale:
- Normal: 2 EC2 instances
- Sale starts → CPU spikes to 90%
- ASG triggers scale-out → launches 4 more instances
- ALB starts routing traffic to new instances (health check passes)
- Sale ends → CPU drops → ASG scales back to 2 instances
```

```bash
# Create scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name scale-out \
  --scaling-adjustment 2 \
  --adjustment-type ChangeInCapacity
```
{{< /qa >}}

{{< qa num="7" q="How would you secure secrets in a CI/CD pipeline running on AWS?" level="intermediate" >}}
**Ans:**

```bash
# Option 1: AWS Secrets Manager (recommended)
# Store secret
aws secretsmanager create-secret \
  --name prod/db-password \
  --secret-string '{"password":"myS3cret"}'

# Retrieve in pipeline script
SECRET=$(aws secretsmanager get-secret-value \
  --secret-id prod/db-password \
  --query SecretString --output text)

# Option 2: AWS SSM Parameter Store (cheaper)
aws ssm put-parameter \
  --name /myapp/db-password \
  --value "myS3cret" \
  --type SecureString

# Retrieve
aws ssm get-parameter \
  --name /myapp/db-password \
  --with-decryption \
  --query Parameter.Value --output text

# Option 3: Jenkins Credentials Manager
# Store in Jenkins → Manage Credentials → use in pipeline:
# withCredentials([string(credentialsId: 'db-pass', variable: 'DB_PASS')]) {}
```

**Best Practices:**
- Never hardcode secrets in Jenkinsfile, `.env`, or Dockerfile
- Use IAM roles (not access keys) for EC2/ECS/Lambda
- Rotate secrets regularly via Secrets Manager rotation
- Audit secret access with CloudTrail
{{< /qa >}}

{{< qa num="8" q="What is the difference between ECS and EKS?" level="intermediate" >}}
**Ans:**

| Feature | ECS (Elastic Container Service) | EKS (Elastic Kubernetes Service) |
|---------|-------------------------------|----------------------------------|
| Orchestrator | AWS proprietary | Kubernetes (open-source) |
| Learning curve | Lower | Higher |
| Portability | AWS only | Portable (runs on any K8s) |
| Control plane | Fully managed (free) | Managed, but costs ~$0.10/hr |
| Launch types | EC2 or Fargate | EC2, Fargate, or on-prem |
| Config format | Task Definitions (JSON) | YAML manifests |
| Use case | Simple containers, AWS-native teams | Complex apps, multi-cloud, K8s expertise |

```bash
# ECS: Deploy a task
aws ecs run-task --cluster my-cluster --task-definition my-task

# EKS: Standard kubectl
kubectl apply -f deployment.yaml
```
{{< /qa >}}

{{< qa num="9" q="What are the steps to troubleshoot a failing EC2 instance in production?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check instance status in AWS Console
# EC2 → Instances → Status Checks (System reachability, Instance reachability)

# Step 2: Check System Log
# EC2 → Actions → Monitor → Get System Log

# Step 3: Try to connect
ssh -i key.pem ec2-user@<public-ip>

# Step 4: If SSH fails, check Security Group allows port 22
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Step 5: Check if instance has public IP / Elastic IP
aws ec2 describe-instances --instance-ids i-xxxxx

# Step 6: Check CloudWatch metrics
# CPU, Network In/Out, StatusCheckFailed

# Step 7: Check application logs (if accessible via SSM Session Manager)
sudo journalctl -xe
sudo tail -f /var/log/messages

# Step 8: If truly stuck — stop/start (not reboot) to move to new host
aws ec2 stop-instances --instance-ids i-xxxxx
aws ec2 start-instances --instance-ids i-xxxxx

# Step 9: If OS is corrupt, detach EBS, attach to another instance, inspect
```
{{< /qa >}}

{{< qa num="10" q="How would you design a highly available architecture on AWS for a web application?" level="advanced" >}}
**Ans:**

```
Internet
    │
Route 53 (DNS with health checks)
    │
CloudFront (CDN + DDoS protection via WAF)
    │
Application Load Balancer (Multi-AZ)
    │
Auto Scaling Group
    ├── EC2 in AZ-a (Private Subnet)
    └── EC2 in AZ-b (Private Subnet)
    │
RDS Multi-AZ (Primary in AZ-a, Standby in AZ-b)
    │
ElastiCache (Redis cluster mode)
    │
S3 (Static assets, backups)
```

**Key components:**
- **Route 53** with failover routing for DNS-level HA
- **ALB** across minimum 2 AZs
- **ASG** with min=2 instances across AZs
- **RDS Multi-AZ** — automatic failover in 60-120 seconds
- **ElastiCache** cluster for session/cache HA
- **VPC** with public subnets (ALB) and private subnets (EC2, RDS)
- **NAT Gateway** in each AZ for private subnet internet access
- **CloudWatch + SNS** for alerting
{{< /qa >}}

{{< qa num="11" q="Your application running on EC2 suddenly starts failing health checks behind an ALB. How would you troubleshoot?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check Target Group health in ALB console
# ALB → Target Groups → Targets → Check health status & reason

# Step 2: Check the health check configuration
# Protocol, Port, Path, Healthy/Unhealthy thresholds, Timeout

# Step 3: SSH to EC2 and test the health check endpoint locally
curl http://localhost:80/health

# Step 4: Check application logs
tail -f /var/log/app/application.log
journalctl -u myapp -f

# Step 5: Check if the app is listening on the expected port
ss -tulnp | grep :80
netstat -tulnp | grep :80

# Step 6: Check Security Group allows ALB to reach EC2
# ALB SG → EC2 SG: allow inbound on app port (e.g., 8080)

# Step 7: Check EC2 disk/memory (app may be crashing)
df -h
free -m
top

# Step 8: Restart the application service
sudo systemctl restart myapp
```
{{< /qa >}}

{{< qa num="12" q="A production RDS database is running out of storage. What immediate actions would you take?" level="intermediate" >}}
**Ans:**

```bash
# Immediate Actions:

# Step 1: Enable Storage Autoscaling (if not already on)
# AWS Console → RDS → Modify → Storage Autoscaling → Enable
# Or via CLI:
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --max-allocated-storage 200 \
  --apply-immediately

# Step 2: Manually increase allocated storage (no downtime for most engines)
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --allocated-storage 200 \
  --apply-immediately

# Step 3: Identify storage hogs on the DB side
# Connect to RDS and run:
# MySQL:
SELECT table_schema, SUM(data_length + index_length)/1024/1024 AS 'Size (MB)'
FROM information_schema.TABLES
GROUP BY table_schema
ORDER BY 2 DESC;

# Step 4: Purge old data or move to archive table
# Step 5: Set up CloudWatch alarm on FreeStorageSpace metric
```
{{< /qa >}}

{{< qa num="13" q="How would you implement centralized logging for multiple AWS accounts?" level="advanced" >}}
**Ans:**

```
Account A → CloudWatch Logs → Kinesis Data Firehose ──┐
Account B → CloudWatch Logs → Kinesis Data Firehose ──┤→ Central S3 / OpenSearch
Account C → CloudWatch Logs → Kinesis Data Firehose ──┘
                                                        ↓
                                                    Grafana / Kibana
```

**Implementation steps:**
1. Enable **CloudWatch Logs** on all services in each account
2. Create a **Kinesis Data Firehose** in a central logging account
3. Use **cross-account IAM roles** to allow accounts to put logs to Firehose
4. Firehose delivers to **S3** (long-term) and **OpenSearch** (search/query)
5. Use **AWS Organizations** + CloudTrail organization trail for audit logs
6. Set up **Grafana** or **Kibana** dashboards for visualization

```bash
# Create organization CloudTrail (all accounts → central S3)
aws cloudtrail create-trail \
  --name org-trail \
  --s3-bucket-name central-audit-logs \
  --is-organization-trail \
  --is-multi-region-trail
```
{{< /qa >}}

{{< qa num="14" q="Your Lambda function is timing out frequently. How do you troubleshoot and optimize it?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check current timeout setting
aws lambda get-function-configuration \
  --function-name my-function \
  --query 'Timeout'

# Step 2: Increase timeout (max 15 minutes)
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 300

# Step 3: Check CloudWatch Logs for WHERE it's timing out
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "Task timed out"

# Step 4: Check for cold start issues
# Add X-Ray tracing:
aws lambda update-function-configuration \
  --function-name my-function \
  --tracing-config Mode=Active

# Step 5: Common causes and fixes:
# - DB connection pool exhaustion → use RDS Proxy
# - External API calls slow → add timeouts, caching
# - Large payload → increase memory (also increases CPU)
# - VPC cold start → enable Provisioned Concurrency
# - Recursive calls → add exit conditions

# Step 6: Increase memory (improves CPU allocation)
aws lambda update-function-configuration \
  --function-name my-function \
  --memory-size 512
```
{{< /qa >}}

{{< qa num="15" q="How do you check if EC2 has correct IAM role access?" level="intermediate" >}}
**Ans:**

```bash
# From inside the EC2 instance:

# Check attached IAM role
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Get credentials for the role
ROLE_NAME=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE_NAME

# Test specific permissions
aws sts get-caller-identity          # Who am I?
aws s3 ls s3://my-bucket/            # Can I access S3?
aws ec2 describe-instances           # Can I describe EC2?

# From AWS CLI (on your laptop):
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].IamInstanceProfile'

# Check effective permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789:role/MyRole \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/*
```
{{< /qa >}}

{{< qa num="16" q="How do you check EBS volume attachment and size?" level="basic" >}}
**Ans:**

```bash
# From inside EC2:

# List block devices and sizes
lsblk
# Output shows: NAME, SIZE, TYPE, MOUNTPOINT

# Check filesystem and usage
df -hT

# Check disk details
sudo fdisk -l

# Using AWS CLI (from your laptop or EC2 with IAM role):
aws ec2 describe-volumes \
  --filters Name=attachment.instance-id,Values=i-1234567890abcdef0

# Check which volume is attached to which device
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[].Instances[].BlockDeviceMappings'
```
{{< /qa >}}

{{< qa num="17" q="How do you increase EC2 disk size safely?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Modify EBS volume size in AWS Console or CLI (no downtime)
aws ec2 modify-volume \
  --volume-id vol-xxxxxxxx \
  --size 100

# Step 2: Wait for modification to complete
aws ec2 describe-volumes-modifications \
  --volume-ids vol-xxxxxxxx

# Step 3: SSH into EC2 and extend the partition/filesystem
# Check current state
lsblk

# Grow partition (for nvme/xvd devices)
sudo growpart /dev/xvda 1
# or
sudo growpart /dev/nvme0n1 1

# Step 4: Resize the filesystem
# For ext4:
sudo resize2fs /dev/xvda1

# For xfs (Amazon Linux 2 default):
sudo xfs_growfs /

# Step 5: Verify
df -h
lsblk
```
{{< /qa >}}

{{< qa num="18" q="How do you create a snapshot of an EBS volume?" level="basic" >}}
**Ans:**

```bash
# Method 1: AWS CLI
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxx \
  --description "My backup snapshot $(date +%Y-%m-%d)"

# Method 2: AWS Console
# EC2 → Volumes → Select volume → Actions → Create Snapshot

# Automate snapshots with DLM (Data Lifecycle Manager):
aws dlm create-lifecycle-policy \
  --description "Daily EBS Snapshots" \
  --state ENABLED \
  --execution-role-arn arn:aws:iam::xxxx:role/AWSDataLifecycleManagerDefaultRole \
  --policy-details file://policy.json

# List snapshots
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[*].{ID:SnapshotId,Size:VolumeSize,Date:StartTime}'
```
{{< /qa >}}

{{< qa num="19" q="How do you mount an unmounted EBS volume?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Attach volume to EC2 from AWS Console/CLI
aws ec2 attach-volume \
  --volume-id vol-xxxxxxxx \
  --instance-id i-xxxxxxxx \
  --device /dev/sdf

# Step 2: Verify attachment inside EC2
lsblk
# New device appears (e.g., xvdf or nvme1n1)

# Step 3: Check if it has a filesystem
sudo file -s /dev/xvdf
# If output: "/dev/xvdf: data" → no filesystem, format it:
sudo mkfs -t ext4 /dev/xvdf

# Step 4: Create mount point and mount
sudo mkdir -p /data
sudo mount /dev/xvdf /data

# Step 5: Make mount persistent across reboots
# Get UUID
sudo blkid /dev/xvdf

# Add to /etc/fstab
echo "UUID=xxxx-xxxx  /data  ext4  defaults,nofail  0  2" | sudo tee -a /etc/fstab

# Verify fstab
sudo mount -a
df -h
```
{{< /qa >}}

{{< qa num="20" q="How do you troubleshoot 'read-only file system' errors?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Identify the filesystem
df -h
mount | grep "ro,"   # Look for read-only mounts

# Step 2: Check system logs for errors
dmesg | grep -i "remount\|read-only\|error\|ext4"
journalctl -xe | grep -i "filesystem\|read-only"

# Step 3: Try remounting read-write
sudo mount -o remount,rw /

# Step 4: Check for filesystem errors
# Unmount first (use liveCD/rescue mode for root fs):
sudo umount /dev/xvdf
sudo fsck -y /dev/xvdf    # Check and auto-fix errors
sudo mount /dev/xvdf /data

# Step 5: For EBS — check AWS CloudWatch for I/O errors
# EBS volume may have failed → create new from snapshot

# Step 6: Check /etc/fstab for incorrect options
cat /etc/fstab

# Root cause: filesystem errors cause kernel to remount as read-only to prevent corruption
```
{{< /qa >}}

{{< qa num="21" q="If an S3 bucket accidentally gets deleted, how would you recover the data?" level="intermediate" >}}
**Ans:**

```bash
# Prevention is key. Recovery options depend on what you had enabled:

# Option 1: S3 Versioning (best protection)
# If versioning was enabled, deleted objects have delete markers
# List delete markers:
aws s3api list-object-versions \
  --bucket my-deleted-bucket \
  --query 'DeleteMarkers[*]'

# Remove delete markers to restore:
aws s3api delete-object \
  --bucket my-bucket \
  --key myfile.txt \
  --version-id <delete-marker-version-id>

# Option 2: S3 Replication backup
# If cross-region/cross-account replication was configured,
# copy objects back from the destination bucket

# Option 3: Restore from backup
aws s3 sync s3://backup-bucket/ s3://recreated-bucket/

# Option 4: If no versioning/backup existed → data is permanently lost

# Best Practice to prevent this:
# - Enable versioning
# - Enable MFA Delete
# - Enable S3 Replication
# - Set bucket policy to deny DeleteBucket
```
{{< /qa >}}

{{< qa num="22" q="How would you implement centralized logging for multiple AWS services on one account?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Enable CloudWatch Logs for each service:
# EC2: Install CloudWatch Agent
sudo yum install amazon-cloudwatch-agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Lambda: Auto-logs to /aws/lambda/<function-name>

# ALB/ELB: Enable access logs
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --attributes Key=access_logs.s3.enabled,Value=true \
               Key=access_logs.s3.bucket,Value=my-alb-logs

# Step 2: Create Log Groups with retention
aws logs put-retention-policy \
  --log-group-name /myapp/production \
  --retention-in-days 30

# Step 3: Subscribe logs to Kinesis or Lambda for aggregation
# Step 4: Use CloudWatch Insights for querying
aws logs start-query \
  --log-group-name /myapp/production \
  --start-time $(date -d "1 hour ago" +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/'
```
{{< /qa >}}

{{< qa num="23" q="How do you confirm health of EC2 target behind ALB?" level="basic" >}}
**Ans:**

```bash
# Method 1: AWS Console
# ALB → Target Groups → Select TG → Targets tab → Check Status column

# Method 2: AWS CLI
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:region:account:targetgroup/...

# Method 3: From inside EC2 — test health check endpoint locally
curl http://localhost:80/health
curl http://localhost:8080/

# Method 4: Check registration status
aws elbv2 describe-target-health \
  --target-group-arn <arn> \
  --query 'TargetHealthDescriptions[*].{ID:Target.Id,Port:Target.Port,Health:TargetHealth.State,Reason:TargetHealth.Reason}'

# Common unhealthy reasons:
# - Target.FailedHealthChecks: App not responding on health check port/path
# - Target.NotRegistered: Not in target group
# - Target.NotInUse: Target group not attached to ALB
```
{{< /qa >}}

{{< qa num="24" q="What is Amazon Route 53?" level="basic" >}}
**Ans:**

**Route 53** is AWS's highly available DNS (Domain Name System) and domain registration service.

**Key features:**
- **DNS resolution** — Translates domain names to IP addresses
- **Domain registration** — Buy and manage domains
- **Health checks** — Monitor endpoints, failover automatically
- **Routing policies:**

| Policy | Use Case |
|--------|---------|
| Simple | Single resource |
| Weighted | A/B testing, traffic split |
| Latency | Route to lowest-latency region |
| Failover | Primary/secondary DR |
| Geolocation | Route by user's country/continent |
| Multi-value | Multiple healthy records |

```bash
# Create a simple A record
aws route53 change-resource-record-sets \
  --hosted-zone-id ZONEID \
  --change-batch '{"Changes":[{"Action":"CREATE","ResourceRecordSet":{"Name":"app.example.com","Type":"A","TTL":300,"ResourceRecords":[{"Value":"1.2.3.4"}]}}]}'
```
{{< /qa >}}

{{< qa num="25" q="What is User Data in EC2?" level="basic" >}}
**Ans:**

**User Data** is a script that runs automatically when an EC2 instance launches for the first time. It's used to bootstrap instances — install software, configure services, pull code.

```bash
#!/bin/bash
# Example User Data script
yum update -y
yum install -y nginx
systemctl start nginx
systemctl enable nginx
echo "<h1>Hello from EC2</h1>" > /var/www/html/index.html
```

**Key points:**
- Runs as **root** at launch
- Only runs **once** (on first boot, unless configured otherwise)
- Maximum size: **16KB**
- Logs go to: `/var/log/cloud-init-output.log`

```bash
# View user data from inside EC2
curl http://169.254.169.254/latest/user-data

# Debug user data execution
cat /var/log/cloud-init-output.log
```
{{< /qa >}}

{{< qa num="26" q="What is SSL termination? On what basis do we use it?" level="intermediate" >}}
**Ans:**

**SSL termination** is the process of decrypting HTTPS traffic at the load balancer (or reverse proxy) level, so backend servers receive plain HTTP traffic.

```
Client ──HTTPS──► ALB (SSL Termination) ──HTTP──► EC2/Container
```

**When to use SSL termination:**
- At **ALB/ELB** — reduces CPU load on backend EC2 instances
- Certificates managed centrally (AWS ACM) — easier rotation
- Backend services don't need SSL knowledge

**When NOT to terminate (End-to-End Encryption):**
- Compliance requirements (PCI-DSS, HIPAA) mandating encryption in transit
- Financial/medical data requiring encryption all the way to the application

```bash
# Configure HTTPS listener on ALB with ACM certificate
aws elbv2 create-listener \
  --load-balancer-arn <alb-arn> \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:region:account:certificate/xxx \
  --default-actions Type=forward,TargetGroupArn=<tg-arn>
```
{{< /qa >}}

{{< qa num="27" q="Difference between Application Load Balancer and Network Load Balancer?" level="intermediate" >}}
**Ans:**

| Feature | ALB (Layer 7) | NLB (Layer 4) |
|---------|--------------|--------------|
| OSI Layer | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP/TLS) |
| Routing | By path, host, headers, query string | By IP and port |
| Performance | Moderate | Extreme (millions of req/sec) |
| Static IP | No (use with NLB for static) | Yes (1 static IP per AZ) |
| WebSockets | Yes | Yes |
| Use cases | Microservices, REST APIs, web apps | Gaming, IoT, real-time, TCP apps |
| SSL termination | Yes | Yes (TLS listener) |
| Health checks | HTTP/HTTPS | TCP/HTTP/HTTPS |

**Rule of thumb:**
- HTTP/HTTPS web traffic → **ALB**
- TCP performance-critical → **NLB**
- Both needed → ALB behind NLB
{{< /qa >}}

{{< qa num="28" q="What is NACL in AWS?" level="basic" >}}
**Ans:**

**NACL (Network Access Control List)** is a stateless firewall at the **subnet level** in a VPC.

```
VPC
└── Subnet (has NACL)
    └── EC2 instance (has Security Group)
```

**Key characteristics:**
- Applied to **all resources** in a subnet
- **Stateless** — must explicitly allow both inbound AND outbound
- Rules evaluated by **rule number** (lowest first; first match wins)
- Has explicit **Deny** rules (unlike Security Groups)
- Rule number 100 evaluated before 200, etc.

```bash
# Default NACL: Allow all inbound and outbound
# Custom NACL: Deny all by default

# Example: Allow HTTP inbound and deny a specific IP
Rule 100: Allow TCP port 80 inbound from 0.0.0.0/0
Rule 200: Deny ALL from 1.2.3.4/32
Rule *:   Deny ALL (implicit)
```
{{< /qa >}}

{{< qa num="29" q="Is Elastic IP single per AWS account?" level="basic" >}}
**Ans:**

No. By default, AWS allows **5 Elastic IPs per region per account**. This limit can be increased by submitting a service quota increase request.

```bash
# List Elastic IPs
aws ec2 describe-addresses

# Allocate a new Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with an EC2 instance
aws ec2 associate-address \
  --instance-id i-xxxxxxxx \
  --allocation-id eipalloc-xxxxxxxx

# Release (to avoid charges when not associated)
aws ec2 release-address --allocation-id eipalloc-xxxxxxxx
```

**Important:** Elastic IPs are **free when associated** with a running instance. You're charged when it's allocated but not associated.
{{< /qa >}}

{{< qa num="30" q="How would you reduce AWS costs for idle resources?" level="intermediate" >}}
**Ans:**

```bash
# 1. Right-size EC2 instances (use AWS Compute Optimizer)
aws compute-optimizer get-ec2-instance-recommendations

# 2. Use Reserved Instances or Savings Plans (up to 72% savings)
# For predictable workloads → purchase 1 or 3-year RI

# 3. Use Spot Instances for non-critical/batch workloads
aws ec2 request-spot-instances \
  --spot-price "0.05" \
  --instance-count 2 \
  --type one-time \
  --launch-specification file://spec.json

# 4. Auto-stop idle EC2 instances
# Use AWS Instance Scheduler or CloudWatch Events

# 5. Delete unused EBS volumes
aws ec2 describe-volumes --filters Name=status,Values=available
# Volumes in 'available' state → not attached → costing money

# 6. Remove unused Elastic IPs
aws ec2 describe-addresses --query 'Addresses[?!InstanceId]'

# 7. Use S3 lifecycle policies to move old data to Glacier
# 8. Enable S3 Intelligent-Tiering
# 9. Review NAT Gateway data transfer costs
# 10. Use AWS Cost Explorer + Budgets for ongoing monitoring
aws budgets create-budget --account-id ACCOUNT_ID --budget file://budget.json
```
{{< /qa >}}

</div>
