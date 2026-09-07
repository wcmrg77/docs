---
title: "Settings"
description: "Configure organization, security, and appearance settings"
---

The Settings page (`/settings`) contains account, organization, and system configuration.

## Organization settings

- **Organization name** — Display name for your workspace
- **Created date** — When the organization was created
- **Your role** — Your role within this organization
- **Invite users** — Send invitations to new team members (admins only)
- **Calendar status sync** — Update employee status automatically from calendar events
- **Forward calls by email** — Allow single calls to be forwarded from the call detail sheet to
  the employee assigned to them (off by default; see
  [Call management](/product/call-management#forward-by-email))

## Security

- **Change password** — Update your account password
- **Session management** — View and manage active sessions

## Appearance

- **Theme** — Switch between light and dark mode
- **Language** — Interface language preference

## API Keys

Create and manage API keys for programmatic access to TalkPilot.

| Action | Description |
|--------|-------------|
| **Create key** | Set name, permissions, rate limits, expiration, agent restrictions |
| **Toggle active** | Enable/disable a key without deleting it |
| **Delete** | Permanently revoke a key |
| **View info** | See creation date, last used, expiration, permissions |

The raw API key is shown only once at creation time. Store it securely.

**Available to:** Dev-Admin

See [Authentication](/authentication) and [API Keys API](/api/api-keys) for details.

## Trash (Papierkorb)

Manage soft-deleted calls:

| Action | Description |
|--------|-------------|
| **View** | Browse all calls in the trash |
| **Restore** | Move a call back to the active list |
| **Delete** | Permanently remove a call from the customer's view (cannot be undone) |
| **Empty trash** | Remove all trashed calls at once |

Deleting a call sets `calls.hidden_at`. The row and all of its data stay in the database — the call is simply no longer visible in the dashboard or the public API, and it cannot be restored from the UI. The data is removed by the regular retention cleanup once the agent's `retention_days` (default 90) have passed, or on a GDPR erasure request.

Dev-Admins see a separate **Permanently deleted calls** card below the trash listing these hidden calls (read-only).

### Auto-delete done calls

Toggle per organization: when enabled, calls marked as "done" are automatically moved to the trash. They can still be restored from the trash if needed.

**Available to:** Dev-Admin, Client-Admin
