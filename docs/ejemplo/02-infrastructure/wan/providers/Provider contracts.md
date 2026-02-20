# Contratos y Proveedores WAN

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Inventario centralizado de todos los proveedores de conectividad WAN: contratos, Circuit IDs, contactos operacionales y de escalado. Es el documento de primera consulta durante cualquier incidente con proveedor.

> ⚠️ **Información sensible:** Los precios, cláusulas contractuales y datos de facturación se guardan en SharePoint Legal: [URL]. Este documento contiene solo la información operacional necesaria para operar y escalar incidencias.

---

## 1. Resumen de Proveedores Activos

| ID | Proveedor | Servicios | Nº Circuitos | Contrato vigente hasta | Account Manager |
|----|-----------|-----------|-------------|----------------------|----------------|
| PRV-001 | [Proveedor MPLS A] | MPLS L3VPN | [N] | YYYY-MM-DD | [Nombre / email] |
| PRV-002 | [ISP Internet B] | Internet dedicado | [N] | YYYY-MM-DD | [Nombre / email] |
| PRV-003 | [SD-WAN Vendor] | SD-WAN managed | Global | YYYY-MM-DD | [Nombre / email] |
| PRV-004 | [Operador LTE] | LTE 4G backup | [N] SIMs | YYYY-MM-DD | [Nombre / email] |
| PRV-005 | [ISP Local País X] | Internet broadband | [N] | YYYY-MM-DD | [Nombre / email] |

---

## 2. Inventario Detallado de Circuitos

### 2.1 Circuitos MPLS

| CID | Proveedor | Site A | Site B | Tipo | Velocidad | VLAN/VC-ID | Router CE | IP PE | IP CE | Estado |
|-----|-----------|--------|--------|------|-----------|-----------|-----------|-------|-------|--------|
| [CID-001] | PRV-001 | HQ | DC-Principal | MPLS L3VPN | [X] Gbps | [N] | CE-HQ-01 | [X.X.X.X] | [X.X.X.X] | ✅ Activo |
| [CID-002] | PRV-001 | HQ | DC-DR | MPLS L3VPN | [X] Gbps | [N] | CE-HQ-02 | [X.X.X.X] | [X.X.X.X] | ✅ Activo |
| [CID-003] | PRV-001 | DC-Principal | Site-A | MPLS L3VPN | [X] Mbps | [N] | CE-SiteA-01 | [X.X.X.X] | [X.X.X.X] | ✅ Activo |

> IPs completas en IPAM: [URL NetBox/Infoblox]

### 2.2 Circuitos Internet

| CID | Proveedor | Site | Tipo | Velocidad | IP pública(s) | ASN proveedor | BGP peer IP | Estado |
|-----|-----------|------|------|-----------|--------------|--------------|------------|--------|
| [CID-010] | PRV-002 | HQ | Fibra dedicada | [X] Gbps | [X.X.X.0/28] | [AS-XXXXX] | [X.X.X.X] | ✅ Activo |
| [CID-011] | PRV-002 | HQ | Fibra dedicada (backup) | [X] Gbps | [X.X.X.0/30] | [AS-XXXXX] | [X.X.X.X] | ✅ Activo |
| [CID-012] | PRV-002 | DC-Principal | Fibra dedicada | [X] Gbps | [X.X.X.0/28] | [AS-XXXXX] | [X.X.X.X] | ✅ Activo |
| [CID-020] | PRV-005 | Site-A | FTTH/Cable | [X] Mbps | Dinámica | — | — | ✅ Activo |

### 2.3 Circuitos LTE / Backup Móvil

| SIM ID | Operador | Site | Dispositivo | Nº tarjeta SIM | APN | IP asignada | Datos/mes | Estado |
|--------|----------|------|-------------|---------------|-----|------------|----------|--------|
| SIM-001 | PRV-004 | Site-B | WAN Edge WE-SiteB-01 | [ICCID] | [APN corporativo] | Dinámica | [X] GB | ✅ Activo |
| SIM-002 | PRV-004 | Site-C | WAN Edge WE-SiteC-01 | [ICCID] | [APN corporativo] | Dinámica | [X] GB | ✅ Activo |

---

## 3. Contactos Operacionales por Proveedor

> 🔑 **Uso:** Estos contactos son para incidencias en producción. Para cambios contractuales, contactar siempre al Procurement / Legal.

