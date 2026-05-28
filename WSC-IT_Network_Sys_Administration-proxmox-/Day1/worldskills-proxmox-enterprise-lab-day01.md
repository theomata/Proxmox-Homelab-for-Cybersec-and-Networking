# WorldSkills-Style Proxmox Enterprise Network Lab — Day 01

## Project Overview

This lab documents the first build session of a **WorldSkills-style enterprise network topology** using **Proxmox VE**, **OPNsense**, and **Debian 12** virtual machines.

The goal of this session was not to complete the full WorldSkills Module A environment immediately. Instead, the goal was to build the **network foundation** correctly before installing advanced services such as DNS, LDAP, Samba, DHCP/DDNS, Syslog, mail, reverse proxy, VPN, and high availability.

The main focus was:

- Creating isolated virtual network segments in Proxmox
- Deploying OPNsense as a firewall/router
- Configuring enterprise-style interfaces
- Enabling DHCP for testing
- Installing the first internal server VM
- Validating DHCP, routing, NAT, and internet connectivity
- Converting the internal server from DHCP to static addressing

This is the beginning of a larger homelab project intended to prepare for **IT Network Systems Administration / WorldSkills-style infrastructure tasks**.

---

## Target Topology for This Session

The simplified topology built during this session:

```text
Internet / Home LAN
        |
      vmbr0
        |
 [ OPNsense Firewall ]
        |
 ------------------------------------------------
 |                      |                       |
vmbr10                vmbr20                  vmbr40
 INT                   DMZ                     CLT
 |                      |                       |
int-srv01           future DMZ servers       future clients
```

The full WorldSkills-style topology will eventually include:

```text
WAN / Internet
     |
Firewall / Router / Proxy / VPN
     |
------------------------------------------------
|                     |                        |
INT                  DMZ                      CLT
|                     |                        |
int-srv01            mail                     int-client
DNS/LDAP/Samba       ha-prx01/02              DHCP client
DHCP/DDNS/Syslog     web01/web02
CA
```

---

## Main Lab Components

| Component | Role |
|---|---|
| Proxmox VE | Hypervisor / virtualization platform |
| OPNsense | Firewall, router, DHCP server, NAT gateway |
| Debian 12 | Internal server VM |
| vmbr0 | WAN / outside bridge |
| vmbr10 | INT / internal server bridge |
| vmbr20 | DMZ bridge |
| vmbr40 | CLT / client bridge |

---

## Important Networking Terms Learned

### Virtual Bridge / `vmbr`

A Proxmox `vmbr` is a **virtual switch**.

It connects VMs together at **Layer 2**.

Example:

```text
VM1 ----+
        |
VM2 ----+---- vmbr10
        |
VM3 ----+
```

A bridge connects devices in the same network, but it does **not** route traffic between different networks. Routing is done by the firewall/router, which in this lab is OPNsense.

---

### WAN

**WAN** means **Wide Area Network**.

In this lab, WAN represents the outside network, which is connected to the real home router / physical LAN through `vmbr0`.

---

### INT

**INT** means **Internal Network**.

This is the trusted infrastructure network where internal servers live.

In this lab:

```text
INT network = 10.1.10.0/24
INT gateway = 10.1.10.1
```

The internal server `int-srv01` is placed here.

---

### DMZ

**DMZ** means **Demilitarized Zone**.

It is a separated network for public-facing services such as:

- Web servers
- Mail servers
- Reverse proxies
- Public DNS services

The purpose of the DMZ is to isolate exposed services from the internal trusted network.

In this lab:

```text
DMZ network = 10.1.20.0/24
DMZ gateway = 10.1.20.1
```

---

### CLT

**CLT** means **Client Network**.

It represents end-user devices such as:

- Workstations
- Laptops
- Employee PCs

In this lab:

```text
CLT network = 10.1.40.0/24
CLT gateway = 10.1.40.1
```

---

### Gateway

A **gateway** is the router address that a device uses when sending traffic outside its own subnet.

Example:

```text
int-srv01 = 10.1.10.10/24
Gateway   = 10.1.10.1
```

