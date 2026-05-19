# Linux Filesystem Navigation

**Category:** Linux / Administration  
**Applies to:** Any Linux system  
**WorldSkills Topic:** Section 1 — Linux Navigation & Filesystem

---

## Core Commands

```bash
pwd          # Print Working Directory — shows your current location
ls           # List directory contents
ls -l        # Long format (permissions, owner, size, date)
ls -la       # Long format including hidden files (starting with .)
ls -lh       # Human-readable file sizes (KB, MB, GB)
cd /etc      # Change to /etc (absolute path)
cd logs      # Change to logs/ inside current directory (relative)
cd ..        # Move up one directory
cd ~         # Go to your home directory
cd -         # Go back to previous directory
tree         # Visual directory tree (install: apt install tree)
tree /etc -L 2   # Tree of /etc, max 2 levels deep
find / -name "sshd_config" 2>/dev/null   # Find file by name
find /etc -type f -name "*.conf"         # Find all .conf files in /etc
locate sshd_config   # Fast search (uses database — run updatedb first)
```

---

## Absolute vs Relative Paths

### Absolute Path
Starts from the root `/`. Always works regardless of where you are.

```bash
cat /etc/ssh/sshd_config
cd /var/log
```

### Relative Path
Based on your current location. Changes meaning based on `pwd`.

```bash
# If you are in /etc:
cd ssh           # Goes to /etc/ssh
cat ../hosts     # Goes up one level, reads /etc/hosts

# If you are in /home/user:
cd ../root       # Goes to /root
```

**Rule of thumb:** Use absolute paths in scripts and configuration. Use relative paths when navigating interactively.

---

## The Linux Filesystem Explained

Linux uses a single unified tree starting at `/` (root). Everything — files, devices, processes — is a file or directory somewhere in this tree.

### Top-Level Directory Reference

| Directory | Full Name | What Lives Here | Examples |
|-----------|-----------|-----------------|---------|
| `/etc` | Et Cetera | System configuration files | `sshd_config`, `hosts`, `fstab`, `nginx/` |
| `/var` | Variable | Data that changes at runtime | Logs, mail, databases, web content |
| `/home` | Home | User home directories | `/home/username/` |
| `/usr` | Unix System Resources | Installed programs and libraries | `/usr/bin/`, `/usr/lib/` |
| `/bin` | Binaries | Essential user commands | `ls`, `cat`, `cp`, `ping` |
| `/sbin` | System Binaries | System admin commands | `ip`, `iptables`, `fdisk` |
| `/tmp` | Temporary | Temporary files, cleared on reboot | Scratch space |
| `/proc` | Processes | Live kernel and process information | `/proc/cpuinfo`, `/proc/meminfo` |
| `/sys` | System | Hardware and kernel interface | Device configuration |
| `/root` | Root Home | Home directory for the root user | Root's personal files |
| `/boot` | Boot | Bootloader and kernel files | `vmlinuz`, `grub/` |
| `/dev` | Devices | Device files | `/dev/sda` (disk), `/dev/null` |
| `/opt` | Optional | Third-party software | Manually installed applications |
| `/mnt` | Mount | Mount points for temporary filesystems | External drives, ISOs |

---

## Most Important Directories for Sysadmin Work

### `/etc` — Where You Configure Everything

```bash
/etc/ssh/sshd_config         # SSH server configuration
/etc/nginx/nginx.conf        # Nginx web server config
/etc/apache2/apache2.conf    # Apache web server config
/etc/bind/named.conf         # Bind9 DNS server config
/etc/dhcp/dhcpd.conf         # DHCP server config
/etc/hosts                   # Static hostname → IP mappings
/etc/resolv.conf             # DNS resolver configuration
/etc/fstab                   # Filesystem mount table
/etc/crontab                 # Scheduled tasks
/etc/network/interfaces      # Network configuration (Debian)
/etc/netplan/                # Network configuration (Ubuntu)
/etc/apt/sources.list        # APT repository list
/etc/passwd                  # User account database
/etc/shadow                  # Password hashes (root-only)
/etc/group                   # Group database
```

### `/var/log` — Where Problems Are Diagnosed

```bash
/var/log/syslog              # General system log
/var/log/auth.log            # Authentication events (SSH logins, sudo)
/var/log/apache2/access.log  # Apache HTTP access log
/var/log/apache2/error.log   # Apache error log
/var/log/nginx/access.log    # Nginx access log
/var/log/kern.log            # Kernel messages
/var/log/dpkg.log            # Package install/remove log
```

Checking logs is almost always the first step in troubleshooting:
```bash
tail -f /var/log/syslog           # Follow syslog live
tail -n 50 /var/log/auth.log      # Last 50 authentication events
grep "Failed" /var/log/auth.log   # Find failed login attempts
```

---

## Useful Navigation Tricks

```bash
# See your full path at any time
pwd

# Go home fast
cd ~
# or just
cd

# Go to previous directory (toggle)
cd -

# List only directories
ls -d */

# Find large files (>100MB) in current directory tree
find . -size +100M -type f

# Count files in a directory
ls | wc -l

# Check disk usage by directory
du -sh /var/log/*
```

---

## Key Takeaway

> In Linux, everything is a file — and everything has a path.  
> Knowing the filesystem structure means knowing exactly where to look when something breaks.  
> `/etc` is where you configure. `/var/log` is where you diagnose. `/proc` is where you observe.
