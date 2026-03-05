---
page_meta:
  title: API Changelog
  labels:
    - nexus
    - changelog
deploy_config:
  ci_banner: true
---

# API Changelog

Changes to the Nexus Platform API. Breaking changes are marked ::Breaking::red:: and announced at least 6 months in advance.

---

## v2.8.0 — @2026-02-20@

::Released::green::

- **New**: `POST /orders/bulk` — create up to 500 orders in a single request
- **New**: `GET /orders/{id}/events` — order lifecycle event history
- **New**: `inventory.low_stock` webhook event with configurable threshold per SKU
- **Improved**: Rate limit headers (`X-RateLimit-*`) now returned on all endpoints
- **Improved**: Validation error responses now include a `field` attribute

---

## v2.7.0 — @2026-01-10@

::Released::green::

- **New**: `PATCH /inventory/{sku}` — update item metadata without affecting stock level
- **New**: `POST /inventory/{sku}/adjust` — apply signed delta adjustments to stock
- **New**: `GET /account/usage` — API usage statistics for current billing period
- **Fixed**: Pagination `total` count was incorrect when multiple filters were applied
- **Fixed**: `order.cancelled` webhook was not fired when cancellation originated from the Dashboard

---

## v2.6.0 — @2025-12-05@

::Released::green::

- **New**: OAuth 2.0 support — Authorization Code flow for user-facing integrations
- **New**: API key scoping — restrict keys to specific resource types
- **New**: Webhook delivery history and manual retry via Dashboard
- **Improved**: `POST /webhooks/{id}/test` now accepts an optional `event_type` parameter

---

## v2.5.0 — @2025-10-22@

::Released::green::

- **New**: `X-Nexus-Signature` header on all webhook deliveries (HMAC-SHA256)
- **New**: `payment.succeeded` and `payment.failed` webhook events
- **Improved**: Webhook retry backoff intervals increased for more reliable delivery at burst traffic

---

## v2.4.0 — @2025-09-15@

::Breaking::red:: ::Released::green::

- **Breaking**: `webhook.payload.timestamp` field changed from Unix epoch (integer) to ISO 8601 string. Update your consumers before upgrading. [Migration guide →](page:"Nexus Platform Documentation")
- **New**: `order.fulfilled` webhook event
- **Deprecated**: `v1/orders` endpoint — scheduled for removal @2026-12-31@

---

## Upcoming

| Change | Version | Target date | Status |
| --- | --- | --- | --- |
| Multi-currency order support | v2.9.0 | @2026-04-01@ | ::In Progress::blue:: |
| GraphQL API (beta) | v2.9.0 | @2026-04-01@ | ::In Progress::blue:: |
| Partner API scopes | v2.10.0 | @2026-06-15@ | ::Planned::grey:: |
| v1 API end of life | — | @2026-12-31@ | ::Planned::grey:: |
