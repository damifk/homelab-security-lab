# Custom Detection Rules — Closing the Blind SQLi Gap

## Objective
Write custom Suricata rules to detect blind SQL injection attacks that
evaded the default Emerging Threats ruleset, directly closing a detection
gap identified during Phase 3 attack simulations.

## Background
During Phase 3, sqlmap confirmed a real blind SQL injection vulnerability
in the Juice Shop search endpoint. Suricata generated zero alerts because:
- Blind injection payloads are subtle — boolean and time-based techniques
  avoid obvious keywords like UNION SELECT
- Emerging Threats rules target noisy, known-pattern attacks
- Application-layer blind SQLi requires endpoint-specific detection logic

This phase addresses that gap with custom rules targeting the specific
endpoint and payload patterns observed.

## Custom Rules

Rules are stored in `/var/lib/suricata/rules/local.rules` and referenced
in `/etc/suricata/suricata.yaml`.

```
# Detect blind SQL injection attempts on Juice Shop search endpoint
alert http any any -> $HOME_NET 8081 (
  msg:"LOCAL SQLi Blind Injection Attempt - Juice Shop Search";
  flow:established,to_server;
  http.method; content:"GET";
  http.uri; content:"/rest/products/search";
  content:"AND"; nocase;
  pcre:"/search\?q=.*(\%27|'|AND|OR|UNION|SELECT|RANDOMBLOB|LIKE\(CHAR)/i";
  classtype:web-application-attack;
  sid:9000001; rev:1;
)

# Detect SQL time-based blind injection (RANDOMBLOB pattern)
alert http any any -> $HOME_NET 8081 (
  msg:"LOCAL SQLi Time-Based Blind - RANDOMBLOB Detected";
  flow:established,to_server;
  http.uri; content:"RANDOMBLOB"; nocase;
  classtype:web-application-attack;
  sid:9000002; rev:1;
)

# Detect gobuster/dirbuster style scanning by User-Agent
alert http any any -> $HOME_NET any (
  msg:"LOCAL Web Scanner Detected - gobuster User-Agent";
  flow:established,to_server;
  http.user_agent; content:"gobuster"; nocase;
  classtype:web-application-activity;
  sid:9000003; rev:1;
)

# Detect Nikto scanner by User-Agent
alert http any any -> $HOME_NET any (
  msg:"LOCAL Web Scanner Detected - Nikto User-Agent";
  flow:established,to_server;
  http.user_agent; content:"nikto"; nocase;
  classtype:web-application-activity;
  sid:9000004; rev:1;
)
```

## Rule SID Allocation
Custom rules use SIDs starting at 9000001 to avoid conflicts with
Emerging Threats rules (which use lower SID ranges).

## Deployment

```bash
# Copy rules to Suricata rules directory
sudo cp /etc/suricata/rules/local.rules /var/lib/suricata/rules/local.rules

# Add to suricata.yaml rule-files section
# rule-files:
#   - /var/lib/suricata/rules/local.rules
#   - suricata.rules

# Test configuration
sudo suricata -T -c /etc/suricata/suricata.yaml -v 2>&1 | tail -5

# Restart to load new rules
sudo systemctl restart suricata
```

## Challenges Encountered

### local.rules path mismatch
The yaml `rule-files` section references files relative to the default
rules directory (`/var/lib/suricata/rules/`), not `/etc/suricata/rules/`.
The file must exist in `/var/lib/suricata/rules/` or be referenced with
an absolute path.

### USR2 reload insufficient for new rule files
`sudo kill -USR2 $(cat /run/suricata.pid)` reloads existing rule files
but does not pick up newly added rule files. A full service restart is
required when adding `local.rules` for the first time.

## Validation

Reran sqlmap with `--flush-session` to force full payload retransmission:

```bash
sqlmap -u "http://192.168.64.2:8081/rest/products/search?q=test" \
  --batch --level=3 --flush-session
```

### Results

| Rule | SID | Fires |
|---|---|---|
| LOCAL SQLi Blind Injection Attempt - Juice Shop Search | 9000001 | 71 |
| LOCAL SQLi Time-Based Blind - RANDOMBLOB Detected | 9000002 | Multiple |

### Alert detail confirmed in Wazuh dashboard

```
data.alert.signature:    LOCAL SQLi Blind Injection Attempt - Juice Shop Search
data.alert.signature_id: 9000001
data.alert.category:     Web Application Attack
data.http.url:           /rest/products/search?q=test%25%27%20AND%20...
data.http.http_user_agent: sqlmap/1.9.11#stable
data.src_ip:             192.168.64.4
rule.firedtimes:         71
rule.groups:             ids, suricata
```

The exact payload, attacker IP, tool identity, and target endpoint are
all captured and forwarded to Wazuh — providing complete investigative
context that was previously absent.

## Before vs After

| Scenario | Before custom rules | After custom rules |
|---|---|---|
| sqlmap blind SQLi | 0 alerts | 71+ alerts |
| RANDOMBLOB time-based | 0 alerts | Multiple alerts |
| gobuster scan | Metadata only | User-Agent alert |
| Nikto scan | ET signatures only | ET + User-Agent alert |

## Key Takeaways

- Default rulesets cover known, noisy attacks — custom rules are required
  for targeted, application-specific detection
- Detection engineering is iterative: simulate → identify gaps → write rules
  → validate → repeat
- Suricata's PCRE and content matching provides precise payload inspection
- SID namespacing prevents conflicts between custom and vendor rules
- A full service restart is required when adding new rule files
