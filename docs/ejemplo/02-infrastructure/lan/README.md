# LAN — Red de Área Local

> **Propietario:** @network-team @lan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Versión:** 1.0

Documentación completa del departamento LAN: diseño de campus, switching, wireless, segmentación, servicios de red y procedimientos operacionales.

**Stack principal:** Cisco Catalyst switches, Cisco DNA Center, Aruba/Cisco wireless

---

## 📋 Documentos

### 🏗️ Arquitectura
- [Campus Network Design](./architecture/campus-network-design.md) — Topología global, core-distribution-access
- [Switching Architecture](./architecture/switching-architecture.md) — VLANs, STP, port-channels, VSS/StackWise
- [Wireless Architecture](./architecture/wireless-architecture.md) — Controllers, APs, SSIDs, roaming

### 🔀 Segmentación
- [VLAN Strategy](./segmentation/vlan-strategy.md) — Diseño de VLANs corporativas, tagging, trunking
- [Subnet Allocation](./segmentation/subnet-allocation.md) — Esquema de direccionamiento IP, IPAM
- [Access Control](./segmentation/access-control.md) — 802.1X, NAC, ACLs, port-security

### 🌐 Servicios
- [DHCP Design](./services/dhcp-design.md) — DHCP servers, relay, scopes, reservations
- [DNS Architecture](./services/dns-architecture.md) — DNS interno, split-brain, DNSSEC
- [NTP Configuration](./services/ntp-configuration.md) — Stratum, NTP servers, clients

### 📘 Runbooks
- [Switch Configuration](./runbooks/switch-configuration.md) — Provisioning, baseline config, backups
- [Wireless Deployment](./runbooks/wireless-deployment.md) — AP deployment, survey, troubleshooting

---

## 🌐 Inventario de Red LAN

| Site | Core Switches | Distribution | Access | Total Ports | APs |
|------|--------------|-------------|--------|-------------|-----|
| HQ | 2x C9500 | 4x C9300 | 40x C9200 | [N] | [N] |
| Site-A | — | 2x C9300 | 10x C9200 | [N] | [N] |
| **Total** | [N] | [N] | [N] | **[N]** | **[N]** |

> Inventario completo en IPAM/NetBox: [URL]

---

## 📐 Stack Tecnológico LAN

| Componente | Vendor/Modelo | Notas |
|-----------|--------------|-------|
| **Core Switches** | Cisco Catalyst 9500 | VSS o StackWise |
| **Distribution** | Cisco Catalyst 9300 | Layer 3, StackWise |
| **Access** | Cisco Catalyst 9200 / 9200L | Layer 2, StackWise |
| **Wireless Controller** | Cisco DNA Center / vWLC | Centralizado |
| **Access Points** | Cisco Catalyst 9100 / Aruba AP | Wi-Fi 6 (802.11ax) |
| **Management** | Cisco DNA Center / Prime | Config, monitoring |
| **IPAM** | NetBox / Infoblox | IP Address Management |
| **Monitoring** | PRTG / Zabbix / SolarWinds | SNMP, NetFlow |

---

## 📊 KPIs del Departamento

| Métrica | SLA Objetivo | Medición |
|---------|-------------|----------|
| Disponibilidad core switches | 99,99% | SNMP uptime |
| Disponibilidad access layer | 99,9% | SNMP uptime |
| MTTR switch failure | < 4 horas | ServiceNow |
| Cobertura Wi-Fi | > 95% del área | Site surveys |
| Wireless roaming time | < 50 ms | Controller metrics |

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
