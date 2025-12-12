# System Administration - Interview Q&A

## Linux/Unix Administration

### Q1: What are the essential Linux commands every DevOps engineer should know?

**Answer:**
**File operations:**
```bash
ls -la          # List files with details
cd /path        # Change directory
pwd             # Print working directory
cp file1 file2  # Copy file
mv file1 file2  # Move/rename file
rm file         # Remove file
mkdir dir       # Create directory
rmdir dir       # Remove directory
find /path -name "*.log"  # Find files
```

**Text processing:**
```bash
cat file        # Display file
head file       # First 10 lines
tail file       # Last 10 lines
tail -f file    # Follow file (watch logs)
grep "pattern" file  # Search text
sed 's/old/new/g' file  # Replace text
awk '{print $1}' file   # Process columns
```

**System information:**
```bash
ps aux          # Running processes
top             # Process monitor
htop            # Better process monitor
df -h           # Disk usage
du -sh *        # Directory sizes
free -h         # Memory usage
uname -a        # System info
```

**Permissions:**
```bash
chmod 755 file  # Change permissions
chown user:group file  # Change owner
ls -l           # View permissions
```

**Network:**
```bash
netstat -tulpn  # Network connections
ss -tulpn       # Modern netstat
ping host       # Test connectivity
curl url        # HTTP request
wget url        # Download file
```

**Package management:**
```bash
# Ubuntu/Debian
apt update
apt install package
apt remove package

# RHEL/CentOS
yum install package
yum remove package
# or
dnf install package
```

### Q2: Explain Linux file permissions and how to manage them.

**Answer:**
**Permission structure:**
```
-rwxrwxrwx
│││││││││
││││││││└─ Other (others)
│││││││└── Group
││││││└─── Owner
│││││└──── Execute
││││└───── Write
│││└────── Read
││└─────── File type (- = file, d = directory)
│└──────── Special permissions
```

**Permission types:**
- **Read (r)**: Can view file contents
- **Write (w)**: Can modify file
- **Execute (x)**: Can run file as program

**Numeric notation:**
```
7 = rwx (read + write + execute)
6 = rw- (read + write)
5 = r-x (read + execute)
4 = r-- (read only)
0 = --- (no permissions)
```

**Common permissions:**
```bash
755 = rwxr-xr-x  # Owner: full, Group/Other: read+execute
644 = rw-r--r--  # Owner: read+write, Group/Other: read
600 = rw-------  # Owner only
```

**Change permissions:**
```bash
chmod 755 script.sh      # Numeric
chmod u+x script.sh      # Add execute for owner
chmod g+w file.txt       # Add write for group
chmod o-r file.txt       # Remove read for others
chmod -R 755 directory/  # Recursive
```

**Change ownership:**
```bash
chown user:group file
chown -R user:group directory/
```

