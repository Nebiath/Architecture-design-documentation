# VPN Site-to-Site — Conectividad entre Sedes y Cloud

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Diseño y configuración de túneles VPN IPSec site-to-site: entre sedes corporativas, hacia proveedores cloud (Azure / AWS) y hacia socios/partners.

---

## 1. Inventario de Túneles VPN

### 1.1 VPN Corporativos (Site-to-Site)

| ID | Nombre | Origen | Destino | Tecnología | BW | Estado |
|----|--------|--------|---------|-----------|-----|--------|
| VPN-001 | HQ → DC-DR | HQ ([IP GW]) | DC-DR ([IP GW]) | IPSec IKEv2 | [X] Mbps | ✅ Activo |
| VPN-002 | HQ → Site-A | HQ ([IP GW]) | Site-A ([IP GW]) | IPSec IKEv2 | [X] Mbps | ✅ Activo |
| VPN-003 | HQ → Azure vNet | HQ ([IP GW]) | Azure VPN GW | IPSec IKEv2 | [X] Mbps | ✅ Activo |
| VPN-004 | HQ → AWS VPC | HQ ([IP GW]) | AWS VGW | IPSec IKEv2 | [X] Mbps | ✅ Activo |

> IPs exactas en IPAM: [URL]

### 1.2 VPN con Socios / Extranet

| ID | Socio / Partner | Dirección IP Remota | Subredes remotas | Propósito | Responsable negocio |
|----|----------------|--------------------|-----------------|-----------|--------------------|
| EXT-001 | [Partner A] | [IP pública] | [X.X.X.0/24] | [EDI / API / Acceso sistemas] | [Nombre] |
| EXT-002 | [Partner B] | [IP pública] | [X.X.X.0/24] | [Descripción] | [Nombre] |

---

## 2. Parámetros de Cifrado

### 2.1 Perfil de Criptografía Estándar (IKEv2 / IPSec)

| Parámetro | Valor estándar corporativo |
|-----------|--------------------------|
| **IKE Phase 1** | |
| Versión IKE | IKEv2 |
| Algoritmo cifrado | AES-256-GCM |
| Autenticación | SHA-384 / SHA-256 |
| Grupo Diffie-Hellman | Group 20 (ECDH 384-bit) / Group 14 (mínimo) |
| Lifetime | 86.400 segundos (24 h) |
| Autenticación peers | PSK / Certificados x509 |
| **IKE Phase 2 (IPSec)** | |
| Protocolo | ESP |
| Algoritmo cifrado | AES-256-GCM |
| PFS (Perfect Forward Secrecy) | Habilitado — Grupo 20 |
| Lifetime | 3.600 segundos (1 h) / [X] GB |
| Mode | Tunnel |

> ⚠️ Las PSK se almacenan en CyberArk / Azure Key Vault. **Nunca en este repositorio.**

### 2.2 Excepciones al Estándar

| Túnel | Excepción | Justificación | Aprobado por | Revisión |
|-------|-----------|---------------|-------------|----------|
| EXT-001 | AES-128 / SHA-1 | Partner legacy no soporta AES-256 | [Nombre] | YYYY-MM-DD |

---

## 3. Diseño de Alta Disponibilidad VPN

### 3.1 Topología de Redundancia

```
     SEDE ORIGEN                           SEDE DESTINO
  ┌──────────────┐                      ┌──────────────┐
  │  FW/GW-A     │══IPSec Tunnel 1═════▶│  FW/GW-A     │
  │  (Activo)    │                      │  (Activo)    │
  └──────┬───────┘                      └──────┬───────┘
         │ HSRP/VRRP                           │ HSRP/VRRP
  ┌──────┴───────┐                      ┌──────┴───────┐
  │  FW/GW-B     │══IPSec Tunnel 2════▶ │  FW/GW-B     │
  │  (Standby)   │                      │  (Standby)   │
  └──────────────┘                      └──────────────┘
```

