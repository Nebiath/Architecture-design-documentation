# Architecture Team Demand Management & Prioritization

> **Version:** 1.0  
> **Date:** 2025-03-20  
> **Status:** Approved  
> **Author:** IT Architecture Team  
> **Owner:** Chief Architect  
> **Purpose:** Manage incoming requests, prioritize effectively, prevent burnout, ensure complete documentation

---

## 1. Executive Summary

### 1.1 The Problem

Small architecture teams face a common challenge:
- **Unlimited demand** vs **limited capacity** (everything is "urgent")
- **Stakeholder pressure** ("My project is blocked waiting for you")
- **Context switching** (interruptions prevent deep work)
- **Incomplete documentation** (rush to next request before finishing current)
- **Burnout risk** (team works weekends to catch up)

### 1.2 The Solution

This document establishes a **demand management system** that:
- **Objectively prioritizes** requests (not "who shouts loudest")
- **Makes capacity visible** (stakeholders see the queue)
- **Protects team focus** (WIP limits prevent context switching)
- **Ensures completeness** (Definition of Done prevents incomplete work)
- **Maintains sustainability** (realistic workload prevents burnout)

### 1.3 Key Principles

**Transparency over gatekeeping:**
- All requests are visible in a shared backlog
- Prioritization criteria are public and objective
- Stakeholders can see where their request is in the queue

**Capacity-based planning:**
- Team capacity is known and protected
- Work in Progress (WIP) limits prevent overload
- Saying "no" or "not now" is acceptable (with explanation)

**Complete over fast:**
- Better to finish 5 requests completely than 10 half-done
- Definition of Done ensures quality
- No "documentation debt"

**Sustainable pace:**
- Team works normal hours (40h/week)
- Buffer for urgent issues (20% capacity reserved)
- Regular retrospectives to improve process

---

## 2. Intake Process

### 2.1 How Requests Enter the System

**Single point of entry:** All requests come through one channel (no hallway requests, no direct DMs).

**Intake options:**
1. **Email:** `architecture-requests@company.com`
2. **Jira form:** "Request Architecture Support" (auto-creates ticket)
3. **Weekly office hours:** 1-hour slot for quick consultations

**No exceptions:** Even if the CTO asks, the request goes through intake (ensures fairness).

### 2.2 Intake Form Fields

All requests must include:

| Field | Purpose |
|-------|---------|
| **Requestor** | Name, email, department |
| **Business sponsor** | Who is paying for this? (budget owner) |
| **Request type** | New service, review, consultation, documentation update |
| **Business objective** | What business problem are we solving? |
| **Success criteria** | How will we know this is done? |
| **Deadline** | Why this date? (real constraint vs wishful thinking) |
| **Impact if delayed** | What happens if we don't do this now? |
| **Budget** | CAPEX/OPEX (helps prioritize) |

**Incomplete requests:** Automatically rejected with request to resubmit.

### 2.3 Initial Triage (Architecture Lead — 1 hour/day)

**Daily review** of new requests (first hour of day):

1. **Validate completeness** — All fields filled?
2. **Classify urgency** — P0/P1/P2/P3 (see Section 3)
3. **Estimate effort** — S/M/L/XL (see Section 4)
4. **Assign to backlog** — Prioritized queue

**Auto-accept (no triage needed):**
- Critical production issues (P0)
- Security vulnerabilities
- Regulatory compliance (deadline-driven)

**Auto-defer (not architecture work):**
- Operational tasks (ticket to Ops team)
- Software development (ticket to Dev team)
- Purchasing/procurement (ticket to Procurement)

---

## 3. Prioritization Framework

### 3.1 Priority Levels

| Priority | Criteria | Response Time | Examples |
|----------|----------|--------------|----------|
| **P0 - Critical** | Production down, security breach, regulatory deadline < 1 week | Immediate (drop everything) | Production outage, data breach, SOX audit in 3 days |
| **P1 - Urgent** | Business impact, deadline < 4 weeks, high revenue risk | Within 3 days | Major project blocked, vendor deadline, executive commitment |
| **P2 - Normal** | Standard business need, deadline > 4 weeks | Within 2 weeks | New service design, annual review, documentation update |
| **P3 - Low** | Nice-to-have, no firm deadline | Best effort (fill gaps) | Optimization, exploration, nice-to-haves |

**Priority ≠ Order:** A P2 request submitted before a P1 may still get done first if the P2 is already in progress and nearly complete (avoid waste).

