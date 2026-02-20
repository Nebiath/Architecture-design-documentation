# [Nombre del Servicio] - Service Ownership & RACI

> **Version:** 1.0  
> **Fecha:** YYYY-MM-DD  
> **Service Owner:** [Nombre]  
> **Última Revisión:** YYYY-MM-DD

---

## 1. Service Overview

**Nombre del Servicio:** [Service Name]  
**Tier:** [Tier 1 - Critical | Tier 2 - Important | Tier 3 - Normal]  
**Estado:** [Active | Deprecated | Sunset]  
**Repositorio:** https://github.com/company/service-name  
**Dashboard:** https://grafana.company.com/d/service-name

---

## 2. Service Ownership

### 2.1 Primary Owner

**Service Owner:** [Nombre Completo]  
**Role:** [Engineering Manager / Tech Lead]  
**Email:** owner@company.com  
**Slack:** @owner  
**Timezone:** UTC+1 (Madrid)

**Responsabilidades:**
- ✅ Decisiones arquitectónicas finales
- ✅ Aprobación de cambios mayores
- ✅ Budget y resource allocation
- ✅ Escalation point para incidentes P0/P1
- ✅ Quarterly reviews del servicio
- ✅ Stakeholder communication

### 2.2 Technical Lead

**Tech Lead:** [Nombre Completo]  
**Email:** tech-lead@company.com  
**Slack:** @tech-lead  
**Timezone:** UTC+1 (Madrid)

**Responsabilidades:**
- ✅ Technical roadmap
- ✅ Code reviews y architecture decisions
- ✅ Mentorship del equipo
- ✅ On-call schedule management
- ✅ Performance optimization
- ✅ Security compliance

### 2.3 Product Owner

**Product Owner:** [Nombre Completo]  
**Email:** product@company.com  
**Slack:** @product  
**Timezone:** UTC+1 (Madrid)

**Responsabilidades:**
- ✅ Feature prioritization
- ✅ Business requirements
- ✅ Roadmap planning
- ✅ Stakeholder management
- ✅ Success metrics definition

---

## 3. Team Structure

### 3.1 Core Team

| Nombre | Role | Responsabilidad Principal | Backup |
|--------|------|--------------------------|--------|
| Alice Johnson | Senior SRE | Infrastructure, monitoring | Bob Smith |
| Bob Smith | SRE | CI/CD, deployments | Alice Johnson |
| Carlos García | Backend Developer | API development | Diana Lee |
| Diana Lee | Backend Developer | Database, migrations | Carlos García |
| Emma Wilson | QA Engineer | Testing, quality gates | Frank Miller |

### 3.2 Extended Team

| Nombre | Role | Departamento | When to Involve |
|--------|------|--------------|-----------------|
| Security Team | Security Engineers | Security | Vulnerability, compliance |
| DBA Team | Database Admins | Infrastructure | Database issues, migrations |
| Network Team | Network Engineers | Infrastructure | Routing, DNS, CDN issues |
| Legal Team | Lawyers | Legal | Data privacy, compliance |

---

## 4. RACI Matrix

### 4.1 Development & Deployment

| Actividad | Service Owner | Tech Lead | Developer | SRE | Product | Security | QA |
|-----------|--------------|-----------|-----------|-----|---------|----------|-----|
| Feature development | A | R | R | C | A | I | C |
| Code review | I | A | R | C | I | I | I |
| Security review | I | C | I | C | I | A/R | I |
| Testing | I | C | C | C | I | I | A/R |
| Staging deployment | I | A | C | R | I | I | C |
| Production deployment | A | A | I | R | I | C | I |
| Rollback decision | A | A | C | R | I | I | I |
| Hotfix approval | A | R | R | C | C | I | I |

### 4.2 Operations & Maintenance

