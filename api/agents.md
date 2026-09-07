---
title: "Agents"
description: "List, read, and update AI agent configurations"
---

Agents are the central entity in TalkPilot — AI voice bots that handle phone calls. Every other resource (tools, employees, forwarding slots, knowledge base, calls) is scoped to an agent.

## Key concepts

- Each agent has a **phone number** and belongs to one **organization**
- Agents are configured with an **LLM** (language model), **voice**, and **system prompt**
- Agents can have a **schedule** (business hours)
- A **backup agent** handles calls when the primary agent is unavailable

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Unique agent identifier |
| `organization_id` | uuid | Owning organization (read-only) |
| `name` | string | Agent display name |
| `phone_number` | string | Assigned phone number (read-only) |
| `is_active` | boolean | Whether the agent accepts calls |
| `language` | string | Language code: `de`, `en`, `fr`, `es` |
| `formality` | string | `formal` (Sie) or `informal` (Du) |
| `llm_provider` | string | `azure`, `openai`, `gemini`, `gemini_live`, `gemini_live_cascade`, `openai_realtime`, `openai_realtime_cascade`, `groq` |
| `llm_model` | string | Model identifier (e.g., `gpt-4.1`, `gemini-2.5-flash`) |
| `llm_temperature` | number | Creativity (0 = deterministic, 2 = creative) |
| `tts_provider` | string | `cartesia`, `elevenlabs`, `gemini_live`, `openai_realtime` |
| `voice_id` | string | Voice identifier for TTS |
| `realtime_voice` | string | Voice for native realtime providers |
| `speaking_rate` | number | Speech speed (0.5 - 2.0) |
| `stt_provider` | string | `deepgram`, `elevenlabs`, `mistral`, `groq` (null for realtime) |
| `stt_keywords` | string[] | Words the speech recognizer should prefer (names, products), max 100 |
| `prompt` | string | System prompt (max 50,000 chars) |
| `greeting` | string | Initial greeting message (max 2,000 chars) |
| `greeting_variants` | array | Personalised greetings by contact tag, time window or repeat calls — see below |
| `post_call_webhook_url` | string | Webhook URL called after each call |
| `schedule` | object | Business hours configuration |
| `backup_agent_id` | uuid | Fallback agent when outside the schedule |
| `allow_greeting_interruption` | boolean | Allow callers to interrupt greeting |
| `short_calls_auto_edit` | boolean | Auto-mark very short calls as done |
| `track_caller_name` | boolean | Ask for and track caller's name |
| `transfer_timeout_seconds` | integer | Transfer timeout (5-45 seconds) |
| `bundesland` | string | German state for public holidays, never null (`DE` = default): `DE` (nationwide only), `BW`, `BY`, `BE`, `BB`, `HB`, `HH`, `HE`, `MV`, `NI`, `NW`, `NRW`, `RP`, `SL`, `SN`, `ST`, `SH`, `TH` |
| `background_audio_enabled` | boolean | Ambient background audio during calls |
| `background_audio_type` | string | `office`, `cafe`, `home_office` |
| `background_audio_volume` | number | 0–1 |
| `thinking_sound_enabled` | boolean | Subtle sound while the agent waits for the LLM |
| `thinking_sound_volume` | number | 0–1 |
| `aic_enhancement_enabled` | boolean | On-device voice-focus enhancement before speech recognition (never null, use `false`) |
| `retention_days` | integer | GDPR retention of call data in days, null = default 90 (read-only) |
| `metadata` | object | Provider tuning such as Flux/EOU settings (read-only) |
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

### Greeting variants

```json
[
  { "tag": "stammkunde", "greeting": "Guten Tag {{anrede}} {{nachname}}, schön, dass Sie anrufen." },
  { "from": "18:00", "to": "07:00", "greeting": "Guten Abend, Sie erreichen den Nachtdienst." },
  { "min_calls_today": 2, "greeting": "Schön, dass Sie noch einmal anrufen." }
]
```

The first variant whose conditions all match wins; otherwise the plain `greeting` is spoken. Each variant needs `greeting` and at least one condition:

| Condition | Meaning |
|-----------|---------|
| `tag` | The caller's contact (customer database) carries this tag, case-insensitive |
| `from` + `to` | Time window, `HH:MM` Europe/Berlin, start inclusive / end exclusive; `from` > `to` wraps past midnight. Both required |
| `min_calls_today` | Caller has already called at least this many times today (integer ≥ 1) |

Placeholders `{{vorname}}`, `{{nachname}}`, `{{anrede}}` come from the contact; pre-call webhook variables work too. A variant is only spoken when every placeholder has a value. Unknown keys are rejected with `400`.

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

Returns only the agent's active status and schedule. Includes a computed `is_within_schedule` field. Use this for monitoring dashboards where you don't need the full config.

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Rezeption Bot",
  "is_active": true,
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

**Read-only fields** (cannot be changed via API): `phone_number`, `organization_id`, `created_by`, `retention_days`, `metadata`

**Unknown fields are rejected.** Sending any field that is not in the data model above (or a read-only one) returns `400 VALIDATION_ERROR` with one `details` entry per field — this endpoint is strict, unlike the other `PATCH` endpoints, which ignore unknown fields.

**Not part of this endpoint:** `workflow`. Multi-state agents keep their conversation flow
in a separate, validated resource — see [Workflow](/api/workflow). Sending `workflow` here
returns `400 VALIDATION_ERROR`.

```bash
# Update prompt and greeting
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"prompt": "Du bist ein freundlicher Kundenservice-Agent.", "greeting": "Hallo, wie kann ich helfen?"}' \
  "$TP_BASE/agents/550e8400-e29b-41d4-a716-446655440000"
```

## Common patterns

### Pause an agent temporarily

```bash
# Stop accepting calls (the number keeps ringing on the backup agent, if one is set)
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"is_active": false}' \
  "$TP_BASE/agents/{agentId}"

# Resume
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"is_active": true}' \
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

- [Workflow](/api/workflow) — Multi-state conversation flow
- [Tools](/api/tools) — Functions the agent can invoke during calls
- [Employees](/api/employees) — Human staff linked to the agent
- [Forwarding Slots](/api/forwarding-slots) — Call routing rules
- [Knowledge Base](/api/knowledge-base) — Documents the agent can reference
- [Calls](/api/calls) — Call records for this agent