### 3.2 RICE Scoring (for P1/P2 requests)

When multiple P1 or P2 requests compete, use **RICE scoring** to rank:

**RICE = (Reach × Impact × Confidence) / Effort**

| Factor | Definition | Scale |
|--------|-----------|-------|
| **Reach** | How many people/plants affected? | 1 (1 plant) to 10 (all plants) |
| **Impact** | How much does it improve outcomes? | 1 (minimal) to 5 (massive) |
| **Confidence** | How sure are we of Reach/Impact? | 0.5 (low) to 1.0 (high) |
| **Effort** | Person-weeks to complete | 1 (1 week) to 8 (2 months) |

**Example:**

| Request | Reach | Impact | Confidence | Effort | RICE Score |
|---------|-------|--------|-----------|--------|------------|
| **Backup standard** (30 plants) | 10 | 5 | 1.0 | 4 | (10×5×1.0)/4 = **12.5** |
| **New monitoring tool** (1 plant pilot) | 1 | 3 | 0.7 | 2 | (1×3×0.7)/2 = **1.05** |
| **DR testing automation** (all plants) | 10 | 4 | 0.8 | 6 | (10×4×0.8)/6 = **5.3** |

**Priority order:** Backup standard (12.5) > DR automation (5.3) > Monitoring tool (1.05)

### 3.3 MoSCoW Categorization

For roadmap planning (quarterly), categorize requests:

- **Must-Have:** Regulatory, contractual commitment, critical dependency
- **Should-Have:** High business value, strategic importance
- **Could-Have:** Good to have if time permits
- **Won't-Have (this quarter):** Deferred to future quarter

**Capacity allocation:**
- Must-Have: 60% of capacity
- Should-Have: 25% of capacity
- Could-Have: 15% of capacity (buffer for learning, innovation)

---

## 4. Effort Estimation

### 4.1 T-Shirt Sizing

Use relative sizing, not absolute hours (humans are bad at estimating hours):

| Size | Definition | Examples | Typical Duration |
|------|-----------|----------|------------------|
| **XS** | Quick consultation | 15-min call, email clarification | < 1 hour |
| **S** | Simple task | Review 1 diagram, update 1 document | 1-4 hours |
| **M** | Standard task | Write 1 new standard, conduct 1 architecture review | 1-2 days |
| **L** | Complex task | Design new service (10+ pages), pilot + documentation | 1-2 weeks |
| **XL** | Major initiative | Multi-service architecture, enterprise-wide rollout | 1-2 months |

**Uncertainty buffer:**
- S/M tasks: +25% (known unknowns)
- L tasks: +50% (more unknowns)
- XL tasks: +100% (many unknowns)

**Re-estimate:** If a task takes >50% longer than estimated, retrospect why (improve future estimates).

### 4.2 Team Capacity

**Available capacity per week:**
- Team size: 3 people × 40 hours = 120 hours/week
- Meetings/admin: -20% = 96 hours/week
- **Buffer for urgent issues (P0/P1):** -20% = 77 hours/week **planned capacity**

**Monthly capacity:**
- 77 hours/week × 4 weeks = **~300 hours/month** for planned work

**Don't overcommit:** Only accept work that fits in planned capacity. P0/P1 urgents will fill the buffer.

---

## 5. Work in Progress (WIP) Limits

### 5.1 The Problem with No WIP Limits

**Without limits:**
- Team works on 15 things simultaneously
- Constant context switching (lose 20-30% productivity)
- Nothing finishes (all items 80% done)
- Stakeholders frustrated (no visibility to completion)

**With WIP limits:**
- Team focuses on 3-5 things
- Items move to "Done" faster
- Stakeholders see progress
- Team feels accomplishment (not overwhelm)

### 5.2 Recommended WIP Limits

**For a 3-person team:**

| Status | WIP Limit | Reasoning |
|--------|-----------|----------|
| **In Progress** | 3 items | 1 per person (allows pairing if needed) |
| **In Review** | 2 items | Waiting for stakeholder feedback |
| **Blocked** | No limit | Tracking only (escalate if > 3 items) |

**Pull-based system:**
- When item moves to "Done", **then** pull next highest priority from backlog
- Never start new work until capacity frees up

### 5.3 Handling "Everything is Urgent"

**Stakeholder:** "Can you just squeeze this in? It's urgent!"

