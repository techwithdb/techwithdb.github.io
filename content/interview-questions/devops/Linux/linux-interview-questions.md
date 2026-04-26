---
title: "Linux Interview Questions & Answers (2026)"
description: "50+ Linux interview questions and answers covering processes, file system, permissions, services, and troubleshooting — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["linux", "interview", "sysadmin", "devops"]
tool: "linux"
level: "All Levels"
question_count: 52
draft: "false"
---

<div class="qa-list">

{{< qa num="1" q="Explain the booting process of Linux." level="basic" >}}
**Ans:**

1. **BIOS/UEFI** — Hardware runs Power-On Self Test (POST). BIOS/UEFI finds the bootable device.
2. **MBR/GPT** — Master Boot Record (first 512 bytes of disk) or GPT partition table is read. Bootloader location is found.
3. **GRUB2 (Bootloader)** — GRUB2 loads the Linux kernel (`vmlinuz`) and initial RAM disk (`initrd/initramfs`) into memory.
4. **Kernel Initialization** — Kernel decompresses itself, initializes hardware drivers, mounts the root filesystem.
5. **initramfs** — Temporary root filesystem used to load necessary drivers before the real root is mounted.
6. **systemd (PID 1)** — The first process started by the kernel. It reads unit files and brings up the system.
7. **Targets** — systemd reaches the configured target (e.g., `multi-user.target` or `graphical.target`), starting all required services.
8. **Login prompt** — Finally, a TTY login prompt or display manager appears.

```bash
# View boot messages
journalctl -b

# View systemd target
systemctl get-default
```
{{< /qa >}}

{{< qa num="2" q="What are services in Linux?" level="basic" >}}
**Ans:**

A **service** is a background process (daemon) managed by `systemd`. Services start automatically at boot and run without user interaction.

- **Unit files** define services and are located in `/etc/systemd/system/` or `/lib/systemd/system/`
- **Common commands:**

```bash
# Check status of a service
systemctl status nginx

# Start / Stop / Restart a service
systemctl start nginx
systemctl stop nginx
systemctl restart nginx

# Enable service at boot
systemctl enable nginx

# List all running services
systemctl list-units --type=service --state=running
```
{{< /qa >}}

{{< qa num="3" q="How do you check open ports in Linux?" level="basic" >}}
**Ans:**

```bash
# Using ss (recommended, modern)
ss -tulnp

# Using netstat (older systems)
netstat -tulnp

# Using lsof
lsof -i -P -n

# Check if a specific port is open
ss -tulnp | grep :80

# Using nmap (scan remotely)
nmap -p 80,443 <server-ip>
```

**Flag meanings for ss/netstat:**
- `-t` — TCP
- `-u` — UDP
- `-l` — Listening only
- `-n` — Numeric (no DNS resolution)
- `-p` — Show process name/PID
{{< /qa >}}

{{< qa num="4" q="Explain the Linux file system structure." level="basic" >}}
**Ans:**

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the entire filesystem |
| `/bin` | Essential user binaries (ls, cp, mv) |
| `/sbin` | System binaries (for root/admin) |
| `/etc` | Configuration files |
| `/home` | User home directories |
| `/var` | Variable data — logs, spools, temp files |
| `/tmp` | Temporary files (cleared on reboot) |
| `/usr` | User programs and libraries |
| `/opt` | Optional third-party software |
| `/proc` | Virtual filesystem — kernel/process info |
| `/dev` | Device files (disks, terminals) |
| `/mnt` / `/media` | Mount points for external/removable drives |
| `/boot` | Kernel and bootloader files |

```bash
# View tree structure
ls /

# Check filesystem type
df -Th
```
{{< /qa >}}

{{< qa num="5" q="What is the command to check OS version in Linux?" level="basic" >}}
**Ans:**

```bash
# Most universal
cat /etc/os-release

# For RHEL/CentOS
cat /etc/redhat-release

# For Debian/Ubuntu
cat /etc/debian_version

# Kernel version
uname -r

# All system info
uname -a

# Using lsb_release
lsb_release -a
```
{{< /qa >}}

