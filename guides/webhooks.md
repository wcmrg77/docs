---
title: "Webhooks"
description: "Pre-call lookups and real-time call events (call_started, call_ended)"
---

TalkPilot talks to your systems at three points of a call. The payloads on this page are
**generated from the code that sends them** (`talkpilot-shared/docs/webhook-samples/`, kept in
sync by a test), so what you see here is what arrives.

| Event | When | Direction | Configured where | Retries |
|-------|------|-----------|------------------|---------|
| **Pre-call request** | Before the agent answers | TalkPilot → you, **you answer with variables** | Per phone number (set by TalkPilot support) | 3 attempts, 3 s timeout each |
| **`call_started`** | Right after the call is set up | TalkPilot → you | Agent `post_call_webhook_url` | none (fire-and-forget) |
| **`call_ended`** | After the call ends | TalkPilot → you | Agent `post_call_webhook_url` | 3 attempts, 10 s timeout each |

## Configuration

**Post-call URL** — set `post_call_webhook_url` on the agent; both `call_started` and `call_ended`
are delivered there:

```bash
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"post_call_webhook_url": "https://your-app.com/webhooks/talkpilot"}' \
  "$TP_BASE/agents/{agentId}"
```

Or configure it in the Dashboard under **Agenten > [Agent] > Webhook**.

**Pre-call URL** — bound to the phone number, not the agent. Contact
[support@talkpilot.io](mailto:support@talkpilot.io) to set it.

## Pre-call request

Sent before the agent greets the caller. Use it to look the caller up in your CRM and hand the
agent variables for its prompt and greeting (`{{var.name}}`), or to route the call to another agent.

Request body:

```json
{
  "caller_phone": "+491701234567",
  "called_phone": "+4930123456",
  "agent_id": "5c1f0f9e-2b7a-4d3e-9c1a-0f6b2d7e8a11",
  "agent_name": "Rezeption Musterfirma",
  "timestamp": "2026-09-07T10:15:42+00:00"
}
```

Respond with `200` and JSON within 3 seconds. Two shapes are accepted:

```json
{ "variables": { "kundenname": "Erika Musterfrau", "kundennummer": "K-10442" }, "agent_id": null }
```

```json
{ "call_inbound": { "dynamic_variables": { "kundenname": "Erika Musterfrau" }, "override_agent_id": null } }
```

- `variables` / `dynamic_variables` — become `{{var.<name>}}` in prompt and greeting and are echoed back as `pre_call_variables` in both events below.
- `agent_id` / `override_agent_id` — optional: the UUID of another agent of your organization that should take the call instead.

An empty or non-JSON answer counts as a failure and is retried; after the last attempt the call proceeds without variables.

## `call_started`

Sent once the call is set up, before the first sentence. No retries — treat it as a hint, not as a
guarantee.

```json
{
  "event": "call_started",
  "call_id": "call_3f9a1c7e2b8d4e6f",
  "caller_phone": "+491701234567",
  "called_phone": "+4930123456",
  "agent_id": "5c1f0f9e-2b7a-4d3e-9c1a-0f6b2d7e8a11",
  "agent_name": "Rezeption Musterfirma",
  "pre_call_variables": {
    "kundenname": "Erika Musterfrau",
    "kundennummer": "K-10442"
  },
  "timestamp": "2026-09-07T10:15:42+00:00"
}
```

## `call_ended`

Sent after the call ends, with the full transcript.