**Response template:**
> "I understand this is important to you. Our current WIP limit is 3 items to ensure we complete work with quality and don't burn out the team. Here's what we're working on now: [list 3 items]. Which of these should we pause to start yours? Or would you like to schedule it as next in queue when one completes?"

**This conversation:**
- Makes trade-offs visible
- Puts prioritization decision on stakeholder (they decide what to deprioritize)
- Protects team from overcommitment

---

## 6. Definition of Done (Prevents Incomplete Documentation)

### 6.1 The Problem

**Common scenario:**
- Team writes 80% of a design document
- "Urgent" request comes in
- Team switches to new request
- Original document sits unfinished for months
- **Documentation debt accumulates**

### 6.2 Definition of Done Checklist

**A work item is "Done" when:**

For **Design Documents:**
- [ ] All sections completed (no TODOs, no placeholders)
- [ ] Diagrams included (C4 Level 2 minimum)
- [ ] Reviewed by peer (another architect)
- [ ] Reviewed by stakeholder (business owner signs off)
- [ ] Published to wiki/SharePoint (accessible to consumers)
- [ ] Announced to relevant teams (email, Slack)

For **Architecture Reviews:**
- [ ] Review meeting conducted
- [ ] Decision documented (Approved/Conditional/Rejected)
- [ ] Feedback sent to project team (within 2 days)
- [ ] ADRs filed (if applicable)
- [ ] Conditions tracked in Jira (if conditional approval)

For **Standards:**
- [ ] Document written (all sections complete)
- [ ] Peer reviewed (2 architects)
- [ ] Stakeholder reviewed (security, ops, etc.)
- [ ] Templates created (if applicable)
- [ ] Approval signatures obtained (CTO, CISO)
- [ ] Published and announced
- [ ] Training materials created (if needed)

**No partial credit:** Item is not "Done" until all checkboxes are complete. 90% done = 0% done for planning purposes.

### 6.3 Incomplete Work Policy

**If urgent work interrupts:**
- Document current state in ticket ("paused at section 4, need to add diagrams + peer review")
- Estimate remaining effort to complete
- Assign to backlog with appropriate priority
- Resume **before** claiming item is "Done"

**Monthly cleanup:**
- Review all items "In Progress" > 2 weeks
- Escalate blockers or deprioritize (move to backlog)
- Don't leave zombies (half-done work that never finishes)

---

## 7. Calendar and Roadmap

### 7.1 Visible Backlog

**Use Kanban board** (Jira, Trello, or physical board):

```
┌─────────────┬────────────┬─────────────┬──────────┬────────┐
│   BACKLOG   │   READY    │ IN PROGRESS │ IN REVIEW│  DONE  │
│             │            │             │          │        │
│ [30 items]  │ [5 items]  │   [3 items] │ [2 items]│ [120]  │
│             │            │   (WIP=3)   │          │        │
│ Prioritized │ Next up    │   Active    │ Waiting  │Archive │
│ by RICE     │            │             │          │        │
└─────────────┴────────────┴─────────────┴──────────┴────────┘
```

**Public access:**
- Stakeholders can view board (read-only)
- See where their request is in queue
- Reduces "status check" emails

### 7.2 Quarterly Roadmap

**Every quarter, publish roadmap:**

**Q2 2025 Architecture Roadmap**

| Theme | Initiative | Size | Status | Owner |
|-------|-----------|------|--------|-------|
| **Standardization** | Backup standard (all plants) | XL | In Progress | Maria |
| **Standardization** | Kubernetes standard | L | Planned (Q2) | Hans |
| **Governance** | Architecture review process | M | Done ✅ | Lisa |
| **Security** | IT/OT firewall ruleset review | M | Planned (Q2) | Sarah |
| **Innovation** | Azure migration POC | L | Backlog (Q3) | Robert |

**Communicate:**
- Share roadmap with all stakeholders (email, wiki)
- Explain what's **in** and what's **out** (and why)
- Set expectations (no surprises)

### 7.3 Sprint Planning (Bi-Weekly)

**Every 2 weeks:**
1. Review completed work (celebrate wins)
2. Review backlog priorities (re-score RICE if new info)
3. Pull next items into "Ready" (enough for 2 weeks)
4. Identify dependencies or blockers
5. Communicate plan to stakeholders

**Velocity tracking:**
- Measure: How many story points completed per sprint?
- Use historical velocity to forecast future capacity
- Adjust commitments if velocity changes

---