**Best practices:**
- Use least privilege (don't give more than needed)
- Scripts need execute permission
- Sensitive files: 600 (owner only)
- Web files: 644 (readable, not writable)
- Executables: 755

### Q3: How do you manage services in Linux?

**Answer:**
**systemd (modern Linux):**
```bash
# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Enable on boot
sudo systemctl enable nginx

# Disable on boot
sudo systemctl disable nginx

# Check status
sudo systemctl status nginx

# List all services
sudo systemctl list-units --type=service

# View logs
sudo journalctl -u nginx
sudo journalctl -u nginx -f  # Follow logs
```

**Service files location:**
```
/etc/systemd/system/  # Custom services
/lib/systemd/system/  # System services
```

**Example service file:**
```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

**Old init.d (legacy):**
```bash
sudo service nginx start
sudo service nginx stop
sudo service nginx restart
sudo service nginx status
```

**Best practices:**
- Use systemd for new services
- Enable services that should start on boot
- Use restart policies
- Run services as non-root user
- Log to journald or files

### Q4: Explain process management in Linux.

**Answer:**
**View processes:**
```bash
ps aux              # All processes
ps aux | grep nginx # Filter
top                 # Interactive monitor
htop                # Better interactive monitor
pgrep nginx         # Find process ID by name
```

**Process information:**
- **PID**: Process ID
- **PPID**: Parent Process ID
- **USER**: Owner
- **%CPU**: CPU usage
- **%MEM**: Memory usage
- **STAT**: Status (R=running, S=sleeping, Z=zombie)

**Manage processes:**
```bash
kill PID            # Send TERM signal (graceful)
kill -9 PID         # Force kill (SIGKILL)
killall nginx       # Kill all nginx processes
pkill nginx         # Kill by name
```

**Signals:**
- **SIGTERM (15)**: Graceful shutdown (default for kill)
- **SIGKILL (9)**: Force kill (can't be ignored)
- **SIGHUP (1)**: Reload configuration
- **SIGINT (2)**: Interrupt (Ctrl+C)

**Background processes:**
```bash
command &           # Run in background
nohup command &     # Run in background, survive logout
jobs                # List background jobs
fg %1               # Bring to foreground
bg %1               # Send to background
```

**Best practices:**
- Try graceful shutdown first (kill, then kill -9 if needed)
- Use process managers (systemd, supervisor) for production
- Monitor process resources
- Set resource limits (ulimit)

## Windows Administration

### Q5: What are essential Windows administration tasks?

**Answer:**
**PowerShell basics:**
```powershell
# Get system info
Get-ComputerInfo
Get-Service
Get-Process
Get-EventLog -LogName Application

# Manage services
Get-Service
Start-Service -Name "ServiceName"
Stop-Service -Name "ServiceName"
Restart-Service -Name "ServiceName"
Set-Service -Name "ServiceName" -StartupType Automatic

# Manage processes
Get-Process
Stop-Process -Name "ProcessName"
Stop-Process -Id 1234

# File operations
Get-ChildItem
Copy-Item source dest
Move-Item source dest
Remove-Item file
New-Item -ItemType Directory -Name "dir"
```

**Windows Services:**
```powershell
# List services
Get-Service

# Start/Stop service
Start-Service -Name "W3SVC"  # IIS
Stop-Service -Name "W3SVC"

# Service status
Get-Service -Name "W3SVC"
```

**Event Logs:**
```powershell
# View logs
Get-EventLog -LogName Application -Newest 10
Get-EventLog -LogName System
Get-WinEvent -LogName Application | Where-Object {$_.LevelDisplayName -eq "Error"}

# Filter by time
Get-EventLog -LogName Application -After (Get-Date).AddHours(-1)
```

**IIS Management:**
```powershell
# IIS commands
Import-Module WebAdministration
Get-Website
Start-Website -Name "Default Web Site"
Stop-Website -Name "Default Web Site"
Get-WebAppPoolState
```

### Q6: How do you manage Windows services and scheduled tasks?

**Answer:**
**Windows Services:**
```powershell
# View all services
Get-Service

# View specific service
Get-Service -Name "W3SVC"

# Start service
Start-Service -Name "ServiceName"

# Stop service
Stop-Service -Name "ServiceName"

# Set startup type
Set-Service -Name "ServiceName" -StartupType Automatic
Set-Service -Name "ServiceName" -StartupType Manual
Set-Service -Name "ServiceName" -StartupType Disabled

# Service dependencies
Get-Service -Name "ServiceName" | Select-Object -ExpandProperty DependentServices
Get-Service -Name "ServiceName" | Select-Object -ExpandProperty ServicesDependedOn
```

**Scheduled Tasks:**
```powershell
# List tasks
Get-ScheduledTask

# Create task
$action = New-ScheduledTaskAction -Execute "C:\Scripts\backup.ps1"
$trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -TaskName "DailyBackup" -Action $action -Trigger $trigger

# Run task
Start-ScheduledTask -TaskName "DailyBackup"

# Disable task
Disable-ScheduledTask -TaskName "DailyBackup"

# Remove task
Unregister-ScheduledTask -TaskName "DailyBackup"
```

**Task Scheduler GUI:**
- Open: `taskschd.msc`
- Create/view/edit tasks
- View task history
- Export/import tasks

**Best practices:**
- Use service accounts for scheduled tasks
- Set appropriate permissions
- Log task execution
- Test tasks before production
- Document task purposes

## Common Tasks and Troubleshooting

### Q7: How do you troubleshoot high CPU usage?

**Answer:**
**Linux:**
```bash
# Find processes using CPU
top                    # Interactive, sort by CPU (press P)
htop                   # Better interface
ps aux --sort=-%cpu | head -10

# Per-core usage
mpstat -P ALL 1

# Process details
pidstat -u 1          # CPU usage per process
strace -p PID         # System calls (advanced)
```

**Windows:**
```powershell
# CPU usage
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Per-process
Get-Counter "\Process(*)\% Processor Time" | Select-Object -ExpandProperty CounterSamples | Sort-Object CookedValue -Descending

# Task Manager
taskmgr
```

**Common causes:**
- Infinite loops in code
- High traffic
- Resource-intensive operations
- Malware
- Too many processes

**Solutions:**
- Optimize code
- Scale horizontally (more instances)
- Scale vertically (more CPU)
- Kill runaway processes
- Investigate with profiling tools

### Q8: How do you troubleshoot high memory usage?

**Answer:**
**Linux:**
```bash
# Memory usage
free -h
top                    # Sort by memory (press M)
ps aux --sort=-%mem | head -10

# Detailed memory
cat /proc/meminfo
vmstat 1

# Process memory
pmap -x PID            # Memory map of process
```

**Windows:**
```powershell
# Memory usage
Get-Counter "\Memory\Available MBytes"
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10

# System memory
systeminfo | findstr "Total Physical Memory"
```

**Common causes:**
- Memory leaks
- Too many processes
- Large datasets in memory
- Inefficient code
- Insufficient memory

**Solutions:**
- Fix memory leaks
- Optimize code
- Add more memory
- Scale horizontally
- Use caching wisely
- Restart services periodically

### Q9: How do you troubleshoot disk space issues?

**Answer:**
**Linux:**
```bash
# Disk usage
df -h                 # Filesystem usage
du -sh *              # Directory sizes
du -h --max-depth=1   # One level deep

# Find large files
find / -type f -size +100M
find / -type f -size +1G -exec ls -lh {} \;

# Find large directories
du -h / | sort -rh | head -20

# Clean up
# Logs
journalctl --vacuum-time=7d  # Keep last 7 days
# Package cache
apt clean             # Ubuntu
yum clean all         # RHEL
# Docker
docker system prune
```

**Windows:**
```powershell
# Disk usage
Get-PSDrive -PSProvider FileSystem
Get-ChildItem -Recurse | Measure-Object -Property Length -Sum

# Large files
Get-ChildItem -Recurse -File | Sort-Object Length -Descending | Select-Object -First 10

# Clean up
# Disk cleanup
cleanmgr
# Windows updates
Remove-WindowsUpdateLog
```

**Common issues:**
- Log files growing
- Temporary files
- Old backups
- Docker images/containers
- Package caches

**Solutions:**
- Set up log rotation
- Clean temporary files
- Remove old backups
- Clean Docker
- Monitor disk usage
- Add more storage
- Use cloud storage

### Q10: How do you troubleshoot network connectivity issues?

**Answer:**
**Linux:**
```bash
# Test connectivity
ping host
ping -c 4 host        # 4 packets

# DNS resolution
nslookup domain
dig domain
host domain

# Network interfaces
ip addr show
ifconfig
ip link show

# Routing
ip route show
route -n
traceroute host

# Port connectivity
telnet host port
nc -zv host port
curl -v http://host:port

# Network statistics
netstat -tulpn
ss -tulpn
iftop                 # Bandwidth usage
```

**Windows:**
```powershell
# Test connectivity
Test-NetConnection host
Test-NetConnection host -Port 80

# DNS
Resolve-DnsName domain
nslookup domain

# Network interfaces
Get-NetIPAddress
Get-NetAdapter
ipconfig /all

# Routing
Get-NetRoute
tracert host

# Port connectivity
Test-NetConnection -ComputerName host -Port 80
```

**Common issues:**
- DNS not resolving
- Firewall blocking
- Network interface down
- Routing problems
- Service not listening

**Solutions:**
- Check DNS servers
- Verify firewall rules
- Restart network service
- Check routing tables
- Verify service is running
- Check network cables (physical)

## Performance Optimization

### Q11: How do you optimize system performance?

**Answer:**
**CPU optimization:**
- Identify CPU bottlenecks
- Optimize code
- Use caching
- Scale horizontally
- Upgrade CPU

**Memory optimization:**
- Fix memory leaks
- Optimize data structures
- Use connection pooling
- Implement caching
- Add more RAM

**Disk optimization:**
- Use SSDs
- Optimize database queries
- Implement log rotation
- Clean up regularly
- Use RAID for performance

**Network optimization:**
- Use CDN for static content
- Optimize payload sizes
- Implement compression
- Use connection pooling
- Optimize DNS

**Application optimization:**
- Profile application
- Optimize database queries
- Use async operations
- Implement caching
- Minimize dependencies

**Monitoring:**
- Set up performance monitoring
- Track key metrics
- Set up alerts
- Regular performance reviews

**Best practices:**
- Monitor first, optimize second
- Measure before and after
- Focus on bottlenecks
- Test optimizations
- Document changes

### Q12: Explain system backup and recovery procedures.

**Answer:**
**Backup types:**
- **Full backup**: Everything
- **Incremental**: Changes since last backup
- **Differential**: Changes since full backup

**What to backup:**
- Application data
- Databases
- Configuration files
- User data
- System state (Windows)

**Backup tools:**
- **Azure Backup**: For Azure VMs
- **rsync**: Linux file sync
- **tar**: Archive files
- **Windows Backup**: Built-in Windows tool
- **Database backups**: Native tools (mysqldump, pg_dump)

**Linux backup example:**
```bash
# Full backup
tar -czf backup-$(date +%Y%m%d).tar.gz /important/data

# Incremental with rsync
rsync -av --link-dest=/backup/last /source/ /backup/current
```

**Windows backup:**
```powershell
# Windows Server Backup
wbadmin start backup -backupTarget:E: -include:C:,D: -allCritical

# File backup
Copy-Item -Path "C:\Data" -Destination "E:\Backup" -Recurse
```

**Recovery procedures:**
1. Identify what needs recovery
2. Stop affected services
3. Restore from backup
4. Verify data integrity
5. Restart services
6. Test functionality

**Best practices:**
- Regular backups (daily minimum)
- Test restore procedures
- Store backups offsite
- Encrypt backups
- Document procedures
- Monitor backup success
- Keep multiple backup copies
- Test disaster recovery regularly

