---
name: devops-networking
description: Setup, troubleshoot, maintain infrastructure — server, Docker, CI/CD, networking, monitoring. Diagnosa dulu, fix kemudian, verify selalu.
version: 2.0.0
metadata:
  hermes:
    tags: [devops, networking, docker, ci-cd, server, deploy, linux, monitoring]
    category: infrastructure
    requires_toolsets: [terminal]
---

# DevOps & Networking

## KAPAN PAKAI

```
IF user minta server setup / troubleshoot → PAKAI
IF user minta Docker / container ops → PAKAI
IF user minta CI/CD pipeline → PAKAI
IF user minta networking (DNS, firewall, VPN, SSL) → PAKAI
IF user minta monitoring / alerting → PAKAI
IF user minta deployment automation → PAKAI
IF user minta frontend / coding → JANGAN PAKAI
```

---

## PROCEDURE (ikuti exact)

### Step 1: Diagnosa DULU (sebelum fix apapun)

```bash
uname -a                          # OS + kernel
df -h                             # disk usage
free -h                           # memory
ss -tlnp                          # listening ports
systemctl list-units --failed     # failed services
docker ps -a 2>/dev/null          # containers (kalau Docker ada)
```

### Step 2: Identify problem category

```
IF service down → cek logs: journalctl -u [service] -n 50
IF port gak bisa diakses → cek: ss -tlnp | grep [port], iptables -L
IF disk full → cek: du -sh /* | sort -rh | head -10
IF memory full → cek: ps aux --sort=-%mem | head -10
IF SSL expired → cek: openssl s_client -connect domain:443 2>/dev/null | openssl x509 -dates
IF DNS gak resolve → cek: dig [domain], cat /etc/resolv.conf
```

### Step 3: Fix + SELALU verify

```
RULE: Setiap fix HARUS diikuti verification command
RULE: Backup config SEBELUM edit
RULE: JANGAN edit config tanpa backup

FORMAT:
1. Backup: cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
2. Edit: [change]
3. Test: nginx -t
4. Reload: systemctl reload nginx
5. Verify: curl -I https://domain.com
```

---

## DECISION TREE: Per Topic

### Docker

```
IF "bikin Dockerfile":
  1. Multi-stage build (small final image)
  2. Non-root user
  3. .dockerignore (exclude node_modules, .git)
  4. HEALTHCHECK instruction
  5. Pin base image version (bukan :latest)

IF "container gak jalan":
  1. docker logs [container]
  2. docker inspect [container] | grep -i status
  3. cek port mapping: docker port [container]
  4. cek network: docker network ls

IF "disk full dari Docker":
  docker system prune -af --volumes  # WARNING: hapus semua unused
  docker image prune -af             # hapus images unused
```

### Nginx

```
IF "setup reverse proxy":
  server {
      listen 80;
      server_name domain.com;
      location / {
          proxy_pass http://127.0.0.1:3000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
      }
  }
  VERIFY: nginx -t && systemctl reload nginx && curl -I http://domain.com

IF "setup SSL":
  1. apt install certbot python3-certbot-nginx
  2. certbot --nginx -d domain.com
  3. VERIFY: curl -I https://domain.com
  4. Auto-renew: certbot renew --dry-run
```

### Firewall (UFW)

```
IF "setup firewall":
  ufw default deny incoming
  ufw default allow outgoing
  ufw allow 22/tcp      # SSH — SELALU allow SSH DULU
  ufw allow 80/tcp
  ufw allow 443/tcp
  ufw enable
  VERIFY: ufw status verbose

⚠️ ALWAYS allow SSH BEFORE enable firewall
⚠️ IF lock out → butuh console access dari provider
```

### SSH

```
IF "harden SSH":
  1. Backup: cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
  2. Edit:
     PermitRootLogin no
     PasswordAuthentication no
     PubkeyAuthentication yes
     MaxAuthTries 3
  3. Test: sshd -t
  4. Reload: systemctl reload sshd
  ⚠️ PASTIKAN public key udah di ~/.ssh/authorized_keys SEBELUM disable password
```

### CI/CD (GitHub Actions)

```
IF "bikin CI pipeline":
  name: CI
  on: [push, pull_request]
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with: { node-version: '20' }
        - run: npm ci
        - run: npm run lint
        - run: npm test
        - run: npm run build
```

### Monitoring

```
IF "setup monitoring":
  MINIMAL:
  1. Uptime check: curl endpoint setiap 1 menit
  2. Disk alert: df -h | awk '$5+0 > 80 {print}'
  3. Memory alert: free | awk '/Mem/ {if($3/$2*100 > 85) print "HIGH MEM"}'
  4. Log errors: journalctl -p err --since "1 hour ago"

  PROPER (recommended):
  - Prometheus + Grafana (metrics)
  - Loki (logs)
  - Alertmanager (notifications)
```

---

## OUTPUT TEMPLATE

```markdown
## [Action]: [What]

### Diagnosa
```bash
[commands yang dijalanin]
[output relevant]
```

### Fix Applied
```bash
[commands yang dijalanin untuk fix]
```

### Verification
```bash
[commands untuk verify fix works]
[output showing success]
```

### Status: ✅ Fixed / ⚠️ Partial / ❌ Gagal
[1 kalimat summary]
```

---

## CONTOH OUTPUT

```markdown
## Fix: Nginx 502 Bad Gateway

### Diagnosa
```bash
$ systemctl status nginx
● nginx.service - active (running)

$ curl -I http://myapp.com
HTTP/1.1 502 Bad Gateway

$ ss -tlnp | grep 3000
(nothing — app gak listen di port 3000)

$ systemctl status myapp
● myapp.service - inactive (dead) — FOUND: app mati
```

### Fix Applied
```bash
$ journalctl -u myapp -n 20
# Error: EADDRINUSE port 3000 → ada zombie process

$ fuser -k 3000/tcp
$ systemctl start myapp
```

### Verification
```bash
$ ss -tlnp | grep 3000
LISTEN 0 511 127.0.0.1:3000 — OK, app listening

$ curl -I http://myapp.com
HTTP/1.1 200 OK — ✅ Fixed
```

### Status: ✅ Fixed
App mati karena zombie process hold port 3000. Killed zombie, restart app.
```

---

## VERIFICATION

```
□ Diagnosa dilakuin sebelum fix?
□ Config di-backup sebelum edit?
□ Verification command dijalanin setelah fix?
□ Port gak exposed unnecessary?
□ Credentials gak hardcoded?
□ SSH tetap accessible setelah firewall change?

IF ada □ TIDAK → fix sebelum deliver
```