## 8. Stakeholder Communication

### 8.1 Managing Expectations

**When a request comes in, respond within 1 business day:**

**Template email:**
```
Subject: RE: Architecture Request - [Topic]

Hi [Stakeholder],

Thanks for submitting your request for [topic]. I've reviewed it and here's the status:

**Priority:** P2 (Normal)
**Estimated Effort:** M (1-2 days)
**Current Queue Position:** #7 in backlog
**Estimated Start Date:** ~3 weeks (mid-April)

**What happens next:**
1. Your request is in our prioritized backlog (you can view here: [link])
2. We'll reach out 1 week before we start to confirm details
3. Expected completion: late April (2 weeks after start)

**If this timeline doesn't work:**
Please reply with the business impact if we delay beyond [date]. We can escalate to our Chief Architect if this is blocking a critical initiative.

Questions? Reply to this email or join our weekly office hours (Thursdays 2-3pm).

Thanks,
[Architecture Team]
```

**This communication:**
- Sets clear expectations (no surprises)
- Gives visibility (they can track progress)
- Offers escalation path (if truly urgent)

### 8.2 Weekly Status Email

**Every Friday, send status update to all active requestors:**

**Subject: Architecture Team - Weekly Status (Mar 15, 2025)**

**Completed this week:**
- ✅ Backup standard (EU plants) — Published to wiki
- ✅ Architecture review: SAP S/4HANA migration — Conditionally approved

**In progress:**
- 🔄 Kubernetes standard (60% complete, on track)
- 🔄 Firewall ruleset review (blocked on Security approval)

**Starting next week:**
- 🆕 DR automation design (Plant NA-03)

**Backlog top 5:**
1. Azure migration POC (estimated start: Apr 5)
2. Monitoring platform selection
3. Network segmentation standard
4. ...

**Questions or concerns?** Reply to this email or join office hours.

### 8.3 Quarterly Business Reviews

**With executive stakeholders (CTO, CISO, CIO):**

**Agenda:**
1. **Metrics** (last quarter)
   - Requests received: 45
   - Completed: 38 (84%)
   - In progress: 7
   - Average lead time: 15 days (target: < 21 days)
   - Stakeholder satisfaction: 8.5/10

2. **Highlights**
   - Major deliverables (backup standard, architecture review process)
   - Business impact (€2M savings from backup optimization)

3. **Challenges**
   - Team capacity vs demand (45 requests, can handle ~35)
   - Recommendations: Hire 1 additional architect OR reduce scope

4. **Next Quarter Roadmap**
   - Priorities (Kubernetes, cloud migration)
   - Risks and dependencies

**Outcome:** Executive alignment on priorities, capacity, and expectations.

---

## 9. Handling Escalations

### 9.1 When Stakeholders Disagree with Prioritization

**Scenario:** "Why is my request P2 but theirs is P1?"

**Response process:**
1. **Explain criteria** (show RICE scoring)
2. **Invite re-evaluation** ("Did I miss something? What's the impact if we delay?")
3. **Offer trade-off** ("If yours becomes P1, which current P1 should drop to P2?")
4. **Escalate if needed** (Chief Architect or CTO makes final call)

**Decision log:**
- Document all escalations (who, what, decision, rationale)
- Prevents revisiting same decision multiple times

### 9.2 Escalation Levels

| Level | Who | Timeline | Typical Use |
|-------|-----|----------|-------------|
| **L1 - Team** | Architecture Lead | 1 day | Most requests (95%) |
| **L2 - Chief Architect** | Chief Architect | 3 days | Conflicting P1s, resource constraints |
| **L3 - CTO** | CTO | 1 week | Strategic decisions, budget |

**Escalation criteria:**
- Requestor disagrees with priority
- Multiple P1s compete for same capacity
- Request requires budget approval
- Political/organizational conflict

---

## 10. Preventing Burnout

### 10.1 Sustainable Pace

**Work-life balance commitments:**
- 40-hour weeks (no regular overtime)
- No weekend work (except P0 emergencies)
- Vacation is mandatory (use it or lose it)
- On-call rotation (if 24/7 coverage needed)

**Buffer capacity:**
- 20% capacity reserved for urgents (prevents overload)
- If buffer consumed for 2 weeks straight → signal to management (unsustainable demand)

### 10.2 Focus Time

**Deep work blocks:**
- Every architect gets 4 hours/day of uninterrupted time (no meetings)
- Suggested: Mornings (9am-1pm) for deep work, afternoons for meetings

