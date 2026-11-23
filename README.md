# Serving a Website on an Azure VM with Nginx

> Work in progress. Last modified: 23/11/2026

A practice project: hosting a small type-the-words browser game on an Ubuntu
VM, served by Nginx over HTTPS with a DuckDNS domain.

## What I built

- Single-file HTML mini-game (created with AI)
- Ubuntu VM on Azure (Standard B2ats v2, 2 vCPUs, 1 GiB RAM)
- Nginx web server with a self-signed certificate
- Free subdomain from DuckDNS
- Azure inbound rules for SSH, HTTP and HTTPS

## First attempt: WSL (local)

I installed Nginx on WSL Ubuntu, set the site root, configured the firewall
and verified the site worked on my computer. It never loaded on my phone,
even on the same network and with the firewall off, so I moved the project to
a cloud VM.

## Setup on Azure

### 1. Create the VM

Ubuntu image, size Standard B2ats v2. Azure applies default vCPU quotas to
new subscriptions, which blocked this B-series size, so I submitted a quota
increase request.

### 2. Open ports

Added inbound rules: 22 (SSH), 80 (HTTP), 443 (HTTPS).

### 3. Copy the site to the VM

```bash
scp -i ~/.ssh/<key>.pem index.html azureuser@<VM-IP>:~/projeto/gamelab
```

Then, on the VM, move the files to the web root:

```bash
sudo mkdir -p /var/www/mywebsite
sudo cp ~/projeto/gamelab/index.html /var/www/mywebsite/
```

### 4. Generate a self-signed certificate
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -subj "/CN=fugadigitada.duckdns.org" \
  -keyout /etc/ssl/private/mywebsite.key \
  -out /etc/ssl/certs/mywebsite.crt
```

### 5. Configure Nginx
Edit  `/etc/nginx/sites-available/default`:

-`root /var/www/mywebsite;`
-`server_name fugadigitada.duckdns.org;`
- Uncomment the 443 SSL server block and set:
  - `ssl_certificate /etc/ssl/certs/mywebsite.crt;`
  - `ssl_certificate /etc/ssl/private/mywebsite.key;`

  ### 6. Validate and reload

  ```bash
  sudo nginx -t && sudo systemctl  reload nginx
  ```

  Expect output: `syntax is ok` and `test is successful`.

  ### Testing
  From outside the VM:
  
  ```bash
  curl -kI https://fugadigitada.duck.dns.org
  ```

  Expect result:`HTTP/1.1 200 OK`. (`-k` is needed because the certificate
  is self-signed.) I also opened the site on my phone, where the browser shows
  an untrusted-certificate warning, as expected.
