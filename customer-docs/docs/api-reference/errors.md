---
page_meta:
  title: Error Reference
  labels:
    - nexus
    - api-reference
deploy_config:
  ci_banner: true
---

# Error Reference

All Nexus API errors return a consistent JSON structure with an HTTP status code and a machine-readable error code.

```json
{
  "error": {
    "code": "order_not_found",
    "message": "No order found with ID ord_01h8xr...",
    "status": 404,
    "request_id": "req_01h8xz..."
  }
}
```

Always log the `request_id` — include it when contacting support.

---

## HTTP status codes

| Status | Meaning |
| --- | --- |
| `200 OK` | Request succeeded |
| `201 Created` | Resource created successfully |
| `204 No Content` | Request succeeded, no response body |
| `400 Bad Request` | Invalid request body or parameters |
| `401 Unauthorized` | Missing or invalid authentication |
| `403 Forbidden` | Valid auth, insufficient scope |
| `404 Not Found` | Resource does not exist |
| `409 Conflict` | Request conflicts with current state |
| `422 Unprocessable Entity` | Validation failed |
| `429 Too Many Requests` | Rate limit exceeded |
| `500 Internal Server Error` | Nexus server error |

---

## Error codes by category

:::expand Authentication errors (401 / 403)
| Code | Status | Description |
| --- | --- | --- |
| `missing_authorization` | 401 | `Authorization` header not provided |
| `invalid_api_key` | 401 | API key format is invalid or key does not exist |
| `expired_api_key` | 401 | API key has been rotated or revoked |
| `invalid_oauth_token` | 401 | OAuth access token is invalid or expired |
| `insufficient_scope` | 403 | Token lacks required scope for this operation |
| `account_suspended` | 403 | Account has been suspended — contact support |
:::

:::expand Validation errors (400 / 422)
| Code | Status | Description |
| --- | --- | --- |
| `invalid_json` | 400 | Request body is not valid JSON |
| `missing_required_field` | 422 | A required field is absent — see `field` in response |
| `invalid_field_type` | 422 | Field value has incorrect type — see `field` |
| `invalid_currency` | 422 | Currency code is not supported |
| `invalid_sku` | 422 | SKU format is invalid (max 64 chars, alphanumeric + hyphens) |
| `duplicate_external_ref` | 409 | `external_ref` already exists for this account |
| `quantity_must_be_positive` | 422 | Order item quantity must be ≥ 1 |
| `price_must_be_positive` | 422 | Unit price must be a positive integer |
:::

:::expand Order errors (404 / 409)
| Code | Status | Description |
| --- | --- | --- |
| `order_not_found` | 404 | Order with given ID does not exist |
| `order_not_cancellable` | 409 | Order is in a state that cannot be cancelled (shipped/delivered) |
| `order_already_cancelled` | 409 | Order has already been cancelled |
| `bulk_limit_exceeded` | 422 | Bulk request contains more than 500 orders |
| `insufficient_stock` | 409 | One or more items have insufficient stock to fulfil |
:::

:::expand Inventory errors
| Code | Status | Description |
| --- | --- | --- |
| `sku_not_found` | 404 | No inventory item with given SKU |
| `sku_already_exists` | 409 | SKU is already registered in your inventory |
| `stock_level_negative` | 422 | Adjustment would result in negative stock level |
:::

:::expand Rate limit errors (429)
| Code | Status | Description |
| --- | --- | --- |
| `rate_limit_exceeded` | 429 | Request rate limit exceeded — see `Retry-After` header |
| `daily_limit_exceeded` | 429 | Daily request quota exceeded — resets at midnight UTC |

See [Rate Limiting](page:"Rate Limiting") for tier limits and how to handle backoff.
:::

:::expand Server errors (5xx)
| Code | Status | Description |
| --- | --- | --- |
| `internal_error` | 500 | Unexpected server error — include `request_id` when reporting |
| `service_unavailable` | 503 | Nexus is temporarily unavailable — retry with exponential backoff |
| `gateway_timeout` | 504 | Upstream service did not respond in time — safe to retry |

> [!warning]
> 5xx errors indicate a problem on Nexus's side. Check [status.nexus.io](https://status.nexus.io) for active incidents. Safe to retry with backoff; `POST` requests should only be retried if idempotency keys are used.
:::
