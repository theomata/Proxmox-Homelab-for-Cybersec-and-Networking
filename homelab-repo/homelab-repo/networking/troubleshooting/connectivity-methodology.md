# Connectivity Troubleshooting Methodology

**Category:** Networking / Troubleshooting  
**Applies to:** Any Linux system, homelab, enterprise network

---

## The Structured Approach

Never troubleshoot randomly. Use a layered methodology that isolates each component of the network stack.

```
Step 1: Local → Gateway
Step 2: Gateway → Internet (routing)
Step 3: Internet → DNS (name resolution)
```

Each step tests a different layer. When one step fails, you've identified the problem layer.

---

## The Three-Ping Test

```bash
# Step 1: Test local routing and gateway reachability
ping 192.168.1.1

# Step 2: Test internet routing (bypasses DNS entirely)
ping 8.8.8.8

# Step 3: Test DNS resolution
ping google.com
```

### Interpreting Results

| Step 1 | Step 2 | Step 3 | Diagnosis |
|--------|--------|--------|-----------|
| ✅ | ✅ | ✅ | Everything works |
| ❌ | ❌ | ❌ | Local network issue (IP, gateway, bridge) |
| ✅ | ❌ | ❌ | Routing issue (default route, ISP, gateway config) |
| ✅ | ✅ | ❌ | **DNS-only issue** (internet works, name resolution broken) |

---

## Why This Matters

Without this methodology, you might spend an hour "fixing" DNS when the real problem is a missing default route — or vice versa.

The three-ping test pinpoints the exact failure layer in under 30 seconds.

---

## Follow-Up Commands by Failure Layer

### Layer 1 Failure — Can't reach gateway (192.168.1.1)

```bash
# Check your IP address
ip a

# Check your routing table — is there a local route?
ip route

# Is the interface up?
ip link show

# Proxmox specific: is vmbr0 configured?
cat /etc/network/interfaces
```

**Common causes:**
- VM has no IP address (DHCP failed)
- Wrong gateway configured
- Network interface down
- vmbr0 misconfigured in Proxmox

---

### Layer 2 Failure — Can reach gateway but not internet

```bash
# Verify default route exists
ip route | grep default

# Check if router is forwarding (try an alternative gateway)
ping 1.1.1.1

# Check if it's a firewall issue on the router
traceroute 8.8.8.8
```

**Common causes:**
- Missing or incorrect default route
- Router not set up for NAT/forwarding
- ISP issue
- pfSense misconfiguration (when added)

---

### Layer 3 Failure — Internet works but DNS fails

```bash
# Test DNS directly
nslookup google.com
dig google.com

# Check which DNS server you're using
cat /etc/resolv.conf

# Test against a known-good DNS
nslookup google.com 8.8.8.8

# Check systemd-resolved status
systemctl status systemd-resolved
resolvectl status
```

**Common causes:**
- Wrong DNS server configured
- systemd-resolved not running
- DNS server unreachable
- `/etc/resolv.conf` misconfigured

---

## SSH Connectivity Troubleshooting

When SSH fails from workstation to VM:

```bash
# On the VM — is SSH running?
systemctl status ssh

# Is SSH listening on the right port?
sudo ss -tulnp | grep :22

# Is there a firewall blocking it?
sudo ufw status

# From workstation — verbose SSH connection
ssh -v user@192.168.1.60

# Test with ping first
ping 192.168.1.60
```

---

## Proxmox-Specific Troubleshooting

```bash
# On Proxmox host — check bridge
ip link show vmbr0
bridge link show

# Check VM is using correct bridge in Proxmox UI
# Network → vmbr0 → correct VLAN tag (none for untagged)

# Restart Proxmox networking (careful in production)
systemctl restart networking
```

---

## Key Principle

> Troubleshoot from the **inside out**.  
> Start at your machine → move outward toward the internet.  
> Isolate each layer before changing anything.  
> Change one thing at a time. Verify after each change.

This is how professional network engineers and sysadmins approach problems. Random changes without methodology make problems worse and harder to diagnose.
