---
title: "Linux Interview Questions & Answers (2026) Part 03"
description: "40+ Linux interview questions and answers covering processes, file system, permissions, services, and troubleshooting — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["linux", "interview", "sysadmin", "devops"]
tool: "linux"
level: "All Levels"
question_count: 52
draft: "false"
---



{{< qa num="1" q="You cannot SSH into your EC2 instance. How do you systematically troubleshoot?" level="basic" >}}

**Answer:**

Work through each layer from outside in:

**Layer 1 – AWS-level checks:**
```bash
# Check instance is running
aws ec2 describe-instances --instance-ids i-xxxxxxxx \
  --query 'Reservations[].Instances[].[State.Name,PublicIpAddress]'

# Check Security Group allows port 22 inbound
aws ec2 describe-security-groups --group-ids sg-xxxxxxxx \
  --query 'SecurityGroups[].IpPermissions'

# Check Network ACL for subnet
aws ec2 describe-network-acls \
  --filters Name=association.subnet-id,Values=subnet-xxxxxxxx
```

**Layer 2 – Network path checks:**
```bash
# Test TCP reachability (from your machine)
nc -zv <ec2-public-ip> 22
telnet <ec2-public-ip> 22

# Traceroute to spot where packets drop
traceroute <ec2-public-ip>
```

**Layer 3 – Instance-level checks (via SSM if network is the issue):**
```bash
# Connect via Session Manager instead
aws ssm start-session --target i-xxxxxxxx

# Check SSHD is running
sudo systemctl status sshd

# Check SSHD is listening
sudo ss -tulnp | grep :22

# Check auth logs for clues
sudo tail -50 /var/log/auth.log
```

**Layer 4 – Key and user checks:**
```bash
# Correct permissions on local key
chmod 400 mykey.pem

# Correct username (ubuntu for Ubuntu AMI, ec2-user for Amazon Linux)
ssh -i mykey.pem ubuntu@<ip>

# Verbose mode for details
ssh -vvv -i mykey.pem ubuntu@<ip>
```

**Decision tree summary:**

| Symptom | Likely Cause |
|---------|-------------|
| Connection refused | SSHD not running or port 22 blocked at SG |
| Connection times out | Security Group or NACL blocking port 22 |
| Permission denied | Wrong key, wrong user, or corrupted authorized_keys |
| Network unreachable | No route / wrong public IP / instance in private subnet |

{{< /qa >}}
{{< qa num="2" q="SSH connection hangs at 'debug1: expecting SSH2_MSG_KEX_ECDH_REPLY'. What is wrong?" level="advanced" >}}

**Answer:**

This hang during key exchange typically means the TCP connection was established but the SSH handshake stalls. Common causes:

**Cause 1 – MTU mismatch (most common on EC2):**
```bash
# Test with reduced packet size
ping -M do -s 1400 <ec2-ip>

# Fix on client side
ssh -o "IPQoS=throughput" -i key.pem ubuntu@<ip>

# Fix on the EC2 instance (via SSM)
sudo ip link set dev eth0 mtu 1450
# Make persistent
echo 'MTU=1450' | sudo tee -a /etc/network/interfaces.d/eth0.cfg
```

**Cause 2 – SSHD is overwhelmed or misconfigured:**
```bash
# Check SSHD process count
ps aux | grep sshd | wc -l

# Check MaxStartups setting in sshd_config
grep MaxStartups /etc/ssh/sshd_config
# Default: 10:30:100 (start throttling at 10, drop at 100)

# Increase if needed
sudo sed -i 's/#MaxStartups.*/MaxStartups 50:30:200/' /etc/ssh/sshd_config
sudo systemctl reload sshd
```

**Cause 3 – Firewall doing stateful inspection and dropping packets:**
```bash
# Test with a different cipher
ssh -c aes128-ctr -i key.pem ubuntu@<ip>
```

{{< /qa >}}
{{< qa num="3" q="You get 'Host key verification failed' when SSHing. How do you fix it?" level="basic" >}}

**Answer:**

This happens when the server's host key no longer matches the stored key in `~/.ssh/known_hosts`. Common on EC2 when an instance is replaced or rebuilt.

```bash
# View the offending line number from the error message
# "Offending ECDSA key in /home/user/.ssh/known_hosts:42"

# Option 1: Remove just the specific host entry
ssh-keygen -R <ec2-ip>
# or
ssh-keygen -R <hostname>

# Option 2: Remove the specific line manually
sed -i '42d' ~/.ssh/known_hosts

# Option 3: For automation/scripting (suppresses strict checking)
ssh -o StrictHostKeyChecking=no -i key.pem ubuntu@<ip>
# WARNING: Only use this in trusted environments

# After removal, SSH again — it will ask to add the new key
ssh -i key.pem ubuntu@<ip>
# Type 'yes' to accept new host key
```

**Root cause prevention:** When creating AMIs and launching new instances, host keys regenerate. Use `StrictHostKeyChecking=accept-new` in scripts instead of `no`.

{{< /qa >}}
{{< qa num="4" q="SSH is extremely slow to connect (30+ seconds). What could cause this?" level="intermediate" >}}

**Answer:**

**Cause 1 – Reverse DNS lookup timeout (most common):**
```bash
# On the EC2 instance, disable DNS lookup in SSHD
sudo nano /etc/ssh/sshd_config
# Set:
# UseDNS no

sudo systemctl reload sshd
```

**Cause 2 – GSSAPI authentication timeout:**
```bash
# On the client side
ssh -o GSSAPIAuthentication=no -i key.pem ubuntu@<ip>

# Permanent fix in ~/.ssh/config
Host *
  GSSAPIAuthentication no
  UseDNS no
```

**Cause 3 – PAM delay or motd scripts:**
```bash
# On the EC2 instance via SSM
# Disable slow motd scripts
sudo chmod -x /etc/update-motd.d/90-updates-available
sudo chmod -x /etc/update-motd.d/91-release-upgrade

# Check PAM configuration
sudo nano /etc/pam.d/sshd
# Comment out unnecessary pam_motd lines
```

**Cause 4 – High system load causing delay:**
```bash
uptime
# If load is high, the instance itself is slow to respond
top
```

{{< /qa >}}
{{< qa num="5" q="You are locked out of your EC2 instance after modifying /etc/ssh/sshd_config. How do you recover?" level="advanced" >}}

**Answer:**

Since SSH is broken, use out-of-band access:

**Method 1 – AWS Systems Manager Session Manager (preferred if SSM agent is running):**
```bash
aws ssm start-session --target i-xxxxxxxx

# Fix the sshd_config
sudo nano /etc/ssh/sshd_config

# Validate config before restarting
sudo sshd -t

# Restart SSHD
sudo systemctl restart sshd
```

**Method 2 – EC2 Serial Console (if enabled for the account):**
- AWS Console → EC2 → Instance → Connect → EC2 Serial Console

**Method 3 – Volume detach and repair:**
```bash
# 1. Stop instance
aws ec2 stop-instances --instance-ids i-xxxxxxxx

# 2. Detach root volume
aws ec2 detach-volume --volume-id vol-xxxxxxxx

# 3. Attach to a rescue instance as /dev/xvdf
aws ec2 attach-volume --volume-id vol-xxxxxxxx \
  --instance-id i-rescue --device /dev/xvdf

# 4. On rescue instance, fix the config
sudo mount /dev/xvdf1 /mnt/rescue
sudo nano /mnt/rescue/etc/ssh/sshd_config
# Fix whatever was changed
sudo umount /mnt/rescue

# 5. Re-attach to original instance and start
```

**Prevention:** Always validate with `sudo sshd -t` before restarting SSHD.

{{< /qa >}}
{{< qa num="6" q="Too many authentication failures error appears when SSHing. How do you fix it?" level="intermediate" >}}

**Answer:**

OpenSSH tries all keys in your agent before the correct one, exhausting the `MaxAuthTries` limit.

```bash
# Immediately fix by specifying only the correct key and disabling agent
ssh -i mykey.pem -o IdentitiesOnly=yes ubuntu@<ip>

# Permanent fix in ~/.ssh/config
Host <ec2-ip>
  IdentityFile ~/.ssh/mykey.pem
  IdentitiesOnly yes
  User ubuntu

# Check how many keys your SSH agent is holding
ssh-add -l

# Remove all keys from agent if too many
ssh-add -D

# On the server side, increase MaxAuthTries if needed (via SSM)
sudo grep MaxAuthTries /etc/ssh/sshd_config
# Set: MaxAuthTries 10
sudo systemctl reload sshd
```

{{< /qa >}}
{{< qa num="7" q="df -h shows disk is 100% full but du cannot find large files. What is happening?" level="advanced" >}}

**Answer:**

This is a classic **deleted-but-open file** problem. A process deleted a file but still holds a file descriptor open — the OS cannot free the space until that process releases it.

```bash
# Step 1: Confirm the mismatch
df -h /
du -sh /* 2>/dev/null | sort -rh | head -10
# du total will be much less than df shows

# Step 2: Find deleted files still held open
sudo lsof | grep deleted
sudo lsof +L1   # Files with link count < 1 (deleted but open)

# Example output:
# nginx  1234  root  10w  REG  202,1  5368709120  12345 /var/log/nginx/access.log (deleted)

# Step 3a: Restart the holding process to release the file
sudo systemctl restart nginx

# Step 3b: If you cannot restart, truncate the file descriptor in place
# Get PID and FD number from lsof output
sudo truncate -s 0 /proc/1234/fd/10
# Space is freed immediately without restarting the process
```
{{< /qa >}}
{{< qa num="8" q="You extended an EBS volume in AWS console but the OS still shows the old size. Why?" level="basic" >}}

**Answer:**

AWS resizes the block device, but the partition table and filesystem must be extended separately inside the OS.

