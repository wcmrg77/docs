---
title: "Performance"
description: "Read call performance and QA data — latencies, token stats, redflags, and transcripts including tool calls"
---

Performance records are read-only QA data the system writes after an agent handles a call: pipeline latencies, token consumption, an automated quality assessment, and the full transcript including the agent's tool calls. This is the same data the Dashboard shows under [Performance Analysis](/product/performance-control).

These records are separate from [Calls](/api/calls). Calls are the customer-facing contact history (who called, what they wanted, the summary staff works with). Performance records are the technical and qualitative analysis of the same conversation — useful for monitoring agent quality, debugging slow responses, or feeding a QA dashboard.

## Access

**Permission:** `performance:read`

Issue an API key with this permission scoped to a single organization when a partner or internal tool needs read access to that organization's QA data — that is narrower than granting Dashboard access, which would expose far more.

With Dashboard JWT authentication the endpoints are additionally restricted to the `super_admin`, `dev_admin`, and `dev_employee` roles, mirroring the Dashboard's own guard on the Performance Analysis page. `client_admin` and `client_employee` receive `403 FORBIDDEN`.

## List vs detail view

| Field | List | Detail |
|-------|:----:|:------:|
| Identifiers, phone numbers, duration, status | yes | yes |
| Latency percentiles (end-to-end, STT, LLM, TTS) | yes | yes |
| Token stats | yes | yes |
| Pipeline configuration (providers, model, mode) | yes | yes |
| `final_result` and redflags | yes | yes |
| `pre_call_variables`, `extracted_variables` | yes | yes |
| `transcript` | — | yes |

The transcript is detail-only on purpose: it is by far the largest field, and a list request would multiply it by the page size. Internal debug logs are never returned by the API.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Performance record identifier — the `statisticId` path parameter |
| `call_id` | string | LiveKit call identifier; use it to correlate with the [Calls](/api/calls) resource |
| `caller_phone` | string | Caller's phone number |
| `agent_phone_number` | string | Number the caller dialled |
| `agent_id` | uuid | Agent that handled the call |
| `duration_seconds` | integer | Call duration in seconds |
| `disconnect_reason` | string | How the session ended |
| `session_status` | string | Final status of the agent session |
| `e2e_latency_p90` | number | End-to-end response latency, 90th percentile (seconds) |
| `e2e_latency_median` | number | End-to-end response latency, median |
| `e2e_latency_min` | number | End-to-end response latency, fastest turn |
| `transcription_latency_p90` | number | Speech-to-text latency, 90th percentile |
| `transcription_latency_median` | number | Speech-to-text latency, median |
| `transcription_latency_min` | number | Speech-to-text latency, minimum |
| `llm_latency_p90` | number | Language model latency, 90th percentile |
| `llm_latency_median` | number | Language model latency, median |
| `llm_latency_min` | number | Language model latency, minimum |
| `tts_latency_p90` | number | Text-to-speech latency, 90th percentile |
| `tts_latency_median` | number | Text-to-speech latency, median |
| `tts_latency_min` | number | Text-to-speech latency, minimum |
| `llm_total_tokens` | integer | Total tokens consumed by the call |
| `llm_prompt_tokens` | integer | Prompt tokens |
| `llm_completion_tokens` | integer | Completion tokens |
| `pre_call_variables` | object | Variables handed to the agent before the call started |
| `extracted_variables` | object | Variables the agent extracted during the call |
| `llm_provider` | string | Language model provider |
| `llm_model` | string | Language model used |
| `tts_provider` | string | Text-to-speech provider |
| `stt` | string | Speech-to-text provider |
| `pipeline_mode` | string | Voice pipeline mode |
| `created_at` | datetime | When the record was written (call start) |
| `final_result` | object | Overall evaluation, see below |
| `prompt_fingerprint` | string | SHA-256 of the prompt template — groups calls that ran on the same prompt version |

**Redflags.** Seven fields carry an automated assessment of a specific failure pattern: `assistant_misunderstood_multiple_times`, `repeated_same_sentence`, `abrupt_end_without_goodbye`, `connection_issue_repeated`, `assistant_policy_violation`, `technical_error`, `early_hangup_by_caller`.

Each is an object, not a boolean — check `detected`, do not test the field for truthiness:

| Field | Type | Description |
|-------|------|-------------|
| `detected` | boolean | Whether the pattern occurred |
| `reasoning` | string | Short explanation of the assessment |
| `score_impact` | number | How strongly this lowered the final score |

**`final_result`:**

| Field | Type | Description |
|-------|------|-------------|
| `outcome` | boolean | Whether the call achieved its goal |
| `outcome_label` | string | `success`, `failed`, or `neutral` |
| `final_score` | number | Overall quality score |
| `summary_reasoning` | string | Explanation of the score |

**Detail-only field:**

