---
title: "Tools"
description: "Create, update, and delete agent tools"
---

Tools are functions that an AI agent can invoke during a call. They extend the agent's capabilities beyond conversation — making API calls, transferring calls, extracting data, and more.

## Tool types

TalkPilot supports 7 tool types:

| Type | Description |
|------|-------------|
| `http_request` | Make HTTP requests to external APIs (CRM lookup, ticket creation) |
| `transfer_call` | Cold or warm call transfer to a phone number |
| `monitored_transfer` | SIP INVITE transfer with call monitoring |
| `end_call` | End the call with an optional goodbye message |
| `extract_variable` | Extract structured data from the conversation (email, phone, etc.) |
| `play_tone` | Play DTMF tones (for IVR navigation) |
| `knowledge_base` | Search the agent's knowledge base documents |

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Tool identifier |
| `agent_id` | uuid | Parent agent |
| `name` | string | Internal name for LLM function calling (max 100 chars) |
| `display_name` | string | Human-readable name (max 255 chars) |
| `description` | string | Description shown to the LLM (max 2,000 chars) |
| `tool_type` | string | One of the 7 types above |
| `config` | object | Tool-specific configuration (see below) |
| `is_enabled` | boolean | Whether the tool is active |
| `priority` | integer | Execution priority (lower = higher priority) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

## Tool configurations

### http_request

Make HTTP requests to external APIs during a call.

```json
{
  "url": "https://crm.example.com/api/customers",
  "method": "GET",
  "headers": [
    { "key": "Authorization", "value": "Bearer {{API_KEY}}" }
  ],
  "query_params": [
    { "key": "phone", "value": "{{caller_phone}}" }
  ],
  "timeout": 30,
  "speak_during_execution": true,
  "speak_after_execution": true
}
```

| Config field | Type | Description |
|-------------|------|-------------|
| `url` | string | Target URL (SSRF-protected — no internal IPs) |
| `method` | string | `GET`, `POST`, `PUT`, `DELETE` |
| `headers` | array | Key-value header pairs |
| `query_params` | array | Key-value query parameters |
| `timeout` | number | Request timeout in seconds |
| `speak_during_execution` | boolean | Agent speaks while waiting for response |
| `speak_after_execution` | boolean | Agent speaks the result to the caller |

### transfer_call

Transfer the call to a human agent or external number.

```json
{
  "phone_number": "+4930999888",
  "transfer_mode": "cold",
  "play_hold_music": false
}
```

| Config field | Type | Description |
|-------------|------|-------------|
| `phone_number` | string | Target phone number |
| `transfer_mode` | string | `cold` (immediate) or `warm` (agent stays on line) |
| `play_hold_music` | boolean | Play music while transferring |

### monitored_transfer

SIP INVITE transfer with monitoring — the AI stays on the line to monitor the transfer.

```json
{
  "phone_number": "+4930999888",
  "timeout_seconds": 20
}
```

### end_call

Terminate the call with an optional goodbye message.

```json
{
  "goodbye_message": "Vielen Dank fuer Ihren Anruf. Auf Wiederhoeren!"
}
```

### extract_variable

Extract structured data from the conversation.

```json
{
  "variable_name": "email",
  "variable_type": "email",
  "prompt_text": "Bitte nennen Sie mir Ihre E-Mail-Adresse.",
  "example_format": "name@example.com",
  "confirmation_enabled": true
}
```

| Config field | Type | Description |
|-------------|------|-------------|
| `variable_name` | string | Name of the extracted variable |
| `variable_type` | string | `email`, `phone`, `name`, `number`, `date`, `custom` |
| `prompt_text` | string | What the agent asks to extract the data |
| `example_format` | string | Example shown to the LLM |
| `confirmation_enabled` | boolean | Ask the caller to confirm the extracted value |

### play_tone

Play DTMF tones for IVR navigation.

```json
{
  "tone_type": "dtmf",
  "dtmf_digit": "1",
  "duration": 500,
  "volume": 0.8
}
```

### knowledge_base

Search the agent's knowledge base using RAG (Retrieval-Augmented Generation).

```json
{
  "top_k": 5,
  "similarity_threshold": 0.7,
  "bridging_sentence": "Einen Moment, ich schaue kurz nach..."
}
```

| Config field | Type | Description |
|-------------|------|-------------|
| `top_k` | number | Number of document chunks to retrieve |
| `similarity_threshold` | number | Minimum similarity score (0-1) |
| `bridging_sentence` | string | What the agent says while searching |

## Endpoints

### List tools

```
GET /v1/agents/{agentId}/tools
```

**Permission:** `tools:read` | **Pagination:** no (returns all tools)

### Get tool

```
GET /v1/agents/{agentId}/tools/{toolId}
```

**Permission:** `tools:read`

### Create tool

```
POST /v1/agents/{agentId}/tools
```

**Permission:** `tools:write`

**Required fields:** `name`, `description`, `tool_type`, `config`

Returns `409 Conflict` if a tool with the same `name` already exists for the agent.

```bash
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "name": "lookup_customer",
    "display_name": "Kunden-Lookup",
    "description": "Looks up customer info by phone number",
    "tool_type": "http_request",
    "config": {
      "url": "https://crm.example.com/api/customers",
      "method": "GET",
      "headers": [{"key": "Authorization", "value": "Bearer YOUR_KEY"}],
      "timeout": 30,
      "speak_during_execution": true,
      "speak_after_execution": true
    },
    "priority": 1,
    "is_enabled": true
  }' \
  "$TP_BASE/agents/{agentId}/tools"
```

### Update tool

```
PATCH /v1/agents/{agentId}/tools/{toolId}
```

**Permission:** `tools:write`

### Delete tool

```
DELETE /v1/agents/{agentId}/tools/{toolId}
```

**Permission:** `tools:write` | Returns `204 No Content`

## Related resources

- [Agents](/agents) — Parent resource
- [Knowledge Base](/knowledge-base) — Documents used by the `knowledge_base` tool
- [Tools Configuration](/product/tools-configuration) — Dashboard UI guide
