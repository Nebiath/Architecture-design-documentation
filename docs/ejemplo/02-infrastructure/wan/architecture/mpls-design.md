# Diseño MPLS — Red Privada Corporativa

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Diseño técnico de la red MPLS L3VPN corporativa: topología, routing, VRFs, QoS y procedimientos de operación con proveedor.

---

## 1. Resumen del Servicio MPLS

| Campo | Valor |
|-------|-------|
| Tipo de servicio | MPLS L3VPN / VPLS / MPLS L2VPN |
| Proveedor principal | [Nombre proveedor] |
| Contrato ID | [Número contrato] |
| Número de sites conectados | [N] |
| AS corporativo (CE) | [AS Number] |
| AS del proveedor (PE) | [AS Number] |
| Protocolo CE-PE | BGP / OSPF / Static |
| Fecha inicio contrato | YYYY-MM-DD |
| Fecha renovación | YYYY-MM-DD |

---

## 2. Arquitectura MPLS L3VPN

### 2.1 Diagrama Lógico

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    RED MPLS PROVEEDOR                        │
  │                                                              │
  │   PE-[Ciudad-A]──────────────────────────PE-[Ciudad-B]      │
  │        │                                        │            │
  │   PE-[Ciudad-C]──────────────────────────PE-[Ciudad-D]      │
  │                                                              │
  └──────┬───────────────────────────────────────────┬──────────┘
         │ CE-PE Link                                 │ CE-PE Link
         │                                            │
   ┌─────┴──────┐                             ┌──────┴──────┐
   │  CE-HQ-01  │                             │  CE-DC-01   │
   │  (Nuestra  │                             │  (Nuestra   │
   │   red)     │                             │   red)      │
   └────────────┘                             └─────────────┘
```

> 📎 Diagrama completo: [`./diagrams/mpls-logical.png`](./diagrams/mpls-logical.png)

### 2.2 Inventario de Routers CE

| Hostname | Site | Modelo | IOS/Version | IP Loopback | Estado |
|----------|------|--------|-------------|-------------|--------|
| CE-HQ-01 | HQ | [Modelo] | [Versión] | [X.X.X.X/32] | ✅ Activo |
| CE-HQ-02 | HQ | [Modelo] | [Versión] | [X.X.X.X/32] | ✅ Activo |
| CE-DC-01 | DC Principal | [Modelo] | [Versión] | [X.X.X.X/32] | ✅ Activo |
| CE-[SITE]-01 | [Site] | [Modelo] | [Versión] | [X.X.X.X/32] | ✅ Activo |

> Inventario completo en IPAM/CMDB: [URL]

---

## 3. Diseño de VRFs

### 3.1 VRFs Definidos

| VRF Name | Route Target Export | Route Target Import | Uso | Sites |
|----------|--------------------|--------------------|-----|-------|
| `CORP-PROD` | `[AS]:[N]` | `[AS]:[N]` | Producción corporativa | Todos |
| `CORP-MGMT` | `[AS]:[N]` | `[AS]:[N]` | Gestión de red (OOB) | HQ, DCs |
| `CORP-DEV` | `[AS]:[N]` | `[AS]:[N]` | Entornos de desarrollo | HQ, DC-Principal |
| `CORP-GUEST` | `[AS]:[N]` | `[AS]:[N]` | WiFi invitados (si aplica) | Offices |

### 3.2 Política de Fuga de Rutas Entre VRFs (Route Leaking)

| Origen VRF | Destino VRF | Rutas permitidas | Justificación |
|-----------|-------------|-----------------|---------------|
| `CORP-MGMT` | `CORP-PROD` | Solo rango [X.X.X.0/24] | Acceso monitoring a servidores prod |
| `CORP-PROD` | `CORP-DEV` | Solo [X.X.X.0/24] | Acceso a servicios compartidos |

---

## 4. Configuración BGP CE-PE

### 4.1 Parámetros BGP por Defecto

| Parámetro | Valor |
|-----------|-------|
| AS corporativo | [AS número] |
| Timers (keepalive/hold) | [X] / [X] segundos |
| BFD habilitado | Sí — Intervalo [X] ms |
| Prefijos máximos anunciados | [N] |
| Autenticación MD5 | Sí |
| Route Reflector | Proveedor actúa como RR |

### 4.2 Comunidades BGP en Uso

| Comunidad | Significado | Acción |
|-----------|-------------|--------|
| `[AS]:[N]` | Ruta preferente por MPLS | Aumentar LOCAL_PREF a [X] |
| `[AS]:[N]` | No anunciar a otros sites | Filtro de exportación en PE |
| `[AS]:[N]` | Ruta de backup | Reducir LOCAL_PREF a [X] |

### 4.3 Template de Configuración CE (referencia)

```
! ============================================
! PLANTILLA BGP CE — [Proveedor MPLS]
! Adaptar variables en [CORCHETES]
! ============================================

