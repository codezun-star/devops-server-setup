# Linux Server Administration & DevOps — Hands-On Practice

This repository documents my hands-on, practical work learning Linux server administration and DevOps fundamentals, with the goal of offering professional freelance sysadmin/DevOps services.

All exercises were performed on **real infrastructure** (DigitalOcean Droplets running Ubuntu 24.04 LTS) — not simulations or copy-pasted tutorials. Every command was tested, and every real issue that came up along the way was diagnosed and fixed independently.

> 📸 Supporting screenshots for this work are available in the `/screenshots` folder of this repository.

---

## Table of Contents

1. [Server Hardening](#server-hardening)
2. [Web Server, Domain & SSL Deployment](#web-server-domain--ssl-deployment)
3. [Real Problems Faced & How I Solved Them](#real-problems-faced--how-i-solved-them)
4. [Key Lessons Learned](#key-lessons-learned)
5. [Tech Stack](#tech-stack)

---

## Server Hardening

Took a freshly created Ubuntu 24.04 VPS and secured it following industry best practices.

### Steps performed

1. **Created a DigitalOcean Droplet**
   - Ubuntu 24.04 LTS, SSH key authentication (no password)

2. **Updated the system before any configuration**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo reboot
   ```

3. **Created a personal sudo user** (instead of working as root)
   ```bash
   adduser jose
   usermod -aG sudo jose
   ```

4. **Configured SSH key access for the new user**
   ```bash
   mkdir -p /home/jose/.ssh
   cp /root/.ssh/authorized_keys /home/jose/.ssh/authorized_keys
   chown -R jose:jose /home/jose/.ssh
   chmod 700 /home/jose/.ssh
   chmod 600 /home/jose/.ssh/authorized_keys
   ```

5. **Disabled password authentication** (`/etc/ssh/sshd_config`)
   ```
   PasswordAuthentication no
   ```

6. **Disabled direct root login over SSH** (`/etc/ssh/sshd_config`)
   ```
   PermitRootLogin no
   ```
   ```bash
   sudo systemctl restart ssh
   ```

7. **Configured the firewall (UFW)**
   ```bash
   sudo ufw allow OpenSSH
   sudo ufw enable
   sudo ufw status verbose
   ```

8. **Installed Fail2ban** (brute-force protection)
   ```bash
   sudo apt install fail2ban -y
   sudo systemctl status fail2ban
   ```

9. **Configured a swap file** (for low-RAM VPS instances)
   ```bash
   sudo fallocate -l 1G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```
   Made it persistent across reboots:
   ```bash
   sudo cp /etc/fstab /etc/fstab.bak
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   sudo mount -a
   ```

10. **Verified system time/timezone**
    ```bash
    timedatectl
    ```

---

## Web Server, Domain & SSL Deployment

Deployed a real website from GitHub to the VPS, served it with Nginx, connected a real domain, and issued a valid SSL certificate.

### Steps performed

1. **Configured Git locally and connected to GitHub via SSH**
   ```bash
   git config --global user.name "Jose"
   git config --global user.email "my_email@example.com"
   ssh -T git@github.com
   ```

2. **Created a site locally and pushed it to GitHub**
   ```bash
   git init
   git add index.html
   git commit -m "first commit"
   git branch -M main
   git remote add origin git@github.com:username/repo.git
   git push -u origin main
   ```

3. **Cloned the repository on the VPS**
   ```bash
   git clone https://github.com/username/repo.git
   ```

4. **Installed and configured Nginx**
   ```bash
   sudo apt install nginx -y
   sudo ufw allow 'Nginx Full'
   sudo systemctl status nginx
   ```

5. **Deployed the cloned site**
   ```bash
   sudo cp ~/repo/index.html /var/www/html/index.html
   ```

6. **Pointed a real domain to the server's IP**
   - Created `A` records (`@` and `www`) pointing to the Droplet's IP
   - Verified propagation with `nslookup`

7. **Updated Nginx server_name to match the domain**
   ```nginx
   server_name mydomain.com www.mydomain.com;
   ```

8. **Issued a real SSL certificate with Certbot (Let's Encrypt)**
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   sudo certbot --nginx -d mydomain.com -d www.mydomain.com
   ```
   Result: automatic HTTPS redirection and auto-renewal configured.

---

## Real Problems Faced & How I Solved Them

Documenting real troubleshooting, because this is what actually happens in production work — not a clean, linear tutorial.

### 1. Invalid SSH public key format when creating the Droplet
**Error:** `invalid public key: ssh: no key found; illegal base64 data`
**Cause:** the key got corrupted while copy-pasting through the browser.
**Fix:** copied the key directly to clipboard from WSL using `clip.exe` instead of manual selection, avoiding copy/paste corruption.

### 2. Permission denied (publickey) after configuring SSH access
**Cause:** extra whitespace was introduced into `authorized_keys` during manual copy-paste in the web console.
**Fix:** recreated the file cleanly with `echo "key" > authorized_keys` instead of manual editing, then re-verified permissions (`700` / `600`).

### 3. Same error on a different droplet, different root cause
**Cause:** this time the key was never copied at all — the command had been entered incomplete.
**Fix:** diagnosed independently by checking `cat ~/.ssh/authorized_keys` (empty/missing), then re-ran the copy command correctly.

### 4. DigitalOcean web Console failing with "All configured authentication methods failed"
**Cause:** the Console requires a system-level password for the OS user — unrelated to `PermitRootLogin` or `PasswordAuthentication`, since the Console doesn't use SSH at all.
**Fix:** set a password directly on the OS for the user that needs Console access. This does not weaken SSH security — SSH hardening still fully applies; the Console is a separate access channel with its own authentication.

### 5. SSH sessions disconnecting after inactivity
**Cause:** normal SSH timeout behavior.
**Fix:** added a client-side keep-alive setting in `~/.ssh/config`:
```
Host *
    ServerAliveInterval 60
```

---

## Key Lessons Learned

- **Order matters:** update packages *before* hardening SSH, to avoid configuration-merge conflicts.
- **Always keep a fallback session open** before restarting SSH or changing firewall rules.
- **The provider's web Console and SSH are independent systems** — hardening SSH does not affect Console access, which depends on OS-level passwords and account permissions instead.
- **`Destroy`, not `Power Off`**, is required to actually stop billing on a Droplet.
- **Swap is a disk-based safety net, not real RAM** — useful for small VPS instances, not a substitute for upgrading RAM when the workload genuinely needs more memory.
- A real deliverable isn't just "make it work" — it includes giving the client independent access to their own server, so they're never locked out.

---

## Tech Stack

`Ubuntu 24.04 LTS` · `Nginx` · `UFW` · `Fail2ban` · `Certbot / Let's Encrypt` · `Git & GitHub` · `DigitalOcean` · `DNS Management` · `Bash`

---

*This is a living document, updated as new work is completed.*