| Actividad | Service Owner | Tech Lead | Developer | SRE | DBA | Security |
|-----------|--------------|-----------|-----------|-----|-----|----------|
| On-call primary | I | C | I | R | I | I |
| Incident response (P0/P1) | A | A | C | R | C | C |
| Incident response (P2/P3) | I | C | R | R | I | I |
| Post-mortem | A | R | C | R | I | C |
| Monitoring setup | I | A | C | R | I | I |
| Alert configuration | I | A | C | R | I | I |
| Performance tuning | I | A | R | R | C | I |
| Database maintenance | C | C | I | C | A/R | I |
| Security patching | A | C | I | R | I | A/R |
| Backup verification | I | C | I | R | R | I |

### 4.3 Architecture & Planning

| Actividad | Service Owner | Tech Lead | Architect | SRE | Product | Legal |
|-----------|--------------|-----------|-----------|-----|---------|-------|
| Architecture design | A | R | R | C | I | I |
| Technology selection | A | R | C | C | I | I |
| Capacity planning | A | C | I | R | I | I |
| Cost optimization | A | C | I | R | I | I |
| DR planning | A | C | C | R | I | I |
| Compliance review | A | I | I | C | I | R |
| SLO definition | A | R | C | R | R | I |
| Roadmap planning | A | C | I | I | R | I |

### 4.4 Documentation

| Actividad | Service Owner | Tech Lead | Developer | SRE | QA |
|-----------|--------------|-----------|-----------|-----|-----|
| Architecture docs | A | R | C | C | I |
| API documentation | C | A | R | I | I |
| Runbooks | I | C | C | R | I |
| Deployment guides | I | C | C | R | I |
| User documentation | I | C | R | I | R |
| Change logs | I | C | R | C | I |

**Leyenda:**
- **R** = Responsible (hace el trabajo)
- **A** = Accountable (responsable final, solo uno por actividad)
- **C** = Consulted (consultado antes de decisión)
- **I** = Informed (informado después de decisión)

---

## 5. On-Call Rotation

### 5.1 Schedule

**Rotation Type:** Weekly rotation  
**Coverage:** 24/7  
**Handoff:** Lunes 9:00 AM UTC

| Week | Primary On-Call | Secondary On-Call | Manager On-Call |
|------|----------------|-------------------|-----------------|
| Week 1 | Alice Johnson | Bob Smith | Tech Lead |
| Week 2 | Bob Smith | Carlos García | Tech Lead |
| Week 3 | Carlos García | Diana Lee | Tech Lead |
| Week 4 | Diana Lee | Alice Johnson | Tech Lead |

**PagerDuty Schedule:** https://company.pagerduty.com/schedules/ABC123

### 5.2 On-Call Responsibilities

**Primary On-Call:**
- Respond to alerts within 15 minutes
- Triage incidents and assign severity
- Execute runbooks
- Escalate to secondary if needed
- Update incident status
- Write incident summary

**Secondary On-Call:**
- Available as backup (30 min response time)
- Support primary during complex incidents
- Take over if primary is unavailable
- Review incident handling

**Manager On-Call:**
- Available for P0/P1 escalation
- Coordinate with stakeholders
- Make business decisions during incidents
- Approve emergency changes

### 5.3 On-Call Procedures

**When Alert Fires:**

```
1. Acknowledge alert en PagerDuty (< 15 min)
2. Check service dashboard: https://grafana.company.com/d/service-name
3. Check #incidents Slack channel
4. Determinar severity (ver sección 6)
5. Seguir runbook correspondiente
6. Update incident en PagerDuty cada 30 min
7. Escalate si no se resuelve en 1 hora (P0/P1) o 4 horas (P2)
```

**Handoff Checklist:**

```markdown
- [ ] Review open incidents from last week
- [ ] Check monitoring for anomalies
- [ ] Review upcoming deployments/maintenance
- [ ] Verify PagerDuty schedule is correct
- [ ] Test pager (send test alert)
- [ ] Confirm laptop/phone charged
- [ ] VPN credentials working
- [ ] Access to all systems verified
```

---

## 6. Incident Severity Matrix

### 6.1 Severity Definitions

