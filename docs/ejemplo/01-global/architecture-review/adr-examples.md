# Architecture Decision Records — Examples

> **Purpose:** This document provides real-world examples of Architecture Decision Records (ADRs) to illustrate best practices.

---

## Example 1: Database Selection for Plant Monitoring

### ADR-0023: PostgreSQL for Plant Sensor Data Storage

**Date:** 2025-01-15  
**Status:** Accepted  
**Deciders:** Maria González (Tech Lead), James Wilson (Architect)  
**Technical Story:** PROJ-456 — Plant Monitoring System

---

#### Context

We are building a new monitoring system for industrial plant sensors that will:

- Collect data from 500+ sensors (temperature, pressure, flow rate)
- Write ~1,000 records/second during peak operation
- Support real-time dashboards (last 24h data)
- Enable historical analysis (5 years retention)
- Budget: €50,000 CAPEX, €10,000/year OPEX
- Team skills: 3 years PostgreSQL experience, no MongoDB experience

---

#### Decision Drivers

- **Performance:** Must handle 1,000 writes/sec with p95 read latency < 100ms
- **Cost:** Total cost < €50k CAPEX + €10k/year OPEX
- **Reliability:** 99.9% uptime (< 9h/year downtime)
- **Skills:** Prefer technologies team already knows
- **Query flexibility:** Need complex analytical queries (joins, aggregations)

---

#### Options Considered

**Option 1: PostgreSQL (Relational DB)**

**Pros:**

- ✅ Team has 3 years operational experience
- ✅ Excellent query flexibility (SQL, joins, window functions)
- ✅ Proven performance at this scale with proper indexing
- ✅ Strong data integrity (ACID transactions)
- ✅ Cost: €35k for HA setup + €8k/year support

**Cons:**

- ❌ Horizontal scaling requires partitioning (more complex)
- ❌ Less flexible schema evolution vs. document DB

**Risk:** Medium

---

**Option 2: MongoDB (Document DB)**

**Pros:**

- ✅ Slightly better write performance (1,500 writes/sec tested)
- ✅ Horizontal sharding built-in
- ✅ Flexible schema (JSON documents)

**Cons:**

- ❌ Team has no MongoDB experience (2-month learning curve)
- ❌ Complex analytical queries harder than SQL
- ❌ Cost: €45k + €12k/year (higher license fees)

**Risk:** High (skill gap)

---

**Option 3: InfluxDB (Time-Series DB)**

**Pros:**

- ✅ Optimized for time-series data
- ✅ Excellent write performance (2,000+ writes/sec)
- ✅ Built-in downsampling and retention policies

**Cons:**

- ❌ Team has no InfluxDB experience
- ❌ Less mature ecosystem vs. PostgreSQL/MongoDB
- ❌ Cost: €55k + €15k/year (enterprise edition for HA)
- ❌ Limited join capabilities (harder for multi-sensor analysis)

**Risk:** High (new technology + cost)

---

#### Decision

**We chose PostgreSQL (Option 1).**

**Rationale:**

PostgreSQL meets our performance requirements (benchmarked at 1,200 writes/sec with proper indexing on similar hardware), stays well within budget (€35k vs €55k for InfluxDB), and leverages our team's existing expertise. While MongoDB and InfluxDB offer marginal performance advantages, the 2-month training delay would push our launch date beyond the Q2 target. 

For historical analysis, SQL's rich query capabilities are essential — data scientists need joins across sensor types, which is cumbersome in MongoDB's aggregation framework.

We will mitigate scaling concerns by:

- Partitioning tables by month (TimescaleDB extension if needed in future)
- Archiving data >1 year to cheaper storage
- Monitoring capacity monthly

---

#### Consequences

**Positive:**

- Team can implement immediately (no training delay)
- Familiar operational runbooks (backups, HA, monitoring)
- Rich ecosystem of tools (pg_stat_statements, pgAdmin, Grafana connector)
- Lower total cost of ownership

**Negative:**

- Horizontal scaling will require partitioning strategy if we exceed 5,000 writes/sec
- Schema changes require migrations (less flexible than document DB)

**Neutral:**

- We will need to purchase PostgreSQL monitoring tools (already budgeted)

---

#### Action Items

