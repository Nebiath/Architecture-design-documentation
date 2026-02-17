# Diseño de Redundancia WAN

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Diseño de alta disponibilidad para la red WAN: mecanismos de failover, tiempos de convergencia, tests periódicos y procedimientos de failback.

---

## 1. Objetivos de Disponibilidad

| Métrica | Objetivo | Medición |
|---------|----------|----------|
| Disponibilidad WAN global | 99,95% mensual (~22 min downtime/mes) | Zabbix uptime |
| RTO (Recovery Time Objective) | < [X] minutos | Test de failover |
| RPO (Recovery Point Objective) | < [X] minutos (para sesiones TCP) | — |
| MTTR incidente de red | < 4 horas | ServiceNow |
| Frecuencia test de failover | Trimestral | Calendar |

---

## 2. Mapa de Redundancia por Tipo de Site

### 2.1 Tier 1 — HQ y Datacenters (Máxima Redundancia)

```
  ISP-A (Primario)    ISP-B (Secundario)     MPLS (Proveedor)
       │                    │                       │
  ┌────┴────┐          ┌────┴────┐            ┌────┴────┐
  │ Router  │          │ Router  │            │ Router  │
  │ WAN-A   │          │ WAN-B   │            │ WAN-C   │
  └────┬────┘          └────┬────┘            └────┬────┘
       │                    │                       │
  ┌────┴────────────────────┴───────────────────────┴────┐
  │                    Core Switch (HA pair)              │
  └──────────────────────────────────────────────────────┘
```

**Mecanismos activos:**

| Capa | Mecanismo | Protocolo | Tiempo convergencia |
|------|-----------|-----------|---------------------|
| Enlace físico | Redundancia física + LAG | LACP | < 1 seg |
| Gateway IP | Failover de router | HSRP / VRRP (preempt) | < [X] seg |
| Routing | Reconvergencia IGP/BGP | BGP + BFD | < [X] seg |
| SD-WAN | Failover de transport | BFD + OMP | < [X] seg |
| Aplicación | SD-WAN App-Aware | IP SLA / NBAR | < [X] seg |

### 2.2 Tier 2 — Oficinas Estándar

| Componente | Primario | Backup | Failover |
|-----------|----------|--------|---------|
| Transporte WAN | MPLS ([X] Mbps) | Internet broadband ([X] Mbps) | Automático SD-WAN BFD |
| Internet | ISP local (DIA) | Failover a través de overlay | SD-WAN automático |
| Router/WAN Edge | Único dispositivo | Sin redundancia HW | N/A |

### 2.3 Tier 3 — Sites Pequeños

| Componente | Primario | Backup | Failover |
|-----------|----------|--------|---------|
| Transporte WAN | Internet broadband | LTE 4G ([Operador]) | Automático SD-WAN |
| Tiempo de failover a LTE | — | < [X] min | BFD + OMP |

---

## 3. Matriz de Escenarios de Fallo y Respuesta

| # | Escenario | Detección | Acción automática | Acción manual | Impacto usuarios |
|---|-----------|-----------|-------------------|---------------|-----------------|
| 1 | Caída enlace físico ISP-A (HQ) | Interface down, BFD | BGP withdraw ISP-A, tráfico a ISP-B | Monitorizar, contactar ISP | Ninguno (< 1 seg) |
| 2 | Caída total internet HQ | Probe externo falla | SD-WAN redirige sites a DIA local | Contactar ISP-A y ISP-B | Servicios cloud degradados |
| 3 | Caída circuito MPLS site spoke | BFD timeout | SD-WAN failover a Internet | Contactar proveedor MPLS | Latencia aumenta |
| 4 | Caída WAN Edge físico (site spoke) | SNMP unreachable | Ninguna (single device) | Enviar reemplazo / RMA | Site sin WAN |
| 5 | Caída controlador SD-WAN (vManage) | Health check | Failover a vManage secundario | Verificar HA cluster | Solo gestión afectada |
| 6 | Caída BGP con MPLS PE | BFD, BGP keepalive | SD-WAN failover a Internet | Contactar proveedor NOC | Latencia aumenta |
| 7 | Degradación severa MPLS (> 10% pérdida) | IP SLA / BFD | SD-WAN redirige a Internet (SLA policy) | Abrir ticket proveedor | Servicios de baja prioridad afectados |
| 8 | Caída del site HQ completo | Multiple alerts | DR plan activado (ver [DR WAN]) | Plan de continuidad | Crítico — Plan DR |

---

## 4. Configuración de Mecanismos de Detección

### 4.1 BFD (Bidirectional Forwarding Detection)

```
# Parámetros BFD estándar para WAN Edge routers
bfd interval [X] min_rx [X] multiplier [X]
# Ejemplo conservador (estabilidad primero): 500ms / 500ms / 3 = detección en 1,5s
# Ejemplo agresivo (HQ links críticos):      100ms / 100ms / 3 = detección en 300ms
```

