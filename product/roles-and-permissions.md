---
title: "Roles & Permissions"
description: "Understand the role-based access control system"
---

TalkPilot uses a 4-tier role system to control access to features and data.

## Roles

### Super-Admin

Full access to everything. Can see all organizations, all agents, and all data.

### Dev-Admin

Can create agents and manage phone numbers. Can only see agents they created. Full configuration access on their agents.

### Client-Admin

Can edit assigned agents with limited access (greeting, voice, employees, schedule). Cannot create agents or access phone numbers. Can add existing agents by phone number.

### Client-Employee

Read-only access to assigned agents. Can add existing agents by phone number. Cannot edit any configuration.

## Permission matrix

| Feature | Super-Admin | Dev-Admin | Client-Admin | Client-Employee |
|---------|:-----------:|:---------:|:------------:|:---------------:|
| **View all agents** | All | Own only | Assigned only | Assigned only |
| **Create agents** | Yes | Yes | No | No |
| **Add agent by phone** | No | No | Yes | Yes |
| **Edit agent prompt** | Yes | Yes | No | No |
| **Edit agent greeting** | Yes | Yes | Yes | No |
| **Edit LLM settings** | Yes | Yes | No | No |
| **Edit voice settings** | Yes | Yes | Yes | No |
| **Configure tools** | Yes | Yes | No | No |
| **Manage knowledge base** | Yes | Yes | No | No |
| **Manage employees** | Yes | Yes | Yes | No |
| **Configure forwarding** | Yes | Yes | Yes | No |
| **Edit schedule** | Yes | Yes | Yes | No |
| **View calls** | Yes | Yes | Yes | Yes |
| **Manage calls (done, notes)** | Yes | Yes | Yes | No |
| **Delete agents** | Yes | Yes | No | No |
| **Phone numbers page** | Yes | Yes | No | No |
| **Organizations page** | Yes | Yes | No | No |
| **Create organizations** | Yes | Yes | No | No |
| **API key management** | Yes | Yes | Yes | No |
| **Trash management** | Yes | Yes | Yes | No |

## Client-Admin limited access mode

When a Client-Admin opens an agent detail page, they see a simplified view:

- **Visible:** Greeting, voice/speech settings, language, employees, forwarding slots, schedule
- **Hidden:** System prompt, LLM settings, tools, knowledge base
- **Save button:** Shows "Save Greeting" instead of "Save"

This ensures clients can customize their agent's voice and behavior without modifying the core AI configuration.

## Route access

| Route | Required roles |
|-------|---------------|
| `/agenten/new` | super_admin, dev_admin |
| `/telefonnummern` | super_admin, dev_admin |
| `/organisationen` | super_admin, dev_admin |
| All other routes | Any authenticated role |
