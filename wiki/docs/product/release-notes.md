---
page_meta:
  title: Release Notes
  labels:
    - product
    - releases
deploy_config:
  ci_banner: true
---

# Release Notes

Release notes for Acme Corp's platform. Major and minor releases are listed here; patch releases and hotfixes are tracked in the `#deployments` Slack channel.

---

## v3.4.0 — @2026-02-18@

::Released::green::

### What's new

- **SSO / SAML 2.0**: Enterprise customers can now configure SAML-based single sign-on via the admin portal. Supports Okta, Azure AD, and Google Workspace as identity providers.
- **Bulk order API**: New `POST /v2/orders/bulk` endpoint accepts up to 500 orders in a single request. Responses include per-item status and a summary of failures.
- **Inventory webhooks**: Subscribe to `inventory.level.changed` and `inventory.item.low_stock` events. Configure endpoints in Settings → Integrations.

### Improvements

- Order search now supports filtering by `external_ref` — useful for correlating platform orders with your own order management system.
- API rate limit headers (`X-RateLimit-Remaining`, `X-RateLimit-Reset`) are now returned on all endpoints, not just rate-limited responses.
- Improved error messages for validation failures — responses now include a `field` attribute identifying the offending parameter.

### Fixes

- Fixed a race condition in the inventory reservation flow that could result in double-allocation under high concurrency.
- Resolved an issue where webhook delivery retries were not honouring the configured backoff interval.
- CSV exports now correctly escape fields containing commas.

---

## v3.3.0 — @2026-01-14@

::Released::green::

### What's new

- **Self-serve billing portal**: Customers can now update payment methods, download invoices, and manage subscription plans without contacting support.
- **API key scoping**: API keys can now be restricted to specific endpoints or resource types. Existing keys retain full access.

### Improvements

- Dashboard load time reduced by 40% following query optimisation on the orders summary endpoint.
- Webhook payloads now include a `X-Acme-Signature` header for request verification (HMAC-SHA256).

### Fixes

- Fixed pagination returning incorrect `total` count when filters were applied.
- Resolved a memory leak in the notification service affecting long-running workers.

> [!warning]
> v3.3.0 introduced a breaking change to the webhook payload schema — the `timestamp` field is now ISO 8601 (was Unix epoch). Update your consumers before upgrading. See the [migration guide](page:"Webhook Migration Guide") for details.

---

## v3.2.1 — @2025-12-20@

::Released::green::

Hotfix release addressing a critical bug in the authentication service that caused intermittent 401 errors for valid API keys after a token cache flush. All customers on v3.2.0 are advised to upgrade.

---

## v3.2.0 — @2025-12-10@

::Released::green::

### What's new

- **Order Service v2**: Fully rewritten order processing engine. Throughput increased 3×; p99 latency reduced from 480ms to 145ms.
- **Event streaming**: Order lifecycle events (`order.created`, `order.fulfilled`, `order.cancelled`) are now available as webhook subscriptions.

---

## Upcoming

| Version | Target date | Headline feature | Status |
| --- | --- | --- | --- |
| v3.5.0 | @2026-04-01@ | Multi-currency support | ::In Progress::blue:: |
| v3.6.0 | @2026-06-15@ | Partner API public beta | ::Planned::grey:: |
