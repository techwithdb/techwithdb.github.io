---
title: "EC2 Scenario Based Interview Questions & Answers (2026)"
description: "30 scenario-based AWS EC2 interview questions covering instance types, Auto Scaling, AMIs, pricing models, networking, storage, security, and troubleshooting — Basic to Advanced."
date: 2026-04-09
author: "DB"
tags: ["aws", "ec2", "interview", "certification"]
tool: "aws"
level: "All Levels"
question_count: 30
---

<div class="qa-list">

## 🟢 Basic

{{< qa num="1" q="Your manager asks you to explain EC2 to a non-technical stakeholder. How do you explain it?" level="basic" >}}
Think of EC2 like renting a computer in Amazon's data center. Instead of buying a physical server and managing the hardware yourself, you log into AWS, pick the size of computer you need (small, medium, large), and have it running in under 2 minutes. You pay only for the time it runs — like a taxi meter. When you no longer need it, you switch it off and stop paying.

**Real-world analogy:** You're opening a pop-up restaurant. Instead of buying ovens, refrigerators, and a building, you rent a fully equipped commercial kitchen by the hour. When business is slow, you rent a smaller kitchen. When a food festival happens, you rent three kitchens at once. That's EC2 — infrastructure on demand, billed by the hour.

**Technical definition:** EC2 (Elastic Compute Cloud) is AWS's virtual machine service. It provides resizable compute capacity in the cloud. You choose the OS (Linux/Windows), CPU, RAM, storage, and network capacity. It runs on physical hosts in AWS data centers, virtualized using the Nitro hypervisor.
{{< /qa >}}

{{< qa num="2" q="A startup says they need a server for a web app but aren't sure how much traffic they'll get. What EC2 instance type do you recommend and why?" level="basic" >}}
Recommend a **T3 or T4g instance** (General Purpose, burstable performance).

**Why T3:** T3 instances earn CPU credits when idle and spend those credits during traffic spikes. For a startup with unpredictable traffic this is ideal — low traffic hours earn credits, and sudden spikes burst to 100% CPU using accumulated credits automatically.

**When NOT to use T3:** If the workload is consistently high CPU (e.g., video encoding running 24/7), T3 will run out of credits and throttle. In that case use C5 (Compute Optimized).

| Use Case | Instance Family | Real Example |
|----------|----------------|--------------|
| Web servers, small apps | T3, T4g | Startup web app |
| High CPU workloads | C5, C6i | Video transcoding, CI/CD builds |
| In-memory databases | R5, R6i | Redis, SAP HANA |
| ML training | P4, G5 | PyTorch model training |
| Big data, analytics | I3, D3 | Hadoop, Spark clusters |
| General production | M5, M6i | E-commerce backend |
{{< /qa >}}

{{< qa num="3" q="You have an application running on t2.micro and it keeps crashing due to CPU throttling. What happened and how do you fix it?" level="basic" >}}
**What happened:** t2.micro earns CPU credits at 6 credits per hour. When the application runs at high CPU, it spends credits faster than it earns them. Once the credit balance hits zero, AWS throttles the CPU to the baseline — only 10% for t2.micro. The app becomes unresponsive.

**How to diagnose:** Go to EC2 → Monitoring → look at `CPUCreditBalance` in CloudWatch. If the line trends toward zero, this confirms the issue.

| Option | When to use | Action |
|--------|-------------|--------|
| Enable Unlimited Mode | Workload occasionally needs burst | EC2 → Instance → Change credit specification → Unlimited |
| Resize to M5 or C5 | Workload consistently needs CPU | Stop instance → Change instance type → Start |
| Add Auto Scaling | Traffic-based CPU spikes | Scale out when CPUUtilization exceeds 70% |

**Real-world example:** An e-commerce site runs on t2.micro. On a flash sale, CPU credit balance hits zero at 11 AM and the site becomes unresponsive. Fix: resize to t3.medium or enable T3 Unlimited.
{{< /qa >}}

{{< qa num="4" q="Explain the difference between On-Demand, Reserved, Spot, and Savings Plans. A CTO asks which to use for their production database." level="basic" >}}
**On-Demand:** Pay per second, no commitment. Like renting a car at the airport — expensive per day but zero obligation.

**Reserved Instances (RI):** Commit to 1 or 3 years and get 40–75% discount. Like signing a 1-year apartment lease — cheaper monthly but you are locked in.

**Spot Instances:** Bid for unused AWS capacity at 70–90% discount. AWS can terminate with 2-minute notice. Like last-minute flight tickets — very cheap but the airline can cancel.

**Savings Plans:** Flexible commitment to a dollar-per-hour spend for 1 or 3 years. Applies automatically across instance families and even Lambda and Fargate.

**For a production database — use Reserved Instances:** Production databases run 24/7 so RI savings (40–60%) are fully realized every hour. They cannot tolerate interruptions so Spot is disqualified.

**Real-world cost comparison** for r5.2xlarge running PostgreSQL:
- On-Demand: $0.504/hr × 8,760 hr/year = **$4,415/year**
- 1-year Reserved (Partial Upfront): approximately **$2,600/year** — saving $1,800/year on a single instance
{{< /qa >}}

