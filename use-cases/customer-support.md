---
title: "Customer Support Agent"
description: "Build an AI-powered customer support hotline"
---

Set up an AI agent that handles first-level customer support — answering product questions, looking up customer data, and escalating complex issues.

## Scenario

A SaaS company or service provider wants to automate their support hotline. The AI agent:

- Identifies the customer and their issue
- Looks up account data from the CRM
- Answers common questions from the knowledge base
- Creates support tickets for complex issues
- Escalates to human support when needed

## Setup steps

### 1. Write the prompt

```
Du bist der Kundenservice-Agent von TechCorp GmbH.

Deine Aufgaben:
- Begruessung und Identifikation des Kunden (Name und Kundennummer erfragen)
- Nutze das CRM-Lookup-Tool, um Kundendaten abzurufen
- Beantworte haeufige Fragen aus der Wissensdatenbank
- Bei technischen Problemen: erfasse eine detaillierte Problembeschreibung
- Erstelle ein Support-Ticket ueber das HTTP-Tool, wenn das Problem nicht sofort geloest werden kann
- Leite an den technischen Support weiter (Slot 01) wenn:
  - Der Kunde ausdruecklich einen Menschen sprechen moechte
  - Das Problem zu komplex fuer eine telefonische Loesung ist
- Leite an die Buchhaltung weiter (Slot 02) bei Rechnungsfragen

Verhalten:
- Bleibe stets freundlich und loesungsorientiert
- Fasse das Anliegen kurz zusammen, bevor du eine Loesung vorschlaegst
- Biete immer eine Alternative an (Rueckruf, E-Mail, Ticket)
```

### 2. Set up the greeting

```
Willkommen beim TechCorp Kundenservice. Mein Name ist Alex, wie kann ich Ihnen helfen?
```

### 3. Add knowledge base

Upload product documentation, FAQs, and troubleshooting guides:

- **Product FAQ** — Common questions and answers
- **Troubleshooting Guide** — Step-by-step fixes for known issues
- **Pricing & Plans** — Current pricing, plan features, upgrade options
- **Account Management** — How to reset passwords, update billing, cancel

### 4. Configure tools

**HTTP Request: CRM Lookup**
```json
{
  "name": "lookup_customer",
  "description": "Look up customer by phone number in the CRM",
  "tool_type": "http_request",
  "config": {
    "url": "https://crm.techcorp.com/api/customers/search",
    "method": "GET",
    "headers": [{"key": "Authorization", "value": "Bearer YOUR_CRM_KEY"}],
    "query_params": [{"key": "phone", "value": "{{caller_phone}}"}],
    "timeout": 10,
    "speak_during_execution": true,
    "speak_after_execution": false
  }
}
```

**HTTP Request: Create Ticket**
```json
{
  "name": "create_ticket",
  "description": "Create a support ticket when the issue cannot be resolved immediately",
  "tool_type": "http_request",
  "config": {
    "url": "https://helpdesk.techcorp.com/api/tickets",
    "method": "POST",
    "headers": [{"key": "Authorization", "value": "Bearer YOUR_HELPDESK_KEY"}],
    "timeout": 15,
    "speak_during_execution": true,
    "speak_after_execution": true
  }
}
```

**Extract Variable: Customer Email**
```json
{
  "name": "capture_email",
  "description": "Capture the customer's email for ticket confirmation",
  "tool_type": "extract_variable",
  "config": {
    "variable_name": "email",
    "variable_type": "email",
    "prompt_text": "Unter welcher E-Mail-Adresse kann ich Ihnen die Ticketbestaetigung senden?",
    "example_format": "name@example.com",
    "confirmation_enabled": true
  }
}
```

**Transfer Call + End Call** — For escalation and call termination.

### 5. Configure forwarding slots

| Slot | Cases | Assignment | Priority 1 |
|------|-------|------------|------------|
| 01 | Technischer Support, Komplexe Probleme | Tech Team | +491701111111 |
| 02 | Rechnungen, Zahlungen, Abonnement | Buchhaltung | +491702222222 |

### 6. Set up post-call webhook

Configure a webhook to send call data to your CRM or helpdesk after each call:

```bash
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"post_call_webhook_url": "https://helpdesk.techcorp.com/webhooks/talkpilot"}' \
  "$TP_BASE/agents/{agentId}"
```

## Result

Most support inquiries are handled automatically — the agent answers questions, looks up customer data, and creates tickets. Complex issues are smoothly escalated to human agents with full context captured in the transcript and ticket.