{{< qa num="6" q="What is a process in Linux?" level="basic" >}}
**Ans:**

A **process** is a running instance of a program. Every process has:
- A unique **PID** (Process ID)
- A **PPID** (Parent Process ID)
- An owner (UID)
- A state (Running, Sleeping, Zombie, Stopped)

```bash
# List all processes
ps aux

# Tree view showing parent-child
pstree

# Dynamic real-time view
top
htop
```
{{< /qa >}}

{{< qa num="7" q="What is a daemon in Linux?" level="basic" >}}
**Ans:**

A **daemon** is a background process that runs continuously without user interaction. Daemons usually:
- Start at boot
- Have names ending in `d` (e.g., `sshd`, `httpd`, `crond`)
- Run as root or a dedicated service user

```bash
# List daemons managed by systemd
systemctl list-units --type=service

# Check if sshd is running
systemctl status sshd
```
{{< /qa >}}

{{< qa num="8" q="What are the different commands to check running processes?" level="basic" >}}
**Ans:**

```bash
# Static snapshot
ps aux            # All processes with details
ps -ef            # Full format listing
ps -u username    # Processes for a specific user

# Dynamic / real-time
top               # Interactive, updates every 3s
htop              # Colored, more user-friendly (install separately)

# Tree format
pstree            # Parent-child hierarchy

# Filter specific process
ps aux | grep nginx
pgrep nginx
```
{{< /qa >}}

{{< qa num="9" q="What is the meaning of 'ps'? Can we get dynamic output from the ps command?" level="basic" >}}
**Ans:**

`ps` stands for **Process Status**. It takes a snapshot of currently running processes at that moment — it is **not dynamic**.

For dynamic/real-time output, use:

```bash
# Real-time with auto-refresh every 2 seconds
watch -n 2 ps aux

# Or use top/htop for true dynamic view
top
htop
```
{{< /qa >}}

{{< qa num="10" q="What are the fields in top and ps commands?" level="basic" >}}
**Ans:**

**`top` fields:**
| Field | Meaning |
|-------|---------|
| PID | Process ID |
| USER | Owner of process |
| PR | Priority |
| NI | Nice value |
| VIRT | Virtual memory used |
| RES | Resident (physical) memory |
| SHR | Shared memory |
| S | State (R=running, S=sleeping, Z=zombie) |
| %CPU | CPU usage |
| %MEM | Memory usage |
| TIME+ | Total CPU time |
| COMMAND | Process name |

**`ps aux` fields:**
| Field | Meaning |
|-------|---------|
| USER | Process owner |
| PID | Process ID |
| %CPU | CPU usage |
| %MEM | Memory usage |
| VSZ | Virtual memory size |
| RSS | Resident memory size |
| TTY | Terminal associated |
| STAT | Process state |
| START | Start time |
| TIME | CPU time consumed |
| COMMAND | Command that started it |
{{< /qa >}}

{{< qa num="11" q="How to check load average in Linux?" level="basic" >}}
**Ans:**

Load average shows how many processes are waiting to run (averaged over 1, 5, and 15 minutes).

```bash
# Using uptime
uptime
# Output: 10:30  up 5 days,  load average: 0.45, 0.60, 0.55

# Using top (first line)
top

# From /proc
cat /proc/loadavg

# Using w command
w
```

**Interpretation:** If load average > number of CPU cores, system is overloaded.

```bash
# Check number of CPU cores
nproc
cat /proc/cpuinfo | grep processor | wc -l
```
{{< /qa >}}

{{< qa num="12" q="What is a zombie process? Can we create one?" level="intermediate" >}}
**Ans:**

A **zombie process** is a process that has finished execution but its entry still exists in the process table because the parent hasn't called `wait()` to read the exit status.

- Zombie processes consume **no CPU or memory** — just a PID slot
- They appear as state `Z` in `ps`

```bash
# Find zombie processes
ps aux | grep 'Z'
# or
ps -el | grep zombie
```