{{< qa num="5" q="What is the difference between stopping, hibernating, and terminating an EC2 instance? Your team accidentally terminated a production instance — what do you do?" level="basic" >}}
**Stop:** Instance shuts down. RAM contents are lost. EBS root volume is preserved. Instance retains its Instance ID, Elastic IP, and private IP. No compute charges while stopped.

**Hibernate:** RAM contents are written to the EBS root volume before shutdown. On restart, RAM is restored and applications resume exactly where they left off. Best for long-running data processing jobs you want to pause and resume.

**Terminate:** Instance is permanently destroyed. Root EBS volume is deleted by default. Cannot be undone.

**Recovery after accidental termination:**
1. Check if an AMI was created from the instance — launch a new one from it
2. Check EBS snapshots — if "Delete on termination" was disabled, the volume still exists and can be attached to a new instance
3. Check AWS Backup or automated snapshot policies for the latest restore point

**Prevention going forward:**
```
EC2 → Instance → Actions → Instance settings → Change termination protection → Enable
```
With termination protection enabled, even the AWS CLI `terminate-instances` command fails with an explicit error — someone must consciously disable the protection first.
{{< /qa >}}

## 🟡 Intermediate

{{< qa num="1" q="Your company deploys new EC2 instances every week. Each setup takes 45 minutes. How do you reduce this to under 2 minutes?" level="intermediate" >}}
**Solution: Golden AMI pattern**

An AMI (Amazon Machine Image) is a complete snapshot of an EC2 instance — the OS, installed software, configuration files, and application binaries. Launching from an AMI skips all installation steps.

**Process:**
1. Launch a base EC2 instance (e.g., Amazon Linux 2)
2. Install and configure everything: nginx, Node.js, your application, monitoring agents
3. Test thoroughly
4. Create an AMI: EC2 → Instance → Actions → Image and templates → Create image
5. Use this AMI ID in all Auto Scaling launch templates going forward

**Result:** New instances launch in 60–90 seconds. The 45-minute installation step does not happen on each launch.

**Real-world example:** Netflix uses this pattern extensively. Their internal AMI Bakery (built with HashiCorp Packer) automatically builds new Golden AMIs every time application code changes.

**AMI refresh strategy:**
- Rebuild the AMI when OS patches release or application version changes
- Automate with HashiCorp Packer integrated into your CI/CD pipeline
- Keep the last 3 AMI versions so you can roll back by launching from an older AMI ID
{{< /qa >}}

{{< qa num="2" q="Your production database crashed and you need to recover data from 2 days ago. Walk me through the recovery process using EBS snapshots." level="intermediate" >}}
EBS snapshots are incremental point-in-time backups stored in S3.

**Step 1 — Find the snapshot:** Go to EC2 → Elastic Block Store → Snapshots. Filter by Volume ID and select the snapshot from 2 days ago.

**Step 2 — Create a new EBS volume from the snapshot:** Snapshots → Actions → Create volume from snapshot. Choose the same Availability Zone as your target EC2 instance — volumes must be in the same AZ as the instance they attach to.

**Step 3 — Restore options:**

Option A (Full instance recovery): Launch a new EC2 instance. Stop it. Detach its root EBS volume. Attach the restored snapshot volume as the root device. Start the instance and update DNS.

Option B (Data extraction only): Attach the restored volume to a running rescue instance as a secondary volume. Mount it, copy only the needed files, then detach and delete the recovery volume.

**Step 4 — Validate:** Verify data integrity, test application connectivity, and check logs.

**Real-world lesson:** Always enable automated EBS snapshots with lifecycle policies. For a production database, take snapshots every 4 hours and retain them for 7 days. AWS Backup can manage this centrally.
{{< /qa >}}

{{< qa num="3" q="You need to copy an EC2 instance from us-east-1 to ap-south-1 for a new regional deployment. How do you do it?" level="intermediate" >}}
EC2 instances and EBS volumes are region-locked. You cannot directly move them. The correct method is to copy the AMI across regions.

**Step 1 — Create an AMI from the running instance:** EC2 → Instance → Actions → Image and templates → Create image.

**Step 2 — Copy the AMI to ap-south-1:** EC2 → AMIs → Select your AMI → Actions → Copy AMI → Destination region: ap-south-1. This copies the AMI and all underlying EBS snapshots. Takes 15–60 minutes depending on AMI size.

**Step 3 — Launch in ap-south-1:** Switch to the ap-south-1 region in the AWS Console. EC2 → AMIs → Launch instance from AMI.

**Step 4 — Recreate supporting infrastructure:** VPC, subnets, security groups, and key pairs are NOT copied with the AMI — recreate them in ap-south-1. IAM roles are global and can be reused directly.

**Real-world use case:** An Indian startup originally deployed in us-east-1. Copying infrastructure to ap-south-1 (Mumbai) reduced API response times from 200ms to 20ms for Indian users.
{{< /qa >}}

{{< qa num="4" q="Your application writes massive temporary files during processing and needs the fastest possible disk I/O. What storage do you recommend?" level="intermediate" >}}
**Recommendation: EC2 Instance Store (NVMe SSD)**

