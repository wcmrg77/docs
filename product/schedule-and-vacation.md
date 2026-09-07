---
title: "Schedule & Backup"
description: "Set business hours and backup agents"
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

## Pausing an agent

There is no separate vacation mode any more. To pause an agent, switch it off in the agent list (or set `is_active: false` via the [Agents API](/api/agents)); calls then go to the backup agent, if one is configured. For planned absences of your staff, use the employee status (`urlaub`, `krank`, …) — see [Employee Management](/product/employee-management).
