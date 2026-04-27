---
title: "Prompt & Greeting"
description: "Write effective system prompts and greeting messages"
---

The system prompt and greeting define your agent's personality, behavior, and how it opens conversations.

## System prompt

The system prompt is the most important configuration for your agent. It tells the AI how to behave, what information to collect, and how to handle different situations.

**Max length:** 50,000 characters

### What to include

- **Role description** — Who the agent is and what company it represents
- **Behavior rules** — How to handle different caller scenarios
- **Information to collect** — What data to gather (name, phone, email, reason)
- **Escalation rules** — When to transfer to a human
- **Knowledge instructions** — How to use the knowledge base
- **Tone and style** — Professional, friendly, formal, casual

### Example prompt

```
Du bist der freundliche Telefonassistent der Mustermann GmbH.

Deine Aufgaben:
- Begruesse den Anrufer und frage nach seinem Anliegen
- Erfasse den Namen des Anrufers
- Leite Rechnungsfragen an Slot 01 weiter
- Leite technische Fragen an Slot 02 weiter
- Beantworte allgemeine Fragen aus der Wissensdatenbank
- Wenn du die Antwort nicht weisst, biete einen Rueckruf an

Verhalte dich stets hoeflich und professionell. Sprich den Kunden mit "Sie" an.
```

### Tips

- Be specific about what the agent should and shouldn't do
- Include example phrases the agent should use
- Define clear escalation criteria
- Test with real scenarios and iterate

## Greeting message

The greeting is the first thing the agent says when a call starts.

**Max length:** 2,000 characters

### Example greetings

```
Guten Tag, Sie sprechen mit der Mustermann GmbH. Mein Name ist Lisa, wie kann ich Ihnen helfen?
```

```
Willkommen bei Firma X. Wie kann ich Ihnen heute weiterhelfen?
```

### Allow greeting interruption

When enabled, callers can start speaking before the greeting finishes. The agent will stop its greeting and listen.

- **Enabled** (default) — Better for experienced callers who know what they want
- **Disabled** — Ensures the full greeting is heard (useful for important disclaimers)

Configure this in the agent's advanced settings or via the API:

```bash
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"allow_greeting_interruption": false}' \
  "$TP_BASE/agents/{agentId}"
```

## Who can edit

| Role | Prompt | Greeting |
|------|:------:|:--------:|
| Super-Admin / Dev-Admin | Edit | Edit |
| Client-Admin | Hidden | Edit |
| Client-Employee | Hidden | Read-only |
