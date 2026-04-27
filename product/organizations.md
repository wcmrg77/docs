---
title: "Organizations"
description: "Create and manage organizations with multi-org support"
---

Organizations are the top-level workspaces in TalkPilot. Agents, employees, and settings are all scoped to an organization.

## Organization management

Access via **Organisationen** in the sidebar (Super-Admin and Dev-Admin only).

### Each organization shows

- **Name** — Click to edit (owners only)
- **Agent count** — Number of agents
- **Member count** — Number of team members
- **Employee count** — Number of employees
- **Pending invites** — Unaccepted invitations

### Create an organization

1. Click **Organisation erstellen** (Create Organization)
2. Enter a name
3. The organization is created and you're the owner

### Invite members

1. Expand an organization card
2. Click **Einladen** (Invite)
3. Enter the email address
4. Select a role (see [Roles & Permissions](/roles-and-permissions))
5. The invitee receives an email to join

## Multi-organization view

**Available to:** Super-Admin and Dev-Admin only

When you manage multiple organizations, you can view combined data across all of them:

1. On the **Organisationen** page, toggle individual organizations as "active"
2. Use the **Master Toggle** to activate/deactivate all at once
3. When active, data pages (Anrufe, Mitarbeiter, Agenten, Telefonnummern) show combined data from all active organizations

The active organization selection is saved locally and persists between sessions.

### How it affects data pages

| Page | Without multi-org | With multi-org |
|------|-------------------|----------------|
| Anrufe | Calls from current org only | Calls from all active orgs |
| Mitarbeiter | Employees from current org | Employees from all active orgs |
| Agenten | Agents from current org | Agents from all active orgs |
| Telefonnummern | Numbers from current org | Numbers from all active orgs |

## Switching organizations

Regular users (Client-Admin, Client-Employee) work within a single organization at a time. If you belong to multiple organizations, switch between them in the organization selector.
