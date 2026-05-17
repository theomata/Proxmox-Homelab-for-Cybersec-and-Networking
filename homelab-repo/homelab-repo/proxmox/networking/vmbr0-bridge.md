# Proxmox Bridge Networking — vmbr0 Architecture

**Category:** Proxmox / Networking  
**Applies to:** Proxmox VE

---

## Overview

Proxmox uses **Linux bridge networking** to connect VMs to the physical network. Understanding this architecture is fundamental to understanding how your homelab communicates.

---

## The Architecture

```
Physical Network (LAN)
        │
        │  192.168.1.1 (Router/Gateway)
        │
[ Physical Switch / Router ]
        │
        │ Ethernet cable
        │
[ Acer Aspire 5 — Proxmox Host ]
        │
      nic0   ← Physical NIC (enslaved to bridge)
        │
      vmbr0  ← Linux Bridge (Layer 2 Software Switch)
       / \
      /   \
Proxmox  VMs
 Host   (ubuntu-server-01, kali, etc.)
```

---

## How It Works

### Physical NIC (nic0)
- The actual network card in the Acer Aspire 5
- Connected to the home router via ethernet
- **Does not have its own IP address** — it is enslaved to the bridge

### Linux Bridge (vmbr0)
- A software Layer 2 switch running inside Proxmox
- Holds the host IP address (`192.168.1.50`)
- Acts as the uplink for all VMs
- VMs attached to `vmbr0` appear as real devices on the LAN

### Why This Design
- VMs get real MAC addresses
- VMs get their own IPs from DHCP or static assignment
- VMs are reachable from anywhere on the LAN
- The Proxmox host itself also communicates through the bridge

---

## IP Configuration

| Device | Interface | IP Address |
|--------|-----------|------------|
| Proxmox Host | vmbr0 | 192.168.1.50 |
| ubuntu-server-01 | ens18 (VirtIO) | 192.168.1.60 |
| Router/Gateway | — | 192.168.1.1 |

---

## Proxmox Network Config File

```bash
cat /etc/network/interfaces
```

Typical output:
```
auto lo
iface lo inet loopback

iface eno1 inet manual    ← Physical NIC enslaved (no IP)

auto vmbr0
iface vmbr0 inet static   ← Bridge holds the IP
    address 192.168.1.50/24
    gateway 192.168.1.1
    bridge-ports eno1     ← Physical NIC attached to bridge
    bridge-stp off
    bridge-fd 0
```

---

## VirtIO Network Adapter

VMs in this lab use **VirtIO** network adapters rather than emulated hardware adapters. Reasons:

| Feature | Emulated (e1000) | VirtIO (paravirtual) |
|---------|-----------------|----------------------|
| Speed | Slower | Near-native performance |
| CPU overhead | Higher | Lower |
| Driver | Generic, always works | Requires VirtIO driver |
| Use case | Compatibility | Performance |

VirtIO is the correct choice for Linux VMs. Windows VMs require VirtIO drivers to be installed.

---

## Layer 2 vs Layer 3

The bridge operates at **Layer 2** (Data Link) — it forwards frames based on MAC addresses, not IP addresses. This is why:

- VMs get their own MAC addresses
- The bridge doesn't "route" — it switches
- IP routing still happens at Layer 3, handled by the OS

When you add pfSense/OPNsense later, pfSense will handle **Layer 3 routing** between VLANs. The bridge handles Layer 2 within each VLAN.

---

## Future: VLAN Segmentation

When VLANs are added, the architecture evolves:

```
vmbr0 (trunk bridge)
  ├── VLAN 10 — Management  (Proxmox host, admin access)
  ├── VLAN 20 — Lab         (VMs, attack/defense)
  ├── VLAN 30 — DMZ         (exposed services)
  └── VLAN 99 — IoT         (isolated devices)
```

Each VLAN is a separate broadcast domain. Traffic between VLANs requires a router (pfSense).

---

## Troubleshooting Bridge Issues

```bash
# View bridge status
ip link show vmbr0

# Show bridge ports
bridge link show

# Show MAC address table
bridge fdb show

# Test connectivity through bridge
ping 192.168.1.1     # Gateway reachable?
ping 192.168.1.60    # VM reachable?
```

---

## Key Takeaway

> `vmbr0` is not just a Proxmox configuration option.  
> It is a software Layer 2 switch that makes your VMs real citizens of your LAN.  
> Understanding this bridge is understanding how enterprise virtualization networking works.
