# Apache & Nginx — Web Server Setup

**Category:** Linux / Services  
**Applies to:** Ubuntu Server  
**WorldSkills Topic:** Section 6 — First Infrastructure Lab

---

## Apache vs Nginx — Quick Comparison

| Feature | Apache | Nginx |
|---------|--------|-------|
| Architecture | Process-based | Event-driven (async) |
| Performance | Good | Better under high load |
| Config style | `.htaccess` per-directory | Centralized config |
| Module system | Dynamic modules | Compiled modules |
| Default port | 80/443 | 80/443 |
| Use case | Traditional, PHP heavy | Modern, reverse proxy, static |

For the homelab and WorldSkills: **either works**. Apache is slightly more common in exam/competition scenarios.

---

## Apache — Installation & Configuration

### Install

```bash
sudo apt update
sudo apt install -y apache2
```

### Verify

```bash
# Service running?
systemctl status apache2

# Listening on port 80?
sudo ss -tulnp | grep :80

# Test locally
curl http://localhost

# Test from another VM
curl http://192.168.1.60
```

If Apache is running, you'll see the default Ubuntu Apache page HTML in the curl output. In a browser: navigate to `http://192.168.1.60`.

### Key Files & Directories

```
/etc/apache2/
├── apache2.conf          ← Main configuration file
├── ports.conf            ← Which ports Apache listens on
├── sites-available/      ← Virtual host config files (inactive)
│   └── 000-default.conf  ← Default site config
├── sites-enabled/        ← Symlinks to active sites
├── mods-available/       ← Available modules
└── mods-enabled/         ← Active modules (symlinks)

/var/www/html/            ← Default web root (put your files here)
/var/log/apache2/
├── access.log            ← HTTP request log
└── error.log             ← Error log (check this when things break)
```

### Basic Virtual Host (Hosting a Site)

```bash
sudo nano /etc/apache2/sites-available/lab.conf
```

```apache
<VirtualHost *:80>
    ServerName lab.local
    ServerAdmin admin@lab.local
    DocumentRoot /var/www/lab

    ErrorLog ${APACHE_LOG_DIR}/lab-error.log
    CustomLog ${APACHE_LOG_DIR}/lab-access.log combined
</VirtualHost>
```

```bash
# Create the web root
sudo mkdir -p /var/www/lab
echo "<h1>Lab Server - Day 2</h1>" | sudo tee /var/www/lab/index.html
sudo chown -R www-data:www-data /var/www/lab

# Enable the site
sudo a2ensite lab.conf

# Disable the default site (optional)
sudo a2dissite 000-default.conf

# Test config syntax
apache2ctl configtest

# Reload Apache (no downtime)
sudo systemctl reload apache2
```

### Troubleshooting Apache

```bash
# Config syntax check
apache2ctl configtest

# See error details
journalctl -u apache2 -n 30
tail -f /var/log/apache2/error.log

# Common issues:
# Port 80 already in use → ss -tulnp | grep :80
# Permission denied on files → chown -R www-data:www-data /var/www/
# Syntax error in config → configtest output shows the line
```

---

## Nginx — Installation & Configuration

### Install

```bash
sudo apt update
sudo apt install -y nginx
```

### Verify

```bash
systemctl status nginx
sudo ss -tulnp | grep :80
curl http://localhost
```

### Key Files & Directories

```
/etc/nginx/
├── nginx.conf              ← Main configuration
├── sites-available/        ← Site configs (inactive)
│   └── default             ← Default site
└── sites-enabled/          ← Active sites (symlinks)

/var/www/html/              ← Default web root
/var/log/nginx/
├── access.log
└── error.log
```

### Basic Server Block (Virtual Host)

```bash
sudo nano /etc/nginx/sites-available/lab
```

```nginx
server {
    listen 80;
    server_name lab.local 192.168.1.60;

    root /var/www/lab;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    error_log /var/log/nginx/lab-error.log;
    access_log /var/log/nginx/lab-access.log;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/lab /etc/nginx/sites-enabled/

# Test config
sudo nginx -t

# Reload
sudo systemctl reload nginx
```

---

## Testing from Another VM

From the Kali or second Ubuntu VM:

```bash
# Test HTTP response
curl http://192.168.1.60

# Test with verbose output (shows headers)
curl -v http://192.168.1.60

# Test specific page
curl http://192.168.1.60/index.html
```

Expected: HTML content of your web page returned in the terminal.

---

## WorldSkills Competition Notes

In competition scenarios, web server tasks often include:
- Installing and enabling a specific web server
- Hosting a file at a specific URL
- Configuring a virtual host for a domain name
- Setting correct permissions on web directories
- Verifying with curl from another machine

The pattern is always: **install → configure → test config → reload → verify**.

---

## Key Takeaway

> Apache and Nginx both serve HTTP. The configuration patterns are different but the workflow is identical:  
> install → configure → test syntax → reload → verify with curl.  
> Know where the config files are, where the logs are, and how to check syntax before reloading.