**Creating a zombie (C example concept):**
```c
// Child exits, parent sleeps without wait() → zombie created
if (fork() == 0) { exit(0); }  // child exits
else { sleep(60); }             // parent doesn't call wait()
```

**How to remove zombies:**
- Kill or restart the parent process — the zombie is then adopted by `init`/`systemd` which cleans it up
{{< /qa >}}

{{< qa num="13" q="What are the different states of a process?" level="basic" >}}
**Ans:**

| State | Symbol | Meaning |
|-------|--------|---------|
| Running | `R` | Actively using CPU or in run queue |
| Sleeping (interruptible) | `S` | Waiting for an event, can be interrupted |
| Sleeping (uninterruptible) | `D` | Waiting for I/O, cannot be interrupted |
| Stopped | `T` | Paused (Ctrl+Z or SIGSTOP) |
| Zombie | `Z` | Finished but not reaped by parent |
| Idle | `I` | Kernel idle thread |
{{< /qa >}}

{{< qa num="14" q="How to kill a foreground process?" level="basic" >}}
**Ans:**

```bash
# While process is running in terminal:
Ctrl + C    # Sends SIGINT — graceful interrupt

Ctrl + Z    # Sends SIGTSTP — suspends (pauses) it

# If you know the PID
kill -15 <PID>   # SIGTERM — graceful termination
kill -9 <PID>    # SIGKILL — force kill (cannot be caught)
```
{{< /qa >}}

{{< qa num="15" q="How to kill a background process?" level="basic" >}}
**Ans:**

```bash
# Find the PID
ps aux | grep process_name
pgrep process_name

# Kill by PID
kill <PID>          # SIGTERM (graceful)
kill -9 <PID>       # SIGKILL (force)

# Kill by name
pkill nginx
killall nginx

# If started with & and you know job number
jobs               # List background jobs
kill %1            # Kill job number 1
```
{{< /qa >}}

{{< qa num="16" q="How to bring a background process to the foreground?" level="basic" >}}
**Ans:**

```bash
# Step 1: See background jobs
jobs
# Output: [1]+  Stopped    ./script.sh

# Step 2: Bring to foreground
fg %1      # Bring job 1 to foreground

# Or resume a stopped process in background
bg %1      # Resume job 1 in background
```
{{< /qa >}}

{{< qa num="17" q="Where does the PID get stored in Linux?" level="intermediate" >}}
**Ans:**

- PIDs are tracked by the kernel in the **process table** (in kernel memory)
- For many services, the PID is written to a **PID file** on disk

```bash
# Common PID file locations
/var/run/nginx.pid
/var/run/sshd.pid
/run/<service>.pid

# Read a PID file
cat /var/run/nginx.pid

# From /proc filesystem (virtual, kernel-managed)
ls /proc/          # Each numbered directory = a PID
cat /proc/1234/status   # Details of PID 1234
```
{{< /qa >}}

