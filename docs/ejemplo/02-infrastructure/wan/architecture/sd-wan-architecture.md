# Arquitectura SD-WAN

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Diseño técnico de la solución SD-WAN corporativa: plataforma, topología de control plane, políticas de tráfico, integración con MPLS y seguridad.

---

## 1. Resumen de la Solución SD-WAN

| Campo | Valor |
|-------|-------|
| Plataforma SD-WAN | [Cisco Viptela / VMware VeloCloud / Fortinet / Versa / Palo Alto Prisma / ...] |
| Versión de software | [X.X.X] |
| Proveedor / Partner | [Nombre] |
| Número de sites activos | [N] |
| Control plane (vManage/Orchestrator) | [Cloud / On-premise] |
| Fecha de despliegue | YYYY-MM-DD |
| Soporte hasta | YYYY-MM-DD |

---

## 2. Arquitectura del Control Plane

### 2.1 Componentes SD-WAN

```
  ┌─────────────────────────────────────────────────────────────┐
  │                  SD-WAN CONTROL PLANE                       │
  │                                                             │
  │   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
  │   │  vManage /  │   │  vSmart /   │   │  vBond /    │     │
  │   │ Orchestrator│   │  Controller │   │  Validator  │     │
  │   │ (GUI + API) │   │ (Políticas) │   │(Autenticac.)│     │
  │   └──────┬──────┘   └──────┬──────┘   └──────┬──────┘     │
  └──────────┼─────────────────┼─────────────────┼────────────┘
             │                 │                  │
   NETCONF/  │         OMP     │         DTLS     │
   RESTCONF  │        (over    │                  │
             │        TLS)     │                  │
  ┌──────────┴─────────────────┴──────────────────┴────────────┐
  │                     DATA PLANE (WAN Edge)                   │
  │                                                             │
  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
  │  │vEdge/cE │   │vEdge/cE │   │vEdge/cE │   │vEdge/cE │   │
  │  │  HQ-01  │   │  DC-01  │   │ Site-A  │   │ Site-B  │   │
  │  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘   │
  └───────┼─────────────┼─────────────┼──────────────┼────────┘
          │             │             │              │
      MPLS│         MPLS│         Internet        LTE
     + Internet    + Internet
```

### 2.2 Inventario de Controladores

| Componente | Hostname/IP | Ubicación | HA / Redundancia | Versión |
|-----------|-------------|-----------|-----------------|---------|
| vManage Primary | [hostname] | [Cloud/DC] | Active/Active | [X.X] |
| vManage Secondary | [hostname] | [Cloud/DC] | Active/Active | [X.X] |
| vSmart-01 | [hostname] | [Cloud/DC] | Active/Standby | [X.X] |
| vSmart-02 | [hostname] | [Cloud/DC] | Active/Standby | [X.X] |
| vBond | [hostname] | [Cloud/DC] | Single / HA | [X.X] |

**Acceso a consola de gestión:** [URL vManage]  
**API endpoint:** `https://[vManage-IP]/dataservice`

### 2.3 Inventario de WAN Edge Devices

| Hostname | Site | Modelo | Versión | Transports | Estado |
|----------|------|--------|---------|-----------|--------|
| WE-HQ-01 | HQ | [Modelo] | [X.X] | MPLS, Internet | ✅ Activo |
| WE-HQ-02 | HQ | [Modelo] | [X.X] | MPLS, Internet | ✅ Activo |
| WE-DC-01 | DC Principal | [Modelo] | [X.X] | MPLS, Internet | ✅ Activo |
| WE-OFF-[X] | [Site] | [Modelo] | [X.X] | Internet, LTE | ✅ Activo |

---

## 3. Transports y Topología de Data Plane

### 3.1 Colors / Transports Definidos

| Color SD-WAN | Tipo de enlace | Proveedor | BW típico | Sites |
|-------------|----------------|-----------|-----------|-------|
| `mpls` | MPLS L3VPN | [Proveedor] | [X] Mbps | HQ, DCs, Tier-1 Sites |
| `internet` | Internet broadband | [ISP(s)] | [X] Mbps | Todos |
| `lte` | 4G/LTE | [Operador] | [X] Mbps | Sites Tier 2-3 (backup) |
| `biz-internet` | Internet dedicado | [ISP] | [X] Mbps | HQ, DCs |

### 3.2 Topología de Túneles (Overlay)

| Topología | Configuración | Aplicación |
|-----------|--------------|-----------|
| Full mesh | HQ ↔ DCs | Sites críticos, latencia mínima |
| Hub & Spoke | Offices → HQ/DC | Sites standard |
| On-demand tunnels | Spoke ↔ Spoke | Tráfico directo cuando se requiere |

---

## 4. Políticas de Tráfico (Application-Aware Routing)

### 4.1 SLA Classes Definidas

| SLA Class | Latencia máx. | Jitter máx. | Pérdida máx. | Aplicaciones |
|-----------|--------------|-------------|-------------|-------------|
| `REALTIME` | [X] ms | [X] ms | [X]% | VoIP, Teams Audio/Video |
| `BUSINESS-CRITICAL` | [X] ms | [X] ms | [X]% | SAP, ERP, bases de datos |
| `INTERACTIVE` | [X] ms | — | [X]% | RDP, VDI, Citrix |
| `BEST-EFFORT` | — | — | — | Backups, actualizaciones |

