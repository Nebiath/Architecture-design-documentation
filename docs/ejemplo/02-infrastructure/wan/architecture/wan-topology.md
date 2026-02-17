# Topología WAN — Diseño Global de Red

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Documento de referencia para la topología completa de la red WAN corporativa: sitios, tecnologías de transporte, diagramas de conectividad y modelo de direccionamiento.

---

## 1. Resumen Ejecutivo

| Campo | Valor |
|-------|-------|
| Número de sites WAN | [N] |
| Países con presencia | [N] |
| Tecnología principal de transporte | MPLS / SD-WAN / MPLS + SD-WAN |
| Ancho de banda agregado (HQ) | [X] Gbps |
| Proveedores activos | [N] |
| Topología | Hub & Spoke / Full Mesh / Partial Mesh |

**Descripción breve:**
> [2-3 frases describiendo el propósito de la red WAN y qué conecta]

---

## 2. Topología Global

### 2.1 Diagrama de Sites

```
                     ┌─────────────────────────────────────────────────┐
                     │               BACKBONE MPLS/SD-WAN              │
                     │                                                 │
         ┌───────────┴────────────┐           ┌────────────────────────┴──┐
         │     HQ — [Ciudad]      │           │    DC Secundario — [Ciudad]│
         │   [X] Gbps             │           │   [X] Gbps                │
         └───────────┬────────────┘           └────────────────────────┬──┘
                     │                                                 │
       ┌─────────────┼─────────────┐                    ┌─────────────┘
       │             │             │                    │
  ┌────┴────┐   ┌────┴────┐  ┌────┴────┐         ┌────┴────┐
  │ Site A  │   │ Site B  │  │ Site C  │         │ Site D  │
  │[País]   │   │[País]   │  │[País]   │         │[País]   │
  │[X] Mbps │   │[X] Mbps │  │[X] Mbps │         │[X] Mbps │
  └─────────┘   └─────────┘  └─────────┘         └─────────┘
```

> 📎 Diagrama de alta resolución: [`./diagrams/wan-topology-global.png`](./diagrams/wan-topology-global.png)  
> 🔄 Actualizar diagrama en [Fuente Draw.io](./diagrams/wan-topology-global.drawio)

### 2.2 Inventario Completo de Sites

| Site ID | Nombre | Ciudad | País | Región | Tipo | Tecnología | BW Contratado | Estado |
|---------|--------|--------|------|--------|------|-----------|--------------|--------|
| HQ-001 | Sede Central | [Ciudad] | [País] | EMEA | HUB | MPLS + SD-WAN | [X] Gbps | ✅ Activo |
| DC-001 | Datacenter Principal | [Ciudad] | [País] | EMEA | HUB | MPLS + SD-WAN | [X] Gbps | ✅ Activo |
| DC-002 | Datacenter DR | [Ciudad] | [País] | EMEA | HUB | MPLS | [X] Gbps | ✅ Activo |
| OFF-001 | Oficina [Ciudad] | [Ciudad] | [País] | EMEA | Spoke | SD-WAN | [X] Mbps | ✅ Activo |
| OFF-002 | Oficina [Ciudad] | [Ciudad] | [País] | APAC | Spoke | SD-WAN | [X] Mbps | ✅ Activo |
| OFF-003 | Oficina [Ciudad] | [Ciudad] | [País] | AMER | Spoke | SD-WAN + Internet | [X] Mbps | ✅ Activo |

### 2.3 Clasificación de Sites

| Tier | Criterio | Tecnología mínima | Redundancia |
|------|----------|------------------|-------------|
| **Tier 1 — Crítico** | > [N] usuarios o sistemas críticos | MPLS primario + SD-WAN backup | Dual-homed, dual proveedor |
| **Tier 2 — Estándar** | [N]–[N] usuarios | SD-WAN sobre MPLS o Internet | Single-homed, failover a LTE |
| **Tier 3 — Pequeño** | < [N] usuarios | SD-WAN sobre Internet + LTE | LTE backup |

---

## 3. Modelo de Conectividad

### 3.1 Tecnologías de Transporte por Region

| Región | Tecnología Primaria | Tecnología Backup | Justificación |
|--------|--------------------|--------------------|---------------|
| EMEA | MPLS L3VPN | SD-WAN over Internet | Cobertura de proveedor, latencia |
| APAC | SD-WAN over MPLS | SD-WAN over Internet | Variabilidad de proveedor por país |
| AMER | SD-WAN over Internet | LTE / 4G | Costes y flexibilidad |

### 3.2 Modelo de Routing

