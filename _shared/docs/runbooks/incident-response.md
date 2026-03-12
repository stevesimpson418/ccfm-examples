---
page_meta:
  title: Incident Response
  labels:
    - order-service
    - runbook
    - incident-response
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# Incident Response

Response procedures for Order Service production incidents.

**On-call**: PagerDuty → Orders rotation
**Escalation**: `#incidents` Slack channel

---

## Severity levels

| Severity | Definition | Response time | Examples |
| --- | --- | --- | --- |
| ::P1 - Critical::red:: | Complete order creation outage or data loss | Immediate — all hands | Order Service down, orders being lost, billing not receiving events |
| ::P2 - High::yellow:: | Degraded order creation (error rate > 1%) or major feature broken | Within 15 minutes | 5xx spike, p99 latency > 2s, bulk endpoint returning errors |
| ::P3 - Medium::blue:: | Minor degradation, workaround exists | Within 1 hour | Slow list queries, webhook delivery lag, admin API errors |
| ::P4 - Low::grey:: | Cosmetic or minor issue, no customer impact | Next business day | Incorrect metric labels, non-critical log noise |

---

## P1 / P2 response steps

### 1. Acknowledge and declare

1. Acknowledge the PagerDuty alert
2. Post in `#incidents`:
   ```
   :rotating_light: INCIDENT DECLARED — Order Service [P1/P2]
   Impact: [describe what's broken]
   IC: @your-name
   Status thread: [link to this message]
   ```
3. You are the **Incident Commander (IC)** until you hand off

### 2. Gather context (first 5 minutes)

```bash
# Check pod health
kubectl get pods -n orders

# Recent logs (last 5 minutes)
kubectl logs -n orders -l app=order-service --since=5m | tail -100

# Check recent deploys
helm history order-service -n orders | tail -5
```

Open the Datadog dashboards:
- **Order Service Overview** — error rate, latency, throughput
- **Kafka Consumer Lag** — `orders.order-events` consumer groups
- **RDS Performance** — query latency, connection count, replication lag

### 3. Assess and act

| Observation | Likely cause | Action |
| --- | --- | --- |
| Pods crash-looping | Bad deploy / OOM | [Rollback](page:"Rollback Procedure") |
| High DB connection count | Connection pool exhaustion | Scale pods down temporarily; check for slow queries |
| Inventory Service timeouts | Downstream degradation | Monitor; consider circuit breaker manual open if ADR-003 implemented |
| Kafka lag growing | Consumer not processing | Check consumer logs; restart consumer pods |
| No recent deploy, sudden spike | Upstream traffic surge / bot | Check API Gateway metrics; rate limit if needed |

> [!warning]
> If you're unsure of the cause, do not make changes speculatively. Gather evidence first — a wrong action can extend the incident.

### 4. Communicate every 10 minutes

Post status updates in `#incidents` every 10 minutes during an active P1/P2:

```
[14:35] Status update: pods are now stable, error rate dropping from 8% to 2%.
Root cause suspected: slow DB query after migration. Investigating via Datadog APM.
```

### 5. Escalation contacts

| Situation | Escalate to |
| --- | --- |
| RDS issue (replication, disk, failover) | `#platform-oncall` |
| Kafka / MSK issue | `#platform-oncall` |
| Billing data corruption suspected | Marcus Thompson (Finance Eng) + VP Engineering |
| Customer data loss suspected | VP Engineering + Legal (immediate) |
| Sustained P1 > 30 minutes | VP Engineering @sam.okafor |

---

## Post-incident

Within **48 hours** of resolution:

1. Write an incident review in the `ORDERS` Jira project (use the "Incident Review" template)
2. Include: timeline, root cause, impact, what went well, what didn't, action items with owners
3. Share the review in `#incidents` and `#engineering`
4. Action items must be in Jira with owners and target sprint assigned

> [!note]
> Incident reviews are blameless. The goal is systemic improvement, not fault-finding. If you're nervous about writing one, pair with your manager or a senior engineer.
