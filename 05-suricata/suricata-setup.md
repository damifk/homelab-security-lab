# Suricata IDS Setup and Wazuh Integration

## Overview
Suricata 7.0.3 was installed on the ARM64 homeserver VM as a network-layer
IDS, complementing Wazuh's endpoint telemetry with network traffic analysis.
Suricata alerts are forwarded to the Wazuh manager via the agent's log
collector.

## Why Suricata
Wazuh alone provides endpoint visibility — logins, file changes, privilege
escalation. Suricata adds network-layer detection, enabling:
- Signature-based detection of known attack patterns
- HTTP, DNS, and TLS traffic inspection
- Dual-layer detection when combined with Wazuh endpoint events

## Environment
- **Host:** homeserver (Ubuntu 24.04 ARM64)
- **Suricata Version:** 7.0.3
- **Interface Monitored:** enp0s1 (192.168.64.2)
- **Rule Source:** Emerging Threats Open (49,038 rules)
- **Log Output:** /var/log/suricata/eve.json

## Installation

```bash
sudo apt install suricata -y
```

Suricata 7.0.3 is available natively in the Ubuntu 24.04 ARM64 repository.
No architecture workarounds required.

## Configuration

### Update rules
```bash
sudo suricata-update
```

Fetches the Emerging Threats Open ruleset. On first run loaded 49,038 rules
with 49,043 signatures processed.

### Set network interface
```bash
sudo sed -i 's/interface: eth0/interface: enp0s1/' /etc/suricata/suricata.yaml
```

The default config references `eth0` which does not exist on this VM.
The correct interface is `enp0s1`.

### Enable and start
```bash
sudo systemctl enable suricata
sudo systemctl start suricata
```

## Wazuh Integration

The Wazuh agent was configured to read Suricata's JSON alert log by adding
a `<localfile>` block to `/var/ossec/etc/ossec.conf`:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

### Important: single ossec_config block
The agent config must contain only one `<ossec_config>` block. The
`<localfile>` entry must sit inside it alongside other localfile entries,
not nested inside `<client>` or in a separate `<ossec_config>` block.
Both mistakes were encountered during setup — the agent rejects the config
with error 1230 (`Invalid element in the configuration: 'localfile'`) if
either condition is present.

### Test config before restarting
```bash
sudo /var/ossec/bin/wazuh-agentd -t
```

Only restart the agent if this returns no errors.

```bash
sudo systemctl restart wazuh-agent
```

## Challenges Encountered

### eth0 interface not found
Suricata's default config references `eth0`. This VM uses `enp0s1`.
Suricata started but logged errors and failed to capture traffic until
the interface was corrected.

### localfile nested inside client block
During initial config edit, the `<localfile>` block was accidentally placed
inside the `<client>` block. Wazuh rejects this with error 1230.

### Two ossec_config blocks
A second `<ossec_config>` block was created instead of appending to the
existing one. Wazuh only supports a single `<ossec_config>` block per agent.
Resolution: merge all `<localfile>` entries into the existing block and
remove the duplicate `<ossec_config>` tags.

## Validation

```bash
# Suricata running and writing logs
sudo systemctl status suricata
sudo ls -la /var/log/suricata/eve.json

# Wazuh reading Suricata output
sudo grep -i suricata /var/ossec/logs/ossec.log
# Expected: INFO: (1950): Analyzing file: '/var/log/suricata/eve.json'
```

## Outcome
Suricata is running on `enp0s1` with 49,038 Emerging Threats rules loaded.
The Wazuh agent is reading and forwarding Suricata events to the local
wazuh-server manager. Network-layer alerts will appear in the Wazuh dashboard
under Security Events with rule groups containing `suricata`.

Meaningful Suricata alerts will be generated during attack simulations
in Phase 3 (Kali attack scenarios against Juice Shop).
