# DHCP Server — isc-dhcp-server Setup

**Category:** Linux / Services / Networking  
**Applies to:** Ubuntu Server  
**WorldSkills Topic:** Section 6 — First Infrastructure Lab

---

## What Is DHCP?

DHCP (Dynamic Host Configuration Protocol) automatically assigns network configuration to clients:

- IP address
- Subnet mask
- Default gateway
- DNS server(s)
- Lease duration

Without DHCP, every device on a network needs a manually configured static IP. DHCP automates this at scale.

---

## How DHCP Works (DORA Process)

```
Client                          DHCP Server
  │                                  │
  │──── DISCOVER ──────────────────→ │  "Anyone there? I need an IP."
  │                                  │
  │ ←─── OFFER ──────────────────── │  "Here, take 192.168.1.100"
  │                                  │
  │──── REQUEST ───────────────────→ │  "Yes, I'll take that IP please."
  │                                  │
  │ ←─── ACK ────────────────────── │  "Confirmed. Lease is yours."
  │                                  │
```

This four-step process (DORA) happens over UDP broadcast (port 67 server, port 68 client).

---

## Installation

```bash
sudo apt update
sudo apt install -y isc-dhcp-server
```

After install, the service will fail to start — expected, because it has no configuration yet.

---

## Configuration

### Step 1 — Specify the Interface

```bash
sudo nano /etc/default/isc-dhcp-server
```

```
# The interface(s) the DHCP server should listen on
INTERFACESv4="ens18"
```

Replace `ens18` with your actual interface name (check with `ip a`).

### Step 2 — Configure the DHCP Pool

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

```
# Global options
default-lease-time 600;          # Default lease: 10 minutes
max-lease-time 7200;             # Maximum lease: 2 hours
authoritative;                   # This server is the authority for its subnets

# DNS domain and servers to give to clients
option domain-name "lab.local";
option domain-name-servers 192.168.1.60;   # Your Bind9 DNS server

# Subnet declaration
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;     # IP pool for clients
    option routers 192.168.1.1;            # Default gateway
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
}

# Static reservation (give a specific VM always the same IP)
host ubuntu-server-01 {
    hardware ethernet AA:BB:CC:DD:EE:FF;   # VM's MAC address
    fixed-address 192.168.1.60;
}
```

### Finding a VM's MAC Address

```bash
ip a | grep "link/ether"
# or
ip link show ens18
```

---

## Start and Enable

```bash
# Start the service
sudo systemctl start isc-dhcp-server

# Check status
systemctl status isc-dhcp-server

# Enable at boot
sudo systemctl enable isc-dhcp-server

# Verify it's listening
sudo ss -tulnp | grep :67
```

---

## Verify DHCP Is Working

### From the DHCP Server

```bash
# View active leases (who has what IP)
cat /var/lib/dhcp/dhcpd.leases

# Watch DHCP traffic live
sudo tcpdump -i ens18 port 67 or port 68 -n
```

### From a Client VM

```bash
# Release current IP
sudo dhclient -r ens18

# Request a new IP from DHCP
sudo dhclient ens18

# Verify new IP
ip a
```

Or set interface to DHCP in Netplan:

```yaml
network:
  ethernets:
    ens18:
      dhcp4: true
  version: 2
```

```bash
sudo netplan apply
```

---

## Troubleshooting DHCP

```bash
# Check service status
systemctl status isc-dhcp-server

# Check logs
journalctl -u isc-dhcp-server -n 30
tail -f /var/log/syslog | grep dhcp

# Config syntax check
dhcpd -t -cf /etc/dhcp/dhcpd.conf

# Is it listening on port 67?
sudo ss -tulnp | grep :67

# Common issues:
# Interface name wrong in /etc/default/isc-dhcp-server
# Subnet range overlaps with static IPs
# MAC address wrong in static reservation
# Firewall blocking port 67/68
```

### Allow DHCP Through Firewall

```bash
sudo ufw allow 67/udp
sudo ufw allow 68/udp
```

---

## DHCP in the Homelab Context

In a homelab with pfSense (planned for later), you'll eventually run DHCP from pfSense rather than a Linux VM. But understanding Linux DHCP first means:

- You know what the DORA process looks like at the packet level
- You understand lease files and reservations
- You can troubleshoot pfSense DHCP issues by understanding the underlying protocol
- You can run DHCP on isolated lab subnets that pfSense doesn't serve

---

## Key Takeaway

> DHCP is the service that hands out network identities.  
> Understanding DORA, lease files, and reservations is foundational for managing any network.  
> In WorldSkills competitions, DHCP is almost always tested — know `dhcpd.conf` cold.
