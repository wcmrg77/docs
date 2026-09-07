---
title: "Organizations"
description: "Read and update organization-level settings"
---

Organization-level settings for your TalkPilot workspace.

An API key is no longer limited to a single organization. Its scope is derived from the
organization memberships of the user who created the key — a `dev_admin` who belongs to
several organizations can read and write all of them with one key.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Organization identifier |
| `name` | string | Organization name (read-only) |
| `auto_delete_done_calls` | boolean | Automatically soft-delete calls marked as done |
| `cascade_done_same_number` | boolean | Marking a call done also marks the other open calls of the same number |
| `calendar_status_update` | boolean | Derive employee status from connected calendars |
| `forward_call_email` | boolean | Allow forwarding a call by e-mail from the Dashboard |
| `transfer_popup_enabled` | boolean | Show employees a caller card with transcript on transfer |
| `notdienst_bereiche` | string[] | Selectable emergency-duty areas for employees; null/empty = a single emergency service |
| `max_concurrent_calls` | integer | Parallel inbound calls across all numbers, null = unlimited (read-only, managed by TalkPilot) |
| `concurrency_overflow_number` | string | Number overflow callers are forwarded to, null = busy tone (read-only) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

## Endpoints

### List organizations

Returns every organization you have access to.

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
  "data": [
    {
      "id": "org-uuid-1",
      "name": "Mustermann GmbH",
      "auto_delete_done_calls": false,
      "created_at": "2025-01-01T00:00:00Z"
    },
    {
      "id": "org-uuid-2",
      "name": "Beispiel AG",
      "auto_delete_done_calls": true,
      "created_at": "2025-03-14T00:00:00Z"
    }
  ]
}
```

<Note>
  This endpoint previously returned a single organization object. It now returns a
  `data` array, because one key can span multiple organizations.
</Note>

### Update organization settings

The target organization is part of the path and must be one you have access to —
otherwise the endpoint responds with `404 NOT_FOUND`.

```
PATCH /v1/organization/{organization_id}
```

**Permission:** `organization:write`

Updatable: `auto_delete_done_calls`, `cascade_done_same_number`, `calendar_status_update`, `forward_call_email`, `transfer_popup_enabled`, `notdienst_bereiche`. `name`, `max_concurrent_calls` and `concurrency_overflow_number` are read-only; unknown fields are ignored.

```bash
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"auto_delete_done_calls": true, "notdienst_bereiche": ["Elektro", "Sanitär"]}' \
  "$TP_BASE/organization/org-uuid-1"
```

## Settings

### auto_delete_done_calls

When enabled, calls that are marked as `done` (processed) are automatically moved to the trash (soft-deleted). They can still be recovered from the trash in the Dashboard or permanently deleted later.

### notdienst_bereiche

The list an employee's `notdienst_bereich` is chosen from. Values are trimmed and de-duplicated; an empty list is stored as null (single emergency service, Dashboard dropdown unchanged).

## Related resources

- [Organizations](/product/organizations) — Dashboard management guide
- [Settings](/product/settings) — Dashboard settings page
