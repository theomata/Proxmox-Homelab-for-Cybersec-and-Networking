# Bind9 — DNS Server Setup

**Category:** Linux / Services / Networking  
**Applies to:** Ubuntu Server  
**WorldSkills Topic:** Section 6 — First Infrastructure Lab

---

## What Is DNS?

DNS (Domain Name System) translates human-readable names into IP addresses.

```
User types:  lab.local
DNS answers: 192.168.1.60
Browser connects to 192.168.1.60
```

Without DNS, you'd type IP addresses for everything. DNS is the phone book of the internet — and your private lab network.

---

## DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| `A` | Hostname → IPv4 address | `server1.lab.local → 192.168.1.60` |
| `AAAA` | Hostname → IPv6 address | `server1.lab.local → ::1` |
| `PTR` | IP address → hostname (reverse) | `60.1.168.192.in-addr.arpa → server1.lab.local` |
| `CNAME` | Alias → another hostname | `www.lab.local → server1.lab.local` |
| `MX` | Mail server for a domain | `lab.local → mail.lab.local` |
| `NS` | Authoritative nameserver for zone | `lab.local → ns1.lab.local` |
| `SOA` | Start of Authority — zone metadata | TTL, serial, responsible admin |

---

## Installation

```bash
sudo apt update
sudo apt install -y bind9 bind9utils bind9-doc
```

### Verify Install

```bash
systemctl status named        # bind9 runs as "named"
sudo ss -tulnp | grep :53     # Should be listening on 53/UDP and 53/TCP
```

---

## Configuration Files

```
/etc/bind/
├── named.conf                 ← Main config (includes others)
├── named.conf.options         ← Global options (forwarders, recursion)
├── named.conf.local           ← Your zones go here
└── db.lab.local               ← Zone file you create (forward lookup)
```

---

## Step 1 — Configure Options

```bash
sudo nano /etc/bind/named.conf.options
```

```
options {
    directory "/var/cache/bind";

    // Forward unresolved queries to Google DNS
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    // Allow queries from local network only
    allow-query { localhost; 192.168.1.0/24; };

    // Allow recursive queries
    recursion yes;

    dnssec-validation auto;
    listen-on-v6 { any; };
};
```

---

## Step 2 — Declare Your Zone

```bash
sudo nano /etc/bind/named.conf.local
```

```
// Forward zone — name to IP
zone "lab.local" {
    type master;
    file "/etc/bind/db.lab.local";
};

// Reverse zone — IP to name
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
};
```

---

## Step 3 — Create the Zone File (Forward)

```bash
sudo nano /etc/bind/db.lab.local
```

```
$TTL    604800
@   IN  SOA     ns1.lab.local. admin.lab.local. (
                2025010101  ; Serial (YYYYMMDDnn — increment on changes)
                604800      ; Refresh
                86400       ; Retry
                2419200     ; Expire
                604800 )    ; Negative Cache TTL

; Name servers
@           IN  NS      ns1.lab.local.

; A Records
ns1         IN  A       192.168.1.60
server1     IN  A       192.168.1.60
proxmox     IN  A       192.168.1.50
kali        IN  A       192.168.1.61

; CNAME (aliases)
www         IN  CNAME   server1.lab.local.
```

---

## Step 4 — Create Reverse Zone File

```bash
sudo nano /etc/bind/db.192.168.1
```

```
$TTL    604800
@   IN  SOA     ns1.lab.local. admin.lab.local. (
                2025010101
                604800
                86400
                2419200
                604800 )

@       IN  NS      ns1.lab.local.

; PTR Records (last octet of IP → hostname)
50      IN  PTR     proxmox.lab.local.
60      IN  PTR     server1.lab.local.
61      IN  PTR     kali.lab.local.
```

---

## Step 5 — Check and Restart

```bash
# Check named.conf syntax
sudo named-checkconf

# Check zone file syntax
sudo named-checkzone lab.local /etc/bind/db.lab.local
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1

# Restart Bind9
sudo systemctl restart named

# Enable at boot
sudo systemctl enable named
```

---

## Step 6 — Test

```bash
# Test from the DNS server itself
dig @localhost server1.lab.local
dig @localhost proxmox.lab.local
nslookup server1.lab.local localhost

# Test from another VM (point it at your DNS server)
dig @192.168.1.60 server1.lab.local
nslookup server1.lab.local 192.168.1.60

# Test reverse lookup
dig @192.168.1.60 -x 192.168.1.60
```

Expected output for `dig @192.168.1.60 server1.lab.local`:
```
;; ANSWER SECTION:
server1.lab.local.   604800  IN  A  192.168.1.60
```

---

## Point a Client VM at Your DNS Server

On ubuntu-server-01 (or any VM), edit the DNS config:

```bash
# Ubuntu uses Netplan
sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    ens18:
      addresses: [192.168.1.60/24]
      nameservers:
        addresses: [192.168.1.60]    ← Your Bind9 server
      routes:
        - to: default
          via: 192.168.1.1
  version: 2
```

```bash
sudo netplan apply
ping server1.lab.local    # Should now resolve
```

---

## Troubleshooting Bind9

```bash
# Check status and recent errors
systemctl status named
journalctl -u named -n 30

# Config syntax error?
sudo named-checkconf

# Zone file error?
sudo named-checkzone lab.local /etc/bind/db.lab.local

# Is it listening?
sudo ss -tulnp | grep :53

# Is the firewall blocking DNS?
sudo ufw allow 53

# Capture DNS traffic to debug
sudo tcpdump -i ens18 port 53
```

---

## Key Takeaway

> DNS is the foundation of every networked service.  
> In WorldSkills and real infrastructure, misconfigured DNS breaks everything silently.  
> Always check: zone file syntax → named-checkconf → named-checkzone → test with dig.  
> `dig` gives you the real answer — not what your machine *thinks*, but what the DNS server *says*.