### 4.2 Application Policies

| Aplicación / NBAR | SLA Class | Path preferente | Fallback | Acción si no hay path |
|------------------|-----------|----------------|----------|----------------------|
| `Microsoft Teams` | `REALTIME` | `mpls` | `internet` | Degradar calidad |
| `SAP NetWeaver` | `BUSINESS-CRITICAL` | `mpls` | `biz-internet` | Mantener sesión |
| `Office 365` | `INTERACTIVE` | `internet` (DIA) | `mpls` | Redirigir a cloud GW |
| `RDP / Citrix` | `INTERACTIVE` | `mpls` | `internet` | Mantener sesión |
| `YouTube / Netflix` | `BEST-EFFORT` | `internet` | `lte` | Bloquear si aplica |
| `Backup Veeam` | `BEST-EFFORT` | `internet` o `mpls` nocturno | — | Throttle |

### 4.3 Direct Internet Access (DIA) — Local Breakout

| Site Tier | DIA habilitado | Tráfico con DIA | Inspección |
|-----------|---------------|----------------|------------|
| Tier 1 (HQ/DC) | ✅ Sí | O365, SaaS, updates | Firewall local [Modelo] |
| Tier 2 (Offices) | ✅ Sí | O365, SaaS | [Zscaler / Cloud Firewall] |
| Tier 3 (Pequeños) | ⚠️ Hairpin a HQ | Todo | Firewall HQ |

---

## 5. Seguridad SD-WAN

| Control | Implementación | Notas |
|---------|---------------|-------|
| Autenticación dispositivos | Certificados x509 (vBond) | Zero-Touch Provisioning habilitado |
| Cifrado data plane | IPSec AES-256-GCM | IKEv2 |
| Segmentación | VPNs SD-WAN (equivalente VRF) | VPN 1=Prod, VPN 512=Mgmt |
| Firewall integrado | Zone-Based Firewall en WAN Edge | Políticas centralizadas en vManage |
| URL Filtering | [Plataforma] integrada / Zscaler | Aplicado en DIA |
| Cloud Security | [Zscaler / Umbrella / ...] | Para tráfico SaaS desde DIA |

### 5.1 VPN Segments SD-WAN

| VPN ID | Nombre | Uso | VRF equivalente |
|--------|--------|-----|----------------|
| 0 | Transport | Control plane | — |
| 1 | Corporate | Producción corporativa | CORP-PROD |
| 2 | Guest | WiFi invitados | CORP-GUEST |
| 512 | Management | Gestión OOB | CORP-MGMT |

---

## 6. Zero-Touch Provisioning (ZTP)

### 6.1 Flujo de ZTP

```
1. Nuevo WAN Edge arranca
   ↓
2. Contacta vBond (DNS: sdwan-vbond.empresa.com)
   ↓
3. vBond valida certificado del dispositivo
   ↓
4. vBond redirige a vSmart y vManage
   ↓
5. vManage aplica template de configuración por site
   ↓
6. Dispositivo operativo (~ 15-30 minutos)
```

### 6.2 Templates de Configuración por Tipo de Site

| Template | Aplica a | Características |
|----------|----------|----------------|
| `TMP-TIER1-HQ` | HQ y DCs | Dual transport (MPLS+Internet), full mesh, firewall avanzado |
| `TMP-TIER2-OFFICE` | Oficinas > [N] usuarios | Dual transport, Hub & Spoke, DIA |
| `TMP-TIER3-SMALL` | Sites pequeños | Internet + LTE backup, hairpin |

---

## 7. Procedimientos de Operación

### 7.1 Onboarding de Nuevo Site

```bash
# Pasos en vManage:
# 1. Crear Device Template para el nuevo site
# 2. Asignar configuración (variables: IP WAN, site-ID, etc.)
# 3. Activar ZTP o push manual de configuración
# 4. Verificar en Dashboard: site online y tunnels establecidos

# Verificación desde CLI del WAN Edge:
show sdwan control connections           # Estado conexiones con controladores
show sdwan bfd sessions                  # Túneles BFD con otros sites
show sdwan omp routes                    # Rutas recibidas por OMP
show sdwan policy from-vsmart            # Políticas aplicadas
```

### 7.2 Verificación de Estado

```bash
# En WAN Edge (CLI):
show sdwan control connections summary
show sdwan bfd sessions summary
show sdwan app-route stats

# En vManage (API REST):
GET /dataservice/device/vedgeinventory/detail?deviceId=[DEVICE-UUID]
GET /dataservice/device/tunnels/summary
```

---

## 8. Integración con Otros Sistemas

| Sistema | Integración | Propósito |
|---------|-------------|----------|
| [Zabbix / SolarWinds] | SNMP + Streaming Telemetry | Monitorización métricas |
| Splunk / SIEM | Syslog desde vManage | Auditoría y seguridad |
| ServiceNow | API (via script) | Apertura automática de tickets |
| Azure Monitor | Streaming desde vManage | Correlación con cloud |

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
