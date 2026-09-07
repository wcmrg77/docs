---
title: "Workflow"
description: "Read and replace the multi-state conversation flow of an agent"
---

Multi-state agents keep their conversation logic in a **workflow** — a set of states the
agent moves through during a call. The agent's `prompt` only holds the global part
(identity, tone, general rules); the actual call flow lives here.

Single-prompt agents have no workflow. For them this endpoint returns `null`.

<Note>
  The workflow is **not** part of `PATCH /v1/agents/{agent_id}`. It is a large object and
  is validated before it is stored, so it has its own endpoint.
</Note>

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Workflow name (1–100 chars) |
| `description` | string | Optional summary (max 500 chars) |
| `steps` | array | 1–20 states, see below |

### Step

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique within the workflow (1–50 chars) |
| `name` | string | yes | Display name (1–100 chars) |
| `description` | string | yes | The instructions for this state (1–20000 chars) |
| `completion` | string | yes | When this state is considered done (1–500 chars) |
| `step_type` | string | no | `prompt` (default) or `action` |
| `tools` | array | no | Tool names available in this state (max 20) |
| `next` | array | no | Allowed follow-up states, `[{ "id": "..." }]` (max 10) |

Every `next[].id` must reference an existing step `id`.

## Endpoints

### Get workflow

```
GET /v1/agents/{agent_id}/workflow
```

**Permission:** `agents:read`

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/$AGENT_ID/workflow"
```

Response:

```json
{
  "agent_id": "agent-uuid",
  "workflow": {
    "name": "Inbound Call Flow",
    "description": "intent_analysis routes, callback_info is terminal.",
    "steps": [
      {
        "id": "intent_analysis",
        "name": "Anruf-Analyse",
        "description": "Erkenne das Anliegen und route weiter.",
        "completion": "Anliegen erkannt und weitergeleitet.",
        "step_type": "prompt",
        "next": [{ "id": "callback_info" }]
      },
      {
        "id": "callback_info",
        "name": "Rückruf-Daten aufnehmen",
        "description": "Nimm die fehlenden Pflichtangaben auf.",
        "completion": "Daten erfasst, Anruf beendet.",
        "step_type": "prompt",
        "next": []
      }
    ]
  },
  "updated_at": "2025-07-26T14:41:23Z"
}
```

### Replace workflow

Replaces the workflow completely. There is no partial update — send the full object.

```
PUT /v1/agents/{agent_id}/workflow
```

**Permission:** `agents:write`

```bash
curl -X PUT -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d @workflow.json \
  "$TP_BASE/agents/$AGENT_ID/workflow"
```

The body may be the workflow object itself or wrapped as `{"workflow": { ... }}`.

To remove the workflow and turn the agent back into a single-prompt agent, send `null`:

```bash
curl -X PUT -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"workflow": null}' \
  "$TP_BASE/agents/$AGENT_ID/workflow"
```

### Validation

The workflow is validated against the same rules the voice runtime enforces. If it is
invalid, nothing is written and the response is `400 VALIDATION_ERROR` listing every
problem found:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid workflow definition",
    "details": [
      { "field": "steps[0].completion", "message": "Required, must be a non-empty string" },
      { "field": "steps[1].next[0].id", "message": "References unknown step 'callbak_info'" }
    ]
  }
}
```

<Warning>
  This validation matters: the runtime silently discards a workflow it cannot parse and
  the agent then runs **without** its states. Rejecting the write is the safer failure.
</Warning>

## Related resources

- [Agents](/api/agents) — agent configuration and prompt
- [Tools](/api/tools) — tools that steps can reference
