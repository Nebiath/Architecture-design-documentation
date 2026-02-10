# [Nombre del Servicio] - Incident Management

> **Version:** 1.0  
> **Fecha:** YYYY-MM-DD  
> **Service Owner:** [Nombre]  
> **Última Revisión:** YYYY-MM-DD

---

## 1. Incident Response Overview

**Objective:** Restore service quickly, minimize impact, learn from failures

**Core Principles:**
- 🔥 **Stabilize first, root cause later**
- 📢 **Communicate early and often**
- 🤝 **Blameless post-mortems**
- 📚 **Document everything**

---

## 2. Incident Severity Matrix

| Severity | Description | Response Time | Examples | Escalation |
|----------|-------------|---------------|----------|------------|
| **P0 (Critical)** | Complete outage, data loss, security breach | **15 min** | • API completely down<br>• Database corrupted<br>• Security incident<br>• Multi-region failure | Immediate |
| **P1 (High)** | Major degradation, significant user impact | **30 min** | • High latency (> 2s)<br>• Error rate > 5%<br>• Payment processing down<br>• Single region failure | 1 hour |
| **P2 (Medium)** | Partial degradation, workaround exists | **2 hours** | • Non-critical feature down<br>• Slow performance<br>• Intermittent errors | 4 hours |
| **P3 (Low)** | Minor issue, minimal user impact | **Next day** | • Cosmetic issues<br>• Logging problems<br>• Monitoring gaps | N/A |

---

## 3. Incident Roles & Responsibilities

### 3.1 Incident Commander (IC)

**Who:** Primary on-call or first responder  
**Responsibilities:**
- ✅ Declare incident severity
- ✅ Coordinate response team
- ✅ Make critical decisions
- ✅ Communicate with stakeholders
- ✅ Determine when incident is resolved
- ✅ Ensure post-mortem is completed

**Authority:**
- Can pull in any team member
- Can approve emergency changes
- Can decide to rollback

### 3.2 Technical Lead (TL)

**Who:** Senior engineer or specialist for the affected area  
**Responsibilities:**
- ✅ Debug and identify root cause
- ✅ Implement fixes
- ✅ Execute mitigation steps
- ✅ Advise IC on technical decisions

### 3.3 Communications Lead (Comms)

**Who:** Service Owner, Product Manager, or designated person  
**Responsibilities:**
- ✅ Update status page
- ✅ Send customer notifications
- ✅ Update #incidents Slack channel
- ✅ Brief executives
- ✅ Prepare external communications

### 3.4 Scribe

**Who:** Any available team member  
**Responsibilities:**
- ✅ Document timeline
- ✅ Record actions taken
- ✅ Note key decisions
- ✅ Track people involved
- ✅ Capture metrics (start time, detection time, etc.)

---

## 4. Incident Response Workflow

### Phase 1: Detection & Triage (0-15 min)

```
1. Alert fires or issue reported
   ↓
2. On-call acknowledges in PagerDuty (< 15 min)
   ↓
3. Check dashboards & recent changes
   ↓
4. Determine severity using matrix
   ↓
5. If P0/P1: Post in #incidents Slack channel
   ↓
6. If P0: Page Incident Commander + escalate
```

**Slack Notification Template:**

```
🚨 INCIDENT DECLARED 🚨

Severity: P0 / P1 / P2
Service: [service-name]
Impact: [Brief description of user impact]
Started: [Time] UTC
IC: @username
Status: Investigating

Dashboard: [link]
PagerDuty: [link]
War Room: #incident-YYYY-MM-DD-service-name
```

### Phase 2: Response & Mitigation (15 min - 4 hours)

```
1. Assemble incident response team
   - IC: Coordinates
   - TL: Debugs
   - Comms: Updates stakeholders
   - Scribe: Documents
   ↓
2. Create incident war room (Slack channel)
   ↓
3. IC establishes update cadence (every 30 min for P0/P1)
   ↓
4. TL investigates and proposes mitigation
   ↓
5. IC approves mitigation plan
   ↓
6. Execute mitigation (rollback, scale, failover, etc.)
   ↓
7. Monitor metrics for improvement
   ↓
8. If not resolved: Escalate or try alternative mitigation
```

**Mitigation Decision Tree:**