**No-meeting days:**
- 1 day/week (e.g., Wednesdays) — no internal meetings
- Only external stakeholder meetings if critical

### 10.3 Team Retrospectives

**Bi-weekly retrospective (30 minutes):**

**Questions:**
- What went well?
- What didn't go well?
- What should we change?

**Action items:**
- Pick 1-2 improvements to implement next sprint
- Assign owner and track completion

**Examples of improvements:**
- "Too many interrupts" → Set office hours, reduce ad-hoc requests
- "Estimates always wrong" → Improve estimation training
- "Waiting on reviews" → Set SLA for peer reviews (48h)

---

## 11. Metrics and Continuous Improvement

### 11.1 Key Metrics

| Metric | Target | Purpose |
|--------|--------|---------|
| **Lead Time** (request → done) | < 21 days (P2) | Measure responsiveness |
| **Cycle Time** (start → done) | < 7 days (P2) | Measure efficiency |
| **Throughput** (items/month) | 8-10 (baseline) | Measure capacity |
| **WIP Age** (days in progress) | < 14 days | Prevent stale work |
| **Backlog Size** | < 30 items | Prevent overwhelming queue |
| **Stakeholder Satisfaction** | > 8/10 | Measure value delivery |

**Track monthly:**
- Create dashboard (Jira report or spreadsheet)
- Review trends (improving or degrading?)
- Identify bottlenecks

### 11.2 Bottleneck Analysis

**If Lead Time is increasing, analyze:**
- **Where do items spend most time?** (Backlog? In Progress? Review?)
- **What's blocking items?** (Waiting for approvals? Missing info?)
- **Root cause:** Address the constraint (e.g., slow reviews → set SLA)

**Common bottlenecks:**
- Peer review (solution: set 48h SLA for reviews)
- Stakeholder feedback (solution: set deadline, move on if no response)
- External dependencies (solution: escalate blockers weekly)

### 11.3 Continuous Improvement

