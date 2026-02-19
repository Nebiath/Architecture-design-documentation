# Industrial Image Backup to Azure — Service Ownership & RACI

> **Version:** 1.0  
> **Fecha:** 2025-02-19  
> **Estado:** Approved  
> **Autor:** Arquitectura IT  
> **Service Owner:** Arquitectura IT  
> **Última Revisión:** 2025-02-19

---

## 1. Service Owner

**Primary Service Owner:**
- **Name:** [Architecture Lead Name]
- **Team:** Arquitectura IT
- **Email:** architecture-team@company.com
- **Phone:** +XX XXX XXX XXX
- **Responsibilities:**
  - Decisiones estratégicas sobre el servicio
  - Aprobación de cambios arquitectónicos mayores
  - Presupuesto y forecasting de costes
  - Escalation point para problemas críticos
  - Review trimestral de KPIs del servicio

**Backup Service Owner:**
- **Name:** [Cloud Architect Name]
- **Team:** Public Cloud
- **Email:** cloud-team@company.com
- **Responsibilities:** Mismo que primary en ausencia

---

## 2. RACI Matrix

### 2.1 Diseño y Arquitectura

| Actividad | Arquitectura IT | Public Cloud | IT Operations | IT Local (Planta) | Negocio | Security |
|-----------|----------------|--------------|---------------|-------------------|---------|----------|
| **Diseño inicial arquitectura** | **R** A | C | I | I | C | C |
| **Selección de Azure services** | A | **R** | I | I | I | C |
| **Definición de SLOs** | A | **R** | I | I | C | I |
| **Revisión de seguridad (Purdue)** | C | C | I | I | I | **R** A |
| **Aprobación presupuesto** | A | I | I | I | **R** | I |
| **Documentación arquitectura** | **R** A | C | I | I | I | I |

### 2.2 Implementación y Deployment

| Actividad | Arquitectura IT | Public Cloud | IT Operations | IT Local (Planta) | Negocio | Security |
|-----------|----------------|--------------|---------------|-------------------|---------|----------|
| **Provisión Azure resources** | C | **R** A | I | I | I | I |
| **Deployment VM Gateway** | C | **R** | A | C | I | I |
| **Instalación scripts PowerShell** | I | **R** A | C | I | I | I |
| **Configuración firewall Checkpoint** | C | I | **R** A | C | I | C |
| **Configuración OT-side scripts** | I | I | C | **R** A | I | I |
| **Testing end-to-end** | C | **R** | A | A | I | I |
| **Go-live approval** | A | C | I | **R** | **R** | I |

### 2.3 Operaciones Diarias

| Actividad | Arquitectura IT | Public Cloud | IT Operations | IT Local (Planta) | Negocio | Security |
|-----------|----------------|--------------|---------------|-------------------|---------|----------|
| **Monitorización de uploads** | I | C | **R** A | I | I | I |
| **Respuesta a alertas (<4h)** | I | C | **R** A | I | I | I |
| **Provision de datos (generación imágenes)** | I | I | I | I | **R** A | I |
| **Review de logs semanalmente** | I | C | **R** A | I | I | I |
| **Gestión de capacidad local (disk)** | I | C | **R** A | I | I | I |

### 2.4 Incidentes y Troubleshooting

| Actividad | Arquitectura IT | Public Cloud | IT Operations | IT Local (Planta) | Negocio | Security |
|-----------|----------------|--------------|---------------|-------------------|---------|----------|
| **Incidente VM Gateway failure** | I | **R** A | A | C | I | I |
| **Incidente Azure region down** | C | **R** A | C | I | I | I |
| **Incidente firewall down** | I | I | A | **R** A | I | C |
| **Incidente OT equipment failure** | I | I | I | **R** A | C | I |
| **Incidente datos no llegan a Azure** | C | **R** A | A | C | I | I |
| **Post-mortem analysis** | C | **R** A | C | C | I | I |
| **Communication a Negocio** | C | **R** | C | I | I | I |

### 2.5 Mantenimiento y Cambios

