# ubuntu-server-01 — VM Documentation

**Category:** Virtualization / VMs  
**VM Name:** ubuntu-server-01  
**Status:** ✅ Running

---

## Specifications

| Field | Value |
|-------|-------|
| **OS** | Ubuntu Server LTS |
| **IP Address** | 192.168.1.60 |
| **Hostname** | ubuntu-server-01 |
| **Network Adapter** | VirtIO → vmbr0 |
| **SSH** | Enabled (port 22) |
| **Proxmox Host** | 192.168.1.50 |

---

## Purpose

This VM serves as the primary Linux server for:
- Linux administration practice
- Service deployment and management
- Network tool experimentation
- Future: log forwarding to SIEM (Wazuh/Splunk)
- Future: target/victim machine for attack-defense labs

---

## Deployment Process

### 1. Upload ISO
- Downloaded Ubuntu Server LTS ISO
- Uploaded to Proxmox `local` storage (ISOs section)
- Storage path: `/var/lib/vz/template/iso/`

### 2. Create VM in Proxmox
- New VM → assigned name `ubuntu-server-01`
- OS Type: Linux
- CD/DVD: Ubuntu Server ISO
- Network: VirtIO adapter → vmbr0
- Storage: local-lvm (VM disk)

### 3. Ubuntu Installation
- Language and locale configured
- Hostname: `ubuntu-server-01`
- User account created
- **OpenSSH Server** enabled during install
- Automatic security updates: enabled

### 4. Post-Install Configuration

```bash
# Update system
sudo apt update && sudo apt full-upgrade -y

# Install tools
sudo apt install -y \
  net-tools htop curl wget git vim unzip tcpdump nmap

# Verify SSH is running
systemctl status ssh

# Check listening ports
sudo ss -tulnp

# Check network
ip a
ip route
```

---

## Installed Software

| Package | Purpose |
|---------|---------|
| `openssh-server` | Remote SSH access |
| `net-tools` | Legacy networking (`ifconfig`, `netstat`) |
| `htop` | Process and resource monitor |
| `curl` | HTTP/API testing |
| `wget` | File downloading |
| `git` | Version control |
| `vim` | Terminal text editor |
| `tcpdump` | Packet capture |
| `nmap` | Network scanning |

---

## Network Configuration

```bash
# Current network state
ip a
# Shows: ens18 → 192.168.1.60/24

ip route
# default via 192.168.1.1 dev ens18
# 192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.60
```

---

## Open Ports (Baseline — Day 1)

```bash
sudo ss -tulnp
```

| Port | Protocol | Service | Binding |
|------|---------|---------|---------|
| 22 | TCP | SSH (sshd) | 0.0.0.0, [::] |
| 53 | UDP | DNS resolver | 127.0.0.53 |
| 323 | UDP | NTP (chrony) | 127.0.0.1 |

> **Security note:** Port 22 is exposed to all interfaces. In this lab context this is acceptable. In production: restrict via firewall, enforce SSH key auth, disable password login.

---

## SSH Access

From the ASUS TUF A15 workstation:

```bash
ssh username@192.168.1.60
```

First connection will prompt to accept the host key fingerprint — verify it matches the server's key before accepting.

---

## Snapshot Strategy

Snapshots taken at key milestones:
- `clean-install` — immediately after OS install, before any changes
- `tools-installed` — after foundational tools installed
- `pre-lab-X` — before each new lab or experiment

This allows rollback if configurations are broken during cybersecurity exercises.

---

## Future Plans

- [ ] Configure static IP via Netplan (prevent DHCP address changes)
- [ ] Harden SSH (key-only auth, disable root login)
- [ ] Install UFW firewall and configure rules
- [ ] Set up as Wazuh agent (forward logs to SIEM)
- [ ] Use as target machine for Kali scanning exercises
- [ ] Install Docker for containerized service experiments
