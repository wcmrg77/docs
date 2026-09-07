---
title: "Calls"
description: "Read call records, transcripts, and audio URLs"
---

Calls are read-only records created by the system when an agent handles a phone call. The API provides two levels of detail: a summary view (list) and a full detail view (individual call).

## List vs detail view

| Field | List | Detail |
|-------|:----:|:------:|
| Basic info (name, phone, duration, status) | yes | yes |
| `call_summary` | yes | yes |
| `transcript` | — | yes |
| `audio_file_url` | — | yes |
| `final_score` | — | yes |

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Call record identifier |
| `call_id` | string | LiveKit call identifier |
| `customer_name` | string | Caller's name (if tracked) |
| `customer_request` | string | Subject of the call |
| `phone_number` | string | Caller's phone number |
| `duration` | integer | Call duration in seconds |
| `done` | boolean | Whether the call has been processed |
| `general_success` | boolean | Whether the call was successful |
| `disconnection_reason` | string | `user_hangup`, `agent_hangup`, `call_transfer`, `unknown`; outbound calls also `busy`, `no_answer`, `dial_error` |
| `call_summary` | string | AI-generated call summary |
| `employee` | string | Assigned employee name |
| `note` | string | Manual note added by staff |
| `agent_id` | uuid | Agent that handled the call |
| `is_purged` | boolean | Whether sensitive data has been purged |
| `deleted_at` | datetime | Soft-delete timestamp (null if not deleted) |
| `created_at` | datetime | Call start timestamp |
| `updated_at` | datetime | Last update timestamp |

**Detail-only fields:**

| Field | Type | Description |
|-------|------|-------------|
| `transcript` | string | JSON string of conversation `[{"role":"agent","text":"..."},...]` |
| `audio_file_url` | string | Recording URL as stored by the voice agent. It is not re-signed by the API and may have expired — use the Dashboard for playback |
| `final_score` | number | AI-generated quality score (1-10) |

## Endpoints

### List calls

```
GET /v1/agents/{agentId}/calls
```

**Permission:** `calls:read` | **Pagination:** yes (ordered by `created_at` descending)

Soft-deleted calls are excluded by default. Calls that were permanently deleted in the Dashboard trash are never returned, not even with `include_deleted=true`.

**Filters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `done` | boolean | Filter by completion status |
| `from` | datetime | Calls created after this date (ISO 8601) |
| `to` | datetime | Calls created before this date (ISO 8601) |
| `phone_number` | string | Exact match on caller phone number |
| `search` | string | Search in customer name, phone number, or summary |
| `include_deleted` | boolean | Include soft-deleted calls (default: `false`) |

```bash
# Calls from the last 7 days that haven't been processed
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/calls?from=2026-03-15T00:00:00Z&done=false"
```

### Get call details

```
GET /v1/agents/{agentId}/calls/{callId}
```

**Permission:** `calls:read`

Returns full call details including transcript and audio URL.

**Note:** Purged calls (`is_purged: true`) have had their personal data removed by the retention process — expect `transcript`, `audio_file_url`, and `phone_number` to be null.

```json
{
  "id": "call-uuid-1",
  "call_id": "lk_call_abc123",
  "customer_name": "Hans Meier",
  "customer_request": "Frage zu Rechnung #1234",
  "phone_number": "+491701234567",
  "duration": 180,
  "done": true,
  "general_success": true,
  "final_score": 8,
  "disconnection_reason": "user_hangup",
  "call_summary": "Kunde fragte nach Rechnung #1234. Agent konnte Frage beantworten.",
  "transcript": "[{\"role\":\"agent\",\"text\":\"Guten Tag...\"},{\"role\":\"user\",\"text\":\"Hallo...\"}]",
  "audio_file_url": "https://storage.example.com/calls/abc123.wav",
  "employee": "Max Mustermann",
  "note": "Rueckruf vereinbart fuer Donnerstag",
  "is_purged": false,
  "deleted_at": null,
  "agent_id": "550e8400-e29b-41d4-a716-446655440000",
  "created_at": "2026-03-19T10:30:00Z",
  "updated_at": "2026-03-19T10:35:00Z"
}
```

## Transcript format

The `transcript` field is a JSON string containing an array of message objects:

```json
[
  { "role": "agent", "text": "Guten Tag, wie kann ich Ihnen helfen?" },
  { "role": "user", "text": "Ich habe eine Frage zu meiner Rechnung." },
  { "role": "agent", "text": "Natuerlich, um welche Rechnung geht es?" }
]
```

Parse it in your application:

```javascript
const messages = JSON.parse(call.transcript);
messages.forEach(m => console.log(`${m.role}: ${m.text}`));
```

## Common patterns

### Pull new calls since last sync

```javascript
const lastSync = "2026-03-20T00:00:00Z";
const { data: newCalls } = await talkpilot(
  `/agents/${agentId}/calls?from=${lastSync}&limit=100`
);
```

### Export all calls for a date range

```javascript
const allCalls = [];
let page = 1, totalPages = 1;

while (page <= totalPages) {
  const res = await talkpilot(
    `/agents/${agentId}/calls?from=2026-03-01T00:00:00Z&to=2026-03-31T23:59:59Z&page=${page}&limit=100`
  );
  allCalls.push(...res.data);
  totalPages = res.pagination.pages;
  page++;
}
```

## Related resources

- [Agents](/api/agents) — Parent resource
- [Webhooks](/guides/webhooks) — Get notified about calls instead of polling
- [Call Management](/product/call-management) — Dashboard UI guide
