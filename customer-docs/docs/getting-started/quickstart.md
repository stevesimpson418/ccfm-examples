---
page_meta:
  title: Quick Start
  labels:
    - nexus
    - getting-started
deploy_config:
  ci_banner: true
---

# Quick Start

Make your first Nexus API call in under 5 minutes.

> [!info]
> You'll need a Nexus account and an API key. Sign up at [dashboard.nexus.io](https://dashboard.nexus.io) — it's free for development and test environments.

---

## Step 1: Create an account

1. Go to [dashboard.nexus.io/signup](https://dashboard.nexus.io/signup)
2. Enter your work email and create a password
3. Verify your email address (check your spam folder if you don't receive it within 2 minutes)
4. Select your plan — choose **Developer** for free sandbox access

## Step 2: Generate an API key

1. In the Nexus Dashboard, navigate to **Settings → API Keys**
2. Click **Create new key**
3. Give it a name (e.g. `local-dev`) and select **Read/Write** scope
4. Copy the key — it will only be shown once

> [!warning]
> Treat your API key like a password. Never commit it to source control or share it in Slack. Use environment variables or a secrets manager.

```bash
export NEXUS_API_KEY="nxs_live_..."
```

## Step 3: Make your first request

Verify your key works by fetching your account details:

```bash
curl https://api.nexus.io/v2/account \
  -H "Authorization: Bearer $NEXUS_API_KEY" \
  -H "Content-Type: application/json"
```

Expected response:

```json
{
  "id": "acc_01h8xqz3k...",
  "name": "Acme Widgets Ltd",
  "plan": "developer",
  "environment": "sandbox",
  "created_at": "2026-01-15T09:30:00Z"
}
```

## Step 4: Create your first order

```bash
curl -X POST https://api.nexus.io/v2/orders \
  -H "Authorization: Bearer $NEXUS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "external_ref": "order-001",
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
  }'
```

A successful response returns `201 Created` with the order object, including its Nexus ID (`ord_...`).

## Step 5: Check the order status

```bash
curl https://api.nexus.io/v2/orders/ord_01h8xr... \
  -H "Authorization: Bearer $NEXUS_API_KEY"
```

---

## Next steps

- Set up [Authentication](page:"Authentication") for production — consider OAuth 2.0 for user-facing integrations
- Browse the full [API Reference Overview](page:"API Reference Overview")
- Configure [webhooks](page:"Webhook Integration Guide") to receive real-time order events
