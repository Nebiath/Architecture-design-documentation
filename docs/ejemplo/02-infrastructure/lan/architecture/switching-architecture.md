# Switching Architecture — VLANs, STP, Stacking

> **Propietario:** @network-team @lan-lead  
> **Cisco Catalyst 9000 Series**

## 1. Switch Stacking

### 1.1 StackWise vs VSS
| Feature | StackWise (C9300) | VSS (C9500) |
|---------|------------------|-------------|
| Switches soportados | Hasta 8 switches | 2 switches |
| Topología | Ring o Star | Dual-active |
| Bandwidth stack | 480 Gbps | 100G VSL |
| Control plane | Único | Único |
| Management IP | Una IP para todo el stack | Una IP para el par VSS |
| Failover | < 1 seg | < 1 seg |

### 1.2 StackWise Configuration
```cisco
! Ver estado del stack
show switch

! Cambiar prioridad de stack (para elegir master)
switch 1 priority 15
switch 2 priority 14

! Provisioning de nuevo switch en stack
switch 3 provision c9300-48p

! Renombering switch en stack
switch 3 renumber 5
```

### 1.3 VSS Configuration (Catalyst 9500)
```cisco
! Switch 1 (VSS Primary)
switch virtual domain 1
 switch 1
 switch 1 priority 150

interface TenGigabitEthernet1/1/1
 switchport mode trunk
 switchport trunk allowed vlan all
 channel-group 10 mode on
 no shutdown
!
interface Port-channel10
 switch virtual link 1
 no shutdown

! Switch 2 (VSS Secondary)
switch virtual domain 1
 switch 2
 switch 2 priority 100

interface TenGigabitEthernet2/1/1
 switchport mode trunk
 switchport trunk allowed vlan all
 channel-group 10 mode on
 no shutdown
!
interface Port-channel10
 switch virtual link 2
 no shutdown

! Convertir a VSS mode (requiere reinicio)
switch convert mode virtual
```

---

## 2. VTP (VLAN Trunking Protocol)

### 2.1 VTP Modes
| Mode | Descripción | Uso recomendado |
|------|-------------|----------------|
| Server | Crea/modifica/elimina VLANs, propaga cambios | Core/Distribution |
| Client | Recibe VLANs de server, no puede modificar | No recomendado (usar transparent) |
| Transparent | No participa en VTP, reenvía advertisements | **Recomendado para todos** |
| Off | VTP deshabilitado completamente | Alternativa moderna |

**Recomendación:** Usar VTP mode **transparent** o **off** para evitar propagación accidental de cambios.

```cisco
! Configurar VTP transparent (recomendado)
vtp mode transparent
vtp domain COMPANY
vtp version 3

! Verificar
show vtp status
```

---

## 3. Trunk Configuration

### 3.1 802.1Q Trunking
```cisco
! Trunk entre switches
interface TenGigabitEthernet1/0/48
 description Trunk to DIST-SW-01
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,40,50,100
 no shutdown
```

**Native VLAN:** Usar VLAN dedicada (ej: 999) no usada para tráfico, para evitar VLAN hopping attacks.

### 3.2 DTP (Dynamic Trunking Protocol)
```cisco
! Deshabilitar DTP (seguridad)
switchport nonegotiate
```

---

## 4. Port Security

### 4.1 Sticky MAC Address Learning
```cisco
interface GigabitEthernet1/0/1
 description User PC - John Doe
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 no shutdown
```

### 4.2 Port Security Violation Actions
| Action | Behavior |
|--------|----------|
| **shutdown** (default) | Port err-disabled, requiere `shutdown` / `no shutdown` |
| **restrict** | Drop packets, incrementa counter, envía SNMP trap |
| **protect** | Drop packets silenciosamente |

---

## 5. Storm Control

```cisco
! Limitar broadcast/multicast/unicast storms
interface range GigabitEthernet1/0/1-24
 storm-control broadcast level 10.00
 storm-control multicast level 10.00
 storm-control action shutdown
```

---

## 6. Port Monitoring (SPAN)

### 6.1 Local SPAN
```cisco
! Monitor puerto 1 enviando tráfico a puerto 48
monitor session 1 source interface GigabitEthernet1/0/1
monitor session 1 destination interface GigabitEthernet1/0/48

! Ver SPAN sessions
show monitor session 1
```

### 6.2 Remote SPAN (RSPAN)
```cisco
! Crear RSPAN VLAN
vlan 900
 name RSPAN
 remote-span

! Source switch
monitor session 1 source interface Gi1/0/1
monitor session 1 destination remote vlan 900

! Destination switch
monitor session 1 source remote vlan 900
monitor session 1 destination interface Gi1/0/48
```

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