```
Was there a recent deployment?
├─ YES → Rollback
│   ↓
│   Metrics improved?
│   ├─ YES → Monitor & document
│   └─ NO → Continue investigation
│
└─ NO → Check infrastructure
    ↓
    Resource saturation?
    ├─ YES → Scale up
    └─ NO → Check dependencies
        ↓
        External service down?
        ├─ YES → Enable circuit breaker / failover
        └─ NO → Deep dive debugging
```

### Phase 3: Recovery & Verification (30 min - 1 hour)

```
1. Metrics return to normal
   ↓
2. Run smoke tests
   ↓
3. Verify critical user journeys
   ↓
4. Monitor for 30+ minutes
   ↓
5. IC declares incident resolved
   ↓
6. Send resolution notification
```

**Resolution Notification Template:**

```
✅ INCIDENT RESOLVED ✅

Severity: P0
Service: [service-name]
Duration: [X hours Y minutes]
Root Cause: [Brief description]
Resolution: [What was done]

Full post-mortem will be published within 48 hours.
Thanks to the response team: @user1 @user2 @user3
```

### Phase 4: Post-Incident (24-48 hours)

```
1. Schedule post-mortem meeting (within 24h)
   ↓
2. IC writes post-mortem document
   ↓
3. Team reviews and adds input
   ↓
4. Identify action items with owners
   ↓
5. Publish post-mortem (within 48h)
   ↓
6. Track action items to completion
```

---

## 5. Communication Matrix

### Internal Communications

| Audience | Channel | P0 | P1 | P2 | P3 |
|----------|---------|----|----|----|----|
| On-call team | PagerDuty | Immediately | Immediately | Immediately | - |
| Engineering | #incidents Slack | Immediately | Immediately | When detected | - |
| Service Owner | Direct message | Immediately | Within 15 min | Within 1 hour | - |
| Leadership | Email + Slack | Within 15 min | Within 30 min | EOD summary | - |
| Company-wide | #general Slack | If customer-facing | If high impact | - | - |

### External Communications

| Audience | Channel | P0 | P1 | P2 | P3 |
|----------|---------|----|----|----|----|
| All Customers | Status Page | Immediately | Within 30 min | - | - |
| Affected Customers | Email | Within 1 hour | Within 2 hours | - | - |
| Enterprise Accounts | Direct call | Within 30 min | Within 1 hour | - | - |
| Social Media | Twitter | If widespread | If requested | - | - |
| Press | PR team | Only if major | - | - | - |

**Update Frequency:**
- **P0:** Every 30 minutes minimum
- **P1:** Every 1 hour minimum
- **P2:** When significant change occurs

---

## 6. Runbooks for Common Incidents

### 6.1 Service Unavailable (HTTP 503)

**Symptoms:**
- Health checks failing
- 503 errors in logs
- No responses from service

**Quick Diagnosis:**
```bash
# Check pod status
kubectl get pods -n production -l app=service-name

# Check logs
kubectl logs -f deployment/service-name -n production --tail=100

# Check resource usage
kubectl top pods -n production -l app=service-name
```

**Common Causes & Fixes:**

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| OOMKilled | `kubectl describe pod` shows OOMKilled | Increase memory limits or fix memory leak |
| CrashLoopBackOff | Pod restart count increasing | Check logs, likely config or dependency issue |
| ImagePullBackOff | Can't pull Docker image | Check image tag and registry access |
| No healthy pods | All pods failing health checks | Check database connectivity, external deps |

**Escalation:** If not resolved in 30 minutes, escalate to L2

---

### 6.2 High Latency

**Symptoms:**
- p95 latency > 300ms
- User complaints of slowness
- Timeout errors

**Quick Diagnosis:**
```bash
# Check APM traces
# https://app.datadoghq.com/apm/services/service-name

# Check database performance
kubectl exec -it deployment/service-name -- \
  psql -U user -d database -c \
  "SELECT pid, query, state, wait_event FROM pg_stat_activity WHERE state = 'active';"

# Check cache hit rate
redis-cli INFO | grep hit_rate
```

**Common Causes & Fixes:**

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Database slow queries | Long-running queries in pg_stat_activity | Kill queries, add indexes, optimize |
| Cache miss spike | Low cache hit rate | Warm up cache, increase TTL |
| External API slow | High latency to upstream service | Enable circuit breaker, increase timeout |
| Resource saturation | High CPU/memory usage | Scale up pods |
| Too many connections | Connection pool exhausted | Increase pool size, fix connection leaks |

---

### 6.3 High Error Rate

**Symptoms:**
- Error rate > 1%
- 5xx errors increasing
- Failed transactions

