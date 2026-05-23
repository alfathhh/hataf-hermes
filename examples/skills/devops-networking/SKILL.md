---
name: devops-networking
description: Setup, troubleshoot, dan maintain infrastructure — server, Docker, CI/CD, networking, monitoring, deployment. Execution-first, verify sebelum claim selesai.
version: 1.0.0
metadata:
  hermes:
    tags: [devops, networking, docker, ci-cd, server, deploy, linux, monitoring]
    category: infrastructure
    requires_toolsets: [terminal]
---

# DevOps & Networking

Setup, troubleshoot, maintain infrastructure. Diagnosa dulu, fix kemudian, verify selalu.

## When to Use

- Server setup/troubleshoot
- Docker/container ops
- CI/CD pipeline
- Networking (DNS, firewall, VPN, SSL)
- Monitoring/alerting
- Deployment automation

## Procedure

### 1. Diagnosa dulu

```bash
uname -a && df -h && free -h && ss -tlnp && systemctl list-units --failed
```

### 2. Fix + verify

Setelah fix SELALU verify hasilnya jalan.

## Quick Reference: Docker, Nginx, SSL, CI/CD, Firewall, SSH, VPN, Monitoring

(See full skill content for complete reference)

## Pitfalls

1. Edit config tanpa backup
2. Firewall lock-out (block SSH)
3. Docker network isolation
4. Disk full dari Docker images
5. SSL cert expired

## Verification

1. Diagnosa sebelum fix? 2. Verify setelah fix? 3. Config di-backup? 4. Port gak exposed unnecessary? 5. Credentials gak hardcoded?
