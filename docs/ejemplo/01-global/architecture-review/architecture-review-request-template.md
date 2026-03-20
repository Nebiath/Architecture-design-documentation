# Architecture Review Request Form

> **Instructions:** Complete this form and submit to `architecture-review@company.com` at least 2 weeks before planned implementation.

---

## 1. Project Information

| Field                           | Value                                         |
| ------------------------------- | --------------------------------------------- |
| **Project Name**                |                                               |
| **Project ID** (Jira/PM tool)   |                                               |
| **Plant/Department**            |                                               |
| **Project Lead**                | Name, email, phone                            |
| **Technical Lead**              | Name, email, phone                            |
| **Requested Review Date**       | YYYY-MM-DD (must be 2+ weeks from submission) |
| **Planned Implementation Date** | YYYY-MM-DD                                    |

---

## 2. Project Overview

### 2.1 Business Justification

**What business problem does this solve?**

> *(2-3 sentences explaining the need)*

**Expected benefits:**

- [ ] Cost reduction: Estimated €_______/year
- [ ] Risk mitigation: *(describe)*
- [ ] Compliance requirement: *(regulation/standard)*
- [ ] Capacity increase: *(metric)*
- [ ] Other: *(specify)*

**Success criteria:**

> *(How will you measure if this project succeeded?)*

### 2.2 Scope

**In scope:**

- *(What systems/plants are affected?)*

**Out of scope:**

- *(What is explicitly NOT included?)*

**Dependencies:**

- *(What other projects or systems does this depend on?)*

---

## 3. Architecture Summary

### 3.1 Current State (if applicable)

**Existing solution (if replacing):**

> *(Briefly describe what exists today, or write "N/A - greenfield")*

**Pain points with current solution:**

- *(What problems are we solving?)*

### 3.2 Proposed Architecture

**Attach:** Architecture diagram (C4 Level 2 minimum) — use Mermaid, draw.io, or Lucidchart

**High-level description:**

> *(3-5 sentences describing the solution. Reference the diagram.)*

**Major components:**

1. Component A: *(brief description)*
2. Component B: *(brief description)*
3. ...

**Integration points:**

- Integrates with System X via *(protocol/API)*
- Consumes data from System Y
- Publishes events to System Z

### 3.3 Technology Choices

| Technology/Product   | Purpose      | Rationale                              |
| -------------------- | ------------ | -------------------------------------- |
| *(e.g., PostgreSQL)* | *(Database)* | *(Why this over MySQL, Oracle, etc.?)* |
|                      |              |                                        |
|                      |              |                                        |

**New technologies (not currently used in enterprise):**

- [ ] None — uses existing tech stack
- [ ] Yes — see ADRs attached

---

## 4. Non-Functional Requirements

### 4.1 Performance

- **Expected load:** ___ requests/second, ___ concurrent users
- **Latency target:** p95 < ___ ms, p99 < ___ ms
- **Data volume:** ___ TB/year growth

### 4.2 Availability

- **Tier:** (check one)
  
  - [ ] Platinum (< 4h RTO, < 4h RPO)
  - [ ] Gold (< 24h RTO, < 24h RPO)
  - [ ] Silver (< 72h RTO, < 7d RPO)
  - [ ] Bronze (< 2 weeks RTO, < 30d RPO)

- **Uptime target:** ___% (e.g., 99.9%)

- **Planned maintenance window:** *(e.g., Sundays 02:00-06:00)*

### 4.3 Security

- **Data classification:** (check all that apply)
  
  - [ ] Platinum (SAP production, Tier 1 DBs, SCADA)
  - [ ] Gold (File servers, Tier 2 DBs)
  - [ ] Silver (Non-critical apps)
  - [ ] Bronze (Archival, historical)

- **Crosses IT/OT boundary?**
  
  - [ ] Yes — **Security review mandatory**
  - [ ] No

- **Authentication method:**
  
  - [ ] Active Directory
  - [ ] LDAP
  - [ ] Certificate-based
  - [ ] API key
  - [ ] Other: *(specify)*

- **Encryption:**
  
  - [ ] At rest: *(AES-256, etc.)*
  - [ ] In transit: *(TLS 1.3, etc.)*

### 4.4 Compliance

**Regulatory requirements:** (check all that apply)

- [ ] GDPR (data residency, right to deletion)
- [ ] ISO 27001
- [ ] SOX (financial data retention)
- [ ] HIPAA (healthcare data)
- [ ] Other: *(specify)*

---

## 5. Risk Assessment

### 5.1 Technical Risks

| Risk                                  | Likelihood (L/M/H) | Impact (L/M/H) | Mitigation                                |
| ------------------------------------- | ------------------ | -------------- | ----------------------------------------- |
| *(e.g., Vendor discontinues product)* | M                  | H              | *(Multi-vendor strategy, open standards)* |
|                                       |                    |                |                                           |
|                                       |                    |                |                                           |

### 5.2 Operational Risks

