---
title: "Roles & Permissions"
description: "Understand the role-based access control system"
---

TalkPilot uses a 4-tier role system to control access to features and data.

## Roles

### Dev-Admin

Can create agents and manage phone numbers. Can only see agents they created. Full configuration access on their agents.

### Dev-Employee

Read-only counterpart to Dev-Admin: sees everything a Dev-Admin sees in their assigned organizations (full agent details incl. prompt/LLM/tools/knowledge base, phone numbers page, organizations page, performance control) but has **zero write rights anywhere**. Enforced on all three layers: UI (hidden/disabled controls), Edge API (not in `JWT_WRITE_ROLES` → 403 on all writes), and RLS (`is_read_only_role()` guard on all member-reachable write policies). Assigned manually via SQL or via the invite dialog (option visible to Dev-Admins only). Must never be org owner (`user_organizations.is_owner = false`) and must have `profiles.admin = false`.

### Client-Admin

Can edit assigned agents with limited access (greeting, voice, employees, schedule). Cannot create agents or access phone numbers. Can add existing agents by phone number.

### Client-Employee

Read-only access to assigned agents. Can add existing agents by phone number. Cannot edit any configuration.

## Permission matrix

| Feature | Dev-Admin | Dev-Employee | Client-Admin | Client-Employee |
|---------|:---------:|:------------:|:------------:|:---------------:|
| **View all agents** | Own only | Assigned orgs (view) | Assigned only | Assigned only |
| **Create agents** | Yes | No | No | No |
| **Add agent by phone** | No | No | Yes | Yes |
| **View agent prompt/LLM/tools/KB** | Yes | Yes (read-only) | No | No |
| **Edit agent prompt** | Yes | No | No | No |
| **Edit agent greeting** | Yes | No | Yes | No |
| **Edit LLM settings** | Yes | No | No | No |
| **Edit voice settings** | Yes | No | Yes | No |
| **Configure tools** | Yes | No | No | No |
| **Manage knowledge base** | Yes | No | No | No |
| **Manage employees** | Yes | No | Yes | No |
| **Configure forwarding** | Yes | No | Yes | No |
| **Edit schedule** | Yes | No | Yes | No |
| **View calls** | Yes | Yes | Yes | Yes |
| **Manage calls (done, notes)** | Yes | No | Yes | No |
| **Delete agents** | Yes | No | No | No |
| **Phone numbers page** | Yes | View only | No | No |
| **Organizations page** | Yes | View only | No | No |
| **Performance control page** | Yes | Yes | No | No |
| **Create organizations** | Yes | No | No | No |
| **Invite users** | Yes | No | Yes | No |
| **API key management** | Yes | No | Yes | No |
| **Trash management** | Yes | No | Yes | No |

## Client-Admin limited access mode

When a Client-Admin opens an agent detail page, they see a simplified view:

- **Visible:** Greeting, voice/speech settings, language, employees, forwarding slots, schedule
- **Hidden:** System prompt, LLM settings, tools, knowledge base
- **Save button:** Shows "Save Greeting" instead of "Save"

This ensures clients can customize their agent's voice and behavior without modifying the core AI configuration.

## Route access

| Route | Required roles |
|-------|---------------|
| `/telefonnummern` | dev_admin, dev_employee |
| `/organisationen` | dev_admin, dev_employee |
| `/performance-control` | dev_admin, dev_employee |
| All other routes | Any authenticated role |

## How dev_employee is enforced (technical)

- **DB constraint**: `profiles_role_check` includes `dev_employee` (and `blocked` for the no-invite safety net).
- **RLS helpers**: `is_dev_employee()` grants the dev-level SELECTs (phone_numbers via membership, all profiles, org memberships, call_recordings/call_statistics). `is_read_only_role()` is appended as `AND NOT is_read_only_role()` to **every** write policy a plain org member can reach — org-scoped ones (calls, customer_database, employees, kb_*, calendar_*, agent_admin, mitarbeiter-slotting) plus the legacy `agents`-based ones (`agents` itself and the `agent_phone_number`-derived write paths on calls/employees), the mislabelled `service_role_all` policies on kb_documents/kb_chunks, and the org-owner policy on user_organizations.
- **Verified, not assumed**: a rollback-only negative test as a simulated dev_employee attempted a no-op UPDATE and a DELETE against all 40 public tables plus targeted INSERTs — zero rows written anywhere, while reads (calls, agents, tools, phone numbers, statistics, profiles, memberships) stayed visible. The same test confirmed client_admin/client_employee/dev_admin still write exactly as before. This test found the legacy `agents` gap after the first migration, which is why a second one exists.
- **Edge API**: `dev_employee` is intentionally absent from `JWT_WRITE_ROLES` in `supabase/functions/api/lib/auth.ts` → every `*:write` permission returns 403.
- **Signup hardening**: `handle_new_user()` whitelists roles from signup metadata and validates the `invite_token` against `organisation_invites` — self-signup cannot claim a role or org membership.
- **Assignment invariants**: `profiles.admin = false` and `user_organizations.is_owner = false` are load-bearing — the admin flag and org ownership would grant writes through other policies.
