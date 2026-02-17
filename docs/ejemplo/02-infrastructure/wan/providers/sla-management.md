# Gestión de SLAs WAN

> **Propietario:** @network-team @wan-lead  
> **Última actualización:** YYYY-MM-DD  
> **Estado:** Draft | Review | Aprobado  
> **Versión:** 1.0

Control y seguimiento de los SLAs contratados con proveedores WAN: métricas comprometidas, metodología de medición, proceso de reclamación y reporting mensual.

---

## 1. SLAs Contratados por Proveedor

### 1.1 PRV-001 — [Proveedor MPLS]

| Métrica | SLA Contratado | Penalización si incumple | Periodo de medición |
|---------|---------------|--------------------------|---------------------|
| Disponibilidad del servicio | ≥ 99,95% mensual | [X]% crédito en factura por cada 0,1% por debajo | Mensual |
| Latencia HQ → DC (RTT) | ≤ [X] ms | Crédito si supera umbral > [N] horas/mes | Mensual |
| Pérdida de paquetes | ≤ 0,1% | Crédito proporcional | Mensual |
| Jitter | ≤ [X] ms | No penalizado / Crédito | Mensual |
| MTTR incidencia P1 | ≤ [X] horas | [X]% crédito por hora adicional | Por incidente |
| MTTR incidencia P2 | ≤ [X] horas | Crédito según contrato | Por incidente |
| Tiempo de aprovisionamiento circuito nuevo | ≤ [X] días hábiles | [X]€ por día de retraso | Por solicitud |

**Cláusula de exclusión de SLA (excluidos del cálculo):**
- Mantenimientos planificados notificados con ≥ [X] horas de antelación
- Causas de fuerza mayor
- Incidencias causadas por equipo del cliente (CE, LAN)
- Ventanas de mantenimiento acordadas mutuamente

---

### 1.2 PRV-002 — [ISP Internet]

| Métrica | SLA Contratado | Penalización | Periodo |
|---------|---------------|-------------|---------|
| Disponibilidad | ≥ 99,9% mensual | Crédito proporcional a downtime | Mensual |
| Latencia a peer de referencia | ≤ [X] ms | No penalizado | Mensual |
| Ratio de pérdida paquetes | ≤ [X]% | Crédito si supera | Mensual |
| MTTR incidencia | ≤ [X] horas | Crédito por hora | Por incidente |

---

### 1.3 PRV-003 — [SD-WAN Vendor] (Soporte)

| Métrica | SLA Contratado |
|---------|---------------|
| Respuesta inicial P1 | < [X] horas |
| Resolución P1 | < [X] horas |
| Respuesta inicial P2 | < [X] horas |
| Resolución P2 | < [X] horas |
| Disponibilidad portal/plataforma | ≥ 99,99% mensual |

---

## 2. Metodología de Medición

### 2.1 Herramientas de Medición

| Métrica | Herramienta | Protocolo | Frecuencia | Retención datos |
|---------|-------------|-----------|-----------|----------------|
| Disponibilidad circuitos | [Zabbix / SolarWinds / LibreNMS] | SNMP ifOperStatus + ICMP | Cada 1 min | 13 meses |
| Latencia RTT | [Zabbix / IPSLA Cisco / TWAMP] | ICMP / TWAMP | Cada 5 min | 13 meses |
| Pérdida de paquetes | [Zabbix / IPSLA / TWAMP] | ICMP | Cada 5 min | 13 meses |
| Jitter | [IPSLA / TWAMP / SD-WAN] | UDP / TWAMP | Cada 5 min | 13 meses |
| Throughput / utilización | [Zabbix / SNMP] | SNMP ifHCInOctets | Cada 5 min | 13 meses |

### 2.2 Puntos de Medición

```
  [Probe interno CE-HQ] ─────────── MPLS ─────────── [Probe interno CE-Site]
         │                                                       │
  Zabbix mide:                                          Zabbix mide:
  - Disponibilidad                                      - Disponibilidad
  - Latencia RTT                                        - Latencia RTT
  - Pérdida paquetes                                    - Pérdida paquetes
```

> ⚠️ **Importante:** La medición desde el CE (equipo del cliente) puede diferir de la medición del proveedor desde su PE. En caso de disputa, aplica la medición del **proveedor** según contrato, pero documentar la nuestra para negociación.

### 2.3 Cálculo de Disponibilidad

```
Disponibilidad (%) = ((Minutos_totales_mes - Minutos_downtime) / Minutos_totales_mes) × 100

Minutos en mes de 30 días = 43.200
Minutos en mes de 31 días = 44.640

Umbral 99,95% en mes de 30 días = máx. 21,6 minutos de downtime
Umbral 99,90% en mes de 30 días = máx. 43,2 minutos de downtime

El downtime se cuenta desde la primera alerta confirmada hasta la resolución verificada.
Los mantenimientos planificados se RESTAN del tiempo total del denominador.
```

---

## 3. Dashboard y Reporting

