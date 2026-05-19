# `systemctl` & systemd — Service Management

**Category:** Linux / Services  
**Applies to:** Any modern Linux system (Debian, Ubuntu, Proxmox)

---

## What Is systemd?

`systemd` is the **init system** and service manager for modern Linux. It is the first process started by the kernel (PID 1) and is responsible for:

- Starting and stopping services
- Managing the boot sequence
- Handling service dependencies
- Logging (via `journald`)
- Managing timers, sockets, devices, and mounts

Understanding systemd is foundational — it controls everything from SSH to web servers to SIEM agents.

---

## Core Command: `systemctl`

`systemctl` is the CLI tool for interacting with systemd.

### Essential Commands

```bash
# View status of a service
systemctl status ssh

# Start a service
systemctl start ssh

# Stop a service
systemctl stop ssh

# Restart a service
systemctl restart ssh

# Reload config without full restart
systemctl reload ssh

# Enable at boot
systemctl enable ssh

# Disable at boot
systemctl disable ssh

# Check if enabled
systemctl is-enabled ssh

# List all running services
systemctl list-units --type=service --state=running
```

---

## Reading `systemctl status` Output

```bash
systemctl status ssh
```

```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-01-01 10:00:00 UTC; 2h ago
   Main PID: 1531 (sshd)
      Tasks: 1 (limit: 4915)
     Memory: 5.2M
        CPU: 45ms
     CGroup: /system.slice/ssh.service
             └─1531 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

| Field | Meaning |
|-------|---------|
| `Loaded` | Service unit file found and loaded |
| `enabled` | Will start automatically at boot |
| `Active: active (running)` | Service is currently running |
| `Main PID` | Process ID of the service |
| `Memory` | Current RAM usage |

---

## Listing Running Services

```bash
systemctl list-units --type=service --state=running
```

### Column Reference

| Column | Meaning |
|--------|---------|
| `UNIT` | Service name (e.g., `ssh.service`) |
| `LOAD` | Whether unit file was loaded successfully |
| `ACTIVE` | High-level state (`active`, `inactive`, `failed`) |
| `SUB` | Detailed state (`running`, `exited`, `dead`) |
| `DESCRIPTION` | Human-readable description |

### Important Services to Know

| Service | Purpose |
|---------|---------|
| `ssh.service` | Remote SSH access |
| `cron.service` | Scheduled task execution |
| `rsyslog.service` | System logging |
| `systemd-networkd.service` | Network interface management |
| `chrony.service` | NTP time synchronization |
| `ufw.service` | Uncomplicated Firewall |
| `docker.service` | Docker container engine |

---

## Why This Matters for Cybersecurity

Services are attack surface. Every running service:

1. Listens on a port (potential entry point)
2. Runs as a user (potential privilege escalation target)
3. Reads and writes files (potential persistence mechanism)

**Security audit workflow:**

```bash
# What services are running?
systemctl list-units --type=service --state=running

# Cross-reference with listening ports
sudo ss -tulnp

# Disable anything you don't need
sudo systemctl stop unnecessary.service
sudo systemctl disable unnecessary.service
```

**Attacker persistence technique:** Malware often installs itself as a systemd service so it survives reboots. Reviewing running services is part of incident response.

---

## Viewing Service Logs

systemd captures all service output via `journald`:

```bash
# View logs for SSH
journalctl -u ssh

# Follow live logs
journalctl -u ssh -f

# Logs since last boot
journalctl -u ssh -b

# Logs with timestamps, last 50 lines
journalctl -u ssh -n 50 --no-pager
```

---

## Service File Location

Unit files (service definitions) live at:

```
/lib/systemd/system/     ← System-provided services
/etc/systemd/system/     ← Admin-created or overridden services
```

Viewing a service definition:
```bash
cat /lib/systemd/system/ssh.service
```

This shows how the service starts, what user it runs as, and its dependencies.

---

## Key Takeaway

> systemd controls the operational state of your entire Linux system.  
> `systemctl` is the command that controls systemd.  
> Knowing which services run, why they run, and what they expose is foundational to both **administration** and **security**.
