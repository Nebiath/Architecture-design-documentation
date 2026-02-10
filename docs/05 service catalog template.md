# Service Catalog Entry - [Nombre del Servicio]

> **Propósito:** Registro centralizado para descubrimiento de servicios  
> **Última Actualización:** YYYY-MM-DD

---

## 1. Basic Information

| Field | Value |
|-------|-------|
| **Service Name** | service-name |
| **Display Name** | [Service Name] |
| **Service ID** | srv-12345 |
| **Version** | 1.2.3 |
| **Status** | ✅ Active / ⚠️ Deprecated / 🔴 Sunset |
| **Tier** | Tier 1 (Critical) / Tier 2 / Tier 3 |
| **Environment** | Production |
| **Deployment Date** | 2023-01-15 |

---

## 2. Ownership

| Role | Name | Contact |
|------|------|---------|
| **Service Owner** | [Name] | owner@company.com |
| **Tech Lead** | [Name] | tech-lead@company.com |
| **Team** | Platform Engineering | @platform-team |
| **On-Call** | Rotation | pagerduty.com/schedules/ABC123 |

---

## 3. Service Characteristics

### Type
- [ ] API/Microservice
- [ ] Web Application
- [ ] Background Worker
- [ ] Data Pipeline
- [ ] Library/SDK

### Technology Stack
| Component | Technology |
|-----------|------------|
| Language | Python 3.11 |
| Framework | FastAPI |
| Database | PostgreSQL 14 |
| Cache | Redis 7 |
| Message Queue | RabbitMQ |

### Deployment
- **Platform:** Kubernetes (EKS)
- **Namespace:** production
- **Replicas:** 3-10 (auto-scaling)
- **Region:** eu-west-1, us-east-1

---

## 4. Service Level Objectives

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Availability | 99.9% | 99.95% | ✅ |
| Latency (p95) | < 300ms | 250ms | ✅ |
| Error Rate | < 0.1% | 0.05% | ✅ |

**Error Budget (30d):** 78% remaining

---

## 5. Dependencies

### Upstream (We Depend On)
| Service | Type | Criticality | Fallback |
|---------|------|-------------|----------|
| auth-service | Sync API | Critical | Circuit breaker |
| payment-gateway | Sync API | Critical | Retry queue |
| email-service | Async queue | Medium | Best effort |

### Downstream (Depends On Us)
| Service | Owner | Impact if Down |
|---------|-------|----------------|
| mobile-app | mobile-team@ | Users can't login |
| web-app | frontend-team@ | Limited functionality |
| analytics | data-team@ | No real-time data |

---

## 6. Data & Compliance

### Data Classification
- [ ] Public
- [ ] Internal
- [x] Confidential (PII)
- [ ] Restricted (Financial/Health)

### Compliance Tags
- [x] GDPR
- [x] SOC2
- [ ] HIPAA
- [ ] PCI-DSS

### Data Residency
- EU data: Stored in eu-west-1 only
- US data: Stored in us-east-1 only
- No cross-region replication without consent

---

## 7. APIs & Integrations