| Severity | Description | Response Time | Examples | Escalation |
|----------|-------------|---------------|----------|------------|
| **P0** | Complete outage, revenue impact | 15 min | API down, database unreachable, security breach | Immediate to Manager |
| **P1** | Major degradation, customer impact | 30 min | High latency, partial outage, data inconsistency | 1 hour to Manager |
| **P2** | Minor degradation, workaround exists | 1 hour | Single region slow, non-critical feature down | 4 hours to Tech Lead |
| **P3** | No customer impact, low priority | 4 hours | Logging issues, monitoring gaps, doc updates | Next business day |
| **P4** | Informational, future work | Best effort | Tech debt, nice-to-have improvements | Backlog |

### 6.2 Incident Response Team

**P0/P1 Incidents:**

| Role | Responsibility | Person |
|------|---------------|--------|
| Incident Commander | Coordinate response, make decisions | Primary On-Call |
| Technical Lead | Debug and fix | Secondary On-Call or Tech Lead |
| Communications | Update stakeholders | Service Owner or Product |
| Scribe | Document timeline | Any available team member |

**All hands on deck:** All team members should be available for P0 incidents

---

## 7. Escalation Paths

### 7.1 Technical Escalation

```
Level 1: Primary On-Call (15 min)
         ↓ (no resolution after 1h)
Level 2: Secondary On-Call + Tech Lead (30 min)
         ↓ (no resolution after 2h)
Level 3: Service Owner + Engineering Manager
         ↓ (no resolution after 4h)
Level 4: VP Engineering + CTO
```

### 7.2 Business Escalation

```
For customer-facing issues:
Product Owner → VP Product → CEO

For security issues:
Security Team → CISO → CEO

For legal/compliance:
Legal Team → General Counsel
```

### 7.3 External Vendor Escalation

| Vendor | Service | Support Level | Contact | SLA |
|--------|---------|--------------|---------|-----|
| AWS | Cloud Infrastructure | Enterprise | aws-support@ | 15 min (Critical) |
| Datadog | Monitoring | Premium | datadog-support@ | 1 hour |
| Auth0 | Authentication | Enterprise | auth0-support@ | 30 min |

---

## 8. Communication Plan

### 8.1 Regular Communications

| Type | Frequency | Participants | Format | Owner |
|------|-----------|--------------|--------|-------|
| Daily Standup | Daily | Core team | Slack/Video | Tech Lead |
| Sprint Planning | Bi-weekly | Core + Product | Video | Product Owner |
| Architecture Review | Monthly | Core + Architect | Video | Tech Lead |
| Incident Review | Weekly | Core + SRE | Video | Service Owner |
| Stakeholder Update | Monthly | Leadership | Email/Doc | Service Owner |
| Quarterly Business Review | Quarterly | All stakeholders | Presentation | Service Owner |

### 8.2 Incident Communications

**Internal:**
- **#incidents** Slack channel: All severity incidents
- **#service-name-team** Slack: Team-specific discussion
- **PagerDuty:** Alert notifications
- **Email:** Post-mortem distribution

**External (Customer-Facing):**
- **Status Page:** https://status.company.com
- **Customer Email:** For P0/P1 with customer impact
- **Social Media:** For widespread outages (approved by PR)

**Communication Templates:**

```markdown
# P0 Incident Communication (Initial)
Subject: [P0] Service Outage - [Service Name]

We are currently experiencing an outage affecting [description].

Impact: [What customers are seeing]
Start Time: [Time in UTC]
Current Status: [Investigating/Identified/Fixing/Monitoring]
ETA: [Best estimate or "Unknown"]

Updates will be posted every 30 minutes.

# P0 Incident Communication (Resolution)
Subject: [RESOLVED] Service Outage - [Service Name]

The incident has been resolved.

Root Cause: [Brief description]
Resolution: [What was done]
Impacted Time: [Duration]

A full post-mortem will be shared within 48 hours.
```

### 8.3 Change Communication