| Actividad | Arquitectura IT | Public Cloud | IT Operations | IT Local (Planta) | Negocio | Security |
|-----------|----------------|--------------|---------------|-------------------|---------|----------|
| **Actualización scripts PowerShell** | C | **R** A | I | I | I | I |
| **Patching VM Gateway** | I | C | **R** A | I | I | I |
| **Upgrade de AzCopy** | I | **R** | A | I | I | I |
| **Cambios de firewall rules** | C | I | C | **R** A | I | A |
| **Añadir nueva planta** | C | **R** A | A | C | I | C |
| **Revisión de costes Azure** | **R** A | C | I | I | C | I |

### 2.6 Compliance y Auditoría

| Actividad | Arquitectura IT | Public Cloud | IT Operations | IT Local (Planta) | Negocio | Security |
|-----------|----------------|--------------|---------------|-------------------|---------|----------|
| **Auditoría de seguridad anual** | C | C | C | I | I | **R** A |
| **Review de logs de acceso** | I | C | C | I | I | **R** A |
| **Compliance con Purdue model** | C | C | C | C | I | **R** A |
| **GDPR data residency review** | C | C | I | I | **R** | A |
| **Retention policy compliance** | **R** A | C | I | I | C | C |

---

## 3. Legend

- **R** = Responsible (Ejecuta la tarea)
- **A** = Accountable (Responsable final, aprueba)
- **C** = Consulted (Se le consulta, proporciona input)
- **I** = Informed (Se le informa del resultado)

**Nota:** Puede haber múltiples **R** (varios ejecutores), pero solo un **A** (un solo accountable que aprueba).

---

## 4. Escalation Path

### 4.1 Technical Escalation

```
Level 1: IT Operations (on-call)
   └─ Response time: 30 minutes
   └─ Scope: Restart services, basic troubleshooting
         │
         ▼ (si no resuelto en 1 hora)
Level 2: Public Cloud Team
   └─ Response time: 1 hour
   └─ Scope: Azure issues, script debugging, advanced troubleshooting
         │
         ▼ (si no resuelto en 4 horas)
Level 3: Arquitectura IT + Cloud Director
   └─ Response time: 2 hours
   └─ Scope: Architectural decisions, vendor escalation
```

### 4.2 Business Escalation

```
P1 Incident (Data loss risk, >24h sin backups):
   └─ Notify immediately:
      • Service Owner (Arquitectura IT)
      • Negocio (Plant Manager)
      • Cloud Director
      
P2 Incident (Degradation, algunos archivos fallan):
   └─ Notify within 4 hours:
      • IT Operations Manager
      • Service Owner
      
P3 Incident (Minor issue, self-recovering):
   └─ Daily report to Service Owner
```

---

## 5. On-Call Rotation

### 5.1 Primary On-Call: IT Operations

| Week | Primary | Backup | Hours |
|------|---------|--------|-------|
| Week 1 | John Smith | Maria García | 24×7 |
| Week 2 | Maria García | Peter Johnson | 24×7 |
| Week 3 | Peter Johnson | John Smith | 24×7 |
| Week 4 | John Smith | Maria García | 24×7 |

**PagerDuty Schedule:** https://company.pagerduty.com/schedules/INDBACK-OPS

### 5.2 Secondary On-Call: Public Cloud Team

| Week | Primary | Backup | Hours |
|------|---------|--------|-------|
| Week 1 | Cloud Eng A | Cloud Eng B | Business hours only |
| Week 2 | Cloud Eng B | Cloud Eng C | Business hours only |

**Note:** Secondary on-call solo responde a escalations desde L1, no directamente a alertas.

---

## 6. Communication Channels

### 6.1 Internal Communication

| Channel | Purpose | Members |
|---------|---------|---------|
| **Slack #industrial-backup** | Daily ops, questions, updates | IT Ops, Cloud Team, Arquitectura IT |
| **Email DL: industrial-backup@company.com** | Formal communication, incidents | All stakeholders |
| **Teams Channel: Industrial Backup** | Meetings, collaboration | All stakeholders |
| **PagerDuty** | Incident alerts | IT Ops (primary), Cloud Team (secondary) |

### 6.2 External Communication (to Plants)

| Event | Channel | Frequency | Owner |
|-------|---------|-----------|-------|
| **Service outage (P1)** | Email + Phone | Immediately | Public Cloud |
| **Planned maintenance** | Email | 7 days advance | IT Operations |
| **Monthly status report** | Email | Monthly (1st of month) | Arquitectura IT |
| **Quarterly review** | Teams Meeting | Quarterly | Arquitectura IT + Negocio |

