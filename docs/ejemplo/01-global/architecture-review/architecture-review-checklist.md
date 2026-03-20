# Architecture Review Checklist

> **Purpose:** This checklist helps review panel members systematically evaluate architecture submissions. Not every item applies to every project — use judgment.

---

## Pre-Review (Before Meeting)

### Materials Completeness

- [ ] Architecture diagram provided (C4 Level 2 minimum)
- [ ] Design document attached
- [ ] ADRs provided for significant technology choices
- [ ] Cost estimate included (CAPEX/OPEX)
- [ ] Risk assessment completed
- [ ] Alternatives documented with rationale

### Initial Red Flags

**Mark for deep dive if:**
- [ ] Budget > €200k but no TCO analysis
- [ ] New technology with no skill gap analysis
- [ ] Security-sensitive but no threat model
- [ ] OT/IT boundary crossing without security review
- [ ] Single vendor dependency for critical function
- [ ] No rollback plan documented

---

## Review Meeting Checklist

### 1. Business Alignment

**Problem & Justification**
- [ ] Business problem clearly articulated
- [ ] Quantified benefits (cost savings, risk reduction, etc.)
- [ ] Success criteria defined and measurable
- [ ] Aligns with enterprise strategy

**Scope & Constraints**
- [ ] In-scope vs out-of-scope clearly defined
- [ ] Dependencies identified
- [ ] Timeline realistic (not overly aggressive)
- [ ] Budget reasonable for scope

**Questions to ask:**
- *"What happens if we don't do this project?"*
- *"How will you measure success?"*

---

### 2. Architecture Design

**Clarity & Completeness**
- [ ] Architecture diagram is clear and well-labeled
- [ ] Major components identified
- [ ] Integration points documented
- [ ] Data flows understood

**Design Quality**
- [ ] No obvious single points of failure (for critical systems)
- [ ] Scales to meet expected load
- [ ] Follows enterprise patterns (or has good reason not to)
- [ ] Appropriate level of complexity (not over-engineered)

**Questions to ask:**
- *"Walk me through what happens when a user does X."*
- *"What happens if component Y fails?"*
- *"Why this architecture vs. a simpler alternative?"*

---

### 3. Technology Choices

**Vendor/Product Selection**
- [ ] Technology choices justified (not just "popular")
- [ ] Vendor stability considered (not startup risk without mitigation)
- [ ] Licensing model understood (perpetual vs subscription)
- [ ] Support model defined (vendor SLA, community, in-house)

**Standards & Compatibility**
- [ ] Uses open standards where possible (avoid lock-in)
- [ ] Compatible with existing enterprise tech stack
- [ ] Interoperates with related systems
- [ ] API versioning strategy defined (if exposing APIs)

**Skills & Training**
- [ ] Team has required skills OR training plan exists
- [ ] Documentation/knowledge transfer planned
- [ ] Runbooks will be created
- [ ] Hiring needs identified (if applicable)

**Questions to ask:**
- *"Why this product over the alternatives?"*
- *"What happens if this vendor goes out of business?"*
- *"Do we have the skills to operate this?"*

---

### 4. Security & Compliance

**Authentication & Authorization**
- [ ] Authentication method appropriate (AD, LDAP, SAML, etc.)
- [ ] Least privilege principle applied
- [ ] Role-based access control (RBAC) defined
- [ ] Secrets management strategy (not hardcoded passwords)

**Data Protection**
- [ ] Data classification identified (Platinum/Gold/Silver/Bronze)
- [ ] Encryption at rest (for sensitive data)
- [ ] Encryption in transit (TLS 1.2+ for external, TLS 1.3 preferred)
- [ ] PII handling compliant with GDPR (if applicable)

**Network Security**
- [ ] Firewall rules defined (minimum necessary ports)
- [ ] IT/OT segregation maintained (Purdue model)
- [ ] DMZ or bastion hosts used (for external access)
- [ ] Network segmentation appropriate

**Compliance**
- [ ] Regulatory requirements identified (GDPR, SOX, ISO 27001, etc.)
- [ ] Audit logging enabled (who, what, when)
- [ ] Data retention policies defined
- [ ] Security patching strategy documented

