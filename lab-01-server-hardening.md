# Lab 01 — Server Hardening

Took a freshly created Ubuntu 24.04 VPS and secured it following industry best practices.

## Steps performed

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
PasswordAuthentication no

6. **Disabled direct root login over SSH** (`/etc/ssh/sshd_config`)
PermitRootLogin no
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

## Real Problems Faced & How I Solved Them

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
Host *

ServerAliveInterval 60

## Key Lessons Learned

- **Order matters:** update packages *before* hardening SSH, to avoid configuration-merge conflicts.
- **Always keep a fallback session open** before restarting SSH or changing firewall rules.
- **The provider's web Console and SSH are independent systems** — hardening SSH does not affect Console access.
- **`Destroy`, not `Power Off`**, is required to actually stop billing on a Droplet.
- **Swap is a disk-based safety net, not real RAM** — useful for small VPS instances, not a substitute for upgrading RAM.

- 
