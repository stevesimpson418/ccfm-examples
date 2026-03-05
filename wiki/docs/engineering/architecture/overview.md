---
page_meta:
  title: Architecture Overview
  labels:
    - engineering
    - architecture
deploy_config:
  ci_banner: true
---

# Architecture Overview

Acme Corp's platform is a set of loosely coupled services communicating over a mix of synchronous REST APIs and asynchronous messaging via Kafka. This page gives a high-level view; each service has its own dedicated docs in the relevant team's space.

## System diagram

```
                        ┌──────────────────────────────────────┐
                        │           API Gateway (Kong)          │
                        └──────────────┬───────────────────────┘
                                       │ REST
              ┌────────────────────────┼──────────────────────┐
              │                        │                       │
    ┌─────────▼──────┐    ┌────────────▼───────┐   ┌──────────▼──────┐
    │  Order Service │    │  Inventory Service  │   │  User Service   │
    │  (Python/FastAPI)   │  (Go)               │   │  (Node/Express) │
    └────────┬───────┘    └────────────┬────────┘   └──────────┬──────┘
             │ Kafka                   │ Kafka                  │ Postgres
             │                        │                         │
    ┌────────▼────────────────────────▼────────┐      ┌─────────▼──────┐
    │              Kafka (MSK)                  │      │   RDS Postgres  │
    │   order.created  inventory.reserved       │      │   (users DB)    │
    └───────────────────────────────────────────┘      └────────────────┘
```

> [!info]
> The API Gateway handles authentication, rate limiting, and routing. Internal service-to-service calls bypass the gateway and use mTLS.

---

## Service catalogue

| Service | Language | Owner | Purpose |
| --- | --- | --- | --- |
| **Order Service** | Python / FastAPI | Orders Team | Create, manage, and fulfil customer orders |
| **Inventory Service** | Go | Platform Team | Track stock levels, reserve items, emit events |
| **User Service** | Node / Express | Identity Team | Authentication, profiles, API key management |
| **Notification Service** | Python | Platform Team | Email/SMS delivery via SES/SNS |
| **Billing Service** | Python | Finance Eng | Stripe integration, invoicing, subscription lifecycle |
| **Analytics Ingestion** | Go | Data Team | Consumes Kafka events, writes to Redshift |
| **API Gateway** | Kong | Platform Team | Edge routing, auth, rate limiting |

---

## Infrastructure

| Component | Technology | Notes |
| --- | --- | --- |
| Container orchestration | EKS (Kubernetes 1.30) | All services deployed via Helm charts |
| Messaging | MSK (Kafka 3.7) | Managed Kafka on AWS |
| Databases | RDS Postgres 16 | One cluster per service (schema isolation) |
| Object storage | S3 | Artefacts, exports, backups |
| Secrets | AWS Secrets Manager | No plaintext secrets in environment variables |
| CI/CD | GitHub Actions | Deploy to staging on PR merge; production on tag |
| Observability | Datadog | APM, logs, dashboards, alerting |

---

## SLOs

> [!warning]
> These are targets, not guarantees. Check the Datadog dashboard for current error budget status before making changes to core services.

| Service | Availability SLO | Latency SLO (p99) | Error Budget (rolling 30d) |
| --- | --- | --- | --- |
| Order Service | 99.9% | < 500ms | ::Healthy::green:: |
| Inventory Service | 99.9% | < 200ms | ::Healthy::green:: |
| User Service | 99.95% | < 300ms | ::At Risk::yellow:: |
| API Gateway | 99.99% | < 50ms | ::Healthy::green:: |