| Tipo de enlace | Interval (ms) | Min_rx (ms) | Multiplier | Detección total |
|---------------|--------------|-------------|-----------|----------------|
| MPLS CE-PE (HQ) | [100] | [100] | [3] | [300] ms |
| Internet (HQ) | [300] | [300] | [3] | [900] ms |
| MPLS CE-PE (spoke) | [500] | [500] | [3] | [1.500] ms |
| Internet (spoke) | [1000] | [1000] | [3] | [3.000] ms |

### 4.2 IP SLA para Verificación de Calidad de Enlace

```
! IP SLA para monitorización de latencia / pérdida hacia probe externo
ip sla [N]
 icmp-echo [PROBE-IP] source-interface [INTERFAZ]
  frequency [X]
  threshold [X]
  timeout [X]
ip sla schedule [N] life forever start-time now
!
track [N] ip sla [N] reachability
!
! Uso en routing:
ip route 0.0.0.0 0.0.0.0 [GW-PRIMARIO] track [N]
ip route 0.0.0.0 0.0.0.0 [GW-BACKUP] [HIGHER-AD]
```

### 4.3 SD-WAN SLA Polling

```
# En vManage — App-Route Policy
# Parámetros de verificación de path quality:
poll-interval  [X]   # segundos entre sondas
multiplier     [X]   # número de fallos para declarar path DOWN
```

---

## 5. Procedimiento de Test de Failover

### 5.1 Calendario de Tests

| Test | Frecuencia | Ventana | Responsable | Último test |
|------|-----------|---------|-------------|-------------|
| Failover ISP-A → ISP-B (HQ) | Trimestral | Sábado 03:00–04:00 | @wan-lead | YYYY-MM-DD |
| Failover MPLS → Internet (site spoke) | Semestral | Sábado 03:00–04:00 | @network-team | YYYY-MM-DD |
| Failover a LTE (Tier 3) | Anual | Laborable 08:00–09:00 | @network-team | YYYY-MM-DD |
| Simulacro DR completo | Anual | Planificado con negocio | @wan-lead + @sre-team | YYYY-MM-DD |

### 5.2 Procedimiento de Test (HQ ISP-A → ISP-B)

```
PRE-TEST (24h antes):
□ Notificar a @network-team y stakeholders afectados
□ Crear Change Request en ServiceNow
□ Confirmar que ISP-B está operativo (comprobar BGP session activa)
□ Verificar que el monitoring capturará el evento

EJECUCIÓN:
□ T+00:00 — Apagar BGP hacia ISP-A (shutdown neighbor [IP-PE-ISP-A])
□ T+00:30 — Verificar BGP session ISP-A = Idle/Active
□ T+01:00 — Verificar que routing conmutó a ISP-B
□ T+02:00 — Test de aplicaciones críticas: Teams, SAP, O365
□ T+05:00 — Verificar latencia y pérdida en dashboard
□ T+10:00 — Comprobar alertas en PagerDuty (deben haberse disparado y auto-cerrado)

POST-TEST:
□ Restaurar BGP ISP-A (no shutdown)
□ Verificar reconvergencia a estado primario
□ Documentar resultados en tabla de tests
□ Cerrar Change Request en ServiceNow
□ Actualizar "Último test" en este documento
```

### 5.3 Registro de Resultados de Tests

| Fecha | Test realizado | Tiempo failover | Resultado | Issues detectados | Acciones |
|-------|---------------|----------------|-----------|-------------------|---------|
| YYYY-MM-DD | [Tipo test] | [X] seg | ✅ OK / ❌ Fallo | [Descripción] | [Acciones] |

---

## 6. Procedimiento de Failback

> Tan importante como el failover, el failback mal ejecutado puede causar una segunda interrupción.

### 6.1 Criterios para Failback

- ✅ Causa del fallo original identificada y resuelta
- ✅ Enlace primario estable durante mínimo [X] minutos
- ✅ Sin tráfico crítico en vuelo que pueda interrumpirse
- ✅ Ventana de cambio acordada (si aplica)

### 6.2 Procedimiento de Failback Controlado

```
1. Confirmar que el enlace primario está estable:
   show ip bgp neighbor [IP-PEER] | include state|prefix
   # BGP state debe ser Established, prefijos recibidos correctos

2. Preparar failback (pre-notify a usuarios si impacto esperado):
   Informar en Teams: #network-ops "Failback planificado en 5 min hacia [enlace]"

3. Restaurar routing primario:
   # Si fue manual (BGP shutdown): no shutdown neighbor
   # Si fue automático (SD-WAN): verificar que policy reconvergió sola

4. Monitorizar durante 15 minutos:
   - Dashboard WAN: latencia, pérdida, throughput
   - PagerDuty: sin alertas nuevas
   - Test ping hacia probes críticos

5. Confirmar estado normal y documentar en changelog de incidente
```

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
