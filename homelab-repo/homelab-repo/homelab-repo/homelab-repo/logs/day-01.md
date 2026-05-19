# Day 1 — Lab Journal

**Date:** Day 1  
**Focus:** Proxmox installation · Linux networking · First VM · SSH · Foundational tools  
**Status:** ✅ Complete

---

## 🎯 Session Goals

- Install and configure Proxmox VE on the Acer Aspire 5
- Understand hypervisor concepts and bridge networking
- Deploy first Ubuntu Server VM
- Establish SSH remote access from the workstation
- Learn core Linux networking and system commands

---

## ✅ What Was Accomplished

### 1. Proxmox VE Installation
Installed Proxmox VE on the Acer Aspire 5 (headless server). The machine now runs as a Type 1 bare-metal hypervisor — no underlying OS, Proxmox itself owns the hardware.

- Web UI accessible at: `https://192.168.1.50:8006`
- Physical interface: `nic0`
- Linux bridge: `vmbr0` (acts as a Layer 2 software switch)
- LAN connectivity: ✅
- Internet connectivity: ✅
- SSH to Proxmox host: ✅

### 2. Networking Configuration
Configured the bridge architecture that allows VMs to communicate on the LAN:

```
Physical NIC (nic0)
    └── vmbr0 (Linux Bridge)
            ├── Proxmox Host
            └── VMs (appear as real LAN devices)
```

Key insight: `vmbr0` behaves like a switch. VMs plugged into it get their own MAC addresses and LAN presence — identical to physical machines on the network.

### 3. Ubuntu Server VM — ubuntu-server-01
- ISO: Ubuntu Server LTS (uploaded to Proxmox local storage)
- Network: VirtIO adapter attached to vmbr0
- Hostname configured
- User account created
- OpenSSH Server installed during setup
- VM IP: `192.168.1.60`

### 4. SSH Remote Administration
Successfully connected from the ASUS TUF A15 workstation to the Ubuntu Server VM:

```bash
ssh user@192.168.1.60
```

This is the first real remote Linux administration workflow. All subsequent VM management happens over SSH — no monitor, no keyboard attached.

### 5. Foundational Tools Installed

```bash
sudo apt install -y \
  net-tools \
  htop \
  curl \
  wget \
  git \
  vim \
  unzip \
  tcpdump \
  nmap
```

| Tool | Purpose |
|------|---------|
| `net-tools` | Legacy networking tools (`ifconfig`, `netstat`) |
| `htop` | Interactive process and resource monitor |
| `curl` | HTTP/API testing and data transfer |
| `wget` | File downloading from URLs |
| `git` | Version control |
| `vim` | Terminal-based text editor |
| `tcpdump` | Live packet capture |
| `nmap` | Network scanning and host discovery |

---

## 📚 Concepts Learned

### Hypervisors
- **Type 1 (Bare-Metal):** Runs directly on hardware. No host OS. Examples: Proxmox, VMware ESXi, Hyper-V.
- **Type 2 (Hosted):** Runs as an application on top of an OS. Examples: VirtualBox, VMware Workstation.
- Proxmox = Type 1. It allocates CPU, RAM, and storage to VMs directly.

### Linux Bridge (vmbr0)
- A software Layer 2 switch inside the Proxmox host
- `nic0` is the physical uplink to the LAN
- `vmbr0` connects VMs to that physical network
- VMs get real MAC addresses and behave like physical devices

### Storage in Proxmox
- `local` — stores ISOs, backups, and VM templates
- `local-lvm` — stores actual VM disk images (thin-provisioned LVM volumes)

### Snapshots
- VM snapshots freeze the state of a VM at a point in time
- Allows safe experimentation — break things, roll back instantly
- Critical for cybersecurity labs where things get deliberately broken

---

## 🔍 Commands Practiced

See dedicated command reference pages:
- [`networking/commands/ip-route.md`](../../networking/commands/ip-route.md)
- [`networking/commands/ss-tulnp.md`](../../networking/commands/ss-tulnp.md)
- [`linux/services/systemctl.md`](../../linux/services/systemctl.md)
- [`linux/package-management/apt.md`](../../linux/package-management/apt.md)

---

## 🧠 Key Realizations

1. `apt full-upgrade` is not just "updating." It handles kernel updates, dependency replacement, and boot component upgrades — real enterprise Linux maintenance.
2. `vmbr0` is not just a config option. Understanding it means understanding how enterprise virtualization networking works.
3. SSH access to a headless server changes everything. This is how real servers are managed — no GUI, no monitor.
4. Every open port is an attack surface. Understanding `ss -tulnp` output is a security skill, not just a networking one.

---

## 🔧 Troubleshooting Performed

### Connectivity Verification Method
Used a structured methodology — not random guessing:

```
Step 1: ping 192.168.1.1     → Tests local routing to gateway
Step 2: ping 8.8.8.8         → Tests internet routing (bypasses DNS)
Step 3: ping google.com      → Tests DNS resolution
```

If Step 1 fails → local network issue (bridge, IP, gateway)  
If Step 2 fails → routing issue (default route, ISP)  
If Step 3 fails → DNS issue only (internet works but name resolution broken)

---

## 📌 Next Steps (Day 2)

- [ ] Set up pfSense VM for firewall and routing
- [ ] Configure VLAN segmentation
- [ ] Practice `tcpdump` packet capture
- [ ] Study firewall rules and NAT
- [ ] Document Proxmox backup strategy
