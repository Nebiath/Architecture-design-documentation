# Cross-Cutting Concerns — Servicios Transversales

> **Propietario:** @architecture-team @sre-team  
> **Última actualización:** YYYY-MM-DD  
> **Versión:** 1.0

Documentación de servicios y políticas transversales que aplican a toda la infraestructura IT, independientemente del departamento o tecnología específica.

---

## 📋 Áreas Cross-Cutting

### 🔒 Seguridad
- [Encryption Standards](./security/encryption-standards.md) — Cifrado en tránsito y en reposo
- [Certificate Management](./security/certificate-management.md) — PKI, renovación, inventario
- [Vulnerability Management](./security/vulnerability-management.md) — Scanning, patching, CVEs
- [Security Monitoring](./security/security-monitoring.md) — SIEM, threat detection, SOC

### 📊 Monitorización
- [Monitoring Strategy](./monitoring/monitoring-strategy.md) — Estrategia global, herramientas, métricas
- [Alerting Standards](./monitoring/alerting-standards.md) — Umbrales, severidades, on-call
- [Dashboard Guidelines](./monitoring/dashboard-guidelines.md) — Diseño de dashboards, KPIs

### 📝 Logging
- [Log Aggregation](./logging/log-aggregation.md) — ELK, Splunk, centralización
- [Log Retention](./logging/log-retention.md) — Políticas de retención, compliance
- [Log Analysis](./logging/log-analysis.md) — Queries, alertas, investigación

### 💾 Backup & Recovery
- [Backup Strategy](./backup-recovery/backup-strategy.md) — 3-2-1 rule, RPO/RTO
- [Retention Policy](./backup-recovery/retention-policy.md) — Retención por tipo de dato
- [Disaster Recovery](./backup-recovery/disaster-recovery.md) — DR plan, failover, tests

### 🔄 Change Management
- [Change Process](./change-management/change-process.md) — RFC workflow, aprobaciones
- [CAB Procedures](./change-management/cab-procedures.md) — Change Advisory Board
- [Emergency Changes](./change-management/emergency-changes.md) — Proceso de emergencia

### 🚨 Incident Management
- [Incident Process](./incident-management/incident-process.md) — Detección, respuesta, resolución
- [Escalation Matrix](./incident-management/escalation-matrix.md) — Niveles de escalado
- [Post-Mortem Template](./incident-management/post-mortem-template.md) — RCA, lecciones aprendidas

---

## 🎯 Principios Cross-Cutting

**Aplicabilidad universal:** Estos estándares aplican a:
- ✅ Todos los departamentos (WAN, LAN, Linux, Windows, Cloud, etc.)
- ✅ Todos los entornos (Producción, Pre-producción, Desarrollo)
- ✅ Todos los proveedores (On-premise, Azure, AWS, SaaS)

**Excepciones:** Requieren aprobación de @architecture-team + @security-team + @cto-office

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @architecture-team | Creación inicial |
