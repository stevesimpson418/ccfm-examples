---
page_meta:
  title: Endpoints
  labels:
    - nexus
    - api-reference
deploy_config:
  ci_banner: true
---

# Endpoints

Full list of Nexus Platform API endpoints. All paths are relative to `https://api.nexus.io/v2`.

For authentication details, see [Authentication](page:"Authentication"). For error codes, see [Error Reference](page:"Error Reference").

---

## Orders

| Method | Path | Auth scope | Description |
| --- | --- | --- | --- |
| `GET` | `/orders` | `orders:read` | List orders with optional filters |
| `POST` | `/orders` | `orders:write` | Create a new order |
| `GET` | `/orders/{id}` | `orders:read` | Retrieve a specific order |
| `PATCH` | `/orders/{id}` | `orders:write` | Update order fields (address, external_ref) |
| `POST` | `/orders/{id}/cancel` | `orders:write` | Cancel an order |
| `POST` | `/orders/bulk` | `orders:write` | Create up to 500 orders in a single request |
| `GET` | `/orders/{id}/events` | `orders:read` | List lifecycle events for an order |

### Order object

```json
{
  "id": "ord_01h8xr...",
  "object": "order",
  "external_ref": "your-order-123",
  "status": "pending",
  "items": [
    {
      "id": "item_01h8xs...",
      "sku": "WIDGET-RED-M",
      "quantity": 2,
      "unit_price": 1499,
      "line_total": 2998
    }
  ],
  "total": 2998,
  "currency": "GBP",
  "shipping_address": { ... },
  "created_at": "2026-03-01T12:00:00Z",
  "updated_at": "2026-03-01T12:00:00Z"
}
```

---

## Inventory

| Method | Path | Auth scope | Description |
| --- | --- | --- | --- |
| `GET` | `/inventory` | `inventory:read` | List all inventory items |
| `POST` | `/inventory` | `inventory:write` | Create a new inventory item |
| `GET` | `/inventory/{sku}` | `inventory:read` | Get stock level for a SKU |
| `PATCH` | `/inventory/{sku}` | `inventory:write` | Update item details or stock level |
| `POST` | `/inventory/{sku}/adjust` | `inventory:write` | Apply a stock adjustment (delta) |
| `GET` | `/inventory/{sku}/history` | `inventory:read` | Stock level change history |

---

## Webhooks

| Method | Path | Auth scope | Description |
| --- | --- | --- | --- |
| `GET` | `/webhooks` | `webhooks:manage` | List configured webhook endpoints |
| `POST` | `/webhooks` | `webhooks:manage` | Register a new webhook endpoint |
| `GET` | `/webhooks/{id}` | `webhooks:manage` | Get webhook configuration |
| `PATCH` | `/webhooks/{id}` | `webhooks:manage` | Update endpoint URL or events |
| `DELETE` | `/webhooks/{id}` | `webhooks:manage` | Delete a webhook endpoint |
| `GET` | `/webhooks/{id}/deliveries` | `webhooks:manage` | View recent delivery attempts |
| `POST` | `/webhooks/{id}/test` | `webhooks:manage` | Send a test event to the endpoint |

---

## Account

| Method | Path | Auth scope | Description |
| --- | --- | --- | --- |
| `GET` | `/account` | `account:read` | Get account details and plan information |
| `GET` | `/account/usage` | `account:read` | API usage statistics for current billing period |

---

## Order status values

| Status | Description |
| --- | --- |
| `pending` | Order received, awaiting inventory reservation |
| `confirmed` | Inventory reserved, order confirmed |
| `processing` | Order is being picked and packed |
| `shipped` | Order dispatched — tracking info available |
| `delivered` | Delivery confirmed |
| `cancelled` | Order cancelled (see `cancelled_reason`) |
| `failed` | Order could not be fulfilled |