### 3.1 Dashboard de SLAs en Tiempo Real

| Dashboard | URL | Actualización |
|-----------|-----|--------------|
| SLA Overview WAN | [URL Grafana] | Tiempo real |
| Disponibilidad por circuito (mes actual) | [URL Grafana] | Cada 5 min |
| Histórico latencia HQ-sites | [URL Grafana] | Cada 5 min |
| Alertas SLA activas | [URL PagerDuty] | Tiempo real |

### 3.2 Informe Mensual de SLAs

**Destinatarios:** @wan-lead, @network-director, @service-management  
**Fecha de envío:** Antes del día 5 del mes siguiente  
**Formato:** Email + PDF adjunto + datos en [SharePoint/Confluence]

**Contenido del informe:**

```markdown
## Informe SLA WAN — [Mes] [Año]

### Resumen ejecutivo
| Proveedor | SLA comprometido | SLA real | Cumplimiento | Créditos a reclamar |
|-----------|-----------------|----------|-------------|---------------------|
| PRV-001   | 99,95%          | XX,XX%   | ✅/❌        | [X]€ / N/A          |
| PRV-002   | 99,90%          | XX,XX%   | ✅/❌        | [X]€ / N/A          |

### Incidencias del mes
| Ticket | Proveedor | CID | Inicio | Resolución | Duración | SLA OK |
| ...    | ...       | ... | ...    | ...        | ...      | ...    |

### Tendencias
[Gráfica de disponibilidad últimos 12 meses por proveedor]

### Acciones abiertas
[Issues de calidad persistentes, reclamaciones en curso]
```

---

## 4. Proceso de Reclamación de Créditos (SLA Breach)

### 4.1 Criterios para Iniciar Reclamación

Iniciar reclamación cuando:
- La disponibilidad mensual del proveedor cae por debajo del umbral contratado, **Y**
- El evento está documentado en nuestros sistemas de monitoring, **Y**
- El proveedor ha confirmado el incidente (ticket abierto y cerrado)

### 4.2 Procedimiento de Reclamación

```
Paso 1 — Detectar incumplimiento (automático, fin de mes)
│  El informe mensual identifica si hay SLA breach
│  Responsable: @wan-lead
│
Paso 2 — Recopilar evidencia (días 1–3 del mes siguiente)
│  - Exportar datos de Zabbix/Grafana del período afectado
│  - Descargar tickets de incidente del portal del proveedor
│  - Calcular minutos de downtime y % de disponibilidad real
│  Responsable: @network-engineer
│
Paso 3 — Notificar al proveedor (día 5 del mes siguiente)
│  - Email formal a Account Manager + CC a NOC
│  - Adjuntar: cálculo de disponibilidad, evidencias, importe del crédito
│  - Solicitar confirmación de recepción y plazo de resolución
│  Responsable: @wan-lead
│
Paso 4 — Seguimiento (día 10 si no hay respuesta)
│  - Recordatorio formal por email
│  - Si no responde en 5 días hábiles → escalar a @network-director
│  Responsable: @wan-lead
│
Paso 5 — Confirmación del crédito
│  - El proveedor confirma el crédito (nota de crédito o descuento en factura)
│  - Comunicar a Finanzas/Compras para verificar en factura
│  - Registrar en historial de reclamaciones (tabla abajo)
│  Responsable: @wan-lead + Finanzas
│
Paso 6 — Escalado si rechazan el crédito
   - Consultar a Legal / Procurement para revisión contractual
   - Si persiste: proceso formal de disputa según contrato
   Responsable: @network-director + Legal
```

### 4.3 Historial de Reclamaciones

| Mes | Proveedor | CID | Downtime (min) | Disponibilidad real | Crédito reclamado | Estado | Crédito recibido |
|-----|-----------|-----|---------------|--------------------|--------------------|--------|-----------------|
| YYYY-MM | PRV-001 | [CID] | [N] | [X,XX]% | [X]€ | ✅ Cobrado / 🔄 En trámite / ❌ Rechazado | [X]€ |

---

## 5. Revisión de Contratos y Renovaciones

| Proveedor | Próxima renovación | Acción requerida | Responsable | Estado |
|-----------|-------------------|-----------------|-------------|--------|
| PRV-001 | YYYY-MM-DD | Negociar mejora SLA latencia | @wan-lead + Procurement | 🔄 Pendiente |
| PRV-002 | YYYY-MM-DD | Evaluar alternativas de mercado | @wan-lead | ⬜ Por iniciar |

**Proceso de revisión anual (cada Q4):**

1. Revisar incumplimientos del año y créditos recibidos
2. Evaluar satisfacción con el proveedor (rendimiento técnico + servicio)
3. Solicitar ofertas del mercado para benchmark de precio y SLA
4. Proponer a @network-director: renovar / renegociar / cambiar proveedor
5. Coordinar con Procurement para proceso formal de contratación si aplica

---

## Changelog

| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @wan-lead | Creación inicial |
