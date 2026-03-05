---
page_meta:
  title: Webhook Integration Guide
  labels:
    - nexus
    - guides
    - webhooks
deploy_config:
  ci_banner: true
---

# Webhook Integration Guide

Nexus webhooks deliver real-time notifications to your application when events occur on your account — order status changes, inventory updates, and more.

---

## Overview

1. Register an HTTPS endpoint in the Nexus Dashboard or via the API
2. Nexus delivers a POST request to your endpoint when a subscribed event occurs
3. Respond with `2xx` within 10 seconds to acknowledge delivery
4. Nexus retries failed deliveries with exponential backoff

> [!info]
> Webhooks are delivered at least once — your endpoint must be idempotent. Use the event `id` field to deduplicate.

---

## Registering a webhook endpoint

### Via the Dashboard

1. Navigate to **Settings → Webhooks**
2. Click **Add endpoint**
3. Enter your HTTPS URL and select events to subscribe to
4. Save — Nexus will send a test event immediately

### Via the API

```bash
curl -X POST https://api.nexus.io/v2/webhooks \
  -H "Authorization: Bearer $NEXUS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://yourapp.com/nexus/webhooks",
    "events": [
      "order.created",
      "order.status.changed",
      "inventory.low_stock"
    ],
    "description": "Production order pipeline"
  }'
```

---

## Available events

| Event | Trigger |
| --- | --- |
| `order.created` | New order created via API or Dashboard |
| `order.status.changed` | Order status transitions (e.g. pending → confirmed) |
| `order.cancelled` | Order cancelled (includes reason) |
| `order.fulfilled` | Order marked as delivered |
| `inventory.low_stock` | SKU stock falls below configured threshold |
| `inventory.level.changed` | Any stock level adjustment |
| `payment.succeeded` | Payment captured successfully |
| `payment.failed` | Payment attempt failed |

---

## Webhook payload format

All events share a common envelope:

```json
{
  "id": "evt_01h9ab...",
  "type": "order.status.changed",
  "created_at": "2026-03-01T14:32:00Z",
  "account_id": "acc_01h8xq...",
  "data": {
    ...
  }
}
```

### Example: `order.status.changed`

```json
{
  "id": "evt_01h9ab...",
  "type": "order.status.changed",
  "created_at": "2026-03-01T14:32:00Z",
  "account_id": "acc_01h8xq...",
  "data": {
    "order_id": "ord_01h8xr...",
    "external_ref": "your-order-123",
    "previous_status": "pending",
    "new_status": "confirmed",
    "changed_at": "2026-03-01T14:31:58Z"
  }
}
```

### Example: `inventory.low_stock`

```json
{
  "id": "evt_01h9ac...",
  "type": "inventory.low_stock",
  "created_at": "2026-03-01T15:00:00Z",
  "account_id": "acc_01h8xq...",
  "data": {
    "sku": "WIDGET-RED-M",
    "current_stock": 5,
    "low_stock_threshold": 10,
    "warehouse": "LON-1"
  }
}
```

---

## Verifying webhook signatures

Nexus signs every webhook delivery with HMAC-SHA256. Verify the signature to confirm the request came from Nexus.

```python
import hmac
import hashlib

def verify_nexus_webhook(payload: bytes, signature_header: str, secret: str) -> bool:
    """Verify a Nexus webhook signature.

    Args:
        payload: Raw request body bytes.
        signature_header: Value of the X-Nexus-Signature header.
        secret: Your webhook signing secret from the Dashboard.
    """
    expected = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature_header)
```

> [!warning]
> Always verify signatures in production. Skipping verification exposes your endpoint to spoofed payloads. Use `hmac.compare_digest` (not `==`) to prevent timing attacks.

---

## Retry behaviour

| Attempt | Delay |
| --- | --- |
| 1st retry | 30 seconds |
| 2nd retry | 5 minutes |
| 3rd retry | 30 minutes |
| 4th retry | 2 hours |
| 5th retry | 8 hours |

After 5 failed attempts, the event is marked as failed and no further retries occur. Failed events are visible in **Settings → Webhooks → Deliveries** and can be manually retried from the Dashboard.

> [!note]
> Your endpoint must respond within **10 seconds**. If you need to do heavy processing, acknowledge the webhook immediately and process the payload asynchronously using a queue.