- [x] Purchase PostgreSQL Enterprise license — *Maria* — 2025-01-20
- [x] Configure streaming replication to standby — *James* — 2025-01-25
- [ ] Create partitioning plan for sensor_data table — *Maria* — 2025-02-05
- [ ] Train junior dev on advanced indexing — *James* — 2025-02-15

---

#### Validation

**Success criteria:**

- System handles 1,000 writes/sec with p95 latency < 100ms
- Query dashboard response time < 2 seconds for 24h data
- No database-related incidents in first 6 months

**Review date:** 2025-07-15 (6 months post-deployment)

---

#### References

- Benchmark results: https://wiki.company.com/postgres-benchmark-2025
- PostgreSQL HA setup guide: https://docs.company.com/postgres-ha
- Related ADRs: ADR-0012 (Monitoring stack selection)

---

## Example 2: Authentication Mechanism

### ADR-0047: OAuth 2.0 + OIDC for External Partner Integration

**Date:** 2025-02-10  
**Status:** Accepted  
**Deciders:** Sarah Chen (Security Architect), Robert Kumar (Integration Lead)  
**Technical Story:** PROJ-892 — Supply Chain Portal

---

#### Context

We are exposing APIs to external partners (suppliers, logistics providers) who need to:

- Access inventory status and order information
- Submit delivery confirmations and invoices
- Integrate with our ERP (SAP)

**Requirements:**

