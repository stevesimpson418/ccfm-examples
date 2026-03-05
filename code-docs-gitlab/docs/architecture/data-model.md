---
page_meta:
  title: Data Model
  labels:
    - order-service
    - architecture
    - database
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# Data Model

The Order Service uses PostgreSQL (RDS) as its primary store. All tables are in the `orders` schema. Migrations are managed with Alembic.

---

## Entities

### Order

The central entity. One row per order. Status transitions are enforced at the application layer and recorded in `order_events`.

| Column | Type | Nullable | Description |
| --- | --- | --- | --- |
| `id` | `uuid` | No | Primary key (UUIDv4) |
| `external_ref` | `varchar(255)` | Yes | Caller-supplied correlation ID (unique per account) |
| `account_id` | `uuid` | No | FK → `accounts.id` |
| `status` | `order_status` (enum) | No | Current order status |
| `currency` | `char(3)` | No | ISO 4217 currency code |
| `total_amount` | `integer` | No | Total in smallest currency unit |
| `shipping_address_id` | `uuid` | No | FK → `addresses.id` |
| `cancelled_reason` | `text` | Yes | Populated when status = cancelled |
| `created_at` | `timestamptz` | No | |
| `updated_at` | `timestamptz` | No | Updated on any status change |
| `fulfilled_at` | `timestamptz` | Yes | Populated when status = delivered |

**Status enum values**: `pending`, `confirmed`, `processing`, `shipped`, `delivered`, `cancelled`, `failed`

---

### OrderItem

Line items within an order. Immutable after order confirmation.

| Column | Type | Nullable | Description |
| --- | --- | --- | --- |
| `id` | `uuid` | No | Primary key |
| `order_id` | `uuid` | No | FK → `orders.id` |
| `sku` | `varchar(64)` | No | Product SKU from Inventory Service |
| `quantity` | `integer` | No | Must be ≥ 1 |
| `unit_price` | `integer` | No | Price at time of order (snapshot) |
| `line_total` | `integer` | No | `quantity × unit_price` |

> [!note]
> `unit_price` is snapshotted at order creation time and does not change if the catalogue price changes later. This ensures accurate billing regardless of downstream price updates.

---

### OrderEvent

Append-only audit log of all order state transitions. Never updated or deleted.

| Column | Type | Nullable | Description |
| --- | --- | --- | --- |
| `id` | `uuid` | No | Primary key |
| `order_id` | `uuid` | No | FK → `orders.id` |
| `event_type` | `varchar(64)` | No | e.g. `order.created`, `order.status.changed` |
| `previous_status` | `order_status` | Yes | Null on creation events |
| `new_status` | `order_status` | Yes | Null for non-transition events |
| `actor` | `varchar(255)` | No | API key ID or system identifier |
| `metadata` | `jsonb` | Yes | Event-specific additional data |
| `occurred_at` | `timestamptz` | No | Wall clock time of the event |

---

### Address

Shipping addresses. Stored independently to allow future reuse and address book features.

| Column | Type | Nullable | Description |
| --- | --- | --- | --- |
| `id` | `uuid` | No | Primary key |
| `line1` | `varchar(255)` | No | |
| `line2` | `varchar(255)` | Yes | |
| `city` | `varchar(100)` | No | |
| `state_province` | `varchar(100)` | Yes | Required for US, AU, CA |
| `postcode` | `varchar(20)` | No | |
| `country` | `char(2)` | No | ISO 3166-1 alpha-2 |

---

## Indexes

| Table | Index | Columns | Purpose |
| --- | --- | --- | --- |
| `orders` | `idx_orders_account_status` | `account_id, status` | List orders by status |
| `orders` | `idx_orders_external_ref` | `account_id, external_ref` | Deduplication lookup |
| `orders` | `idx_orders_created_at` | `created_at DESC` | Recency-sorted list queries |
| `order_items` | `idx_items_order_id` | `order_id` | Join from orders |
| `order_events` | `idx_events_order_id_time` | `order_id, occurred_at DESC` | Event history lookup |

---

## Migration policy

> [!warning]
> All migrations must be backwards-compatible with the currently deployed version. Zero-downtime deployments require that the old code continues to function while the migration runs.

Approved patterns:
- Adding nullable columns
- Adding new tables
- Adding indexes (`CONCURRENTLY`)
- Widening `varchar` lengths

Patterns requiring a deploy gate:
- Dropping columns (add `deploy_config: deploy_page: false` to the column first, remove in the next deploy)
- Renaming columns (use a view or alias in the interim)
- Changing column types
