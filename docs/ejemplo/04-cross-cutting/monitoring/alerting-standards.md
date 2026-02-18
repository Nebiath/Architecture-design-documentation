# Alerting Standards — Estándares de Alertas

> **Propietario:** @sre-team

## 1. Alert Severity Levels

| Severity | Definition | Response | Example |
|----------|-----------|----------|---------|
| **P1 - Critical** | Servicio Tier 1 caído, impacto usuarios masivo | Inmediato (PagerDuty) | Database down, website offline |
| **P2 - High** | Degradación severa o Tier 2 caído | < 30 min | High latency (>2s), partial outage |
| **P3 - Medium** | Problema no urgente pero requiere atención | < 4 horas | Disk usage 85%, certificate expiring 30d |
| **P4 - Low** | Informacional | Next business day | Performance tuning opportunity |

---

## 2. Alert Naming Convention

```
[SEVERITY] [COMPONENT] [METRIC] [THRESHOLD]

Examples:
✅ [CRITICAL] API-Gateway ResponseTime >5s
✅ [HIGH] PostgreSQL-Prod ConnectionPool 90%
✅ [MEDIUM] Disk-Usage /var 85%
❌ Database problem (too vague)
```

---

## 3. Alert Routing

| Alert Severity | Notification Channel | Acknowledgement Required |
|---------------|---------------------|------------------------|
| P1 - Critical | PagerDuty phone call + SMS + email | Yes (< 15 min) |
| P2 - High | PagerDuty push + email | Yes (< 30 min) |
| P3 - Medium | Email + Slack | Optional |
| P4 - Low | Email only | No |

---

## 4. Alert Fatigue Prevention

**Reglas:**
- ❌ NO alertar en lo mismo cada 1 min (usar aggregation)
- ✅ Alert de-duplication window: 15 minutos
- ✅ Silence durante ventanas de mantenimiento
- ✅ Auto-resolve cuando métrica vuelve a normal
- ✅ Revisar alertas no-actionables mensualmente (remove/adjust)

**Target:** < 10 alerts/semana por servicio

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @sre-team | Creación inicial |
