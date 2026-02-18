# Access Control — 802.1X, NAC, Port Security

> **Propietario:** @network-team @security-team

## 1. 802.1X Authentication

### 1.1 RADIUS Configuration
```cisco
aaa new-model
radius server ISE-PRIMARY
 address ipv4 10.10.2.10 auth-port 1812 acct-port 1813
 key MyRadiusSecret

aaa group server radius AAA-ISE
 server name ISE-PRIMARY

aaa authentication dot1x default group AAA-ISE
aaa authorization network default group AAA-ISE
dot1x system-auth-control
```

### 1.2 Port Configuration (802.1X)
```cisco
interface GigabitEthernet1/0/1
 description User PC - Authenticated via 802.1X
 switchport mode access
 authentication port-control auto
 dot1x pae authenticator
 spanning-tree portfast
```

---

## 2. MAB (MAC Authentication Bypass)

```cisco
interface GigabitEthernet1/0/5
 description Printer - MAB
 switchport mode access
 authentication port-control auto
 mab
```

---

## 3. ACLs (Access Control Lists)

### 3.1 Standard ACL
```cisco
access-list 10 permit 10.10.10.0 0.0.0.255
access-list 10 deny any log

interface Vlan100
 ip access-group 10 in
```

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
