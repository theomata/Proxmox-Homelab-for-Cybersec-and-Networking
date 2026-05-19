# Day 2 — Lab Journal

**Date:** Day 2  
**Focus:** WorldSkills Assignment — Linux Administration, Filesystem, Users, Services, Networking, Infrastructure  
**Reference:** WorldSkills IT Network Systems Administration – Initial Training Assignment  
**Status:** 🔄 In Progress

---

## 🎯 Session Goals

- [ ] Master Linux navigation and filesystem structure
- [ ] Practice file operations confidently
- [ ] Understand users, groups, and permissions
- [ ] Manage services with systemctl and read logs with journalctl
- [ ] Deepen networking command skills
- [ ] Build first infrastructure lab (SSH, Apache/Nginx, Bind9 DNS, DHCP)
- [ ] Practice deliberate break-and-fix troubleshooting

---

## 📚 Section 1 — Linux Navigation & Filesystem

See dedicated reference: [`linux/administration/filesystem-navigation.md`](../../linux/administration/filesystem-navigation.md)

### Key Commands Practiced

```bash
pwd          # Print working directory — where am I right now?
ls -la       # List all files including hidden, with permissions
cd /etc      # Change directory (absolute path)
cd ../       # Move up one level (relative path)
tree /etc    # Visual directory tree (install with apt install tree)
find / -name "sshd_config"   # Find a file by name
locate sshd_config           # Faster find (uses pre-built index)
```

### Linux Filesystem Structure

| Directory | Purpose |
|-----------|---------|
| `/etc` | System-wide configuration files |
| `/var` | Variable data — logs, mail, databases |
| `/home` | User home directories |
| `/usr` | User programs and utilities |
| `/bin` | Essential system binaries |
| `/sbin` | System administration binaries |
| `/tmp` | Temporary files (cleared on reboot) |
| `/proc` | Virtual filesystem — live kernel/process info |
| `/sys` | Virtual filesystem — hardware and kernel info |
| `/root` | Home directory for the root user |

### Key Realization
Absolute vs relative paths matter constantly in scripting and configuration:
- **Absolute:** `/etc/ssh/sshd_config` — always works from anywhere
- **Relative:** `../ssh/sshd_config` — depends on your current directory

---

## 📚 Section 2 — File Operations

See dedicated reference: [`linux/administration/file-operations.md`](../../linux/administration/file-operations.md)

### Key Commands Practiced

```bash
mkdir -p /home/user/projects/lab    # Create nested directories
touch notes.txt                      # Create empty file
cp notes.txt notes-backup.txt        # Copy file
mv notes.txt renamed.txt             # Move/rename file
rm renamed.txt                       # Delete file
rm -rf directory/                    # Delete directory recursively (careful!)
cat /etc/hosts                       # Display file contents
less /var/log/syslog                 # Page through large files
nano /etc/hosts                      # Edit file in nano
```

### Editing Config Files Safely

Always back up before editing:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

If you break it, restore from backup:
```bash
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
sudo systemctl restart ssh
```

---

## 📚 Section 3 — Users & Permissions

See dedicated reference: [`linux/administration/users-permissions.md`](../../linux/administration/users-permissions.md)

### Key Commands Practiced

```bash
whoami                        # Current logged-in user
id                            # User ID, group ID, all groups
sudo whoami                   # Confirm sudo works (returns: root)
groups                        # List groups current user belongs to

# Create user and group
sudo adduser labuser
sudo groupadd labgroup
sudo usermod -aG labgroup labuser

# Permissions
chmod 755 script.sh           # rwxr-xr-x
chmod 600 private.key         # rw------- (SSH key permissions)
chown labuser:labgroup file   # Change owner and group
```

### Understanding `rwx` Permissions

```
-rwxr-xr--  1  user  group  size  date  filename
 |||||||||||
 |└──┬──┘└──┬──┘└──┬──┘
 |  owner  group  others
 |
 file type: - = file, d = directory, l = symlink
```

| Symbol | Numeric | Meaning |
|--------|---------|---------|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `rwx` | 7 | Full access |
| `r-x` | 5 | Read + execute |
| `r--` | 4 | Read only |
| `---` | 0 | No access |

### Real-World Example

SSH private keys **must** be 600 or they'll be rejected:
```bash
chmod 600 ~/.ssh/id_rsa
# If permissions are too open, SSH refuses to use the key
```

---

## 📚 Section 4 — Services & Logs

See dedicated reference: [`linux/services/systemctl.md`](../../linux/services/systemctl.md)

### Key Commands Practiced

```bash
systemctl status apache2          # Is Apache running?
systemctl start apache2           # Start it
systemctl stop apache2            # Stop it
systemctl restart apache2         # Restart (full stop + start)
systemctl reload apache2          # Reload config without downtime
systemctl enable apache2          # Auto-start at boot
systemctl disable apache2         # Don't auto-start

journalctl -u apache2             # All logs for Apache
journalctl -u apache2 -f          # Follow live logs
journalctl -u apache2 -n 50       # Last 50 lines
journalctl -xe                    # Extended log with context (good for errors)
```