**Quarterly review:**
1. Analyze metrics (trends, outliers)
2. Celebrate wins (what went well?)
3. Identify problems (what's hurting us?)
4. Pick top 2-3 improvements (focus)
5. Implement and measure

**Example improvements:**
- Problem: "Too much rework because requirements unclear"
- Solution: Add "Requirements Review" gate before starting work
- Measure: Reduce rework from 30% to 10% in Q3

---

## 12. Tools and Templates

### 12.1 Recommended Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| **Jira** | Backlog, Kanban board, reporting | Industry standard |
| **Trello** | Simple Kanban (if no Jira) | Lightweight alternative |
| **Confluence** | Documentation, roadmaps | Knowledge base |
| **Slack** | Communication, office hours | #architecture-requests channel |
| **Google Sheets** | RICE scoring, capacity planning | Simple and accessible |

### 12.2 Templates

**Included in this package:**
- **Request Intake Form** (Appendix A)
- **RICE Scoring Template** (Appendix B)
- **Quarterly Roadmap Template** (Appendix C)
- **Weekly Status Email Template** (Appendix D)

---

## 13. Implementation Plan

### Phase 1: Setup (Week 1-2)

- [ ] Create intake form (Jira or email template)
- [ ] Set up Kanban board (columns: Backlog, Ready, In Progress, Review, Done)
- [ ] Document prioritization criteria (RICE scoring, priority levels)
- [ ] Announce new process to stakeholders (email, town hall)

### Phase 2: Pilot (Week 3-6)

- [ ] Route all requests through intake
- [ ] Score and prioritize existing backlog
- [ ] Set WIP limits (start with 3)
- [ ] Send weekly status emails
- [ ] Retrospective after 4 weeks (what's working? what's not?)

### Phase 3: Optimize (Week 7-12)

- [ ] Refine RICE criteria based on learnings
- [ ] Adjust WIP limits if needed
- [ ] Implement Definition of Done
- [ ] Track metrics (lead time, cycle time, throughput)
- [ ] Quarterly business review with executives

### Phase 4: Scale (Month 4+)

- [ ] Train new team members on process
- [ ] Automate reporting (Jira dashboards)
- [ ] Continuous improvement (quarterly retrospectives)
- [ ] Expand to other teams (if successful)

---

## 14. FAQ

**Q: What if the CTO asks for something urgently?**  
A: Same process. Request goes through intake, gets prioritized. CTO's request may be P1, but still visible in queue. Transparency applies to everyone.

**Q: What if we genuinely have too much demand?**  
A: Measure and communicate. If demand exceeds capacity for 3+ months, escalate to management with data: "We received 45 requests/month but can only handle 35. Options: (1) Hire, (2) Reduce scope, (3) Accept delays."

**Q: How do we handle "drive-by" requests (hallway, Slack DM)?**  
A: Politely redirect: "Can you submit through [intake form]? This ensures we don't lose track and can prioritize fairly against other requests."

**Q: What if a stakeholder bypasses the process?**  
A: Consistent enforcement. Redirect to process every time. Escalate to Chief Architect if someone repeatedly bypasses.

**Q: Can we make exceptions for small requests?**  
A: Yes, but define "small." E.g., <1 hour = quick answer via email (no ticket). >1 hour = goes through intake.

---

## 15. Success Criteria

**You'll know this process is working when:**

✅ Team works normal hours (no weekend work)  
✅ Stakeholders know where their request is (no "status check" emails)  
✅ Work gets finished (not 10 things 80% done)  
✅ Prioritization is transparent (stakeholders accept "not now" with explanation)  
✅ Documentation quality is high (no incomplete work)  
✅ Team feels in control (not overwhelmed by chaos)  

---

## Appendix A: Request Intake Form

```markdown
## Architecture Request Form

**Requestor Information:**
- Name:
- Email:
- Department:
- Business Sponsor (budget owner):

**Request Details:**
- Request Type: [ ] New Service [ ] Architecture Review [ ] Consultation [ ] Documentation
- Brief Description (1-2 sentences):
- Business Objective (what problem are we solving?):
- Success Criteria (how will we know this is done?):

**Timeline:**
- Requested Start Date:
- Deadline (if any):
- Reason for deadline:

**Impact:**
- Number of people/plants affected:
- What happens if we delay?:
- Budget (CAPEX/OPEX):

**Additional Context:**
- Related projects/dependencies:
- Attachments (diagrams, docs, etc.):
```

---

## Appendix B: RICE Scoring Template

| Request ID | Description | Reach (1-10) | Impact (1-5) | Confidence (0.5-1.0) | Effort (weeks) | RICE Score | Priority |
|-----------|-------------|--------------|--------------|---------------------|----------------|------------|----------|
| ARCH-001 | Backup standard | 10 | 5 | 1.0 | 4 | 12.5 | P1 |
| ARCH-002 | K8s standard | 8 | 4 | 0.8 | 3 | 8.5 | P1 |
| ARCH-003 | Monitoring POC | 1 | 3 | 0.7 | 2 | 1.05 | P2 |

**Sort by RICE Score (descending) to get priority order.**

---

## Appendix C: Quarterly Roadmap Template

**Q2 2025 Architecture Team Roadmap**

**Theme: Standardization & Governance**

**Committed (Must-Have):**
- [ ] Kubernetes standard (4 weeks) — All plants
- [ ] Firewall ruleset review (2 weeks) — Security compliance
- [ ] Architecture review process (done) — Governance

**Planned (Should-Have):**
- [ ] Azure migration POC (6 weeks) — 2 plants pilot
- [ ] Network segmentation standard (3 weeks) — OT/IT segregation

**Backlog (Could-Have):**
- Monitoring platform selection
- DR automation design
- Cost optimization review

**Won't-Have (Deferred to Q3):**
- Service mesh evaluation
- Multi-cloud strategy

---

## Appendix D: Weekly Status Email Template

**Subject: Architecture Team Status - Week of [Date]**

**Completed:**
- ✅ [Item 1] — [Brief outcome]
- ✅ [Item 2] — [Brief outcome]

**In Progress:**
- 🔄 [Item 3] — [% complete, on track / blocked / at risk]
- 🔄 [Item 4] — [Status]

**Upcoming:**
- 🆕 [Item 5] — Starting [date]

**Backlog Highlights (Top 3):**
1. [Item] — Est. start [date]
2. [Item] — Est. start [date]
3. [Item] — Est. start [date]

**Blockers/Risks:**
- [Blocker if any] — Need [action] from [person/team]

**Office Hours:** Thursdays 2-3pm (drop in for quick questions)

Questions? Reply to this email.

---

**Version:** 1.0  
**Last Updated:** 2025-03-20  
**Owner:** Chief Architect  
**Maintained by:** IT Architecture Team
