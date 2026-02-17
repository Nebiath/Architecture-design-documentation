# Conectividad Internet — Diseño y Política de Acceso

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Diseño de la conectividad a internet corporativa: circuitos, modelo de salida (centralizada vs. DIA), proxies, filtrado y redundancia.

---

## 1. Inventario de Circuitos Internet

| ID | Site | ISP | Tipo | BW Contratado | BW Burst | IP pública(s) | CID | Estado |
|----|------|-----|------|--------------|---------|--------------|-----|--------|
| INT-001 | HQ | [ISP-A] | Fibra dedicada | [X] Gbps | [X] Gbps | [Rango /28] | [CID] | ✅ Activo |
| INT-002 | HQ | [ISP-B] | Fibra dedicada | [X] Gbps | — | [Rango /30] | [CID] | ✅ Activo (Backup) |
| INT-003 | DC Principal | [ISP-A] | Fibra dedicada | [X] Gbps | — | [Rango /28] | [CID] | ✅ Activo |
| INT-004 | Site-A | [ISP Local] | FTTH/Cable | [X] Mbps | — | [IP dinámica] | [CID] | ✅ Activo |

> IPs exactas y rangos en IPAM: [URL]

---

## 2. Modelo de Salida a Internet

### 2.1 Arquitectura por Tier de Site

```
 ┌─────────────────────────────────────────────────────────────┐
 │  TIER 1 (HQ/DC): Salida directa dual-ISP con BGP           │
 │                                                             │
 │  Internet ─── ISP-A ─── FW-Primary ─┐                     │
 │                                      ├── Core Switch ── LAN│
 │  Internet ─── ISP-B ─── FW-Secondary┘                     │
 └─────────────────────────────────────────────────────────────┘

 ┌─────────────────────────────────────────────────────────────┐
 │  TIER 2 (Oficinas): Direct Internet Access (DIA) via SD-WAN│
 │                                                             │
 │  Internet ─── ISP Local ─── WAN Edge ─── LAN              │
 │                               (SD-WAN                       │
 │                              con DIA policy)                │
 └─────────────────────────────────────────────────────────────┘

 ┌─────────────────────────────────────────────────────────────┐
 │  TIER 3 (Sites pequeños): Hairpin vía HQ / Cloud Proxy      │
 │                                                             │
 │  LAN ─── WAN Edge ─── SD-WAN Overlay ─── HQ Internet       │
 │                                                             │
 └─────────────────────────────────────────────────────────────┘
```

### 2.2 Política de Direct Internet Access (DIA)

| Tráfico / Aplicación | Tier 1 (HQ/DC) | Tier 2 (Offices) | Tier 3 (Small) |
|---------------------|----------------|-----------------|----------------|
| Microsoft 365 / Teams | DIA directo | DIA directo | Hairpin HQ |
| SaaS corporativo (Salesforce, Workday...) | DIA directo | DIA directo | Hairpin HQ |
| Google (Workspace, YouTube) | DIA directo | DIA con filtrado | Hairpin HQ |
| Navegación general | DIA con proxy | DIA con Cloud Proxy | Hairpin HQ |
| Backups cloud | DIA con throttling | DIA con throttling | — |
| Acceso desconocido / bloqueado | FW deny | Cloud Proxy deny | FW HQ deny |

---

## 3. Filtrado y Seguridad de Navegación

### 3.1 Plataforma de Filtrado Web

| Capa | Herramienta | Modo | Sites aplicables |
|------|-------------|------|----------------|
| Proxy / URL Filtering | [Zscaler / Umbrella / Proxy Squid / ...] | [Explícito / Transparente / Cloud] | Todos |
| DNS Filtering | [Cisco Umbrella / Zscaler / Pi-hole Enterprise] | DNS sobre HTTPS o redirect | Todos |
| Firewall perimetral | [Palo Alto / Fortinet / Check Point] | Stateful + App-ID | HQ, DCs |
| SSL Inspection | [Plataforma] | Intercepción TLS 1.3 | HQ, DCs, Tier 2 |

