---
page_meta:
  title: Rate Limiting
  labels:
    - nexus
    - guides
    - rate-limiting
deploy_config:
  ci_banner: true
---

# Rate Limiting

All Nexus API requests are rate-limited per API key to ensure fair usage and platform stability.

---

## Rate limit tiers

| Plan | Requests / minute | Requests / day | Burst allowance |
| --- | --- | --- | --- |
| Developer | 60 | 1,000 | Up to 20 req/s for 5 seconds |
| Growth | 300 | 50,000 | Up to 100 req/s for 5 seconds |
| Enterprise | 2,000 | Unlimited | Up to 500 req/s for 10 seconds |

> [!warning]
> The Developer plan is for testing only. Do not use a Developer plan key in a production application — you will exhaust the daily limit quickly under real traffic.

---

## Rate limit headers

Every API response includes rate limit headers:

| Header | Description |
| --- | --- |
| `X-RateLimit-Limit` | Your plan's per-minute request limit |
| `X-RateLimit-Remaining` | Requests remaining in the current window |
| `X-RateLimit-Reset` | Unix timestamp when the window resets |
| `Retry-After` | Seconds to wait (only present on 429 responses) |

```bash
HTTP/2 200
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 247
X-RateLimit-Reset: 1740841260
```

---

## Handling rate limit errors

When you exceed the limit, you'll receive:

```
HTTP 429 Too Many Requests
```

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Retry after 14 seconds.",
    "status": 429,
    "retry_after": 14
  }
}
```

### Recommended backoff strategy

```python
import time
import requests

def nexus_request_with_backoff(url, headers, max_retries=5):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)

        if response.status_code == 429:
            retry_after = int(response.headers.get("Retry-After", 10))
            print(f"Rate limited. Waiting {retry_after}s before retry {attempt + 1}/{max_retries}")
            time.sleep(retry_after)
            continue

        return response

    raise Exception("Max retries exceeded")
```

---

## Bulk endpoints

For high-volume use cases, prefer bulk endpoints over individual calls:

| Use case | Avoid | Use instead |
| --- | --- | --- |
| Creating many orders | `POST /orders` in a loop | `POST /orders/bulk` (up to 500 per call) |
| Reading order lists | Repeated `GET /orders/{id}` | `GET /orders` with filters + pagination |

:::expand Bulk request limits
- Maximum 500 orders per `POST /orders/bulk` request
- Bulk requests count as a single request toward your rate limit
- Individual items within a bulk request may fail independently — check the `results` array in the response for per-item status
:::

> [!note]
> Enterprise customers can request a rate limit increase for specific endpoints. Contact your account manager with a description of your use case and expected request volume.