```bash
# Step 1: Verify AWS-side resize completed
aws ec2 describe-volumes-modifications --volume-id vol-xxxxxxxx
# Wait until ModificationState = "completed"

# Step 2: Check block device sees new size
lsblk
# /dev/xvda should show new size

# Step 3: Extend the partition (if partitioned)
sudo growpart /dev/xvda 1
# "CHANGED" confirms success

# Step 4: Extend the filesystem
# For ext4:
sudo resize2fs /dev/xvda1

# For xfs (Amazon Linux default):
sudo xfs_growfs /

# Step 5: Verify
df -h /
# Should now show new size
```

**If growpart fails with "NOCHANGE"** — the partition already uses full disk; only filesystem resize needed.

{{< /qa >}}
{{< qa num="9" q="The instance boots into emergency mode with 'Give root password for maintenance'. What do you do?" level="advanced" >}}

**Answer:**

Since this is EC2 (no physical console), use the volume detach method:

```bash
# Step 1: Get console output to see the error
aws ec2 get-console-output --instance-ids i-xxxxxxxx --latest \
  --query Output --output text

# Common causes shown in output:
# - Failed to mount /data (bad fstab entry)
# - Filesystem corruption on /dev/xvda1
# - SELinux relabeling issue

# Step 2: Stop the instance
aws ec2 stop-instances --instance-ids i-xxxxxxxx

# Step 3: Detach root volume
aws ec2 detach-volume --volume-id vol-xxxxxxxx

# Step 4: Attach to rescue instance
aws ec2 attach-volume --volume-id vol-xxxxxxxx \
  --instance-id i-rescue --device /dev/xvdf

# Step 5: Mount and repair
sudo mount /dev/xvdf1 /mnt/rescue

# Fix bad fstab entry
sudo nano /mnt/rescue/etc/fstab

# Or run filesystem check
sudo umount /mnt/rescue
sudo fsck -y /dev/xvdf1
sudo mount /dev/xvdf1 /mnt/rescue

sudo umount /mnt/rescue

# Step 6: Re-attach and start
aws ec2 attach-volume --volume-id vol-xxxxxxxx \
  --instance-id i-original --device /dev/xvda
aws ec2 start-instances --instance-ids i-original
```

{{< /qa >}}
{{< qa num="10" q="You see 'Input/output error' when reading or writing files. What does this mean?" level="advanced" >}}

**Answer:**

I/O errors typically indicate **EBS volume degradation, filesystem corruption, or a detached/failing underlying storage**.

```bash
# Step 1: Check kernel messages for I/O errors
sudo dmesg | grep -i "error\|i/o\|blk\|xvd" | tail -30
sudo journalctl -k | grep -i "error\|i/o" | tail -30

# Step 2: Check EBS volume health in AWS
aws ec2 describe-volume-status --volume-ids vol-xxxxxxxx

# Step 3: Check filesystem errors
sudo tune2fs -l /dev/xvda1 | grep -i "mount count\|errors\|last checked"

# Step 4: If the filesystem is corrupted:
# Unmount (if not root volume)
sudo umount /dev/xvdf

# Run fsck
sudo fsck -y /dev/xvdf1

# Step 5: For root volume, schedule fsck on next reboot
sudo touch /forcefsck
sudo reboot

# Step 6: If EBS is degraded, create a snapshot and restore to new volume
aws ec2 create-snapshot --volume-id vol-xxxxxxxx \
  --description "Emergency snapshot before replacement"
```

{{< /qa >}}
{{< qa num="11" q="/tmp is full and causing application failures. How do you fix it without a reboot?" level="basic" >}}

**Answer:**

```bash
# Step 1: Confirm /tmp usage
df -h /tmp
du -sh /tmp/* 2>/dev/null | sort -rh | head -20

# Step 2: Find what is using /tmp
sudo lsof | grep /tmp | sort -k7 -rn | head -20

# Step 3: Safe cleanup — remove old temp files
sudo find /tmp -type f -atime +1 -delete    # Files not accessed in 1 day
sudo find /tmp -type f -size +100M -delete   # Large files

# Step 4: If a specific process is holding /tmp files
# Restart that process after identifying it via lsof

# Step 5: Expand /tmp if it is on tmpfs
# Check current size
mount | grep tmpfs
# Example: tmpfs on /tmp type tmpfs (rw,size=1024m)

# Remount with larger size (no reboot needed)
sudo mount -o remount,size=4G /tmp

# Make permanent in /etc/fstab
# tmpfs  /tmp  tmpfs  defaults,size=4G  0  0

# Step 6: If /tmp shares the root filesystem, check root disk
df -h /
```

{{< /qa >}}
{{< qa num="12" q="After detaching and re-attaching an EBS volume, the device name changed. How do you handle it?" level="intermediate" >}}

**Answer:**

EC2 Linux kernels use NVMe naming (`/dev/nvme1n1`) even if you specify `/dev/xvdf`. Device names are not stable identifiers.

```bash
# Find the new device name
lsblk
# or
ls -la /dev/disk/by-id/
ls -la /dev/disk/by-uuid/

# Get UUID of the volume
sudo blkid

# Mount by UUID (stable across renames)
sudo mount UUID=<uuid> /data

# Fix /etc/fstab to use UUID instead of device name
# WRONG (fragile):
# /dev/xvdf  /data  ext4  defaults  0  2

# RIGHT (stable):
# UUID=a1b2c3d4-xxxx  /data  ext4  defaults,nofail  0  2

# Identify which nvme device corresponds to which EBS volume
sudo nvme list
# Shows SerialNumber which matches EBS volume ID (vol-xxxxxxxx → vol0xxxxxxxx)
```

{{< /qa >}}
{{< qa num="13" q="You accidentally deleted a large file but disk space was not freed. Why?" level="advanced" >}}

**Answer:**

Same as Q7 — a process still holds the file descriptor open. The inode is not freed until all file descriptors are closed.

```bash
# Confirm: file is deleted but inode still in use
sudo lsof | grep deleted

# Example output shows the culprit process:
# java  5678  appuser  22w  REG  ...  /opt/app/logs/app.log (deleted)

# Option 1: Restart the process (cleanest)
sudo systemctl restart myapp

# Option 2: Truncate in-place without restarting
# Get PID=5678, FD=22 from lsof output
sudo ls -la /proc/5678/fd/22   # Verify it's the right file
sudo truncate -s 0 /proc/5678/fd/22
# Disk space freed immediately!

# Option 3: Close the FD using gdb (advanced, no restart)
sudo gdb -p 5678 --batch \
  --eval-command="call close(22)" \
  --eval-command="quit"

# Verify space recovered
df -h /
```

{{< /qa >}}
{{< qa num="14" q="Load average is very high but CPU usage appears low. What is the cause?" level="advanced" >}}

**Answer:**

Load average counts **runnable + uninterruptible-sleep (I/O waiting) processes**. High load + low CPU = I/O bottleneck (disk or network).

```bash
# Step 1: Confirm I/O wait is high
top
# Look for %wa (iowait) in the CPU line
# Example: %Cpu(s): 5.0 us, 1.0 sy, 0.0 ni, 40.0 id, 52.0 wa

# Step 2: Identify the I/O-heavy processes
sudo iotop -o -P
# -o shows only active processes

# Step 3: Identify which disk is busy
iostat -xz 1 5
# Look for: %util close to 100, high await (ms per request)

# Step 4: Check if EBS is throttled (burst credits exhausted)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EBS \
  --metric-name BurstBalance \
  --dimensions Name=VolumeId,Value=vol-xxxxxxxx \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average

# Solutions:
# 1. Upgrade from gp2 to gp3 for consistent IOPS
aws ec2 modify-volume --volume-id vol-xxxxxxxx --volume-type gp3 --iops 3000

# 2. Identify and optimize the I/O-heavy application
# 3. Use instance store (NVMe) for temp files
```

{{< /qa >}}
{{< qa num="15" q="A process is stuck in D state (uninterruptible sleep). What does this mean and how do you handle it?" level="advanced" >}}

**Answer:**

`D` state = process waiting on I/O (usually disk or NFS). It **cannot** be killed with `kill -9` — the kernel must complete the I/O first.

```bash
# Identify D-state processes
ps aux | awk '$8 == "D"'
# or
ps -eo pid,stat,wchan,comm | grep "^[0-9]* D"

# Find what I/O the process is waiting on
sudo cat /proc/<PID>/wchan      # Shows kernel function it's waiting in
sudo strace -p <PID>            # May show what syscall it's stuck in

# Common D-state causes:
# 1. NFS mount timeout
showmount -e <nfs-server>
sudo umount -l /mnt/nfs         # Lazy unmount

# 2. EBS volume detached while in use
aws ec2 describe-volumes --volume-ids vol-xxxxxxxx
# Re-attach or reboot to clear

# 3. Disk I/O error causing retry loop
sudo dmesg | grep -i error | tail -20

# Recovery options:
# - Fix the underlying I/O issue (re-attach EBS, fix NFS, fix disk)
# - Reboot the instance if I/O cannot be resolved
# Note: kill -9 will NOT work on D-state processes
```

{{< /qa >}}
{{< qa num="16" q="Your application is running but responding very slowly. CPU and memory look fine. What else do you check?" level="intermediate" >}}

**Answer:**

```bash
# 1. Check I/O wait
iostat -xz 1
top  # Look at %wa

# 2. Check network latency and drops
ip -s link show eth0
# Look for: RX/TX errors, drops

# 3. Check TCP connection states
ss -s
# Too many TIME_WAIT or CLOSE_WAIT can exhaust ports

# 4. Check open file descriptors (hitting limits?)
ulimit -n                         # Per-process limit
cat /proc/sys/fs/file-max         # System-wide limit
sudo lsof | wc -l                 # Current total

# Check per-process
sudo cat /proc/<PID>/limits | grep "open files"
sudo ls /proc/<PID>/fd | wc -l    # FDs actually open

# 5. Check connection queue (SYN backlog overflow?)
ss -lnt
# Recv-Q > 0 on listening socket = backlog full

# 6. Check for swap thrashing
vmstat 1 5
# High 'si' (swap in) and 'so' (swap out) = memory pressure

# 7. Check application-level timeouts
sudo strace -p <PID> -e trace=network 2>&1 | head -50
```

