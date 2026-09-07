---
title: "Rate Limiting"
description: "Respect rate limits and handle 429 responses"
---

The TalkPilot API enforces per-key rate limits to ensure fair usage and system stability.

## Default limits

| Window | Limit |
|--------|-------|
| Per minute | 60 requests |
| Per hour | 1,000 requests |

Both limits are enforced and can be customized per API key when creating the key in the Dashboard.

## Rate limit headers

Every response to a request authenticated with an API key includes these headers (requests authenticated with a Dashboard JWT are not rate limited and carry no headers):

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests per minute for this key |
| `X-RateLimit-Remaining` | Requests remaining in the current minute window |
| `X-RateLimit-Reset` | Unix timestamp roughly 60 seconds in the future (the window is a sliding minute) |

Example response headers:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1711108920
```

## When you hit the limit

When rate limited, the API returns `429 Too Many Requests` with `Retry-After: 60` (minute window) or `Retry-After: 3600` (hour window):

```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1711108920
```

```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded. Try again in 60 seconds.",
    "request_id": "req_abc123"
  }
}
```

## Handling rate limits

### Proactive: Check headers before hitting the limit

```javascript
async function apiRequest(url, options = {}) {
  const response = await fetch(url, {
    ...options,
    headers: { "X-API-Key": API_KEY, ...options.headers },
  });

  // Track remaining quota
  const remaining = parseInt(response.headers.get("X-RateLimit-Remaining") || "0");
  const resetAt = parseInt(response.headers.get("X-RateLimit-Reset") || "0");

  if (remaining < 5) {
    console.warn(`Rate limit low: ${remaining} requests remaining, resets at ${new Date(resetAt * 1000)}`);
  }

  return response;
}
```

### Reactive: Retry on 429

```javascript
async function apiRequestWithRetry(url, options = {}, maxRetries = 3) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    const response = await fetch(url, {
      ...options,
      headers: { "X-API-Key": API_KEY, ...options.headers },
    });

    if (response.status !== 429) return response;

    const retryAfter = parseInt(response.headers.get("Retry-After") || "60");
    console.log(`Rate limited. Retrying in ${retryAfter}s (attempt ${attempt + 1}/${maxRetries})`);
    await new Promise((r) => setTimeout(r, retryAfter * 1000));
  }

  throw new Error("Max retries exceeded due to rate limiting");
}
```

## Tips for staying within limits

- **Batch reads with pagination** — Use `limit=100` to fetch more data per request
- **Use `PUT /forwarding-slots`** to replace all slots in one request instead of multiple creates/deletes
- **Cache responses** when possible — Agent configurations change infrequently
- **Use the status endpoint** (`GET /agents/{id}/status`) instead of the full agent endpoint when you only need active/schedule status
- **Spread requests over time** — Avoid burst patterns; space requests evenly across the minute window
- **Request higher limits** — If your integration needs more throughput, set custom rate limits when creating the API key in the Dashboard

## Rate limit scoping

Rate limits are tracked **per API key**, not per IP address. If you have multiple integrations, create separate API keys so they each get their own quota.
