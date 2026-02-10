# [Nombre del Servicio] - Observability & Monitoring

> **Version:** 1.0  
> **Fecha:** YYYY-MM-DD  
> **Service Owner:** [Nombre]  
> **SRE Lead:** [Nombre]  
> **Última Revisión:** YYYY-MM-DD

---

## 1. Observability Overview

**Los Tres Pilares:**
1. **Metrics:** Valores numéricos agregados (Prometheus, Datadog)
2. **Logs:** Eventos discretos (ELK, Splunk)
3. **Traces:** Request flow end-to-end (Jaeger, Zipkin)

**Herramientas en uso:**

| Pilar | Herramienta | URL | Acceso |
|-------|-------------|-----|--------|
| Metrics | Datadog | https://app.datadoghq.com | SSO |
| Logs | Splunk | https://splunk.company.com | SSO |
| Traces | Datadog APM | https://app.datadoghq.com/apm | SSO |
| Dashboards | Grafana | https://grafana.company.com | SSO |
| Alerts | PagerDuty | https://company.pagerduty.com | SSO |

---

## 2. Service Level Indicators (SLIs)

### 2.1 Availability SLI

**Definición:** Porcentaje de requests exitosos (HTTP 200-299, 400-499 excluidos)

**Fórmula:**
```
SLI = (successful_requests / total_requests) * 100
```

**Fuente de datos:**
```
Datadog Query:
sum:aws.elb.request_count{service:service-name,http.status_code:2*}
÷ sum:aws.elb.request_count{service:service-name}
```

**Target:** 99.9%

**Medición:** Ventana rolling de 30 días

**Error Budget:** 43.2 minutos/mes

### 2.2 Latency SLI

**Definición:** Porcentaje de requests que completan en < 300ms (p95)

**Fórmula:**
```
SLI = (requests_under_300ms / total_requests) * 100
```

**Fuente de datos:**
```
Datadog Query:
percentile(request_duration_ms, 95) < 300
```

**Targets:**
- p50: < 100ms
- p95: < 300ms
- p99: < 500ms

**Medición:** Ventana rolling de 30 días

### 2.3 Error Rate SLI

**Definición:** Porcentaje de requests que resultan en error 5xx

**Fórmula:**
```
SLI = (1 - (5xx_errors / total_requests)) * 100
```

**Fuente de datos:**
```
Datadog Query:
sum:aws.elb.httpcode_target_5xx{service:service-name}
÷ sum:aws.elb.request_count{service:service-name}
```

**Target:** < 0.1% (99.9% success rate)

**Medición:** Ventana rolling de 30 días

### 2.4 Data Freshness SLI

**Definición:** Tiempo máximo entre actualización de datos

**Fórmula:**
```
SLI = percentage_of_records_updated_within_5min
```

**Fuente de datos:**
```
SELECT 
  COUNT(*) FILTER (WHERE updated_at > NOW() - INTERVAL '5 minutes') * 100.0 / COUNT(*)
FROM critical_data_table
```

**Target:** > 99% de records actualizados en < 5 minutos

**Medición:** Continua

---

## 3. Service Level Objectives (SLOs)

### 3.1 SLO Summary Table

| SLO | SLI | Target | Window | Current | Error Budget | Status |
|-----|-----|--------|--------|---------|--------------|--------|
| Availability | Request success rate | 99.9% | 30d | 99.95% | 78% remaining | ✅ |
| Latency | p95 < 300ms | 95% | 30d | 97% | 40% remaining | ✅ |
| Error Rate | 5xx errors | < 0.1% | 30d | 0.05% | 50% remaining | ✅ |
| Data Freshness | Records < 5min old | 99% | 24h | 99.8% | 80% remaining | ✅ |

### 3.2 Error Budget Policy

**Cálculo del Error Budget:**
```
Error Budget = (1 - SLO) × Time Window
```

**Para Availability SLO de 99.9% en 30 días:**
```
Error Budget = (1 - 0.999) × 43,200 min = 43.2 min/mes
```

**Error Budget Policies:**

| Error Budget Restante | Acciones |
|-----------------------|----------|
| > 50% | ✅ Normal operations - Deploy anytime |
| 25-50% | ⚠️ Caution - Reduce deployment frequency, focus on reliability |
| 10-25% | 🔶 Alert - Freeze non-critical deployments, investigate issues |
| < 10% | 🚨 Critical - Feature freeze, all hands on reliability |

**Burn Rate Alerts:**

