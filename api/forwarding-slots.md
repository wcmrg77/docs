---
title: "Forwarding Slots"
description: "Configure call routing with priority-based forwarding"
---

Forwarding slots define how the AI agent routes calls to the right person. Each slot handles specific case types and has up to 3 priority levels for fallback routing.

## How routing works

1. The AI agent identifies the caller's issue during the conversation
2. It matches the issue to a slot via the `cases` field
3. The agent tries the **Priority 1** contact first
4. If unavailable, falls back to **Priority 2**, then **Priority 3**
5. If all contacts fail, the agent handles the call based on its configuration

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Slot identifier |
| `slotnummer` | string | Two-digit slot number (`"01"`, `"02"`, etc.) |
| `cases` | string | Comma-separated case types (e.g., `"Billing, Payments"`) |
| `slot_belegung` | string | Name of the person assigned to this slot |
| `prioritaet_1` | string | Primary contact (phone number or name) |
| `prioritaet_2` | string | Secondary fallback contact |
| `prioritaet_3` | string | Tertiary fallback contact |
| `agent_id` | uuid | Parent agent |

## Endpoints

### List forwarding slots

```
GET /v1/agents/{agentId}/forwarding-slots
```

**Permission:** `forwarding:read` | **Pagination:** no (returns all, ordered by slot number)

### Create forwarding slot

```
POST /v1/agents/{agentId}/forwarding-slots
```

**Permission:** `forwarding:write`

**Required fields:** `slotnummer`, `cases`

```bash
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "slotnummer": "01",
    "cases": "Rechnungsfragen, Zahlungsprobleme",
    "slot_belegung": "Max Mustermann",
    "prioritaet_1": "+491701234567",
    "prioritaet_2": "+491709876543"
  }' \
  "$TP_BASE/agents/{agentId}/forwarding-slots"
```

### Update forwarding slot

```
PATCH /v1/agents/{agentId}/forwarding-slots/{slotId}
```

**Permission:** `forwarding:write`

### Delete forwarding slot

```
DELETE /v1/agents/{agentId}/forwarding-slots/{slotId}
```

**Permission:** `forwarding:write` | Returns `204 No Content`

### Replace all forwarding slots (bulk)

```
PUT /v1/agents/{agentId}/forwarding-slots
```

**Permission:** `forwarding:write`

Replaces **all** existing slots with the provided list in a single operation. Existing slots are deleted first. Use this for bulk synchronization from external systems.

```bash
curl -X PUT -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "slots": [
      {
        "slotnummer": "01",
        "cases": "Rechnungsfragen",
        "slot_belegung": "Max Mustermann",
        "prioritaet_1": "+491701234567"
      },
      {
        "slotnummer": "02",
        "cases": "Technischer Support",
        "slot_belegung": "Erika Musterfrau",
        "prioritaet_1": "+491705555555",
        "prioritaet_2": "+491706666666"
      }
    ]
  }' \
  "$TP_BASE/agents/{agentId}/forwarding-slots"
```

Response includes `replaced_count` — the number of previous slots that were removed:

```json
{
  "data": [ ... ],
  "replaced_count": 3
}
```

## Best practices

- Use `PUT` for full sync from external systems, `POST`/`PATCH`/`DELETE` for individual changes
- Slot numbers should be sequential two-digit strings: `"01"`, `"02"`, `"03"`
- Keep `cases` descriptive — the AI uses this text to match caller issues

## Related resources

- [Agents](/agents) — Parent resource
- [Employees](/employees) — People referenced in slot priorities
- [Forwarding Slots](/product/forwarding-slots) — Dashboard UI guide
