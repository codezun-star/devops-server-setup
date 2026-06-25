# Lab 02 — Nginx, Domain & SSL Deployment

Deployed a real website from GitHub to the VPS, served it with Nginx, connected a real domain, and issued a valid SSL certificate.

## Steps performed

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

## Key Lessons Learned

- A real deliverable isn't just "make it work" — it includes giving the client independent access to their own server, so they're never locked out.
- DNS propagation should always be verified (`nslookup`) before attempting to issue an SSL certificate.
- Certbot's Nginx plugin handles both the certificate issuance and the Nginx config update automatically — no manual server block editing required.