| Burn Rate | Time to Exhaustion | Alert Action |
|-----------|-------------------|--------------|
| 1x | 30 días | No alert |
| 2x | 15 días | Warning |
| 5x | 6 días | Page on-call |
| 10x | 3 días | Page senior on-call |
| 20x | 36 horas | Page incident commander |

---

## 4. Golden Signals

### 4.1 Latency

**Definición:** Tiempo que toma procesar un request

**Métricas Clave:**

```yaml
# Datadog metrics
- metric: request.duration.p50
  threshold: 100ms
  
- metric: request.duration.p95
  threshold: 300ms
  
- metric: request.duration.p99
  threshold: 500ms
  
- metric: request.duration.max
  threshold: 2000ms
```

**Desglose por endpoint:**
```
GET /api/users - p95: 120ms
POST /api/orders - p95: 250ms
GET /api/products - p95: 80ms
```

**Dashboard:** https://grafana.company.com/d/latency-service-name

### 4.2 Traffic

**Definición:** Demanda en el sistema (requests/segundo)

**Métricas Clave:**

```yaml
- metric: requests_per_second
  normal_range: 100-500 req/s
  peak_range: 800-1200 req/s
  
- metric: active_connections
  normal_range: 500-2000
  
- metric: bandwidth_usage
  normal_range: 50-200 Mbps
```

**Patrones de tráfico:**
- Lunes-Viernes: 8:00-18:00 UTC → Peak traffic
- Fines de semana: 50% del tráfico normal
- Eventos especiales: 3x traffic normal

**Dashboard:** https://grafana.company.com/d/traffic-service-name

### 4.3 Errors

**Definición:** Tasa de requests fallidos

**Métricas Clave:**

```yaml
# HTTP Errors
- metric: http_errors_4xx
  by: status_code
  threshold: 5% of total requests
  
- metric: http_errors_5xx
  by: status_code
  threshold: 0.1% of total requests

# Application Errors
- metric: exception_count
  by: exception_type
  alert_threshold: > 10/min
  
- metric: database_errors
  alert_threshold: > 5/min
```

**Error Types:**
```
500 - Internal Server Error: < 0.05%
502 - Bad Gateway: < 0.01%
503 - Service Unavailable: < 0.02%
504 - Gateway Timeout: < 0.02%
```

**Dashboard:** https://grafana.company.com/d/errors-service-name

### 4.4 Saturation

**Definición:** Utilización de recursos del sistema

**Métricas Clave:**

```yaml
# Compute
- metric: cpu_utilization
  threshold: 70%
  critical: 85%
  
- metric: memory_utilization
  threshold: 75%
  critical: 90%

# Database
- metric: database_connections
  max: 100
  threshold: 80
  
- metric: database_iops
  max: 10000
  threshold: 8000

# Network
- metric: network_bandwidth
  max: 10 Gbps
  threshold: 7 Gbps

# Disk
- metric: disk_utilization
  threshold: 70%
  critical: 85%
  
- metric: disk_iops
  max: 16000
  threshold: 12000
```

**Dashboard:** https://grafana.company.com/d/saturation-service-name

---

## 5. Dashboards

### 5.1 Service Overview Dashboard

**URL:** https://grafana.company.com/d/overview-service-name

**Widgets:**

```yaml
Row 1: SLOs
- Availability (30d rolling)
- Latency p95 (30d rolling)
- Error Rate (30d rolling)
- Error Budget Remaining

Row 2: Golden Signals
- Request Rate (1h)
- Latency Percentiles (1h)
- Error Rate by Status Code (1h)
- CPU/Memory Utilization (1h)

Row 3: Business Metrics
- Active Users
- Successful Transactions
- Revenue Impact
- Geographic Distribution

Row 4: Dependencies
- Database Response Time
- Cache Hit Rate
- External API Latency
- Message Queue Depth
```

### 5.2 Detailed Metrics Dashboard

**URL:** https://grafana.company.com/d/detailed-service-name

**Sections:**

1. **Application Metrics**
   - Request rate by endpoint
   - Response times by endpoint
   - Error rates by endpoint
   - Top slowest endpoints

2. **Infrastructure Metrics**
   - Pod/Container health
   - Node resources
   - Network I/O
   - Storage IOPS

3. **Database Metrics**
   - Query performance
   - Connection pool usage
   - Slow queries
   - Replication lag

4. **Cache Metrics**
   - Hit/miss ratio
   - Eviction rate
   - Memory usage
   - Connection count

### 5.3 On-Call Dashboard