**Quick Diagnosis:**
```bash
# Check error distribution
index=production service=service-name level=ERROR
| stats count by error.type

# Check recent deployments
kubectl rollout history deployment/service-name -n production

# Check upstream dependencies
curl -w "@curl-format.txt" -o /dev/null -s https://upstream-api.com/health
```

**Common Causes & Fixes:**

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Bad deployment | Recent deployment in rollout history | Rollback immediately |
| Upstream API down | Upstream health check failing | Enable circuit breaker, use fallback |
| Database connection errors | "connection refused" in logs | Check database health, restart connections |
| Configuration error | "config not found" errors | Verify ConfigMap/Secret exists |

---

### 6.4 Database Issues

**Symptoms:**
- Connection errors
- Query timeouts
- Replication lag

**Quick Diagnosis:**
```bash
# Check RDS metrics in AWS Console
# - CPU utilization
# - IOPS
# - Connection count
# - Replication lag

# Check active connections
SELECT count(*) FROM pg_stat_activity;

# Check slow queries
SELECT pid, now() - pg_stat_activity.query_start AS duration, query 
FROM pg_stat_activity 
WHERE state = 'active' 
ORDER BY duration DESC;
```

**Common Causes & Fixes:**

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Connection pool exhausted | Connections = max_connections | Kill idle connections, increase pool |
| Slow query | Query running > 30s | Kill query, add index |
| High CPU | RDS CPU > 90% | Optimize queries, add read replicas |
| Replication lag | Lag > 60s | Reduce write load, check replica health |
| Storage full | Disk utilization > 85% | Expand storage, archive old data |

---

### 6.5 Security Incidents

**⚠️ CRITICAL - Different Process**

**Symptoms:**
- Suspicious access patterns
- Data breach alerts
- Unauthorized access
- DDoS attack

**Immediate Actions:**
```
1. DO NOT attempt to fix yourself
2. Page Security Team IMMEDIATELY
3. Preserve all logs (do not delete anything)
4. Isolate affected systems if possible
5. Follow Security Incident Response Plan
6. Notify Legal and Compliance teams
```

**Security Incident Hotline:** security-emergency@company.com

---

## 7. Escalation Procedures

### Technical Escalation

```
Level 0: Primary On-Call
    ↓ (15 min, no progress)
Level 1: Secondary On-Call + Tech Lead
    ↓ (30 min, no progress)
Level 2: Service Owner + Specialist (DB/Network/Security)
    ↓ (1 hour, no progress)
Level 3: Engineering Manager + VP Engineering
    ↓ (2 hours, no progress)
Level 4: CTO + External Vendor Support
```

### Business Escalation

**Customer Impact:**
```
Product Owner → VP Product → CEO
```

**Revenue Impact > $100k:**
```
Service Owner → CFO → CEO
```

**Legal/Compliance:**
```
Compliance Officer → General Counsel
```

### External Vendor Escalation

| Vendor | Service | Escalation Procedure | SLA |
|--------|---------|---------------------|-----|
| AWS | Infrastructure | Open P1 support ticket + call TAM | 15 min |
| Datadog | Monitoring | Email support + Slack channel | 1 hour |
| Stripe | Payments | Email + dedicated Slack | 30 min |

---

## 8. Post-Mortem Template

**Post-Mortem Document:** Use this template after EVERY P0/P1 incident

