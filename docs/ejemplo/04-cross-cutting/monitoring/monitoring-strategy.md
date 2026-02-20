# Monitoring Strategy — Estrategia Global

> **Propietario:** @sre-team @architecture-team

## 1. Monitoring Stack

| Layer | Tool | Metrics | Logs | Traces |
|-------|------|---------|------|--------|
| **Infrastructure** | Zabbix / Prometheus | ✅ | — | — |
| **Applications** | Prometheus + Grafana | ✅ | — | ✅ (Jaeger) |
| **Cloud (Azure)** | Azure Monitor | ✅ | ✅ | ✅ (AppInsights) |
| **Logs** | ELK / Splunk | — | ✅ | — |
| **Synthetic monitoring** | Pingdom / UptimeRobot | ✅ | — | — |

---

## 2. Golden Signals

**Para cada servicio, monitorizar:**
1. **Latency:** Tiempo de respuesta (p50, p95, p99)
2. **Traffic:** Requests/segundo
3. **Errors:** Error rate (%)
4. **Saturation:** CPU, RAM, disk, network utilization

---

## 3. Tiering de Servicios

| Tier | RTO | Monitoring Interval | Alerting |
|------|-----|-------------------|----------|
| **Tier 1 (Crítico)** | < 1 hora | 1 minuto | 24×7 PagerDuty |
| **Tier 2 (Importante)** | < 4 horas | 5 minutos | Business hours + on-call |
| **Tier 3 (Estándar)** | < 24 horas | 15 minutos | Business hours only |

---

## 4. Dashboards Estándar

| Dashboard | URL | Audience |
|-----------|-----|----------|
| Infrastructure Overview | [URL Grafana] | SRE Team |
| Application Health | [URL Grafana] | Dev Teams |
| Business Metrics | [URL Grafana] | Management |
| Incident Response | [URL] | On-call engineers |

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @sre-team | Creación inicial |