**URL:** https://grafana.company.com/d/oncall-service-name

**Designed for incident response:**

```yaml
- Current active alerts
- Error rate (last 4h)
- Latency spike detection
- Traffic anomalies
- Failed deployments
- Recent changes
- Upstream dependency status
- Logs panel (last 100 errors)
```

**Auto-refresh:** Every 30 seconds

---

## 6. Alerting

### 6.1 Alert Philosophy

**Principles:**
- ✅ Every alert must be actionable
- ✅ Alert on symptoms, not causes
- ✅ Use multi-window/multi-burn-rate alerts
- ✅ Avoid alert fatigue
- ✅ Every alert has a runbook

**Alert Severities:**

| Severity | Response Time | Examples |
|----------|---------------|----------|
| P0 (Critical) | 15 min | Service down, data loss |
| P1 (High) | 30 min | SLO at risk, high error rate |
| P2 (Medium) | 2 hours | Performance degradation |
| P3 (Low) | Next business day | Warnings, capacity alerts |

### 6.2 SLO-Based Alerts

**Availability Alert:**

```yaml
alert: HighErrorBudgetBurn
expr: |
  (
    error_budget_remaining{service="service-name"} < 0.1
  )
severity: P0
annotations:
  summary: "Error budget critically low ({{ $value }}% remaining)"
  runbook: https://wiki.company.com/runbooks/error-budget-burn
  description: |
    Only {{ $value }}% of error budget remains for this 30-day window.
    Immediate action required to prevent SLO breach.
labels:
  service: service-name
  team: platform
actions:
  - Page on-call
  - Freeze deployments
  - Create incident
```

**Multi-Burn Rate Alert:**

```yaml
# Fast burn (2% budget in 1h)
alert: FastErrorBudgetBurn
expr: |
  (
    (1 - sli_availability) / slo_availability > 0.02
  ) AND
  (
    (1 - sli_availability:1h) / slo_availability > 0.02
  )
severity: P0
for: 5m

# Slow burn (10% budget in 6h)
alert: SlowErrorBudgetBurn
expr: |
  (
    (1 - sli_availability:6h) / slo_availability > 0.10
  )
severity: P1
for: 30m
```

### 6.3 Golden Signal Alerts

**Latency Alerts:**

```yaml
# P95 latency exceeds SLO
alert: HighLatencyP95
expr: |
  histogram_quantile(0.95, 
    rate(request_duration_bucket[5m])
  ) > 0.3
severity: P1
for: 10m
annotations:
  summary: "p95 latency is {{ $value }}s (threshold: 0.3s)"
  runbook: https://wiki.company.com/runbooks/high-latency
actions:
  - Page on-call
  - Check database performance
  - Check external dependencies
```

**Traffic Alerts:**

```yaml
# Unusual traffic spike
alert: TrafficSpike
expr: |
  rate(requests_total[5m]) > 
  avg_over_time(rate(requests_total[5m])[1h:5m]) * 2
severity: P2
for: 5m
annotations:
  summary: "Traffic spike detected: {{ $value }} req/s"
  description: "Traffic is 2x normal levels"
  
# Traffic drop (possible issue)
alert: TrafficDrop
expr: |
  rate(requests_total[5m]) < 
  avg_over_time(rate(requests_total[5m])[1h:5m]) * 0.3
severity: P1
for: 5m
annotations:
  summary: "Traffic drop: {{ $value }} req/s (possible outage)"
```

**Error Rate Alerts:**

```yaml
# High 5xx error rate
alert: High5xxErrors
expr: |
  (
    sum(rate(http_requests_total{status=~"5.."}[5m]))
    /
    sum(rate(http_requests_total[5m]))
  ) > 0.01
severity: P0
for: 2m
annotations:
  summary: "5xx error rate is {{ $value | humanizePercentage }}"
  runbook: https://wiki.company.com/runbooks/5xx-errors

# High 4xx error rate (possible client issue or attack)
alert: High4xxErrors
expr: |
  (
    sum(rate(http_requests_total{status=~"4.."}[5m]))
    /
    sum(rate(http_requests_total[5m]))
  ) > 0.10
severity: P2
for: 10m
annotations:
  summary: "4xx error rate is {{ $value | humanizePercentage }}"
  description: "Possible client issues or API abuse"
```

**Saturation Alerts:**

