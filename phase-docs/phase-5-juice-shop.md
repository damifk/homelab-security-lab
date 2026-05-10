# Phase 5 — OWASP Juice Shop

## Objective
Deploy OWASP Juice Shop as a vulnerable web application target for attack simulation.

## Installation

On `ubuntu-sensor`:

```bash
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker dami
```

Log out and back in, then:

```bash
docker run -d --name juiceshop -p 3000:3000 bkimminich/juice-shop
```

## Verification

```bash
docker ps
# juiceshop container listed with 0.0.0.0:3000->3000/tcp
```

Accessible from Windows host browser at `http://192.168.100.129:3000`.

## Notes
- Juice Shop is a single-page application — returns HTTP 200 for all routes, which requires `--exclude-length` flag when using Gobuster
- The container will stop under heavy scan load (Nikto, sqlmap) — use `docker restart juiceshop` to recover
- Discovered endpoints: `/ftp` (exposed file directory), `/api`, `/apis`, `/assets`