router bgp [AS-CORPORATIVO]
 bgp router-id [LOOPBACK-IP]
 bgp log-neighbor-changes
 !
 address-family ipv4 vrf CORP-PROD
  neighbor [PE-IP] remote-as [AS-PROVEEDOR]
  neighbor [PE-IP] activate
  neighbor [PE-IP] description "PE [Proveedor] - [Site]"
  neighbor [PE-IP] password [HASH-MD5]
  neighbor [PE-IP] bfd
  neighbor [PE-IP] soft-reconfiguration inbound
  neighbor [PE-IP] send-community both
  neighbor [PE-IP] maximum-prefix [N] [WARNING%]
  !
  network [RED-LOCAL] mask [MASK]
 exit-address-family
!

! BFD en interfaz CE-PE
interface [INTERFAZ-CE-PE]
 bfd interval [X] min_rx [X] multiplier [X]
!
```

> ⚠️ La contraseña MD5 se obtiene de CyberArk/Vault — **nunca en el repositorio**

---

## 5. QoS MPLS

### 5.1 Marcado en el CE (entrada al MPLS)

| Tráfico | Clasificación | DSCP marcado | EXP bits MPLS |
|---------|--------------|-------------|---------------|
| VoIP RTP | ACL / NBAR | EF (46) | 5 |
| VoIP Señalización | ACL (SIP/H.323) | CS3 (24) | 3 |
| SAP / ERP | ACL por puerto / NBAR | AF31 (26) | 3 |
| Best Effort | Default | BE (0) | 0 |

### 5.2 Política de Salida (Shaping hacia MPLS)

```
! Policy-map de salida en interfaz WAN CE-PE
policy-map PM-WAN-EGRESS
 class VOIP-RTP
  priority [BW-VOIP-kbps]          ! LLQ — garantía estricta
 class BUSINESS-CRITICAL
  bandwidth percent [X]             ! CBWFQ
  queue-limit [N] packets
 class class-default
  fair-queue
  bandwidth percent [X]
!
```

---

## 6. Gestión y Monitorización

### 6.1 Acceso a Dispositivos CE

| Método | Protocolo | Red de acceso | Autenticación |
|--------|-----------|---------------|--------------|
| CLI administración | SSH v2 | VRF CORP-MGMT | TACACS+ ([Servidor]) |
| Monitorización SNMP | SNMPv3 | VRF CORP-MGMT | Auth: SHA / Priv: AES-128 |
| Syslog | UDP 514 | VRF CORP-MGMT | [Servidor Syslog] |
| NetFlow / IPFIX | UDP 2055 | VRF CORP-MGMT | [Colector] |

### 6.2 Métricas y Umbrales de Alerta

| Métrica | Warning | Critical | Herramienta |
|---------|---------|----------|-------------|
| Utilización enlace CE-PE | > 70% | > 90% | [Zabbix/SNMP] |
| BGP session down | — | Inmediato | [Herramienta] |
| BFD session down | — | Inmediato | [Herramienta] |
| Pérdida de paquetes | > 0,5% | > 1% | TWAMP / IPSLA |
| Latencia CE-CE (HQ→Site) | > [X] ms | > [X] ms | IPSLA |
| Jitter | > [X] ms | > [X] ms | IPSLA |

---

## 7. Procedimientos de Cambio

| Tipo de cambio | Ventana permitida | Aprobación | Tiempo estimado |
|---------------|-------------------|-----------|----------------|
| Actualización IOS CE | Sábados 02:00–06:00 UTC | CAB + WAN Lead | 45 min por router |
| Cambio de configuración BGP | Laborables 07:00–09:00 | WAN Lead | 15 min |
| Añadir nuevo prefijo a MPLS | Laborables (cualquier hora) | WAN Lead | 5 min |
| Modificación QoS | Sábados 02:00–06:00 UTC | CAB | 30 min |
| Sustitución router CE | Sábados 00:00–06:00 UTC | CAB + Service Owner | 2–4 h |

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
