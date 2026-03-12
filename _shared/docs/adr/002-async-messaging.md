---
page_meta:
  title: "ADR-002: Async Messaging via Kafka"
  labels:
    - order-service
    - adr
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# ADR-002: Async Messaging via Kafka

**Status**: ::Accepted::green::
**Date**: @date:2024-09-03
**Authors**: Priya Mehta, Jordan Lee

---

## Context

The Order Service needs to notify downstream systems (Billing, Notification, Analytics) when order state changes. We need to decide how to integrate:

1. **Synchronous REST calls** from Order Service → each downstream service
2. **Async messaging** (event bus) — Order Service emits events; consumers handle them independently

The initial v1 implementation used synchronous REST calls. This created several problems:

- Cascading failures: if Notification Service was down, order creation would fail or require complex retry logic
- Tight coupling: adding a new consumer (e.g. Analytics) required changes to Order Service code
- Latency: order creation p99 was 1.2s because it waited for 3 downstream HTTP calls

---

## Decision

Replace synchronous REST calls with **Kafka event streaming via Amazon MSK** for all Order Service → downstream communication.

Order Service emits events to Kafka topics. Downstream services consume independently. The Schema Registry enforces Avro schemas, preventing schema drift.

Topics created:
- `orders.order-events` — all order lifecycle events (partitioned by `account_id`)

---

## Rationale

**Why Kafka over alternatives (SNS/SQS, EventBridge)**:

| Criterion | Kafka (MSK) | SNS/SQS | EventBridge |
| --- | --- | --- | --- |
| Message replay | :white_check_mark: Yes (configurable retention) | No (SQS) / No (SNS) | Limited (archive add-on) |
| Schema enforcement | :white_check_mark: Schema Registry | No native support | JSON schema (basic) |
| Ordering guarantees | :white_check_mark: Per-partition | SQS FIFO (limited) | No |
| Consumer group semantics | :white_check_mark: Native | Via SQS | No |
| Existing platform usage | :white_check_mark: MSK already provisioned | Additional service | Additional service |

Replay capability is a hard requirement — Analytics Service needs to backfill historical events when schema changes are deployed.

---

## Consequences

**Positive**:
- Order Service is decoupled from downstream systems — new consumers subscribe to Kafka without any Order Service changes
- Order creation p99 improved from 1.2s to 185ms (only synchronous call remaining is Inventory gRPC reservation)
- Event replay enables analytics backfill and consumer bug recovery
- Schema Registry prevents breaking schema changes from deploying silently

**Negative / trade-offs**:
- Eventual consistency: Billing and Notification now process events asynchronously — there is a lag between order creation and invoice generation
- Operational complexity: Kafka topic management, consumer group lag monitoring, and schema evolution require ongoing attention
- At-least-once delivery: consumers must be idempotent (use `event.id` for deduplication)

**Mitigations**:
- Kafka event lag alerting configured in Datadog (alert at > 120s lag on the orders topic)
- Consumer libraries include a deduplication helper using Redis (TTL 24h per `event_id`)
- Schema Registry enforces backward-compatible schema evolution by default