If `int-srv01` wants to reach `8.8.8.8`, it sends the packet to `10.1.10.1`, which is the OPNsense firewall.

---

### DHCP

**DHCP** means **Dynamic Host Configuration Protocol**.

It automatically provides clients with:

- IP address
- Subnet mask
- Default gateway
- DNS server

During this lab, OPNsense successfully gave the Debian VM an IP address through DHCP.

---

### Static IP Addressing

A **static IP address** is manually assigned and does not change.

Infrastructure servers usually use static IPs because other systems depend on them.

Example:

```text
int-srv01 = 10.1.10.10/24
```

Servers should usually be static. Clients usually use DHCP.

---

### NAT

**NAT** means **Network Address Translation**.

It allows private internal IP addresses such as `10.1.10.10` to access the internet through the firewall.

In this lab, internet connectivity from the internal server to `8.8.8.8` proved that NAT was working.

---

### Dual Stack

**Dual stack** means the network supports both:

- IPv4
- IPv6

This lab configured both IPv4 and IPv6 gateway addresses on OPNsense interfaces.

---

## Proxmox Network Bridges Created

The following Proxmox Linux bridges were used:

| Bridge | Purpose | Notes |
|---|---|---|
| `vmbr0` | WAN / outside network | Connected to real LAN/home router |
| `vmbr10` | INT network | Internal infrastructure network |
| `vmbr20` | DMZ network | Public-facing server network |
| `vmbr40` | CLT network | Client/workstation network |

The isolated lab bridges (`vmbr10`, `vmbr20`, `vmbr40`) do not need physical ports. They act as internal virtual switches.

---

## OPNsense Firewall VM Deployment

### VM Name

```text
fw-opnsense
```

### Purpose

The OPNsense VM acts as:

- Firewall
- Router
- DHCP server
- NAT gateway
- Future VPN gateway
- Future traffic policy enforcement point

---

## OPNsense Network Interfaces

Four virtual NICs were attached to the firewall VM.

| OPNsense NIC | Proxmox Bridge | Role |
|---|---|---|
| `vtnet0` | `vmbr0` | WAN |
| `vtnet1` | `vmbr10` | LAN / INT |
| `vtnet2` | `vmbr20` | OPT1 / DMZ |
| `vtnet3` | `vmbr40` | OPT2 / CLT |

---

## OPNsense Interface Assignment

Inside the OPNsense console, the interfaces were assigned as:

| OPNsense Interface | Virtual NIC | Network Role |
|---|---|---|
| WAN | `vtnet0` | Outside / home LAN / internet side |
| LAN | `vtnet1` | INT network |
| OPT1 | `vtnet2` | DMZ network |
| OPT2 | `vtnet3` | CLT network |

This step is important because OPNsense must know which virtual NIC belongs to which network zone.

---

## OPNsense Interface IP Configuration

### WAN Interface

The WAN interface received an address from the real home router using DHCP.

Observed WAN IPv4:

```text
192.168.1.132/24
```

Observed WAN IPv6:

```text
2001:4455:24a:d00:be24:11ff:fe9a:dafa/64
```

This confirmed that the firewall's WAN side was connected to the real network.

---

### INT / LAN Interface

The INT interface was configured as:

```text
IPv4: 10.1.10.1/24
IPv6: 2001:db8:1001:10::1/64
```

This became the default gateway for the internal network.

### INT DHCP Pool

DHCP was enabled temporarily for testing with this range:

```text
10.1.10.100 - 10.1.10.199
```

This allowed the first Debian VM to automatically obtain an IP address for validation.

---

### DMZ / OPT1 Interface

The DMZ interface was configured as:

```text
IPv4: 10.1.20.1/24
IPv6: 2001:db8:1001:20::1/64
```

This became the gateway for future DMZ servers.

### Mistake and Correction

A typo was made during configuration:

```text
Wrong: 10.1.21.1/24
Correct: 10.1.20.1/24
```

This was corrected by re-running:

```text
2) Set interface IP address
```

and selecting OPT1 again.

Lesson learned:

> One wrong subnet number creates an entirely different network. `10.1.20.0/24` and `10.1.21.0/24` are not the same network.

