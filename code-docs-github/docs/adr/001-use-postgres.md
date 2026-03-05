---
page_meta:
  title: "ADR-001: Use PostgreSQL as Primary Database"
  labels:
    - order-service
    - adr
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# ADR-001: Use PostgreSQL as Primary Database

**Status**: ::Accepted::green::
**Date**: @2024-08-12@
**Authors**: Priya Mehta, Sarah Kim

---

## Context

The Order Service needs a persistent store for order records, line items, and an audit log of status transitions. Key requirements:

- Strong consistency — order creation and inventory reservation must be atomic or safely compensated
- Complex queries — filtering by account, date range, status, and external ref; joining items to orders
- Audit trail — immutable event log alongside mutable order state
- Operational familiarity — the team has deep PostgreSQL expertise; reduced operational risk

We evaluated three options: PostgreSQL (RDS), DynamoDB, and MongoDB.

---

## Decision

Use **PostgreSQL 16 on Amazon RDS** as the primary database for the Order Service.

---

## Rationale

| Criterion | PostgreSQL | DynamoDB | MongoDB |
| --- | --- | --- | --- |
| ACID transactions | :white_check_mark: Full | :white_check_mark: (limited cross-table) | :white_check_mark: (multi-document since 4.0) |
| Complex query support | :white_check_mark: SQL, CTEs, window functions | Requires GSIs, limited | :white_check_mark: Aggregation pipeline |
| Schema enforcement | :white_check_mark: Strong | None | Optional |
| Team familiarity | :white_check_mark: High | Medium | Low |
| Managed service (AWS) | :white_check_mark: RDS | :white_check_mark: Native | Requires DocumentDB or Atlas |
| Cost at scale | Medium | Low–medium | Medium |

DynamoDB was ruled out primarily because the access patterns for the audit log (scan by `order_id`, time-ordered) and order lists (multi-attribute filters) are difficult and expensive to model without many GSIs. MongoDB was ruled out due to low team familiarity and higher operational overhead than RDS.

---

## Consequences

**Positive**:
- Full SQL expressiveness for complex reporting queries
- Transactions span `orders`, `order_items`, and `order_events` tables atomically
- RDS handles patching, backups, and Multi-AZ failover
- SQLAlchemy async ORM enables type-safe query composition

**Negative / trade-offs**:
- Vertical scaling — RDS requires instance resizing for large throughput increases (plan for read replicas early)
- Schema migrations require backward-compatible changes during zero-downtime deploys (see [Data Model](page:"Data Model") for policy)
- Connection pool management needed (using `asyncpg` via SQLAlchemy async engine, pool size tuned per environment)

**Follow-up**:
- Add a read replica for the reporting/list API endpoints to offload read traffic from the primary — tracked as `ORDERS-412`