**Questions to ask:**
- *"What is the threat model?"*
- *"How are credentials managed?"*
- *"What audit trails exist?"*

---

### 5. Resilience & Reliability

**High Availability**
- [ ] Redundancy for critical components
- [ ] Failover mechanism tested
- [ ] Load balancing configured (if applicable)
- [ ] No single points of failure (or acceptable risk documented)

**Disaster Recovery**
- [ ] RPO/RTO defined and aligned with business needs
- [ ] Backup strategy documented (3-2-1 rule for critical data)
- [ ] DR testing plan exists
- [ ] Runbook for recovery procedures

**Fault Tolerance**
- [ ] Graceful degradation (doesn't crash entirely)
- [ ] Circuit breakers for external dependencies
- [ ] Timeouts and retries configured
- [ ] Error handling robust (not just "fail fast")

**Questions to ask:**
- *"What is the RTO if the primary database fails?"*
- *"How often will DR be tested?"*
- *"What happens if the integration partner is down?"*

---

### 6. Performance & Scalability

**Capacity Planning**
- [ ] Expected load quantified (users, transactions/sec, data volume)
- [ ] Performance targets defined (latency, throughput)
- [ ] Growth projections for 3-5 years
- [ ] Right-sized (not over-provisioned or under-provisioned)

**Scalability Strategy**
- [ ] Horizontal scaling possible (add instances)
- [ ] Vertical scaling limits understood
- [ ] Database partitioning/sharding strategy (if large scale)
- [ ] Caching strategy defined (if needed)

**Performance Testing**
- [ ] Load testing planned before production
- [ ] Performance benchmarks available (or will be measured)
- [ ] Bottlenecks identified and mitigated

**Questions to ask:**
- *"What happens if load doubles?"*
- *"Have you tested with realistic data volumes?"*
- *"What is the slowest component in the system?"*

---

### 7. Operational Readiness

**Monitoring & Observability**
- [ ] Metrics defined (SLIs: latency, error rate, throughput)
- [ ] Logging strategy (centralized, retention policy)
- [ ] Distributed tracing (if microservices)
- [ ] Dashboards for operations team

**Alerting**
- [ ] Alert thresholds defined (not too noisy, not too quiet)
- [ ] On-call team identified
- [ ] Escalation path documented
- [ ] Alert fatigue considered (actionable alerts only)

**Deployment & Updates**
- [ ] CI/CD pipeline exists (or planned)
- [ ] Zero-downtime deployment (or planned maintenance windows)
- [ ] Rollback procedure documented and tested
- [ ] Canary or blue-green deployment (for high-risk changes)

**Support Model**
- [ ] L1/L2/L3 support defined
- [ ] Runbooks created (install, troubleshoot, recover)
- [ ] Knowledge transfer plan (from project team to ops)
- [ ] Vendor support contract (if critical)

**Questions to ask:**
- *"How will you know if the system is healthy?"*
- *"What does the on-call person do when paged?"*
- *"How do you deploy updates without downtime?"*

---

### 8. Cost & ROI

**Cost Analysis**
- [ ] CAPEX itemized (hardware, licenses, services)
- [ ] OPEX itemized (maintenance, cloud, staff)
- [ ] TCO calculated (3-5 years)
- [ ] Hidden costs considered (training, integration, decommissioning)

**ROI & Justification**
- [ ] Benefits quantified (€ savings, % risk reduction)
- [ ] Payback period reasonable (typically < 3 years for infrastructure)
- [ ] Compared to alternatives (is this the most cost-effective?)
- [ ] Cost vs. value balanced (not gold-plating)

**Questions to ask:**
- *"What is the 5-year TCO?"*
- *"What is the ROI, and when does payback occur?"*
- *"Is there a cheaper way to solve this problem?"*

---

### 9. Risk Management

**Technical Risks**
- [ ] Technology maturity assessed (avoid bleeding edge)
- [ ] Vendor lock-in risk acceptable or mitigated
- [ ] Scalability risks identified
- [ ] Integration risks documented

**Operational Risks**
- [ ] Skills gap identified and training planned
- [ ] Support availability (24/7 for critical systems)
- [ ] Complexity managed (not over-complicated)

**Mitigation Strategies**
- [ ] Each high/medium risk has a mitigation plan
- [ ] Pilot/PoC planned for unproven technology
- [ ] Contingency budget allocated (typically 10-20%)

**Questions to ask:**
- *"What keeps you up at night about this project?"*
- *"What is the worst-case scenario, and how do we handle it?"*

---

### 10. Alternatives & Trade-offs

**Due Diligence**
- [ ] At least 2 alternatives considered
- [ ] Trade-offs documented (why chosen option is better)
- [ ] "Do nothing" option evaluated
- [ ] Bias check (not just picking favorite vendor)

**Decision Quality**
- [ ] Objective criteria used (not subjective preference)
- [ ] Assumptions documented
- [ ] Unknowns acknowledged ("we don't know X yet")

**Questions to ask:**
- *"What other approaches did you consider?"*
- *"Why is this better than doing nothing?"*
- *"What would change your decision?"*

---

## Post-Meeting

### Decision Documentation

- [ ] Decision recorded (Approved / Conditional / Rejected)
- [ ] Conditions documented (if conditional approval)
- [ ] Feedback provided to project team (constructive, actionable)
- [ ] ADRs filed in architecture repository

### Follow-Up Actions

- [ ] Conditions tracked in Jira
- [ ] Review notes shared with project team (within 2 days)
- [ ] Post-implementation review scheduled (for major projects)

---

## Common Pitfalls to Watch For

### Anti-Patterns

- **Resume-driven development:** "We chose Kubernetes because it's cool" (no business justification)
- **Not invented here:** Rejecting external solutions just because they're external
- **Big ball of mud:** No clear separation of concerns, everything tightly coupled
- **Single point of failure:** Critical component has no redundancy
- **Vendor lock-in:** Proprietary formats, no export capability
- **Over-engineering:** Solving problems that don't exist yet
- **Under-engineering:** Ignoring known future requirements

### Red Flags

- **Vague answers:** "We'll figure it out later" for critical questions
- **No alternatives:** Only one option considered
- **No risks:** "There are no risks" (unrealistic)
- **No metrics:** "We'll know it's working when users are happy" (not measurable)
- **No rollback:** "If it fails, we'll fix forward" (risky for critical systems)

---

## Review Panel Scoring (Optional)

**Use this scoring guide to reach consensus:**

### Score Scale

| Score | Meaning | Action |
|-------|---------|--------|
| **5** | Excellent design, exemplary | Approve — share as best practice |
| **4** | Good design, minor gaps | Approve with minor conditions |
| **3** | Acceptable, some concerns | Conditional approval (address concerns) |
| **2** | Significant issues | Conditional approval with major rework OR Reject |
| **1** | Fundamental flaws | Reject — major redesign required |

### Scoring Dimensions

Rate each dimension 1-5, then average:

- **Business alignment:** Does it solve the right problem?
- **Technical soundness:** Is the design robust?
- **Security & compliance:** Are risks managed?
- **Operational readiness:** Can we support it?
- **Cost-effectiveness:** Is the ROI reasonable?

**Consensus:** Panel should aim for agreement within ±1 point. If spread is >1, discuss differences before deciding.

---

## Reviewer Mindset

### Do:

✅ **Ask "why"** — understand the reasoning, not just the "what"  
✅ **Challenge assumptions** — test if they hold up  
✅ **Offer alternatives** — suggest improvements  
✅ **Acknowledge good work** — positive feedback motivates  
✅ **Focus on high-impact issues** — don't nitpick minor details  

### Don't:

❌ **Impose your preferences** — "I prefer X" is not valid feedback  
❌ **Block without justification** — explain why something is a problem  
❌ **Dismiss context** — "That's not how we do it at Google" (we're not Google)  
❌ **Be adversarial** — reviews are collaborative, not interrogations  
❌ **Demand perfection** — good enough is often good enough  

---

## Checklist Version

**Version:** 1.0  
**Last Updated:** 2025-03-20  
**Owner:** Chief Architect

**Feedback:** architecture-review@company.com
