---
title: "Employee Management"
description: "Manage employee records, status, and availability"
---

Employees are the human staff members your AI agents can route calls to. Manage their availability, contact information, and notification preferences on the **Mitarbeiter** page.

## Employee statuses

Each employee has a status indicating their availability:

| Status | Color | Meaning |
|--------|-------|---------|
| **Anwesend** (Present) | Green | Available for calls |
| **Urlaub** (Vacation) | Orange | Away on vacation |
| **Krank** (Sick) | Red | On sick leave |
| **Weiterbildung** (Training) | Blue | In training |
| **Notdienst** (Emergency) | Purple | On-call for emergencies |

Change an employee's status by clicking the status badge on their card and selecting the new status from the dropdown.

## Employee cards

Each employee is displayed as a card showing:

- **Name** — Click to edit inline
- **Phone number** — Contact number for call transfers
- **Email** — Click to edit inline
- **Status badge** — Click to change status
- **Active toggle** — Enable/disable the employee
- **Email notifications** — Toggle whether the employee receives call-related emails (`get_mail`)

## Adding employees

1. Go to **Mitarbeiter** in the sidebar
2. Click **Mitarbeiter hinzufuegen** (Add Employee)
3. Fill in the details:
   - **Name** (required)
   - **Phone number** (optional)
   - **Email** (optional)
   - **Select agents** — Which agents this employee is linked to
4. Click **Speichern** (Save)

## Forwarding rules

When creating or editing an employee, you can set up forwarding rules:

| Field | Description |
|-------|-------------|
| **Start date** | When the forwarding takes effect |
| **Substitute** | Who handles this employee's calls while away |
| **Responsibilities** | What this employee is responsible for |
| **Reasons** | Why calls should be forwarded (e.g., vacation, training) |

When a forwarding rule is created, a webhook notification is sent (if configured in profile settings).

## Statistics

At the top of the Mitarbeiter page, you'll see:

- **Total employees** — Count across all linked agents
- **Present** — Number currently available
- **Absent** — Number currently unavailable

## Via API

Manage employees programmatically — see [Employees API](/api/employees). Common use case: sync employee status from your HR system.