Instance Store is physically attached NVMe SSDs on the host machine. There is no network between the CPU and the disk — it is direct hardware access.

**Performance:** 1–3 million IOPS and sub-millisecond latency. This is the fastest storage available anywhere in AWS.

**Critical caveat:** Data is lost when the instance stops or terminates. Instance Store survives reboots but not stop/start cycles.

**When to use it:**
- Temporary files during ML model training (intermediate tensors, epoch checkpoints)
- Scratch space for Hadoop and Spark shuffle data
- Database buffer pools and temp tables during large queries

| Feature | EBS gp3 | Instance Store |
|---------|---------|----------------|
| IOPS | Up to 16,000 | 1–3 million |
| Latency | Approx 1ms | Under 0.1ms |
| Persistence | Survives stop and terminate | Lost on stop or terminate |
| Cost | $0.08 per GB per month | Included in instance price |
| Best for | Root volumes, databases | Temporary data, caches, scratch |

**Real-world example:** An analytics company runs Spark jobs generating 2 TB of shuffle data. Using EBS gp3 the job takes 4 hours. Using an i3.4xlarge with NVMe Instance Store, the same job takes 45 minutes.
{{< /qa >}}

{{< qa num="5" q="Your developers say the EC2 disk is getting full even though they haven't added data. What are the common causes and how do you fix them?" level="intermediate" >}}
**Cause 1 — Log files growing without rotation:** Check with `du -sh /var/log/* | sort -rh | head -10`. Fix: configure logrotate or stream logs directly to CloudWatch Logs.

**Cause 2 — Docker image accumulation:** A CI/CD server running Docker builds every day accumulates old image layers. Fix: add `docker system prune -a` to the build pipeline cleanup step.

**Cause 3 — Core dump files:** A crashing application writes a full memory dump each time. An instance with 16 GB RAM generates a 16 GB core dump file. Check `/var/crash/`.

**Cause 4 — Old kernel versions:** Each kernel update leaves the previous kernel installed. On Ubuntu run `apt autoremove`. On Amazon Linux run `package-cleanup --oldkernels --count=1`.

**Cause 5 — Temporary files in /tmp:** Check with `ls -lhS /tmp | head -20`.

**Long-term fix — expand EBS without downtime:**
```bash
# In AWS Console: EC2 → Volumes → Modify Volume → Increase size
# Then inside the instance:
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1       # for ext4
# OR for xfs:
sudo xfs_growfs /
```
This expands the live filesystem with zero downtime.
{{< /qa >}}

{{< qa num="6" q="When would you choose EFS over EBS? Give a real-world scenario." level="intermediate" >}}
**EBS:** Attached to ONE EC2 instance at a time. Highest performance. Best for single-instance databases and root volumes.

**EFS:** Mounted by MULTIPLE EC2 instances simultaneously. Scales automatically to petabytes. Higher latency than EBS. Best when multiple instances need to share the same files.

**Real-world scenario where EFS is the right answer:**

A media company runs a video editing platform on 20 EC2 instances. With EBS, you need complex logic to move video files between instances — synchronization problems and wasted storage. With EFS: mount the same filesystem on all 20 instances. A user uploads a video and any of the 20 instances can immediately read and process it. Simple and stateless.

**Another common use case:** A web server fleet where all instances need to serve the same user-uploaded content — product images, profile photos, PDF invoices. EFS stores them once and every server reads from the same shared filesystem.

**Cost consideration:** EFS costs roughly 3× more per GB than EBS gp3. Only use it when shared simultaneous access from multiple instances is genuinely required.
{{< /qa >}}

{{< qa num="7" q="You deployed an EC2 instance but cannot SSH into it. Walk me through your troubleshooting checklist." level="intermediate" >}}
Approach from the outside in — network layer first, then OS layer.

**Check 1 — Security Group inbound rules:** EC2 → Security Groups → Inbound rules. Must have: Type = SSH, Protocol = TCP, Port = 22, Source = your IP. This is the most common cause.

**Check 2 — Public subnet with route to Internet Gateway:** The subnet's route table must have 0.0.0.0/0 pointing to an Internet Gateway. If it points to a NAT Gateway, the instance is in a private subnet and not reachable from the internet.

**Check 3 — Instance has a public IP or Elastic IP:** If auto-assign public IP was disabled at launch, associate an Elastic IP.

**Check 4 — Correct key pair and OS username:** Amazon Linux uses `ec2-user`, Ubuntu uses `ubuntu`, CentOS uses `centos`. Wrong username returns "Permission denied (publickey)."

**Check 5 — Network ACL:** NACLs are stateless. A custom NACL might block port 22 even if the security group allows it.

**Check 6 — EC2 instance status checks:** EC2 → Instance → Status checks. If either check is failing, the instance may not be reachable at all.

**Real-world shortcut:** AWS Systems Manager Session Manager gives you a shell without port 22, key pairs, or public IPs. For production environments, disable SSH entirely and use SSM — it is more secure and creates a full audit trail.
{{< /qa >}}

