# [Nombre del Servicio] - Change Management

> **Version:** 1.0  
> **Fecha:** YYYY-MM-DD  
> **Service Owner:** [Nombre]  
> **Última Revisión:** YYYY-MM-DD

---

## 1. Change Management Overview

**Purpose:** Control and coordinate changes to minimize risk and disruption

**Key Principles:**
- ✅ All production changes must be tracked
- ✅ High-risk changes require approval
- ✅ Changes should be reversible
- ✅ Communication is mandatory
- ✅ Learn from failures

---

## 2. Change Categories

### 2.1 Standard Changes (Pre-Approved)

**Definition:** Low-risk, routine changes with documented procedures

**Examples:**
- Code deployments via CI/CD (passing all tests)
- Scaling operations (within limits)
- Certificate renewals (automated)
- Log level changes
- Feature flag toggles

**Requirements:**
- ✅ Follow documented procedure
- ✅ Automated rollback available
- ✅ Post in #deployments Slack channel
- ❌ No approval required
- ❌ No CAB review

**Process:**
```
Developer commits → CI/CD pipeline → Automated tests
    ↓ (all pass)
Deploy to staging → Automated tests
    ↓ (all pass)
Auto-deploy to production → Monitor
    ↓ (metrics healthy)
✅ Change complete
```

---

### 2.2 Normal Changes

**Definition:** Medium-risk changes requiring review but not CAB

**Examples:**
- Infrastructure scaling (beyond auto-scale limits)
- Database parameter tuning
- New monitoring/alerts
- Dependency upgrades
- Non-breaking API changes

**Requirements:**
- ✅ Create change request in Jira
- ✅ Tech Lead approval
- ✅ Documented rollback plan
- ✅ Scheduled during business hours
- ✅ Post-change verification
- ❌ No CAB required

**Lead Time:** 2-3 business days

---

### 2.3 Significant Changes

**Definition:** High-risk changes with potential for widespread impact

**Examples:**
- Database schema changes
- Network topology changes
- Security policy updates
- Major version upgrades
- Breaking API changes
- Multi-service deployments

**Requirements:**
- ✅ Create RFC (Request for Change)
- ✅ Service Owner approval
- ✅ Impact analysis
- ✅ Detailed rollback plan
- ✅ Testing in staging
- ✅ Scheduled maintenance window
- ✅ Communication to stakeholders
- ❌ CAB optional (depends on scope)

**Lead Time:** 1-2 weeks

---

### 2.4 Major Changes (CAB Required)

**Definition:** Critical changes affecting multiple services or customers

**Examples:**
- Data center migration
- Complete architecture overhaul
- Security incident remediation
- Compliance-driven changes
- Deprecation of major features

**Requirements:**
- ✅ Full RFC document
- ✅ CAB approval (Change Advisory Board)
- ✅ Executive approval
- ✅ Detailed implementation plan
- ✅ Comprehensive testing
- ✅ Customer notification (if applicable)
- ✅ Rollback AND forward recovery plans

**Lead Time:** 4+ weeks

---

### 2.5 Emergency Changes

**Definition:** Urgent changes to resolve P0/P1 incidents

**Examples:**
- Hotfix for critical bug
- Security patch for active exploit
- Incident mitigation
- Rollback of failed change

**Requirements:**
- ✅ Incident Commander authorization
- ✅ Expedited approval (verbal OK)
- ✅ Document in incident report
- ✅ Retrospective review within 24h
- ❌ Can bypass normal windows

**Approval:** Service Owner or Incident Commander

**Post-Change:** Full documentation and review

---

## 3. Change Decision Matrix

| Change Type | Example | Approval | Lead Time | Window Required | Rollback Plan | CAB |
|-------------|---------|----------|-----------|----------------|---------------|-----|
| **Standard** | Code deploy via CI/CD | None | Immediate | No | Automatic | No |
| **Normal** | Increase RDS size | Tech Lead | 2-3 days | Preferred | Manual | No |
| **Significant** | Database migration | Service Owner | 1-2 weeks | Required | Detailed | Optional |
| **Major** | Multi-region setup | CAB + Exec | 4+ weeks | Scheduled | Comprehensive | Yes |
| **Emergency** | Hotfix P0 bug | IC / Owner | Immediate | No | Required | No |