DHCP was disabled on the DMZ interface because DMZ servers should use static addresses.

---

### CLT / OPT2 Interface

The client network interface was configured as:

```text
IPv4: 10.1.40.1/24
IPv6: 2001:db8:1001:40::1/64
```

This became the gateway for future client machines.

DHCP was enabled for CLT during the firewall configuration workflow because client networks usually use DHCP.

Recommended DHCP range for CLT:

```text
10.1.40.100 - 10.1.40.199
```

---

## Final OPNsense Interface Summary

At the end of firewall configuration, the OPNsense console showed:

```text
LAN  (vtnet1) -> v4: 10.1.10.1/24
                 v6: 2001:db8:1001:10::1/64

OPT1 (vtnet2) -> v4: 10.1.20.1/24
                 v6: 2001:db8:1001:20::1/64

OPT2 (vtnet3) -> v4: 10.1.40.1/24
                 v6: 2001:db8:1001:40::1/64

WAN  (vtnet0) -> v4/DHCP4: 192.168.1.132/24
                 v6/DHCP6: 2001:4455:24a:d00:be24:11ff:fe9a:dafa/64
```

This confirmed that the firewall had become a proper multi-interface Layer 3 device.

---

## Secure Web GUI Configuration

During interface configuration, OPNsense asked whether to change the web GUI from HTTPS to HTTP.

The answer was:

```text
No
```

Reason:

- HTTPS encrypts management traffic
- Firewall administration should not be done through plaintext HTTP
- Management credentials should be protected

OPNsense also asked whether to generate a self-signed GUI certificate.

The answer was:

```text
Yes
```

A self-signed certificate provides encryption but is not trusted by public certificate authorities. This is acceptable for an internal lab firewall.

When asked whether to restore web GUI access defaults, the answer was:

```text
No
```

because the current secure configuration should be preserved.

---

## Debian Internal Server VM Installation

### VM Name

```text
int-srv01
```

### Purpose

This VM will eventually become the internal infrastructure server for:

- DNS / BIND9
- LDAP
- Samba
- Certificate Authority
- DHCP/DDNS
- Syslog

For this session, it was used to test the new INT network.

---

## Debian VM Network Placement

The VM was connected to:

```text
vmbr10
```

This placed the server inside the INT network.

---

## Debian User Account

During installation, the module instructions were reviewed. The WorldSkills Module A default credentials are:

```text
Username: root / user
Password: Skill39@PH
```

For the custom Proxmox practice lab, a normal local user was used instead of creating a user named `root`, because `root` is already the Linux superuser account.

Recommended lab account:

```text
Username: admin or theo
Password: Skill39@PH
```

In the actual installed system, the prompt showed:

```text
theo@int-srv01
```

which confirms the user account and hostname were successfully created.

---

## Debian Disk Partitioning

The installer used:

```text
Guided - use entire disk
```

This was selected because the lab focuses on networking and server services, not custom disk partitioning.

The installer created:

```text
sda1 -> ext4
sda5 -> swap
```

### ext4

`ext4` is a common Linux filesystem used to store files and directories.

### swap

Swap is disk space used as virtual memory when RAM is under pressure.

---

## Debian Package Manager

When Debian asked to scan extra installation media, the answer was:

```text
No
```

Reason:

- No additional Debian DVD/ISO was being used
- Packages can be installed later through APT

### APT

APT stands for **Advanced Package Tool**.

Examples:

```bash
sudo apt update
sudo apt install bind9
sudo apt install samba
sudo apt install ldap-utils
```

---

## Initial DHCP Validation on `int-srv01`

After installation, the first goal was to verify DHCP on the INT network.

### Command Used

```bash
ip a
```

### Observed Address

```text
inet 10.1.10.103/24
```

The output also showed:

```text
scope global dynamic
```

This means the address was assigned by DHCP.

---

## Ping Test to INT Gateway

### Command Used

```bash
ping 10.1.10.1
```

### Result

```text
12 packets transmitted, 12 received, 0% packet loss
```

This proved that `int-srv01` could communicate with the OPNsense INT gateway.

Validated path:

```text
int-srv01
   |
vmbr10
   |
OPNsense LAN / INT interface
10.1.10.1
```