---

## 7. Key Contacts

### 7.1 Core Team

| Role | Name | Email | Phone | Timezone |
|------|------|-------|-------|----------|
| **Service Owner** | [Name] | architecture-team@company.com | +XX XXX XXX XXX | CET |
| **Tech Lead (Cloud)** | [Name] | cloud-team@company.com | +XX XXX XXX XXX | CET |
| **Ops Lead** | [Name] | it-ops@company.com | +XX XXX XXX XXX | CET |
| **Security Lead** | [Name] | security-team@company.com | +XX XXX XXX XXX | CET |

### 7.2 Plant Contacts (Examples)

| Plant Code | IT Local Contact | Negocio Contact | Timezone |
|------------|-----------------|-----------------|----------|
| EU01 | Juan García (juan.garcia@plant-eu01.com) | María López | CET |
| EU02 | Peter Schmidt (peter.schmidt@plant-eu02.com) | Anna Müller | CET |
| US01 | Michael Brown (michael.brown@plant-us01.com) | Sarah Wilson | EST |

**Full list:** See SharePoint → Plant Contacts

---

## 8. Service Level Agreements (Internal)

### 8.1 Response Times

| Severity | Detection to Response | Detection to Resolution |
|----------|---------------------|----------------------|
| **P1** (Service down, data loss risk) | 30 minutes | 4 hours |
| **P2** (Degraded, partial failure) | 2 hours | 24 hours |
| **P3** (Minor issue) | 24 hours | 1 week |
| **P4** (Enhancement request) | 1 week | Best effort |

### 8.2 Availability Targets

| Component | Target Availability | Measured How |
|-----------|-------------------|--------------|
| **Upload Service** | 99.5% (ventana 22:00-06:00) | % successful uploads |
| **VM Gateway** | 99.9% | Uptime monitoring |
| **Azure Storage** | 99.99% | Azure SLA (fuera de control) |

---

## 9. Decision Authority

### 9.1 Change Approval Matrix

| Type of Change | Approval Required | Examples |
|----------------|------------------|----------|
| **Minor (Standard)** | IT Operations | Restart service, update logs retention |
| **Normal** | Public Cloud Team | Update scripts, change monitoring threshold |
| **Major** | Arquitectura IT + Security | Change Azure region, new firewall rules, add >10 plants |
| **Emergency** | Service Owner (post-facto approval) | Hotfix during P1 incident |

### 9.2 Budget Authority

| Decision | Approval Required | Threshold |
|----------|------------------|-----------|
| **Monthly Azure spend** | Arquitectura IT | < 10% variance from forecast |
| **Add new plant** | Arquitectura IT | < $500/mes per plant |
| **Change tier strategy** | Arquitectura IT + CFO | Impact > $5k/mes |
| **New Azure service** | Arquitectura IT + CTO | Any amount |

---

## 10. Handover Procedure

### 10.1 When Service Owner Changes

**Checklist:**
- [ ] Update this document with new Service Owner name/contact
- [ ] Update PagerDuty escalation policy
- [ ] Grant Azure permissions (Owner role on Resource Group)
- [ ] Add to Slack channel #industrial-backup as admin
- [ ] Schedule handover meeting (2 hours)
- [ ] Review:
  - Architecture documentation
  - Current issues/risks
  - Budget and cost trends
  - Upcoming projects (new plants)
- [ ] Introduce to key stakeholders (email to all plants)

### 10.2 When On-Call Engineer Changes

**Checklist:**
- [ ] Update PagerDuty schedule
- [ ] Grant access to VM Gateway (local admin)
- [ ] Grant access to Azure Portal (Reader role)
- [ ] Share runbooks location
- [ ] Schedule shadow shift (new engineer follows current on-call for 1 week)

---

## 11. Change History

| Version | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 1.0 | 2025-02-19 | Arquitectura IT | Versión inicial — RACI definida |

---

**Notes:**
- Este documento debe revisarse cuando hay cambios de personal o responsabilidades
- RACI debe ser acordada por todos los stakeholders antes de go-live
- Conflictos de RACI se escalan a Service Owner para resolución
