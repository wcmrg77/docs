---
title: "Forwarding Slots"
description: "Configure call routing with priority-based forwarding"
---

Forwarding slots define how the AI agent routes calls to the right person. Each slot handles specific case types and has up to 3 priority levels for fallback routing.

## How routing works

1. The AI agent identifies the caller's issue during the conversation
2. It matches the issue to a slot via the `cases` field
3. The agent tries the **Priority 1** target first
4. If unavailable, falls back to **Priority 2**, then **Priority 3**
5. If all targets fail, the agent handles the call based on its configuration

Each priority is one of two things:

- the name of a **transfer tool** of the agent, `transfer_to_<name>` (lowercase, umlauts as `ae`/`oe`/`ue`, everything else as `_`, e.g. `transfer_to_melanie_bender`) — the agent transfers the call to that employee if they are available
- a **plain text** the agent reads out instead of transferring, e.g. `Allgemeines Postfach` or `Rückruf vereinbaren -> Allgemeines Postfach` — the caller is offered a callback

Slots are linked to the agent's phone number (`agent_phone_number`) automatically: the API copies it when a slot is written and updates existing slots when a pool number is [assigned](/api/agents#phone-number) or released.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Slot identifier |
| `slotnummer` | string | Two-digit slot number (`"01"`, `"02"`, etc.) |
| `cases` | string | Comma-separated case types (e.g., `"Billing, Payments"`) |
| `slot_belegung` | string | Primary target, by convention the same value as `prioritaet_1` |
| `prioritaet_1` | string | Primary target: `transfer_to_<name>` or a text to read out |
| `prioritaet_2` | string | Fallback target (same format) |
| `prioritaet_3` | string | Second fallback target (same format) |
| `kunde` | string | Free-text customer/segment label for the slot |
| `agent_id` | uuid | Parent agent |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

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
    "slot_belegung": "transfer_to_max_mustermann",
    "prioritaet_1": "transfer_to_max_mustermann",
    "prioritaet_2": "Rückruf vereinbaren -> Allgemeines Postfach"
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

Every slot is validated first (`slotnummer` and `cases` are required, other fields must be strings or null); invalid rows are rejected with `400 VALIDATION_ERROR` (`details[].field` = `slots[i].<field>`) and nothing is written. The replacement itself runs in a single database transaction.

```bash
curl -X PUT -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "slots": [
      {
        "slotnummer": "01",
        "cases": "Rechnungsfragen",
        "slot_belegung": "transfer_to_max_mustermann",
        "prioritaet_1": "transfer_to_max_mustermann",
        "prioritaet_2": "Rückruf vereinbaren -> Allgemeines Postfach"
      },
      {
        "slotnummer": "02",
        "cases": "Technischer Support",
        "slot_belegung": "transfer_to_erika_musterfrau",
        "prioritaet_1": "transfer_to_erika_musterfrau",
        "prioritaet_2": "transfer_to_max_mustermann"
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
- Use `transfer_to_<name>` only for employees that exist on the agent with a matching name (see [Tools](/api/tools) for transfer tools); anything else is read out as text

## Related resources

- [Agents](/api/agents) — Parent resource
- [Employees](/api/employees) — People referenced in slot priorities
- [Forwarding Slots](/product/forwarding-slots) — Dashboard UI guide
