---
title: "API Keys"
description: "Manage API keys, permissions, and agent scoping"
---

API key management endpoints. These endpoints require **Supabase JWT authentication** (Dashboard login), not API key authentication.

## Key concepts

- API keys are created and managed through the Dashboard or these endpoints
- The raw key (`tp_live_...`) is returned **only once** at creation time — store it securely
- Keys are SHA-256 hashed before storage and cannot be retrieved later
- A key carries configurable permissions and agent access. Its **organization** field is only the home organization used for key management — at request time, a key's data scope is resolved from the current organization memberships of the user who created it
- Key management is restricted to the `dev_admin` role, and only via JWT — a key cannot manage keys. Other roles (including `client_admin`) receive `403 FORBIDDEN`

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Key identifier |
| `name` | string | Display name for the key |
| `key_prefix` | string | First 10 characters of the key (`tp_live_xy`), for identification |
| `organization_id` | uuid | Home organization of the key |
| `organization_name` | string | Display name of that organization (null if unresolvable) |
| `created_by_email` | string | Email of the creator (null for legacy keys without a creator) |
| `is_own` | boolean | Whether the requesting user created this key |
| `permissions` | array | List of granted permissions |
| `allowed_agent_ids` | array | Agent UUIDs this key can access (null = all) |
| `rate_limit_per_minute` | integer | Custom rate limit per minute |
| `rate_limit_per_hour` | integer | Custom rate limit per hour |
| `is_active` | boolean | Whether the key is active |
| `last_used_at` | datetime | Last request timestamp |
| `expires_at` | datetime | Expiration date (null = never) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

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
| `contacts:read` | List and read contacts (customer database) |
| `contacts:write` | Create, update, delete contacts |

## Endpoints

### List API keys

```
GET /v1/api-keys
```

**Auth:** JWT only

Returns keys the caller is allowed to manage. Key values are never returned — only the prefix.

- `dev_admin` — only keys they created themselves, across every organization those keys belong to

The same scope applies to `PATCH` and `DELETE`: a key outside it responds `404 NOT_FOUND`, whether it does not exist or belongs to someone else.

### Create API key

```
POST /v1/api-keys
```

**Auth:** JWT only

```json
{
  "name": "n8n Production",
  "permissions": ["agents:read", "agents:write", "employees:read", "employees:write"],
  "organization_id": "org-uuid-1",
  "allowed_agent_ids": null,
  "rate_limit_per_minute": 60,
  "expires_at": null
}
```

`organization_id` is optional: it sets the key's home organization and must be one of your memberships (otherwise `400 VALIDATION_ERROR`). Without it, your profile organization is used.

Response includes the raw key (**shown only once**):

```json
{
  "key": "tp_live_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
  "id": "key-uuid",
  "name": "n8n Production",
  "permissions": ["agents:read", "agents:write", "employees:read", "employees:write"],
  "allowed_agent_ids": null,
  "rate_limit_per_minute": 60,
  "rate_limit_per_hour": 1000,
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
