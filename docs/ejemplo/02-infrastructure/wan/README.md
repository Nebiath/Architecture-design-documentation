# WAN — Red de Área Amplia

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Aprobado  
> **Versión:** 1.0

Documentación técnica completa del departamento WAN: arquitectura, conectividad, proveedores y procedimientos operacionales para toda la red de área amplia corporativa.

---

## 📋 Índice de Documentos

### 🏗️ Arquitectura

| Documento | Descripción | Estado |
|-----------|-------------|--------|
| [Topología WAN](./architecture/wan-topology.md) | Diseño global de la red WAN, diagrama de sites y conectividad | ⬜ Draft |
| [Diseño MPLS](./architecture/mpls-design.md) | Arquitectura y configuración de la red MPLS corporativa | ⬜ Draft |
| [Arquitectura SD-WAN](./architecture/sd-wan-architecture.md) | Diseño y políticas de la solución SD-WAN | ⬜ Draft |

### 🔌 Conectividad

| Documento | Descripción | Estado |
|-----------|-------------|--------|
| [VPN Site-to-Site](./connectivity/site-to-site-vpn.md) | Túneles VPN entre sedes y hacia cloud | ⬜ Draft |
| [Conectividad Internet](./connectivity/internet-connectivity.md) | Acceso a internet, breakout local y centralizado | ⬜ Draft |
| [Diseño de Redundancia](./connectivity/redundancy-design.md) | Alta disponibilidad, failover y balanceo de carga WAN | ⬜ Draft |

### 🤝 Proveedores

| Documento | Descripción | Estado |
|-----------|-------------|--------|
| [Contratos y Proveedores](./providers/provider-contracts.md) | Inventario de proveedores, CIDs, contratos y contactos | ⬜ Draft |
| [Gestión de SLAs](./providers/sla-management.md) | SLAs contratados, medición, penalizaciones y reporting | ⬜ Draft |

### 📘 Runbooks Operacionales

| Documento | Descripción | Estado |
|-----------|-------------|--------|
| [Provisioning de Circuitos](./runbooks/circuit-provisioning.md) | Procedimiento de alta, cambio y baja de circuitos WAN | ⬜ Draft |
| [Troubleshooting WAN](./runbooks/wan-troubleshooting.md) | Diagnóstico y resolución de incidencias WAN | ⬜ Draft |

> **Estados:** ⬜ Draft · 🔄 En revisión · ✅ Aprobado · ⚠️ Desactualizado

---

## 👥 Equipo y Contactos

| Rol | Nombre | Contacto | Responsabilidad |
|-----|--------|----------|----------------|
| WAN Lead | [Nombre] | [email] | Responsable técnico y arquitectura |
| Network Engineer Senior | [Nombre] | [email] | Diseño y configuración |
| Network Engineer | [Nombre] | [email] | Operaciones y monitorización |
| On-Call Primary | [Rotación] | PagerDuty | Guardia 24×7 |
| On-Call Backup | [Rotación] | PagerDuty | Escalado nivel 2 |

**Canales de contacto:**
- 💬 Teams: `#network-wan`
- 📧 Email equipo: `network-team@empresa.com`
- 🚨 Incidentes: PagerDuty — Servicio `WAN-Infrastructure`

---

## 🌍 Inventario de Sites WAN

| Site | País | Código | Tipo | Estado |
|------|------|--------|------|--------|
| [Site A] | [País] | [COD] | HUB / Spoke | ✅ Activo |
| [Site B] | [País] | [COD] | Spoke | ✅ Activo |
| [Site C] | [País] | [COD] | Spoke | ✅ Activo |

> Lista completa en [Topología WAN](./architecture/wan-topology.md)

---

## 🔭 Monitorización Rápida

| Sistema | URL | Qué monitoriza |
|---------|-----|----------------|
| Dashboard WAN | [URL Grafana/Zabbix] | Disponibilidad y latencia por circuito |
| SD-WAN Console | [URL] | Estado de túneles y políticas de tráfico |
| Alertas activas | [URL PagerDuty] | Incidentes WAN en curso |

---

## 📐 Stack Tecnológico WAN

| Capa | Tecnología | Versión | Proveedor |
|------|-----------|---------|-----------|
| MPLS Backbone | [Cisco/Juniper/...] | [ver.] | [Proveedor] |
| SD-WAN | [Cisco Viptela / VMware VeloCloud / ...] | [ver.] | [Proveedor] |
| Edge Routers | [Modelo] | [ver. IOS] | [Fabricante] |
| VPN Gateway | [Plataforma] | [ver.] | [Proveedor] |
| Monitorización | [Zabbix / SolarWinds / ...] | [ver.] | — |

---

## 📊 KPIs del Departamento

| Métrica | SLA Objetivo | Medición | Dashboard |
|---------|-------------|----------|-----------|
| Disponibilidad WAN global | 99,95% mensual | Zabbix/SNMP | [Link] |
| Latencia HQ → Site principal | < [X] ms | ICMP/TWAMP | [Link] |
| Pérdida de paquetes | < 0,1% | TWAMP | [Link] |
| MTTR incidente WAN | < 4 h | ServiceNow | [Link] |
| Circuitos en SLA | 100% | Proveedor | [Link] |

---

## 🔗 Documentos Relacionados (Otros Departamentos)

- [Arquitectura LAN](../lan/README.md) — Interconexión LAN-WAN en sites
- [Arquitectura Cloud Azure](../cloud-azure/networking/hybrid-connectivity.md) — ExpressRoute y VPN hacia Azure
- [DevOps — Kubernetes](../devops/README.md) — Tráfico inter-cluster sobre WAN
- [Cross-cutting: Seguridad](../../04-cross-cutting/security/encryption-standards.md) — Cifrado en tránsito
- [Cross-cutting: Monitorización](../../04-cross-cutting/monitoring/monitoring-strategy.md) — Estrategia global de monitoring

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial del README del departamento |
