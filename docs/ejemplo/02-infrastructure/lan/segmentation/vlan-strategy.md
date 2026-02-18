# VLAN Strategy — Diseño de VLANs Corporativas

> **Propietario:** @network-team @lan-lead

## 1. VLANs Estándar Corporativas

| VLAN ID | Nombre | Subnet | Gateway | Uso |
|---------|--------|--------|---------|-----|
| 10 | VLAN-IT | 10.10.10.0/24 | 10.10.10.1 | IT Department |
| 20 | VLAN-Finance | 10.10.20.0/24 | 10.10.20.1 | Finance Department |
| 30 | VLAN-Voice | 10.10.30.0/23 | 10.10.30.1 | VoIP phones |
| 40 | VLAN-Guest | 10.10.40.0/24 | 10.10.40.1 | Guest WiFi |
| 50 | VLAN-IoT | 10.10.50.0/24 | 10.10.50.1 | IoT devices |
| 100 | VLAN-Mgmt | 10.10.1.0/24 | 10.10.1.1 | Network management |
| 200 | VLAN-Servers | 10.10.200.0/22 | 10.10.200.1 | Server farm |
| 999 | VLAN-Native | — | — | Native VLAN (no usado) |

---

## 2. Crear VLANs en Switch

```cisco
! Crear VLANs
vlan 10
 name VLAN-IT
vlan 20
 name VLAN-Finance
vlan 30
 name VLAN-Voice
vlan 40
 name VLAN-Guest
vlan 100
 name VLAN-Mgmt

! Verificar
show vlan brief
```

---

## 3. Inter-VLAN Routing

### 3.1 SVI (Switched Virtual Interface) en Distribution Layer
```cisco
! Distribution Switch (Layer 3)
ip routing

interface Vlan10
 description VLAN-IT Gateway
 ip address 10.10.10.1 255.255.255.0
 ip helper-address 10.10.1.50
 no shutdown

interface Vlan20
 description VLAN-Finance Gateway
 ip address 10.10.20.1 255.255.255.0
 ip helper-address 10.10.1.50
 no shutdown
```

---

## 4. VLAN Pruning

```cisco
! En trunk, permitir solo VLANs necesarias
interface TenGigabitEthernet1/0/48
 switchport trunk allowed vlan 10,20,30,100
```

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
