# DHCP Design — Dynamic Host Configuration

> **Propietario:** @network-team @lan-lead

## 1. DHCP Server Architecture

### 1.1 Deployment
- **Primary DHCP:** Windows Server 2022 (10.10.1.50)
- **Secondary DHCP:** Windows Server 2022 (10.10.1.51)
- **Split scope:** 80/20 (Primary 80%, Secondary 20%)

---

## 2. DHCP Relay (IP Helper)

```cisco
! En cada VLAN SVI (Distribution switch)
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 ip helper-address 10.10.1.50
 ip helper-address 10.10.1.51
```

---

## 3. DHCP Scopes

### 3.1 Scope VLAN 10 - IT
```
Scope: 10.10.10.0/24
Range: 10.10.10.101 - 10.10.10.254
Gateway: 10.10.10.1
DNS: 10.10.1.10, 10.10.1.11
Lease: 8 hours (workstations), 30 days (servers)
```

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
