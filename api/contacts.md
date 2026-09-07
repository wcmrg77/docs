---
title: "Contacts"
description: "Manage the customer database the agent recognises callers from"
---

Contacts are the agent's customer database. When a call comes in, the voice agent looks the caller up by number (`phone_number` or `mobile_number`) and uses the record to greet the caller by name and to pick a tag-specific greeting from the agent's `greeting_variants` (see [Agents](/api/agents)). Contacts are also what the Dashboard shows under *Kontakte*.

Contacts belong to one agent. A key needs `contacts:read` / `contacts:write` and access to that agent.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Contact identifier |
| `agent_id` | uuid | Owning agent (read-only) |
| `customer_name` | string | Display name. Composed from the parts when they are sent |
| `salutation` | string | `Herr`, `Frau`, … |
| `first_name` | string | First name |
| `last_name` | string | Last name. When the name could not be split, the full original name is kept here |
| `name_parsed` | string | Read-only: `manual` (parts sent explicitly), `auto` (split unambiguously from `customer_name`), `llm` (split by the backfill), `unclear` (not split). **The agent greets by name only when this is not `unclear`** |
| `name_source` | string | Read-only: `manual` for names maintained by a person (API or Dashboard) — the AI never overwrites those; `ai` for names captured from calls |
| `phone_number` | string | Primary number, E.164. The agent matches callers against this and `mobile_number`. Derived from `mobile_number` (preferred) or `landline_number` whenever either is sent |
| `mobile_number` | string | Mobile number, E.164 |
| `landline_number` | string | Landline number, E.164 |
| `email` | string | E-mail address |
| `address` | string | Postal address (free text) |
| `company` | string | Company |
| `tags` | string[] | Free tags, trimmed and de-duplicated; empty → `null`. Used by `greeting_variants` |
| `note` | string | Free note |
| `customer_number` | number | Your customer number |
| `last_contact_date` | datetime | Last contact (ISO 8601) |
| `created_at` / `updated_at` | datetime | Timestamps |

All phone numbers must be E.164 (`+4922112345678`). Spaces, dashes and parentheses are stripped; anything else is rejected with `400` — the agent compares the exact string, so a national format would never match a caller.

## Name handling

The rules are the same as in the Dashboard, so contacts look identical no matter where they were created:

- **Parts sent** (`salutation`, `first_name`, `last_name`): `customer_name` is composed from them (`"Herr Fey"`, `"Tobias Lutz"`), `name_parsed` becomes `manual`.
- **Only `customer_name` sent**: it is split where unambiguous (`"Tobias Lutz"` → first/last, `"Herr Fey"` → salutation/last). Company names, two people in one field or anything ambiguous stay unsplit: the full name goes to `last_name` and `name_parsed` is `unclear`, so the agent greets neutrally instead of guessing.
- **`customer_name: null`** on update clears every name field.
- On update, parts are merged with the stored contact — `{"first_name": "Max"}` keeps salutation and last name.
- A created or changed name sets `name_source` to `manual`.

## Endpoints

### List contacts

```
GET /v1/agents/{agentId}/contacts
```

**Permission:** `contacts:read` | **Pagination:** yes | newest first

**Filters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `phone_number` | string | Exact E.164 number, matched against `phone_number`, `mobile_number` and `landline_number` (`400` if not E.164) |
| `tag` | string | Only contacts carrying this tag (exact match) |
| `search` | string | Case-insensitive substring search over name, company, e-mail and the number fields |

```bash
# Who is calling? (same lookup the agent does)
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/contacts?phone_number=%2B491711234567"

# All VIPs
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/contacts?tag=vip"
```

### Get contact

```
GET /v1/agents/{agentId}/contacts/{contactId}
```

**Permission:** `contacts:read`

### Create contact

```
POST /v1/agents/{agentId}/contacts
```

**Permission:** `contacts:write` — all fields optional

```bash
# Greeted as "Herr Fey"
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "salutation": "Herr",
    "last_name": "Fey",
    "mobile_number": "+491711234567",
    "email": "fey@example.com",
    "company": "Fey GmbH",
    "tags": ["vip"]
  }' \
  "$TP_BASE/agents/{agentId}/contacts"

# Display name only — split automatically ("Tobias" / "Lutz", name_parsed: auto)
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"customer_name": "Tobias Lutz", "landline_number": "+492215550"}' \
  "$TP_BASE/agents/{agentId}/contacts"
```

### Update contact

```
PATCH /v1/agents/{agentId}/contacts/{contactId}
```

**Permission:** `contacts:write` — partial update, unknown and read-only fields are ignored

```bash
# Tag a contact so the agent uses the "vip" greeting variant
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"tags": ["vip"]}' \
  "$TP_BASE/agents/{agentId}/contacts/{contactId}"

# New mobile number — phone_number follows automatically
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"mobile_number": "+491719876543"}' \
  "$TP_BASE/agents/{agentId}/contacts/{contactId}"
```

### Delete contact

```
DELETE /v1/agents/{agentId}/contacts/{contactId}
```

**Permission:** `contacts:write` | Returns `204 No Content`, `404` for unknown IDs

## Common patterns

### Sync customers from a CRM

```javascript
for (const customer of crmCustomers) {
  const { data } = await talkpilot(
    `/agents/${agentId}/contacts?phone_number=${encodeURIComponent(customer.mobile)}`
  );
  const body = {
    salutation: customer.salutation,
    first_name: customer.firstName,
    last_name: customer.lastName,
    mobile_number: customer.mobile,
    customer_number: customer.id,
    tags: customer.isKeyAccount ? ["vip"] : null,
  };
  if (data.length === 0) {
    await talkpilot(`/agents/${agentId}/contacts`, { method: "POST", body: JSON.stringify(body) });
  } else {
    await talkpilot(`/agents/${agentId}/contacts/${data[0].id}`, { method: "PATCH", body: JSON.stringify(body) });
  }
}
```

## Related resources

- [Agents](/api/agents) — `greeting_variants` pick a greeting per contact tag
- [Calls](/api/calls) — call records carry the caller's number and name
- [CRM Integration](/use-cases/crm-integration) — end-to-end example