### Public APIs
| Endpoint | Method | Auth | Rate Limit | Docs |
|----------|--------|------|------------|------|
| /api/v1/users | GET | OAuth2 | 1000/min | [Swagger](https://api.company.com/docs) |
| /api/v1/orders | POST | API Key | 100/min | [Swagger](https://api.company.com/docs) |

### Internal APIs
| Endpoint | Consumers | Protocol |
|----------|-----------|----------|
| /internal/metrics | Monitoring | HTTP |
| /internal/health | Load Balancer | HTTP |

### Events Published
| Event | Topic/Queue | Schema | Consumers |
|-------|-------------|--------|-----------|
| user.created | user-events | v1.json | analytics, email |
| order.completed | order-events | v1.json | billing, shipping |

---

## 8. Resources & Links

### Code & Docs
- **Git Repository:** https://github.com/company/service-name
- **API Documentation:** https://api.company.com/docs/service-name
- **Architecture Doc:** [Link](./01-architecture-design.md)
- **Runbooks:** [Link](./02-deployment-runbook.md)

### Operations
- **CI/CD Pipeline:** https://jenkins.company.com/job/service-name
- **Grafana Dashboard:** https://grafana.company.com/d/service-name
- **Datadog APM:** https://app.datadoghq.com/apm/services/service-name
- **Splunk Logs:** https://splunk.company.com/app/search?q=service%3Aservice-name
- **PagerDuty Service:** https://company.pagerduty.com/services/ABC123

### Infrastructure
- **AWS Account:** prod-account (123456789012)
- **Kubernetes Cluster:** prod-cluster
- **Database:** prod-db.cluster-abc123.eu-west-1.rds.amazonaws.com
- **Redis Cluster:** prod-redis.abc123.cache.amazonaws.com

---

## 9. Costs

### Monthly Breakdown
| Resource | Cost (USD) |
|----------|------------|
| Compute (EC2/EKS) | $1,200 |
| Database (RDS) | $800 |
| Cache (ElastiCache) | $200 |
| Storage (S3/EBS) | $150 |
| Data Transfer | $450 |
| Monitoring (Datadog) | $200 |
| **Total** | **$3,000** |

**Cost per Request:** $0.0001  
**Cost per User/Month:** $0.50

---

## 10. Security & Access

### Authentication
- **Method:** OAuth 2.0 + API Keys
- **Provider:** Auth0
- **Token Expiry:** 1 hour (access), 30 days (refresh)

### Network Access
- **Public Endpoints:** via CloudFront CDN
- **Internal Endpoints:** VPC only
- **Database:** Private subnet, security group restricted

### Secrets Management
- **Tool:** AWS Secrets Manager
- **Rotation:** Automatic every 90 days
- **Access:** IAM role-based

---

## 11. Disaster Recovery

| Metric | Target | Last Tested |
|--------|--------|-------------|
| **RTO** | < 1 hour | 2024-02-01 |
| **RPO** | < 5 minutes | 2024-02-01 |
| **Backup Frequency** | Continuous | Verified daily |
| **DR Region** | us-east-1 | Tested quarterly |

---

## 12. Change Management

### Deployment Windows
- **Allowed:** Mon-Thu 10:00-16:00 UTC
- **Blackout:** Fridays, weekends, holidays
- **Emergency:** Any time with approval

### Approval Requirements
| Change Type | Approval Required |
|-------------|-------------------|
| Code deployment | Tech Lead (auto via CI/CD) |
| Database migration | Service Owner + DBA |
| Infrastructure change | Service Owner + Cloud Team |
| Breaking API change | Product Owner + all consumers |

---

## 13. Metrics & KPIs

### Business Metrics
- **Active Users:** 50,000/month
- **Transactions:** 1M/month
- **Revenue Impact:** $100k/month direct attribution

### Technical Metrics
- **Deployment Frequency:** 10/week
- **MTTR:** 30 minutes (avg)
- **Change Failure Rate:** 2%
- **Lead Time:** 2 hours (commit to production)

---

## 14. Lifecycle

### Planned Changes
| Date | Change | Impact |
|------|--------|--------|
| 2024-Q2 | Migrate to Kafka | Improved event streaming |
| 2024-Q3 | API v2 release | Deprecate v1 endpoints |

### Deprecation Plan
- **Status:** Active (no planned deprecation)
- **End of Life:** N/A
- **Migration Path:** N/A

---

## 15. Contact & Support

### Primary Contacts
- **Slack Channel:** #service-name-team
- **Email:** service-name-team@company.com
- **Office Hours:** Mon-Fri 09:00-17:00 UTC
- **On-Call:** 24/7 via PagerDuty

### Getting Help
1. Check documentation: https://wiki.company.com/service-name
2. Ask in #service-name-team Slack
3. For incidents: Page on-call via PagerDuty
4. For feature requests: Create Jira ticket

---

## 16. Tags & Metadata

```yaml
tags:
  team: platform-engineering
  cost-center: engineering
  environment: production
  compliance: [gdpr, soc2]
  tier: tier-1
  language: python
  framework: fastapi
  data-classification: confidential
```

---

**This entry should be updated:**
- On major version releases
- When ownership changes
- Quarterly compliance review
- After significant architecture changes

**Last Review:** YYYY-MM-DD  
**Next Review:** YYYY-MM-DD  
**Reviewed By:** [Service Owner]