```yaml
# High CPU usage
alert: HighCPU
expr: |
  avg(cpu_usage_percent{service="service-name"}) > 85
severity: P1
for: 15m
annotations:
  summary: "CPU usage is {{ $value }}%"
  runbook: https://wiki.company.com/runbooks/high-cpu
actions:
  - Auto-scale if possible
  - Page on-call if sustained

# Memory pressure
alert: HighMemory
expr: |
  avg(memory_usage_percent{service="service-name"}) > 90
severity: P0
for: 5m
annotations:
  summary: "Memory usage is {{ $value }}%"
  description: "Risk of OOM kills"

# Database connection pool exhaustion
alert: DatabaseConnectionPoolExhausted
expr: |
  (
    database_connections_active 
    / 
    database_connections_max
  ) > 0.90
severity: P1
for: 5m
annotations:
  summary: "Database connection pool at {{ $value | humanizePercentage }}"
```

### 6.4 Dependency Alerts

```yaml
# External API degradation
alert: ExternalAPIHighLatency
expr: |
  histogram_quantile(0.95,
    rate(external_api_duration_bucket{api="payment-gateway"}[5m])
  ) > 2
severity: P2
for: 10m
annotations:
  summary: "Payment gateway p95 latency: {{ $value }}s"
  description: "External dependency degraded"
  
# Database replication lag
alert: DatabaseReplicationLag
expr: |
  database_replication_lag_seconds > 60
severity: P1
for: 5m
annotations:
  summary: "DB replication lag: {{ $value }}s"
  description: "Read replicas may have stale data"
```

### 6.5 Alert Routing

**PagerDuty Integration:**

```yaml
# Routing rules
routes:
  - match:
      severity: P0
    receiver: pagerduty-critical
    continue: false
    
  - match:
      severity: P1
    receiver: pagerduty-high
    continue: false
    
  - match:
      severity: P2
    receiver: slack-alerts
    continue: true
    
  - match:
      severity: P3
    receiver: slack-warnings
    continue: true

receivers:
  - name: pagerduty-critical
    pagerduty_configs:
      - service_key: xxx
        severity: critical
        
  - name: slack-alerts
    slack_configs:
      - channel: '#alerts-service-name'
        title: "{{ .GroupLabels.alertname }}"
```

---

## 7. Logging

### 7.1 Log Levels

**Standard Levels:**

| Level | When to Use | Examples |
|-------|-------------|----------|
| FATAL | Application can't continue | Database unreachable, config missing |
| ERROR | Request failed | API error, validation failure |
| WARN | Degraded but functional | Slow query, retry attempt |
| INFO | Normal operations | Request started, cache hit |
| DEBUG | Development/troubleshooting | Variable values, function calls |
| TRACE | Very detailed debugging | Loop iterations, low-level calls |

**Production Log Level:** INFO (DEBUG for specific components if needed)

### 7.2 Structured Logging

**Format:** JSON

**Required Fields:**

```json
{
  "timestamp": "2024-03-15T10:30:45.123Z",
  "level": "ERROR",
  "service": "service-name",
  "version": "1.2.3",
  "environment": "production",
  "trace_id": "abc123def456",
  "span_id": "span789",
  "user_id": "user_12345",
  "request_id": "req_xyz789",
  "message": "Database query failed",
  "error": {
    "type": "DatabaseConnectionError",
    "message": "Connection timeout after 30s",
    "stack_trace": "..."
  },
  "metadata": {
    "query": "SELECT * FROM users WHERE id = ?",
    "duration_ms": 30000
  }
}
```

**Example Log Statements:**

```python
# Python example
logger.info(
    "User login successful",
    extra={
        "user_id": user.id,
        "login_method": "oauth",
        "ip_address": request.ip,
        "duration_ms": 150
    }
)

logger.error(
    "Payment processing failed",
    extra={
        "order_id": order.id,
        "amount": order.total,
        "payment_gateway": "stripe",
        "error_code": "card_declined"
    },
    exc_info=True  # Include stack trace
)
```

### 7.3 Log Retention

| Environment | Retention Period | Storage | Cost/Month |
|-------------|------------------|---------|------------|
| Production | 90 days (hot), 1 year (cold) | Splunk + S3 | $5,000 |
| Staging | 30 days | Splunk | $800 |
| Development | 7 days | CloudWatch | $100 |

**Compliance:**
- Audit logs: 7 years (regulatory requirement)
- Security logs: 1 year
- Access logs: 90 days

### 7.4 Log Aggregation

**Splunk Queries:**

