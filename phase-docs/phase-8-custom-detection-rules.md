# Phase 8 — Custom Suricata Detection Rules

## Objective
Identify a detection gap in the default Suricata ruleset exposed during the sqlmap attack simulation, write custom rules to close it, and validate detection on retest.

---

## The Detection Gap

During the sqlmap attack simulation in Phase 7, sqlmap confirmed a boolean-based blind SQL injection vulnerability on the Juice Shop `q` parameter and identified a SQLite backend. Despite generating 65 HTTP 500 errors and triggering multiple Emerging Threats signatures for CVE-specific exploits, **zero SQL injection-specific alerts were produced**.

Verification:
```bash
sudo cat /var/log/suricata/fast.log | grep -i "sql"
# No output
```

The Emerging Threats ruleset detects known CVE exploit patterns and HTTP anomalies but does not cover blind SQL injection techniques — specifically boolean-based blind and time-based blind payloads which are structurally different from classic error-based or UNION-based injection.

This is the gap.

---

## Custom Rules

Rules stored at: `/var/lib/suricata/rules/local.rules`

```
alert http any any -> $HTTP_SERVERS any (msg:"LOCAL SQLi - Boolean-Based Blind Injection Attempt"; flow:established,to_server; http.uri; content:"1=1"; nocase; classtype:web-application-attack; sid:9000001; rev:1;)

alert http any any -> $HTTP_SERVERS any (msg:"LOCAL SQLi - Time-Based Blind Injection SLEEP"; flow:established,to_server; http.uri; content:"SLEEP("; nocase; classtype:web-application-attack; sid:9000002; rev:1;)

alert http any any -> $HTTP_SERVERS any (msg:"LOCAL SQLi - UNION SELECT Injection Attempt"; flow:established,to_server; http.uri; content:"UNION"; nocase; content:"SELECT"; nocase; distance:0; classtype:web-application-attack; sid:9000003; rev:1;)

alert http any any -> $HTTP_SERVERS any (msg:"LOCAL SQLi - SQLite Time-Based Blind Heavy Query"; flow:established,to_server; http.uri; content:"randomblob("; nocase; classtype:web-application-attack; sid:9000004; rev:1;)
```

### Rule Design Notes

- **SID 9000001** targets the `1=1` boolean tautology used in boolean-based blind injection — sqlmap's primary technique against this target
- **SID 9000002** targets `SLEEP(` — used in time-based blind injection on MySQL backends; included for coverage
- **SID 9000003** uses two `content` keywords with `distance:0` to require UNION and SELECT appear in the same URI without gap — reduces false positives from single keyword matches
- **SID 9000004** targets `randomblob(` — SQLite's equivalent of `SLEEP()` for time-based blind injection; confirmed used by sqlmap against this SQLite backend
- All rules use `any any` as source (not `$EXTERNAL_NET`) because the attacker (`192.168.100.130`) is within the `HOME_NET` range

---

## Validation

Retest with sqlmap using boolean-based technique only (`--technique=B`) to keep Juice Shop stable:

```bash
sqlmap -u "http://192.168.100.129:3000/rest/products/search?q=test" --batch --level=2 --risk=1 --random-agent --threads=1 --technique=B
```

Result on `ubuntu-sensor`:

```bash
sudo cat /var/log/suricata/fast.log | grep "LOCAL SQLi"
```

```
05/08/2026-22:20:17.481413 [**] [1:9000001:1] LOCAL SQLi - Boolean-Based Blind Injection Attempt [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.100.130:37898 -> 192.168.100.129:3000
05/08/2026-22:20:17.481413 [**] [1:9000003:1] LOCAL SQLi - UNION SELECT Injection Attempt [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.100.130:37898 -> 192.168.100.129:3000
05/08/2026-22:20:22.935824 [**] [1:9000004:1] LOCAL SQLi - SQLite Time-Based Blind Heavy Query [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.100.130:38402 -> 192.168.100.129:3000
```

![Custom SQLi Rules Firing](../screenshots/03-custom-sqli-rules.png)

Detection count moved from **0 to 3 active rule matches** on retest.

---

## Deployment Steps

1. Create rules file:
```bash
sudo nano /var/lib/suricata/rules/local.rules
```

2. Register rules file in Suricata config:
```bash
sudo nano /etc/suricata/suricata.yaml
# Add under rule-files:
#  - /var/lib/suricata/rules/local.rules
```

3. Validate config before restart:
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

4. Restart Suricata (full restart required — USR2 signal reload does not load new rule files):
```bash
sudo systemctl restart suricata
```

---

## Key Takeaway

Default IDS rulesets are built around known signatures. Blind SQL injection — which infers data through application behaviour rather than error messages — produces no visible exploit pattern for signature matching. Custom rules targeting the specific payload strings used by blind injection tools close this gap. This is detection engineering: understanding attacker technique, identifying what it looks like on the wire, and writing rules that catch it.
