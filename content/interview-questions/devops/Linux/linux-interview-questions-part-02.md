---
title: "Linux Interview Questions & Answers (2026) Part 02"
description: "40+ Linux interview questions and answers covering processes, file system, permissions, services, and troubleshooting — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["linux", "interview", "sysadmin", "devops"]
tool: "linux"
level: "All Levels"
question_count: 52
draft: "false"
---


> A comprehensive guide covering real-world scenarios for Linux, Ubuntu, and AWS EC2 interviews.



{{< qa num="1" q="You launched an EC2 instance but cannot SSH into it. How do you troubleshoot?" level="basic" >}}
**Answer:**

Follow a systematic approach:

**Step 1 – Check Security Group inbound rules:**
```bash
# Verify port 22 is open for your IP
aws ec2 describe-security-groups --group-ids sg-xxxxxxxx
```

**Step 2 – Check the instance state:**
```bash
aws ec2 describe-instances --instance-ids i-xxxxxxxx --query 'Reservations[].Instances[].State'
```

**Step 3 – Verify key pair:**
- Ensure you're using the correct `.pem` file that was associated at launch.

**Step 4 – Check network (VPC/Subnet/Route Table):**
- Confirm the subnet has an Internet Gateway attached.
- Check Route Table has `0.0.0.0/0 → igw-xxxxxx`.

**Step 5 – Use EC2 Instance Connect or Session Manager:**
```bash
# Via AWS CLI Session Manager (no port 22 needed)
aws ssm start-session --target i-xxxxxxxx
```

**Step 6 – Retrieve system logs via Console:**
- EC2 Console → Actions → Monitor and troubleshoot → Get system log.

{{< /qa >}}

{{< qa num="2" q="Your SSH key (.pem) was accidentally deleted. How do you recover access to a running EC2 instance?" level="intermediate" >}}

**Answer:**

**Method 1 – Using EC2 Instance Connect (if enabled):**
```bash
aws ec2-instance-connect send-ssh-public-key \
  --instance-id i-xxxxxxxx \
  --instance-os-user ubuntu \
  --ssh-public-key file://~/.ssh/new_key.pub
```

**Method 2 – Attach EBS volume to another instance:**
1. Stop the affected instance.
2. Detach its root EBS volume.
3. Attach it to a working instance as `/dev/xvdf`.
4. Mount and modify `authorized_keys`:

```bash
sudo mount /dev/xvdf1 /mnt/recovery
sudo nano /mnt/recovery/home/ubuntu/.ssh/authorized_keys
# Paste your new public key
sudo umount /mnt/recovery
```

5. Detach volume, re-attach as root (`/dev/xvda`) on original instance, start it.

**Method 3 – AWS Systems Manager Run Command (if SSM agent is running):**
```bash
aws ssm send-command \
  --instance-ids i-xxxxxxxx \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["echo \"NEW_PUBLIC_KEY\" >> /home/ubuntu/.ssh/authorized_keys"]'
```

{{< /qa >}}
{{< qa num="3" q="You receive 'Permission denied (publickey)' when SSHing. What could be wrong?" level="basic" >}}

**Answer:**

Possible causes and fixes:

| Cause | Fix |
|-------|-----|
| Wrong key file | Use `-i /path/to/correct.pem` |
| Wrong username | Use `ubuntu@` for Ubuntu, `ec2-user@` for Amazon Linux |
| Key permissions too open | `chmod 400 mykey.pem` |
| `authorized_keys` corrupted | Recover via volume detach method |
| SELinux/file context issue | `restorecon -Rv ~/.ssh` |

```bash
# Debug SSH connection verbosely
ssh -vvv -i mykey.pem ubuntu@<public-ip>
```

{{< /qa >}}
{{< qa num="4" q="Your EC2 instance is running out of disk space. How do you diagnose and fix it?" level="basic" >}}

**Answer:**

**Step 1 – Identify disk usage:**
```bash
df -hT          # Disk space by filesystem
du -sh /*       # Top-level directory sizes
du -sh /var/log/* | sort -rh | head -20   # Find large log files
```

**Step 2 – Clean up:**
```bash
# Remove old logs
sudo journalctl --vacuum-size=200M
sudo find /var/log -name "*.gz" -delete

# Remove unused packages
sudo apt autoremove -y
sudo apt clean

# Find large files
find / -type f -size +500M 2>/dev/null
```

