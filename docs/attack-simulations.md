# Attack Simulations — Kali vs Juice Shop

## Objective
Validate dual-layer detection by running structured attack scenarios from
a Kali Linux VM against the OWASP Juice Shop target, observing detections
at both the network layer (Suricata) and endpoint layer (Wazuh).

## Environment
- **Attacker:** Kali Linux (192.168.64.4)
- **Target:** OWASP Juice Shop on homeserver (192.168.64.2:8081)
- **Detection layers:**
  - Suricata IDS on homeserver (enp0s1) — network layer
  - Wazuh agent on homeserver — endpoint layer
  - Wazuh manager on wazuh-server (192.168.64.5) — SIEM

---

## Attack 1 — Web Reconnaissance (Nikto)

### Tool
Nikto v2.5.0 — automated web vulnerability scanner

### Command
```bash
nikto -h http://192.168.64.2:8081
```

### Findings (Nikto)
- Exposed `/ftp/` directory (200 response)
- Missing X-Content-Type-Options header
- Potentially interesting backup files discoverable
- Uncommon header `x-recruiting` leaking internal info
- 169 items reported across 7,951 requests in 91 seconds

### Detections (Suricata)
Suricata fired on multiple ET rule categories:

| Signature | Category |
|---|---|
| ET EXPLOIT Cisco ASA CVE-2020-3452 Path Traversal | Exploit |
| ET EXPLOIT Cisco ASA/Firepower Unauthenticated File Read | Exploit |
| ET EXPLOIT F5 TMUI RCE CVE-2020-5902 | Exploit |
| ET EXPLOIT Citrix ADC Arbitrary Code Execution CVE-2019-19781 | Exploit |
| ET EXPLOIT Fortinet CVE-2018-13379 Path Traversal | Exploit |
| ET EXPLOIT D-Link DSL-2750B Command Injection CVE-2016-20017 | Exploit |
| ET EXPLOIT Cisco RV320/RV325 Config Disclosure CVE-2019-1653 | Exploit |
| ET WEB_SERVER ColdFusion administrator access | Web Server |
| ET WEB_SPECIFIC_APPS WordPress LiteSpeed Cache CVE-2024-4400 | Web App |
| ET INFO Request to Hidden Environment File | Info |
| SURICATA HTTP Host header ambiguous/invalid | Protocol Anomaly |
| ET INFO Possible Kali Linux hostname in DHCP Request Packet | Recon |

### Notable finding
Suricata identified the attacker's OS (Kali Linux) from a DHCP broadcast
packet — before any attack ran. This demonstrates network-layer attacker
fingerprinting.

### Endpoint layer
Nikto's HTTP requests do not cross the OS authentication boundary, so no
Wazuh endpoint alerts were generated. This is expected behaviour — web
scanning is invisible to auth.log, PAM, and sudo monitoring.

---

## Attack 2 — SQL Injection (sqlmap)

### Tool
sqlmap 1.9.11 — automated SQL injection testing tool

### Command
```bash
sqlmap -u "http://192.168.64.2:8081/rest/products/search?q=test" --batch --level=3
```

### Findings (sqlmap)
A real SQL injection vulnerability was confirmed in the Juice Shop search endpoint:

```
Parameter: q (GET)
  Type: boolean-based blind
  Payload: q=test%' AND 4465=4465 AND 'FCtj%'='FCtj

  Type: time-based blind
  Payload: q=test%' AND 1062=LIKE(CHAR(65,66,67,68,69,70,71),
           UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'ujbT%'='ujbT

Back-end DBMS: SQLite
HTTP 500 errors: 139 times
```

### Detections (Suricata)
No Suricata alerts were generated for this attack.

**Why:** Blind SQL injection uses subtle boolean and time-based payloads
that do not match Emerging Threats signatures. Suricata's ruleset is
optimised for noisy, obvious injection patterns (UNION SELECT, error-based).
Blind injection is deliberately designed to evade signature-based detection.

**Detection gap identified:** Blind SQLi is a known blind spot for
signature-based IDS. Effective detection would require:
- WAF with behavioural analysis (e.g. ModSecurity with anomaly scoring)
- Application-layer logging and anomaly detection
- Custom Suricata rules tuned to the specific endpoint

### Endpoint layer
No Wazuh endpoint alerts — the injection targets the Node.js application
layer, not the OS.

---

## Attack 3 — Directory Enumeration (gobuster)

### Tool
gobuster v3.8 — directory and file brute-forcing tool

### Command
```bash
gobuster dir -u http://192.168.64.2:8081 \
  -w /usr/share/wordlists/dirb/common.txt \
  --exclude-length 75055 -b "" -s 301,302,403
```

### Findings (gobuster)
- `/assets` directory confirmed (301 redirect)
- Juice Shop returns HTTP 200 for all non-existent paths (SPA behaviour)
  requiring response length filtering to identify real paths
- Juice Shop became unresponsive partway through the scan due to Node.js
  being overwhelmed by 10 concurrent threads

### Detections (Suricata)
Suricata captured gobuster traffic and fingerprinted the tool by its
HTTP User-Agent string:

```
data.http.http_user_agent: gobuster/3.8
data.src_ip: 192.168.64.4
data.dest_ip: 192.168.64.2
data.dest_port: 8081
data.in_iface: enp0s1
location: /var/log/suricata/eve.json
```

The tool identity, source IP, and target were all visible in the Suricata
event forwarded to Wazuh. This demonstrates that even when specific exploit
signatures don't fire, Suricata's HTTP logging provides investigative value
through flow metadata and user agent fingerprinting.

### Endpoint layer
No Wazuh endpoint alerts — directory enumeration does not cross the OS
authentication boundary.

---

## Summary — Dual Layer Detection

| Attack | Suricata | Wazuh Endpoint |
|---|---|---|
| Nikto web scan | ✅ Multiple ET EXPLOIT signatures | ❌ Expected — no OS boundary |
| sqlmap blind SQLi | ❌ Detection gap (blind payloads) | ❌ Expected — app layer only |
| gobuster dir enum | ✅ Tool fingerprinted via User-Agent | ❌ Expected — no OS boundary |
| SSH brute force (prior) | ✅ DHCP Kali fingerprint | ✅ PAM, sshd, brute force rules |
| Privilege escalation (prior) | N/A | ✅ sudo, PAM, MITRE T1548.003 |

## Key Takeaways

- Suricata and Wazuh are complementary — neither covers everything alone
- Signature-based IDS is effective against noisy, known-pattern attacks
- Blind SQL injection is a real detection gap for Suricata's default ruleset
- HTTP metadata (User-Agent, flow data) provides attacker fingerprinting
  even when no signatures fire
- Web attacks that stay in the application layer are invisible to OS-level
  endpoint monitoring
- Defence in depth requires multiple detection layers with different
  visibility angles

## Next Steps
- Write custom Suricata rules to detect blind SQLi patterns on the
  Juice Shop search endpoint
- Explore WAF integration for application-layer coverage