### 3.2 Categorías de URL — Política Global

| Categoría | Permitido | Bloqueado | Excepciones |
|-----------|-----------|-----------|------------|
| Redes sociales | Solo lectura (no upload) | Upload / streaming personal | Equipo Marketing |
| Streaming video (YouTube, Netflix) | ✅ Laboral | ⛔ Personal / entretenimiento | — |
| Almacenamiento cloud personal (Dropbox personal, etc.) | ⛔ | ⛔ | Solo corporativo |
| Torrents / P2P | ⛔ | ✅ siempre bloqueado | — |
| Malware / Phishing (categoría automática) | ⛔ | ✅ siempre bloqueado | — |
| Juegos online | ⛔ | ⛔ | — |

> Política completa de uso aceptable: [Link a política corporativa]

---

## 4. BGP hacia ISPs (Tier 1 Sites)

### 4.1 Configuración BGP Multi-homed

| Parámetro | ISP-A (Primario) | ISP-B (Backup) |
|-----------|-----------------|----------------|
| AS ISP | [AS-XXXXX] | [AS-YYYYY] |
| IP Peer BGP | [X.X.X.X] | [Y.Y.Y.Y] |
| Prefijo anunciado | [IP-PUBLICA/28] | [IP-PUBLICA/30] |
| MED outbound | 100 | 200 |
| Local Preference inbound | 200 | 100 |
| BFD | Sí ([X] ms) | Sí ([X] ms) |
| Communities recibidas | [Lista] | [Lista] |

### 4.2 Estrategia de Failover Internet

| Evento | Detección | Failover | Tiempo estimado |
|--------|-----------|---------|----------------|
| Caída enlace físico ISP-A | Interface down | BGP withdraw / ruta estática elimina | < 1 seg |
| Degradación ISP-A (no caída) | BFD / IP SLA probe | Manualmente o SLA-based routing | Manual o 30 seg |
| Fallo total HQ Internet | — | SD-WAN redirige sites a DIA local | < [X] seg |

---

## 5. Monitorización de Conectividad Internet

### 5.1 Métricas y Umbrales

| Métrica | Warning | Critical | Herramienta |
|---------|---------|----------|-------------|
| Utilización enlace internet | > 70% | > 85% | SNMP / Grafana |
| BGP session down ISP | — | Inmediato | Zabbix |
| Latencia a 8.8.8.8 (o probe externo) | > [X] ms | > [X] ms | ICMP probe |
| Pérdida paquetes externo | > 1% | > 5% | ICMP probe |
| DNS resolution failure rate | > 1% | > 5% | DNS monitoring |
| Disponibilidad M365 endpoints | < 99,9% | < 99% | [Microsoft 365 Health] |

### 5.2 Dashboard

- **Dashboard Internet:** [URL Grafana]
- **Estado M365:** https://status.office.com
- **Alertas PagerDuty:** filtro `internet-connectivity`

---

## 6. IPs Públicas y NAT

### 6.1 Bloques de IPs Públicas

| Bloque | ISP | Uso | Sites |
|--------|-----|-----|-------|
| [X.X.X.0/28] | ISP-A | NAT corporativo HQ + Servicios expuestos | HQ |
| [X.X.X.0/30] | ISP-B | NAT failover HQ | HQ |
| [X.X.X.0/28] | ISP-A | NAT DC Principal | DC-001 |

> Asignación detallada IP por IP en IPAM: [URL]

### 6.2 Política de Publicación de Servicios (Inbound NAT)

| Servicio publicado | IP pública | Puerto | Destino interno | Firewall rule ID |
|-------------------|------------|--------|----------------|-----------------|
| [Servicio A] | [X.X.X.X] | [443] | [IP interna:443] | FW-RULE-[N] |
| [Servicio B (API)] | [X.X.X.X] | [443] | [IP interna:443] | FW-RULE-[N] |

> ⚠️ Toda publicación de servicio a internet requiere aprobación de @security-team

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
