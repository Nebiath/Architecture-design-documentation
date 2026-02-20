# DNS Architecture — Name Resolution

> **Propietario:** @network-team @windows-team

## 1. DNS Servers

### 1.1 Internal DNS (Active Directory Integrated)
| Hostname | IP | Role | OS |
|----------|-----|------|-----|
| DC01 | 10.10.1.10 | Primary DNS | Windows Server 2022 |
| DC02 | 10.10.1.11 | Secondary DNS | Windows Server 2022 |

### 1.2 DNS Zones
- **company.com** (AD-integrated, primary zone)
- **10.10.in-addr.arpa** (reverse lookup)

---

## 2. DNS Forwarders

```powershell
# External DNS forwarders (for internet resolution)
Add-DnsServerForwarder -IPAddress "1.1.1.1" -PassThru
Add-DnsServerForwarder -IPAddress "8.8.8.8" -PassThru
```

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