**Protocolo de routing:** BGP (eBGP con proveedor) + OSPF/EIGRP interno

```
AS [Número AS corporativo]
│
├── eBGP ──▶ Proveedor MPLS (AS [XXXXX])
│              PE-CE routing: BGP / OSPF redistribución
│
├── SD-WAN overlay routing
│              Control plane: SD-WAN controller
│              Data plane: túneles IPSec/GRE
│
└── Interconexión DC
               eBGP / iBGP según diseño
```

### 3.3 Esquema de Direccionamiento IP WAN

> ⚠️ Las IPs exactas se encuentran en la IPAM (NetBox/Infoblox/[herramienta]). Aquí solo el esquema.

| Bloque | Uso | CIDR |
|--------|-----|------|
| Loopbacks WAN routers | `/32` por dispositivo | `[X.X.X.0/24]` |
| Point-to-point MPLS CE-PE | `/30` por enlace | `[X.X.X.0/23]` |
| SD-WAN overlay tunnels | Asignado por controller | `[X.X.X.0/20]` |
| VPN site-to-site | `/30` por túnel | `[X.X.X.0/24]` |

**IPAM:** [URL NetBox / Infoblox / Excel en SharePoint]

---

## 4. Diseño de Alta Disponibilidad

### 4.1 Topología de Redundancia HQ / DCs

```
                      Internet
                      │     │
                  ┌───┘     └───┐
           ISP-A (Primary)   ISP-B (Backup)
                  │               │
           ┌──────┴──┐     ┌──────┴──┐
           │ Router  │     │ Router  │
           │ WAN-A   │     │ WAN-B   │
           └────┬────┘     └────┬────┘
                │               │
                └───────┬───────┘
                        │
                  LAN Core Switches
```

### 4.2 Mecanismos de Failover

| Escenario | Tiempo de failover | Mecanismo | Impacto |
|-----------|-------------------|-----------|---------|
| Caída circuito primario | < [X] seg | BFD + BGP | Tráfico redirige a backup |
| Caída router WAN-A | < [X] seg | HSRP/VRRP | Conmutación a WAN-B |
| Caída de site SD-WAN controller | < [X] seg | Controller HA cluster | Operación continua |
| Pérdida de enlace MPLS | < [X] seg | SD-WAN failover automático | Degradación de QoS |

---

## 5. QoS y Políticas de Tráfico

### 5.1 Clases de Servicio WAN

| Clase | DSCP | CoS | % BW Reservado | Tráfico incluido |
|-------|------|-----|----------------|-----------------|
| **Real-Time** | EF (46) | 5 | [X]% | VoIP, videoconferencia |
| **Business Critical** | AF31 (26) | 3 | [X]% | SAP, ERP, bases de datos |
| **Business** | AF21 (18) | 2 | [X]% | Email, colaboración |
| **Best Effort** | BE (0) | 0 | Resto | Navegación general |
| **Scavenger** | CS1 (8) | 1 | < [X]% | Torrents, backup no crítico |

### 5.2 Políticas SD-WAN por Aplicación

| Aplicación / Categoría | Path preferente | Path failover | SLA threshold |
|------------------------|----------------|---------------|---------------|
| Microsoft Teams / VoIP | MPLS (baja latencia) | Internet directa | Latencia < [X] ms, Jitter < [X] ms |
| SAP / ERP | MPLS | MPLS secundario | Pérdida < [X]% |
| Office 365 / SaaS | Internet Direct Breakout | MPLS | Disponibilidad > 99% |
| Backups | Internet | MPLS (si disponible) | Sin SLA estricto |

---

## 6. Seguridad WAN

| Control | Implementación | Responsable |
|---------|---------------|-------------|
| Cifrado en tránsito | IPSec AES-256 (SD-WAN) / MPLS VPN L3 | @network-team |
| Segmentación | VRFs separados por entorno (Prod/Dev/Mgmt) | @network-team |
| Firewall perimetral | [Fabricante/Modelo] en HQ y DCs | @security-team |
| IDS/IPS WAN | [Plataforma] | @security-team |
| Acceso remoto | [Solución VPN SSL / ZTNA] | @security-team |
| Gestión out-of-band | [Consolas / Terminal Servers] | @network-team |

---

## 7. Documentos Relacionados

- [Diseño MPLS](./mpls-design.md)
- [Arquitectura SD-WAN](./sd-wan-architecture.md)
- [Diseño de Redundancia](../connectivity/redundancy-design.md)
- [Contratos de Proveedores](../providers/provider-contracts.md)
- [ADR: Elección de plataforma SD-WAN](../../../06-decisions/) ← *Crear si aplica*

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
