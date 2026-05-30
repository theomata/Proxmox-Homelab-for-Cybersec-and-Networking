# Day 02 — Enterprise DNS Infrastructure with OPNsense and BIND9

## Objective
Build an enterprise-style DNS infrastructure using OPNsense, Debian 12, and BIND9.

## Network Architecture

```text
Internet
   |
OPNsense
   |
+---------------------------+
| INT 10.1.10.0/24          |
| DMZ 10.1.20.0/24          |
| CLT 10.1.40.0/24          |
+---------------------------+

INT-SRV01
10.1.10.10
BIND9 DNS
```

## OPNsense Interfaces

| Interface | IP |
|-----------|----|
| WAN | DHCP |
| INT | 10.1.10.1/24 |
| DMZ | 10.1.20.1/24 |
| CLT | 10.1.40.1/24 |

## Debian Server

Hostname: int-srv01

### Verification

```bash
ip a
ip route
ping 10.1.10.1
ping 8.8.8.8
ping google.com
hostnamectl
```

## Install BIND9

```bash
sudo apt update
sudo apt install bind9 bind9-utils dnsutils -y
```

## Verify Service

```bash
sudo systemctl status named
sudo ss -tulnp | grep named
```

Expected:
- active (running)
- TCP/UDP 53 listening

## named.conf.local

```conf
zone "worldskills.ph" {
    type master;
    file "/etc/bind/db.worldskills.ph";
};

zone "10.1.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.1.10";
};
```

## Forward Zone

File: /etc/bind/db.worldskills.ph

```dns
$TTL 604800

@ IN SOA ns1.worldskills.ph. admin.worldskills.ph. (
    2026052901
    604800
    86400
    2419200
    604800 )

@ IN NS ns1.worldskills.ph.

ns1 IN A 10.1.10.10
int-srv01 IN A 10.1.10.10

@ IN A 10.1.10.10
```

## Reverse Zone

File: /etc/bind/db.10.1.10

```dns
$TTL 604800

@ IN SOA ns1.worldskills.ph. admin.worldskills.ph. (
    2026052901
    604800
    86400
    2419200
    604800 )

@ IN NS ns1.worldskills.ph.

10 IN PTR int-srv01.worldskills.ph.
```

## Validation

```bash
sudo named-checkconf
sudo named-checkzone worldskills.ph /etc/bind/db.worldskills.ph
sudo named-checkzone 10.1.10.in-addr.arpa /etc/bind/db.10.1.10
```

## Reload Service

```bash
sudo systemctl reload named
```

## DNS Tests

### Forward

```bash
dig @10.1.10.10 ns1.worldskills.ph
dig @10.1.10.10 int-srv01.worldskills.ph
```

### Reverse

```bash
dig @10.1.10.10 -x 10.1.10.10
```

Result:

```text
10.1.10.10 -> int-srv01.worldskills.ph
```

## Troubleshooting

Problem:

```text
NXDOMAIN
```

Root Cause:

```conf
zone "10.1.10.-in-addr.arpa"
```

Correct:

```conf
zone "10.1.10.in-addr.arpa"
```

After correcting the typo and reloading BIND, reverse DNS worked successfully.

## Skills Demonstrated

- Linux Administration
- DNS Administration
- BIND9 Configuration
- IPv4 Networking
- IPv6 Networking
- Enterprise Troubleshooting
- OPNsense Administration
- Technical Documentation

## Outcome

Successfully deployed a working authoritative DNS server with forward and reverse lookup zones inside a segmented enterprise-style homelab.
