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

**Available to:** Super-Admin, Dev-Admin, Client-Admin

See [Authentication](/authentication) and [API Keys API](/api/api-keys) for details.

## Trash (Papierkorb)

Manage soft-deleted calls:

| Action | Description |
|--------|-------------|
| **View** | Browse all calls in the trash |
| **Restore** | Move a call back to the active list |
| **Delete** | Permanently remove a call from the database |
| **Empty trash** | Delete all trashed calls at once |

### Auto-delete done calls

Toggle per organization: when enabled, calls marked as "done" are automatically moved to the trash. They can still be restored from the trash if needed.

**Available to:** Super-Admin, Dev-Admin, Client-Admin
