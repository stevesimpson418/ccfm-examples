---
page_meta:
  title: Authentication
  labels:
    - nexus
    - getting-started
    - security
deploy_config:
  ci_banner: true
---

# Authentication

Nexus supports two authentication methods. Choose based on your integration type.

| Method | Best for | Token lifespan |
| --- | --- | --- |
| **API Key** | Server-to-server, backend integrations | Long-lived (manual rotation) |
| **OAuth 2.0** | User-facing integrations, third-party apps | Short-lived (auto-refresh) |

> [!warning]
> Never use API keys in client-side code (browser JavaScript, mobile apps). If your key is exposed, rotate it immediately in the dashboard and audit your access logs.

---

## API Key authentication

API keys are passed as a Bearer token in the `Authorization` header.

```bash
curl https://api.nexus.io/v2/orders \
  -H "Authorization: Bearer nxs_live_Bk3mXqZ9..."
```

### Key scopes

When creating an API key, select the minimum scopes required:

| Scope | Access granted |
| --- | --- |
| `orders:read` | Read orders and order events |
| `orders:write` | Create, update, and cancel orders |
| `inventory:read` | Read stock levels and item details |
| `inventory:write` | Update stock levels, create items |
| `webhooks:manage` | Create and delete webhook subscriptions |
| `account:read` | Read account and billing information |

> [!note]
> Read/Write scope grants all of the above. For production integrations, prefer granular scopes following the principle of least privilege.

### Key rotation

Rotate API keys:
- On a regular schedule (at least every 90 days)
- Immediately if you suspect a key has been exposed
- When a team member with key access leaves the organisation

---

## OAuth 2.0 (Authorization Code flow)

Use OAuth 2.0 when your application acts on behalf of individual Nexus users.

### Flow overview

```
1. Redirect user to Nexus authorization endpoint
2. User authenticates and grants consent
3. Nexus redirects back with an authorization code
4. Exchange code for access + refresh tokens
5. Use access token in API requests (Bearer)
6. Refresh access token when it expires (every 60 minutes)
```

### Step 1: Redirect to authorization endpoint

```
GET https://auth.nexus.io/oauth2/authorize
  ?client_id=YOUR_CLIENT_ID
  &redirect_uri=https://yourapp.com/oauth/callback
  &response_type=code
  &scope=orders:read+orders:write
  &state=RANDOM_STATE_VALUE
```

> [!warning]
> Always validate the `state` parameter in the callback to prevent CSRF attacks. Generate a random, unguessable value per session and verify it matches on return.

### Step 2: Exchange code for tokens

```bash
curl -X POST https://auth.nexus.io/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE" \
  -d "redirect_uri=https://yourapp.com/oauth/callback" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

Response:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "nxs_rt_Hk9pRq...",
  "scope": "orders:read orders:write"
}
```

### Step 3: Refresh access tokens

```bash
curl -X POST https://auth.nexus.io/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token" \
  -d "refresh_token=nxs_rt_Hk9pRq..." \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

Refresh tokens are single-use — store the new refresh token returned in each response.

---

## Security checklist

:::expand Production security checklist
- [ ] API keys stored in environment variables or secrets manager — not in code
- [ ] API keys scoped to minimum required permissions
- [ ] Key rotation scheduled (90-day maximum for production keys)
- [ ] OAuth client secret stored securely — never in client-side code
- [ ] `state` parameter validated in OAuth callback
- [ ] HTTPS enforced for all redirect URIs
- [ ] Access logs reviewed regularly for unexpected activity
:::
