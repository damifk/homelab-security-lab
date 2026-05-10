# Phase 4 — Suricata IDS

## Objective
Deploy Suricata on `ubuntu-sensor`, configure for the lab network, and integrate with Wazuh via eve.json.

## Installation

```bash
sudo add-apt-repository ppa:oisf/suricata-stable -y
sudo apt update
sudo apt install suricata -y
```

## Configuration

Edit `/etc/suricata/suricata.yaml`:

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.100.0/24]"

af-packet:
  - interface: ens33
```

## Rules Update

```bash
sudo suricata-update
sudo systemctl restart suricata
```

## Wazuh Integration

Add to `/var/ossec/etc/ossec.conf` before closing `</ossec_config>` tag:

```xml
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
```

Restart Wazuh agent:
```bash
sudo systemctl restart wazuh-agent
```

## Validation

Test alert generation:
```bash
curl http://testmynids.org/uid/index.html
sudo cat /var/log/suricata/fast.log
```

Expected: `GPL ATTACK_RESPONSE id check returned root` alert — confirms Suricata is inspecting traffic and generating alerts that flow into Wazuh.

## Notes
- Interface name confirmed as `ens33` via `ip a | grep "192.168.100"`
- Full service restart required when adding rule files — USR2 signal reload is insufficient
- Suricata alerts visible in Wazuh dashboard under agent `ubuntu-sensor`