| Field | Type | Description |
|-------|------|-------------|
| `transcript` | array | Speech turns and tool calls, see [Transcript format](#transcript-format) |

## Endpoints

### List performance records

```
GET /v1/agents/{agentId}/performance
```

**Permission:** `performance:read` | **Pagination:** yes (ordered by `created_at` descending)

**Filters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `from` | datetime | Records created after this date (ISO 8601) |
| `to` | datetime | Records created before this date (ISO 8601) |

```bash
# Performance records from the last 7 days
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/performance?from=2026-03-15T00:00:00Z&limit=50"
```

### Get performance record details

```
GET /v1/agents/{agentId}/performance/{statisticId}
```

**Permission:** `performance:read`

Returns one record including the full transcript.

```bash
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/performance/9f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f"
```

```json
{
  "id": "9f1c2d3e-4a5b-6c7d-8e9f-0a1b2c3d4e5f",
  "call_id": "lk_call_abc123",
  "caller_phone": "+491701234567",
  "agent_id": "550e8400-e29b-41d4-a716-446655440000",
  "duration_seconds": 184,
  "session_status": "completed",
  "e2e_latency_p90": 1.42,
  "e2e_latency_median": 0.98,
  "llm_total_tokens": 4820,
  "llm_provider": "openai",
  "llm_model": "gpt-4o-mini",
  "created_at": "2026-03-19T10:30:00Z",
  "final_result": {
    "outcome": true,
    "outcome_label": "success",
    "final_score": 8,
    "summary_reasoning": "Termin erfolgreich vereinbart, keine Rueckfragen offen."
  },
  "technical_error": {
    "detected": false,
    "reasoning": "Keine technischen Fehler.",
    "score_impact": 0
  },
  "transcript": [
    { "role": "assistant", "text": "Guten Tag, Praxis Mustermann, was kann ich fuer Sie tun?", "time_sec": 2.1 },
    { "role": "user", "text": "Ich moechte einen Termin am zweiten April.", "time_sec": 9.4 }
  ]
}
```

## Transcript format

The `transcript` is an **array of objects** — not a JSON string, and not the simple `{role, text}` array used by [Calls](/api/calls). Do not reuse the parsing example from there.

All entries live in one list sorted chronologically by `time_sec` (seconds since call start). Three kinds of entry are distinguished by `role`:

**Speech turn** — `role` is `user` or `assistant`, the text is in `text`:

```json
{ "role": "assistant", "text": "Guten Tag, was kann ich fuer Sie tun?", "timestamp": "2026-03-19T10:30:02Z", "time_sec": 2.1 }
```

**Tool call** — the agent invoking a tool. `arguments` is a **JSON string**, not an object:

```json
{ "role": "tool_call_invocation", "tool_call_id": "call_7Hd2", "name": "check_availability", "arguments": "{\"date\":\"2026-04-02\"}", "time_sec": 10.8 }
```

**Tool result** — the tool's answer. The payload is in `content`, not in `text`:

```json
{ "role": "tool_call_result", "tool_call_id": "call_7Hd2", "successful": true, "content": "{\"slots\":[\"09:00\",\"14:30\"]}", "time_sec": 11.6 }
```

Match a result to its invocation via `tool_call_id`. Parse accordingly:

```javascript
for (const entry of record.transcript) {
  switch (entry.role) {
    case 'user':
    case 'assistant':
      console.log(`${entry.role}: ${entry.text}`);
      break;
    case 'tool_call_invocation':
      console.log(`-> ${entry.name}(${JSON.parse(entry.arguments)})`);
      break;
    case 'tool_call_result':
      console.log(`<- ${entry.successful ? 'ok' : 'failed'}: ${entry.content}`);
      break;
  }
}
```

`transcript` is `null` for calls recorded before transcript capture was enabled.

## Personal data

Transcripts are partially redacted before they are stored: phone numbers, e-mail addresses, IBANs, and digit runs of seven or more are removed. **Names and addresses remain in clear text.** Treat the data accordingly, and scope keys to the organization that actually owns the calls.

## Common patterns

### Watch response latency over time

```javascript
const res = await talkpilot(
  `/agents/${agentId}/performance?from=2026-03-01T00:00:00Z&limit=100`
);
const slow = res.data.filter(r => r.e2e_latency_p90 > 2);
console.log(`${slow.length} of ${res.data.length} calls above 2s p90`);
```

### Find calls with a detected failure pattern

```javascript
const REDFLAGS = [
  'assistant_misunderstood_multiple_times', 'repeated_same_sentence',
  'abrupt_end_without_goodbye', 'connection_issue_repeated',
  'assistant_policy_violation', 'technical_error', 'early_hangup_by_caller',
];

const flagged = res.data.filter(r => REDFLAGS.some(f => r[f]?.detected));
```

### Export a date range with transcripts

The list view omits transcripts, so fetch the detail per record:

```javascript
const list = await talkpilot(
  `/agents/${agentId}/performance?from=2026-03-01T00:00:00Z&to=2026-03-31T23:59:59Z&limit=100`
);

for (const record of list.data) {
  const full = await talkpilot(`/agents/${agentId}/performance/${record.id}`);
  // full.transcript is now available
}
```

## Related resources

- [Agents](/api/agents) — Parent resource
- [Calls](/api/calls) — Customer-facing call history and summaries
- [Performance Analysis](/product/performance-control) — Dashboard UI guide