---

## Routing Table Validation

### Command Used

```bash
ip route
```

### Output

```text
default via 10.1.10.1 dev ens18 proto dhcp src 10.1.10.103 metric 100
10.1.10.0/24 dev ens18 proto kernel scope link src 10.1.10.103 metric 100
```

### Explanation

The default route:

```text
default via 10.1.10.1
```

means that unknown destinations are sent to OPNsense.

The connected route:

```text
10.1.10.0/24 dev ens18
```

means the server knows the INT network is directly connected to its interface.

---

## Internet Connectivity Test

### Command Used

```bash
ping -c 4 8.8.8.8
```

### Result

```text
4 packets transmitted, 4 received, 0% packet loss
```

Observed latency:

```text
23 ms - 27 ms
```

This proved that:

- The Debian VM had a working IP
- The VM had a default gateway
- OPNsense routing worked
- NAT worked
- WAN connectivity worked
- Internet access from INT was successful

Validated path:

```text
int-srv01
   |
vmbr10
   |
OPNsense
   |
WAN / vmbr0
   |
Home router
   |
Internet
   |
8.8.8.8
```

---

## Converting `int-srv01` from DHCP to Static IP

After DHCP and connectivity were validated, `int-srv01` was converted from a DHCP client to a static infrastructure server.

Initial DHCP address:

```text
10.1.10.103/24
```

Target static address:

```text
10.1.10.10/24
```

Target IPv6 address:

```text
2001:db8:1001:10::10/64
```

Gateway:

```text
10.1.10.1
```

IPv6 gateway:

```text
2001:db8:1001:10::1
```

---

## NetworkManager Discovery

The system was checked to see how networking was managed.

### Commands Used

```bash
ls /etc/NetworkManager/
nmcli connection show
```

### Output Summary

NetworkManager was present, and the active connection was:

```text
Wired connection 1
```

Interface:

```text
ens18
```

---

## Static IP Configuration with `nmcli`

### Static IPv4 Configuration

```bash
sudo nmcli connection modify "Wired connection 1" \
ipv4.method manual \
ipv4.addresses 10.1.10.10/24 \
ipv4.gateway 10.1.10.1 \
ipv4.dns 10.1.10.1
```

### Static IPv6 Configuration

```bash
sudo nmcli connection modify "Wired connection 1" \
ipv6.method manual \
ipv6.addresses 2001:db8:1001:10::10/64 \
ipv6.gateway 2001:db8:1001:10::1
```

### Apply Configuration

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

---

## Static IP Verification

### Command Used

```bash
ip a
```

### Verified Output

```text
inet 10.1.10.10/24 brd 10.1.10.255 scope global noprefixroute ens18
inet6 2001:db8:1001:10::10/64 scope global noprefixroute
```

This confirmed that `int-srv01` now matches the WorldSkills-style addressing plan.

---

## Final State at End of Session

### OPNsense Firewall

| Interface | Role | IPv4 | IPv6 |
|---|---|---|---|
| WAN / `vtnet0` | Outside network | `192.168.1.132/24` | ISP-provided IPv6 |
| LAN / `vtnet1` | INT | `10.1.10.1/24` | `2001:db8:1001:10::1/64` |
| OPT1 / `vtnet2` | DMZ | `10.1.20.1/24` | `2001:db8:1001:20::1/64` |
| OPT2 / `vtnet3` | CLT | `10.1.40.1/24` | `2001:db8:1001:40::1/64` |

### Internal Server

| Hostname | Interface | IPv4 | IPv6 | Network |
|---|---|---|---|---|
| `int-srv01` | `ens18` | `10.1.10.10/24` | `2001:db8:1001:10::10/64` | INT |

---

## Commands Used During This Session

### Check IP Addresses

```bash
ip a
```

### Check Routing Table

```bash
ip route
```

### Ping Gateway

```bash
ping 10.1.10.1
```

### Ping Internet

```bash
ping -c 4 8.8.8.8
```

### Check NetworkManager Directory

```bash
ls /etc/NetworkManager/
```

### Show NetworkManager Connections

