# `ip route` — Routing Table Deep Dive

**Category:** Networking / Commands  
**Applies to:** Linux (Proxmox host, Ubuntu Server VM, any Linux system)

---

## What It Does

`ip route` displays the system's **routing table** — the map Linux uses to decide where to send every packet.

Think of it as GPS for network traffic. Before any packet leaves your machine, Linux checks this table and asks:

- Is the destination local or remote?
- Which interface should I use?
- Where is the gateway?

---

## Running It

```bash
ip route
```

### Typical Output (Proxmox Host)

```
default via 192.168.1.1 dev vmbr0
192.168.1.0/24 dev vmbr0 proto kernel scope link src 192.168.1.50
```

---

## Breaking Down the Output

### Line 1 — Default Route

```
default via 192.168.1.1 dev vmbr0
```

| Part | Meaning |
|------|---------|
| `default` | "If you don't know where to send traffic…" |
| `via 192.168.1.1` | "…send it to this router/gateway" |
| `dev vmbr0` | "…using this network interface" |

This is the path to the internet and all external networks. The router (`192.168.1.1`) becomes the **next hop** for anything not local.

### Line 2 — Local Network Route

```
192.168.1.0/24 dev vmbr0 proto kernel scope link src 192.168.1.50
```

| Part | Meaning |
|------|---------|
| `192.168.1.0/24` | The local subnet (all `.1.x` addresses) |
| `dev vmbr0` | Reachable through this interface |
| `src 192.168.1.50` | Source IP used when sending from this host |

This tells Linux: "Devices in `192.168.1.x` are directly reachable — do NOT send to the router."

---

## How Linux Decides (Route Selection)

Linux always chooses the **most specific route** first.

```
192.168.1.0/24    ← more specific (local)
default           ← least specific (catch-all)
```

**Example — Local traffic:**
```
Destination: 192.168.1.60 (your Ubuntu VM)
↓ Matches 192.168.1.0/24
↓ Send directly through vmbr0
↓ No router involved
```

**Example — Internet traffic:**
```
Destination: 8.8.8.8 (Google DNS)
↓ No local route matches
↓ Falls to default route
↓ Sent to 192.168.1.1 (router)
↓ Router forwards to internet
```

---

## Why `vmbr0` Appears Instead of `eth0`

On Proxmox, the physical NIC (`nic0`) is **enslaved to the bridge** (`vmbr0`). The bridge becomes the active interface. All traffic — for the host and for VMs — flows through `vmbr0`.

This is an important virtualization concept: the bridge replaces the physical NIC at the routing level.

---

## Troubleshooting with `ip route`

Most network failures in a homelab trace back to routing. Use this table:

| Problem | Likely Cause |
|---------|-------------|
| VM has no internet | Missing default route |
| Can ping LAN but not internet | Bad or missing gateway |
| SSH fails across networks | Wrong route for destination |
| VLANs can't communicate | Missing inter-VLAN routing |
| pfSense breaks connectivity | NAT or routing misconfiguration |

### Verify the exact path Linux will use:

```bash
ip route get 8.8.8.8
```

Example output:
```
8.8.8.8 via 192.168.1.1 dev vmbr0 src 192.168.1.50
```

This tells you exactly: which gateway, which interface, which source IP. Extremely useful for diagnosing routing issues.

---

## Related Commands

```bash
ip route           # Show routing table
ip route get X.X.X.X  # Show exact path to a destination
ip a               # Show interfaces and IP addresses
ping 8.8.8.8       # Test internet routing (bypasses DNS)
ping google.com    # Test DNS + routing
```

---

## Broader Importance

Routing is foundational to:

- **CCNA** — core exam topic
- **pfSense** — all traffic flows through routing decisions
- **VLANs** — require inter-VLAN routing to communicate
- **Firewalls** — rules act on routed traffic
- **VPNs** — add routes for tunnel traffic
- **Pentesting** — pivoting between networks requires understanding routes
- **SOC analysis** — anomalous routes can indicate compromise
- **Kubernetes / Cloud** — overlay networks built on routing principles

---

## Key Takeaway

> The routing table is the nervous system of your network.  
> When traffic doesn't go where you expect — check the routing table first.
