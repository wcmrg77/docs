---
title: "API Keys"
description: "Manage API keys, permissions, and agent scoping"
---

API key management endpoints. These endpoints require **Supabase JWT authentication** (Dashboard login), not API key authentication.

## Key concepts

- API keys are created and managed through the Dashboard or these endpoints
- The raw key (`tp_live_...`) is returned **only once** at creation time — store it securely
- Keys are SHA-256 hashed before storage and cannot be retrieved later
- Each key is scoped to one organization with configurable permissions and agent access

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Key identifier |
| `name` | string | Display name for the key |
| `key_prefix` | string | First characters of the key (for identification) |
| `permissions` | array | List of granted permissions |
| `allowed_agent_ids` | array | Agent UUIDs this key can access (null = all) |
| `rate_limit_per_minute` | integer | Custom rate limit per minute |
| `rate_limit_per_hour` | integer | Custom rate limit per hour |
| `is_active` | boolean | Whether the key is active |
| `last_used_at` | datetime | Last request timestamp |
| `expires_at` | datetime | Expiration date (null = never) |
| `created_at` | datetime | Creation timestamp |

## Available permissions

| Permission | Allows |
|-----------|--------|
| `agents:read` | List and read agent configurations |
| `agents:write` | Update agent settings |
| `employees:read` | List and read employees |
| `employees:write` | Create, update, delete employees |
| `tools:read` | List and read agent tools |
| `tools:write` | Create, update, delete tools |
| `forwarding:read` | List forwarding slots |
| `forwarding:write` | Create, update, delete, replace slots |
| `kb:read` | List and read knowledge base documents |
| `kb:write` | Create, update, delete KB documents |
| `calls:read` | List and read call records |
| `organization:read` | Read organization settings |
| `organization:write` | Update organization settings |

## Endpoints

### List API keys

```
GET /v1/api-keys
```

**Auth:** JWT only

Returns all keys for the current user's organization. Key values are never returned — only the prefix.

### Create API key

```
POST /v1/api-keys
```

**Auth:** JWT only

```json
{
  "name": "n8n Production",
  "permissions": ["agents:read", "agents:write", "employees:read", "employees:write"],
  "rate_limit_per_minute": 60,
  "expires_at": null
}
```

Response includes the raw key (**shown only once**):

```json
{
  "key": "tp_live_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
  "id": "key-uuid",
  "name": "n8n Production",
  "permissions": ["agents:read", "agents:write", "employees:read", "employees:write"],
  "created_at": "2026-03-22T10:00:00Z"
}
```

### Update API key

```
PATCH /v1/api-keys/{keyId}
```

**Auth:** JWT only

Update name, permissions, rate limits, or active status. The key value cannot be changed.

```bash
# Restrict to read-only
curl -X PATCH -H "Authorization: Bearer $JWT" -H "Content-Type: application/json" \
  -d '{"name": "n8n Read-Only", "permissions": ["agents:read", "employees:read", "calls:read"]}' \
  "$TP_BASE/api-keys/{keyId}"
```

### Revoke API key

```
DELETE /v1/api-keys/{keyId}
```

**Auth:** JWT only | Returns `204 No Content`

Permanently deletes the key. Any requests using this key will immediately receive `401 Unauthorized`.

## Key rotation workflow

1. Create a new key with the same permissions
2. Update your integration to use the new key
3. Verify the integration works
4. Delete the old key

## Related resources

- [Authentication](/authentication) — How API key auth works
- [Settings](/product/settings) — Dashboard key management UI