```spl
# Find all errors in last hour
index=production service=service-name level=ERROR
| timechart count by error.type

# Slow requests (> 1s)
index=production service=service-name
| where duration_ms > 1000
| stats avg(duration_ms) as avg_duration by endpoint
| sort -avg_duration

# User activity
index=production service=service-name user_id=*
| transaction user_id maxspan=30m
| table _time user_id action endpoint status_code

# Error rate trend
index=production service=service-name
| timechart span=5m count(eval(level="ERROR")) as errors, count as total
| eval error_rate = (errors/total)*100
```

---

## 8. Distributed Tracing

### 8.1 Tracing Configuration

**Tool:** Datadog APM / OpenTelemetry

**Sampling Rate:**
- Production: 10% of requests (to control costs)
- Staging: 100% of requests
- Errors: Always sampled

**Instrumentation:**

```python
# Auto-instrumentation (preferred)
from ddtrace import patch_all
patch_all()

# Manual spans for critical paths
from ddtrace import tracer

@tracer.wrap(service="service-name", resource="process_order")
def process_order(order_id):
    with tracer.trace("validate_order") as span:
        span.set_tag("order.id", order_id)
        # validation logic
        
    with tracer.trace("charge_payment") as span:
        span.set_tag("payment.gateway", "stripe")
        # payment logic
        
    with tracer.trace("send_confirmation") as span:
        # email logic
```

### 8.2 Trace Analysis

**Common Queries:**

```
# Slowest endpoints
Service: service-name
Duration: p95 > 500ms
Group by: endpoint
Time: Last 1 hour

# Error traces
Service: service-name
Status: Error
Group by: error.type
Time: Last 24 hours

# Database query performance
Service: service-name
Resource: postgres.query
Duration: p95 > 100ms
Time: Last 1 hour
```

**Dashboard:** https://app.datadoghq.com/apm/services/service-name

---

## 9. Synthetic Monitoring

### 9.1 Health Check Endpoints

**Basic Health Check:**

```
GET /health
Response: 200 OK
{
  "status": "healthy",
  "timestamp": "2024-03-15T10:30:45Z",
  "version": "1.2.3"
}
```

**Detailed Health Check:**

```
GET /health/detailed
Response: 200 OK (or 503 if any dependency unhealthy)
{
  "status": "healthy",
  "checks": {
    "database": {
      "status": "healthy",
      "latency_ms": 12,
      "connections": 45
    },
    "cache": {
      "status": "healthy",
      "latency_ms": 2,
      "hit_rate": 0.95
    },
    "external_api": {
      "status": "degraded",
      "latency_ms": 1500,
      "error_rate": 0.02
    }
  }
}
```

### 9.2 Synthetic Tests

**Datadog Synthetics:**

| Test Name | Type | Frequency | Locations | Endpoint |
|-----------|------|-----------|-----------|----------|
| Homepage Load | Browser | 5 min | 5 global | https://company.com |
| API Health | API | 1 min | 3 regional | https://api.company.com/health |
| User Login Flow | Browser | 10 min | 3 regional | https://app.company.com/login |
| Checkout Flow | Browser | 15 min | 5 global | https://app.company.com/checkout |

**Alert Thresholds:**
- 2 consecutive failures → Alert
- 3+ locations failing → Page on-call

---

## 10. Capacity Planning Metrics

### 10.1 Growth Metrics

**Track Monthly:**

```yaml
Users:
  - Active users (MAU)
  - New signups
  - Churn rate
  
Traffic:
  - Requests per second (avg/peak)
  - Data transferred (GB)
  
Data:
  - Database size (GB)
  - Storage growth rate (GB/month)
  
Compute:
  - CPU utilization (avg/peak)
  - Memory utilization (avg/peak)
  - Pod/instance count
```

**Forecasting:**

```
# Linear regression for next 12 months
- Requests/second: Current 500 → Projected 800 (60% growth)
- Database size: Current 2TB → Projected 3.5TB (75% growth)
- Monthly cost: Current $50k → Projected $75k (50% growth)
```

### 10.2 Capacity Alerts

```yaml
# Approaching capacity limits
alert: HighDatabaseUtilization
expr: |
  (database_size_gb / database_max_size_gb) > 0.70
severity: P2
annotations:
  summary: "Database at {{ $value | humanizePercentage }} capacity"
  description: "Plan for expansion"
  
alert: HighStorageUtilization
expr: |
  (disk_used_gb / disk_total_gb) > 0.75
severity: P2
annotations:
  summary: "Disk at {{ $value | humanizePercentage }} capacity"
```

---

## 11. Runbook Integration

**Every alert MUST link to a runbook**