**Major Changes (require notification):**
- Breaking API changes
- Significant architecture changes
- Multi-hour maintenance
- Feature deprecations

**Notification Timeline:**
- **7 days before:** Email to stakeholders
- **3 days before:** Reminder email
- **1 day before:** Final reminder + Slack announcement
- **During change:** Status updates
- **After change:** Confirmation email

---

## 9. Decision Making

### 9.1 Decision Authority Matrix

| Decision Type | Authority | Requires Approval From | Timeline |
|---------------|-----------|------------------------|----------|
| Routine code changes | Developer | Tech Lead (code review) | Immediate |
| New dependencies | Tech Lead | Service Owner | 1-2 days |
| Architecture changes | Tech Lead | Service Owner + Architect | 1 week |
| API breaking changes | Service Owner | Product + all consumers | 2 weeks |
| Budget > $10k | Service Owner | Engineering Manager | 1 week |
| Security exceptions | Security Team | CISO | Varies |
| Data privacy changes | Service Owner | Legal + DPO | 2 weeks |

### 9.2 Architecture Decision Records (ADRs)

Todas las decisiones arquitectónicas significativas deben documentarse:

**Template:**

```markdown
# ADR-XXX: [Título]

Date: YYYY-MM-DD
Status: [Proposed | Accepted | Deprecated | Superseded]
Deciders: [Lista de personas involucradas]

## Context
[Describe el problema y constraints]

## Decision
[Qué se decidió y por qué]

## Alternatives Considered
1. [Alternativa 1] - [Por qué no se eligió]
2. [Alternativa 2] - [Por qué no se eligió]

## Consequences
### Positive
- [Consecuencia positiva 1]

### Negative
- [Consecuencia negativa 1]

## Implementation
[Pasos para implementar]

## Review Date
[Cuándo revisar esta decisión]
```

**Ubicación:** `/docs/adr/` en el repositorio

---

## 10. Training & Onboarding

### 10.1 Onboarding Checklist

**New Team Member:**

**Week 1:**
- [ ] Access provisioning (GitHub, AWS, PagerDuty, etc.)
- [ ] Read all documentation (Architecture, Runbooks, etc.)
- [ ] Setup local development environment
- [ ] Shadow on-call for 1 week
- [ ] Attend team meetings

**Week 2-3:**
- [ ] First pull request (pair programming)
- [ ] Deploy to staging
- [ ] Fix a P3 bug
- [ ] Write/update documentation

**Week 4:**
- [ ] Join on-call rotation (secondary)
- [ ] Lead a small feature
- [ ] Participate in incident response

**Month 2:**
- [ ] Primary on-call
- [ ] Present in team meeting
- [ ] Complete service-specific training

### 10.2 Required Skills

| Skill | Required For | Training Resource |
|-------|--------------|-------------------|
| Kubernetes basics | All engineers | [Internal k8s course] |
| Terraform | Infrastructure changes | [Terraform docs] |
| Python/Node.js | Application development | [Team wiki] |
| Monitoring (Datadog) | On-call rotation | [Datadog certification] |
| Incident response | On-call rotation | [Incident response training] |
| Security best practices | All engineers | [Security training] |

### 10.3 Knowledge Transfer

**Documentation:**
- [ ] Architecture diagram up to date
- [ ] Runbooks for all common issues
- [ ] API documentation current
- [ ] Deployment guide tested recently

**Shadowing:**
- New members shadow experienced on-call for 1 week
- Reverse shadowing: Experienced member observes new member

**Brown Bag Sessions:**
- Monthly tech talks on service internals
- Share lessons learned from incidents
- Demo new features

---

## 11. Performance & Metrics

### 11.1 Team Metrics

| Metric | Target | Current | Trend |
|--------|--------|---------|-------|
| Mean Time to Recovery (MTTR) | < 1 hour | 45 min | ⬇️ |
| Deployment Frequency | > 5/week | 7/week | ➡️ |
| Change Failure Rate | < 5% | 3% | ⬇️ |
| On-call incidents per week | < 10 | 8 | ⬇️ |
| P0 incidents per month | 0 | 0 | ➡️ |
| P1 incidents per month | < 2 | 1 | ⬇️ |