| Mecanismo | Descripción | Tiempo failover |
|-----------|-------------|----------------|
| IKE Dead Peer Detection (DPD) | Detecta peer caído | [X] seg (interval [X], retry [X]) |
| Dual-gateway HSRP/VRRP | Failover local | < [X] seg |
| BGP sobre túnel | Re-routing automático | < [X] seg |

---

## 4. VPN hacia Azure (Hybrid Connectivity)

### 4.1 Parámetros Azure VPN Gateway

| Parámetro | Valor |
|-----------|-------|
| Subscription ID | [XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX] |
| Resource Group | [RG-name] |
| VNet Name | [VNet-name] |
| Región Azure | [West Europe / East US / ...] |
| VPN Gateway SKU | VpnGw2 / VpnGw3 / ... |
| BGP ASN Azure | 65515 |
| BGP peer IP Azure | [X.X.X.X] |
| Active-Active Gateway | Sí / No |

### 4.2 Local Network Gateway (lado on-premises)

| LNG Name | IP pública on-prem | Subredes on-prem anunciadas |
|----------|-------------------|----------------------------|
| LNG-HQ | [IP pública GW HQ] | [X.X.0.0/16], [Y.Y.0.0/16] |
| LNG-DC | [IP pública GW DC] | [X.X.0.0/16] |

### 4.3 Rutas Anunciadas vía BGP

| Dirección | Prefijos anunciados | Via |
|-----------|--------------------|----|
| On-prem → Azure | [Subredes corporativas] | BGP sobre IPSec |
| Azure → On-prem | [VNet ranges] | BGP sobre IPSec |

---

## 5. Monitorización VPN

### 5.1 Métricas y Umbrales

| Métrica | Warning | Critical | Herramienta / Query |
|---------|---------|----------|---------------------|
| Túnel IPSec down | — | Inmediato | SNMP / Syslog trigger |
| IKE negotiation failures | > 5/hora | > 20/hora | Syslog parsing |
| Throughput túnel < esperado | < 80% BW nominal | < 50% BW nominal | SNMP ifHCInOctets |
| Latencia end-to-end | > [X] ms | > [X] ms | ICMP Probe |
| Expiración de cert (para cert-based VPN) | 30 días | 7 días | Script de auditoría |

### 5.2 Dashboard

- **Dashboard VPN:** [URL Grafana/Zabbix/SolarWinds]
- **Alertas activas:** [URL PagerDuty filtro VPN]

---

## 6. Procedimientos de Operación

### 6.1 Alta de Nuevo Túnel VPN

```
Pre-requisitos:
1. Obtener: IP pública remota, subredes remotas, parámetros cripto acordados
2. Solicitar PSK a CyberArk o acordar uso de certificados
3. Crear ticket de cambio en ServiceNow (CAT: Network Change)

Pasos de configuración (adaptar a plataforma):
1. Configurar Phase 1 (ISAKMP/IKEv2 policy)
2. Configurar Phase 2 (Transform-set / IPSec profile)
3. Crear crypto map o VTI
4. Configurar routing (estático/BGP sobre túnel)
5. Verificar con contraparte: ping end-to-end
6. Actualizar este documento y el IPAM
7. Notificar a solicitante y a @network-team
```

### 6.2 Baja de Túnel VPN

```
1. Confirmar con responsable de negocio que se puede eliminar
2. Revisar si hay rutas dependientes del túnel
3. Eliminar configuración en ambos extremos (coordinar con contraparte)
4. Eliminar entradas de ACL y NAT si aplica
5. Actualizar inventario en este documento
6. Mover a sección "Túneles retirados"
7. Archivar PSK como expirada en CyberArk
```

---

## 7. Túneles Retirados (Histórico)

| ID | Nombre | Fecha baja | Motivo | Archivado por |
|----|--------|-----------|--------|--------------|
| — | — | — | — | — |

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
