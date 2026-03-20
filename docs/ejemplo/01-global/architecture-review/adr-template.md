# ADR-NNNN: [Short Title of Decision]

> **ADR Number:** NNNN *(incremental, e.g., ADR-0001)*  
> **Date:** YYYY-MM-DD  
> **Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX  
> **Deciders:** *(Names of people who made the decision)*  
> **Technical Story:** *(Link to Jira ticket, design doc, or context)*

---

## Context

**What is the problem we're trying to solve?**

> *Describe the issue or need that requires a decision. Include:*
> - *What is the current situation?*
> - *Why is this a problem?*
> - *What constraints exist (budget, timeline, skills, regulations)?*
> - *Who are the stakeholders?*

*Example:*
> We need to select a database for the new plant monitoring system. The system will collect sensor data from 500+ devices, with writes at ~1000 records/second and read queries for dashboards and historical analysis. Current constraint: €50k budget, 3-month timeline, team has PostgreSQL experience but not MongoDB.

---

## Decision Drivers

**What factors influence this decision?**

- *(e.g., Performance: Must handle 1000 writes/sec)*
- *(e.g., Cost: Total cost < €50k CAPEX + €10k/year OPEX)*
- *(e.g., Skills: Team has PostgreSQL experience)*
- *(e.g., Vendor support: Must have 24/7 support available)*
- *(e.g., Compliance: Must support encryption at rest for GDPR)*

---

## Options Considered

### Option 1: [Option Name]

**Description:**
> *(What is this option? How does it work?)*

**Pros:**
- ✅ *(Advantage 1)*
- ✅ *(Advantage 2)*

**Cons:**
- ❌ *(Disadvantage 1)*
- ❌ *(Disadvantage 2)*

**Cost:** €_______ CAPEX, €_______ OPEX/year

**Risk level:** Low | Medium | High

---

### Option 2: [Option Name]

**Description:**
> *(What is this option?)*

**Pros:**
- ✅ *(Advantage 1)*

**Cons:**
- ❌ *(Disadvantage 1)*

**Cost:** €_______ CAPEX, €_______ OPEX/year

**Risk level:** Low | Medium | High

---

### Option 3: [Option Name]

*(Repeat pattern above)*

---

## Decision

**We have decided to: [Chosen Option]**

**Rationale:**
> *Why was this option chosen over the alternatives? Reference the decision drivers.*

*Example:*
> We chose PostgreSQL (Option 1) because it meets our performance requirements (1000 writes/sec achievable with proper indexing), stays within budget (€35k vs €60k for Oracle), and our team already has 3 years of operational experience with it. While MongoDB (Option 2) might have marginally better write performance, the team would require 2 months of training, delaying the project.

---

## Consequences

### Positive Consequences

- *(What benefits do we get from this decision?)*
- *(e.g., Faster time-to-market due to existing skills)*
- *(e.g., Lower operational risk)*

### Negative Consequences

- *(What trade-offs are we accepting?)*
- *(e.g., Relational model might be less flexible for future schema changes)*
- *(e.g., Horizontal scaling will require partitioning strategy)*

### Neutral Consequences

- *(What else changes as a result of this decision?)*
- *(e.g., We will need to purchase additional PostgreSQL monitoring tools)*

---

## Action Items

**What needs to happen to implement this decision?**

- [ ] *(Task 1: e.g., Purchase PostgreSQL Enterprise license)*
- [ ] *(Task 2: e.g., Configure replication to standby server)*
- [ ] *(Task 3: e.g., Train team on advanced indexing)*

**Owner:** *(Name)*  
**Target Date:** YYYY-MM-DD

---

## Validation

**How will we know if this decision was correct?**

- *(Success criterion 1: e.g., System handles 1000 writes/sec with p95 latency < 100ms)*
- *(Success criterion 2: e.g., No database-related incidents in first 6 months)*

**Review date:** YYYY-MM-DD *(typically 6-12 months post-implementation)*

---

## References

**Links to supporting materials:**
- Architecture diagram: *(link)*
- Benchmark results: *(link)*
- Vendor proposals: *(link)*
- Related ADRs: ADR-XXXX, ADR-YYYY

---

## Notes

**Additional context or history:**
> *(Capture informal discussions, assumptions, or anything not covered above)*

---

## Approval

| Role | Name | Date |
|------|------|------|
| **Technical Lead** | | |
| **Architecture Review** | | |
| **Security** (if applicable) | | |

---

## ADR Template Tips

### When to Write an ADR

Write an ADR for decisions that are:
- **Significant:** Affects multiple teams or has long-term impact
- **Controversial:** Multiple valid options, trade-offs between them
- **Irreversible:** Difficult or expensive to change later
- **Architectural:** Impacts system structure, patterns, or standards

### Examples of ADR-Worthy Decisions

✅ **Good candidates for ADRs:**
- Choosing a database (PostgreSQL vs MongoDB)
- Selecting a cloud provider (Azure vs AWS)
- Authentication mechanism (LDAP vs SAML vs OAuth)
- Monolith vs microservices architecture
- Event-driven vs request-response integration

❌ **Not worth an ADR:**
- Naming a variable or function
- Choosing indentation style (code formatting)
- Updating a library version (routine maintenance)

### ADR Lifecycle

1. **Proposed:** Draft ADR, circulate for feedback
2. **Accepted:** Decision finalized, implementation begins
3. **Deprecated:** Decision no longer recommended (but not replaced)
4. **Superseded:** Decision replaced by a newer ADR (link to replacement)

### Storage and Versioning

- Store ADRs in Git repository (version controlled)
- Use sequential numbering (ADR-0001, ADR-0002, ...)
- Use Markdown format (easy to read, diff, and search)
- Keep ADRs immutable once accepted (don't edit history)

### Template Sections Explained

- **Context:** The "why" — what problem are we solving?
- **Decision Drivers:** The "what matters" — criteria for evaluation
- **Options Considered:** The "what else" — alternatives we explored
- **Decision:** The "what" — final choice
- **Consequences:** The "so what" — implications of the choice

---

**ADR Template Version:** 1.0  
**Last Updated:** 2025-03-20  
**More Info:** https://adr.github.io/