### PRV-001 — [Proveedor MPLS A]

| Tipo de contacto | Nombre | Teléfono | Email | Disponibilidad |
|-----------------|--------|----------|-------|---------------|
| NOC (apertura de averías) | — | [+X XXX XXX XXX] | [noc@proveedor.com] | 24×7 |
| Account Manager técnico | [Nombre] | [+X XXX XXX XXX] | [email] | Laborables 09-18h |
| Escalado Nivel 2 | [Nombre / Cargo] | [+X XXX XXX XXX] | [email] | 24×7 (incid. críticos) |
| Escalado Nivel 3 (Management) | [Nombre / Cargo] | [+X XXX XXX XXX] | [email] | Según urgencia |

**Portal de cliente:** [URL portal NOC]  
**ID de cliente en portal:** [ID]  
**Formato de ticket:** `INC-XXXXXXX`

**Proceso de apertura de avería:**
1. Llamar al NOC: [+X XXX XXX XXX]
2. Facilitar: Circuit ID (CID), descripción del problema, contacto de retorno
3. Solicitar número de ticket y SLA comprometido
4. Si no hay resolución en [X] horas → escalar a Account Manager técnico

---

### PRV-002 — [ISP Internet B]

| Tipo de contacto | Nombre | Teléfono | Email | Disponibilidad |
|-----------------|--------|----------|-------|---------------|
| NOC averías | — | [+X XXX XXX XXX] | [noc@isp.com] | 24×7 |
| Account Manager técnico | [Nombre] | [+X XXX XXX XXX] | [email] | Laborables 09-18h |
| Escalado | [Nombre] | [+X XXX XXX XXX] | [email] | 24×7 críticos |

**Portal de cliente:** [URL]  
**ID de cliente:** [ID]

---

### PRV-003 — [SD-WAN Vendor / Managed Service]

| Tipo de contacto | Nombre | Teléfono | Email | Disponibilidad |
|-----------------|--------|----------|-------|---------------|
| TAC (soporte técnico) | — | [+X XXX XXX XXX] | [tac@vendor.com] | 24×7 |
| Customer Success Manager | [Nombre] | [+X XXX XXX XXX] | [email] | Laborables |
| Escalado crítico P1 | — | [+X XXX XXX XXX] | [email] | 24×7 |

**Portal TAC:** [URL]  
**Número de contrato soporte:** [N]  
**SLA soporte P1:** Respuesta < [X] horas, resolución < [X] horas

---

### PRV-004 — [Operador LTE]

| Tipo de contacto | Nombre | Teléfono | Email | Disponibilidad |
|-----------------|--------|----------|-------|---------------|
| Soporte empresas | — | [+X XXX XXX XXX] | [b2b@operador.com] | Laborables 08-20h |
| Account Manager | [Nombre] | [+X XXX XXX XXX] | [email] | Laborables |

**Portal de gestión SIMs:** [URL]  
**ID cuenta empresarial:** [ID]

---

## 4. Matriz de Escalado por Severidad

| Severidad | Definición | Tiempo máx. respuesta proveedor | Escalado interno si no responde |
|-----------|-----------|--------------------------------|--------------------------------|
| **P1 — Crítico** | Site HQ/DC sin conectividad | 30 minutos | @wan-lead inmediatamente |
| **P2 — Alto** | Site spoke sin conectividad o degradación >50% HQ | 1 hora | @wan-lead en 1h |
| **P3 — Medio** | Degradación parcial o site spoke degradado | 4 horas | @wan-lead si no hay respuesta en 4h |
| **P4 — Bajo** | Problema no urgente, planificación, consulta | 1 día hábil | Seguimiento a 2 días |

**Proceso de escalado interno si el proveedor no responde:**

```
Nivel 1 (0–30 min): @network-engineer que detecta el problema
Nivel 2 (30–60 min): @wan-lead
Nivel 3 (60–120 min): @network-director + notificación a @sre-team
Nivel 4 (> 2h, P1): @cto-office + Plan de continuidad
```

---

## 5. Histórico de Circuitos Retirados

| CID | Proveedor | Site | Fecha baja | Motivo | Reemplazado por |
|-----|-----------|------|-----------|--------|----------------|
| [CID-OLD] | [Proveedor] | [Site] | YYYY-MM-DD | [Motivo] | [CID nuevo o N/A] |

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial del inventario |
