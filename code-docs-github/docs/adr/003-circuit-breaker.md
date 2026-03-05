---
page_meta:
  title: "ADR-003: Circuit Breaker Pattern"
  labels:
    - order-service
    - adr
    - resilience
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# ADR-003: Circuit Breaker Pattern for Downstream Calls

**Status**: ::Proposed::yellow::
**Date**: @2026-02-10@
**Authors**: Sarah Kim
**Review deadline**: @2026-03-28@

---

## Context

The Order Service makes one synchronous downstream call during order creation: a gRPC inventory reservation request to the Inventory Service. If Inventory Service is slow or returning errors, order creation latency and error rate spike — directly impacting customers.

In a December 2025 incident (`INC-2847`), a memory leak in Inventory Service caused p99 response times to climb to 8s. Order creation timed out for approximately 12 minutes before the Inventory Service pod was restarted. 340 orders failed.

The current mitigation is a 2s gRPC timeout, but this results in hard failures for customers rather than graceful degradation.

> [!warning]
> This ADR is under active review. Do not implement until the decision is marked Accepted. Reach out to Sarah Kim or the Orders team in `#orders-eng` with feedback.

---

## Options considered

### Option A: Circuit breaker with fail-open fallback

Wrap the Inventory Service gRPC call in a circuit breaker (using `pybreaker` or similar). When the circuit opens (after a configurable failure threshold), fall back to accepting orders optimistically — reserve inventory asynchronously and compensate if stock is insufficient.

**Pros**: Near-zero order creation failure rate during downstream outages
**Cons**: Risk of overselling inventory during the open-circuit window; requires compensation logic (saga pattern)

### Option B: Circuit breaker with fail-closed (current behaviour)

Circuit breaker opens and returns an error to the caller. No change to failure behaviour — just faster detection and recovery (stops hammering a failing downstream).

**Pros**: No overselling risk; simpler implementation
**Cons**: Customers still see errors during outages; limited UX improvement

### Option C: Async inventory reservation (bigger change)

Move inventory reservation off the critical path entirely — accept the order immediately and validate stock asynchronously. Cancel if stock is unavailable.

**Pros**: Eliminates dependency on Inventory Service from order creation path
**Cons**: Significant architectural change; complicates the order creation UX (order accepted then cancelled confuses customers)

---

## Proposed decision

Implement **Option A** (fail-open circuit breaker) with a hard oversell limit per SKU configured in Inventory Service. Accept orders optimistically for up to 60 seconds of circuit-open time, with automated compensation (cancel + notify) if inventory cannot be confirmed.

---

## Open questions

1. What is an acceptable oversell tolerance per SKU? (Need input from Product + Ops)
2. Should the circuit breaker state be shared across pods (Redis) or per-instance?
3. How do we communicate to customers when their order is in "pending confirmation" state?

---

## Consequences (if accepted)

**Positive**:
- Order creation remains available during Inventory Service degradation
- Limits blast radius of Inventory Service incidents on customer experience

**Negative / trade-offs**:
- Compensation logic adds complexity to the order lifecycle
- Potential for short-term inventory oversell during open-circuit windows
- Requires customer-facing communication changes for the "pending confirmation" state