{{< /qa >}}
{{< qa num="17" q="The OOM killer terminated your application. How do you investigate and prevent it?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Confirm OOM kill happened
sudo dmesg | grep -i "killed process\|oom"
sudo journalctl -k | grep -i oom
sudo grep -i "out of memory\|oom" /var/log/syslog

# Example output:
# [123456.789] Out of memory: Kill process 4567 (java) score 987

# Step 2: Understand why
free -h
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree"

# Step 3: Check OOM score of your process
cat /proc/<PID>/oom_score          # Higher = more likely to be killed
cat /proc/<PID>/oom_score_adj      # -1000 = never kill, 1000 = always kill first

# Step 4: Protect critical processes from OOM killer
echo -500 | sudo tee /proc/<PID>/oom_score_adj
# or permanent via systemd service:
# OOMScoreAdjust=-500

# Step 5: Fix root cause
# Option A: Add swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile

# Option B: Tune application memory (JVM heap, etc.)
# Option C: Upgrade to larger instance type

# Step 6: Set up monitoring/alerting for memory
aws cloudwatch put-metric-alarm \
  --alarm-name HighMemory --metric-name mem_used_percent \
  --namespace CWAgent --statistic Average --period 300 \
  --threshold 85 --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:...:MyTopic
```

{{< /qa >}}
{{< qa num="18" q="fork() failed: Cannot allocate memory even though free -h shows available RAM. Why?" level="advanced" >}}

**Answer:**

This is a **PID limit** or **max processes** exhaustion issue, not a RAM issue.

```bash
# Check current process/thread count
ps aux | wc -l
cat /proc/sys/kernel/pid_max          # Maximum PID value
cat /proc/sys/kernel/threads-max      # Max threads system-wide

# Check per-user process limits
ulimit -u       # Max user processes for current session

# Find which user/process is spawning too many threads
ps -eLf | wc -l                       # Total thread count
ps -eLf | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Fix: Increase limits
# Temporary
sudo sysctl kernel.pid_max=4194304
sudo sysctl kernel.threads-max=4194304

# Permanent
echo 'kernel.pid_max=4194304' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Per-user limit (edit /etc/security/limits.conf)
echo '* soft nproc 65536' | sudo tee -a /etc/security/limits.conf
echo '* hard nproc 65536' | sudo tee -a /etc/security/limits.conf

# For systemd services, set TasksMax
# [Service]
# TasksMax=infinity
```
{{< /qa >}}
{{< qa num="19" q="A zombie process is appearing repeatedly. How do you find and eliminate it?" level="advanced" >}}

**Answer:**

Zombies are processes that finished but whose parent hasn't called `wait()`. They can't be killed directly.

```bash
# Find zombie processes
ps aux | awk '$8=="Z"'
# or
ps -eo pid,ppid,stat,comm | grep Z

# Find the parent process
# Example: zombie PID=5678
cat /proc/5678/status | grep PPid
# Output: PPid: 1234

# Option 1: Restart the parent process
sudo kill -SIGCHLD 1234    # Ask parent to reap children
# If parent ignores SIGCHLD, restart it:
sudo systemctl restart parent-service

# Option 2: Kill the parent (zombies get reparented to init which reaps them)
sudo kill -9 1234
# All orphaned zombies are reaped by PID 1 (systemd/init)

# Option 3: Send SIGCHLD to parent
sudo kill -17 <PPID>

# Root cause: Fix application code to properly call wait()/waitpid()
# in the parent process after forking
```
{{< /qa >}}
{{< qa num="20" q="System clock on your EC2 instance is drifting. How do you fix it?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Check current time and sync status
timedatectl
chronyc tracking        # If chrony is installed
ntpq -p                 # If ntpd is used

# Step 2: Check if time sync service is running
sudo systemctl status chronyd
sudo systemctl status ntp

# Step 3: For Ubuntu 20.04+ with chrony (AWS recommended)
sudo apt install chrony -y
sudo systemctl enable --now chronyd

# Configure to use Amazon Time Sync Service (169.254.169.123)
sudo nano /etc/chrony/chrony.conf
# Add or replace pool lines:
# server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4

sudo systemctl restart chronyd
chronyc makestep    # Force immediate sync
chronyc tracking    # Verify

# Step 4: For systems using systemd-timesyncd
sudo timedatectl set-ntp true
sudo nano /etc/systemd/timesyncd.conf
# [Time]
# NTP=169.254.169.123
sudo systemctl restart systemd-timesyncd
timedatectl show-timesync

# Step 5: Verify after fix
date
timedatectl
```


{{< /qa >}}
{{< qa num="21" q="Your application is listening on a port but external requests time out. How do you debug?" level="basic" >}}

**Answer:**

```bash
# Step 1: Confirm application is listening
ss -tulnp | grep <port>
# Must show: LISTEN 0 ... 0.0.0.0:<port>
# NOT: 127.0.0.1:<port>  (only accepts local connections)

# Critical: Check bind address
# If app binds to 127.0.0.1, external requests will time out even if port 22 works
# Fix: Change app config to bind to 0.0.0.0 or specific private IP

# Step 2: Verify from inside the instance
curl http://localhost:<port>/health
curl http://<private-ip>:<port>/health

# Step 3: Check UFW (local firewall)
sudo ufw status verbose
sudo ufw allow <port>/tcp

# Step 4: Check iptables rules
sudo iptables -L INPUT -n -v | grep <port>
sudo iptables -L -n -v --line-numbers

# Step 5: Check AWS Security Group
aws ec2 describe-security-groups --group-ids sg-xxxxxxxx \
  --query 'SecurityGroups[].IpPermissions[?ToPort==`<port>`]'

# Step 6: Check Network ACL (stateless — must allow both inbound AND return traffic)
# Inbound: allow TCP port <port> from 0.0.0.0/0
# Outbound: allow TCP 1024-65535 to 0.0.0.0/0 (ephemeral ports)
```

{{< /qa >}}
{{< qa num="22" q="EC2 instance loses network connectivity after a few hours. How do you diagnose?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Check via CloudWatch before loss recurs
# Set up a ping/connectivity alarm

# Step 2: After connectivity is lost, check via EC2 Serial Console or SSM (if still running)
# Check if network interface is still up
ip link show eth0
ip addr show eth0

# Step 3: Check for DHCP lease expiry
sudo journalctl -u systemd-networkd | grep -i dhcp
sudo cat /var/lib/dhcp/dhclient.leases

# Step 4: Check for interface flapping
sudo dmesg | grep -i "eth0\|link\|carrier" | tail -30

# Step 5: Check if it's memory-related causing kernel to drop network
sudo dmesg | grep -i "oom\|memory" | tail -20

# Step 6: Check Enhanced Networking driver
ethtool -i eth0
# Should show: driver: ena (Elastic Network Adapter)
# If not: aws ec2 modify-instance-attribute --instance-id i-xxx --ena-support

# Step 7: Check for stuck processes holding socket connections
sudo ss -s
sudo ss -tulnp | wc -l

# Fix: Ensure DHCP renewal works
sudo dhclient -v eth0
```

{{< /qa >}}
{{< qa num="23" q="DNS resolution is failing on your EC2 instance. How do you troubleshoot?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Test DNS resolution
nslookup google.com
dig google.com
host google.com

# Step 2: Check current DNS servers
cat /etc/resolv.conf
# Should show: nameserver 169.254.169.253 (AWS VPC DNS)

# Step 3: Test directly against AWS DNS
dig @169.254.169.253 google.com

# Step 4: Check if systemd-resolved is interfering
sudo systemctl status systemd-resolved
resolvectl status

# Step 5: Check if enableDnsSupport is enabled on VPC
aws ec2 describe-vpc-attribute \
  --vpc-id vpc-xxxxxxxx \
  --attribute enableDnsSupport

aws ec2 describe-vpc-attribute \
  --vpc-id vpc-xxxxxxxx \
  --attribute enableDnsHostnames

# Fix if disabled:
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-xxxxxxxx \
  --enable-dns-support '{"Value": true}'

# Step 6: Fix /etc/resolv.conf if wrong
# For Ubuntu with systemd-resolved
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

# Manual fix (temporary)
echo "nameserver 169.254.169.253" | sudo tee /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
```

{{< /qa >}}
{{< qa num="24" q="You cannot reach the internet from a private subnet EC2 instance. What do you check?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Confirm the instance is in a private subnet
# (no public IP, no direct internet gateway route)
aws ec2 describe-instances --instance-ids i-xxxxxxxx \
  --query 'Reservations[].Instances[].[SubnetId,PublicIpAddress]'

# Step 2: Check route table for subnet
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxxxxx

# Private subnet should have: 0.0.0.0/0 → nat-xxxxxxxx (NAT Gateway)
# NOT: 0.0.0.0/0 → igw-xxxxxxxx (that's public subnet)

# Step 3: Check NAT Gateway exists and is available
aws ec2 describe-nat-gateways \
  --filter Name=state,Values=available

# Step 4: Check NAT Gateway is in a PUBLIC subnet
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxxxxx \
  --query 'NatGateways[].SubnetId'

# Step 5: Check NAT Gateway has an Elastic IP
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxxxxx \
  --query 'NatGateways[].NatGatewayAddresses'

# Step 6: Check Security Group allows outbound
aws ec2 describe-security-groups --group-ids sg-xxxxxxxx \
  --query 'SecurityGroups[].IpPermissionsEgress'

# Step 7: Test from instance (via SSM)
curl https://checkip.amazonaws.com   # Should return NAT Gateway's EIP
ping 8.8.8.8
```

{{< /qa >}}
{{< qa num="25" q="Network throughput on your EC2 instance is much lower than expected. What could cause this?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Measure actual throughput
iperf3 -s                         # On receiver
iperf3 -c <server-ip> -t 30       # On sender

# Step 2: Check Enhanced Networking is enabled
ethtool -i eth0
# driver should be 'ena' or 'ixgbevf'

# Enable ENA if not present
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxx \
  --ena-support

# Step 3: Check network bandwidth limits per instance type
# t3.micro: Up to 5 Gbps
# c5.xlarge: Up to 10 Gbps
# These are SHARED network limits

# Step 4: Check if instance is CPU-throttled (T-series burst)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUSurplusCreditBalance \
  --dimensions Name=InstanceId,Value=i-xxxxxxxx \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 --statistics Average

# Step 5: Check for network errors
ip -s link show eth0
# Look for: errors, dropped, overruns

# Step 6: Check placement group (if high throughput needed between instances)
aws ec2 describe-instances --instance-ids i-xxxxxxxx \
  --query 'Reservations[].Instances[].Placement'
# Move to cluster placement group for low-latency, high-bandwidth
```

{{< /qa >}}
{{< qa num="25" q="Network throughput on your EC2 instance is much lower than expected. What could cause this?" level="advanced" >}}

**Answer:**

"No route to host" differs from "Connection refused" — it means the TCP packet couldn't reach the destination.

```bash
# Step 1: Check basic reachability
ping <target-private-ip>
traceroute <target-private-ip>

# Step 2: Check both instances are in same VPC
aws ec2 describe-instances \
  --instance-ids i-source i-target \
  --query 'Reservations[].Instances[].[InstanceId,VpcId,SubnetId]'

# Step 3: Check route table on source instance's subnet
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-source
# Must have local route: 10.0.0.0/16 → local

# Step 4: Check Security Group on TARGET instance
aws ec2 describe-security-groups --group-ids sg-target
# Must allow inbound from source IP or source SG

# Step 5: Check if target has a local OS firewall blocking
sudo ufw status          # On target instance via SSM
sudo iptables -L -n -v | grep <port>

# Step 6: Check NACL (stateless — needs both inbound AND outbound rules)
# Inbound rule: allow from source subnet
# Outbound rule: allow ephemeral ports 1024-65535 back to source

# Fix SG on target to allow source:
aws ec2 authorize-security-group-ingress \
  --group-id sg-target \
  --protocol tcp --port 8080 \
  --source-group sg-source
```

{{< /qa >}}
{{< qa num="27" q="Intermittent packet loss is occurring on your EC2 instance. How do you investigate?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Quantify the loss
ping -c 1000 -i 0.2 <target-ip> | tail -5
# Look at: X% packet loss

# Step 2: Check interface errors
watch -n 1 'ip -s link show eth0'
# Incrementing: errors, dropped = driver/hardware issue

# Step 3: Check for ENA driver issues
sudo dmesg | grep -i "ena\|timeout\|reset\|hang" | tail -30

# Step 4: Check for CPU steal (EC2 hypervisor contention)
top
# %st (steal time) > 5% = noisy neighbor issue
# Fix: Change instance type or AZ

# Step 5: Check MTU issues causing fragmentation drops
sudo ip link show eth0 | grep mtu
# AWS default: 9001 (jumbo frames on same VPC)
# Internet: should be 1500
ping -M do -s 8972 <target-in-same-vpc>   # Tests jumbo frame path

# Fix MTU mismatch:
sudo ip link set dev eth0 mtu 1500

# Step 6: Check for security group rate limiting
# AWS SG has implicit rate limits on tracked connections
# Large numbers of short-lived connections can cause drops
# Fix: Use connection pooling, increase keep-alive timeouts

# Step 7: Use EC2 Network Performance metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name NetworkPacketsOut \
  --dimensions Name=InstanceId,Value=i-xxxxxxxx \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 60 --statistics Sum
```

{{< /qa >}}
{{< qa num="28" q="A user gets 'Permission denied' running a script even though it is executable. Why?" level="basic" >}}

**Answer:**

Multiple possible causes beyond simple execute permission:

```bash
# Check permissions
ls -la /path/to/script.sh
# Needs: ---x------ or more permissive

# Cause 1: Script mounted on noexec filesystem
mount | grep noexec
# If /tmp is noexec, scripts there can't be run directly
# Fix: Move script to /usr/local/bin or similar

# Cause 2: Wrong shebang line
head -1 /path/to/script.sh
# Must be: #!/bin/bash  (correct)
# NOT:    #!/bin/bash  (with Windows \r\n line endings!)

# Detect Windows line endings
file /path/to/script.sh
# Output: "with CRLF line terminators" = problem

# Fix Windows line endings
sudo sed -i 's/\r//' /path/to/script.sh
# or
dos2unix /path/to/script.sh

# Cause 3: SELinux/AppArmor context
ls -Z /path/to/script.sh   # Check SELinux context
sudo aa-status             # Check AppArmor

# Cause 4: Script interpreter doesn't exist
head -1 script.sh          # e.g., #!/usr/bin/python3
which python3              # Check if interpreter exists

# Cause 5: ACL overriding standard permissions
getfacl /path/to/script.sh
```

{{< /qa >}}
{{< qa num="29" q="sudo: command not found for a user who should have sudo access. How do you fix it?" level="intermediate" >}}

**Answer:**

```bash
# Cause 1: sudo is not installed
which sudo
apt list --installed | grep sudo
sudo apt install sudo -y

# Cause 2: User not in sudo group
groups username
id username
# Fix:
sudo usermod -aG sudo username
# Must log out and back in for group change to take effect
# Verify:
su - username
sudo whoami

# Cause 3: /etc/sudoers is misconfigured
sudo visudo
# Ensure this line exists:
# %sudo   ALL=(ALL:ALL) ALL

# Cause 4: PATH doesn't include /usr/bin where sudo lives
echo $PATH
# Fix for current session:
export PATH=$PATH:/usr/bin:/usr/sbin

# Cause 5: sudo binary has wrong permissions (rare)
ls -la $(which sudo)
# Must have: -rwsr-xr-x (setuid bit)
sudo chmod 4755 /usr/bin/sudo  # Fix if wrong

# Cause 6: Session started before group change — new group not applied
# User must re-login or run:
newgrp sudo
```

{{< /qa >}}
{{< qa num="29" q="sudo: command not found for a user who should have sudo access. How do you fix it?" level="intermediate" >}}

**Answer:**

```bash
# IMMEDIATE: Isolate the instance FIRST (preserve evidence)
# Move to an isolation Security Group with NO inbound/outbound rules
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxx \
  --groups sg-isolation

# Step 1: Capture volatile evidence (do this BEFORE taking a snapshot)
# Via SSM:
# Running processes
ps auxf > /tmp/forensics_ps.txt

# Network connections
ss -tulnp > /tmp/forensics_netstat.txt

# Logged in users
w > /tmp/forensics_users.txt
last > /tmp/forensics_last.txt

# Active cron jobs
crontab -l > /tmp/forensics_cron.txt
ls /etc/cron.* >> /tmp/forensics_cron.txt

# Recently modified files
find / -mtime -7 -type f 2>/dev/null > /tmp/forensics_recent_files.txt

# Copy forensics data to S3
aws s3 cp /tmp/forensics_*.txt s3://security-forensics-bucket/

# Step 2: Take a snapshot for forensic analysis
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxx \
  --description "FORENSICS-$(date +%Y%m%d-%H%M%S)"

# Step 3: Check for indicators of compromise
sudo last                              # Recent logins
sudo cat /var/log/auth.log | grep "Accepted\|Failed"
sudo crontab -l -u root                # Root cron jobs
sudo find / -name "authorized_keys" 2>/dev/null   # All auth key files
sudo find /tmp /var/tmp -type f -executable        # Executables in temp dirs
sudo netstat -tulnp | grep LISTEN      # Unexpected listeners
sudo rkhunter --check                  # Rootkit check

# Step 4: Notify security team, launch a clean replacement instance
# DO NOT try to "clean" a compromised instance — rebuild from a known-good AMI
```

{{< /qa >}}
{{< qa num="31" q="fail2ban is blocking legitimate users. How do you unban an IP and tune the rules?" level="intermediate" >}}

**Answer:**

```bash
# Check fail2ban status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# List currently banned IPs
sudo fail2ban-client status sshd | grep "Banned IP"

# Unban a specific IP
sudo fail2ban-client set sshd unbanip 203.0.113.50

# Whitelist (ignore) an IP permanently
sudo nano /etc/fail2ban/jail.local
```
```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 203.0.113.50/32 10.0.0.0/8
```
```bash
sudo systemctl restart fail2ban

# Tune ban rules (make less aggressive)
sudo nano /etc/fail2ban/jail.local
```
```ini
[sshd]
enabled  = true
maxretry = 10           # Allow 10 failures (default is 5)
findtime = 600          # Within 10 minutes
bantime  = 3600         # Ban for 1 hour (default 600s)
```
```bash
sudo systemctl restart fail2ban

# Check fail2ban logs
sudo tail -f /var/log/fail2ban.log

# Verify a specific IP is not banned
sudo fail2ban-client get sshd banip | grep 203.0.113.50
```

{{< /qa >}}
{{< qa num="32" q="AppArmor or SELinux is silently blocking your application. How do you diagnose it?" level="advanced" >}}

**Answer:**

Ubuntu uses AppArmor by default.

```bash
# Check AppArmor status
sudo aa-status
sudo apparmor_status

# Check if AppArmor is blocking anything
sudo dmesg | grep -i "apparmor\|denied"
sudo grep "DENIED" /var/log/kern.log
sudo grep "DENIED" /var/log/syslog

# Check audit log (if auditd installed)
sudo ausearch -m avc -ts recent

# Identify the profile blocking your app
sudo aa-status | grep -A5 enforce

# Temporarily put the profile in complain mode (log but don't block)
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx

# Run your application and watch what gets logged
sudo tail -f /var/log/syslog | grep apparmor

# Generate a new profile from audit logs
sudo aa-logprof

# Permanently allow a specific rule
sudo nano /etc/apparmor.d/usr.sbin.myapp
# Add the needed allow rules

# Reload profile
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.myapp

# Disable AppArmor for a specific profile (last resort)
sudo aa-disable /etc/apparmor.d/usr.sbin.myapp
```

{{< /qa >}}
{{< qa num="33" q="apt is broken with 'dpkg was interrupted'. How do you fix it?" level="basic" >}}

**Answer:**

```bash
# Symptom:
# E: dpkg was interrupted, you must manually run
# 'sudo dpkg --configure -a' to correct the problem.

# Step 1: Run the suggested fix
sudo dpkg --configure -a

# If that fails with more errors:

# Step 2: Force fix broken installs
sudo apt --fix-broken install
sudo apt --fix-missing install

# Step 3: Clear locks if apt/dpkg is stuck
sudo rm -f /var/lib/dpkg/lock
sudo rm -f /var/lib/dpkg/lock-frontend
sudo rm -f /var/cache/apt/archives/lock
sudo rm -f /var/lib/apt/lists/lock

# Step 4: Reconfigure all partially installed packages
sudo dpkg --configure -a --force-confdef

# Step 5: If a specific package is broken
sudo dpkg --remove --force-remove-reinstreq <broken-package>
sudo apt install <broken-package>

# Step 6: Nuclear option — clear and rebuild package lists
sudo rm -rf /var/lib/apt/lists/*
sudo apt clean
sudo apt update
sudo apt upgrade
```

{{< /qa >}}
{{< qa num="34" q="A package upgrade broke a running service. How do you roll it back?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Identify which version broke the service
apt list --installed | grep <package>
sudo dpkg --list | grep <package>

# Step 2: Check apt history
cat /var/log/apt/history.log | grep -A10 "$(date +%Y-%m-%d)"

# Step 3: Find available older versions
apt-cache showpkg <package>
apt-cache madison <package>

# Step 4: Downgrade to previous version
sudo apt install <package>=<old-version>

# Example: Downgrade nginx from 1.24 to 1.22
sudo apt install nginx=1.22.0-1ubuntu3

# Step 5: Pin the version to prevent auto-upgrade
sudo apt-mark hold <package>
apt-mark showhold

# Step 6: If version is no longer in repo, use dpkg cache
ls /var/cache/apt/archives/ | grep <package>
sudo dpkg -i /var/cache/apt/archives/<package>_<old-version>.deb

# Step 7: Check if a config file conflict caused the break
# During upgrades, dpkg may install .dpkg-new files
ls /etc/<service>/*.dpkg-new
ls /etc/<service>/*.dpkg-old
# Compare and restore if needed
diff /etc/nginx/nginx.conf /etc/nginx/nginx.conf.dpkg-old
```

{{< /qa >}}
{{< qa num="35" q="You get 'Package has no installation candidate' for a package you know exists. Why?" level="intermediate" >}}

**Answer:**

```bash
# Cause 1: Package is in a universe/multiverse repo that isn't enabled
sudo add-apt-repository universe
sudo add-apt-repository multiverse
sudo apt update
sudo apt install <package>

# Cause 2: Wrong Ubuntu version — package name differs
apt-cache search <keyword>

# Cause 3: Repository list is outdated
sudo apt update

# Cause 4: Third-party repo not added
# Example: Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update

# Cause 5: Architecture mismatch
dpkg --print-architecture
apt-cache policy <package>
# Check if package is available for your architecture

# Cause 6: Repo key expired
sudo apt-key list          # Check for expired keys
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys <KEYID>
```

{{< /qa >}}
{{< qa num="36" q="Shared library error: 'error while loading shared libraries'. How do you fix it?" level="advanced" >}}

**Answer:**

```bash
# Full error example:
# ./myapp: error while loading shared libraries: libssl.so.1.1: cannot open shared object file

# Step 1: Find which libraries are missing
ldd /path/to/binary
# Lines showing "not found" are the problem

# Step 2: Search for the library on the system
find / -name "libssl.so*" 2>/dev/null
ldconfig -p | grep libssl

# Step 3: Install the missing library package
apt-cache search libssl
sudo apt install libssl1.1

# Step 4: If library exists but is not found
# Update the linker cache
sudo ldconfig

# Step 5: If library is in a non-standard path
echo "/opt/myapp/lib" | sudo tee /etc/ld.so.conf.d/myapp.conf
sudo ldconfig

# Or set LD_LIBRARY_PATH temporarily
export LD_LIBRARY_PATH=/opt/myapp/lib:$LD_LIBRARY_PATH
./myapp

# Step 6: If wrong version (e.g., need libssl.so.1.1 but have libssl.so.3)
# Create a symlink (use carefully)
sudo ln -s /usr/lib/x86_64-linux-gnu/libssl.so.3 \
           /usr/lib/x86_64-linux-gnu/libssl.so.1.1
sudo ldconfig
```

{{< /qa >}}
{{< qa num="37" q="A systemd service fails to start with 'Failed to execute'. How do you debug it?" level="intermediate" >}}

**Answer:**

```bash
# Get full error details
sudo systemctl status myapp.service
sudo journalctl -u myapp.service -n 50 --no-pager

# Common error: "Failed to execute /opt/myapp/start.sh: Permission denied"

# Check 1: File exists and is executable
ls -la /opt/myapp/start.sh
# Fix:
chmod +x /opt/myapp/start.sh

# Check 2: Shebang is correct
head -1 /opt/myapp/start.sh
# Must be: #!/bin/bash  (not Windows line endings)

# Check 3: SELinux/AppArmor context
ls -Z /opt/myapp/start.sh

# Check 4: Service file ExecStart path
sudo systemctl cat myapp.service | grep ExecStart
# Must be absolute path, no shell expansion
# WRONG:  ExecStart=~/myapp/start.sh
# RIGHT:  ExecStart=/opt/myapp/start.sh

# Check 5: User the service runs as exists
grep User /etc/systemd/system/myapp.service
id appuser   # Must exist

# Check 6: Working directory exists
grep WorkingDirectory /etc/systemd/system/myapp.service
ls -la /opt/myapp   # Must exist and be accessible by the service user

# After fixing:
sudo systemctl daemon-reload
sudo systemctl start myapp.service
sudo systemctl status myapp.service
```

{{< /qa >}}
{{< qa num="38" q="A service starts successfully but exits immediately. How do you diagnose it?" level="intermediate" >}}

**Answer:**

```bash
# See start and exit in status
sudo systemctl status myapp.service
# Shows: Active: failed (Result: exit-code)

# Get detailed logs
sudo journalctl -u myapp.service -n 100 --no-pager

# Common causes:

# Cause 1: Service type mismatch
# If app forks/daemonizes, Type=simple will think it exited
sudo systemctl cat myapp.service | grep Type
# Options: simple, forking, oneshot, notify, idle
# For apps that daemonize: use Type=forking with PIDFile=

# Cause 2: Missing environment variable or config file
# Add to service file for debugging:
# [Service]
# StandardOutput=journal
# StandardError=journal
# Environment=DEBUG=true

# Cause 3: Port already in use
sudo ss -tulnp | grep <app-port>
# Kill the conflicting process

# Cause 4: Run as wrong user (missing file access)
sudo -u appuser /opt/myapp/start.sh
# Test if it runs as the service user

# Cause 5: Missing dependency (database not ready)
# Add to [Unit] section:
# After=mysql.service
# Requires=mysql.service

# Debug by running ExecStart manually
sudo -u appuser /opt/myapp/start.sh
# See the error directly
```
{{< /qa >}}
{{< qa num="39" q="systemd service is stuck in 'activating' state. What do you do?" level="advanced" >}}

**Answer:**

```bash
# Check status
sudo systemctl status myapp.service
# Active: activating (start) since ...

# Step 1: Check how long it's been in activating
# If it just started, wait; if minutes, it's hung

# Step 2: Check for WantedBy/After dependencies blocking it
sudo systemctl list-dependencies myapp.service
sudo systemctl status <dependency>   # Check if a dependency is failed

# Step 3: Check if ExecStartPre is hanging
sudo systemctl cat myapp.service | grep ExecStartPre
# A pre-start script may be stuck waiting

# Step 4: Check TimeoutStartSec
sudo systemctl cat myapp.service | grep TimeoutStart
# Default: 90s; if app takes longer to start, increase it
# [Service]
# TimeoutStartSec=300

# Step 5: Check if Type=notify but app never sends sd_notify
# Change to Type=simple if app doesn't use systemd notification

# Step 6: Force stop and investigate
sudo systemctl stop myapp.service
sudo systemctl status myapp.service
sudo journalctl -u myapp.service --since "10 minutes ago"

# Step 7: Kill hung ExecStartPre
sudo systemctl kill myapp.service
sudo journalctl -u myapp.service -n 30
```

{{< /qa >}}
{{< qa num="40" q="A service is not starting on boot even though it is enabled. How do you fix it?" level="intermediate" >}}

**Answer:**

```bash
# Verify it's actually enabled
sudo systemctl is-enabled myapp.service
# Should return: enabled
# If: static, masked, or disabled → fix it
sudo systemctl enable myapp.service

# Check if it's masked (masked takes priority over enabled)
sudo systemctl is-active myapp.service
sudo systemctl status myapp.service
# "masked" means it's blocked from starting

# Unmask if needed
sudo systemctl unmask myapp.service
sudo systemctl enable myapp.service

# Check WantedBy target
sudo systemctl cat myapp.service | grep WantedBy
# Must be: WantedBy=multi-user.target
# Check the symlink exists
ls /etc/systemd/system/multi-user.target.wants/ | grep myapp

# If symlink missing, re-enable
sudo systemctl disable myapp.service
sudo systemctl enable myapp.service

# Check if service has a dependency that is failing at boot
sudo journalctl -b | grep "myapp\|Failed"
sudo journalctl -b -u myapp.service

# Check for race condition — service starting before its dependency
sudo systemctl cat myapp.service
# Add:
# [Unit]
# After=network-online.target
# Wants=network-online.target
sudo systemctl daemon-reload
```
{{< /qa >}}
{{< qa num="41" q="journalctl shows no logs for a service. How do you troubleshoot missing logs?" level="advanced" >}}

**Answer:**

```bash
# Check journald is running
sudo systemctl status systemd-journald

# Check if the service writes to syslog instead of journal
sudo cat /etc/systemd/system/myapp.service | grep -i "StandardOutput\|StandardError"
# If not set, defaults to journal
# If set to syslog, check /var/log/syslog

# Check journal disk usage and if it's full
journalctl --disk-usage
df -h /var/log

# Check if journald log is corrupted
sudo journalctl --verify
# Fix:
sudo rm -rf /var/log/journal/*
sudo systemctl restart systemd-journald

# Ensure journald is set to persistent storage
cat /etc/systemd/journald.conf | grep Storage
# Should be: Storage=persistent (or auto)
# Fix if set to volatile:
sudo nano /etc/systemd/journald.conf
# Set: Storage=persistent
sudo systemctl restart systemd-journald

# Check if service redirects output to file
sudo systemctl cat myapp.service | grep Redirect
# If app logs to its own file:
sudo tail -f /opt/myapp/logs/app.log

# Check syslog
sudo grep myapp /var/log/syslog | tail -20
```

{{< /qa >}}
{{< qa num="42" q="CloudWatch agent is not sending metrics to AWS. How do you debug it?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Check agent status
sudo systemctl status amazon-cloudwatch-agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a status

# Step 2: Check agent logs
sudo tail -100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

# Step 3: Validate config file
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -s \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json

# Step 4: Check IAM role has CloudWatch permissions
aws sts get-caller-identity    # Verify role is attached
aws cloudwatch list-metrics --namespace CWAgent   # Test API access
# Must have: cloudwatch:PutMetricData, logs:CreateLogStream, logs:PutLogEvents

# Step 5: Check network connectivity to CloudWatch endpoint
curl -I https://monitoring.us-east-1.amazonaws.com
# For private subnet, ensure VPC endpoint or NAT gateway exists

# Step 6: Check config is correct
sudo cat /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
# Look for: correct namespace, metrics, log_group_name

# Step 7: Restart and test
sudo systemctl restart amazon-cloudwatch-agent
# Then check CloudWatch console after 1-2 minutes
```

{{< /qa >}}
{{< qa num="43" q="Log rotation is not working and old logs are accumulating. How do you fix it?" level="intermediate" >}}

**Answer:**

```bash
# Check logrotate config
sudo cat /etc/logrotate.conf
cat /etc/logrotate.d/nginx   # App-specific config

# Test logrotate manually (dry run)
sudo logrotate -d /etc/logrotate.d/nginx
# -d flag shows what it would do without doing it

# Force rotation now
sudo logrotate -f /etc/logrotate.d/nginx

# Check logrotate status (last rotation times)
sudo cat /var/lib/logrotate/status | grep nginx

# Check cron is running logrotate daily
ls -la /etc/cron.daily/logrotate
cat /etc/cron.daily/logrotate

sudo systemctl status cron

# Common misconfiguration: wrong log path
ls /var/log/nginx/*.log   # Confirm logs actually exist at this path

# Common issue: postrotate script failing
sudo logrotate -d -v /etc/logrotate.d/nginx 2>&1 | grep -i error

# Fix example for nginx:
sudo nano /etc/logrotate.d/nginx
```
```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 $(cat /var/run/nginx.pid)
        fi
    endscript
}
```

{{< /qa >}}
{{< qa num="44" q="EC2 instance status check fails with 'System reachability check failed'. What do you do?" level="advanced" >}}

**Answer:**

**System reachability check** is performed by AWS infrastructure (not the OS). It means the underlying hardware or hypervisor cannot reach the instance.

```bash
# Step 1: Get console output to see last known state
aws ec2 get-console-output --instance-ids i-xxxxxxxx --latest \
  --query Output --output text

# Step 2: Try stopping and starting (NOT reboot)
# Stop/Start moves the instance to new underlying hardware
aws ec2 stop-instances --instance-ids i-xxxxxxxx
aws ec2 wait instance-stopped --instance-ids i-xxxxxxxx
aws ec2 start-instances --instance-ids i-xxxxxxxx

# Note: STOP + START migrates to new host (fixes hardware issues)
# REBOOT stays on same host (does NOT fix hardware issues)

# Step 3: If stop fails (instance stuck)
aws ec2 stop-instances --instance-ids i-xxxxxxxx --force

# Step 4: Check AWS Health Dashboard for host-level events
aws health describe-events \
  --filter eventTypeCategories=issue \
  --region us-east-1

# Step 5: Check if it's a scheduled maintenance event
aws ec2 describe-instance-status --instance-ids i-xxxxxxxx \
  --query 'InstanceStatuses[].Events'

# Step 6: If instance won't recover, migrate data
# Take snapshot of EBS volume and launch new instance from it
aws ec2 create-snapshot --volume-id vol-xxxxxxxx \
  --description "Migrating from failed host"
```

{{< /qa >}}
{{< qa num="45" q="User data script did not run on instance launch. How do you debug it?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Check cloud-init status
sudo cloud-init status --long

# Step 2: View cloud-init output log (most useful)
sudo cat /var/log/cloud-init-output.log

# Step 3: View detailed cloud-init log
sudo cat /var/log/cloud-init.log | grep -i "error\|warn\|fail"

# Step 4: Confirm user data was received
aws ec2 describe-instance-attribute \
  --instance-id i-xxxxxxxx \
  --attribute userData \
  --query 'UserData.Value' --output text | base64 --decode

# Step 5: Common failure causes

# Cause 1: Missing shebang
# WRONG:
# apt update && apt install nginx -y

# RIGHT:
# #!/bin/bash
# apt update && apt install nginx -y

# Cause 2: User data runs as root but PATH is minimal
# Always use full paths:
# /usr/bin/apt update   NOT   apt update

# Cause 3: Script only runs once (cloud-init default)
# Re-run user data on existing instance:
sudo cloud-init clean --logs --seed
sudo cloud-init init --local
sudo cloud-init init
sudo cloud-init modules --mode=config
sudo cloud-init modules --mode=final

# Cause 4: User data too large (>16KB limit)
# Move script to S3 and download in user data:
# #!/bin/bash
# aws s3 cp s3://my-bucket/setup.sh /tmp/setup.sh
# bash /tmp/setup.sh

# Step 6: Check if script ran but failed
sudo grep -i "error\|failed\|exit" /var/log/cloud-init-output.log
```

{{< /qa >}}
{{< qa num="46" q="EC2 instance metadata service (IMDS) is not reachable from inside the instance. How do you fix it?" level="advanced" >}}

**Answer:**

```bash
# Test IMDS connectivity
curl -s http://169.254.169.254/latest/meta-data/

# For IMDSv2 (required if HttpTokens=required):
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id

# Cause 1: IMDS is disabled on the instance
aws ec2 describe-instance-metadata-options --instance-id i-xxxxxxxx
# Look for: HttpEndpoint = disabled

# Fix: Enable IMDS
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxxxxxx \
  --http-endpoint enabled \
  --http-tokens optional    # or 'required' for IMDSv2 only

# Cause 2: Hop limit too low (common in containers on EC2)
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxxxxxx \
  --http-put-response-hop-limit 2   # Increase for containers

# Cause 3: Local firewall blocking 169.254.169.254
sudo iptables -L OUTPUT -n -v | grep 169.254
# Fix:
sudo iptables -I OUTPUT -d 169.254.169.254 -j ACCEPT

# Cause 4: Route to link-local address missing
ip route show | grep 169.254
# Fix:
sudo ip route add 169.254.169.254 dev eth0

# Cause 5: Docker/container network overriding route
# In containers, use --network host or adjust hop limit above
```

{{< /qa >}}
{{< qa num="47" q="Instance is running but the application is not accessible after an AMI launch. What do you check?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: SSH into the new instance
ssh -i key.pem ubuntu@<new-instance-ip>

# Step 2: Check if the service is running
sudo systemctl status myapp.service

# Step 3: Check startup logs
sudo journalctl -u myapp.service -n 50

# Step 4: Common AMI baking issues

# Issue A: Hard-coded IP addresses in config files
grep -r "<old-private-ip>" /opt/myapp/config/
# Fix: Use instance metadata or DNS names instead of IPs

# Issue B: SSH host keys not regenerated (causes StrictHostKeyChecking failures)
sudo ls /etc/ssh/ssh_host_*
# Should exist — if missing, regenerate:
sudo dpkg-reconfigure openssh-server

# Issue C: Network interfaces have different names or IPs
ip addr
# Config may reference old ENI or IP

# Issue D: Service not enabled to auto-start
sudo systemctl is-enabled myapp.service
sudo systemctl enable myapp.service

# Issue E: Data volume not mounted (if app data is on separate EBS)
df -h
# Check /etc/fstab has nofail option:
# UUID=xxx  /data  ext4  defaults,nofail  0  2

# Issue F: Cloud-init ran and modified configs
sudo cat /var/log/cloud-init-output.log

# Step 5: Check Security Group attached to new instance
aws ec2 describe-instances --instance-ids i-new \
  --query 'Reservations[].Instances[].SecurityGroups'
```

{{< /qa >}}
{{< qa num="48" q="EC2 instance is experiencing high I/O wait. How do you identify and fix it?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Confirm high I/O wait
top
# CPU line: %wa = I/O wait percentage
# > 20% is concerning, > 50% is critical

vmstat 1 10
# Look for: high 'wa' column, high 'bi' (blocks in) or 'bo' (blocks out)

# Step 2: Identify the offending process
sudo iotop -o -P -d 1
# -o: only show active I/O
# -P: show processes not threads

sudo pidstat -d 1
# Shows per-process disk reads/writes

# Step 3: Identify the offending disk
iostat -xz 1
# Look for: high %util, high await (latency), high r/s or w/s

# Step 4: Check if EBS credits are exhausted (gp2 volumes)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EBS --metric-name BurstBalance \
  --dimensions Name=VolumeId,Value=vol-xxxxxxxx \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 --statistics Average
# BurstBalance near 0 = IOPS being throttled

# Step 5: Solutions

# Solution A: Upgrade gp2 to gp3 (cheaper, better baseline IOPS)
aws ec2 modify-volume \
  --volume-id vol-xxxxxxxx \
  --volume-type gp3 \
  --iops 3000 \
  --throughput 125

# Solution B: Optimize the application (add caching, reduce writes)

# Solution C: Use instance store (NVMe) for high-IOPS temp data

# Solution D: Add I/O scheduler tuning
cat /sys/block/xvda/queue/scheduler
echo mq-deadline | sudo tee /sys/block/xvda/queue/scheduler
```

{{< /qa >}}
{{< qa num="49" q="Docker container exits immediately after starting. How do you troubleshoot it?" level="basic" >}}

**Answer:**

```bash
# Step 1: Run without -d to see output directly
docker run --name test myimage:latest

# Step 2: If using -d, check logs immediately
docker run -d --name test myimage:latest
docker logs test
docker logs --tail 50 test

# Step 3: Get exit code
docker inspect test --format='{{.State.ExitCode}}'
# 0 = clean exit (PID 1 finished normally)
# 1 = error
# 137 = OOM killed (128+9)
# 139 = segfault (128+11)

# Step 4: Common causes

# Cause A: CMD is not a long-running process
# WRONG Dockerfile: CMD ["echo", "hello"]  — exits after printing
# RIGHT:            CMD ["nginx", "-g", "daemon off;"]  — stays in foreground

# Cause B: App crashes due to missing env var
docker run -e DB_HOST=localhost myimage:latest
# Check which env vars are needed:
docker inspect myimage:latest | grep Env

# Cause C: Missing files/volumes
docker run -v /host/path:/container/path myimage:latest

# Cause D: Health check killing container
docker inspect test | grep -A10 Health

# Step 5: Run with shell to debug
docker run -it --entrypoint /bin/sh myimage:latest
# Then manually run the startup command inside
```

{{< /qa >}}
{{< qa num="50" q="Container cannot reach the internet but the host EC2 instance can. How do you fix it?" level="advanced" >}}

**Answer:**

```bash
# Test from inside container
docker exec -it mycontainer sh
curl https://google.com
ping 8.8.8.8

# Step 1: Check Docker network settings
docker network ls
docker network inspect bridge

# Step 2: Check IP forwarding is enabled on host
sudo sysctl net.ipv4.ip_forward
# Must be: net.ipv4.ip_forward = 1

# Fix:
sudo sysctl -w net.ipv4.ip_forward=1
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf

# Step 3: Check iptables FORWARD chain
sudo iptables -L FORWARD -n -v
# Docker adds rules here; if policy is DROP, containers can't route

# Fix:
sudo iptables -P FORWARD ACCEPT
# Or restart Docker (it re-adds its rules):
sudo systemctl restart docker

# Step 4: Check Docker daemon DNS config
cat /etc/docker/daemon.json
# Add DNS if missing:
echo '{"dns": ["8.8.8.8", "169.254.169.253"]}' | \
  sudo tee /etc/docker/daemon.json
sudo systemctl restart docker

# Step 5: Check for conflicting iptables rules
sudo iptables -t nat -L -n -v | grep MASQUERADE
# Docker adds masquerade rule; if missing, re-add:
sudo iptables -t nat -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```

{{< /qa >}}
{{< qa num="51" q="Docker daemon fails to start after a reboot. How do you recover?" level="advanced" >}}

**Answer:**

```bash
# Check service status and error
sudo systemctl status docker
sudo journalctl -u docker -n 100 --no-pager

# Cause 1: Corrupted Docker state
sudo systemctl stop docker
sudo rm -f /var/lib/docker/network/files/local-kv.db
sudo systemctl start docker

# Cause 2: Corrupted PID file
sudo rm -f /var/run/docker.pid
sudo systemctl start docker

# Cause 3: Storage driver conflict
sudo cat /etc/docker/daemon.json
# Check storage-driver value
# Ubuntu default: overlay2
# If wrong:
echo '{"storage-driver": "overlay2"}' | sudo tee /etc/docker/daemon.json
sudo systemctl start docker

# Cause 4: /var/lib/docker on separate volume not mounted yet
df -h | grep docker
# Check /etc/fstab has the volume mount with nofail
# And docker.service has After=local-fs.target

# Cause 5: Kernel modules missing
sudo modprobe overlay
sudo modprobe br_netfilter
sudo systemctl start docker

# Cause 6: docker.sock permissions
ls -la /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock

# Nuclear recovery (WARNING: loses containers/images)
sudo systemctl stop docker
sudo rm -rf /var/lib/docker
sudo systemctl start docker
```

{{< /qa >}}
{{< qa num="52" q="'No space left on device' error in Docker even though the host has free disk space. Why?" level="advanced" >}}

**Answer:**

Docker has its own overlay filesystem and inodes that can be exhausted separately.

```bash
# Step 1: Check Docker disk usage
docker system df
docker system df -v   # Verbose — shows individual image/container sizes

# Step 2: Check if inodes are exhausted (not blocks)
df -i /var/lib/docker
# If IUse% is 100%, inodes are exhausted even if blocks are free

# Step 3: Check which Docker partition is full
df -h /var/lib/docker

# Step 4: Clean up Docker resources
# Remove stopped containers
docker container prune -f

# Remove unused images
docker image prune -a -f

# Remove unused volumes
docker volume prune -f

# Remove unused networks
docker network prune -f

# All at once:
docker system prune -af --volumes

# Step 5: Check for runaway logs (container logs have no size limit by default)
docker inspect <container> | grep LogPath
sudo du -sh /var/lib/docker/containers/*/
# Truncate a specific container log:
sudo truncate -s 0 /var/lib/docker/containers/<id>/<id>-json.log

# Step 6: Set log rotation limits in daemon.json
sudo nano /etc/docker/daemon.json
```
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```
```bash
sudo systemctl restart docker
```

{{< /qa >}}
{{< qa num="53" q="Nginx returns 502 Bad Gateway. How do you trace the root cause?" level="intermediate" >}}

**Answer:**

502 = Nginx received an invalid or no response from the upstream (your app).

```bash
# Step 1: Check Nginx error logs
sudo tail -50 /var/log/nginx/error.log
# Look for: "connect() failed", "upstream timed out", "no live upstreams"

# Step 2: Is the upstream app running?
sudo systemctl status myapp
sudo ss -tulnp | grep <app-port>

# Step 3: Can Nginx reach the upstream locally?
curl http://127.0.0.1:<app-port>/
# If this fails, the app itself is the problem

# Step 4: Check Nginx upstream config
sudo nginx -T | grep upstream -A5
sudo cat /etc/nginx/sites-enabled/myapp
# Verify proxy_pass points to correct host:port

# Step 5: Test with increased timeout (if app is slow)
sudo nano /etc/nginx/sites-available/myapp
# Add:
# proxy_connect_timeout 60s;
# proxy_read_timeout 60s;
# proxy_send_timeout 60s;
sudo nginx -t && sudo systemctl reload nginx

# Step 6: Check upstream app logs
sudo journalctl -u myapp -n 50

# Step 7: Check if it's a socket permission issue
# If using Unix socket:
ls -la /run/myapp.sock
# Nginx user (www-data) must have read/write
sudo chown www-data:www-data /run/myapp.sock

# Step 8: Check for too many connections
sudo ss -s | grep CLOSE_WAIT
# High CLOSE_WAIT = app not closing connections properly
```

{{< /qa >}}
{{< qa num="54" q="MySQL/PostgreSQL refuses connections with 'Too many connections'. How do you fix it?" level="advanced" >}}

**Answer:**

```bash
# MySQL:

# Check current connections
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"

# Temporarily increase max connections (no restart)
mysql -u root -p -e "SET GLOBAL max_connections = 500;"

# Make permanent
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
# Add/update: max_connections = 500
sudo systemctl restart mysql

# Find what's using connections
mysql -u root -p -e "SHOW PROCESSLIST;" | head -30
mysql -u root -p -e "SELECT user, host, COUNT(*) FROM information_schema.processlist GROUP BY user, host ORDER BY COUNT(*) DESC;"

# Kill idle connections
mysql -u root -p -e "SELECT CONCAT('KILL ',id,';') FROM information_schema.processlist WHERE Command='Sleep' AND Time>300;"

# PostgreSQL:

# Check connections
sudo -u postgres psql -c "SELECT count(*) FROM pg_stat_activity;"
sudo -u postgres psql -c "SHOW max_connections;"

# Increase limit
sudo nano /etc/postgresql/14/main/postgresql.conf
# max_connections = 200
sudo systemctl restart postgresql

# Kill idle connections
sudo -u postgres psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state='idle' AND state_change < now() - interval '10 minutes';"

# Long-term fix: Use connection pooling (PgBouncer / ProxySQL)
sudo apt install pgbouncer
```

{{< /qa >}}
{{< qa num="55" q="Application throws 'Address already in use' on startup. How do you resolve it?" level="basic" >}}

**Answer:**

```bash
# Error: "bind: address already in use" or "EADDRINUSE"

# Step 1: Find what is using the port
sudo ss -tulnp | grep :<port>
sudo lsof -i :<port>
sudo fuser <port>/tcp

# Step 2: Identify the process
sudo ss -tulnp | grep :8080
# Output: users:(("node",pid=1234,fd=22))

# Step 3: Options to resolve

# Option A: Kill the conflicting process
sudo kill -9 1234
# Verify port is free:
sudo ss -tulnp | grep :8080

# Option B: If it's the same service left running from a crash
sudo systemctl stop myapp
sudo systemctl start myapp

# Option C: If a previous instance left a ghost socket file
ls /run/myapp.sock
rm -f /run/myapp.sock
# Then restart

# Option D: Change your application to use a different port

# Step 4: Add SO_REUSEADDR to application config (for quick restarts)
# This is application-level: set SO_REUSEADDR=true in app config

# Step 5: Wait for TIME_WAIT to clear
# Connections in TIME_WAIT hold the port for ~60s after close
sudo ss -s | grep timewait
# If too many, tune:
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
echo 'net.ipv4.tcp_tw_reuse=1' | sudo tee -a /etc/sysctl.conf
```

{{< /qa >}}
{{< qa num="56" q="Your application writes files but another process cannot read them. What is wrong?" level="intermediate" >}}

**Answer:**

```bash
# Step 1: Check file permissions and ownership
ls -la /path/to/file

# Step 2: Check the running users
ps aux | grep <writer-process>
ps aux | grep <reader-process>
# They may run as different users

# Step 3: Check group membership
id writer-user
id reader-user

# Fix A: Change file ownership
sudo chown writer-user:shared-group /path/to/files
sudo chmod 664 /path/to/files   # Owner rw, Group rw, Others r

# Fix B: Create shared group
sudo groupadd appdata
sudo usermod -aG appdata writer-user
sudo usermod -aG appdata reader-user
sudo chown -R :appdata /path/to/files
sudo chmod -R g+rw /path/to/files

# Fix C: Set setgid bit so new files inherit group
sudo chmod g+s /path/to/directory
# All new files will belong to directory's group

# Step 4: Check umask (affects default permissions of new files)
# Process writing files may have restrictive umask
umask
# 0022 = new files are 644 (no group write)
# 0002 = new files are 664 (group can write)

# Set in service file:
# [Service]
# UMask=0002

# Step 5: Check ACLs
getfacl /path/to/file
# Grant explicit access:
setfacl -m u:reader-user:r /path/to/file
```

{{< /qa >}}
{{< qa num="57" q="EC2 instance is stuck in a reboot loop. How do you recover it?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Check console output to see what's happening
aws ec2 get-console-output --instance-ids i-xxxxxxxx --latest \
  --query Output --output text

# Look for:
# - Kernel panic messages
# - Failed to mount messages
# - OOM killer activity before reboot

# Step 2: Check for automated reboot scripts
# Did cloud-init or user data trigger a reboot?
# Is there a watchdog (e.g., microcode update loop)?

# Step 3: Stop the instance (if it keeps auto-starting, check Auto Scaling)
aws ec2 stop-instances --instance-ids i-xxxxxxxx

# Step 4: If in an Auto Scaling Group, put in Standby first
aws autoscaling enter-standby \
  --instance-ids i-xxxxxxxx \
  --auto-scaling-group-name my-asg \
  --should-decrement-desired-capacity

aws ec2 stop-instances --instance-ids i-xxxxxxxx

# Step 5: Attach root volume to rescue instance
aws ec2 detach-volume --volume-id vol-root --force
aws ec2 attach-volume --volume-id vol-root \
  --instance-id i-rescue --device /dev/xvdf

# Step 6: On rescue instance, diagnose
sudo mount /dev/xvdf1 /mnt/rescue
sudo chroot /mnt/rescue /bin/bash

# Check for reboot-triggering cron or script
crontab -l
cat /etc/rc.local
# Check for bad update or kernel module
dmesg
# Check fstab
cat /etc/fstab

# Step 7: Fix the issue, umount, re-attach, start
sudo umount /mnt/rescue
aws ec2 attach-volume --volume-id vol-root \
  --instance-id i-original --device /dev/xvda
aws ec2 start-instances --instance-ids i-original
```

{{< /qa >}}
{{< qa num="58" q="Kernel panic after a kernel upgrade. How do you boot into the old kernel?" level="advanced" >}}
**Answer:**

Since EC2 has no traditional GRUB console access:

```bash
# Step 1: View console output to confirm kernel panic
aws ec2 get-console-output --instance-ids i-xxxxxxxx --latest \
  --query Output --output text | grep -i "kernel\|panic"

# Step 2: Stop the instance
aws ec2 stop-instances --instance-ids i-xxxxxxxx

# Step 3: Attach root volume to rescue instance
# Mount on rescue instance
sudo mount /dev/xvdf1 /mnt/rescue

# Step 4: Check GRUB configuration
sudo cat /mnt/rescue/boot/grub/grub.cfg | grep menuentry
# Note the old kernel entry name

# Step 5: Set default kernel to old one
sudo chroot /mnt/rescue /bin/bash
grub-set-default "Advanced options for Ubuntu>Ubuntu, with Linux 5.15.0-88-generic"
# Or set by index:
grub-set-default "1>2"   # Submenu 1, entry 2
update-grub

# Verify
cat /boot/grub/grub.cfg | grep "set default"

# Exit chroot
exit
sudo umount /mnt/rescue

# Step 6: Re-attach and start
aws ec2 attach-volume --volume-id vol-root \
  --instance-id i-original --device /dev/xvda
aws ec2 start-instances --instance-ids i-original

# Step 7: After recovery, remove broken kernel
sudo apt remove linux-image-<broken-version>
sudo update-grub
```

{{< /qa >}}
{{< qa num="59" q="/etc/fstab has a bad entry and the instance will not boot. How do you fix it?" level="advanced" >}}

**Answer:**

```bash
# Step 1: Confirm via console output
aws ec2 get-console-output --instance-ids i-xxxxxxxx --latest \
  --query Output --output text
# Look for: "dependency failed", "Failed to mount /data"

# Step 2: Stop instance
aws ec2 stop-instances --instance-ids i-xxxxxxxx

# Step 3: Attach root volume to rescue instance as secondary disk
aws ec2 attach-volume --volume-id vol-root \
  --instance-id i-rescue --device /dev/xvdf

# Step 4: Mount and fix fstab
sudo mount /dev/xvdf1 /mnt/rescue
sudo nano /mnt/rescue/etc/fstab

# Fix: Either remove the bad line or add 'nofail' option
# WRONG:  UUID=bad-uuid  /data  ext4  defaults  0  2
# RIGHT:  UUID=bad-uuid  /data  ext4  defaults,nofail  0  2
# OR: Simply remove or comment out the bad entry

# Verify the fix before unmounting
sudo mount --bind /dev /mnt/rescue/dev
sudo mount --bind /proc /mnt/rescue/proc
sudo chroot /mnt/rescue findmnt --verify --verbose

sudo umount /mnt/rescue

# Step 5: Re-attach and start
aws ec2 detach-volume --volume-id vol-root
aws ec2 attach-volume --volume-id vol-root \
  --instance-id i-original --device /dev/xvda
aws ec2 start-instances --instance-ids i-original

# Prevention: Always use 'nofail' for non-root mounts
# UUID=xxx  /data  ext4  defaults,nofail  0  2
```

{{< /qa >}}
{{< qa num="60" q="Root filesystem is mounted as read-only after an unexpected reboot. How do you fix it?" level="advanced" >}}

**Answer:**

This happens when the kernel detects filesystem errors and remounts read-only to prevent further damage.

```bash
# Confirm read-only mount
mount | grep "on / "
# Output: /dev/xvda1 on / type ext4 (ro,relatime)

# Check dmesg for errors
sudo dmesg | grep -i "error\|remount\|read-only\|ext4" | tail -30

# Option 1: Remount read-write (if filesystem is not actually corrupted)
sudo mount -o remount,rw /

# Verify:
touch /tmp/test_write && echo "Writable" && rm /tmp/test_write

# Option 2: If filesystem is corrupted (errors in dmesg)
# Schedule fsck on next reboot

# Check filesystem error flag
sudo tune2fs -l /dev/xvda1 | grep "Filesystem state"
# "not clean" or errors present = needs fsck

sudo touch /forcefsck
sudo reboot
# fsck runs automatically on boot

# Option 3: fsck from rescue instance (for severe corruption)
# Stop instance, attach to rescue, unmount, run fsck:
sudo fsck -y /dev/xvdf1    # On rescue instance
# Reattach and start original

# Prevention:
# 1. Enable EBS-optimized instances
# 2. Monitor EBS metrics (VolumeWriteBytes, BurstBalance)
# 3. Set up CloudWatch alarms for disk errors
# 4. Regularly test filesystem health:
sudo tune2fs -l /dev/xvda1 | grep -E "state|errors|check"
```

{{< /qa >}}


## 📌 Quick Troubleshooting Reference

| Problem | First Command to Run |
|---------|---------------------|
| Can't SSH | `aws ec2 get-console-output --instance-ids i-xxx` |
| Disk full | `df -hT && sudo lsof \| grep deleted` |
| High CPU | `top` then `ps aux --sort=-%cpu \| head -10` |
| High load, low CPU | `iostat -xz 1` (check I/O wait) |
| Memory issues | `free -h && sudo dmesg \| grep -i oom` |
| Network timeout | `ss -tulnp \| grep <port>` |
| Service failing | `sudo journalctl -u <svc> -n 50` |
| DNS failing | `dig @169.254.169.253 google.com` |
| Package broken | `sudo dpkg --configure -a` |
| Boot failure | `aws ec2 get-console-output --latest` |
| Permission denied | `ls -la && id && getfacl <file>` |
| Container exits | `docker logs <container> && docker inspect <c>` |
| 502 Bad Gateway | `curl localhost:<app-port> && tail /var/log/nginx/error.log` |

---

*Last Updated: 2024 | Coverage: Ubuntu 20.04 / 22.04 on AWS EC2*
