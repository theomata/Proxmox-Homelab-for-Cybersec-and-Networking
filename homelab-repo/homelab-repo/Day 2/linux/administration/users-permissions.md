# Users, Groups & Permissions

**Category:** Linux / Administration  
**Applies to:** Any Linux system  
**WorldSkills Topic:** Section 3 — Users & Permissions

---

## Identity Commands

```bash
whoami                  # Your current username
id                      # Your UID, GID, and all group memberships
id username             # Same info for a specific user
groups                  # Groups you belong to
sudo whoami             # Returns "root" — confirms sudo works
```

Example output of `id`:
```
uid=1000(labuser) gid=1000(labuser) groups=1000(labuser),4(adm),27(sudo)
```

| Field | Meaning |
|-------|---------|
| `uid=1000` | User ID — unique number for this user |
| `gid=1000` | Primary Group ID |
| `groups=...` | All groups this user belongs to |
| `sudo` | This user can run commands as root |

---

## Managing Users

```bash
# Create a new user (interactive — sets password, home dir, etc.)
sudo adduser labuser

# Create user non-interactively
sudo useradd -m -s /bin/bash labuser

# Set or change password
sudo passwd labuser

# Delete user (keep home directory)
sudo userdel labuser

# Delete user AND home directory
sudo userdel -r labuser

# Modify user — add to group
sudo usermod -aG sudo labuser      # Add to sudo group
sudo usermod -aG labgroup labuser  # Add to custom group

# Switch to another user
su - labuser

# View all users
cat /etc/passwd
```

---

## Managing Groups

```bash
# Create a group
sudo groupadd labgroup

# Delete a group
sudo groupdel labgroup

# View all groups
cat /etc/group

# View groups a user belongs to
groups labuser
```

---

## Understanding Permissions

Every file and directory has three permission sets: **owner**, **group**, **others**.

```
-rwxr-xr--  1  labuser  labgroup  1234  Jan 1  file.sh
^└──┬──┘└──┬──┘└──┬──┘
|  owner  group  others
|
file type: - = file, d = directory, l = symlink, b = block device
```

### Permission Breakdown

| Symbol | Value | Meaning on a File | Meaning on a Directory |
|--------|-------|-------------------|------------------------|
| `r` | 4 | Can read file contents | Can list directory contents |
| `w` | 2 | Can modify file | Can create/delete files inside |
| `x` | 1 | Can execute file | Can enter (cd into) directory |
| `-` | 0 | No permission | No permission |

### Numeric (Octal) Mode

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

Common permission patterns:

| Octal | Symbolic | Use Case |
|-------|----------|---------|
| `777` | `rwxrwxrwx` | Everyone full access (avoid — dangerous) |
| `755` | `rwxr-xr-x` | Standard for scripts and directories |
| `644` | `rw-r--r--` | Standard for regular files |
| `600` | `rw-------` | SSH private keys, sensitive files |
| `700` | `rwx------` | Private scripts or directories |
| `400` | `r--------` | Read-only, owner only |

---

## `chmod` — Change Permissions

```bash
# Numeric (recommended for precision)
chmod 755 script.sh         # rwxr-xr-x
chmod 600 ~/.ssh/id_rsa     # rw------- (required for SSH keys)
chmod 644 /etc/hosts        # rw-r--r--

# Symbolic (more readable for small changes)
chmod +x script.sh          # Add execute for everyone
chmod u+x script.sh         # Add execute for owner only
chmod o-r file.txt          # Remove read from others
chmod g+w directory/        # Add write for group

# Recursive (apply to directory and all contents)
chmod -R 755 /var/www/html/
```

---

## `chown` — Change Ownership

```bash
# Change owner
sudo chown labuser file.txt

# Change owner AND group
sudo chown labuser:labgroup file.txt

# Change group only
sudo chown :labgroup file.txt

# Recursive
sudo chown -R www-data:www-data /var/www/html/
```

---

## `sudo` — Run as Root

```bash
# Run a single command as root
sudo apt update

# Open a root shell session
sudo -i
# or
sudo su -

# Run as a specific user
sudo -u labuser command

# Check what sudo permissions your account has
sudo -l
```

### The sudoers File

Sudo permissions are configured in `/etc/sudoers`. Edit it safely with:
```bash
sudo visudo    # Validates syntax before saving — never edit directly
```

---

## Real-World Security Scenarios

### SSH Key Permissions (Critical)
SSH will refuse to use a private key if permissions are too open:
```bash
# Correct permissions for SSH keys
chmod 700 ~/.ssh/              # Directory: owner access only
chmod 600 ~/.ssh/id_rsa        # Private key: owner read/write only
chmod 644 ~/.ssh/id_rsa.pub    # Public key: readable by all
chmod 600 ~/.ssh/authorized_keys

# If permissions are wrong, SSH gives this error:
# WARNING: UNPROTECTED PRIVATE KEY FILE!
# Permissions 0644 are too open.
```

### Web Server Files
Apache/Nginx needs to read web files:
```bash
# Web content owned by www-data (web server user)
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

---

## Key Takeaway

> Permissions are not just a Linux concept — they are a security boundary.  
> Wrong permissions on SSH keys, config files, or web directories cause real failures.  
> Understanding `rwx` and octal notation is mandatory for any sysadmin or security professional.
