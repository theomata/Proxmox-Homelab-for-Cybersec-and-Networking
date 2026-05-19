# Attack Surface Analysis — Concepts & Fundamentals

**Category:** Cybersecurity / Concepts  
**Introduced:** Day 1

---

## What Is an Attack Surface?

The **attack surface** is the sum total of all points where an attacker could attempt to enter or extract data from a system.

Every:
- Open network port
- Running service
- User account
- Installed application
- Open file share

...is part of the attack surface.

**Core principle:** Minimize the attack surface. Every service you don't need is a risk you don't have to take.

---

## Attack Surface in the Context of Ports and Services

### Every Listening Port = Potential Attack Entry Point

```
Port 22 (SSH)
 → Brute force attacks
 → Credential stuffing
 → Exploit of specific SSH daemon version
 → Weak key exchange algorithm exploitation

Port 80/443 (HTTP/HTTPS)
 → SQL injection
 → XSS, CSRF
 → Path traversal
 → API abuse

Port 445 (SMB — Windows)
 → EternalBlue (MS17-010)
 → Ransomware propagation
 → Lateral movement
```

A port doesn't have to be misconfigured to be dangerous. The service behind it may have vulnerabilities regardless of configuration.

---

## Auditing Your Attack Surface

### Step 1: Enumerate Listening Ports

```bash
sudo ss -tulnp
```

Document every port. For each one, ask:
- What service is this?
- Does it need to be exposed?
- Is it bound to localhost or all interfaces?
- What version of the software is running?

### Step 2: Identify Exposed vs. Internal Services

| Binding | Meaning | Action |
|---------|---------|--------|
| `127.0.0.1:port` | Localhost only | Generally safe |
| `0.0.0.0:port` | All IPv4 interfaces | **Assess exposure** |
| `[::]:port` | All IPv6 interfaces | **Assess exposure** |

### Step 3: Disable Unnecessary Services

```bash
# Stop a service
sudo systemctl stop unnecessary.service

# Disable at boot
sudo systemctl disable unnecessary.service

# Verify it's gone
sudo ss -tulnp | grep port
```

### Step 4: Restrict Necessary Services

For SSH (must remain open):
```bash
# /etc/ssh/sshd_config — harden it

# Disable root login
PermitRootLogin no

# Disable password auth (keys only)
PasswordAuthentication no

# Restrict to specific users
AllowUsers yourusername

# Change default port (minor deterrence)
Port 2222
```

---

## Baseline and Monitor

**Baselining** = recording the normal state of your system so you can detect deviations.

```bash
# Create a baseline of open ports
sudo ss -tulnp > /root/baseline-ports-$(date +%Y%m%d).txt

# Compare after incidents or changes
diff /root/baseline-ports-20250101.txt <(sudo ss -tulnp)
```

If a new port appears that wasn't in your baseline — investigate immediately.

---

## Real-World Example: Malware Detection

Malware often opens a reverse shell back to the attacker's server, or listens for commands on an unusual port.

```
tcp  LISTEN  0  0  0.0.0.0:4444  0.0.0.0:*  users:(("bash",pid=9999))
```

This would appear in `ss -tulnp` output. Red flags:
- `bash` owning a listening port (unusual)
- Port 4444 (common Metasploit reverse shell port)
- PID that doesn't correspond to a known service

This is exactly the kind of anomaly you'd catch if you know your baseline.

---

## Attack Surface in the Homelab

Current homelab attack surface (Day 1):

| Asset | Exposed Services | Risk Level |
|-------|-----------------|------------|
| Proxmox Host (192.168.1.50) | Web UI (:8006), SSH (:22) | Medium — admin interface |
| ubuntu-server-01 (192.168.1.60) | SSH (:22) | Low — single service |

As the lab grows, this table expands. Tracking it deliberately is a security habit worth building from day one.

---

## Key Takeaway

> Attack surface analysis is not a one-time task.  
> It is a continuous practice: enumerate, assess, minimize, baseline, and monitor.  
> Every service you add to your lab should be a deliberate decision with a documented reason.