---

## 4. Change Windows

### 4.1 Preferred Windows

**Purpose:** Minimize customer impact

| Day | Window (UTC) | Type | Notes |
|-----|--------------|------|-------|
| **Mon-Thu** | 10:00-16:00 | Standard | Normal business hours, teams available |
| **Friday** | ❌ Blackout | None | No changes before weekend |
| **Saturday** | 02:00-08:00 | Maintenance | Lowest traffic, for significant changes |
| **Sunday** | 02:00-08:00 | Maintenance | Lowest traffic, preferred for major changes |

### 4.2 Blackout Periods

**No non-emergency changes allowed:**

- **End of Quarter:** Last 5 business days
- **Black Friday / Cyber Monday:** Week before through week after
- **Holiday Season:** December 15 - January 2
- **Product Launches:** As announced by Product team
- **Company Events:** As announced (e.g., annual conference)

**Exception:** P0/P1 emergency fixes only

### 4.3 Timezone Considerations

**Global Operations:** We operate in multiple regions

| Region | Local Time | UTC Window | Considerations |
|--------|------------|------------|----------------|
| EU | 11:00-17:00 CET | 10:00-16:00 UTC | Business hours |
| US East | 05:00-11:00 EST | 10:00-16:00 UTC | Early morning |
| US West | 02:00-08:00 PST | 10:00-16:00 UTC | Night time |
| APAC | 18:00-24:00 SGT | 10:00-16:00 UTC | Evening |

**Best Window for Global Impact:** Sunday 02:00-08:00 UTC

---

## 5. Request for Change (RFC) Template

### RFC-[NUMBER]: [Title]

**Status:** Draft | Submitted | Approved | Scheduled | Implemented | Rejected

---

#### 1. Change Summary

**Change ID:** RFC-2024-0315-001  
**Submitted By:** [Name]  
**Submitted Date:** YYYY-MM-DD  
**Service(s):** [service-name]  
**Change Type:** Normal | Significant | Major  
**Priority:** Low | Medium | High | Critical

**Brief Description:** [1-2 sentences]

---

#### 2. Business Justification

**Why is this change needed?**
[Detailed explanation]

**What happens if we don't do this?**
[Risk of not changing]

**Expected Benefits:**
- [Benefit 1]
- [Benefit 2]

**Cost of Change:** $[amount] (infrastructure, labor, downtime)

---

#### 3. Impact Analysis

**Customer Impact:**
- Users affected: [Number or "All"]
- Impact level: None | Low | Medium | High
- Impact duration: [X minutes/hours]
- Mitigation: [How to minimize impact]

**Service Impact:**
| Service | Impact Level | Description |
|---------|--------------|-------------|
| service-name | High | 15 min downtime |
| dependent-service | Medium | Degraded performance |
| other-service | None | No impact |

**Compliance/Regulatory Impact:**
- GDPR: [Yes/No - explain if yes]
- SOC2: [Yes/No - explain if yes]
- Other: [List any]

---

#### 4. Technical Details

**What will change?**
[Detailed technical description]

**Architecture Diagram:**
```
[Before]          [After]
  ...     →         ...
```

**Components Affected:**
- [ ] Application code
- [ ] Database schema
- [ ] Infrastructure (specify: networking/compute/storage)
- [ ] Configuration
- [ ] Security policies
- [ ] External integrations

---

#### 5. Implementation Plan

**Pre-Change Steps:**
1. [Step 1]
2. [Step 2]

**Implementation Steps:**
1. [Step 1 - Duration: X min]
2. [Step 2 - Duration: X min]
3. [Step 3 - Duration: X min]

**Total Duration:** [X hours Y minutes]

**Post-Change Verification:**
1. [ ] Health checks passing
2. [ ] Metrics within normal range
3. [ ] Smoke tests successful
4. [ ] Customer validation (if applicable)

---

#### 6. Rollback Plan

