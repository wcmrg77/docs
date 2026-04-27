---
title: "Error Handling"
description: "Understand error responses and handle them gracefully"
---

The TalkPilot API uses standard HTTP status codes and returns structured error responses to help you handle failures gracefully.

## Error response format

Every error response follows this structure:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable description of what went wrong",
    "details": [],
    "request_id": "req_abc123"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Machine-readable error code (see table below) |
| `message` | string | Human-readable description |
| `details` | array | Field-level validation errors (only for `VALIDATION_ERROR`) |
| `request_id` | string | Unique request ID for debugging — include this when contacting support |

## HTTP status codes

| Status | Code | Description |
|--------|------|-------------|
| `400` | `VALIDATION_ERROR` | Request body failed validation |
| `400` | `BAD_REQUEST` | Malformed request (invalid JSON, missing headers, etc.) |
| `401` | `UNAUTHORIZED` | Missing, invalid, inactive, or expired API key |
| `403` | `FORBIDDEN` | API key lacks required permission |
| `404` | `NOT_FOUND` | Resource does not exist or is not accessible |
| `409` | `CONFLICT` | Resource already exists (e.g., duplicate tool name) |
| `429` | `RATE_LIMITED` | Too many requests — see [Rate Limiting](/rate-limiting) |
| `500` | `INTERNAL_ERROR` | Server error — retry or contact support |

## Validation errors

When a request fails validation, the `details` array contains field-level errors:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      { "field": "name", "message": "Required" },
      { "field": "phone_number", "message": "Must be a valid E.164 phone number" },
      { "field": "status", "message": "Must be one of: anwesend, urlaub, krank, weiterbildung, notdienst" }
    ],
    "request_id": "req_abc123"
  }
}
```

## Handling errors in code

### JavaScript / TypeScript

```javascript
async function makeApiRequest(url, options = {}) {
  const response = await fetch(url, {
    ...options,
    headers: {
      "X-API-Key": API_KEY,
      "Content-Type": "application/json",
      ...options.headers,
    },
  });

  if (!response.ok) {
    const body = await response.json();
    const error = body.error;

    switch (response.status) {
      case 401:
        throw new Error(`Authentication failed: ${error.message}`);
      case 403:
        throw new Error(`Permission denied: ${error.message}`);
      case 404:
        return null; // Resource not found
      case 429:
        // Wait and retry
        const retryAfter = response.headers.get("Retry-After") || 60;
        await new Promise((r) => setTimeout(r, retryAfter * 1000));
        return makeApiRequest(url, options);
      default:
        throw new Error(`API error [${error.code}]: ${error.message} (${error.request_id})`);
    }
  }

  return response.json();
}
```

### Python

```python
import requests
import time

def make_api_request(url, method="GET", data=None):
    response = requests.request(
        method, url,
        headers={"X-API-Key": API_KEY, "Content-Type": "application/json"},
        json=data,
    )

    if response.ok:
        return response.json() if response.content else None

    error = response.json().get("error", {})

    if response.status_code == 429:
        retry_after = int(response.headers.get("Retry-After", 60))
        time.sleep(retry_after)
        return make_api_request(url, method, data)

    if response.status_code == 404:
        return None

    raise Exception(
        f"API error [{error.get('code')}]: {error.get('message')} "
        f"(request_id: {error.get('request_id')})"
    )
```

## Debugging tips

1. **Check the `request_id`** — Include it when contacting support for faster resolution
2. **Read `details` for validation errors** — They tell you exactly which fields failed and why
3. **Check rate limit headers** — If you're getting `429`s, inspect `X-RateLimit-Remaining` before retrying
4. **Verify permissions** — `403` errors always include the missing permission in the message
