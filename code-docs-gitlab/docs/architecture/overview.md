---
page_meta:
  title: Service Overview
  labels:
    - order-service
    - architecture
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# Service Overview

The Order Service manages the full lifecycle of customer orders within the Acme Corp platform. It is the authoritative source of truth for order state and coordinates with Inventory, Billing, and Notification services via Kafka.

---

## Purpose

- Accept order creation requests (REST API and internal gRPC)
- Validate items against Inventory Service availability
- Persist order state and emit lifecycle events to Kafka
- Expose a read API for order history and status queries
- Handle cancellations, amendments, and fulfilment confirmations

---

## Tech stack

| Component | Technology | Version | Notes |
| --- | --- | --- | --- |
| Language | Python | 3.12 | |
| Web framework | FastAPI | 0.115 | Async handlers throughout |
| Database | PostgreSQL | 16 | Managed via RDS |
| ORM | SQLAlchemy | 2.0 | Async sessions |
| Message broker | Kafka | 3.7 (MSK) | Schema registry enforced |
| Serialisation | Pydantic v2 | 2.8 | Request/response models |
| Testing | pytest + pytest-asyncio | — | Coverage ≥ 80% enforced |
| Containerisation | Docker | — | EKS deployment via Helm |

---

## Service dependencies

```
┌──────────────────────────────────────────────────────────────────┐
│                        Order Service                              │
│                                                                    │
│  REST API (inbound)        Kafka (outbound events)                │
│  ├── POST /orders          ├── order.created                      │
│  ├── GET  /orders          ├── order.status.changed               │
│  ├── GET  /orders/{id}     ├── order.cancelled                    │
│  └── POST /orders/cancel   └── order.fulfilled                    │
└─────────────────┬──────────────────────────────────────────────┘
                  │ Synchronous calls
      ┌───────────┼────────────────────┐
      ▼           ▼                    ▼
 Inventory    Billing           Notification
 Service      Service           Service
 (gRPC)       (internal REST)   (Kafka consumer)
```

> [!info]
> Inventory availability checks are synchronous (gRPC) to provide immediate feedback on order creation. All other downstream coordination is async via Kafka to avoid cascading failures.

---

## SLOs

| Metric | Target | Alert threshold |
| --- | --- | --- |
| Availability | 99.9% (rolling 30d) | < 99.8% |
| Order creation p99 latency | < 500ms | > 800ms |
| Order read p99 latency | < 150ms | > 300ms |
| Kafka event lag | < 30 seconds | > 120 seconds |

Error budget: **43.8 minutes/month** downtime. Current status visible in Datadog → Order Service SLO dashboard.

---

## Deployment

- **Staging**: Deployed automatically on merge to `main`
- **Production**: Deployed via manual trigger in the GitLab pipeline UI
- **Helm chart**: `charts/order-service` in the `acme-corp/platform-charts` repo
- **Namespace**: `orders` (staging), `orders` (production — separate cluster)

See [Deployment Runbook](page:"Deployment") for step-by-step instructions.
