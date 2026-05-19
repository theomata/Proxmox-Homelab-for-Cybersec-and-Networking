# `ss -tulnp` — Network Socket & Port Auditing

**Category:** Networking / Cybersecurity / Commands  
**Applies to:** Any Linux system

---

## What It Does

`ss` (socket statistics) is the modern replacement for `netstat`. It shows active network connections, listening services, and which processes own which ports.

The cybersecurity mindset when running this command:

> **"Which services are exposed on this machine right now?"**

This is port auditing — a foundational security skill.

---

## The Command

```bash
sudo ss -tulnp
```

### Flag Breakdown

| Flag | Means | Why It Matters |
|------|-------|----------------|
| `-t` | Show TCP connections | TCP is used by SSH, HTTP, most services |
| `-u` | Show UDP connections | UDP is used by DNS, NTP, some VPNs |
| `-l` | Show only listening ports | Services *waiting* for connections |
| `-n` | Numeric addresses/ports | Skips slow DNS lookups, shows raw numbers |
| `-p` | Show the process using the port | Identifies *what* owns each port |

---

## Reading the Output

```
Netid  State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
tcp    LISTEN  0       128     0.0.0.0:22           0.0.0.0:*          users:(("sshd",pid=1531,fd=3))
tcp    LISTEN  0       128     [::]:22              [::]:*             users:(("sshd",pid=1531,fd=4))
udp    UNCONN  0       0       127.0.0.53:53        0.0.0.0:*          users:(("systemd-resolved",pid=890))
udp    UNCONN  0       0       127.0.0.1:323        0.0.0.0:*          users:(("chronyd",pid=743))
```

### State Values

| State | Meaning |
|-------|---------|
| `LISTEN` | Service is waiting for incoming connections |
| `UNCONN` | UDP socket (UDP is connectionless — this is normal) |
| `ESTAB` | Established connection (active session) |

### Address Meanings

| Address | Meaning | Security Implication |
|---------|---------|----------------------|
| `127.0.0.1` | Localhost only | Not reachable externally — safe |
| `0.0.0.0` | All IPv4 interfaces | **Externally reachable** |
| `[::]` | All IPv6 interfaces | **Externally reachable** |
| `0.0.0.0:*` | Peer field — accepts from anywhere | Normal for listening services |

---

## Port Reference — Day 1 Lab Findings

| Port | Protocol | Service | Bound To | Risk |
|------|---------|---------|----------|------|
| `22` | TCP | SSH (sshd) | `0.0.0.0` | ⚠️ Exposed — secure it |
| `53` | UDP | DNS resolver (systemd-resolved) | `127.0.0.53` | ✅ Local only |
| `323` | UDP | NTP/Chrony (time sync) | `127.0.0.1` | ✅ Local only |

### SSH on Port 22

```
0.0.0.0:22   → SSH listening on ALL IPv4 interfaces
[::]:22      → SSH also listening on IPv6
```

This means: anyone on the LAN can attempt SSH connections. Acceptable in a lab — but in production you would:
- Restrict to specific IPs via firewall
- Disable password auth (use SSH keys only)
- Change default port (security through obscurity — minor benefit)

---

## Cybersecurity Significance

### Every Open Port = Potential Attack Surface

```
Port 22 (SSH)
 → Brute-force attacks
 → Credential stuffing
 → Exploit attempts against the SSH daemon version

Port 80/443 (HTTP/HTTPS) — if enabled
 → Web application attacks
 → Directory traversal
 → Injection attacks
```

**Principle of minimal services:** Every service you don't need is a port you should close. Fewer open ports = smaller attack surface.

### Malware Detection Use Case

If malware infects your system, it often opens a **reverse shell** or **command-and-control (C2)** listener. Example:

```
tcp  LISTEN  0  0  0.0.0.0:4444  0.0.0.0:*  users:(("bash",pid=9999))
```

A bash process listening on port 4444 is a massive red flag. `ss -tulnp` would catch it immediately — provided you know what your *normal* baseline looks like.

**This is why baselining matters.** Run `ss -tulnp` on a clean system. Save the output. Compare it after incidents.

---

## Security Audit Workflow

```bash
# 1. Capture current port state (baseline)
sudo ss -tulnp > baseline-ports.txt

# 2. After changes or incidents, compare
sudo ss -tulnp > current-ports.txt
diff baseline-ports.txt current-ports.txt

# 3. Investigate a specific port
sudo lsof -i :22

# 4. Check all active connections (not just listening)
sudo ss -tunap
```

---

## Related Commands

```bash
sudo ss -tulnp        # Listening ports + process owners
sudo ss -tunap        # All connections including established
sudo ss -tln          # Only listening TCP ports (fast)
sudo lsof -i :22      # Identify what's using port 22
nmap -sV localhost    # Scan your own machine for open ports
```

---

## Key Takeaway

> `ss -tulnp` answers the question every security engineer asks first:  
> **"What is listening on this machine, and should it be?"**

Run it on every new system. Run it after every configuration change. Run it when something feels wrong. Know your baseline.
