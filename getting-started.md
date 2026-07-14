---
title: "Getting Started"
description: "Create your first API key and make your first request"
---

This guide walks you through creating an API key, making your first request, and understanding the response format.

## Step 1: Create an API Key

1. Log in to the [TalkPilot Dashboard](https://app.talkpilot.io)
2. Navigate to **Settings > API**
3. Click **Create API Key**
4. Choose a name, set permissions, and optionally restrict to specific agents
5. Copy the key — it's only shown once

Your key looks like this:

```
tp_live_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
```

## Step 2: Make your first request

Test your key with the health endpoint (no authentication required):

```bash
curl https://api.talkpilot.io/v1/health
```

Response:

```json
{
  "status": "ok",
  "version": "1.0.0",
  "timestamp": "2026-03-22T10:00:00Z"
}
```

Now list your agents using your API key:

```bash
curl -H "X-API-Key: tp_live_YOUR_KEY_HERE" \
  https://api.talkpilot.io/v1/agents
```

Response:

```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Rezeption Bot",
      "phone_number": "+4930123456",
      "is_active": true,
      "language": "de",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "voice_id": "sarah",
      "created_at": "2025-06-01T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1,
    "pages": 1
  }
}
```

## Step 3: Understand the response format

### Success responses

All list endpoints return a `data` array and a `pagination` object:

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 42,
    "pages": 3
  }
}
```

Single-resource endpoints return the object directly:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Rezeption Bot",
  ...
}
```

### Error responses

All errors follow a consistent structure:

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Agent not found",
    "request_id": "req_abc123"
  }
}
```

Validation errors include field-level details:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      { "field": "name", "message": "Required" },
      { "field": "phone_number", "message": "Must be a valid E.164 phone number" }
    ],
    "request_id": "req_def456"
  }
}
```

## Step 4: Common operations

### Update an employee's status

```bash
curl -X PATCH \
  -H "X-API-Key: tp_live_YOUR_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{"status": "urlaub"}' \
  https://api.talkpilot.io/v1/agents/{agentId}/employees/{employeeId}
```

### Get recent calls

```bash
curl -H "X-API-Key: tp_live_YOUR_KEY_HERE" \
  "https://api.talkpilot.io/v1/agents/{agentId}/calls?limit=10&from=2026-03-01T00:00:00Z"
```

### Enable vacation mode for an agent

```bash
curl -X PATCH \
  -H "X-API-Key: tp_live_YOUR_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{"vacation_mode": true, "vacation_end": "2026-04-01T00:00:00Z", "vacation_notdienst": true}' \
  https://api.talkpilot.io/v1/agents/{agentId}
```

## Next steps

- [Authentication](/authentication) — Permissions, scoping, and key management
- [Pagination](/guides/pagination) — Navigate large result sets
- [Error Handling](/guides/error-handling) — Handle errors gracefully
- [Code Examples](/examples/javascript) — Full working examples in JavaScript, Python, and cURL
