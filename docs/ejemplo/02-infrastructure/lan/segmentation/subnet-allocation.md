# Subnet Allocation — Esquema de Direccionamiento IP

> **Propietario:** @network-team @lan-lead  
> **IPAM:** NetBox / Infoblox

## 1. Esquema de Subnetting

### 1.1 Esquema Global
| Network | CIDR | Uso | Capacity |
|---------|------|-----|----------|
| 10.0.0.0/8 | — | RFC 1918 privado (completo) | 16M IPs |
| 10.10.0.0/16 | HQ | Sede central | 65k IPs |
| 10.20.0.0/16 | Site-A | Oficina A | 65k IPs |
| 10.30.0.0/16 | Site-B | Oficina B | 65k IPs |
| 172.16.0.0/12 | Cloud | Azure/AWS networks (via VPN) | 1M IPs |

### 1.2 HQ Subnetting (/16 dividido)
| Subnet | CIDR | Hosts | Uso |
|--------|------|-------|-----|
| 10.10.1.0/24 | /24 | 254 | Management (switches, routers) |
| 10.10.10.0/24 | /24 | 254 | IT Department |
| 10.10.20.0/24 | /24 | 254 | Finance |
| 10.10.30.0/23 | /23 | 510 | Voice (VoIP) |
| 10.10.40.0/24 | /24 | 254 | Guest WiFi |
| 10.10.50.0/24 | /24 | 254 | IoT |
| 10.10.200.0/22 | /22 | 1022 | Servers |

---

## 2. Reservaciones DHCP

### 2.1 Static IPs Reserved Ranges
| Range | Uso |
|-------|-----|
| .1 - .10 | Gateways, VIPs |
| .11 - .50 | Infrastructure (switches, APs, printers) |
| .51 - .100 | Servers |
| .101 - .254 | DHCP scope (dynamic) |

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
