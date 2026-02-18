# Dashboard Guidelines — Diseño de Dashboards

> **Propietario:** @sre-team

## 1. Dashboard Types

| Type | Purpose | Refresh Rate | Audience |
|------|---------|-------------|----------|
| **Operational** | Day-to-day monitoring | 30s | SRE/Ops |
| **Tactical** | Service health overview | 1min | Dev Teams |
| **Strategic** | Business KPIs | 5min | Management |
| **Incident Response** | Troubleshooting | 10s | On-call |

---

## 2. Layout Principles

```
┌─────────────────────────────────────────┐
│  TITLE: Service Name - Overview         │
├─────────────────────────────────────────┤
│  [Critical KPI 1] [Critical KPI 2] [...]│ ← Single-stat panels arriba
├─────────────────────────────────────────┤
│  [Time series graph: Latency p50/p95]   │ ← Gráficas principales
│  [Time series graph: Requests/sec]      │
│  [Time series graph: Error rate]        │
├─────────────────────────────────────────┤
│  [Heat map: Response time distribution] │ ← Visualizaciones avanzadas
└─────────────────────────────────────────┘
```

**Golden rules:**
- ✅ Most important metrics at the top
- ✅ Use consistent colors (red=bad, green=good, yellow=warning)
- ✅ Include time range selector
- ✅ Label all axes
- ❌ No más de 12 panels por dashboard (split si necesario)

---

## 3. Panel Types by Use Case

| Use Case | Panel Type | Example |
|----------|-----------|---------|
| Single current value | **Stat** | Current CPU usage: 45% |
| Trend over time | **Time series** | Requests/sec last 6h |
| Multiple dimensions | **Table** | Top 10 slowest endpoints |
| Distribution | **Heatmap** | Response time distribution |
| Comparison | **Bar gauge** | Error rate by service |

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @sre-team | Creación inicial |
