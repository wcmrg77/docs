---
title: "Schedule & Vacation"
description: "Set business hours, vacation mode, and backup agents"
---

Control when your agent is available and what happens outside business hours.

## Business hours

Set specific time windows when the agent accepts calls.

### Configuration

1. Open the agent detail page
2. Go to the **Zeitplan** (Schedule) section
3. Enable the schedule toggle
4. Add rules:
   - **Days** — Select weekdays (Mon-Sun)
   - **Start time** — When the agent starts accepting calls
   - **End time** — When the agent stops

You can add multiple rules for different days:

| Rule | Days | Start | End |
|------|------|-------|-----|
| 1 | Mon - Fri | 08:00 | 18:00 |
| 2 | Sat | 09:00 | 14:00 |

### Timezone

All schedule times use the configured timezone (default: `Europe/Berlin`). Set it via the API:

```json
{
  "schedule": {
    "timezone": "Europe/Berlin",
    "rules": [
      { "days": [1,2,3,4,5], "start_time": "09:00", "end_time": "17:00" }
    ]
  }
}
```

Days: `0` = Sunday, `1` = Monday, ..., `6` = Saturday.

## Outside business hours

When a call comes in outside the scheduled hours:

- If a **backup agent** is configured, the call goes to the backup
- If no backup is set, the call is not answered

### Backup agent

Select another agent from your organization as a fallback:

1. In the agent detail page, find **Backup Agent**
2. Select an agent from the dropdown
3. The backup agent handles calls when the primary is unavailable

The backup agent is indicated with a badge in the agent list.

## Vacation mode

Temporarily disable an agent without changing its schedule.

### Settings

| Setting | Description |
|---------|-------------|
| **Vacation mode** | Toggle on/off |
| **End date** | When vacation mode auto-disables (optional) |
| **Notdienst** (Emergency service) | Keep handling emergency calls during vacation |

### How it works

1. **Enable vacation mode** — Agent stops accepting regular calls
2. Set an **end date** (optional) — Vacation mode auto-disables at this time
3. Enable **Notdienst** — Agent still handles calls from employees marked with `notdienst` status

If a backup agent is configured, it handles calls while vacation mode is active.

### Via API

```bash
# Enable vacation with auto-disable and emergency service
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "vacation_mode": true,
    "vacation_end": "2026-04-01T00:00:00Z",
    "vacation_notdienst": true
  }' \
  "$TP_BASE/agents/{agentId}"
```
