---
page_meta:
  title: REST API
  labels:
    - order-service
    - api-reference
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# REST API

The Order Service exposes a REST API consumed by the API Gateway (external) and internal services. Internal callers use mTLS; external callers authenticate via the API Gateway.

**Base path** (internal): `http://order-service.orders.svc.cluster.local:8080`
**Base path** (external, via gateway): `https://api.nexus.io/v2`

---

## Endpoints

| Method | Path | Auth | Description | Response |
| --- | --- | --- | --- | --- |
| `POST` | `/orders` | API key / OAuth | Create a new order | `201` Order object |
| `GET` | `/orders` | API key / OAuth | List orders (paginated, filterable) | `200` Paginated list |
| `GET` | `/orders/{id}` | API key / OAuth | Get order by ID | `200` Order object |
| `PATCH` | `/orders/{id}` | API key / OAuth | Update mutable fields | `200` Order object |
| `POST` | `/orders/{id}/cancel` | API key / OAuth | Cancel an order | `200` Order object |
| `POST` | `/orders/bulk` | API key | Create up to 500 orders | `207` Multi-status |
| `GET` | `/orders/{id}/events` | API key / OAuth | Fetch order event history | `200` Event list |
| `GET` | `/health` | None | Liveness probe | `200` `{"status": "ok"}` |
| `GET` | `/ready` | None | Readiness probe | `200` / `503` |
| `GET` | `/metrics` | Internal only | Prometheus metrics | `200` text |

---

## `POST /orders`

Creates a new order. Synchronously reserves inventory before returning.

**Request body**:

```json
{
  "external_ref": "your-order-abc",
  "items": [
    {
      "sku": "WIDGET-RED-M",
      "quantity": 2,
      "unit_price": 1499
    }
  ],
  "shipping_address": {
    "line1": "123 High Street",
    "city": "London",
    "postcode": "EC1A 1BB",
    "country": "GB"
  }
}
```

**Success**: `201 Created` — returns the full order object with assigned `id`.

**Error cases**:

| Condition | Status | Code |
| --- | --- | --- |
| `external_ref` already exists | `409` | `duplicate_external_ref` |
| SKU not found in Inventory | `422` | `invalid_sku` |
| Insufficient stock | `409` | `insufficient_stock` |
| Validation failure | `422` | `missing_required_field` / `invalid_field_type` |

---

## `POST /orders/bulk`

Accepts an array of up to 500 order objects. Each item is processed independently — partial success is possible.

**Request body**:

```json
{
  "orders": [
    { ... },
    { ... }
  ]
}
```

**Response** (`207 Multi-Status`):

```json
{
  "results": [
    {
      "index": 0,
      "status": "created",
      "order_id": "ord_01h8xr..."
    },
    {
      "index": 1,
      "status": "failed",
      "error": {
        "code": "insufficient_stock",
        "message": "SKU WIDGET-BLUE-L has 0 units available"
      }
    }
  ],
  "summary": {
    "total": 2,
    "created": 1,
    "failed": 1
  }
}
```

---

## `GET /orders`

**Query parameters**:

| Parameter | Type | Description |
| --- | --- | --- |
| `status` | string | Filter by order status |
| `external_ref` | string | Exact match on external ref |
| `created_after` | ISO 8601 | Orders created after this timestamp |
| `created_before` | ISO 8601 | Orders created before this timestamp |
| `limit` | integer | Page size (default 20, max 100) |
| `after` | string | Cursor for next page |
