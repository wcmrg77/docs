---
title: "cURL Examples"
description: "Ready-to-use cURL examples for every endpoint"
---

All examples use environment variables for the base URL and API key. Set them once:

```bash
export TP_BASE="https://api.talkpilot.io/v1"
export TP_KEY="tp_live_YOUR_KEY_HERE"
```

## Health check

```bash
curl "$TP_BASE/health"
```

## Agents

### List all agents

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents"
```

### Get agent details

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}"
```

### Get agent status (lightweight)

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}/status"
```

### Update agent prompt

```bash
curl -X PATCH \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Du bist ein professioneller Kundenservice-Agent fuer Firma X.",
    "greeting": "Willkommen bei Firma X, wie kann ich Ihnen helfen?"
  }' \
  "$TP_BASE/agents/{agentId}"
```

### Enable vacation mode

```bash
curl -X PATCH \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vacation_mode": true,
    "vacation_end": "2026-04-01T00:00:00Z",
    "vacation_notdienst": true
  }' \
  "$TP_BASE/agents/{agentId}"
```

### Set business hours

```bash
curl -X PATCH \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schedule": {
      "timezone": "Europe/Berlin",
      "rules": [
        { "days": [1, 2, 3, 4, 5], "start_time": "08:00", "end_time": "18:00" },
        { "days": [6], "start_time": "09:00", "end_time": "14:00" }
      ]
    }
  }' \
  "$TP_BASE/agents/{agentId}"
```

## Employees

### List employees

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}/employees"
```

### Filter by status

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}/employees?status=anwesend&active=true"
```

### Create an employee

```bash
curl -X POST \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Max Mustermann",
    "phone_number": "+491701234567",
    "email": "max@example.com",
    "status": "anwesend",
    "active": true,
    "get_mail": true
  }' \
  "$TP_BASE/agents/{agentId}/employees"
```

### Update employee status

```bash
curl -X PATCH \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{"status": "urlaub"}' \
  "$TP_BASE/agents/{agentId}/employees/{employeeId}"
```

### Delete an employee

```bash
curl -X DELETE \
  -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/employees/{employeeId}"
```

## Tools

### List agent tools

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}/tools"
```

### Create an HTTP request tool

```bash
curl -X POST \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "lookup_customer",
    "display_name": "Kunden-Lookup",
    "description": "Looks up customer info by phone number",
    "tool_type": "http_request",
    "config": {
      "url": "https://crm.example.com/api/customers",
      "method": "GET",
      "headers": [{ "key": "Authorization", "value": "Bearer YOUR_CRM_KEY" }],
      "timeout": 30,
      "speak_during_execution": true,
      "speak_after_execution": true
    },
    "priority": 1,
    "is_enabled": true
  }' \
  "$TP_BASE/agents/{agentId}/tools"
```

### Create a transfer call tool

```bash
curl -X POST \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "transfer_to_support",
    "description": "Transfers the call to human support",
    "tool_type": "transfer_call",
    "config": {
      "phone_number": "+4930999888",
      "transfer_mode": "cold",
      "play_hold_music": false
    }
  }' \
  "$TP_BASE/agents/{agentId}/tools"
```

## Forwarding Slots

### List forwarding slots

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}/forwarding-slots"
```

### Replace all forwarding slots (bulk)

```bash
curl -X PUT \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "slots": [
      {
        "slotnummer": "01",
        "cases": "Rechnungsfragen, Zahlungsprobleme",
        "slot_belegung": "Max Mustermann",
        "prioritaet_1": "+491701234567",
        "prioritaet_2": "+491709876543"
      },
      {
        "slotnummer": "02",
        "cases": "Technischer Support",
        "slot_belegung": "Erika Musterfrau",
        "prioritaet_1": "+491705555555"
      }
    ]
  }' \
  "$TP_BASE/agents/{agentId}/forwarding-slots"
```

## Knowledge Base

### Add a document

```bash
curl -X POST \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Oeffnungszeiten",
    "content": "Montag - Freitag: 08:00 - 18:00 Uhr\nSamstag: 09:00 - 14:00 Uhr\nSonntag: geschlossen"
  }' \
  "$TP_BASE/agents/{agentId}/knowledge-base"
```

### Upload pre-processed chunks

```bash
# 1. Create the parent document first
DOC=$(curl -s -X POST \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "External Product Data", "content": ""}' \
  "$TP_BASE/agents/{agentId}/knowledge-base")

DOC_ID=$(echo $DOC | jq -r '.id')

# 2. Upload chunks with embeddings
curl -X POST \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "chunks": [
      {
        "content": "Produkt A kostet 29,99 EUR pro Monat...",
        "embedding": [0.012, -0.034, 0.056],
        "chunk_index": 0,
        "token_count": 150,
        "metadata": {"source": "pricing-system"}
      },
      {
        "content": "Produkt B kostet 49,99 EUR pro Monat...",
        "embedding": [0.023, -0.045, 0.067],
        "chunk_index": 1,
        "token_count": 120,
        "metadata": {"source": "pricing-system"}
      }
    ]
  }' \
  "$TP_BASE/agents/{agentId}/knowledge-base/$DOC_ID/chunks"

# 3. Mark document as completed
curl -X PATCH \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{"status": "completed"}' \
  "$TP_BASE/agents/{agentId}/knowledge-base/$DOC_ID"
```

### List chunks for a document

```bash
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/knowledge-base/{documentId}/chunks"
```

### Delete all chunks (before re-upload)

```bash
curl -X DELETE \
  -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/knowledge-base/{documentId}/chunks"
```

## Calls

### List recent calls

```bash
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/calls?limit=10"
```

### Filter calls by date range

```bash
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/calls?from=2026-03-01T00:00:00Z&to=2026-03-22T23:59:59Z"
```

### Search calls

```bash
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/calls?search=Rechnung"
```

### Get call with transcript

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/agents/{agentId}/calls/{callId}"
```

## Organization

### Get organization settings

```bash
curl -H "X-API-Key: $TP_KEY" "$TP_BASE/organization"
```

### Update organization settings

```bash
curl -X PATCH \
  -H "X-API-Key: $TP_KEY" \
  -H "Content-Type: application/json" \
  -d '{"auto_delete_done_calls": true}' \
  "$TP_BASE/organization"
```
