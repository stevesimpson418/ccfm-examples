---
page_meta:
  title: Product Roadmap
  labels:
    - product
    - roadmap
deploy_config:
  ci_banner: true
---

# Product Roadmap

Last updated: @2026-03-01@

This roadmap covers Acme Corp's product initiatives across the current and next two quarters. Status reflects the latest snapshot from sprint planning — check with the Product team for real-time updates.

> [!info]
> Roadmap items marked ::In Progress::blue:: have an active squad assigned. Items marked ::Planned::grey:: are committed but not yet started. ::Discovery::purple:: items are being scoped.

---

## Q1 2026 (Jan – Mar)

| Initiative | Squad | Status | Owner |
| --- | --- | --- | --- |
| Bulk order management API | Orders | ::Done::green:: | Sarah K. |
| Real-time inventory webhooks | Platform | ::Done::green:: | Priya M. |
| SSO / SAML 2.0 integration | Identity | ::In Progress::blue:: | Jordan L. |
| Self-serve billing portal | Finance Eng | ::In Progress::blue:: | Marcus T. |
| Analytics dashboard v2 | Data | ::Review::yellow:: | Anika R. |
| Mobile app (iOS beta) | Mobile | ::In Progress::blue:: | Dev Patel |

---

## Q2 2026 (Apr – Jun)

| Initiative | Squad | Status | Owner |
| --- | --- | --- | --- |
| Multi-currency support | Orders + Billing | ::Planned::grey:: | Sarah K. |
| Inventory forecasting ML | Data | ::Planned::grey:: | Anika R. |
| Partner API (public beta) | Platform | ::Discovery::purple:: | Priya M. |
| Audit log export (SIEM) | Identity | ::Planned::grey:: | Jordan L. |
| Native mobile notifications | Mobile | ::Discovery::purple:: | Dev Patel |

---

## Q3 2026 (Jul – Sep)

| Initiative | Squad | Status | Owner |
| --- | --- | --- | --- |
| EU data residency | Platform | ::Discovery::purple:: | Priya M. |
| AI-assisted order routing | Orders | ::Discovery::purple:: | Sarah K. |
| Partner marketplace | Platform | ::Discovery::purple:: | TBD |

---

## Recently shipped

:::expand Q4 2025 highlights
| Initiative | Shipped | Impact |
| --- | --- | --- |
| Order Service v2 rewrite | @2025-12-10@ | 3× throughput improvement, p99 latency < 150ms |
| Stripe billing integration | @2025-11-20@ | Replaced legacy payment processor |
| Kafka event schema registry | @2025-10-30@ | Eliminated schema drift incidents |
| Two-factor authentication | @2025-10-05@ | Mandatory for admin accounts |
:::

> [!note]
> To request a new roadmap item, open a Product Request in Jira using the `PROD` project. Include a one-paragraph problem statement and a rough estimate of customer impact.