{{< qa num="8" q="What is the difference between Security Groups and Network ACLs? When would you use each?" level="intermediate" >}}
| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| Applied at | Instance level | Subnet level |
| State | Stateful | Stateless |
| Rule types | Allow only | Allow and Deny |
| Default inbound | All blocked | All allowed |
| Rule evaluation | All rules evaluated together | Rules evaluated in number order, first match wins |

**The critical difference — stateful vs stateless:**

A Security Group is stateful: if you allow inbound TCP port 443, the response traffic is automatically allowed outbound.

A Network ACL is stateless: you must explicitly allow both the inbound request AND the outbound response. For HTTPS you need: inbound Allow TCP 443 AND outbound Allow TCP 1024–65535 (ephemeral ports). Forgetting the outbound ephemeral port rule is one of the most common NACL mistakes.

**When to use each:**

Security Groups handle 99% of scenarios — web servers, database access restricted to app server's security group ID, SSH restricted to VPN IP.

NACLs are used as an additional defensive layer — blocking a DDoS source IP range at the subnet level, or meeting compliance requirements to block certain IP ranges at the network layer.
{{< /qa >}}

{{< qa num="9" q="Your EC2 instance's public IP changes every time it restarts, breaking your DNS configuration. How do you solve this?" level="intermediate" >}}
**Why this happens:** When you stop and start an instance (not reboot), AWS releases the public IP back to the shared pool and assigns a new one.

**Solution 1 — Elastic IP (most common fix):** An Elastic IP is a static public IPv4 address that belongs to your AWS account and does not change across stop/start cycles.
```
EC2 → Elastic IPs → Allocate Elastic IP address
EC2 → Elastic IPs → Actions → Associate Elastic IP → Select your instance
```
Elastic IPs are free while associated with a running instance. You are charged $0.005/hour when allocated but not associated.

**Solution 2 — Use private IP for internal DNS:** Private IPs within a VPC never change for the lifetime of the instance. Use a Route 53 private hosted zone record.

**Solution 3 — Load Balancer DNS name:** Point Route 53 to an ALB DNS name. The ALB DNS name is stable while individual EC2 instances behind it can be stopped or replaced.

**Real-world example:** A company runs Jenkins on EC2. Every restart changes the public IP and breaks GitHub webhook URLs. Solution: one Elastic IP on the Jenkins instance, update the webhook URL once — it never needs updating again.
{{< /qa >}}

{{< qa num="10" q="Your e-commerce website crashes every Black Friday due to traffic spikes. Design an auto-scaling solution." level="intermediate" >}}
**Architecture:**
```
Internet → Route 53 → Application Load Balancer (multi-AZ)
                              |
          +-------------------+-------------------+
          |                   |                   |
      EC2 (AZ-a)         EC2 (AZ-b)         EC2 (AZ-c)
          +----------- Auto Scaling Group ---------+
                    Min: 2 | Desired: 4 | Max: 20
```

**Three scaling policies:**

**Policy 1 — Target Tracking (recommended):** Set a target of 1,000 requests per instance on the ALB. The ASG automatically adds or removes instances to maintain that ratio.

**Policy 2 — Step Scaling:** If CPU exceeds 70% for 3 minutes, add 2 instances. If CPU exceeds 90% for 1 minute, add 4 instances. If CPU drops below 30% for 10 minutes, remove 1 instance.

**Policy 3 — Scheduled Scaling (for Black Friday):** The day before Black Friday, automatically pre-warm the fleet to 15 instances. The evening after, scale back to 4. You pay for extra capacity only during the event window.

**Real-world outcome:** Normal traffic: 4 instances. Black Friday peak: ASG scales to 18 instances automatically within minutes. End of day: back to 4. You pay only for what you use, and the site never crashes.
{{< /qa >}}

{{< qa num="11" q="What is the difference between ALB, NLB, and CLB? A gaming company needs ultra-low latency for real-time multiplayer. Which do you choose?" level="intermediate" >}}
**Classic Load Balancer (CLB):** Legacy product with no content-based routing. Not recommended for any new deployment.

**Application Load Balancer (ALB):** Layer 7 (HTTP/HTTPS). Supports path-based routing, WebSockets, HTTP/2, SSL termination. Adds approximately 1–5ms of processing overhead. Ideal for microservices, REST APIs, and web applications.

**Network Load Balancer (NLB):** Layer 4 (TCP, UDP, TLS). Handles millions of requests per second. Ultra-low latency around 100 microseconds. Preserves client source IP. Supports static Elastic IPs per AZ.

**For the gaming company — choose NLB:**

Real-time multiplayer games use UDP for game state updates. UDP does not wait for retransmission — in a fast-paced game a stale position update is worthless so you drop it and send the next one. ALB does not support UDP at all. NLB handles UDP natively with sub-millisecond overhead.

NLB's static IP address per AZ also lets game clients hardcode the server IP, eliminating DNS lookup latency on every connection attempt.
{{< /qa >}}

{{< qa num="12" q="An Auto Scaling Group keeps launching instances that immediately get marked unhealthy and terminated. What could be wrong?" level="intermediate" >}}
This creates an infinite replace loop. Work through this checklist:

