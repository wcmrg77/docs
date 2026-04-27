---
title: "Forwarding Slots"
description: "Configure priority-based call routing rules"
---

Forwarding slots tell your AI agent how to route calls to the right person. Each slot handles specific case types and has up to 3 priority levels for fallback routing.

## How it works

1. A caller describes their issue to the AI agent
2. The agent matches the issue to a forwarding slot based on the **cases** field
3. The agent tries to reach **Priority 1** first
4. If Priority 1 is unavailable, it tries **Priority 2**, then **Priority 3**
5. If no one is available, the agent handles the call based on its prompt instructions

## Slot configuration

| Field | Description | Example |
|-------|-------------|---------|
| **Slot number** | Sequential identifier | `01`, `02`, `03` |
| **Cases** | Comma-separated topics this slot handles | `Rechnungsfragen, Zahlungsprobleme` |
| **Slot assignment** | Name of the person responsible | `Max Mustermann` |
| **Priority 1** | Primary contact (phone or name) | `+491701234567` |
| **Priority 2** | First fallback | `+491709876543` |
| **Priority 3** | Second fallback | `+491705555555` |

## Managing slots in the Dashboard

Forwarding slots are configured per agent on the agent detail page:

1. Open **Agenten** > click an agent
2. Scroll to **Weiterleitungsregeln** (Forwarding Rules)
3. Add, edit, or remove slots
4. Changes are saved when you click **Speichern**

## Example setup

| Slot | Cases | Assignment | Priority 1 | Priority 2 |
|------|-------|------------|------------|------------|
| 01 | Rechnungsfragen, Zahlungen | Max Mustermann | +491701234567 | +491709876543 |
| 02 | Technischer Support | Erika Musterfrau | +491705555555 | +491706666666 |
| 03 | Vertraege, Kuendigungen | Anna Schmidt | +491701111111 | — |

## Bulk sync via API

If you manage routing in an external system, use the `PUT` endpoint to replace all slots at once:

```bash
curl -X PUT -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"slots": [{"slotnummer": "01", "cases": "Billing", "prioritaet_1": "+49170..."}]}' \
  "$TP_BASE/agents/{agentId}/forwarding-slots"
```

See [Forwarding Slots API](/api/forwarding-slots) for details.
