---
title: "CRM Integration"
description: "Integrate TalkPilot with your CRM system"
---

Integrate TalkPilot with your CRM system to sync call data, employee status, and customer records bidirectionally.

## Integration patterns

### 1. Push: Post-call webhook

TalkPilot sends call data to your CRM after every completed call.

**Setup:**
1. Configure the webhook URL on the agent (Dashboard or API)
2. Build a webhook receiver in your CRM or automation tool

**Webhook receiver example (n8n / Make / Zapier):**
1. Create a webhook trigger
2. Parse the incoming call data
3. Create or update a CRM record with the call summary, transcript, and customer info

**Express.js example:**

```javascript
app.post("/webhooks/talkpilot", async (req, res) => {
  const { call } = req.body;

  // Create a CRM record
  await crm.createActivity({
    customerPhone: call.phone_number,
    customerName: call.customer_name,
    subject: call.customer_request,
    notes: call.call_summary,
    duration: call.duration,
    success: call.general_success,
    recordingUrl: null, // Use the API to get the audio URL if needed
    date: call.created_at,
  });

  res.json({ received: true });
});
```

### 2. Pull: API polling

Periodically fetch new calls and employee data from TalkPilot.

**Use case:** Your CRM needs to display call history or doesn't support webhooks.

```python
import requests
from datetime import datetime, timedelta

BASE_URL = "https://{project_ref}.supabase.co/functions/v1/api/v1"
API_KEY = "tp_live_YOUR_KEY"
HEADERS = {"X-API-Key": API_KEY}

def sync_new_calls(agent_id, since):
    """Fetch all calls since the last sync."""
    calls = []
    page = 1

    while True:
        resp = requests.get(
            f"{BASE_URL}/agents/{agent_id}/calls",
            headers=HEADERS,
            params={"from": since, "page": page, "limit": 100},
        )
        data = resp.json()
        calls.extend(data["data"])

        if page >= data["pagination"]["pages"]:
            break
        page += 1

    return calls

# Run every 15 minutes
last_sync = (datetime.utcnow() - timedelta(minutes=15)).isoformat() + "Z"
new_calls = sync_new_calls("your-agent-id", last_sync)

for call in new_calls:
    crm.create_or_update_record(call)
```

### 3. Bidirectional: Full sync

Combine both patterns for a complete integration:

| Direction | What syncs | How |
|-----------|-----------|-----|
| TalkPilot → CRM | Call records after each call | Post-call webhook |
| TalkPilot → CRM | Call history (catch-up) | API polling |
| CRM → TalkPilot | Employee availability | API: PATCH employee status |
| CRM → TalkPilot | Forwarding rules | API: PUT forwarding slots |
| CRM → TalkPilot | Knowledge base updates | API: POST/PATCH KB documents |

## Employee status sync

Keep employee availability in sync between your HR system and TalkPilot:

```javascript
// Run on a schedule (every 5 minutes)
async function syncEmployeeStatus() {
  // Fetch current status from HR system
  const hrEmployees = await hr.getEmployees();

  // Fetch TalkPilot employees
  const { data: tpEmployees } = await talkpilot(`/agents/${AGENT_ID}/employees`);

  for (const hrEmp of hrEmployees) {
    const match = tpEmployees.find(e => e.email === hrEmp.email);
    if (!match) continue;

    // Map HR status to TalkPilot status
    const tpStatus = mapStatus(hrEmp.status); // e.g., "vacation" → "urlaub"

    if (match.status !== tpStatus) {
      await talkpilot(`/agents/${AGENT_ID}/employees/${match.id}`, {
        method: "PATCH",
        body: JSON.stringify({ status: tpStatus }),
      });
      console.log(`Updated ${match.name}: ${match.status} → ${tpStatus}`);
    }
  }
}

function mapStatus(hrStatus) {
  const map = {
    "active": "anwesend",
    "vacation": "urlaub",
    "sick": "krank",
    "training": "weiterbildung",
    "on_call": "notdienst",
  };
  return map[hrStatus] || "anwesend";
}
```

## Forwarding slot sync

Keep call routing rules in sync from your CRM or scheduling system:

```javascript
// Replace all slots from external system
async function syncForwardingSlots() {
  const departments = await crm.getDepartments();

  const slots = departments.map((dept, index) => ({
    slotnummer: String(index + 1).padStart(2, "0"),
    cases: dept.topics.join(", "),
    slot_belegung: dept.managerName,
    prioritaet_1: dept.primaryPhone,
    prioritaet_2: dept.backupPhone || null,
  }));

  await talkpilot(`/agents/${AGENT_ID}/forwarding-slots`, {
    method: "PUT",
    body: JSON.stringify({ slots }),
  });
}
```

## Recommended API key setup

For CRM integrations, create a dedicated API key with specific permissions:

| Integration | Permissions needed |
|-------------|-------------------|
| Read call records | `calls:read` |
| Sync employees | `employees:read`, `employees:write` |
| Sync forwarding slots | `forwarding:read`, `forwarding:write` |
| Update KB content | `kb:read`, `kb:write` |
| Full bidirectional | All of the above |

Scope the key to specific agents if the CRM only manages a subset of your agents.

## Related resources

- [Webhooks Guide](/guides/webhooks) — Webhook setup and best practices
- [Employees API](/api/employees) — Employee CRUD operations
- [Forwarding Slots API](/api/forwarding-slots) — Slot management with bulk replace
- [Calls API](/api/calls) — Read call records