**Issue 1 — Health check endpoint returning errors:** The ALB health check is hitting `/health` but the application is returning 404 or 500. Verify the health check path returns HTTP 200.

**Issue 2 — User Data bootstrap script failing silently:** The startup script crashes partway through so the application never starts. Add error logging to every User Data script:
```bash
#!/bin/bash
set -e
exec > /var/log/user-data.log 2>&1
```
Then check: EC2 → Instance → Monitor and troubleshoot → Get system log.

**Issue 3 — Security group blocking ALB health checks:** ALB health checks originate from the ALB's IP addresses. Add an inbound rule allowing the ALB's security group ID on port 80 or 443.

**Issue 4 — Health check grace period too short:** If your application takes 4 minutes to start and the grace period is 60 seconds, ASG declares the instance unhealthy before the app finishes booting. Fix: set the grace period to at least your worst-case boot time (300–600 seconds).

**Issue 5 — Golden AMI missing the application:** The AMI was rebuilt without the application binary. Fix: add a smoke test to your AMI build pipeline that verifies the application starts correctly before the AMI is finalized.
{{< /qa >}}

{{< qa num="13" q="Design a highly available architecture for a 3-tier web application on EC2." level="intermediate" >}}
The core HA principle: every tier spans at least 2 Availability Zones so a single AZ failure does not take down the application.

**Web Tier:** An ALB distributes incoming traffic across EC2 instances in AZ-a and AZ-b. An ASG with minimum 2 instances ensures capacity even if one instance fails. Nginx handles SSL termination and static file serving.

**Application Tier:** An internal ALB distributes API traffic across EC2 instances in AZ-a and AZ-b. These instances must be stateless — user session data is stored in ElastiCache (Redis), not in instance memory. Losing any instance does not lose any user's session.

**Data Tier:** RDS Multi-AZ with primary in AZ-a and synchronous standby in AZ-b. If AZ-a fails, RDS automatically fails over to AZ-b within approximately 60 seconds with zero data loss.

**Why this design works:** ALB health checks remove unhealthy instances within seconds. ASG replaces failed instances automatically. No state on instances means any instance can serve any user. Multi-AZ at every tier means no single AZ failure causes an outage.

**Recovery objectives:**
- RTO under 60 seconds for a single AZ failure
- RPO of zero (no data loss) since RDS Multi-AZ uses synchronous replication
{{< /qa >}}

{{< qa num="14" q="What is the difference between High Availability and Fault Tolerance? Give an EC2-based example of each." level="intermediate" >}}
**High Availability:** The system recovers from failure quickly — within seconds to minutes. There is a brief interruption but the system comes back automatically without human action.

**Fault Tolerance:** The system continues operating with zero interruption even while a component is failing. There is no impact on users at all.

**EC2 example of High Availability:**
Two EC2 instances behind an ALB. Instance 1 crashes. The ALB detects the failure within 10 seconds via health checks and stops routing traffic to it. ASG launches a replacement instance. The system runs at reduced capacity for 3 minutes, then returns to normal. Users experience at most one failed request. This is HA — brief impact, automatic recovery.

**EC2 example of Fault Tolerance:**
A payment processing service writes transaction data simultaneously to DynamoDB replicated across 3 AZs. An EC2 instance processing a payment crashes mid-operation. Another healthy instance confirms the payment to the user. The user experiences no impact whatsoever. This is FT — zero impact by design.

**Cost reality:** High Availability costs roughly 2× (one active, one standby). Full Fault Tolerance costs 3–6× due to fully redundant active components. HA is appropriate for most web applications. FT is reserved for financial transactions and safety-critical systems where any interruption is unacceptable.
{{< /qa >}}

## 🔴 Advanced

{{< qa num="1" q="Your application on EC2 needs to read from S3 and write to DynamoDB. A junior engineer hardcoded AWS access keys in the application code. What do you do?" level="advanced" >}}
**Treat this as a security incident — immediate actions:**

1. **Rotate the compromised credentials immediately.** IAM → Users → Security credentials → Find the exposed access key → Deactivate → Delete. Assume the key was leaked even if you are not certain.

2. **Check CloudTrail for unauthorized API calls** made with the compromised key. Filter events by AccessKeyId. Look for calls from unexpected IP addresses, unusual AWS regions, or access to S3 buckets the application should not be touching.

3. **Remove the key from the code.** Note: git revert does not help — if the key was ever pushed to a repository, it exists in git history and must be treated as permanently compromised. Rotating it is what actually stops the exposure.

**The correct architecture — IAM Instance Roles:**

