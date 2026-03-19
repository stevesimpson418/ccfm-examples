---
page_meta:
  title: Kafka Events
  labels:
    - order-service
    - api-reference
    - kafka
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# Kafka Events

The Order Service publishes all order lifecycle events to Kafka. Downstream services (Billing, Notification, Analytics) consume these events independently.

**Topic**: `orders.order-events`
**Partition key**: `account_id` (ensures all events for an account are ordered)
**Schema format**: Avro (registered in Schema Registry at `https://schema-registry.internal`)
**Retention**: 7 days

---

## Event catalogue

| Event type | Trigger | Downstream consumers |
| --- | --- | --- |
| `order.created` | New order successfully created | Billing, Notification, Analytics |
| `order.status.changed` | Any order status transition | Notification, Analytics |
| `order.cancelled` | Order cancelled by customer or system | Billing, Notification, Analytics |
| `order.fulfilled` | Order marked delivered | Billing, Analytics |
| `order.failed` | Order failed (e.g. inventory compensation failed) | Billing, Notification |

---

## Common event envelope

All events share this envelope:

```json
{
  "event_id": "evt_01h9ab...",
  "event_type": "order.created",
  "schema_version": 2,
  "occurred_at": "2026-03-01T14:32:00.000Z",
  "account_id": "acc_01h8xq...",
  "correlation_id": "req_01h8xz...",
  "payload": { ... }
}
```

---

## Event schemas

> [!expand order.created]
> **Emitted when**: Order successfully created and inventory reserved.
>
> ```json
> {
>   "event_id": "evt_01h9ab...",
>   "event_type": "order.created",
>   "schema_version": 2,
>   "occurred_at": "2026-03-01T14:32:00.000Z",
>   "account_id": "acc_01h8xq...",
>   "correlation_id": "req_01h8xz...",
>   "payload": {
>     "order_id": "ord_01h8xr...",
>     "external_ref": "your-order-123",
>     "status": "pending",
>     "currency": "GBP",
>     "total_amount": 2998,
>     "items": [
>       {
>         "sku": "WIDGET-RED-M",
>         "quantity": 2,
>         "unit_price": 1499,
>         "line_total": 2998
>       }
>     ],
>     "shipping_address": {
>       "country": "GB",
>       "postcode": "EC1A 1BB"
>     }
>   }
> }
> ```

> [!expand order.status.changed]
> **Emitted when**: Order transitions between any two statuses.
>
> ```json
> {
>   "event_id": "evt_01h9ac...",
>   "event_type": "order.status.changed",
>   "schema_version": 2,
>   "occurred_at": "2026-03-01T14:35:00.000Z",
>   "account_id": "acc_01h8xq...",
>   "correlation_id": "sys_fulfil_01...",
>   "payload": {
>     "order_id": "ord_01h8xr...",
>     "external_ref": "your-order-123",
>     "previous_status": "confirmed",
>     "new_status": "processing",
>     "actor": "fulfil-worker-v2",
>     "changed_at": "2026-03-01T14:34:58.000Z"
>   }
> }
> ```

> [!expand order.cancelled]
> **Emitted when**: Order cancelled by API call or internal system action.
>
> ```json
> {
>   "event_id": "evt_01h9ad...",
>   "event_type": "order.cancelled",
>   "schema_version": 2,
>   "occurred_at": "2026-03-01T16:00:00.000Z",
>   "account_id": "acc_01h8xq...",
>   "correlation_id": "req_01h8aa...",
>   "payload": {
>     "order_id": "ord_01h8xr...",
>     "external_ref": "your-order-123",
>     "cancelled_reason": "customer_request",
>     "cancelled_at": "2026-03-01T16:00:00.000Z",
>     "refund_eligible": true
>   }
> }
> ```

> [!expand order.fulfilled]
> **Emitted when**: Order confirmed as delivered by the fulfilment system.
>
> ```json
> {
>   "event_id": "evt_01h9ae...",
>   "event_type": "order.fulfilled",
>   "schema_version": 2,
>   "occurred_at": "2026-03-03T10:20:00.000Z",
>   "account_id": "acc_01h8xq...",
>   "correlation_id": "sys_delivery_01...",
>   "payload": {
>     "order_id": "ord_01h8xr...",
>     "external_ref": "your-order-123",
>     "fulfilled_at": "2026-03-03T10:20:00.000Z",
>     "carrier": "DPD",
>     "tracking_number": "1Z999AA1012345678"
>   }
> }
> ```

---

## Consumer requirements

> [!warning]
> Events are delivered **at least once**. Consumers must be idempotent. Use `event_id` to deduplicate — a Redis set with 24h TTL is the recommended approach.

> [!info]
> Schema Registry enforces backward-compatible evolution. Adding optional fields is safe. Removing or renaming fields requires a new schema version and a coordinated consumer migration.
