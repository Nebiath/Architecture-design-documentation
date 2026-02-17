# Runbook: Troubleshooting WAN

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Guía de diagnóstico y resolución de incidencias WAN. Organizada por síntoma para localizar rápidamente el procedimiento adecuado, especialmente bajo presión de incidente.

---

## Índice Rápido por Síntoma

| Síntoma | Ir a |
|---------|------|
| Site sin conectividad WAN completa | [§1](#1-site-sin-conectividad-wan) |
| Conectividad intermitente / inestable | [§2](#2-conectividad-intermitente) |
| Degradación de rendimiento (lento, alta latencia) | [§3](#3-degradación-de-rendimiento) |
| BGP caído o rutas incorrectas | [§4](#4-problemas-de-routing-bgp) |
| SD-WAN: túneles caídos o controlador no accesible | [§5](#5-troubleshooting-sd-wan) |
| VPN IPSec caída | [§6](#6-vpn-ipsec-caída) |
| Saturación de ancho de banda | [§7](#7-saturación-de-ancho-de-banda) |
| Problema de QoS / tráfico VoIP degradado | [§8](#8-problemas-de-qos-y-voip) |

---

## Antes de Empezar — Información a Recopilar Siempre

```
□ ¿Qué síntoma exacto reportan los usuarios? (sin conectividad / lento / intermitente)
□ ¿Desde cuándo ocurre? (timestamp exacto si es posible)
□ ¿Afecta a todos los usuarios del site o solo a algunos?
□ ¿Afecta a todas las aplicaciones o solo a algunas?
□ ¿Hubo algún cambio reciente en la red? (ver Change Calendar en ServiceNow)
□ ¿El proveedor tiene mantenimiento planificado? (verificar notificaciones de email)
□ ¿Hay alertas activas en el dashboard? → [URL Dashboard WAN]
□ ¿Hay incidentes abiertos con el proveedor? → [URL portal proveedor]
```

---

## 1. Site sin Conectividad WAN

### 1.1 Árbol de Diagnóstico

```
¿El site tiene acceso WAN?
│
├── NO → ¿El equipo WAN Edge/CE responde por gestión OOB?
│         │
│         ├── NO → Problema físico en el site (equipo apagado, sin luz, HW fallo)
│         │        → Procedimiento 1.2: Fallo físico en site
│         │
│         └── SÍ → ¿La interfaz WAN está up/up?
│                   │
│                   ├── NO → Problema de Layer 1/2 con proveedor
│                   │        → Procedimiento 1.3: Fallo de circuito
│                   │
│                   └── SÍ → ¿BGP / routing está OK?
│                             │
│                             ├── NO → Procedimiento §4: Routing BGP
│                             └── SÍ → Problema de filtrado / ACL / firewall
│                                       → Procedimiento 1.4: Problema lógico
│
└── SÍ (parcial) → Ver §2 o §3
```

### 1.2 Procedimiento: Fallo Físico en Site

```bash
# Intentar acceder al equipo CE/WAN Edge por OOB (consola/terminal server)
ssh [USUARIO]@[IP-OOB-CONSOLA-SITE]
# Si no responde → problema de alimentación o HW grave

# Si hay consola disponible:
show platform                          # Estado del hardware
show power                             # Estado de alimentación (switches/routers con PoE)
show environment all                   # Temperatura, ventiladores, fuentes

# Verificar si el equipo arrancó correctamente
show version | include uptime
# Si uptime es muy bajo → equipo reiniciado recientemente

# Verificar logs del sistema
show logging last 100
# Buscar: crash, power loss, interface error, kernel panic
```

**Acciones:**
- Si el equipo no enciende → contactar con el personal del site para verificar física y alimentación
- Si el equipo reinició inesperadamente → recopilar crash dump y abrir ticket con el fabricante
- Si HW fallo confirmado → iniciar proceso de RMA

### 1.3 Procedimiento: Fallo de Circuito (Layer 1/2)

```bash
# Verificar estado de la interfaz WAN
show interfaces [INTERFAZ-WAN-CE]
# Resultado a analizar:
# - "line protocol is down" → problema Layer 2 o desconexión física
# - "Hardware is [x], address is [x]" → interfaz existe
# - Input/Output errors elevados → problema de señal o HW

# Verificar contadores de errores
show interfaces [INTERFAZ-WAN-CE] counters errors
# Errores altos → problema físico en el cableado o equipo

# Si es interfaz Ethernet (handoff del proveedor)
show interfaces [INTERFAZ-WAN-CE] | include duplex|speed|negotiation
# Verificar que speed/duplex coincide con lo esperado (no auto-negotiation failure)

# Intentar forzar speed/duplex si hay duda
interface [INTERFAZ-WAN-CE]
 speed [1000|10000]
 duplex full
 no shutdown
```

**Escalar al proveedor:**

```
1. Abrir ticket en portal del proveedor: [URL]
2. Facilitar:
   - Circuit ID (CID): [CID del circuito]
   - Descripción: "Interfaz WAN down desde [HH:MM]. El equipo CE no ve señal en el puerto físico"
   - Contacto para retorno: [teléfono + email]
3. Solicitar: número de ticket y SLA comprometido
4. Registrar en el incidente de ServiceNow: número de ticket del proveedor
```

### 1.4 Procedimiento: Problema Lógico (Layer 3 / Filtrado)

```bash
# Verificar tabla de routing en el VRF afectado
show ip route vrf CORP-PROD
# ¿Están las rutas de destino presentes?
# Si no hay rutas → problema de routing, ver §4

# Verificar si hay ACL bloqueando
show ip access-lists
# Buscar entradas con hits elevados en "deny"

# Test de conectividad desde el CE hacia destino conocido
ping vrf CORP-PROD [IP-DESTINO] source [IP-LOOPBACK-CE] repeat 100
traceroute vrf CORP-PROD [IP-DESTINO] source [IP-LOOPBACK-CE]

# Si traceroute llega al PE pero no más allá → problema en la red del proveedor
# Si traceroute se cae en el CE → problema local (ACL, NAT, routing)
```

---

## 2. Conectividad Intermitente

### 2.1 Diagnóstico

```bash
# Ejecutar ping extendido continuo para reproducir el problema
ping vrf CORP-PROD [IP-DESTINO] repeat 10000 timeout 2
# Anotar: frecuencia de pérdidas, patrón (periódico / aleatorio)

# Revisar logs del CE por flaps de interfaz o BGP
show logging | include BGP|Interface|line protocol
# Buscar: "changed state to down" / "BGP neighbor X went down"

# Verificar contadores de errores acumulados
show interfaces [INTERFAZ-WAN] | include error|reset|throttle
# CRC errors altos → problema físico / cable / SFP
# Input errors altos → problema de señal
# Output drops → saturación (ver §7)

# Verificar BFD
show bfd neighbors detail
# ¿Hay "missed echoes"? → inestabilidad del enlace
```

### 2.2 Causas Frecuentes y Acciones

| Síntoma en logs | Causa probable | Acción |
|----------------|---------------|--------|
| `line protocol is down/up` frecuente | Problema físico: cable, SFP, puerto | Cambiar cable/SFP, escalar a proveedor |
| BGP flaps frecuentes | BFD demasiado agresivo o problema de latencia | Ajustar timers BFD, escalar proveedor |
| CRC errors crecientes | Señal eléctrica degradada | Cambiar cable/SFP, escalar proveedor |
| `Input errors` periódicos | MTU mismatch o problema de señal | Verificar MTU end-to-end |
| Pérdida en horas punta | Saturación de ancho de banda | Ver §7 |

---

## 3. Degradación de Rendimiento

### 3.1 Diagnóstico de Latencia Alta

```bash
# Medir latencia RTT desde CE hacia probe del proveedor y hacia destino final
ping vrf CORP-PROD [IP-PE-PROVEEDOR] repeat 100 timeout 2
ping vrf CORP-PROD [IP-DESTINO-REMOTO] repeat 100 timeout 2

# Traceroute para localizar el salto con latencia alta
traceroute vrf CORP-PROD [IP-DESTINO] source [IP-LOCAL] probe 10

# Verificar utilización de interfaces
show interfaces [INTERFAZ-WAN] | include rate|utilization
# O vía SNMP en Zabbix: [URL dashboard utilización]

# Verificar colas QoS (¿hay drops en alguna clase?)
show policy-map interface [INTERFAZ-WAN]
# Buscar: drops en clases de alta prioridad (VOIP, BUSINESS-CRITICAL)
```

### 3.2 Diagnóstico de Pérdida de Paquetes

```bash
# Test de pérdida en diferentes puntos del path
ping vrf CORP-PROD [IP-PE] repeat 1000 timeout 2       # CE → PE
ping vrf CORP-PROD [IP-SITE-REMOTO-CE] repeat 1000     # CE → CE remoto
ping vrf CORP-PROD [IP-SERVIDOR] repeat 1000           # CE → destino final

# Si la pérdida aparece solo en CE→PE: problema del circuito (escalar proveedor)
# Si la pérdida aparece solo más allá del PE: problema interno de la red del proveedor
# Si solo en destinos específicos: problema en el site de destino

# Verificar MTU (pérdida de paquetes grandes puede ser MTU mismatch)
ping vrf CORP-PROD [IP-DESTINO] size 1472 df-bit repeat 100
# Si falla con size grande → problema de MTU/fragmentación
```

---

## 4. Problemas de Routing BGP

### 4.1 Diagnóstico BGP

```bash
# Estado general de sesiones BGP
show bgp vpnv4 unicast all summary
# Columna "State/PfxRcd": debe ser "Established" y número de prefijos > 0
# Si aparece "Idle" / "Active" / "Connect" → sesión caída

# Detalle de sesión BGP con PE
show bgp vpnv4 unicast all neighbors [IP-PE] | include BGP|state|prefix|hold

# Ver rutas recibidas del proveedor
show bgp vpnv4 unicast vrf CORP-PROD neighbors [IP-PE] received-routes
# ¿Se reciben las rutas esperadas de otros sites?

# Ver rutas que estamos anunciando al proveedor
show bgp vpnv4 unicast vrf CORP-PROD neighbors [IP-PE] advertised-routes
# ¿Se anuncian los prefijos locales correctos?

# Ver tabla de routing resultante
show ip route vrf CORP-PROD
# ¿Están las rutas de todos los sites remotos?
```

### 4.2 Restablecer BGP (con cuidado)

```bash
# Soft reset (sin cortar la sesión) — preferible
clear ip bgp [IP-PE] vrf CORP-PROD soft

# Hard reset (corta y restablece la sesión) — solo si soft no resuelve
# ⚠️ ATENCIÓN: Esto cortará la conectividad WAN durante la reconvergencia (~30-60 seg)
# Notificar en Teams: #network-ops antes de ejecutar
clear ip bgp [IP-PE]

# Forzar reanuncio de rutas al PE
clear ip bgp [IP-PE] vrf CORP-PROD out
```

### 4.3 BGP no levanta — Checklist

```bash
□ ¿IP del PE correcta? → show run | include neighbor
□ ¿AS del proveedor correcto? → show bgp vpnv4 unicast all neighbors [IP-PE] | include remote AS
□ ¿Password MD5 correcta? → No se puede ver, pero comprobar logs de autenticación
   show logging | include MD5
□ ¿Firewall bloqueando TCP 179? → Si hay firewall intermedio, verificar regla BGP
□ ¿MTU del enlace CE-PE correcto? → show interfaces [INTERFAZ]
□ ¿BFD muy agresivo causando flaps? → Ajustar timers BFD
```

---

## 5. Troubleshooting SD-WAN

### 5.1 WAN Edge no contacta con controladores

```bash
# Estado de conexiones con el control plane
show sdwan control connections
# Esperado: vManage, vSmart y vBond en estado "up"

# Si alguna conexión está down:
show sdwan control local-properties
# Verificar: system-ip, site-id, token correcto

# Verificar resolución DNS del vBond
nslookup sdwan-vbond.empresa.com
ping sdwan-vbond.empresa.com

# Ver logs de control plane
show sdwan control connection-history
# Buscar: razón de desconexión
```

### 5.2 Túneles BFD / Data plane caídos

```bash
# Estado de sesiones BFD entre WAN Edges
show sdwan bfd sessions
# Esperado: todos los sites relevantes en estado "up"

# Si faltan túneles hacia algunos sites:
show sdwan omp routes
# ¿Se reciben rutas OMP de esos sites?

show sdwan omp tlocs
# ¿Los TLOCs (transport locators) de los sites remotos están presentes?

# Ver estadísticas de calidad de path (app-route)
show sdwan app-route stats
# Comprobar latencia, pérdida, jitter por path/color
```

### 5.3 Política de aplicaciones no funciona como se espera

```bash
# Verificar políticas recibidas desde vSmart
show sdwan policy from-vsmart

# Ver estadísticas de política de tráfico
show sdwan policy data-plane-filter

# Verificar a qué path va el tráfico de una aplicación concreta
show sdwan policy service-path vpn [N] interface [IF] source-ip [IP] dest-ip [IP] protocol [N]
# Muestra el path elegido para ese flujo específico
```

---

## 6. VPN IPSec Caída

### 6.1 Diagnóstico

```bash
# Ver estado de sesiones IKEv2 / ISAKMP
show crypto ikev2 sa                   # IKEv2
show crypto isakmp sa                  # IKEv1 (legacy)
# Esperado: estado QM_IDLE (activo) o ESTABLISHED

# Ver SAs IPSec activas
show crypto ipsec sa peer [IP-REMOTA]
# Verificar: encaps/decaps incrementando, no hay errores

# Ver si hay tráfico pasando
show crypto ipsec sa peer [IP-REMOTA] | include pkts enc|pkts dec|send error|recv error

# Debug (con precaución en producción — usa term mon y duración limitada)
debug crypto ikev2 error
debug crypto ikev2 packet
terminal monitor
# Esperar 30 segundos, capturar output, desactivar:
undebug all
```

### 6.2 Causas Frecuentes

| Síntoma | Causa probable | Acción |
|---------|---------------|--------|
| IKE SA no levanta | PSK incorrecta / distinta en ambos lados | Verificar PSK en CyberArk y reconfigurar |
| IKE SA levanta pero IPSec no | Mismatch en transform-set (enc/auth/PFS) | Verificar parámetros Phase 2 con contraparte |
| Tráfico enrutado fuera del túnel | Crypto ACL o route incorrecta | Revisar crypto map / VTI routing |
| Túnel cae periódicamente | Lifetime demasiado corto o problema de red | Ajustar lifetime o investigar inestabilidad |
| `send errors` elevados | Fragmentación / MTU | Ajustar ip tcp adjust-mss / MTU |

### 6.3 Forzar Renegociación del Túnel

```bash
# Limpiar SAs IKE y IPSec (fuerza renegociación)
# ⚠️ Corta el tráfico del túnel durante ~10-30 segundos
clear crypto ikev2 sa peer [IP-PEER]
clear crypto ipsec sa peer [IP-PEER]

# Verificar que vuelve a levantar
show crypto ikev2 sa
show crypto ipsec sa peer [IP-PEER]
```

---

## 7. Saturación de Ancho de Banda

### 7.1 Diagnóstico

```bash
# Ver utilización actual de las interfaces WAN
show interfaces [INTERFAZ-WAN] | include rate
# O desde el dashboard: [URL Grafana utilización]

# Ver qué tráfico ocupa el ancho de banda (top flows)
# Si hay NetFlow configurado:
# → Consultar en [Colector NetFlow / Grafana / Ntopng]: Top talkers por IP y protocolo

# Ver qué colas QoS están saturadas
show policy-map interface [INTERFAZ-WAN] output
# Buscar: "drop" en alguna clase → esa clase está saturada
# Buscar: clase "class-default" con muchos drops → tráfico best-effort llenando el enlace

# Verificar si el problema es temporal (hora punta) o sostenido
# → Revisar gráfica histórica en Grafana: última semana / mes
```

### 7.2 Acciones a Corto Plazo

```bash
# Opción 1: Throttling de tráfico de baja prioridad (backups, P2P)
# Modificar policy-map para limitar clase BEST-EFFORT:
policy-map PM-WAN-EGRESS
 class BEST-EFFORT
  police rate [X] bps burst [X]
   conform-action transmit
   exceed-action drop

# Opción 2: Bloquear temporalmente tráfico no crítico
ip access-list extended ACL-BLOCK-TORRENTS
 deny tcp any any eq [puerto P2P]
 deny udp any any eq [puerto P2P]
 permit ip any any

# Opción 3: SD-WAN — ajustar política para redirigir tráfico best-effort a Internet
# (si hay DIA disponible)
```

### 7.3 Solución a Largo Plazo

```
Si la saturación es recurrente (> 2 semanas con uso > 80%):
→ Iniciar proceso de ampliación de BW (ver runbook circuit-provisioning.md §2)
→ Documentar la justificación con datos de utilización histórica
→ Solicitar aprobación a @network-director
```

---

## 8. Problemas de QoS y VoIP

### 8.1 Síntomas de VoIP / Teams degradado

Síntomas habituales: voz entrecortada, eco, latencia alta en llamadas, videollamadas pixeladas.

### 8.2 Diagnóstico

```bash
# Verificar que el tráfico VoIP se está marcando correctamente
show policy-map interface [INTERFAZ-WAN] input
# La clase VOIP-RTP debe mostrar tráfico clasificado (pkts > 0)

# Verificar que la cola de prioridad (LLQ) no tiene drops
show policy-map interface [INTERFAZ-WAN] output
# Clase VOIP-RTP / REALTIME → "queue depth 0 drops 0" idealmente
# Drops en esta cola = problema grave de QoS

# Verificar marcado DSCP en el tráfico real (con capture breve)
show monitor capture [CAP] buffer brief
# Ver campo DSCP: debe ser EF (46) para VoIP RTP

# Medir latencia y jitter en tiempo real
show ip sla summary [N]
# O desde SD-WAN:
show sdwan app-route stats | include [IP-DESTINO-TEAMS-O-VOICE]
```

### 8.3 Causas Frecuentes y Acciones

| Causa | Diagnóstico | Acción |
|-------|-------------|--------|
| Tráfico VoIP no clasificado correctamente | Clase VOIP con 0 paquetes en policy-map | Revisar/actualizar clasificación NBAR o ACL |
| LLQ sin suficiente BW asignado | Drops en clase REALTIME | Aumentar % LLQ (requiere change) |
| Enlace saturado (VoIP compite con bulk) | Utilización > 80% + drops en class-default | Throttling de best-effort o ampliar BW |
| Microsoft Teams usa path subóptimo (SD-WAN) | App-route no elige MPLS para Teams | Revisar SLA class REALTIME en política SD-WAN |
| MTU causando fragmentación de paquetes RTP | Jitter alto + paquetes fuera de orden | ip tcp adjust-mss, verificar MTU path |

---

## Información de Soporte de Emergencia

| Recurso | Detalle |
|---------|---------|
| Dashboard WAN | [URL Grafana] |
| Alertas PagerDuty | [URL filtro WAN] |
| Inventario circuitos y CIDs | [provider-contracts.md](../providers/provider-contracts.md) |
| Contactos NOC proveedores | [provider-contracts.md §3](../providers/provider-contracts.md#3-contactos-operacionales-por-proveedor) |
| IPAM (IPs y CIDs) | [URL NetBox/Infoblox] |
| Change Calendar | [URL ServiceNow] |
| Escalado interno | @wan-lead → @network-director → @cto-office |

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