```bash
nmcli connection show
```

### Show Specific Connection Details

```bash
nmcli connection show "Wired connection 1"
```

### Configure Static IPv4

```bash
sudo nmcli connection modify "Wired connection 1" \
ipv4.method manual \
ipv4.addresses 10.1.10.10/24 \
ipv4.gateway 10.1.10.1 \
ipv4.dns 10.1.10.1
```

### Configure Static IPv6

```bash
sudo nmcli connection modify "Wired connection 1" \
ipv6.method manual \
ipv6.addresses 2001:db8:1001:10::10/64 \
ipv6.gateway 2001:db8:1001:10::1
```

### Restart Network Connection

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

---

## Troubleshooting Notes

### Issue: Wrong DMZ IPv4 Address Entered

A typo was entered during OPT1 configuration:

```text
10.1.21.1/24
```

Correct value:

```text
10.1.20.1/24
```

### Fix

The OPT1 interface was reconfigured through the OPNsense console:

```text
2) Set interface IP address
```

Then OPT1 was selected and the correct address was entered.

### Lesson

A one-digit mistake in an IP address can place an interface in a completely different subnet.

---

## Key Lessons Learned

### 1. Build the network foundation first

Before installing services such as DNS, LDAP, Samba, or mail, the network must be validated.

Correct order:

```text
Bridge setup
Firewall interfaces
IP addressing
DHCP
Gateway testing
Internet/NAT testing
Static server addressing
Then services
```

---

### 2. Bridges are virtual switches

`vmbr10`, `vmbr20`, and `vmbr40` act like separate switches for different network zones.

---

### 3. OPNsense is the router between zones

Traffic between INT, DMZ, CLT, and WAN must pass through OPNsense.

---

### 4. Clients can use DHCP, but servers should use static IPs

The Debian VM initially used DHCP for testing, then was converted to a static IP to behave like a real infrastructure server.

---

### 5. NAT was confirmed through internet ping

Successful ping to `8.8.8.8` from the internal server proved that NAT and routing worked.

---

### 6. Always validate using simple commands

Useful first-check commands:

```bash
ip a
ip route
ping <gateway>
ping <internet-ip>
```

These are faster and more reliable than guessing.

---

## Skills Practiced

- Proxmox virtual networking
- Linux bridge concepts
- OPNsense firewall deployment
- Interface assignment
- IPv4 addressing
- IPv6 addressing
- DHCP validation
- Static IP configuration
- NetworkManager / `nmcli`
- Routing table analysis
- Default gateway concepts
- NAT validation
- Basic troubleshooting
- Enterprise network segmentation

---

## Next Planned Steps

The next lab session should continue from this foundation.

Recommended next steps:

1. Verify static routing after the IP change
2. Confirm `int-srv01` can still reach:
   - `10.1.10.1`
   - `8.8.8.8`
3. Configure hostname and `/etc/hosts` cleanly
4. Install and configure BIND9 DNS
5. Create the `int.worldskills.ph` forward zone
6. Create reverse DNS zones
7. Add DNS records for INT, DMZ, CLT, and WAN hosts
8. Create a CLT client VM on `vmbr40`
9. Test inter-network routing
10. Start preparing for DHCP/DDNS, LDAP, Samba, and Syslog

---

## Current Working Topology

```text
                           Internet
                               |
                       Home Router / LAN
                               |
                            vmbr0
                               |
                        OPNsense WAN
                        192.168.1.132
                               |
        -------------------------------------------------
        |                       |                       |
   OPNsense LAN            OPNsense OPT1           OPNsense OPT2
   10.1.10.1               10.1.20.1               10.1.40.1
   vmbr10                  vmbr20                  vmbr40
        |                       |                       |
   int-srv01              future DMZ              future clients
   10.1.10.10             servers                 10.1.40.x
```

---

## Reflection

This session transformed the lab from a theoretical topology into a working virtual enterprise network. The most important achievement was not installing advanced services yet, but proving the foundation:

- The firewall works
- The internal bridge works
- DHCP works
- Routing works
- NAT works
- Static addressing works

This is the correct way to build infrastructure: validate each layer before adding more complexity.