| Risk                                     | Likelihood | Impact | Mitigation                                 |
| ---------------------------------------- | ---------- | ------ | ------------------------------------------ |
| *(e.g., Insufficient skills to support)* |            |        | *(Training plan, vendor support contract)* |
|                                          |            |        |                                            |
|                                          |            |        |                                            |

### 5.3 Security Risks

| Risk                                        | Likelihood | Impact | Mitigation                          |
| ------------------------------------------- | ---------- | ------ | ----------------------------------- |
| *(e.g., Unauthorized access to OT network)* |            |        | *(Firewall rules, least privilege)* |
|                                             |            |        |                                     |
|                                             |            |        |                                     |

---

## 6. Alternatives Considered

**Alternative 1:**

- **Description:** *(What was the alternative approach?)*
- **Why rejected:** *(Cost? Complexity? Lack of skills?)*

**Alternative 2:**

- **Description:**
- **Why rejected:**

**Do nothing (status quo):**

- **Why not viable:** *(Why must we do something?)*

---

## 7. Cost Estimate

### 7.1 CAPEX (Initial Investment)

| Item                  | Cost (EUR)   |
| --------------------- | ------------ |
| Hardware              | €_______     |
| Software licenses     | €_______     |
| Professional services | €_______     |
| **Total CAPEX**       | **€_______** |

### 7.2 OPEX (Annual Recurring)

| Item                 | Cost (EUR/year) |
| -------------------- | --------------- |
| Software maintenance | €_______        |
| Cloud/hosting fees   | €_______        |
| Support contracts    | €_______        |
| Staff (FTE × salary) | €_______        |
| **Total OPEX**       | **€_______**    |

### 7.3 TCO (3-5 Year)

**Total Cost of Ownership (5 years):** €_______ CAPEX + (€_______ OPEX × 5) = **€_______**

---

## 8. Implementation Plan

### 8.1 Timeline

| Phase                 | Duration  | Start Date | End Date   |
| --------------------- | --------- | ---------- | ---------- |
| Design                | ___ weeks | YYYY-MM-DD | YYYY-MM-DD |
| Procurement           | ___ weeks | YYYY-MM-DD | YYYY-MM-DD |
| Development/Config    | ___ weeks | YYYY-MM-DD | YYYY-MM-DD |
| Testing               | ___ weeks | YYYY-MM-DD | YYYY-MM-DD |
| Pilot (if applicable) | ___ weeks | YYYY-MM-DD | YYYY-MM-DD |
| Production rollout    | ___ weeks | YYYY-MM-DD | YYYY-MM-DD |

### 8.2 Rollback Plan

**If implementation fails, how do we rollback?**

> *(Describe rollback procedure or write "N/A - greenfield")*

**Rollback RTO:** ___ hours/days

---

## 9. Operations Handoff

### 9.1 Support Model

**Who will support this system in production?**

- **L1 (helpdesk):** *(Team name)*
- **L2 (engineering):** *(Team name)*
- **L3 (vendor):** *(Vendor name, contract level)*

**On-call rotation:**

- [ ] Yes — *(Team name)*
- [ ] No (business hours only)

### 9.2 Monitoring

**What will be monitored?**

- [ ] Availability (uptime)
- [ ] Performance (latency, throughput)
- [ ] Capacity (disk, CPU, memory)
- [ ] Security (failed logins, unauthorized access)

**Monitoring tools:**

- [ ] Prometheus + Grafana
- [ ] Zabbix
- [ ] Azure Monitor
- [ ] Splunk
- [ ] Other: *(specify)*

### 9.3 Documentation

**Runbooks created:** (check when complete)

- [ ] Installation/deployment
- [ ] Backup and restore
- [ ] Common troubleshooting
- [ ] Disaster recovery

---

## 10. Attachments

**Required:**

- [ ] Architecture diagram (C4 Level 2 minimum)
- [ ] Design document (use template)
- [ ] ADRs for key technology decisions

**Optional but recommended:**

- [ ] Data flow diagram
- [ ] Sequence diagrams
- [ ] Cost model spreadsheet
- [ ] Vendor proposals/quotes

---

## 11. Review Preferences

**Preferred review date:** YYYY-MM-DD (2+ weeks out)

**Attendees from project team:**

1. *(Name, Role)*
2. *(Name, Role)*

**Specific areas where you need feedback:**

> *(E.g., "We're uncertain about the database choice" or "Need validation on security model")*

**Questions for the panel:**

> *(Specific questions you want answered during review)*

---

## Submission Checklist

Before submitting, verify:

- [ ] All required fields completed
- [ ] Architecture diagram attached
- [ ] Design document attached
- [ ] ADRs attached (if new technology)
- [ ] Review requested at least 2 weeks in advance
- [ ] Cost estimate included
- [ ] Risk assessment completed
- [ ] Alternatives documented

**Submit to:** architecture-review@company.com  
**Subject:** [ARCH REVIEW] *(Project Name)* - *(Plant/Department)*

---

**Form Version:** 1.0  
**Last Updated:** 2025-03-20