### 11.2 Individual Responsibilities

**Performance Review Cycle:** Bi-annual

**Key Performance Indicators:**
- **Developers:**
  - Code quality (review feedback)
  - Feature delivery velocity
  - Bug fix efficiency
  - Documentation contributions

- **SREs:**
  - Incident response time
  - Automation contributions
  - System reliability improvements
  - Knowledge sharing

- **Service Owner:**
  - Service availability vs SLO
  - Stakeholder satisfaction
  - Team morale
  - Budget management

---

## 12. Offboarding

### 12.1 Team Member Leaving Checklist

**2 Weeks Before:**
- [ ] Identify knowledge gaps
- [ ] Schedule knowledge transfer sessions
- [ ] Document tribal knowledge
- [ ] Reassign ownership of features/systems

**1 Week Before:**
- [ ] Transfer responsibilities to team members
- [ ] Update RACI matrix
- [ ] Update on-call schedule
- [ ] Final knowledge transfer sessions

**Last Day:**
- [ ] Revoke all access (GitHub, AWS, PagerDuty, etc.)
- [ ] Return company assets (laptop, keys, etc.)
- [ ] Exit interview
- [ ] Update team org chart

---

## 13. Service Deprecation Plan

**Si el servicio necesita ser retirado:**

### Phase 1: Announcement (Month 1)
- [ ] Notify all stakeholders
- [ ] Document migration path
- [ ] Freeze new features

### Phase 2: Migration Support (Months 2-4)
- [ ] Provide migration tools
- [ ] Support customers migrating
- [ ] Reduce SLO commitments

### Phase 3: Read-Only Mode (Month 5)
- [ ] Disable writes
- [ ] Maintain reads only
- [ ] Archive data

### Phase 4: Sunset (Month 6)
- [ ] Take offline
- [ ] Archive documentation
- [ ] Final data export
- [ ] Decommission infrastructure

---

## 14. Review & Updates

**Review Frequency:** Quarterly

**Review Owners:** Service Owner + Tech Lead

**Review Checklist:**
- [ ] Team structure changes?
- [ ] RACI matrix still accurate?
- [ ] On-call rotation working well?
- [ ] Escalation paths correct?
- [ ] Contact information up to date?
- [ ] Metrics improving?
- [ ] Documentation complete?

**Next Review Date:** YYYY-MM-DD

---

## 15. Appendix

### 15.1 Contact Directory

**Team Members:**

| Name | Role | Email | Slack | Phone | Timezone |
|------|------|-------|-------|-------|----------|
| Alice Johnson | Senior SRE | alice@ | @alice | +1-555-0101 | UTC-5 |
| Bob Smith | SRE | bob@ | @bob | +44-7700-900123 | UTC+0 |
| Carlos García | Developer | carlos@ | @carlos | +34-600-123456 | UTC+1 |

**External Contacts:**

| Organization | Contact Person | Purpose | Email | Phone |
|--------------|----------------|---------|-------|-------|
| Security Team | security-oncall@ | Security incidents | | On PagerDuty |
| DBA Team | dba-oncall@ | Database issues | | On PagerDuty |
| Network Team | network@ | Network issues | | +1-555-0199 |

### 15.2 Important Links

- **Service Dashboard:** https://grafana.company.com/d/service-name
- **PagerDuty:** https://company.pagerduty.com/services/ABC123
- **Status Page:** https://status.company.com/services/service-name
- **Git Repository:** https://github.com/company/service-name
- **Documentation Hub:** https://wiki.company.com/services/service-name
- **CI/CD Pipeline:** https://jenkins.company.com/job/service-name
- **Monitoring Alerts:** https://datadog.company.com/monitors#service:service-name

---

**Document Owner:** [Service Owner]  
**Last Updated:** YYYY-MM-DD  
**Next Review:** YYYY-MM-DD