**Step 3 – Extend EBS volume (if cleanup isn't enough):**
```bash
# 1. Resize in AWS Console or CLI
aws ec2 modify-volume --volume-id vol-xxxxxxxx --size 50

# 2. On the instance, extend partition
lsblk
sudo growpart /dev/xvda 1

# 3. Resize filesystem
sudo resize2fs /dev/xvda1      # ext4
# OR
sudo xfs_growfs /               # xfs
```
{{< /qa >}}
{{< qa num="5" q="You attached a new EBS volume to an EC2 instance. How do you make it usable?" level="basic" >}}
**Answer:**

```bash
# Step 1: List block devices
lsblk

# Step 2: Create a filesystem on the new volume (e.g., /dev/xvdf)
sudo mkfs.ext4 /dev/xvdf

# Step 3: Create a mount point
sudo mkdir -p /data

# Step 4: Mount the volume
sudo mount /dev/xvdf /data

# Step 5: Make it persistent across reboots
# Get UUID
sudo blkid /dev/xvdf

# Add to /etc/fstab
echo "UUID=xxxx-xxxx  /data  ext4  defaults,nofail  0  2" | sudo tee -a /etc/fstab

# Step 6: Verify
df -h /data
```

{{< /qa >}}
{{< qa num="6" q="The root volume of your instance is full and the instance is unresponsive. How do you recover?" level="advanced" >}}

**Answer:**

Since you can't log in, use the detach-and-repair method:

1. Stop the instance (not terminate).
2. Create a snapshot of the root EBS volume as a backup.
3. Attach the root volume to a healthy "rescue" instance as a secondary disk.
4. SSH into rescue instance:

```bash
sudo mount /dev/xvdf1 /mnt/rescue

# Clean up space
sudo rm -rf /mnt/rescue/var/log/*.gz
sudo rm -rf /mnt/rescue/tmp/*
sudo du -sh /mnt/rescue/* | sort -rh

sudo umount /mnt/rescue
```

5. Re-attach volume to original instance as `/dev/xvda`, start it.

{{< /qa >}}
{{< qa num="7" q="Your EC2 instance is very slow. How do you diagnose the bottleneck?" level="intermediate" >}}

**Answer:**

```bash
# CPU usage
top
htop
mpstat -P ALL 1

# Memory usage
free -h
vmstat 1 5

# Disk I/O
iostat -xz 1
iotop

# Network
iftop
nethogs
ss -tulnp

# Load average
uptime
# Output: load average: 0.5, 1.2, 0.8 (1min, 5min, 15min)
# If load > number of CPUs, system is overloaded

# Find top CPU-consuming processes
ps aux --sort=-%cpu | head -10

# Find top memory-consuming processes
ps aux --sort=-%mem | head -10
```

**Interpreting results:**
- High CPU + low I/O wait → CPU-bound workload.
- High I/O wait (`%wa`) in `top` → disk bottleneck.
- Low free memory + high swap → memory pressure.

{{< /qa >}}
{{< qa num="8" q="A process is consuming 100% CPU. How do you handle it?" level="basic" >}}

**Answer:**

```bash
# Identify the process
top -c
# or
ps aux --sort=-%cpu | head -5

# Get PID
PID=12345

# Inspect the process
ls -la /proc/$PID/exe
cat /proc/$PID/cmdline | tr '\0' ' '

# Lower priority first (graceful)
sudo renice +10 -p $PID

# Kill gracefully
sudo kill -15 $PID      # SIGTERM

# Force kill if unresponsive
sudo kill -9 $PID       # SIGKILL

# Kill by name
sudo pkill -9 nginx

# Prevent recurrence: check if it's a cron job, service, or runaway script
sudo systemctl status <service-name>
sudo crontab -l
```
{{< /qa >}}
{{< qa num="9" q="Your application crashes with 'Cannot allocate memory'. How do you diagnose?" level="intermediate" >}}

**Answer:**

```bash
# Check current memory
free -h
cat /proc/meminfo

# Check for OOM kills in logs
sudo dmesg | grep -i "oom\|killed"
sudo grep -i "out of memory" /var/log/syslog

# Check swap usage
swapon --show

# Find memory hogs
ps aux --sort=-%mem | head -10

# Solutions:
# 1. Add swap space (temporary fix)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 2. Make swap permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 3. Upgrade instance type (long-term fix)
# Stop instance → Change instance type in console → Start
```

{{< /qa >}}
{{< qa num="10" q="You deployed a web app on port 8080 but cannot access it from the browser. What do you check?" level="basic" >}}

**Answer:**

**Checklist (layer by layer):**

```bash
# 1. Is the app actually listening?
ss -tulnp | grep 8080
netstat -tlnp | grep 8080

# 2. Is the local firewall blocking it?
sudo ufw status
sudo iptables -L -n -v | grep 8080

# 3. Can you access it locally?
curl http://localhost:8080

# 4. Check Security Group in AWS Console
# Inbound rule: TCP 8080 from 0.0.0.0/0 (or your IP)
aws ec2 describe-security-groups --group-ids sg-xxxxxxxx

# 5. Is NACL blocking it? (if using custom NACLs)
# Check Network ACL for the subnet in VPC console

# 6. Check application logs
sudo journalctl -u myapp -n 50
```

{{< /qa >}}
{{< qa num="11" q="How do you block a specific IP address on a Linux EC2 instance?" level="intermediate" >}}

**Answer:**

**Using `ufw` (simple):**
```bash
sudo ufw deny from 192.168.1.100 to any
sudo ufw reload
sudo ufw status
```

**Using `iptables` (granular):**
```bash
# Block IP completely
sudo iptables -I INPUT -s 192.168.1.100 -j DROP

# Block IP on specific port
sudo iptables -I INPUT -s 192.168.1.100 -p tcp --dport 80 -j DROP

# Save rules to persist after reboot
sudo netfilter-persistent save

# Or for Ubuntu
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

**Using AWS Security Group (recommended for EC2):**
- Go to Security Group → Edit inbound rules → Remove rule allowing the IP.
- This is the most effective layer since it blocks traffic before reaching the OS.

{{< /qa >}}
{{< qa num="12" q="Two EC2 instances in the same VPC cannot communicate with each other. How do you fix it?" level="intermediate" >}}

**Answer:**

```bash
# From Instance A, try pinging Instance B using private IP
ping 10.0.1.25

# Traceroute to identify where traffic drops
traceroute 10.0.1.25
```

**Checklist:**
1. **Security Groups** – Instance B's SG must allow inbound from Instance A's SG or IP range.
2. **NACL** – Both inbound and outbound NACLs must allow traffic (NACLs are stateless).
3. **Same VPC** – Confirm both instances are in the same VPC.
4. **Route Table** – For different subnets, check local route exists.

```bash
# Allow Instance A's SG to reach Instance B on port 3306
aws ec2 authorize-security-group-ingress \
  --group-id sg-instanceB \
  --protocol tcp \
  --port 3306 \
  --source-group sg-instanceA
```

{{< /qa >}}
{{< qa num="13" q="A developer needs temporary sudo access. How do you grant and revoke it safely?" level="intermediate" >}}

**Answer:**

```bash
# Grant sudo access
sudo usermod -aG sudo devuser

# Verify
groups devuser
sudo -l -U devuser

# Grant time-limited sudo via sudoers (with timestamp)
# More controlled approach using sudoers.d
echo "devuser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/devuser-temp
sudo visudo -c   # Validate syntax

# Revoke access
sudo deluser devuser sudo
# OR remove the file
sudo rm /etc/sudoers.d/devuser-temp

# Verify revocation
sudo -l -U devuser
```

**Best practice:** Use IAM + SSM Session Manager instead of direct sudo to individual users.

{{< /qa >}}
{{< qa num="14" q="A file has permissions 777. Why is this dangerous and how do you fix it?" level="basic" >}}

**Answer:**

`777` means **everyone** (owner, group, others) can read, write, and execute. This is dangerous because:
- Any user on the system can modify or delete the file.
- If it's a script, any user can execute it (privilege escalation risk).
- Web servers shouldn't have world-writable files.

```bash
# Check permissions
ls -la sensitive_file.sh

# Fix: Apply least-privilege permissions
chmod 750 sensitive_file.sh    # Owner: rwx, Group: r-x, Others: ---
chmod 640 config.env           # Owner: rw-, Group: r--, Others: ---
chmod 600 private_key.pem      # Owner: rw-, Others: none

# Fix ownership
sudo chown ubuntu:ubuntu sensitive_file.sh

# Find all 777 files in a directory
find /var/www -perm 777 -type f
find /var/www -perm 777 -exec chmod 644 {} \;
```

{{< /qa >}}
{{< qa num="15" q="How do you create a service account (non-login user) for running an application?" level="basic" >}}

**Answer:**

```bash
# Create a system user with no login shell and no home directory
sudo useradd --system --no-create-home --shell /usr/sbin/nologin appuser

# Verify
id appuser
grep appuser /etc/passwd

# Assign application directory ownership
sudo chown -R appuser:appuser /opt/myapp

# Run a service as this user (systemd service file)
# In /etc/systemd/system/myapp.service:
# [Service]
# User=appuser
# Group=appuser

# Verify the user cannot log in
sudo su - appuser
# Output: This account is currently not available.
```

{{< /qa >}}
{{< qa num="16" q="Your apt update fails with 'Failed to fetch' errors. How do you fix it?" level="basic" >}}

**Answer:**

```bash
# Step 1: Check internet connectivity
ping -c 3 google.com
curl -I http://archive.ubuntu.com

# Step 2: Check DNS
nslookup archive.ubuntu.com

# Step 3: Clear apt cache and try again
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update

# Step 4: Check for broken sources
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Step 5: Fix broken packages
sudo apt --fix-broken install
sudo dpkg --configure -a

# Step 6: If behind a proxy, set it
export http_proxy=http://proxy:3128
export https_proxy=http://proxy:3128
sudo apt update
```
{{< /qa >}}
{{< qa num="17" q="You need to install a specific version of a package. How do you do it?" level="basic" >}}

**Answer:**

```bash
# List available versions
apt-cache madison nginx
apt-cache showpkg nginx

# Install a specific version
sudo apt install nginx=1.18.0-0ubuntu1

# Prevent auto-upgrade (hold the package)
sudo apt-mark hold nginx

# Verify hold
apt-mark showhold

# Unhold when ready to upgrade
sudo apt-mark unhold nginx
```

{{< /qa >}}
{{< qa num="18" q="A critical service keeps crashing and restarting. How do you diagnose it?" level="intermediate" >}}

**Answer:**

```bash
# Check service status
sudo systemctl status nginx

# View recent logs
sudo journalctl -u nginx -n 100
sudo journalctl -u nginx --since "1 hour ago"

# Follow logs in real-time
sudo journalctl -u nginx -f

# Check how many times it restarted
sudo journalctl -u nginx | grep "Started\|Failed"

# Check restart policy in service file
sudo systemctl cat nginx | grep Restart

# Common fixes:
# 1. Check config file syntax
sudo nginx -t

# 2. Check port conflicts
ss -tulnp | grep :80

# 3. Check file permissions
ls -la /var/run/nginx.pid

# 4. View crash details
sudo dmesg | tail -20
sudo coredumpctl list
```

{{< /qa >}}
{{< qa num="19" q="How do you create a custom systemd service for your application?" level="intermediate" >}}

**Answer:**

```bash
# Create the service file
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application Service
After=network.target
Wants=network.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
Environment=ENV=production
EnvironmentFile=/opt/myapp/.env

[Install]
WantedBy=multi-user.target
```

```bash
# Reload systemd and enable
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```

{{< /qa >}}
{{< qa num="20" q="Your /var/log partition is full due to excessive logging. How do you handle it?" level="intermediate" >}}

**Answer:**

```bash
# Identify large log files
sudo du -sh /var/log/* | sort -rh | head -10

# Check journald size
journalctl --disk-usage

# Immediate cleanup
sudo journalctl --vacuum-size=500M     # Keep only 500MB
sudo journalctl --vacuum-time=7d       # Keep only 7 days

# Rotate logs now
sudo logrotate -f /etc/logrotate.conf

# Delete old compressed logs
sudo find /var/log -name "*.gz" -mtime +7 -delete

# Configure journald limits permanently
sudo nano /etc/systemd/journald.conf
```

```ini
[Journal]
SystemMaxUse=500M
SystemKeepFree=1G
MaxRetentionSec=7day
```

```bash
sudo systemctl restart systemd-journald
```

{{< /qa >}}
{{< qa num="21" q="How do you monitor an EC2 instance for high CPU and trigger an alert?" level="intermediate" >}}

**Answer:**

**Using AWS CloudWatch:**
```bash
# Install CloudWatch agent
sudo apt install amazon-cloudwatch-agent -y

# Configure the agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

**Create a CloudWatch Alarm via CLI:**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU-i-xxxxxxxx" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --dimensions Name=InstanceId,Value=i-xxxxxxxx \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789:MySNSTopic \
  --unit Percent
```

**On the instance with a simple bash monitor:**
```bash
#!/bin/bash
# /usr/local/bin/cpu_alert.sh
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
THRESHOLD=80
if (( $(echo "$CPU > $THRESHOLD" | bc -l) )); then
  echo "HIGH CPU: ${CPU}%" | mail -s "CPU Alert on $(hostname)" admin@example.com
fi
```

{{< /qa >}}
{{< qa num="22" q="A cron job is not running as expected. How do you debug it?" level="basic" >}}

**Answer:**

```bash
# Step 1: Verify cron syntax
crontab -l
# Example: */5 * * * * /path/to/script.sh

# Step 2: Check cron service
sudo systemctl status cron

# Step 3: Check cron logs
grep CRON /var/log/syslog | tail -50
sudo journalctl -u cron -n 50

# Step 4: Common mistakes
# - Script not executable
chmod +x /path/to/script.sh

# - No PATH in cron environment
# Add to top of crontab:
PATH=/usr/local/bin:/usr/bin:/bin

# - Redirect output to log for debugging
*/5 * * * * /path/to/script.sh >> /tmp/cron_debug.log 2>&1

# Step 5: Test script manually as cron user
sudo su -s /bin/bash -c '/path/to/script.sh' ubuntu

# Step 6: Check for environment variable issues
# Cron has minimal env; use full paths for all commands
```

{{< /qa >}}
{{< qa num="23" q="How do you schedule a job to run at 2 AM every Sunday for a database backup?" level="basic" >}}

**Answer:**

```bash
# Open crontab
crontab -e

# Add the entry (minute hour day month weekday)
0 2 * * 0 /opt/scripts/backup_db.sh >> /var/log/db_backup.log 2>&1

# Cron field reference:
# 0      = minute (0-59)
# 2      = hour (0-23)
# *      = any day of month
# *      = any month
# 0      = Sunday (0=Sun, 1=Mon, ..., 6=Sat)

# Verify cron entry
crontab -l

# Sample backup script
cat << 'EOF' > /opt/scripts/backup_db.sh
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/db"
mkdir -p $BACKUP_DIR
mysqldump -u root -p${DB_PASS} mydb > $BACKUP_DIR/mydb_$DATE.sql
# Upload to S3
aws s3 cp $BACKUP_DIR/mydb_$DATE.sql s3://my-backup-bucket/db/
# Keep only last 7 local backups
ls -t $BACKUP_DIR/*.sql | tail -n +8 | xargs rm -f
echo "Backup completed: $DATE"
EOF
chmod +x /opt/scripts/backup_db.sh
```

{{< /qa >}}
{{< qa num="24" q="How do you take a snapshot of an EBS volume and restore it?" level="basic" >}}

**Answer:**

**Create Snapshot:**
```bash
# Get volume ID
aws ec2 describe-instances --instance-ids i-xxxxxxxx \
  --query 'Reservations[].Instances[].BlockDeviceMappings[].Ebs.VolumeId'

# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxx \
  --description "Pre-deployment backup $(date +%Y-%m-%d)"

# Wait for completion
aws ec2 wait snapshot-completed --snapshot-ids snap-xxxxxxxx
```

**Restore from Snapshot:**
```bash
# Create new volume from snapshot
aws ec2 create-volume \
  --snapshot-id snap-xxxxxxxx \
  --availability-zone us-east-1a \
  --volume-type gp3

# Stop the instance
aws ec2 stop-instances --instance-ids i-xxxxxxxx

# Detach current root volume
aws ec2 detach-volume --volume-id vol-current

# Attach restored volume
aws ec2 attach-volume \
  --volume-id vol-newFromSnapshot \
  --instance-id i-xxxxxxxx \
  --device /dev/xvda

# Start instance
aws ec2 start-instances --instance-ids i-xxxxxxxx
```

{{< /qa >}}
{{< qa num="25" q="Your EC2 instance was accidentally terminated. How do you recover?" level="advanced" >}}

**Answer:**

**Prevention (Termination Protection):**
```bash
# Enable termination protection
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxx \
  --disable-api-termination
```

**Recovery options:**
1. **From AMI** – If you had a scheduled AMI backup:
```bash
# Launch new instance from the AMI
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx
```

2. **From EBS snapshot** – Restore volume and attach to a new instance.

3. **From S3 backups** – If application data was being backed up to S3, restore from there.

**Best practice – Automate AMI creation:**
```bash
aws ec2 create-image \
  --instance-id i-xxxxxxxx \
  --name "backup-$(date +%Y-%m-%d)" \
  --no-reboot
```

{{< /qa >}}
{{< qa num="26" q="You notice unusual outbound traffic from your EC2 instance. How do you investigate?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Check active connections
ss -tunp
netstat -tunp

# Step 2: Identify process making connections
sudo lsof -i -n -P | grep ESTABLISHED

# Step 3: Check VPC Flow Logs
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern "REJECT" \
  --start-time $(date -d '1 hour ago' +%s000)

# Step 4: Check for unknown cron jobs
crontab -l
sudo cat /etc/crontab
ls -la /etc/cron.*

# Step 5: Check recently modified files
find / -mtime -1 -type f 2>/dev/null | grep -v proc

# Step 6: Check suspicious processes
ps auxf
sudo chkrootkit
sudo rkhunter --check

# Step 7: Isolate if compromised
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxx \
  --groups sg-isolate   # SG with no outbound rules
```
{{< /qa >}}
{{< qa num="27" q="How do you harden a fresh Ubuntu EC2 instance?" level="advanced" >}}

**Answer:**

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Configure SSH hardening
sudo nano /etc/ssh/sshd_config
```

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers ubuntu
```

```bash
sudo systemctl restart sshd

# 3. Configure UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# 4. Install fail2ban
sudo apt install fail2ban -y
sudo systemctl enable fail2ban

# 5. Disable unnecessary services
sudo systemctl disable bluetooth
sudo systemctl disable cups

# 6. Set up automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades

# 7. Configure sysctl hardening
sudo nano /etc/sysctl.conf
```

```
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_syncookies = 1
```
{{< /qa >}}
{{< qa num="28" q="Your application needs to handle traffic spikes automatically. How do you set up Auto Scaling?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Create a launch template
aws ec2 create-launch-template \
  --launch-template-name my-app-template \
  --launch-template-data '{
    "ImageId": "ami-xxxxxxxx",
    "InstanceType": "t3.medium",
    "KeyName": "my-key",
    "SecurityGroupIds": ["sg-xxxxxxxx"],
    "UserData": "base64-encoded-user-data"
  }'

# Step 2: Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-app-template,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-xxxx,subnet-yyyy" \
  --target-group-arns arn:aws:elasticloadbalancing:...

# Step 3: Create scaling policy (Target Tracking)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 60.0
  }'
```

{{< /qa >}}
{{< qa num="29" q="How do you create a Golden AMI for consistent deployments?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Launch base instance and configure it
# Install all software, configs, security hardening

# Step 2: Run cleanup before creating AMI
sudo apt clean
sudo rm -rf /tmp/* /var/tmp/*
sudo find /var/log -type f -exec truncate -s 0 {} \;
# Remove SSH host keys (they'll regenerate on new instance)
sudo rm -f /etc/ssh/ssh_host_*
# Remove ubuntu user's authorized_keys (optional)
# sudo rm /home/ubuntu/.ssh/authorized_keys

# Step 3: Create AMI
aws ec2 create-image \
  --instance-id i-xxxxxxxx \
  --name "golden-ami-$(date +%Y%m%d)" \
  --description "Hardened Ubuntu 22.04 with app stack" \
  --no-reboot

# Step 4: Tag the AMI
aws ec2 create-tags \
  --resources ami-xxxxxxxx \
  --tags Key=Environment,Value=Production Key=Version,Value=1.0.0

# Step 5: Share with other accounts (if needed)
aws ec2 modify-image-attribute \
  --image-id ami-xxxxxxxx \
  --launch-permission "Add=[{UserId=123456789012}]"
```

{{< /qa >}}
{{< qa num="30" q="Your application requires environment variables that should not be hardcoded. How do you manage them?" level="intermediate" >}}

**Answer:**

**Method 1 – .env file with restricted permissions:**
```bash
sudo nano /opt/myapp/.env
# DB_PASSWORD=secret123
# API_KEY=abc123

sudo chmod 600 /opt/myapp/.env
sudo chown appuser:appuser /opt/myapp/.env

# Load in systemd service
# EnvironmentFile=/opt/myapp/.env
```

**Method 2 – AWS Systems Manager Parameter Store:**
```bash
# Store secret
aws ssm put-parameter \
  --name "/myapp/db_password" \
  --value "secret123" \
  --type SecureString

# Retrieve in script
DB_PASS=$(aws ssm get-parameter \
  --name "/myapp/db_password" \
  --with-decryption \
  --query "Parameter.Value" \
  --output text)
export DB_PASS
```

**Method 3 – AWS Secrets Manager:**
```bash
# Store
aws secretsmanager create-secret \
  --name myapp/credentials \
  --secret-string '{"db_password":"secret123"}'

# Retrieve
SECRET=$(aws secretsmanager get-secret-value \
  --secret-id myapp/credentials \
  --query SecretString \
  --output text)
DB_PASS=$(echo $SECRET | python3 -c "import sys,json; print(json.load(sys.stdin)['db_password'])")
```

{{< /qa >}}
{{< qa num="31" q="Your instance doesn't have enough RAM. How do you add swap space without stopping the instance?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Check current swap
free -h
swapon --show

# Step 2: Create a swap file (2GB)
sudo fallocate -l 2G /swapfile
# If fallocate doesn't work:
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048

# Step 3: Secure permissions
sudo chmod 600 /swapfile

# Step 4: Set up swap
sudo mkswap /swapfile
sudo swapon /swapfile

# Step 5: Verify
free -h
swapon --show

# Step 6: Make persistent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Step 7: Tune swappiness (how aggressively to use swap)
# Default is 60; lower = prefer RAM
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

{{< /qa >}}
{{< qa num="32" q="After a reboot, your mounted EBS volume is no longer mounted. How do you make it persistent?" level="basic" >}}

**Answer:**

```bash
# Step 1: Find the device UUID (UUID is persistent, device name like /dev/xvdf may change)
sudo blkid

# Example output:
# /dev/xvdf: UUID="a1b2c3d4-e5f6-..." TYPE="ext4"

# Step 2: Edit /etc/fstab
sudo cp /etc/fstab /etc/fstab.backup  # Always backup first!
sudo nano /etc/fstab

# Add the line:
# UUID=a1b2c3d4-e5f6-...  /data  ext4  defaults,nofail  0  2

# IMPORTANT: Use 'nofail' option so instance boots even if disk fails

# Step 3: Test without rebooting
sudo mount -a

# Step 4: Verify
df -h /data

# Step 5: Test fstab syntax
sudo findmnt --verify --verbose
```

{{< /qa >}}
{{< qa num="33" q="How do you check and repair a corrupted filesystem on Ubuntu?" level="advanced" >}}

**Answer:**

```bash
# IMPORTANT: fsck must run on unmounted filesystem

# Step 1: Identify the filesystem
lsblk
df -hT

# Step 2: Unmount (if not root)
sudo umount /dev/xvdf

# Step 3: Run fsck
sudo fsck -y /dev/xvdf    # -y auto-answers yes to repairs

# For ext4 specifically
sudo e2fsck -f /dev/xvdf1

# For xfs
sudo xfs_repair /dev/xvdf1

# If it's the root filesystem:
# Boot into rescue mode or attach to another instance
# Or force fsck on next reboot:
sudo touch /forcefsck
# OR
sudo tune2fs -C 1 /dev/xvda1  # Force check after 1 mount

# Step 4: Re-mount
sudo mount /dev/xvdf /data
```

{{< /qa >}}
{{< qa num="34" q="How do you set up SSH tunneling to access a private RDS database through an EC2 bastion host?" level="advanced" >}}

**Answer:**

```bash
# Scenario: RDS is in private subnet, only accessible from EC2 (bastion)
# Goal: Access RDS from local machine via EC2

# Method 1: SSH Local Port Forwarding
ssh -i bastion-key.pem \
  -L 3306:rds-endpoint.rds.amazonaws.com:3306 \
  ubuntu@<bastion-public-ip> \
  -N -f

# Now connect to RDS locally:
mysql -h 127.0.0.1 -P 3306 -u admin -p

# Method 2: SSH Config for convenience (~/.ssh/config)
Host bastion
  HostName <bastion-public-ip>
  User ubuntu
  IdentityFile ~/.ssh/bastion-key.pem

Host rds-tunnel
  HostName rds-endpoint.rds.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/bastion-key.pem
  ProxyJump bastion
  LocalForward 3306 rds-endpoint.rds.amazonaws.com:3306

# Method 3: Dynamic SOCKS proxy
ssh -i bastion-key.pem -D 1080 ubuntu@<bastion-ip> -N -f
```

{{< /qa >}}
{{< qa num="35" q="How do you rotate SSH keys on multiple EC2 instances without losing access?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Generate new key pair
ssh-keygen -t ed25519 -f ~/.ssh/new_ec2_key -C "new-key-2024"

# Step 2: Add new public key to instances BEFORE removing old key
# Do NOT remove old key until new key is confirmed working

# Using AWS SSM Run Command (safest approach):
aws ssm send-command \
  --instance-ids i-xxx i-yyy i-zzz \
  --document-name "AWS-RunShellScript" \
  --parameters commands=["echo '$(cat ~/.ssh/new_ec2_key.pub)' >> /home/ubuntu/.ssh/authorized_keys"]

# Step 3: Verify new key works
ssh -i ~/.ssh/new_ec2_key ubuntu@<instance-ip>

# Step 4: Remove old key from authorized_keys
# (Only after confirming new key works on all instances)
aws ssm send-command \
  --instance-ids i-xxx i-yyy i-zzz \
  --document-name "AWS-RunShellScript" \
  --parameters commands=["sed -i '/OLD_KEY_COMMENT/d' /home/ubuntu/.ssh/authorized_keys"]
```

{{< /qa >}}
{{< qa num="36" q="How does an EC2 instance access AWS services without hardcoding credentials?" level="intermediate" >}}

**Answer:**

EC2 instances use **IAM Instance Profiles** to get temporary credentials via the Instance Metadata Service (IMDS).

```bash
# Attach IAM role during launch or after:
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxxxxx \
  --iam-instance-profile Name=my-ec2-role

# From inside the instance, credentials are auto-fetched:
# AWS SDK/CLI automatically uses IMDS

# Manually fetch credentials from IMDS (IMDSv2):
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Fetch the actual credentials
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/my-ec2-role

# Get instance ID
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id

# Verify from CLI
aws sts get-caller-identity
```

**Enforce IMDSv2 (security best practice):**
```bash
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxxxxxx \
  --http-tokens required \
  --http-endpoint enabled
```

{{< /qa >}}
{{< qa num="37" q="Your application behind an ALB is returning 502 errors. How do you troubleshoot?" level="advanced" >}}

**Answer:**

**502 Bad Gateway = ALB cannot communicate with target (EC2 instance).**

```bash
# Step 1: Check Target Group health
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:...

# Possible states: healthy, unhealthy, draining, unused

# Step 2: On the EC2 instance, verify app is running and listening
ss -tulnp | grep :8080
curl http://localhost:8080/health

# Step 3: Check Security Group
# ALB SG must allow outbound to EC2 SG on app port
# EC2 SG must allow inbound from ALB SG

# Step 4: Check Health Check configuration
# Health check path, port, protocol, thresholds

# Step 5: Check application logs
sudo journalctl -u myapp -n 100

# Step 6: Check ALB access logs (if enabled)
aws s3 ls s3://my-alb-logs/
# Look for 502 entries
```
{{< /qa >}}
{{< qa num="38" q="How do you implement a Blue/Green deployment on EC2?" level="advanced" >}}

**Answer:**

```bash
# Blue = current production, Green = new version

# Step 1: Create Green Auto Scaling Group with new AMI
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name myapp-green \
  --launch-template LaunchTemplateName=myapp-v2,Version='$Latest' \
  --min-size 2 --max-size 10 --desired-capacity 2 \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/myapp-green/...

# Step 2: Register Green target group with ALB (weighted routing)
aws elbv2 modify-rule \
  --rule-arn arn:... \
  --actions Type=forward,ForwardConfig='{
    "TargetGroups": [
      {"TargetGroupArn": "arn:...:blue", "Weight": 80},
      {"TargetGroupArn": "arn:...:green", "Weight": 20}
    ]
  }'

# Step 3: Gradually shift traffic to Green
# 50/50 → 0/100

# Step 4: Full cutover
aws elbv2 modify-listener \
  --listener-arn arn:... \
  --default-actions Type=forward,TargetGroupArn=arn:...:green

# Step 5: Decommission Blue ASG
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name myapp-blue \
  --min-size 0 --desired-capacity 0
```

{{< /qa >}}
{{< qa num="39" q="How do you run Docker on an Ubuntu EC2 instance and manage container lifecycle?" level="intermediate" >}}

**Answer:**

```bash
# Install Docker
sudo apt update
sudo apt install -y ca-certificates curl gnupg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker ubuntu
newgrp docker

# Common container management commands
docker run -d --name myapp -p 80:8080 --restart unless-stopped myimage:latest

# View running containers
docker ps

# View logs
docker logs -f myapp

# Inspect resource usage
docker stats

# Stop and remove
docker stop myapp && docker rm myapp

# Prune unused resources
docker system prune -af --volumes

# Auto-start on EC2 reboot
sudo systemctl enable docker
```

{{< /qa >}}
{{< qa num="40" q="A Docker container on EC2 is consuming too much memory. How do you limit it?" level="intermediate" >}}

**Answer:**

```bash
# Set memory limit when running
docker run -d \
  --name myapp \
  --memory="512m" \
  --memory-swap="1g" \
  --cpus="1.5" \
  myimage:latest

# Check current resource usage
docker stats myapp

# Update running container limits
docker update --memory="1g" --memory-swap="2g" myapp

# Check OOM kills for container
docker inspect myapp | grep -i oom

# Set limits in docker-compose.yml
cat << 'EOF' > docker-compose.yml
services:
  myapp:
    image: myimage:latest
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 512M
        reservations:
          memory: 256M
EOF
```

{{< /qa >}}
{{< qa num="41" q="Your EC2 instance is in a 'stopped' state and won't start. What do you check?" level="advanced" >}}

**Answer:**

```bash
# Check instance state and status
aws ec2 describe-instance-status --instance-ids i-xxxxxxxx

# Common reasons and fixes:

# 1. InsufficientInstanceCapacity
# Fix: Change AZ or instance type
aws ec2 start-instances --instance-ids i-xxxxxxxx
# If capacity error, try a different instance type or AZ

# 2. InstanceLimitExceeded
# Fix: Request limit increase or terminate unused instances
aws ec2 describe-account-attributes

# 3. Corrupted filesystem (can't boot)
# Fix: Attach volume to rescue instance, run fsck

# 4. Check system logs in console
aws ec2 get-console-output --instance-ids i-xxxxxxxx

# 5. EBS volume issue
aws ec2 describe-volumes --filters "Name=attachment.instance-id,Values=i-xxxxxxxx"

# 6. Billing issue
# Check AWS account status in console
```

{{< /qa >}}
{{< qa num="42" q="How do you debug a startup script (user data) that didn't run correctly?" level="intermediate" >}}

**Answer:**

```bash
# User data runs once at first launch by cloud-init

# Check cloud-init logs
sudo cat /var/log/cloud-init.log
sudo cat /var/log/cloud-init-output.log

# Check cloud-init status
sudo cloud-init status --long

# Common issues:
# 1. Missing shebang
# First line must be: #!/bin/bash

# 2. Script not running in correct order
# Check: sudo cloud-init analyze show

# 3. Force re-run of user data (for testing)
sudo cloud-init clean --logs
sudo cloud-init init

# 4. Check if user data was properly base64 encoded
aws ec2 describe-instance-attribute \
  --instance-id i-xxxxxxxx \
  --attribute userData \
  --query 'UserData.Value' \
  --output text | base64 --decode

# 5. Run the script manually to test
sudo bash /var/lib/cloud/instance/scripts/part-001
```

{{< /qa >}}
{{< qa num="43" q="You need to find which process opened a specific file or port. How do you do it?" level="basic" >}}

**Answer:**

```bash
# Find process using a specific port
sudo ss -tulnp | grep :80
sudo lsof -i :80
sudo fuser 80/tcp

# Find process that has a file open
sudo lsof /var/log/nginx/access.log

# Find all files opened by a process (PID 1234)
sudo lsof -p 1234

# Find which process is using a file (even deleted)
sudo lsof | grep deleted | head -20

# Find process holding a lock on a file
sudo fuser -v /var/lock/myapp.lock

# Kill process using a port
sudo fuser -k 80/tcp

# Network connections by process
sudo ss -tulnp
sudo netstat -tulnp
```

{{< /qa >}}
{{< qa num="44" q="How do you perform a zero-downtime deployment on a single EC2 instance?" level="advanced" >}}

**Answer:**

```bash
# Strategy: Use a reverse proxy (nginx) with upstream switching

# Nginx config with two upstream backends
sudo nano /etc/nginx/sites-available/myapp
```

```nginx
upstream myapp {
  server 127.0.0.1:8080;  # Current version
}

server {
  listen 80;
  location / {
    proxy_pass http://myapp;
  }
}
```

```bash
# Deploy new version on port 8081
./deploy_new_version.sh --port 8081

# Test new version
curl http://localhost:8081/health

# Switch nginx upstream (hot reload = zero downtime)
sudo sed -i 's/8080/8081/' /etc/nginx/sites-available/myapp
sudo nginx -t && sudo nginx -s reload

# Verify traffic is hitting new version
curl http://localhost/version

# Stop old version
sudo systemctl stop myapp-8080

# For rolling deployments at scale, prefer Auto Scaling + ALB
```
{{< /qa >}}


## 📌 Quick Reference Cheat Sheet

| Task | Command |
|------|---------|
| Check disk usage | `df -hT` |
| Check memory | `free -h` |
| Check CPU | `top` / `htop` |
| List open ports | `ss -tulnp` |
| Check service status | `systemctl status <svc>` |
| View system logs | `journalctl -xe` |
| Check running processes | `ps aux` |
| Find large files | `find / -size +500M -type f` |
| Check failed logins | `grep "Failed password" /var/log/auth.log` |
| Check EC2 instance type | `curl http://169.254.169.254/latest/meta-data/instance-type` |
| Current user | `whoami` / `id` |
| Network interfaces | `ip addr` / `ifconfig` |
| Route table | `ip route` |
| DNS resolution | `nslookup google.com` / `dig google.com` |
| Check firewall | `sudo ufw status` |

---

*Last Updated: 2024 | Coverage: Ubuntu 20.04 / 22.04 on AWS EC2*