### Investigating a Service Failure

When a service fails to start, this is the exact diagnostic sequence:
```bash
# Step 1: What is the status?
systemctl status apache2

# Step 2: Look at recent logs
journalctl -u apache2 -n 30

# Step 3: Extended context with errors
journalctl -xe

# Step 4: Test config file for syntax errors
apache2ctl configtest       # Apache
nginx -t                    # Nginx
named-checkconf             # Bind9 DNS
```

---

## 📚 Section 5 — Networking Fundamentals

See dedicated references:
- [`networking/commands/ip-route.md`](../../networking/commands/ip-route.md)
- [`networking/commands/ss-tulnp.md`](../../networking/commands/ss-tulnp.md)
- [`networking/commands/dns-tools.md`](../../networking/commands/dns-tools.md)

### Key Commands Practiced

```bash
ip a                         # Show interfaces and IP addresses
ip route                     # Show routing table
ss -tulnp                    # Show listening ports and processes
ping 192.168.1.1             # Test gateway
ping 8.8.8.8                 # Test internet routing
traceroute 8.8.8.8           # Show hop-by-hop path to destination
dig google.com               # DNS lookup (detailed)
nslookup google.com          # DNS lookup (simple)
```

### New Commands — `traceroute` and DNS tools

`traceroute` shows every router hop between you and the destination:
```bash
traceroute 8.8.8.8
# 1  192.168.1.1     1.2ms    ← Home router
# 2  10.x.x.x        8.5ms    ← ISP gateway
# 3  ...             20.1ms   ← ISP backbone
# 4  8.8.8.8         25.3ms   ← Google DNS
```

`dig` queries DNS directly and shows exactly what the DNS server returned:
```bash
dig google.com
# Shows: A record, TTL, which DNS server answered, query time
```

---

## 🏗️ Section 6 — First Infrastructure Lab

See dedicated references:
- [`linux/services/apache-nginx.md`](../../linux/services/apache-nginx.md)
- [`linux/services/bind9-dns.md`](../../linux/services/bind9-dns.md)
- [`linux/services/dhcp-server.md`](../../linux/services/dhcp-server.md)

### Services Deployed

| Service | Package | Port | Purpose |
|---------|---------|------|---------|
| OpenSSH | `openssh-server` | 22/TCP | Remote administration |
| Web Server | `apache2` or `nginx` | 80/TCP | HTTP serving |
| DNS Server | `bind9` | 53/UDP+TCP | Name resolution |
| DHCP Server | `isc-dhcp-server` | 67/UDP | IP address assignment |

### Verification Checklist

After deploying each service:
```bash
# Is it running?
systemctl status <service>

# Is it listening on the right port?
sudo ss -tulnp | grep <port>

# Can another VM reach it?
# From a second VM:
curl http://192.168.1.60          # Test Apache/Nginx
dig @192.168.1.60 google.com     # Test Bind9 DNS
```

---

## 🔧 Section 7 — Break-and-Fix Troubleshooting

See dedicated reference: [`networking/troubleshooting/break-and-fix-scenarios.md`](../../networking/troubleshooting/break-and-fix-scenarios.md)

### Why Deliberate Breaking Is Important

You cannot troubleshoot what you've never broken. WorldSkills competition scenarios present broken infrastructure — you must diagnose and fix it under time pressure.

Breaking things yourself teaches you:
- What broken looks like (error messages, failed status)
- The exact path from symptom → diagnosis → fix
- Confidence that you can recover from anything

### Scenarios Practiced

| Scenario | How to Break | How to Fix |
|----------|-------------|------------|
| SSH fails | Edit sshd_config with syntax error | Restore backup, restart ssh |
| Apache won't start | Delete required config file | Restore from backup or reinstall |
| No DNS resolution | Wrong nameserver in /etc/resolv.conf | Fix nameserver entry |
| Can't reach gateway | Delete default route | `ip route add default via 192.168.1.1` |
| Service not starting at boot | `systemctl disable service` | `systemctl enable service` |

---

## 🧠 Key Realizations — Day 2

1. Linux permissions are everywhere — SSH keys, config files, scripts. Getting them wrong breaks things silently.
2. `journalctl -xe` is the first command to run when any service fails. It almost always points directly at the cause.
3. Every infrastructure service (DNS, DHCP, web) follows the same pattern: install → configure → enable → test → troubleshoot.
4. Breaking things deliberately in a lab (with snapshots!) is one of the most effective ways to learn.
5. WorldSkills competition is really a test of this exact skill: arriving at a broken system and knowing exactly where to look.

---

## 📌 Next Steps (Day 3)

- [ ] Add pfSense VM — firewall and routing
- [ ] Configure VLANs
- [ ] Practice nmap scanning (Kali → ubuntu-server-01)
- [ ] tcpdump packet capture during real traffic
- [ ] Write Bind9 zone file for `lab.local` domain