```markdown
# Post-Mortem: [Incident Title]

## Metadata
- **Date:** YYYY-MM-DD
- **Duration:** [X hours Y minutes]
- **Severity:** P0 / P1
- **Services Affected:** [service-name]
- **Incident Commander:** @username
- **Post-Mortem Author:** @username

---

## Executive Summary
[2-3 sentence summary of what happened and the impact]

Example: "On March 15 at 10:00 UTC, our payment service experienced a complete outage lasting 2 hours, affecting all users globally. Approximately 50,000 transactions failed. The root cause was a database migration that locked a critical table."

---

## Impact
- **Users Affected:** [Number or percentage]
- **Duration:** [Time from detection to resolution]
- **Revenue Impact:** $[Amount] (if applicable)
- **SLO Impact:** [X minutes of error budget consumed]
- **Reputation Impact:** [Customer complaints, social media mentions]

---

## Timeline (All times in UTC)

| Time | Event |
|------|-------|
| 10:00 | Database migration deployed |
| 10:05 | First alerts fire for high latency |
| 10:07 | On-call acknowledges |
| 10:10 | Severity upgraded to P0 |
| 10:15 | Incident Commander paged |
| 10:20 | Root cause identified (table lock) |
| 10:25 | Decision to rollback migration |
| 10:30 | Rollback initiated |
| 10:45 | Service recovering |
| 11:00 | Full recovery confirmed |
| 11:30 | Incident closed |

---

## Root Cause Analysis

### What Happened
[Detailed technical explanation of the failure]

Example: "A database migration added a new index on the 'transactions' table (50M rows) without using CONCURRENTLY option in PostgreSQL. This acquired an ACCESS EXCLUSIVE lock, blocking all reads and writes to the table for 30+ minutes."

### Why It Happened
[Contributing factors - technical, process, human]

Example:
- Migration script was not tested on production-like data volume
- CONCURRENTLY flag was missing from index creation
- No review process for high-risk DB changes
- Insufficient monitoring for table locks

### Why It Wasn't Caught Earlier
[Why pre-production testing didn't catch it]

Example:
- Staging database only has 1M rows (vs 50M in prod)
- Migration completed in < 1 second in staging
- No load testing during migrations

---

## What Went Well
- ✅ Alert fired within 5 minutes of issue
- ✅ On-call responded quickly
- ✅ Root cause identified in 15 minutes
- ✅ Rollback executed cleanly
- ✅ Communication was clear and timely

---

## What Went Poorly
- ❌ Migration was not tested on production-scale data
- ❌ No automated rollback for failed migrations
- ❌ Monitoring didn't catch table locks
- ❌ Runbook for database issues was outdated
- ❌ Took 15 minutes to find right DBA contact

---

## Action Items

| Action | Owner | Priority | Due Date | Status |
|--------|-------|----------|----------|--------|
| Add CONCURRENTLY to all index migrations | @dev-team | P0 | 2024-03-20 | Done ✅ |
| Create production-scale test environment | @sre | P1 | 2024-04-01 | In Progress 🔄 |
| Add monitoring for database locks | @sre | P0 | 2024-03-18 | Done ✅ |
| Implement automated migration rollback | @dev-team | P1 | 2024-04-15 | Planned 📅 |
| Update database runbook | @oncall | P2 | 2024-03-25 | Done ✅ |
| Add DBA contacts to escalation doc | @manager | P2 | 2024-03-17 | Done ✅ |

---

## Lessons Learned

### Technical
- Always use CONCURRENTLY for index creation on large tables
- Test migrations with production-like data volumes
- Monitor for database locks in addition to query performance

### Process
- High-risk changes (DB schema) need mandatory review
- Need better handoff between development and operations
- War room communication was effective - continue this practice

### Cultural
- Blameless culture worked - team felt safe escalating quickly
- Cross-functional collaboration was excellent
- Good practice: IC made decisive rollback call vs debugging longer

---

## Related Incidents
- 2024-02-01: Similar issue with user table migration
- 2023-11-15: Database connection pool exhaustion

---

## Appendix

### Metrics
- MTTD (Mean Time To Detect): 5 minutes
- MTTR (Mean Time To Recovery): 2 hours
- MTTF (Mean Time To Fix): 1 hour 45 minutes

### Supporting Data
- [Link to dashboard during incident]
- [Link to PagerDuty incident]
- [Link to Slack thread]
- [Link to deployment logs]

---

**Post-Mortem Meeting:** 2024-03-17 10:00 UTC  
**Attendees:** [List]  
**Published:** 2024-03-17  
**Review Date:** 2024-04-17 (1 month follow-up)
```

---

## 9. Incident Metrics & Goals

### Key Metrics to Track

| Metric | Definition | Current | Goal |
|--------|------------|---------|------|
| **MTTD** | Mean Time To Detect | 8 min | < 5 min |
| **MTTA** | Mean Time To Acknowledge | 12 min | < 15 min |
| **MTTR** | Mean Time To Recovery | 45 min | < 1 hour |
| **Incident Frequency** | Incidents per month | 8 | < 5 |
| **P0 Incidents** | Critical incidents per quarter | 2 | 0 |
| **Repeat Incidents** | Same root cause within 90 days | 15% | < 5% |

### Monthly Incident Review

**Review Meeting:** First Monday of each month  
**Attendees:** Service Owner, Tech Lead, SRE Team, Product

