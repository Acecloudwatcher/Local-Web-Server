# 🌐 Local Web Server — KDE Plasma Konsole

> A step-by-step guide to spinning up a local web server on a Debian/Ubuntu-based Linux system using **Python** and **Nginx**, built entirely from the terminal.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Phase 1 — Python Quick Server](#phase-1--python-quick-server)
- [Phase 2 — Nginx Production Server](#phase-2--nginx-production-server)
- [Management Commands](#management-commands)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Overview

This project walks through two approaches to running a local web server from the terminal:

| Method | Best For | Port |
|--------|----------|------|
| Python `http.server` | Quick file serving, HTML testing | `8080` |
| Nginx | Multi-site hosting, logging, production-style config | `8081` |

---

## Prerequisites

- KDE Plasma on a **Debian/Ubuntu-based** distro (e.g. KDE Neon, Kubuntu)
- `python3` installed (comes pre-installed on most distros)
- `sudo` privileges

> **Not sure which distro you're on?**
> ```bash
> cat /etc/os-release
> ```

| Distro | Package Manager |
|--------|----------------|
| Ubuntu, Debian, KDE Neon | `apt` |
| Arch, Manjaro | `pacman` |
| Fedora | `dnf` |
| openSUSE | `zypper` |

---

## Phase 1 — Python Quick Server

Up and running in seconds. Great for serving static files or testing HTML locally.

### 1. Create your project folder

```bash
mkdir ~/my-webserver && cd ~/my-webserver
```

### 2. Create a sample HTML page

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
  <head><title>My Local Server</title></head>
  <body><h1>Hello from Konsole!</h1></body>
</html>
EOF
```

### 3. Start the server

```bash
python3 -m http.server 8080
```

### 4. Open in your browser

```
http://localhost:8080
```

Press `Ctrl+C` to stop the server.

---

## Phase 2 — Nginx Production Server

More control — supports custom routes, logging, and multiple virtual hosts.

### 1. Install Nginx

```bash
sudo apt install nginx -y
```

### 2. Create your site directory

```bash
sudo mkdir -p /var/www/mysite
sudo chown $USER:$USER /var/www/mysite
echo "<h1>My Nginx Site</h1>" > /var/www/mysite/index.html
```

### 3. Create the Nginx config

```bash
sudo tee /etc/nginx/sites-available/mysite > /dev/null << 'EOF'
server {
    listen 8081;
    server_name localhost;

    root /var/www/mysite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/mysite_access.log;
    error_log  /var/log/nginx/mysite_error.log;
}
EOF
```

### 4. Enable the site

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

### 5. Test and restart Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Expected output:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 6. Open in your browser

```
http://localhost:8081
```

---

## Management Commands

| Task | Command |
|------|---------|
| Check Nginx status | `sudo systemctl status nginx` |
| Watch access logs live | `sudo tail -f /var/log/nginx/mysite_access.log` |
| Reload config (no downtime) | `sudo systemctl reload nginx` |
| Stop server | `sudo systemctl stop nginx` |
| Start server | `sudo systemctl start nginx` |

---

## Troubleshooting

### ❌ `open() failed (2: No such file or directory)`

The symlink was created before the config file existed. Fix it with:

```bash
# Remove the broken symlink
sudo rm -f /etc/nginx/sites-enabled/mysite

# Recreate the config (see Phase 2, Step 3)
# Then re-enable:
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### ❌ `pacman: command not found`

You're on a Debian/Ubuntu system. Use `apt` instead of `pacman`:

```bash
sudo apt install nginx -y
```

### ❌ Port already in use

Check what's using the port:

```bash
sudo ss -tulpn | grep 8081
```

Then either kill the process or change the port in your Nginx config.

---

## Next Steps

- 🗂️ **Serve a React/Vue app** — point `root` to your `dist/` build folder
- 🌍 **Add a custom local domain** — edit `/etc/hosts` to map `mysite.local → 127.0.0.1`
- 🔒 **Enable local HTTPS** — use [`mkcert`](https://github.com/FiloSottile/mkcert) to generate a trusted SSL cert
- ⚙️ **Auto-start on boot** — `sudo systemctl enable nginx`

---

## License

MIT
