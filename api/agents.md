---
title: "Agents"
description: "List, read, and update AI agent configurations"
---

Agents are the central entity in TalkPilot — AI voice bots that handle phone calls. Every other resource (tools, employees, forwarding slots, knowledge base, calls) is scoped to an agent.

## Key concepts

- Each agent has a **phone number** and belongs to one **organization**
- Agents are configured with an **LLM** (language model), **voice**, and **system prompt**
- Agents can have a **schedule** (business hours) and **vacation mode**
- A **backup agent** handles calls when the primary agent is unavailable

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Unique agent identifier |
| `name` | string | Agent display name |
| `phone_number` | string | Assigned phone number (read-only) |
| `is_active` | boolean | Whether the agent accepts calls |
| `language` | string | Language code: `de`, `en`, `fr`, `es` |
| `formality` | string | `formal` (Sie) or `informal` (Du) |
| `llm_provider` | string | `azure`, `openai`, `gemini`, `gemini_live`, `gemini_live_cascade`, `openai_realtime`, `openai_realtime_cascade` |
| `llm_model` | string | Model identifier (e.g., `gpt-4o`, `gemini-2.5-flash`) |
| `llm_temperature` | number | Creativity (0 = deterministic, 2 = creative) |
| `tts_provider` | string | `cartesia`, `elevenlabs`, `gemini_live`, `openai_realtime` |
| `voice_id` | string | Voice identifier for TTS |
| `realtime_voice` | string | Voice for native realtime providers |
| `speaking_rate` | number | Speech speed (0.5 - 2.0) |
| `stt_provider` | string | `deepgram`, `elevenlabs`, `mistral` (null for realtime) |
| `prompt` | string | System prompt (max 50,000 chars) |
| `greeting` | string | Initial greeting message (max 2,000 chars) |
| `post_call_webhook_url` | string | Webhook URL called after each call |
| `schedule` | object | Business hours configuration |
| `backup_agent_id` | uuid | Fallback agent when unavailable |
| `vacation_mode` | boolean | Whether vacation mode is active |
| `vacation_end` | datetime | When vacation mode auto-disables |
| `vacation_notdienst` | boolean | Emergency service during vacation |
| `allow_greeting_interruption` | boolean | Allow callers to interrupt greeting |
| `short_calls_auto_edit` | boolean | Auto-mark very short calls as done |
| `track_caller_name` | boolean | Ask for and track caller's name |
| `transfer_timeout_seconds` | number | Transfer timeout (10-120 seconds) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

### Schedule object

```json
{
  "timezone": "Europe/Berlin",
  "rules": [
    {
      "days": [1, 2, 3, 4, 5],
      "start_time": "08:00",
      "end_time": "18:00"
    },
    {
      "days": [6],
      "start_time": "09:00",
      "end_time": "14:00"
    }
  ]
}
```

Days: `0` = Sunday, `1` = Monday, ..., `6` = Saturday.

## Endpoints

### List agents

```
GET /v1/agents
```

**Permission:** `agents:read` | **Pagination:** yes

Returns all agents accessible to the API key. If the key has `allowed_agent_ids` set, only those agents are returned.

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents"
```

### Get agent details

```
GET /v1/agents/{agentId}
```

**Permission:** `agents:read`

Returns the full configuration of a single agent.

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/550e8400-e29b-41d4-a716-446655440000"
```

### Get agent status (lightweight)

```
GET /v1/agents/{agentId}/status
```

**Permission:** `agents:read`

Returns only the agent's active status, schedule, and vacation state. Includes a computed `is_within_schedule` field. Use this for monitoring dashboards where you don't need the full config.

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Rezeption Bot",
  "is_active": true,
  "vacation_mode": false,
  "vacation_end": null,
  "vacation_notdienst": false,
  "schedule": {
    "timezone": "Europe/Berlin",
    "rules": [{ "days": [1,2,3,4,5], "start_time": "08:00", "end_time": "18:00" }]
  },
  "is_within_schedule": true
}
```

### Update agent

```
PATCH /v1/agents/{agentId}
```

**Permission:** `agents:write`

Partial update — only include fields you want to change. Omitted fields remain unchanged.

**Read-only fields** (cannot be changed via API): `phone_number`, `organization_id`, `created_by`

```bash
# Update prompt and greeting
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"prompt": "Du bist ein freundlicher Kundenservice-Agent.", "greeting": "Hallo, wie kann ich helfen?"}' \
  "$TP_BASE/agents/550e8400-e29b-41d4-a716-446655440000"
```

## Common patterns

### Vacation mode workflow

```bash
# Enable vacation with end date and emergency service
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"vacation_mode": true, "vacation_end": "2026-04-01T00:00:00Z", "vacation_notdienst": true}' \
  "$TP_BASE/agents/{agentId}"

# Disable vacation
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"vacation_mode": false}' \
  "$TP_BASE/agents/{agentId}"
```

### Set business hours with backup

```bash
# Set Mon-Fri 9-17, Sat 9-14, with backup agent
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "schedule": {
      "timezone": "Europe/Berlin",
      "rules": [
        {"days": [1,2,3,4,5], "start_time": "09:00", "end_time": "17:00"},
        {"days": [6], "start_time": "09:00", "end_time": "14:00"}
      ]
    },
    "backup_agent_id": "backup-agent-uuid"
  }' \
  "$TP_BASE/agents/{agentId}"
```

## Related resources

- [Tools](/tools) — Functions the agent can invoke during calls
- [Employees](/employees) — Human staff linked to the agent
- [Forwarding Slots](/forwarding-slots) — Call routing rules
- [Knowledge Base](/knowledge-base) — Documents the agent can reference
- [Calls](/calls) — Call records for this agent
