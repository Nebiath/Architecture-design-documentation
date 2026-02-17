# Runbook: Provisioning de Circuitos WAN

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Procedimientos operacionales completos para el ciclo de vida de circuitos WAN: alta de nuevo circuito, modificación de ancho de banda y baja de circuito.

---

## Uso de Este Runbook

| Procedimiento | Ir a sección | Tiempo estimado |
|--------------|-------------|----------------|
| Alta de nuevo circuito WAN | [Sección 1](#1-alta-de-nuevo-circuito) | 15–45 días (incluyendo proveedor) |
| Ampliación / reducción de ancho de banda | [Sección 2](#2-modificación-de-ancho-de-banda) | 5–15 días hábiles |
| Baja de circuito | [Sección 3](#3-baja-de-circuito) | 30–90 días (según contrato) |
| Sustitución por avería (RMA circuito) | [Sección 4](#4-sustitución-de-circuito-por-avería) | 1–5 días hábiles |

---

## 1. Alta de Nuevo Circuito

### 1.1 Pre-requisitos

Antes de iniciar el proceso, verificar que existe:

```
□ Aprobación formal de Procurement / Dirección (email o ticket)
□ Justificación técnica documentada (en ADR o en el ticket de cambio)
□ Site address exacta (dirección postal y datos de contacto local)
□ Nombre del contacto técnico en el site para la instalación
□ Rack / armario / espacio físico disponible para el equipo CPE
□ Alimentación eléctrica disponible (y SAI si aplica)
□ IP LAN de gestión para el nuevo CPE (reservar en IPAM antes de pedir)
□ Change Request aprobado en ServiceNow
```

### 1.2 Fase 1 — Solicitud al Proveedor

**Responsable:** @wan-lead  
**Duración estimada de esta fase:** 1–3 días hábiles

```
Paso 1.1 — Completar el formulario de solicitud del proveedor
   - Acceder al portal: [URL portal proveedor]
   - Tipo de servicio: [MPLS L3VPN / Internet dedicado / SD-WAN]
   - Site de instalación: dirección postal completa
   - Velocidad contratada: [X] Mbps / Gbps
   - Redundancia: sí/no, tipo
   - Fecha deseada de puesta en servicio: YYYY-MM-DD

Paso 1.2 — Recibir y revisar el Order Acknowledgement
   - El proveedor enviará confirmación con número de orden: [ORDER-XXXXXX]
   - Verificar datos: site, velocidad, fecha estimada
   - Si hay discrepancias → corregir antes de continuar

Paso 1.3 — Registrar en inventario (estado: "En provisioning")
   - Añadir fila en la tabla de circuitos de provider-contracts.md
   - Estado: ⏳ En provisioning
   - Actualizar IPAM con los IPs reservados para este circuito
```

### 1.3 Fase 2 — Coordinación de Instalación

**Responsable:** @network-engineer  
**Duración estimada:** Variable según proveedor (1–30 días)

```
Paso 2.1 — Confirmar survey / visita técnica del proveedor
   - El proveedor hará una visita de survey al site
   - Coordinar con el contacto local: [nombre y teléfono del site]
   - Confirmar que hay acceso físico al rack el día acordado

Paso 2.2 — Revisar el diseño propuesto por el proveedor
   - Solicitar al proveedor: plano de cableado, modelo de CPE/equipo entregado
   - Verificar que el equipo CPE es compatible con nuestra infraestructura
   - Si el CPE es del cliente (CE propio): coordinar entrega y rack unit

Paso 2.3 — Confirmar fecha de instalación
   - El proveedor confirmará fecha y ventana horaria de instalación
   - Notificar al site: nombre del técnico del proveedor, fecha, hora
   - Abrir ventana de mantenimiento en el Change Calendar
```

### 1.4 Fase 3 — Instalación y Configuración

**Responsable:** @network-engineer + técnico del site  
**Duración estimada:** 1 día

```
Paso 3.1 — Día de instalación: presencia (remota o física)
   □ Confirmar que el técnico del proveedor ha llegado al site
   □ Verificar instalación física: cableado, alimentación, rack

Paso 3.2 — Configuración del equipo CE / WAN Edge
   □ Configurar hostname según naming convention: [CE/WE]-[SITE]-[N]
   □ Aplicar configuración base desde template vManage (SD-WAN)
      O configurar manualmente (MPLS):
   □ Configurar interfaz WAN CE-PE (IP, máscara según IPAM)
   □ Configurar protocolo de routing CE-PE (BGP/OSPF/static)
   □ Configurar QoS según política estándar
   □ Configurar acceso de gestión: SSH, SNMP, Syslog hacia servidores de gestión
   □ Activar BFD sobre enlace CE-PE

Paso 3.3 — Verificación de conectividad con el proveedor
   □ Verificar Layer 1 y Layer 2 (interfaz up/up)
   □ Verificar BGP/OSPF adjacency con PE
   □ Verificar recepción de rutas esperadas
   □ Verificar anuncio de rutas locales al proveedor
```

**Comandos de verificación (adaptar a plataforma):**

```bash
# Verificar estado de interfaz
show interfaces [INTERFAZ-WAN]
# Esperado: line protocol is up

# Verificar BGP (si protocolo CE-PE es BGP)
show bgp vpnv4 unicast all neighbors [IP-PE] summary
# Esperado: state = Established, prefixes received > 0

# Verificar rutas WAN recibidas
show ip route vrf CORP-PROD
# Esperado: rutas de otros sites presentes

# Verificar BFD
show bfd neighbors detail
# Esperado: state = Up

# Test de conectividad extremo a extremo
ping vrf CORP-PROD [IP-DESTINO-REMOTO] source [IP-LOCAL] repeat 100
# Esperado: 100% success, RTT < umbral SLA
```

### 1.5 Fase 4 — Aceptación y Documentación

**Responsable:** @wan-lead  
**Duración estimada:** 1–2 días

```
Paso 4.1 — Pruebas de aceptación formal (SAT - Site Acceptance Test)
   □ Disponibilidad: 99,95% durante las últimas 24h de monitorización
   □ Latencia: ≤ SLA contratado (verificar con IPSLA / TWAMP)
   □ Pérdida paquetes: ≤ SLA contratado
   □ Throughput: alcanza el BW nominal contratado
   □ QoS: marcado de tráfico funciona correctamente
   □ Failover (si circuito de backup): test de corte y recuperación

Paso 4.2 — Actualizar documentación
   □ Actualizar provider-contracts.md: CID real, IPs definitivas, estado ✅ Activo
   □ Actualizar wan-topology.md: añadir site/enlace al diagrama
   □ Actualizar IPAM/NetBox con todos los IPs del circuito
   □ Añadir el circuito al dashboard de monitoring
   □ Configurar alertas en [Zabbix/SolarWinds] para el nuevo circuito

Paso 4.3 — Cerrar el proceso
   □ Firmar el acta de aceptación con el proveedor (si aplica contractualmente)
   □ Notificar a stakeholders: "Circuito [CID] operativo desde YYYY-MM-DD"
   □ Cerrar Change Request en ServiceNow
   □ Actualizar entrada en IPAM: estado = Activo
```

---

## 2. Modificación de Ancho de Banda

**Responsable:** @wan-lead  
**Tiempo estimado proveedor:** 5–15 días hábiles

```
Paso 1 — Análisis de necesidad
   □ Revisar utilización actual: [dashboard utilización WAN]
   □ Si utilización > 70% durante > 2 semanas → justifica ampliación
   □ Documentar justificación en el ticket de cambio

Paso 2 — Solicitar al proveedor
   □ Acceder al portal del proveedor: [URL]
   □ Solicitar modificación de BW sobre CID: [CID]
   □ Nueva velocidad: [X] Mbps / Gbps
   □ Fecha deseada: YYYY-MM-DD

Paso 3 — Coordinación de ventana
   □ El proveedor confirmará fecha de cambio
   □ Crear Change Request en ServiceNow
   □ Notificar a usuarios del site si habrá corte breve (generalmente < 5 min)

Paso 4 — Día del cambio
   □ Monitorizar durante el cambio en [dashboard]
   □ Verificar nueva velocidad: throughput test o show interface BW
   □ Actualizar QoS si el cambio de BW afecta a las políticas

Paso 5 — Documentar
   □ Actualizar provider-contracts.md con nueva velocidad
   □ Actualizar IPAM si hay cambios de IPs
   □ Cerrar Change Request
```

---

## 3. Baja de Circuito

> ⚠️ La baja de un circuito es irreversible. Verificar con negocio antes de proceder.

**Responsable:** @wan-lead + aprobación @network-director  
**Tiempo de preaviso al proveedor:** Ver contrato (típicamente 30–90 días)

```
Paso 1 — Verificación pre-baja
   □ Confirmar aprobación escrita de @network-director y del responsable de negocio del site
   □ Verificar que el site tiene conectividad alternativa (otro circuito activo)
   □ Verificar que NO hay servicios críticos dependientes únicamente de este circuito
   □ Comprobar penalización por baja anticipada si aplica

Paso 2 — Notificación al proveedor
   □ Enviar notificación formal de baja: email a Account Manager + NOC
   □ Indicar: CID, fecha de baja solicitada, confirmar datos de facturación final
   □ Solicitar confirmación escrita de recepción y aceptación de fecha

Paso 3 — Preparación técnica (semana antes de la baja)
   □ Redirigir todo el tráfico al circuito alternativo
   □ Verificar que el tráfico redirigido no satura el circuito restante
   □ Actualizar routing (retirar rutas que dependen del circuito a dar de baja)
   □ Notificar a usuarios del site del cambio

Paso 4 — Día de la baja
   □ Confirmar corte con el proveedor
   □ Retirar configuración del equipo CE/WAN Edge
   □ Si el CPE era del proveedor: coordinar recogida del equipo

Paso 5 — Documentar la baja
   □ Actualizar provider-contracts.md: mover el circuito a la sección "Histórico"
   □ Actualizar wan-topology.md: eliminar el enlace del diagrama
   □ Liberar IPs en IPAM
   □ Eliminar de monitoring y alertas
   □ Cerrar Change Request en ServiceNow
```

---

## 4. Sustitución de Circuito por Avería

**Cuándo usar:** El proveedor determina que el circuito tiene una avería física irrecuperable y debe provisionar un nuevo circuito sustituto.

```
Paso 1 — Confirmar necesidad de sustitución
   □ El proveedor ha declarado el circuito irreparable (notificación escrita)
   □ Obtener: nuevo CID provisional o definitivo, fechas de instalación del nuevo circuito
   □ Negociar que el SLA del circuito en avería se transfiere al nuevo

Paso 2 — Puente temporal (si aplica)
   □ Si hay tiempo de espera > 24h: activar LTE backup o failover manual
   □ Comunicar a usuarios del site

Paso 3 — Instalación del circuito sustituto
   □ Seguir Fase 3 del Procedimiento de Alta (Sección 1.4)
   □ El proveedor suele reutilizar el CPE y solo sustituye el circuito

Paso 4 — Actualizar documentación
   □ Reemplazar CID antiguo por CID nuevo en provider-contracts.md
   □ Registrar el incidente en el historial del proveedor
   □ Iniciar proceso de reclamación de créditos SLA si el downtime supera el umbral
```

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
