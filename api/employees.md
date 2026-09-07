---
title: "Employees"
description: "Manage employee records and availability status"
---

Employees are human staff members linked to agents for call routing. The AI agent uses employee information to decide who to transfer calls to or notify after a call.

## Status system

Each employee has a status indicating their availability:

| Status | Translation | Description |
|--------|------------|-------------|
| `anwesend` | Present | Available for calls |
| `urlaub` | Vacation | Away on vacation |
| `krank` | Sick | On sick leave |
| `weiterbildung` | Training | In training |
| `notdienst` | Emergency duty | On-call for emergencies |
| `im_termin` | In a meeting | Temporarily unavailable |
| `ausser_haus` | Out of office | Away from the office |

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Employee identifier |
| `name` | string | Employee name (required, max 255 chars) |
| `phone_number` | string | Phone number (max 30 chars) |
| `email` | string | Email address |
| `status` | string | One of the 7 statuses above |
| `active` | boolean | Whether the employee is active (default: `true`) |
| `get_mail` | boolean | Receives email notifications (default: `false`) |
| `agent_id` | uuid | Parent agent |
| `organization_id` | uuid | Organization (read-only, derived from the agent) |
| `back_at_work_at` | date | Expected return date while absent (`YYYY-MM-DD`) |
| `notdienst_bereich` | string | Emergency-duty area of the current shift, one of the organization's `notdienst_bereiche`. Only kept while `status` is `notdienst`; any other status clears it |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

## Endpoints

### List employees

```
GET /v1/agents/{agentId}/employees
```

**Permission:** `employees:read` | **Pagination:** yes

**Filters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by status (e.g., `?status=anwesend`); unknown values are ignored, not rejected |
| `active` | boolean | Filter by active state (e.g., `?active=true`) |

```bash
# List only present, active employees
curl -H "X-API-Key: $TP_KEY" \
  "$TP_BASE/agents/{agentId}/employees?status=anwesend&active=true"
```

### Get employee

```
GET /v1/agents/{agentId}/employees/{employeeId}
```

**Permission:** `employees:read`

### Create employee

```
POST /v1/agents/{agentId}/employees
```

**Permission:** `employees:write`

**Required fields:** `name`

```bash
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
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

### Update employee

```
PATCH /v1/agents/{agentId}/employees/{employeeId}
```

**Permission:** `employees:write`

Common use case: changing the status (e.g., marking an employee as on vacation). Updatable: `name`, `phone_number`, `email`, `status`, `active`, `get_mail`, `back_at_work_at`, `notdienst_bereich`; unknown fields are ignored.

```bash
# Mark as on vacation
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"status": "urlaub", "back_at_work_at": "2026-09-22"}' \
  "$TP_BASE/agents/{agentId}/employees/{employeeId}"

# Emergency duty for the electrical area
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"status": "notdienst", "notdienst_bereich": "Elektro"}' \
  "$TP_BASE/agents/{agentId}/employees/{employeeId}"
```

### Delete employee

```
DELETE /v1/agents/{agentId}/employees/{employeeId}
```

**Permission:** `employees:write` | Returns `204 No Content`

## Common patterns

### Sync employee status from HR system

```javascript
const hrEmployees = await fetchFromHR();
const { data: tpEmployees } = await talkpilot(`/agents/${agentId}/employees`);

for (const hr of hrEmployees) {
  const match = tpEmployees.find(e => e.name === hr.name);
  if (match && match.status !== hr.status) {
    await talkpilot(`/agents/${agentId}/employees/${match.id}`, {
      method: "PATCH",
      body: JSON.stringify({ status: hr.status }),
    });
  }
}
```

## Related resources

- [Agents](/api/agents) — Parent resource
- [Forwarding Slots](/api/forwarding-slots) — Call routing using employees
- [Employee Management](/product/employee-management) — Dashboard UI guide
