---
title: "Organization"
description: "Read and update organization-level settings"
---

Organization-level settings for your TalkPilot workspace. Each API key is scoped to one organization.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Organization identifier |
| `name` | string | Organization name |
| `auto_delete_done_calls` | boolean | Automatically soft-delete calls marked as done |
| `created_at` | datetime | Creation timestamp |

## Endpoints

### Get organization settings

```
GET /v1/organization
```

**Permission:** `organization:read`

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/organization"
```

Response:

```json
{
  "id": "org-uuid-1",
  "name": "Mustermann GmbH",
  "auto_delete_done_calls": false,
  "created_at": "2025-01-01T00:00:00Z"
}
```

### Update organization settings

```
PATCH /v1/organization
```

**Permission:** `organization:write`

```bash
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"auto_delete_done_calls": true}' \
  "$TP_BASE/organization"
```

## Settings

### auto_delete_done_calls

When enabled, calls that are marked as `done` (processed) are automatically moved to the trash (soft-deleted). They can still be recovered from the trash in the Dashboard or permanently deleted later.

## Related resources

- [Organizations](/product/organizations) — Dashboard management guide
- [Settings](/product/settings) — Dashboard settings page