### Example Runbook Structure:

```markdown
# High 5xx Error Rate Runbook

## Symptoms
- 5xx error rate > 1%
- Users seeing "Internal Server Error"
- Increased support tickets

## Impact
- User experience degraded
- Potential revenue loss
- SLO at risk

## Diagnosis Steps
1. Check service dashboard: [link]
2. Look for recent deployments
3. Check database health
4. Review error logs:
   ```
   index=production level=ERROR | stats count by error.type
   ```

## Resolution Steps
1. If recent deployment → Rollback
2. If database issue → Scale read replicas
3. If external API issue → Enable circuit breaker
4. If unknown → Escalate to L2

## Prevention
- Add integration tests
- Improve monitoring
- Update ADR with learnings
```

**Runbook Location:** `/docs/runbooks/` in repository

---

## 12. Custom Business Metrics

### 12.1 Revenue Metrics

```yaml
- metric: revenue_per_minute
  calculation: sum(successful_transactions) * avg(transaction_value)
  dashboard: Business Metrics
  
- metric: cart_abandonment_rate
  calculation: (carts_created - carts_completed) / carts_created
  alert_threshold: > 0.75
```

### 12.2 User Experience Metrics

```yaml
- metric: time_to_first_byte (TTFB)
  target: < 200ms
  
- metric: time_to_interactive (TTI)
  target: < 3s
  
- metric: cumulative_layout_shift (CLS)
  target: < 0.1
```

---

## 13. Observability Costs

### 13.1 Monthly Cost Breakdown

| Service | Usage | Cost |
|---------|-------|------|
| Datadog (10 hosts) | 10 hosts × $15 | $150 |
| Datadog APM | 50M spans | $300 |
| Splunk | 50GB/day | $3,000 |
| CloudWatch | Logs + metrics | $500 |
| PagerDuty | 10 users | $290 |
| **Total** | | **$4,240/month** |

### 13.2 Cost Optimization

- [ ] Reduce log retention for non-critical logs
- [ ] Lower APM sampling rate (currently 10%)
- [ ] Archive old logs to S3 ($0.004/GB/month)
- [ ] Use CloudWatch Logs Insights instead of Splunk where possible
- [ ] Review and remove unused metrics

---

## 14. Compliance & Security Monitoring

### 14.1 Security Metrics

```yaml
- Failed login attempts
- API rate limit violations
- Unusual data access patterns
- SSL certificate expiration
- Secrets rotation status
```

### 14.2 Compliance Dashboards

**GDPR Dashboard:**
- Data retention compliance
- Right to deletion requests
- Data access logs
- Cross-border transfers

**SOC2 Dashboard:**
- Access control changes
- Security incidents
- Backup verification
- Patch compliance

---

## 15. On-Call Procedures

### When Alert Fires

1. **Acknowledge** (< 15 min)
   - In PagerDuty
   - In #incidents Slack channel

2. **Assess** (< 5 min)
   - Check dashboard
   - Determine severity
   - Review recent changes

3. **Mitigate** (< 30 min for P0/P1)
   - Follow runbook
   - Rollback if recent deploy
   - Scale if capacity issue

4. **Communicate** (Every 30 min)
   - Update #incidents channel
   - Update status page if customer-facing

5. **Resolve**
   - Verify metrics normal
   - Run smoke tests
   - Document in incident report

6. **Post-Mortem** (Within 48h)
   - Root cause analysis
   - Action items
   - Update runbooks

---

## 16. Documentation Updates

**This document should be updated:**
- After every significant architecture change
- When new monitoring is added
- After post-mortems if monitoring gaps identified
- Quarterly review minimum

**Next Review Date:** YYYY-MM-DD

**Reviewers:**
- [ ] Service Owner
- [ ] SRE Lead
- [ ] Tech Lead

---

## 17. Quick Reference

### Emergency Contacts
- On-call: PagerDuty will page
- Escalation: See [Service Ownership Doc](./03-service-ownership-raci.md)

### Key Dashboards
1. Overview: https://grafana.company.com/d/overview-service-name
2. Golden Signals: https://grafana.company.com/d/golden-signals
3. On-Call: https://grafana.company.com/d/oncall-service-name

### Key Queries
```
# Recent errors
index=production service=service-name level=ERROR
| stats count by error.type

# Slow requests
index=production service=service-name duration_ms>1000
| stats avg(duration_ms) by endpoint
```

---

**Document Owner:** [SRE Lead]  
**Last Updated:** YYYY-MM-DD
