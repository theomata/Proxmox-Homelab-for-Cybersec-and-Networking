# `apt` — Package Management in Debian/Ubuntu Linux

**Category:** Linux / Package Management  
**Applies to:** Debian, Ubuntu, Proxmox VE (Debian-based)

---

## The Big Picture

In Linux, software is distributed as **packages** — pre-compiled bundles of code and configuration. The package manager (`apt`) handles:

- Downloading packages from **repositories** (remote software libraries)
- Resolving and installing **dependencies** automatically
- Upgrading installed packages
- Removing software cleanly

Understanding package management is a core sysadmin skill. In enterprise environments, misconfigured or outdated packages are a significant attack vector.

---

## Key Concepts

### Package
Installable software unit. Format: `.deb` on Debian/Ubuntu systems.  
Examples: `nmap`, `docker`, `vim`, `openssh-server`

### Repository (Repo)
A remote server storing packages. Configured in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/`.

### Dependency
Other packages required for a program to work. `apt` resolves these automatically.

Example: `Wireshark` depends on networking libraries and GUI libraries. Install Wireshark, and `apt` installs all dependencies silently.

### Library
Reusable code shared across multiple packages. Prevents duplication.

Example: `libssl` (encryption library) — used by SSH, curl, wget, and hundreds of other programs.

### Relationship
```
Package
└── depends on → Dependencies
                 └── includes → Libraries
```

---

## Core Commands

```bash
# Update package index (sync with repos — does NOT install anything)
sudo apt update

# Upgrade all installed packages (safe upgrade, no removals)
sudo apt upgrade

# Full upgrade — handles kernel updates, dependency changes, replacements
sudo apt full-upgrade

# Install a package
sudo apt install nmap

# Install multiple packages
sudo apt install -y htop curl wget git vim tcpdump nmap

# Remove a package (keep config files)
sudo apt remove nmap

# Remove a package + config files
sudo apt purge nmap

# Remove unused dependencies
sudo apt autoremove -y

# Search for a package
apt search nmap

# Show package info
apt show nmap
```

---

## `apt update` vs `apt upgrade` vs `apt full-upgrade`

This distinction matters deeply in production:

| Command | What It Does |
|---------|-------------|
| `apt update` | Downloads the latest **package index** only. Nothing is installed. |
| `apt upgrade` | Upgrades packages — but will **not** remove anything or change dependencies. Safe. |
| `apt full-upgrade` | Upgrades everything, **including** removing old packages and replacing dependencies. Required for major kernel updates. |

### Why `full-upgrade` Matters on Proxmox

On a Proxmox host, running only `apt upgrade` is sometimes insufficient. Kernel updates on Proxmox may require:

- Removing old kernel packages
- Replacing dependencies
- Upgrading boot components (GRUB, initramfs)

Using `apt upgrade` alone can leave the system in a partially updated state. `apt full-upgrade` handles the complete update cycle.

**You are not just "updating Linux." You are:**
- Managing repositories
- Managing dependencies
- Maintaining a virtualization host
- Performing enterprise infrastructure operations

---

## Real-World Security Importance

Unpatched packages are one of the most common attack vectors. Routine `apt full-upgrade` is a security control, not just maintenance.

**Workflow for Proxmox or any server:**

```bash
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y
```

Run this regularly. In production environments, this is often automated via cron or configuration management tools (Ansible, Puppet).

---

## Repository Management

```bash
# View configured repos
cat /etc/apt/sources.list

# View additional repo files
ls /etc/apt/sources.list.d/

# Proxmox-specific: free community repo (no subscription required)
# Add: deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```

---

## Installed Packages — Day 1 Lab Reference

```bash
sudo apt install -y \
  net-tools \    # Legacy networking (ifconfig, netstat)
  htop \         # Interactive process monitor
  curl \         # HTTP/API requests and file transfer
  wget \         # File downloading
  git \          # Version control
  vim \          # Terminal text editor
  unzip \        # Archive extraction
  tcpdump \      # Live packet capture
  nmap           # Network scanning
```

These are foundational cybersecurity and sysadmin tools present on nearly every professional Linux system.

---

## Key Takeaway

> `apt` is how you build and maintain a Linux system.  
> `apt full-upgrade` is not optional on a Proxmox host.  
> Understanding packages, repos, and dependencies is real infrastructure engineering — not just running commands.