```json
{
  "event": "call_ended",
  "call_id": "call_3f9a1c7e2b8d4e6f",
  "caller_phone": "+491701234567",
  "called_phone": "+4930123456",
  "agent_id": "5c1f0f9e-2b7a-4d3e-9c1a-0f6b2d7e8a11",
  "agent_name": "Rezeption Musterfirma",
  "duration_seconds": 63,
  "transcript": [
    {
      "role": "assistant",
      "text": "Guten Tag, Musterfirma, was kann ich für Sie tun?",
      "timestamp": "2026-09-07T10:14:40.200000+00:00",
      "time_sec": 1.2
    },
    {
      "role": "user",
      "text": "Ich habe eine Frage zu meiner Rechnung.",
      "timestamp": "2026-09-07T10:14:43.800000+00:00",
      "time_sec": 4.8
    },
    {
      "role": "assistant",
      "text": "Gern, ich verbinde Sie mit der Buchhaltung.",
      "timestamp": "2026-09-07T10:14:46.500000+00:00",
      "time_sec": 7.5
    },
    {
      "role": "tool_call_invocation",
      "tool_call_id": "call_tc_01",
      "name": "transfer_buchhaltung",
      "arguments": "{\"phone_number\": \"+4930123457\"}",
      "time_sec": 9.1,
      "type": "transfer_call"
    },
    {
      "role": "tool_call_result",
      "tool_call_id": "call_tc_01",
      "successful": true,
      "content": "{\"success\": true, \"status\": \"transferred\"}",
      "time_sec": 14.6
    }
  ],
  "total_turns": 5,
  "pre_call_variables": {
    "kundenname": "Erika Musterfrau",
    "kundennummer": "K-10442"
  },
  "extracted_variables": {
    "anliegen": "Rechnungsfrage"
  },
  "recording_url": "https://storage.example/call_recordings/2026/09/call_3f9a1c7e2b8d4e6f.ogg",
  "recording_multi_channel_url": null,
  "public_log_url": "https://talk.talkpilot.io/rooms/call-3f9a1c7e",
  "disconnection_reason": "user_hangup",
  "latency": null,
  "max_response_gap_ms": 1450,
  "timestamp": "2026-09-07T10:15:42+00:00"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `call_id` | string | Stable call identifier — same value in `call_started`, the Calls API (`call_id`) and the Dashboard |
| `caller_phone` / `called_phone` | string \| null | E.164. For outbound calls `caller_phone` is your own number and `called_phone` the dialled target |
| `duration_seconds` | integer | Call duration |
| `transcript` | array | Chronological entries in three shapes, see below |
| `total_turns` | integer | Number of transcript entries |
| `pre_call_variables` | object | What your pre-call webhook (or an outbound API call) supplied |
| `extracted_variables` | object | Values captured by `extract_variable` tools; outbound agents add `_call_result` |
| `recording_url` | string \| null | Recording, if recording is enabled for the agent |
| `recording_multi_channel_url` | null | Reserved |
| `public_log_url` | string \| null | Link to the call's room log |
| `disconnection_reason` | string | `user_hangup`, `agent_hangup`, `call_transfer`, `unknown`; outbound: `busy`, `no_answer`, `dial_error` |
| `latency` | null | Reserved — latency metrics live in the Dashboard's performance view |
| `max_response_gap_ms` | integer \| null | Longest silence between the caller finishing and the agent answering (dead air) |
| `timestamp` | string | ISO 8601, time of sending |

### Transcript entries

| `role` | Fields |
|--------|--------|
| `user`, `assistant` | `text`, `timestamp` (ISO 8601), `time_sec` (seconds since call start) |
| `tool_call_invocation` | `tool_call_id`, `name` (tool name), `arguments` (JSON **string**), `time_sec`, optional `type` (tool type, e.g. `transfer_call`) |
| `tool_call_result` | `tool_call_id`, `successful` (boolean), `content` (JSON **string**), `time_sec` |

`arguments` and `content` are JSON encoded as strings — parse them a second time if you need the fields.

<Note>
The webhook carries the raw call. The customer name, request and summary you see in the Dashboard
are produced afterwards by post-processing; fetch them with the [Calls API](/api/calls) if you need them.
</Note>

## Verifying the signature

Every delivery is signed when TalkPilot has a webhook secret configured for your account
(ask support for it). Headers:

| Header | Value |
|--------|-------|
| `X-TalkPilot-Timestamp` | Unix timestamp (seconds) of the request |
| `X-TalkPilot-Signature` | `sha256=<hex>` — HMAC-SHA256 with the secret over `"<timestamp>.<raw body>"` |
| `X-TalkPilot-Secret` | The raw secret (legacy — prefer the signature) |

Verify over the **raw request body** exactly as received. The body is serialized deterministically
(keys sorted, no whitespace, UTF-8 not escaped), so re-serializing a parsed payload the same way
also works, but the raw body is the safe choice.

```javascript
const crypto = require("crypto");

function verify(rawBody, headers, secret) {
  const timestamp = headers["x-talkpilot-timestamp"];
  const expected = "sha256=" + crypto
    .createHmac("sha256", secret)
    .update(`${timestamp}.${rawBody}`)
    .digest("hex");
  return crypto.timingSafeEqual(
    Buffer.from(expected),
    Buffer.from(headers["x-talkpilot-signature"] || ""),
  );
}
```

In n8n, enable **Raw Body** on the Webhook node — a body that n8n has already parsed and
re-serialized produces a different signature.

**Test vector** (`signature_vector.json` in the samples): with secret `beispiel-webhook-secret`,
timestamp `1788776142` and the `call_ended` payload above serialized canonically, the signature is
`sha256=8470225fe8cb45c069f89a9eea963b3702764a670e542b314094ef57167d3525`.

## Best practices

1. **Respond with 200 quickly** — process asynchronously; `call_ended` waits at most 10 seconds per attempt.
2. **Deduplicate on `call_id`** — `call_ended` is retried up to three times, so a delivery can arrive twice.
3. **Verify the signature** instead of relying on a secret in the URL.
4. **Catch up via the API** — if your endpoint was down for all attempts, the delivery is lost; use the [Calls API](/api/calls) to backfill.
5. **Don't rely on `call_started`** for anything critical — it is not retried.

## Related resources

- [Agents API](/api/agents) — Configure `post_call_webhook_url`
- [CRM Integration](/use-cases/crm-integration) — End-to-end integration guide
