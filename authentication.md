---
title: "Authentication"
description: "How API key authentication works"
---

The TalkPilot API uses API key authentication. Every request (except `/v1/health`) must include a valid API key.

## Passing your API key

Include the key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: tp_live_YOUR_KEY_HERE" \
  https://{project_ref}.supabase.co/functions/v1/api/v1/agents
```

API keys always start with the prefix `tp_live_` followed by 32 hexadecimal characters.

## Permissions

Each API key has granular permissions that control which resources it can access:

| Permission | Allows |
|-----------|--------|
| `agents:read` | List and read agent configurations |
| `agents:write` | Update agent settings |
| `employees:read` | List and read employees |
| `employees:write` | Create, update, and delete employees |
| `tools:read` | List and read agent tools |
| `tools:write` | Create, update, and delete tools |
| `forwarding:read` | List forwarding slots |
| `forwarding:write` | Create, update, delete, and replace forwarding slots |
| `kb:read` | List and read knowledge base documents |
| `kb:write` | Create, update, and delete KB documents |
| `calls:read` | List and read call records and transcripts |
| `organization:read` | Read organization settings |
| `organization:write` | Update organization settings |

When a request requires a permission the key doesn't have, the API returns `403 Forbidden`:

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "API key lacks required permission: employees:write",
    "request_id": "req_abc123"
  }
}
```

## Agent scoping

API keys can optionally be restricted to specific agents via `allowed_agent_ids`. When set:

- **List endpoints** only return agents in the allowed list
- **Resource endpoints** (tools, employees, calls, etc.) only work for allowed agents
- Accessing a non-allowed agent returns `404 Not Found` (not `403`, to avoid leaking agent IDs)

This is useful for integrations that should only access a subset of your agents — e.g., giving a CRM integration access to one specific agent's employee data.

## Creating and managing keys

API keys are managed in the TalkPilot Dashboard under **Settings > API**:

1. **Create** — Set name, permissions, optional agent restrictions, rate limits, and expiration
2. **Activate/Deactivate** — Toggle a key without deleting it
3. **Delete** — Permanently revoke a key
4. **Monitor** — See when a key was last used

Keys are hashed (SHA-256) before storage — the full key is only shown once at creation time. If you lose a key, create a new one.

## Key expiration

API keys can have an optional expiration date. Expired keys return `401 Unauthorized`:

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "API key has expired",
    "request_id": "req_def456"
  }
}
```

## Security best practices

- **Never expose keys in client-side code** — API keys are for server-to-server communication only
- **Use the minimum required permissions** — Only grant permissions the integration actually needs
- **Scope to specific agents** when possible — Limit blast radius if a key is compromised
- **Set expiration dates** for temporary integrations
- **Rotate keys regularly** — Create a new key, update your integration, then delete the old one
- **Monitor usage** — Check the "Last used" timestamp to detect unused or suspicious keys

## Error responses

| Status | Code | Meaning |
|--------|------|---------|
| `401` | `UNAUTHORIZED` | Missing, invalid, inactive, or expired API key |
| `403` | `FORBIDDEN` | Key lacks required permission for this endpoint |
| `429` | `RATE_LIMITED` | Too many requests — see [Rate Limiting](/guides/rate-limiting) |