{{< qa num="18" q="A service like httpd is running in the foreground. How do you shift it to the background?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: Suspend then background
Ctrl + Z          # Suspend the foreground process
bg %1             # Resume it in background

# Method 2: Start it with & from the beginning
httpd &

# Method 3: Use nohup to persist after terminal closes
nohup httpd &

# Method 4: Use disown to detach from shell
httpd &
disown %1

# Best practice: Use systemd
systemctl start httpd    # systemd manages it as a daemon
```
{{< /qa >}}

{{< qa num="19" q="What is the difference between terminating, killing, and stopping a process?" level="intermediate" >}}
**Ans:**

| Action | Signal | Meaning |
|--------|--------|---------|
| **Terminate** | SIGTERM (15) | Politely ask process to stop. Process can catch it and clean up. |
| **Kill** | SIGKILL (9) | Immediately destroy the process. Cannot be caught or ignored. |
| **Stop** | SIGSTOP (19) | Pause/suspend the process. It stays in memory but doesn't run. |
| **Continue** | SIGCONT (18) | Resume a stopped process. |

```bash
kill -15 <PID>   # Terminate (graceful)
kill -9  <PID>   # Kill (force)
kill -19 <PID>   # Stop
kill -18 <PID>   # Continue
```
{{< /qa >}}

{{< qa num="20" q="Which kill signal is best and why?" level="intermediate" >}}
**Ans:**

**SIGTERM (15)** is the best choice in most cases because:
- It asks the process to **cleanly shut down**
- The process can save state, close connections, and release resources
- It avoids data corruption

```bash
kill -15 <PID>   # Preferred
# or simply
kill <PID>       # SIGTERM is the default
```

Only use **SIGKILL (9)** when a process is unresponsive to SIGTERM:

```bash
kill -9 <PID>    # Last resort — no cleanup, risk of data loss
```

**Recommended approach:**
```bash
kill -15 <PID>          # Try graceful first
sleep 5
kill -0 <PID> && kill -9 <PID>   # Force kill only if still alive
```
{{< /qa >}}

{{< qa num="21" q="A process is consuming high CPU and memory. How do you manage it?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Identify the culprit
top              # Press P to sort by CPU, M for memory
htop             # More visual, filter-friendly

# Step 2: Get the PID
ps aux --sort=-%cpu | head -10   # Top CPU consumers
ps aux --sort=-%mem | head -10   # Top memory consumers

# Step 3: Investigate
ls -la /proc/<PID>/exe           # What binary is it?
cat /proc/<PID>/status           # Detailed info
lsof -p <PID>                    # Files it has open

# Step 4: Reduce priority (if you can't kill it)
renice +10 <PID>                 # Lower priority (nicer = less CPU)

# Step 5: Kill if necessary
kill -15 <PID>    # Graceful
kill -9  <PID>    # Force
```
{{< /qa >}}

{{< qa num="22" q="Do you know about htop, iotop, iostat, vmstat?" level="intermediate" >}}
**Ans:**

**`htop`** — Enhanced `top` with colors, mouse support, easy kill/nice controls
```bash
htop
```

**`iotop`** — Shows disk I/O usage per process (like top but for disk)
```bash
sudo iotop
sudo iotop -o    # Only show processes doing I/O
```

**`iostat`** — Reports CPU and disk I/O statistics
```bash
iostat           # One-time snapshot
iostat -x 2 5   # Extended stats, every 2s, 5 times
```

**`vmstat`** — Reports virtual memory, processes, CPU stats
```bash
vmstat           # One-time
vmstat 2 5      # Every 2 seconds, 5 iterations
# Fields: r=run queue, b=blocked, swpd, free, buff, cache, si, so, bi, bo, in, cs, us, sy, id, wa
```
{{< /qa >}}

{{< qa num="23" q="What is difference between htop and top?" level="basic" >}}
**Ans:**

| Feature | top | htop |
|---------|-----|------|
| Interface | Text-based, minimal | Color-coded, visual |
| Mouse support | No | Yes |
| Kill process | Via key commands | Click or F9 |
| Scroll | Limited | Horizontal + vertical |
| CPU per core | No (combined) | Yes (individual bars) |
| Tree view | No | Yes (F5) |
| Installation | Built-in | Needs install (`apt/yum install htop`) |
{{< /qa >}}

{{< qa num="24" q="What is difference between ps and top?" level="basic" >}}
**Ans:**

| Feature | ps | top |
|---------|----|----|
| Output | Static snapshot | Dynamic, real-time |
| Updates | One-time | Continuous (every 3s) |
| Use case | Scripting, logging | Live monitoring |
| Interactivity | None | Kill, renice, filter |
| Resource usage | Minimal | Slightly higher |

```bash
ps aux | grep nginx    # Check if nginx is running (static)
top                    # Monitor all processes live
```
{{< /qa >}}

{{< qa num="25" q="How do you fix 'Permission Denied' errors on EC2?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check permissions of the file/directory
ls -la /path/to/file

# Step 2: Check current user
whoami
id

# Step 3: Check file ownership
stat /path/to/file

# Step 4: Fix ownership (if you have sudo)
sudo chown ec2-user:ec2-user /path/to/file

# Step 5: Fix permissions
sudo chmod 755 /path/to/directory
sudo chmod 644 /path/to/file

# Step 6: For SSH key permission denied
chmod 400 ~/.ssh/my-key.pem       # Key must be owner-read only
chmod 700 ~/.ssh/                  # .ssh dir must be 700
chmod 600 ~/.ssh/authorized_keys   # authorized_keys must be 600

# Step 7: Check SELinux/AppArmor if applicable
getenforce                         # Check SELinux status
sestatus
```
{{< /qa >}}

{{< qa num="26" q="How do you verify file ownership and permissions?" level="basic" >}}
**Ans:**

```bash
# Long listing format — shows permissions, owner, group
ls -la /path/to/file

# Detailed stat output
stat /path/to/file

# Example output of ls -la:
# -rw-r--r-- 1 ec2-user ec2-user 1234 Apr 26 10:00 myfile.txt
# ^type+perms  ^owner   ^group

# Permission breakdown:
# r=4, w=2, x=1
# 755 = rwxr-xr-x (owner:rwx, group:r-x, others:r-x)
# 644 = rw-r--r-- (owner:rw-, group:r--, others:r--)
```
{{< /qa >}}

{{< qa num="27" q="How do you change directory ownership for Jenkins deployments?" level="intermediate" >}}
**Ans:**

```bash
# Change owner to jenkins user
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/

# Change owner of deployment directory
sudo chown -R jenkins:jenkins /opt/myapp/

# Give jenkins write permissions
sudo chmod -R 755 /opt/myapp/

# Verify
ls -la /opt/myapp/

# If Jenkins needs to run as root occasionally, add to sudoers:
sudo visudo
# Add: jenkins ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
```
{{< /qa >}}

{{< qa num="28" q="How do you identify files recently modified by the root user?" level="intermediate" >}}
**Ans:**

```bash
# Files modified in last 24 hours owned by root
find / -user root -mtime -1 2>/dev/null

# Files modified in last 60 minutes
find /etc -user root -mmin -60

# Files changed recently (any user) with timestamps
find /var/log -newer /tmp/reference_file -ls

# Check audit logs for root activity (if auditd is running)
ausearch -ua root -ts today

# Check bash history for root
sudo cat /root/.bash_history

# Check last logins
last root
```
{{< /qa >}}

{{< qa num="29" q="How do you check sudo privileges of a user?" level="basic" >}}
**Ans:**

```bash
# Check what sudo commands a user can run
sudo -l                    # For current user
sudo -l -U username        # For another user (as root)

# View sudoers file
sudo cat /etc/sudoers
sudo visudo               # Safe edit with syntax check

# Check if user is in sudo/wheel group
groups username
id username
getent group sudo          # Debian/Ubuntu
getent group wheel         # RHEL/CentOS
```
{{< /qa >}}

{{< qa num="30" q="How do you give only read-only access to a file?" level="basic" >}}
**Ans:**

```bash
# Remove write permissions for everyone
chmod a-w filename

# Set read-only for owner only
chmod 400 filename      # r--------

# Read for owner and group, no write
chmod 440 filename      # r--r-----

# Read for all, no write for anyone
chmod 444 filename      # r--r--r--

# Make immutable (even root can't write without removing flag)
sudo chattr +i filename
lsattr filename         # Verify immutable flag
```
{{< /qa >}}

{{< qa num="31" q="How do you check which user modified a file last?" level="intermediate" >}}
**Ans:**

```bash
# Check last modification time (but not which user)
stat filename
ls -la filename

# To track WHO modified — use auditd
# Install and enable auditd
sudo systemctl start auditd

# Add a watch rule on the file
sudo auditctl -w /path/to/file -p wa -k file_watch

# Search audit logs for that file
sudo ausearch -f /path/to/file

# Alternative: Check /var/log/auth.log or /var/log/secure
grep "filename" /var/log/auth.log
```

> **Note:** Linux doesn't natively record which user last modified a file without auditd. `stat` only shows timestamps.
{{< /qa >}}

{{< qa num="32" q="How do you allow a non-root user to run Docker commands?" level="intermediate" >}}
**Ans:**

```bash
# Add user to the docker group
sudo usermod -aG docker username

# Apply group change (user must log out and log back in, or:)
newgrp docker

# Verify
docker ps        # Should work without sudo
groups           # Should show 'docker' in list

# Alternative: Use sudo for specific docker commands in sudoers
sudo visudo
# Add: username ALL=(ALL) NOPASSWD: /usr/bin/docker
```

> **Security note:** The `docker` group gives root-equivalent access. Use carefully.
{{< /qa >}}

{{< qa num="33" q="How do you troubleshoot high CPU usage in an EC2 instance?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Identify top CPU-consuming process
top             # Press P to sort by CPU
htop

# Step 2: Get details on the process
ps aux --sort=-%cpu | head -10
lsof -p <PID>   # What files/connections it uses

# Step 3: Check system-wide load
uptime
vmstat 1 5      # CPU wait, idle, system time

# Step 4: Check for CPU steal (in VMs)
top             # %st column = CPU stolen by hypervisor

# Step 5: Check recent deployments or cron jobs
journalctl -xe
crontab -l
cat /var/log/cron

# Step 6: Resolve
renice +10 <PID>    # Reduce priority
kill -15 <PID>      # Terminate if rogue process
```
{{< /qa >}}

{{< qa num="34" q="How do you check free space inside a specific folder?" level="basic" >}}
**Ans:**

```bash
# Disk usage of a folder (human-readable)
du -sh /var/log/

# Show all subdirectory sizes
du -h /var/log/

# Sort by size, show top 10 folders
du -h /var/ | sort -rh | head -10

# Check overall disk space
df -h

# Check inode usage (another type of "space")
df -i
```
{{< /qa >}}

{{< qa num="35" q="How do you archive old log files automatically?" level="intermediate" >}}
**Ans:**

**Method 1: logrotate (recommended)**
```bash
# Configure logrotate
sudo nano /etc/logrotate.d/myapp

# Example config:
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0644 appuser appuser
    postrotate
        systemctl reload myapp
    endscript
}

# Test the config
sudo logrotate -d /etc/logrotate.d/myapp

# Force run
sudo logrotate -f /etc/logrotate.d/myapp
```

**Method 2: Cron + tar**
```bash
# Add to crontab: archive logs older than 7 days
crontab -e

# Daily at midnight, compress logs older than 7 days
0 0 * * * find /var/log/myapp -name "*.log" -mtime +7 -exec gzip {} \;
```
{{< /qa >}}

{{< qa num="36" q="How do you check disk I/O performance on EC2?" level="advanced" >}}
**Ans:**

```bash
# Real-time I/O per process
sudo iotop -o

# Device-level I/O stats
iostat -x 2 5
# Look for: %util (device saturation), await (I/O wait time ms), r/s, w/s

# Check I/O wait in top
top
# %wa column = CPU time waiting for I/O

# Use vmstat for block I/O
vmstat 2 5
# bi = blocks in (reads), bo = blocks out (writes)

# Check specific disk
iostat -x sda 2 5

# AWS-specific: Check EBS CloudWatch metrics
# Metrics: VolumeReadOps, VolumeWriteOps, VolumeQueueLength
```
{{< /qa >}}

{{< qa num="37" q="How do you check system logs for EC2 boot errors?" level="intermediate" >}}
**Ans:**

```bash
# View full boot log for current boot
journalctl -b

# View previous boot log
journalctl -b -1

# Filter for errors only
journalctl -b -p err

# Check kernel ring buffer
dmesg
dmesg | grep -i error
dmesg | grep -i fail

# Traditional log files
cat /var/log/messages       # RHEL/CentOS
cat /var/log/syslog         # Debian/Ubuntu
cat /var/log/boot.log

# EC2 Console Output (from AWS Console)
# Go to: EC2 → Instance → Actions → Monitor and troubleshoot → Get system log
```
{{< /qa >}}

{{< qa num="38" q="How do you troubleshoot slow SSH connections to EC2?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Test with verbose output to see where it hangs
ssh -vvv -i key.pem ec2-user@<ip>

# Step 2: Check DNS resolution (common cause of slow SSH)
# On the server, edit /etc/ssh/sshd_config:
UseDNS no

# Step 3: Check GSSAPI authentication
# In /etc/ssh/sshd_config:
GSSAPIAuthentication no

# Step 4: Restart sshd after changes
sudo systemctl restart sshd

# Step 5: Check security group allows port 22 from your IP
# In AWS Console: Security Group → Inbound rules → Port 22

# Step 6: Check server load
uptime
top
```
{{< /qa >}}

{{< qa num="39" q="How do you schedule EC2 instances to stop/start automatically using cron?" level="intermediate" >}}
**Ans:**

**Method 1: AWS Instance Scheduler (managed service)**

**Method 2: Lambda + EventBridge (CloudWatch Events)**
```bash
# Create Lambda to start/stop instance
# EventBridge Rule: cron(0 8 * * ? *)  → start at 8 AM UTC
# EventBridge Rule: cron(0 20 * * ? *) → stop at 8 PM UTC
```

**Method 3: AWS CLI in cron on a management server**
```bash
# On a management EC2, edit crontab
crontab -e

# Stop instance at 8 PM UTC
0 20 * * * aws ec2 stop-instances --instance-ids i-1234567890abcdef0 --region us-east-1

# Start instance at 8 AM UTC
0 8 * * * aws ec2 start-instances --instance-ids i-1234567890abcdef0 --region us-east-1
```
{{< /qa >}}

{{< qa num="40" q="There is an sshd process taking high CPU and you don't have rights to kill it. What do you do?" level="advanced" >}}
**Ans:**

```bash
# Step 1: Confirm it's actually high CPU
top -p $(pgrep sshd | head -1)

# Step 2: Try to renice (lower CPU priority) — may work without root
renice +19 <PID>

# Step 3: Investigate connections before escalating
ss -anp | grep sshd           # Check active connections
who                            # Who is logged in
last                           # Recent logins

# Step 4: Escalate to admin/ops team with evidence
# Provide: PID, CPU%, user, open connections

# Step 5: If you have sudo for systemctl only:
sudo systemctl restart sshd   # Restart the service gracefully

# Step 6: Report through incident management process
# Document: time, PID, CPU%, connections observed, action taken
```
{{< /qa >}}

{{< qa num="41" q="How do you sync data between two EC2 instances?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: rsync (most efficient — only transfers changes)
rsync -avz -e "ssh -i key.pem" /local/path/ ec2-user@<dest-ip>:/remote/path/

# Method 2: scp (simple copy)
scp -i key.pem -r /source/ ec2-user@<dest-ip>:/destination/

# Method 3: rsync in cron for periodic sync
crontab -e
*/15 * * * * rsync -avz /data/ ec2-user@<dest-ip>:/data/ >> /var/log/sync.log 2>&1

# Method 4: Use S3 as intermediary
aws s3 sync /data/ s3://my-bucket/data/
# On destination:
aws s3 sync s3://my-bucket/data/ /data/
```
{{< /qa >}}

{{< qa num="42" q="How do you back up EC2 files to an S3 bucket?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: AWS CLI sync
aws s3 sync /var/www/html/ s3://my-backup-bucket/webfiles/

# With date-stamped backup
aws s3 cp /var/log/ s3://my-backup-bucket/logs/$(date +%Y-%m-%d)/ --recursive

# Method 2: Cron-based backup
crontab -e
0 2 * * * aws s3 sync /data/ s3://my-backup-bucket/data/ --delete >> /var/log/backup.log 2>&1

# Method 3: Tar + upload
tar -czf /tmp/backup-$(date +%Y%m%d).tar.gz /data/
aws s3 cp /tmp/backup-$(date +%Y%m%d).tar.gz s3://my-backup-bucket/

# Ensure EC2 has IAM role with s3:PutObject permission
aws sts get-caller-identity    # Confirm identity/role
```
{{< /qa >}}

</div>
