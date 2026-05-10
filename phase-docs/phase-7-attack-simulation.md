# Phase 7 — Attack Simulation

## Objective
Run structured attack campaigns from `kali-attacker` against Juice Shop, generating telemetry across both Suricata (network) and Sysmon (endpoint).

## Tool Installation on Kali

```bash
sudo apt update
sudo apt install nikto gobuster sqlmap -y
```

## Attack 1 — Nikto Web Reconnaissance

```bash
nikto -h http://192.168.100.129:3000 -o /tmp/nikto-results.txt
```

**Result:** 8,918 requests, 28 findings. Suricata generated 38+ pages of alerts in Wazuh dashboard including stream anomalies from the aggressive scan pattern.

## Attack 2 — Gobuster Directory Enumeration

```bash
gobuster dir -u http://192.168.100.129:3000 -w /usr/share/wordlists/dirb/common.txt -o /tmp/gobuster-results.txt --exclude-length 75002
```

Note: `--exclude-length 75002` required because Juice Shop returns HTTP 200 with the same SPA content for all non-existent paths.

**Findings:**

| Path | Status | Note |
|---|---|---|
| /ftp | 200 | Exposed file directory — known Juice Shop vulnerability |
| /api | 500 | API endpoint — server error on direct access |
| /apis | 500 | Same |
| /assets | 301 | Redirects to /assets/ |

## Attack 3 — sqlmap SQL Injection

```bash
sqlmap -u "http://192.168.100.129:3000/rest/products/search?q=test" --batch --level=2 --risk=1 --random-agent --threads=1 --output-dir=/tmp/sqlmap-results
```

**Result:** Boolean-based blind SQL injection confirmed on `q` parameter. SQLite backend identified. 65 HTTP 500 errors generated during probing.

**Detection gap identified:** Despite 65 server errors and multiple ET EXPLOIT rule matches for CVE-specific patterns, **zero SQL injection-specific alerts** were produced by Suricata's default ruleset. This finding drove Phase 8.

![Security Alerts](../screenshots/04-wazuh-security-alerts.png)

## Notes
- Juice Shop stops accepting connections under heavy concurrent load — use `docker restart juiceshop` to recover
- sqlmap's `--threads=1` flag keeps the connection stable at the cost of scan speed
- The `/ftp` directory exposure is worth noting for the portfolio — it represents a misconfiguration finding from the recon phase
