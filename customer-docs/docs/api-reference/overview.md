---
page_meta:
  title: API Reference Overview
  labels:
    - nexus
    - api-reference
deploy_config:
  ci_banner: true
---

# API Reference Overview

## Base URL

```
https://api.nexus.io/v2
```

All endpoints are versioned. The current stable version is `v2`. Breaking changes are only introduced in major versions with a minimum 6-month deprecation notice.

## Versioning policy

| Version | Status | End of life |
| --- | --- | --- |
| v2 | ::Active::green:: | No planned EOL |
| v1 | ::Deprecated::yellow:: | @2026-12-31@ |

> [!warning]
> v1 is deprecated and will stop accepting new requests on @2026-12-31@. Migrate to v2 before this date. See the [v1 → v2 migration guide](page:"API Reference Overview") for breaking changes.

## Request format

All request bodies must be JSON with `Content-Type: application/json`. Dates use ISO 8601 (`2026-03-01T12:00:00Z`). Monetary values are integers in the smallest currency unit (e.g. pence for GBP, cents for USD).

## Response format

All responses return JSON. Successful responses use 2xx status codes. Errors follow a consistent format — see [Error Reference](page:"Error Reference") for the full list.

```json
{
  "id": "ord_01h8xr...",
  "object": "order",
  "created_at": "2026-03-01T12:00:00Z",
  ...
}
```

## Pagination

List endpoints return paginated results using cursor-based pagination:

```bash
GET /v2/orders?limit=20&after=ord_01h8xr...
```

| Parameter | Default | Max |
| --- | --- | --- |
| `limit` | 20 | 100 |
| `after` | — | — |
| `before` | — | — |

Responses include a `pagination` object:

```json
{
  "data": [...],
  "pagination": {
    "has_more": true,
    "next_cursor": "ord_01h9ab...",
    "prev_cursor": "ord_01h7zz..."
  }
}
```

## Rate limits

Requests are rate-limited per API key. See [Rate Limiting](page:"Rate Limiting") for tier details and headers.

| Plan | Requests/minute | Requests/day |
| --- | --- | --- |
| Developer | 60 | 1,000 |
| Growth | 300 | 50,000 |
| Enterprise | 2,000 | Unlimited |