**Rollback Trigger Conditions:**
- [Condition 1]
- [Condition 2]

**Rollback Steps:**
1. [Step 1 - Duration: X min]
2. [Step 2 - Duration: X min]

**Rollback Duration:** [X minutes]

**Rollback Tested?** [Yes/No - when and where?]

**Point of No Return:** [At what point can we no longer rollback?]

---

#### 7. Testing

**Testing Completed:**
- [ ] Unit tests
- [ ] Integration tests
- [ ] Staging deployment
- [ ] Load testing
- [ ] Security testing
- [ ] Rollback tested

**Test Results:** [Summary or link to test report]

**Known Issues:** [Any issues discovered during testing]

---

#### 8. Dependencies

**Requires:**
- [ ] Other change RFC-XXXX to complete first
- [ ] Database backup (taken on [date])
- [ ] Approval from [team/person]
- [ ] Vendor coordination

**Blocks:**
- [ ] RFC-YYYY (can't proceed until this completes)

---

#### 9. Communication Plan

**Stakeholders to Notify:**
| Stakeholder | When | Method | Message |
|-------------|------|--------|---------|
| Engineering team | 7 days before | Email + Slack | Technical details |
| Customers | 3 days before | Email | Service window notification |
| Support team | 1 day before | Training | Expected issues |
| Executives | 1 day before | Email | High-level summary |

**Notification Templates:** [Link to templates]

---

#### 10. Schedule

**Proposed Date:** YYYY-MM-DD  
**Proposed Time:** HH:MM UTC  
**Estimated Duration:** [X hours]  
**Change Window:** Standard | Maintenance | Emergency

**Alternative Dates:**
1. YYYY-MM-DD HH:MM UTC
2. YYYY-MM-DD HH:MM UTC

---

#### 11. Team & Roles

| Role | Name | Responsibilities |
|------|------|------------------|
| Change Owner | [Name] | Overall accountability |
| Technical Lead | [Name] | Execute change |
| Communications | [Name] | Stakeholder updates |
| Approver | [Name] | Sign-off authority |
| Observer | [Name] | Monitor and document |

---

#### 12. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Database migration fails | Medium | High | Test on copy, have rollback |
| Extended downtime | Low | High | Practice in staging |
| Data corruption | Low | Critical | Full backup before change |

**Overall Risk Level:** Low | Medium | High | Critical

---

#### 13. Success Criteria

**How do we know the change was successful?**
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

**Metrics to Monitor:**
- [Metric 1: target value]
- [Metric 2: target value]

---

#### 14. Post-Implementation Review

**To be completed within 24 hours:**
- [ ] Change completed as planned
- [ ] Success criteria met
- [ ] No rollback required
- [ ] Documentation updated
- [ ] Stakeholders notified of completion
- [ ] Lessons learned captured

---

#### 15. Approvals

| Role | Name | Approval | Date |
|------|------|----------|------|
| Tech Lead | | ✅ Approved | |
| Service Owner | | ✅ Approved | |
| Security (if needed) | | ✅ Approved | |
| CAB (if needed) | | ✅ Approved | |

---

#### 16. Attachments

- [Link to technical design doc]
- [Link to test results]
- [Link to architecture diagrams]
- [Link to runbooks]

---

## 6. Change Advisory Board (CAB)

### 6.1 CAB Composition

**Core Members:**
- VP Engineering (Chair)
- Service Owners
- SRE Lead
- Security Lead
- Product Lead

**Optional Members (as needed):**
- DBA
- Network Engineer
- Compliance Officer
- Customer Success Lead

### 6.2 CAB Meeting Schedule

**Frequency:** Weekly (Wednesdays 14:00 UTC)

**Duration:** 1 hour

**Agenda:**
1. Review RFCs submitted this week (10 min each)
2. Post-implementation reviews (5 min each)
3. Discuss failed changes (10 min)
4. Upcoming major changes preview (10 min)

### 6.3 CAB Decision Criteria

**Approved if:**
- ✅ Risk is acceptable
- ✅ Rollback plan is solid
- ✅ Testing is complete
- ✅ Communication plan exists
- ✅ Resources are available
- ✅ No conflicting changes

**Rejected if:**
- ❌ Risk too high
- ❌ Insufficient testing
- ❌ No rollback plan
- ❌ During blackout period
- ❌ Conflicts with other changes

**Deferred if:**
- ⏸️ More information needed
- ⏸️ Dependencies not ready
- ⏸️ Better timing available

---

## 7. Change Tracking & Metrics

### 7.1 Change Log

**All changes must be logged in:**
- Jira (for RFCs)
- ServiceNow (for CAB changes)
- Confluence (for documentation)
- Slack #deployments channel (for awareness)

**Log Format:**
```
[CHANGE] [YYYY-MM-DD HH:MM] [service-name] [change-type]
Description: [Brief description]
Owner: @username
RFC: JIRA-1234
Status: Scheduled | In Progress | Complete | Failed | Rolled Back
```

### 7.2 Key Metrics

**Track Monthly:**

| Metric | Current | Goal | Status |
|--------|---------|------|--------|
| Total Changes | 120 | N/A | 📊 |
| Standard Changes | 100 (83%) | > 80% | ✅ |
| Emergency Changes | 5 (4%) | < 5% | ✅ |
| Failed Changes | 3 (2.5%) | < 5% | ✅ |
| Changes Requiring Rollback | 2 (1.7%) | < 2% | ✅ |
| Average Lead Time (Normal) | 3 days | < 3 days | ✅ |
| Changes in Blackout Period | 1 | 0 | ⚠️ |

### 7.3 Change Success Rate

**Formula:**
```
Success Rate = (Completed Changes - Rolled Back Changes) / Total Changes
```

**Current:** 98.3%  
**Goal:** > 95%

---

## 8. Failed Change Procedure

**When a change fails:**

### Immediate Actions (0-15 min)
1. Stop the change immediately
2. Assess impact (customers affected?)
3. Determine: Rollback or Forward Fix?
4. If P0/P1 impact → Declare incident
5. Execute rollback or fix

### Post-Failure (24 hours)
1. Document what went wrong
2. Identify root cause
3. Update RFC with lessons learned
4. If pattern emerges → Update change process
5. Re-submit RFC if change still needed

### Example Failed Change Report

```markdown
## Failed Change Report

**Change ID:** RFC-2024-0315-001  
**Date:** 2024-03-15 10:00 UTC  
**Service:** service-name  
**Change Type:** Database migration

**What Was Attempted:**
Adding index to transactions table

**What Went Wrong:**
Migration locked table for 30 minutes, causing service outage

**Root Cause:**
Missing CONCURRENTLY flag in PostgreSQL index creation

**Impact:**
- P0 incident
- 2 hour downtime
- 50,000 failed transactions

**Actions Taken:**
- Rolled back migration immediately
- Service recovered in 15 minutes

**Lessons Learned:**
- Test migrations on production-scale data
- Always use CONCURRENTLY for large tables
- Add table lock monitoring

**Updated RFC:** [link]  
**Incident Report:** [link]  
**Action Items:** [list]
```

---

## 9. Communication Templates

### 9.1 Planned Maintenance Notification

**To:** All Users  
**Subject:** Scheduled Maintenance - [Service Name] - [Date]

```
Hello,

We will be performing scheduled maintenance on [Service Name] on:

Date: [Day, Month Date, Year]
Time: [Start Time] - [End Time] [Timezone]
Duration: Approximately [X] hours

During this time, you may experience:
- [Expected impact]
- [Expected impact]

We apologize for any inconvenience. If you have questions, please contact support@company.com.

Thank you,
[Team Name]
```

### 9.2 Change Completion Notification

**To:** Stakeholders  
**Subject:** [COMPLETE] [Service Name] Maintenance - [Date]

```
The scheduled maintenance for [Service Name] has been completed successfully.

Start Time: [Time]
End Time: [Time]
Duration: [Actual duration]

Status: ✅ Completed successfully
Impact: No issues reported
Next Steps: Normal operations resumed

Thank you for your patience.
```

### 9.3 Change Failure Notification

**To:** Stakeholders  
**Subject:** [ROLLED BACK] [Service Name] Change - [Date]

```
⚠️ ATTENTION REQUIRED ⚠️

The planned change to [Service Name] has been rolled back due to [reason].

Attempted Change: [Description]
Impact: [What users experienced]
Resolution: Change rolled back, service restored
Status: Under investigation

A full post-mortem will be shared within 48 hours.

Current Service Status: Normal operations
```

---

## 10. Change Automation

### 10.1 CI/CD Integration

**Automated Change Logging:**

```yaml
# .github/workflows/deploy.yml
steps:
  - name: Log Change
    run: |
      curl -X POST https://change-api.company.com/changes \
        -d '{
          "service": "service-name",
          "type": "standard",
          "description": "Deploy ${{ github.sha }}",
          "owner": "${{ github.actor }}",
          "pr": "${{ github.event.pull_request.number }}"
        }'
```

### 10.2 Approval Gates

**For Significant/Major changes:**

```yaml
production:
  environment:
    name: production
    approval-gate: true
    required-reviewers:
      - tech-lead
      - service-owner
```

### 10.3 Automated Rollback

**Trigger automatic rollback if:**
```yaml
monitors:
  - name: error-rate
    threshold: 5%
    duration: 5m
    action: auto-rollback
    
  - name: latency-p95
    threshold: 1000ms
    duration: 10m
    action: auto-rollback
```

---

## 11. Continuous Improvement

### Monthly Change Review

**Agenda:**
1. Review change metrics
2. Analyze failed changes
3. Identify process improvements
4. Update templates/runbooks
5. Share learnings

**Attendees:**
- Service Owners
- Tech Leads
- SRE Team
- CAB Chair

### Quarterly Process Audit

**Review:**
- [ ] Are changes being tracked properly?
- [ ] Is approval process followed?
- [ ] Are blackout periods respected?
- [ ] Is communication effective?
- [ ] Are success rates acceptable?

**Update:** Change management process as needed

---

## 12. Compliance & Audit

### 12.1 Change Records Retention

**Requirement:** All change records must be retained for audit

| Record Type | Retention Period |
|-------------|------------------|
| RFCs | 3 years |
| Change logs | 1 year |
| CAB meeting notes | 3 years |
| Failed change reports | Indefinitely |

**Storage:** ServiceNow + Confluence

### 12.2 Audit Trail

**Every change must have:**
- ✅ Who made the change
- ✅ When it was made
- ✅ What was changed
- ✅ Why it was changed
- ✅ Who approved it
- ✅ What was the result

**Audit Access:** Compliance team, upon request

---

## 13. Quick Reference

### Change Type Decision Tree

```
Is this an emergency fix for P0/P1?
├─ YES → Emergency Change (IC approval)
└─ NO → Continue...

Is this a routine change with documented procedure?
├─ YES → Standard Change (no approval)
└─ NO → Continue...

Does it affect multiple services or have compliance impact?
├─ YES → Major Change (CAB required)
└─ NO → Continue...

Could it cause significant disruption?
├─ YES → Significant Change (Service Owner approval)
└─ NO → Normal Change (Tech Lead approval)
```

### Approval Quick Reference

| Change | Approval | Lead Time | Window |
|--------|----------|-----------|--------|
| Standard | None | 0 | Any |
| Normal | Tech Lead | 2-3 days | Preferred |
| Significant | Service Owner | 1-2 weeks | Required |
| Major | CAB | 4+ weeks | Scheduled |
| Emergency | IC/Owner | 0 | Any |

---

## 14. Contacts

**Change Management:**
- Change Coordinator: change-coordinator@company.com
- CAB Chair: cab-chair@company.com
- Approval Requests: change-approvals@company.com

**Emergency Contacts:**
- Incident Commander: Via PagerDuty
- Service Owner: owner@company.com
- Engineering Manager: manager@company.com

---

**Document Owner:** [Service Owner]  
**Last Updated:** YYYY-MM-DD  
**Next Review:** YYYY-MM-DD
