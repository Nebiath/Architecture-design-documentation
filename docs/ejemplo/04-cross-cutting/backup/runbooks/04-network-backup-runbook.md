# Runbook 04: Backup de Dispositivos de Red

> **Tiempo:** 15 min | **Propietario:** Comunicaciones Team

## Cisco Switches

**Backup manual:**
```bash
# SSH al switch
ssh admin@switch-ip

# Backup running-config
copy running-config scp://user@backup-server/configs/switch-hostname-$(date +%F).cfg

# O vía TFTP (menos seguro)
copy running-config tftp://backup-server/switch-hostname.cfg
```

**Automatizado (Oxidized):**
```yaml
# /etc/oxidized/config
source:
  default: csv
  csv:
    file: /etc/oxidized/router.db
    delimiter: !ruby/regexp /:/
    map:
      name: 0
      model: 1
      username: 2
      password: 3

model_map:
  cisco: ios
  fortinet: fortios

# router.db:
# switch-core-01:ios:backup-user:password123
# firewall-fw01:fortios:admin:password456
```

## Fortinet FortiGate

**Backup vía CLI:**
```bash
# SSH al firewall
ssh admin@fortigate-ip

# Backup config
execute backup config management-station "Daily backup $(date +%F)"

# Download via SCP
execute backup config scp admin@backup-server:/backups/fortigate-config.conf
```

**Backup vía API (automatizado):**
```python
import requests
response = requests.post(
    'https://fortigate-ip/api/v2/monitor/system/config/backup',
    params={'scope': 'global', 'access_token': 'YOUR_TOKEN'},
    verify=False
)
with open(f'fortigate-backup-{date}.conf', 'wb') as f:
    f.write(response.content)
```

## Retención

- Daily backups: 30 días
- Pre/post change backups: 90 días
- Git repository: Indefinido (tracking de cambios)