**Agenda:**
1. Review incident metrics
2. Identify trends
3. Review action item completion
4. Discuss systemic improvements
5. Update runbooks if needed

---

## 10. Incident Response Training

### New Team Members

**Week 1:** Shadow experienced on-call
- Observe incident response
- Read past post-mortems
- Review runbooks

**Week 2:** Practice scenarios
- Simulated incidents
- Practice using tools
- Write practice post-mortem

**Week 3:** Backup on-call
- Respond with mentor
- Lead investigation with support
- Present findings

**Week 4:** Primary on-call rotation

### Chaos Engineering / Fire Drills

**Frequency:** Quarterly  
**Duration:** 1-2 hours  
**Goal:** Test incident response without real impact

**Example Scenarios:**
- Simulate database failure
- Kill random pods
- Inject latency in external API
- Create fake security alert

**Post-Drill:**
- Review response effectiveness
- Update runbooks
- Improve monitoring

---

## 11. Tools & Access

### Required Tools

| Tool | Purpose | Access Request |
|------|---------|----------------|
| PagerDuty | Alerting | IT helpdesk ticket |
| Datadog | Monitoring | Manager approval |
| Grafana | Dashboards | SSO (auto-provisioned) |
| Splunk | Log analysis | Manager approval |
| kubectl | K8s access | Cloud team + training |
| AWS Console | Infrastructure | IAM policy request |

### Emergency Access

**Break-glass Account:** For when normal access fails
- Location: 1Password vault "Emergency Access"
- Rotation: Every 90 days
- Audit: All usage logged and reviewed

---

## 12. Continuous Improvement

### After Each Incident

- [ ] Post-mortem completed within 48h
- [ ] Action items assigned with owners
- [ ] Runbooks updated
- [ ] Monitoring improved
- [ ] Knowledge shared with team

### Quarterly Review

- [ ] Review all incidents from quarter
- [ ] Identify systemic issues
- [ ] Update incident response process
- [ ] Improve tooling
- [ ] Conduct fire drill

---

## 13. Status Page Management

**URL:** https://status.company.com

### When to Update Status Page

| Severity | Update Required | Update Frequency |
|----------|----------------|------------------|
| P0 | Always | Every 30 min |
| P1 | If customer-facing | Every 1 hour |
| P2 | Only if widespread reports | When resolved |
| P3 | No | N/A |

### Status Page Workflow

```
1. IC approves status page update
   ↓
2. Comms Lead writes update
   ↓
3. Post to status page
   ↓
4. Link shared in #incidents
   ↓
5. Update every [X] minutes until resolved
```

---

## 14. Documentation Updates

**This document should be updated:**
- After every P0/P1 incident (within 1 week)
- When new runbooks are created
- When escalation contacts change
- Quarterly review minimum

**Next Review Date:** YYYY-MM-DD

---

## 15. Quick Reference Card

**Print this and keep at desk:**

```
┌────────────────────────────────────────────┐
│      INCIDENT RESPONSE QUICK GUIDE         │
├────────────────────────────────────────────┤
│ 1. ACKNOWLEDGE (< 15 min)                  │
│    - PagerDuty: Acknowledge alert          │
│    - Slack: Post in #incidents             │
│                                            │
│ 2. ASSESS (< 5 min)                        │
│    - Check dashboard                       │
│    - Determine severity (P0-P3)            │
│    - Recent changes?                       │
│                                            │
│ 3. ESCALATE (if needed)                    │
│    - P0: Immediate                         │
│    - P1: After 1 hour no progress          │
│    - P2: After 4 hours no progress         │
│                                            │
│ 4. MITIGATE                                │
│    - Recent deploy? → Rollback             │
│    - Resource issue? → Scale               │
│    - Dependency? → Circuit breaker         │
│                                            │
│ 5. COMMUNICATE (every 30 min)              │
│    - Update #incidents                     │
│    - Update status page (if customer)      │
│                                            │
│ 6. RESOLVE & DOCUMENT                      │
│    - Monitor 30 min after fix              │
│    - Post-mortem within 48h                │
│                                            │
│ CONTACTS:                                  │
│ - On-call: PagerDuty                       │
│ - Security: security-emergency@            │
│ - Dashboard: grafana.company.com           │
│ - Runbooks: wiki.company.com/runbooks      │
└────────────────────────────────────────────┘
```

---

**Document Owner:** [Service Owner]  
**Last Updated:** YYYY-MM-DD  
**Next Review:** YYYY-MM-DD