Attach an IAM Role to the EC2 instance with least-privilege permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {"Effect": "Allow", "Action": ["s3:GetObject"], "Resource": "arn:aws:s3:::my-bucket/*"},
    {"Effect": "Allow", "Action": ["dynamodb:PutItem"], "Resource": "arn:aws:dynamodb:*:*:table/my-table"}
  ]
}
```
The AWS SDK automatically retrieves temporary credentials from the Instance Metadata Service. No access keys exist anywhere. AWS rotates these temporary credentials every 6 hours automatically.
{{< /qa >}}

{{< qa num="2" q="How do you prevent someone from accidentally terminating a critical production EC2 instance?" level="advanced" >}}
Use multiple independent layers so no single mistake causes a disaster.

**Layer 1 — EC2 Termination Protection:**
```
EC2 → Instance → Actions → Instance settings → Change termination protection → Enable
```
Any terminate-instances API call fails with a clear error. The user must explicitly disable protection first — a mandatory deliberate step.

**Layer 2 — IAM Policy denying termination on production resources:**
```json
{
  "Effect": "Deny",
  "Action": "ec2:TerminateInstances",
  "Resource": "*",
  "Condition": {
    "StringEquals": {"ec2:ResourceTag/Environment": "production"}
  }
}
```
This denies termination of any instance tagged `Environment=production` regardless of who the user is. Even an Administrator cannot override a Deny without modifying the IAM policy first.

**Layer 3 — Service Control Policy (SCP) for AWS Organizations:**
SCPs apply to all users in all accounts including the root user. This is the strongest available protection — no one in the account can bypass it without modifying the organization-level policy.

**Layer 4 — CloudWatch alert when termination protection is disabled:**
AWS Config detects when termination protection is turned off on a production instance and triggers a CloudWatch alarm that sends an SNS notification to the security team — creating an audit trail and human review opportunity.
{{< /qa >}}

{{< qa num="3" q="Your AWS bill shows EC2 costs doubled last month. How do you investigate and reduce costs?" level="advanced" >}}
**Investigation:**

Open AWS Cost Explorer. Group costs by Instance Type, sorted by cost descending. Look for new instance types that appeared, unexpected AWS regions, or instances running continuously with no scheduled downtime.

Open AWS Trusted Advisor and check "Low utilization EC2 instances" — instances with less than 10% average CPU utilization over 14 days. These are either oversized or forgotten.

Hunt for forgotten resources: instances running in non-primary regions from old tests, unattached EBS volumes at $0.08/GB/month, Elastic IPs not associated with any instance at $0.005/hour each, and old AMI snapshots accumulating silently.

**Cost reduction actions:**

| Action | Expected Saving | Effort |
|--------|----------------|--------|
| Purchase Reserved Instances for 24/7 workloads | 40–60% | Low |
| Rightsize oversized instances using actual utilization data | 20–40% | Medium |
| Use Spot Instances for dev, test, and batch workloads | 70–90% | Medium |
| Schedule stop/start for dev instances (nights and weekends) | 60% on dev costs | Low |
| Delete all unattached EBS volumes | Variable but immediate | Low |

**Real-world example:** A startup audit found forgotten test instances ($400/month savings), production instances eligible for Reserved Instances ($1,200/month savings), and dev instances running 24/7 fixable with a Lambda scheduler ($600/month savings). Total: $2,200/month from one afternoon audit.
{{< /qa >}}

{{< qa num="4" q="Explain Spot Instances. A company wants to run ML model training but has a limited budget. Design a cost-effective solution." level="advanced" >}}
**What are Spot Instances:** Spot Instances are unused EC2 capacity that AWS sells at 70–90% discount. AWS can reclaim the instance with a 2-minute warning when they need the capacity back for On-Demand customers.

**Why Spot is ideal for ML training:** Training jobs can save checkpoints periodically and resume from the last checkpoint if interrupted. They do not need to respond to users in real time — a training job that takes 10 hours instead of 8 due to one interruption is still far cheaper than paying full On-Demand price.

**Architecture:**
Use a Spot instance with GPU capacity (e.g., p3.2xlarge). Write the training script to:
- Save a checkpoint to S3 every 10 minutes
- Handle the SIGTERM signal that AWS sends 2 minutes before interruption — save a final checkpoint and exit cleanly
- On new Spot instance launch, detect an existing S3 checkpoint and resume from that point

No work is lost beyond the most recent checkpoint interval.

**Real-world cost impact:**
- p3.2xlarge On-Demand: $3.06/hour
- p3.2xlarge Spot: approximately $0.92/hour — 70% discount
- A 100-hour training job: $306 On-Demand vs $92 on Spot
- An ML team with a $50,000 annual compute budget can run 3× as many experiments using Spot
{{< /qa >}}

{{< qa num="5" q="Your EC2 application is running but users say it is slow. How do you diagnose the performance bottleneck?" level="advanced" >}}
Performance issues always fall into one of four categories: CPU, Memory, Disk I/O, or Network. Check each methodically.

**Step 1 — CloudWatch Metrics (outside-in view):**
EC2 → Monitoring. Check CPUUtilization, NetworkIn/NetworkOut, EBSReadOps/EBSWriteOps.

Important: CloudWatch does not show memory utilization or disk space by default. Install the CloudWatch Agent to expose these metrics.

**Step 2 — SSH into the instance (inside view):**

For CPU: run `top` or `htop` to see which process is consuming CPU.

For Memory: run `free -h`. If swap is being used, the OS is using the disk as overflow memory — extremely slow and often the root cause of mysterious slowness on undersized instances.

For Disk I/O: run `iostat -x 1 5`. If `%util` is near 100% or `await` is above 10ms, disk is the bottleneck.

For Network: run `netstat -s` to check for packet drops and retransmissions.

**Step 3 — Application-level tracing:**
Install AWS X-Ray or a commercial APM tool like Datadog. Trace individual user requests to find the specific database query, external API call, or code function that is slow.

**Real-world example:** An e-commerce platform was slow every day at exactly 2 PM. CloudWatch showed EBS WriteOps maxed out at that time. A scheduled cron job generating monthly reports was writing large files to EBS. Fix: redirect the report output to S3 instead. Page load times dropped from 8 seconds to 400ms immediately.
{{< /qa >}}

{{< qa num="6" q="What is the EC2 Instance Metadata Service and how can it be exploited? How do you protect against it?" level="advanced" >}}
**What is IMDS:**
Every EC2 instance can access a special HTTP endpoint at `169.254.169.254` to retrieve information about itself — its instance ID, AZ, IAM role name, and most importantly, temporary IAM credentials. No authentication is required from inside the instance.

**The security risk — SSRF attacks:**
Server-Side Request Forgery (SSRF) occurs when an attacker tricks your application into making HTTP requests on their behalf. If your application fetches user-provided URLs (an image downloader, webhook validator), an attacker submits the IMDS URL. Your application fetches it and returns the IAM role's temporary credentials. The attacker now has AWS access keys.

**Real-world incident:** The 2019 Capital One data breach exposed approximately 100 million customer records. An SSRF vulnerability in a misconfigured WAF was used to query the IMDS endpoint and steal IAM credentials. Those credentials were used to download data from over 700 S3 buckets.

**Protection — enforce IMDSv2:**
IMDSv2 requires a PUT request to obtain a session token before any metadata can be read. SSRF attacks use simple GET requests — they cannot perform the required PUT step, so the attack fails completely.
```
EC2 → Instance → Actions → Modify instance metadata options
HTTP tokens: Required
HTTP PUT response hop limit: 1
```
The hop limit of 1 prevents containers inside the instance from accessing the host instance's IMDS — a critical security boundary for containerized workloads.
{{< /qa >}}

{{< qa num="7" q="You need 50 EC2 instances to automatically configure themselves and deploy your application on every launch. How do you automate this?" level="advanced" >}}
**Best practice: Golden AMI combined with User Data — a two-layer approach**

Pre-bake the slow, stable steps into the AMI. Use User Data only for the fast, environment-specific steps that change between instances or deployments.

**What goes in the AMI (built once, reused many times):**
OS updates applied, web server and runtime installed (nginx, Node.js), systemd service files created, CloudWatch and monitoring agents installed and configured. This work happens once during AMI creation, not on every instance launch.

**What goes in User Data (runs on every launch, fast):**
Pull the latest application artifact from S3. Fetch database connection strings from SSM Parameter Store. Write environment-specific configuration files. Start services with systemctl.

**Why SSM Parameter Store instead of hardcoded values in User Data:**
User Data is stored in plain text in the EC2 console — anyone with EC2 read access can view it. If database passwords are in User Data, they are visible to every person who can list EC2 instances. SSM Parameter Store with KMS encryption stores secrets securely. The User Data script contains only the parameter names, never the actual values.

**A critical mistake to warn about:** Developers sometimes put secrets in User Data scripts inside CloudFormation templates, check them into git repositories, and inadvertently publish credentials to GitHub. The correct pattern is always: parameter names in code, actual values in SSM.
{{< /qa >}}

{{< qa num="8" q="You are building an HPC cluster for scientific simulation where network latency between nodes is critical. What EC2 feature do you use?" level="advanced" >}}
**Use a Cluster Placement Group.**

A Cluster Placement Group packs EC2 instances physically close together within a single Availability Zone, placing them on the same rack or adjacent racks connected via high-bandwidth low-latency networking.

**Performance:** Up to 100 Gbps network bandwidth between instances and latency in the 10–50 microsecond range. This is the highest network performance available anywhere in EC2.

**How to create and use one:**
```
EC2 → Placement Groups → Create placement group
Name: hpc-simulation-cluster
Strategy: Cluster
```

**Three placement group strategies:**

| Strategy | Physical placement | Best use case |
|----------|--------------------|---------------|
| Cluster | Same rack in one AZ | HPC, MPI, financial trading — maximum network speed |
| Spread | Different physical hardware racks (max 7 per AZ) | Small number of critical instances — one hardware failure only affects one instance |
| Partition | Isolated groups of racks | Distributed systems like Hadoop, Cassandra, Kafka |

**Real-world example:** A genomics company runs molecular dynamics simulation on 32 c5n.18xlarge instances. Without a Cluster Placement Group, inter-node latency is 200 microseconds. With it, latency drops to 15 microseconds. The simulation runs 4× faster.

**Trade-off:** Cluster Placement Groups are single-AZ. Never use one for web servers or any workload requiring high availability.
{{< /qa >}}

{{< qa num="9" q="Your EC2 application processes SQS messages. Sometimes messages expire or get processed twice. How do you architect this properly?" level="advanced" >}}
**Root cause of both problems:**
SQS has a visibility timeout — when an instance picks up a message, it becomes invisible to other consumers for N seconds. If processing takes longer than the timeout, SQS makes the message visible again and a second instance picks it up, causing duplicate processing.

**Proper configuration:**
Set the visibility timeout to 6× the maximum expected processing time. If the slowest case takes 5 minutes, set the timeout to 30 minutes.

For long-running jobs, extend the visibility timeout periodically from within the consumer code using the `ChangeMessageVisibility` API. This keeps the message invisible as long as your code is actively working on it.

**Dead Letter Queue (DLQ):**
After 3 failed processing attempts, move the message to a DLQ instead of retrying indefinitely. This prevents one broken message (a "poison pill") from blocking the entire queue. Configure a CloudWatch alarm on DLQ depth so the team is notified immediately when messages land there.

**Auto Scaling based on queue depth:**
Use `ApproximateNumberOfMessagesVisible` (not CPU) as the scaling metric. When messages accumulate, add instances. When the queue is empty for 5 minutes, scale in.

**Real-world example:** A fintech company processes bank payment confirmations via SQS. Some confirmations resolve in 2 seconds, others take 15 minutes due to slow bank APIs. With proper visibility timeout extension and DLQ, no message is ever lost or duplicated. The consumer fleet scales from 2 to 40 instances during peak settlement windows and back down automatically.
{{< /qa >}}

{{< qa num="10" q="A company needs to migrate 200 on-premises servers to EC2. How do you plan and execute this?" level="advanced" >}}
Large-scale migrations follow the **6 R's framework**. Every server is classified before migration begins:

| Strategy | When to use | Example |
|----------|-------------|---------|
| Rehost (Lift and Shift) | Quick migration, optimize later | Move web server as-is to EC2 |
| Replatform | Minor cloud optimization | Move self-managed MySQL to RDS |
| Repurchase | Replace with SaaS | Internal CRM replaced by Salesforce |
| Refactor | Re-architect for cloud-native | Monolith broken into microservices on ECS |
| Retain | Stay on-premises | Mainframes, strict compliance requirements |
| Retire | Decommission | Applications nobody actually uses anymore |

**Execution in four phases:**

**Phase 1 — Discovery (2–4 weeks):** Install AWS Application Discovery Service agents on all 200 servers. Collect CPU utilization, memory, disk, network traffic, and running processes. The output is a dependency map showing which servers communicate with which — critical for grouping applications that must migrate together.

**Phase 2 — Assessment and rightsizing:** Use actual utilization data to select EC2 instance types. On-premises servers are typically 15–20% utilized on average. Avoid provisioning the same size as on-premises — this wastes money.

**Phase 3 — Pilot migration (5–10 servers):** Use AWS Application Migration Service (MGN). Install the replication agent on the source server. It performs continuous block-level replication to AWS with zero downtime. Typical cutover is 15–30 minutes of downtime.

**Phase 4 — Wave-based migration:** Migrate 20–30 servers per wave, always including all servers in the same application dependency group. Start with low-risk internal tools. Migrate business-critical production systems last.

Realistic timeline: A 200-server migration with a team of 10 people typically takes 6–12 months.
{{< /qa >}}

{{< qa num="11" q="Your compliance team requires: no public IPs on EC2, encrypted EBS volumes, IMDSv2 enforced, and CloudTrail enabled — across 50 AWS accounts. How do you enforce this automatically?" level="advanced" >}}
This requires a combination of AWS Organizations, Service Control Policies, AWS Config, and Systems Manager Automation.

**Prevent violations with Service Control Policies (SCPs):**
An SCP applied at the Organization level prevents any instance from launching without IMDSv2 enforced:
```json
{
  "Effect": "Deny",
  "Action": "ec2:RunInstances",
  "Resource": "arn:aws:ec2:*:*:instance/*",
  "Condition": {
    "StringNotEquals": {"ec2:MetadataHttpTokens": "required"}
  }
}
```
SCPs apply to every user in every account including root. This is a hard block — no exceptions possible without modifying the organization-level policy.

**Enforce mandatory EBS encryption at the account level:**
In each account: EC2 → Settings → EBS encryption → Enable by default. Every new EBS volume is automatically encrypted. It becomes impossible to create an unencrypted volume.

**Detect non-compliant existing resources with AWS Config:**
Deploy these managed Config rules across all 50 accounts using CloudFormation StackSets:
- `encrypted-volumes` — detects unencrypted EBS volumes
- `ec2-imdsv2-check` — detects instances not enforcing IMDSv2
- `ec2-instance-no-public-ip` — detects instances with public IPs
- `cloud-trail-enabled` — detects accounts where CloudTrail is not active

**Auto-remediate with Systems Manager Automation:**
When Config flags a non-compliant resource, EventBridge triggers an SSM Automation runbook that fixes it automatically and sends an SNS notification documenting the remediation.

**Centralized visibility with AWS Security Hub:**
Aggregate compliance findings from all 50 accounts into one Security Hub dashboard. Leadership sees at a glance which accounts are compliant without any manual spreadsheet work.
{{< /qa >}}

</div>
