# 🖥️ Homelab — Infrastructure, Networking & Cybersecurity

> Self-directed infrastructure engineering and cybersecurity learning lab.  
> Built from scratch. Documented with intent. Growing continuously.

---

## 📋 Overview

This repository is a structured knowledge base, technical learning journal, and infrastructure engineering portfolio documenting my hands-on journey through:

- **Virtualization** — Proxmox VE, hypervisors, VM lifecycle
- **Linux Administration** — services, networking, package management, system monitoring
- **Networking** — routing, bridges, VLANs, DNS, firewalls
- **Cybersecurity** — attack surface analysis, port auditing, packet capture, defense labs

Everything documented here reflects real configurations, real troubleshooting, and real understanding — not just commands, but **why** things work the way they do.

---

## 🏗️ Lab Hardware

| Device | Role | Specs |
|---|---|---|
| **Acer Aspire 5** | Proxmox Hypervisor (Headless) | Intel Core i5 11th Gen · 16GB DDR4 · 256GB SSD |
| **ASUS TUF A15** | Main Workstation / Client | Ryzen 7 7435HS · 8GB DDR5 · 2×512GB SSD |

---

## 🌐 Network Architecture

```
[ Internet ]
      │
[ Home Router — 192.168.1.1 ]
      │
      ├── [ Acer Aspire 5 — Proxmox Host ]
      │         IP: 192.168.1.50
      │         nic0 → vmbr0 (Linux Bridge)
      │              ├── ubuntu-server-01 (192.168.1.60)
      │              ├── kali-linux        [planned]
      │              ├── pfsense           [planned]
      │              └── windows-server    [planned]
      │
      └── [ ASUS TUF A15 — Workstation ]
                IP: 192.168.1.xx
                SSH client → manages all VMs
```

> `vmbr0` acts as a Layer 2 software switch, allowing VMs to appear as real LAN devices.

---

## 🗂️ Repository Structure

```
homelab-repo/
├── README.md                        ← You are here
├── proxmox/
│   ├── setup/                       ← Installation, config, web UI
│   ├── networking/                  ← Bridge setup, vmbr0, nic config
│   ├── storage/                     ← local vs local-lvm, ISOs, disks
│   └── snapshots/                   ← Snapshot workflows, recovery
├── networking/
│   ├── concepts/                    ← Routing tables, DNS, subnets, VLANs
│   ├── commands/                    ← ip a, ip route, ss, tcpdump, nmap
│   └── troubleshooting/             ← Methodologies and real fixes
├── linux/
│   ├── administration/              ← Users, permissions, system management
│   ├── services/                    ← systemd, systemctl, service management
│   └── package-management/          ← apt, repositories, dependencies
├── virtualization/
│   ├── vms/                         ← Per-VM documentation
│   └── concepts/                    ← Hypervisors, VirtIO, bridges
├── cybersecurity/
│   ├── concepts/                    ← Attack surface, ports, threat modeling
│   ├── attack-surface/              ← Port auditing, ss, nmap findings
│   └── tools/                       ← tcpdump, nmap, Wireshark, Kali
├── logs/                            ← Daily lab logs / progress journal
├── assets/                          ← Diagrams, screenshots
└── templates/                       ← Reusable markdown templates
```

---

## 📅 Progress Log

| Day | Date | Focus | Status |
|-----|------|-------|--------|
| Day 1 | 2026 | Proxmox install · Linux networking · First VM · SSH · Foundational tools | ✅ Complete |
| Day 2 | — | pfSense · VLAN segmentation · Firewall rules | 🔜 Planned |
| Day 3 | — | Kali Linux VM · nmap scanning · tcpdump analysis | 🔜 Planned |
| Day 4 | — | Metasploitable 2 · Vulnerability assessment | 🔜 Planned |
| Day 5 | — | Wazuh/Splunk SIEM · Log ingestion · Alerting | 🔜 Planned |
| Day 6 | — | Windows Server · Active Directory · Domain setup | 🔜 Planned |
| Day 7 | — | Attack-and-defense lab · Full scenario | 🔜 Planned |

---

## 🗺️ Roadmap

### Phase 1 — Foundation (Current)
- [x] Proxmox VE installation and configuration
- [x] Linux bridge networking (vmbr0)
- [x] Ubuntu Server VM deployment
- [x] SSH remote administration
- [x] Linux networking commands mastery
- [x] Foundational sysadmin tools installed
- [ ] Proxmox backup and snapshot strategy

### Phase 2 — Networking Depth
- [ ] pfSense/OPNsense firewall VM
- [ ] VLAN segmentation (management, lab, DMZ)
- [ ] Inter-VLAN routing
- [ ] DNS server (Pi-hole or BIND9)
- [ ] VPN configuration

### Phase 3 — Cybersecurity Lab
- [ ] Kali Linux VM
- [ ] Metasploitable 2 target VM
- [ ] nmap and Nessus scanning workflows
- [ ] tcpdump / Wireshark packet analysis
- [ ] Attack surface documentation

### Phase 4 — SOC/SIEM Infrastructure
- [ ] Wazuh or Splunk deployment
- [ ] Log forwarding from all VMs
- [ ] Alert rule creation
- [ ] Incident response runbooks

### Phase 5 — Enterprise Simulation
- [ ] Windows Server Evaluation VM
- [ ] Active Directory domain
- [ ] Group Policy Objects
- [ ] Domain-joined Linux clients
- [ ] AD attack and defense scenarios

---

## 🛠️ Tools & Technologies

**Virtualization:** Proxmox VE · KVM · LXC  
**Networking:** Linux bridges · VLANs · pfSense · ip route · ss · tcpdump · nmap  
**Linux:** Ubuntu Server · Debian · systemd · apt · vim · htop  
**Security:** Kali Linux · Metasploitable · Wazuh · Splunk · nmap  
**Windows:** Windows Server Evaluation · Active Directory  

---

## 📖 Documentation Philosophy

Every entry in this repository answers two questions:

1. **What did I do?** — the commands and configurations
2. **Why does it work?** — the concepts and reasoning behind them

This is not a cheat sheet. It is an engineering journal.

---

## 🎯 Certifications Target

- [ ] CompTIA Network+
- [ ] CompTIA Security+
- [ ] eJPT (eLearnSecurity Junior Penetration Tester)
- [ ] OSCP (long-term goal)

---

*Self-directed learning. Built in the Philippines. One lab session at a time.*