- Partners manage their own user accounts (we don't want to create/maintain accounts for 200+ partner companies)
- Strong authentication (MFA required)
- Fine-grained authorization (Partner A can't see Partner B's data)
- Audit trail (who accessed what, when)
- Low friction for partners (standard protocols, not custom)

**Constraints:**

- Must integrate with existing Azure AD (corporate identity provider)
- Partners use various identity providers (Google Workspace, Okta, on-premise AD)
- Security team requires MFA for all external access

---

#### Decision Drivers

- **Security:** MFA required, industry-standard protocols
- **Scalability:** Support 200+ partner organizations
- **Partner experience:** Standard protocols they already use
- **Integration:** Works with Azure AD and partner IdPs
- **Compliance:** GDPR (audit logs), ISO 27001

---

#### Options Considered

**Option 1: OAuth 2.0 + OpenID Connect (OIDC)**

**Pros:**

- ✅ Industry standard (partners already familiar)
- ✅ Supports federated identity (partners use their own IdPs)
- ✅ Fine-grained scopes (read-only vs read-write)
- ✅ Native support in Azure AD
- ✅ MFA delegated to partner IdPs

**Cons:**

- ❌ Initial setup complexity (configure trust relationships)
- ❌ Token management (refresh tokens, expiration)

**Cost:** €5k (Azure AD Premium P2 upgrade)  
**Risk:** Low

---

**Option 2: API Keys**

**Pros:**

- ✅ Simple to implement
- ✅ No federated identity complexity

**Cons:**

- ❌ No MFA support
- ❌ Poor security (keys can be leaked, no expiration)
- ❌ No user-level audit trail (all actions appear as "Partner X")
- ❌ Doesn't scale to 200+ partners (key management nightmare)

**Cost:** €0  
**Risk:** High (security team will not approve)

---

**Option 3: SAML 2.0**

**Pros:**

- ✅ Enterprise-grade security
- ✅ Supports federated identity
- ✅ Strong audit capabilities

**Cons:**

- ❌ Older standard (OAuth/OIDC is more modern)
- ❌ XML-based (more complex for developers)
- ❌ Less friendly for API integrations (better for web apps)
- ❌ Partners may not have SAML IdPs (smaller suppliers)

**Cost:** €5k (same Azure AD upgrade)  
**Risk:** Medium (complexity)

---

#### Decision

**We chose OAuth 2.0 + OpenID Connect (Option 1).**

**Rationale:**

OAuth 2.0 + OIDC is the industry standard for API authentication, which means:

- Partners' developers are already familiar with it
- Extensive tooling and libraries available (reduces integration friction)
- Native support in Azure AD (no custom development)

It meets our security requirements:

- MFA enforced by partner IdPs (we verify MFA claim in token)
- User-level audit trail (token contains partner company + individual user)
- Scopes enable fine-grained permissions

While API keys (Option 2) are simpler, they fail basic security requirements (no MFA, no user-level audit). SAML (Option 3) is enterprise-grade but overkill for API integrations — it's designed for web SSO, not programmatic access.

---

#### Consequences

**Positive:**

- Industry-standard approach (well-documented, proven)
- Partners can reuse existing integrations (many already use OAuth for Google/Microsoft APIs)
- Centralized identity management (Azure AD)
- Fine-grained access control (scopes per partner)

**Negative:**

- Initial configuration overhead (set up trust with each partner IdP)
- Token refresh logic required in partner applications
- Onboarding takes ~1 week per partner (vs instant for API keys)

**Neutral:**

- Need to document OAuth flow for partners (create developer portal)
- Token expiration requires refresh tokens (standard practice)

---

#### Action Items

- [x] Upgrade Azure AD to Premium P2 — *Sarah* — 2025-02-15
- [ ] Configure OAuth app registration in Azure AD — *Robert* — 2025-02-20
- [ ] Create developer portal with OAuth integration guide — *Robert* — 2025-03-01
- [ ] Pilot with 3 key partners — *Robert* — 2025-03-15
- [ ] MFA enforcement policy in Azure AD — *Sarah* — 2025-02-25

---

#### Validation

**Success criteria:**

- 100% of partners use MFA (validated via token claims)
- Partner integration time < 2 weeks (documented, tested flow)
- Zero security incidents related to authentication
- Audit logs capture user-level actions (not just company-level)

**Review date:** 2025-08-10 (6 months after rollout)

---

#### References

- OAuth 2.0 RFC: https://tools.ietf.org/html/rfc6749
- Azure AD OAuth guide: https://docs.microsoft.com/azure/active-directory/develop/
- Related ADRs: ADR-0032 (API Gateway selection)

---

## Example 3: Cloud vs On-Premise

### ADR-0061: Azure Blob Storage for Long-Term Backup Archival

**Date:** 2025-03-01  
**Status:** Accepted  
**Deciders:** Hans Mueller (Infrastructure Lead), Lisa Park (Architect)  
**Technical Story:** PROJ-1203 — Backup Modernization

---

#### Context

Our current backup retention strategy:

- Primary: 30 days on local NAS (7 TB per plant)
- Secondary: 90 days on tape (LTO-8)
- No tertiary/off-site copy (regulatory risk for Platinum data)

**Problem:**

- SOX compliance requires 7-year retention for financial data (SAP backups)
- Current tape approach: €15k/year per plant for tapes + off-site storage
- Tape restores are slow (4-24 hours) and error-prone (5% failure rate)
- 30 plants = €450k/year total

**Opportunity:**

- Azure Archive tier: €0.001/GB/month (~90% cheaper than tape)
- Instant retrieval vs tape recall (4-24h → 15h rehydration)
- No physical media management

**Constraints:**

- Data must stay in EU (GDPR)
- Must use ExpressRoute (no public Internet)
- Must integrate with existing Veeam backup solution

---

#### Decision Drivers

- **Cost:** Reduce €450k/year tape costs
- **Compliance:** Meet 7-year SOX retention
- **Reliability:** Improve on 95% tape restore success rate
- **Recovery speed:** Faster than tape (< 24h for RTO)
- **Security:** GDPR compliance, EU data residency

---

#### Options Considered

**Option 1: Azure Blob Archive Tier**

**Pros:**

- ✅ Low cost: €0.001/GB/month (vs €0.015/GB for tape)
- ✅ Reliable: 99.999999999% (11 nines) durability
- ✅ GDPR compliant (EU regions available)
- ✅ Integrates with Veeam (native support)
- ✅ No physical media management

**Cons:**

- ❌ Rehydration time: ~15 hours (vs immediate for disk)
- ❌ Egress fees: €0.018/GB for restores (rarely needed)
- ❌ Requires ExpressRoute: €12k/year (shared across plants)

**5-year TCO:** €245k (storage + ExpressRoute)  
**Risk:** Low

---

**Option 2: Continue with Tape (LTO-9)**

**Pros:**

- ✅ Known process (team has 10 years experience)
- ✅ No cloud dependency
- ✅ No egress fees

**Cons:**

- ❌ High cost: €450k/year × 5 = €2.25M
- ❌ Tape failure rate: 5% (unreliable)
- ❌ Slow restores: 4-24 hours
- ❌ Physical media management (courier, off-site vault)

**5-year TCO:** €2.25M  
**Risk:** Medium (tape reliability)

---

**Option 3: Build Second Datacenter for Replication**

**Pros:**

- ✅ Fast recovery (disk-based)
- ✅ No cloud dependency

**Cons:**

- ❌ Extremely high cost: €500k CAPEX + €100k/year OPEX = €1M (5-year)
- ❌ Requires staff at second site
- ❌ Complex network setup

**5-year TCO:** €1M  
**Risk:** High (complexity, cost)

---

#### Decision

**We chose Azure Blob Archive (Option 1).**

**Rationale:**

Azure Archive offers a 90% cost reduction (€245k vs €2.25M over 5 years) while improving reliability (11 nines durability vs 95% tape success rate). The rehydration time (15 hours) is acceptable because:

- Archive tier is for >1 year old data (rarely restored)
- For recovery, we restore from primary (30 days) or secondary (90 days) first
- 15h meets our Bronze tier RTO (< 2 weeks)

GDPR compliance is met via EU regions (West Europe, North Europe). ExpressRoute ensures private connectivity (no public Internet). Veeam's native Azure integration means minimal operational changes.

Second datacenter (Option 3) is overkill — we don't need fast recovery for archival data. Tape (Option 2) is familiar but fails cost and reliability requirements.

---

#### Consequences

**Positive:**

- €2M savings over 5 years (vs tape)
- Improved restore reliability (99.999999999% vs 95%)
- No physical media management (tapes, courier, vault)
- Scales automatically (no capacity planning)

**Negative:**

- 15-hour rehydration for archive restores (acceptable for archival)
- Egress fees if we restore large amounts (€0.018/GB)
- Cloud dependency (mitigated: Express Route + Azure SLA 99.9%)

**Neutral:**

- Need to configure lifecycle policies (auto-move to Archive after 90 days)
- ExpressRoute required (€12k/year, shared across all Azure services)

---

#### Action Items

- [x] Procure ExpressRoute circuit — *Hans* — 2025-03-10
- [ ] Configure Veeam Azure integration — *Hans* — 2025-03-15
- [ ] Test backup→Azure→restore workflow — *Hans* — 2025-03-20
- [ ] Pilot with 2 plants (EU01, EU02) — *Hans* — 2025-04-01
- [ ] Create runbook for archive restores — *Lisa* — 2025-04-05
- [ ] Rollout to remaining 28 plants — *Hans* — 2025-05-31

---

#### Validation

**Success criteria:**

- Total cost < €50k/year (storage + ExpressRoute allocation)
- Restore success rate > 99% (vs 95% for tape)
- Rehydration time < 20 hours (actual vs 15h target)
- Zero GDPR violations (data stays in EU)

**Review date:** 2025-09-01 (6 months after full rollout)

---

#### References

- Azure pricing calculator: https://azure.microsoft.com/pricing/calculator/
- Veeam Azure integration: https://helpcenter.veeam.com/docs/backup/vsphere/azure_blob_repository.html
- Related ADRs: ADR-0023 (Backup retention policies)

---

## Lessons Learned from These Examples

### What Makes a Good ADR?

1. **Clear context:** Explains the problem and constraints upfront
2. **Objective criteria:** Decision drivers are measurable, not subjective
3. **Honest trade-offs:** Acknowledges cons of chosen option
4. **Evidence-based:** Uses data (benchmarks, costs, experience)
5. **Actionable:** Includes concrete next steps with owners

### Common Mistakes to Avoid

❌ **No alternatives:** Only documenting the chosen option  
❌ **Vague reasoning:** "We chose X because it's better" (better how?)  
❌ **Missing consequences:** Only listing pros, ignoring trade-offs  
❌ **No validation:** How will you know if the decision was right?  
❌ **Overly long:** 10-page ADRs that nobody reads (keep it 1-2 pages)

### When to Write an ADR

✅ **Significant decisions:** Will impact multiple teams or be hard to change  
✅ **Controversial choices:** Multiple valid options, team needs alignment  
✅ **Future reference:** "Why did we do it this way?" will be asked later  

❌ **Routine choices:** Picking a logging library (unless novel)  
❌ **Reversible decisions:** Easy to change later (not architectural)

---

**Document Version:** 1.0  
**Last Updated:** 2025-03-20  
**More Examples:** https://architecture.company.com/adr-examples/
