---
title: "Agent Setup"
description: "Create and configure AI agents in the Dashboard"
---

Agents are AI voice bots that answer phone calls. Each agent has its own phone number, personality, voice, and configuration.

## Creating an agent

How you create an agent depends on your role:

| Role | Method |
|------|--------|
| **Super-Admin / Dev-Admin** | Create from scratch with name, phone number, and organization |
| **Client-Admin / Client-Employee** | Add an existing agent by entering its phone number |

### Create from scratch (Admin)

1. Go to **Agenten** in the sidebar
2. Click **Neuer Agent** (New Agent)
3. Enter a name and select the phone number
4. The agent is created with default settings — configure it in the detail page

### Add by phone number (Client)

1. Go to **Agenten** in the sidebar
2. Click **Agent hinzufuegen** (Add Agent)
3. Enter the agent's phone number
4. The agent appears in your list

## Agent detail page

Click on any agent to open the detail page. It's organized into sections:

| Section | What you configure |
|---------|-------------------|
| **Basic Info** | Name, language, formality, active status |
| **Prompt** | System prompt and greeting message |
| **LLM** | Language model provider and settings |
| **Voice** | TTS/STT providers, voice selection, speaking rate |
| **Tools** | Functions the agent can use during calls |
| **Knowledge Base** | Documents for the agent to reference |
| **Employees** | Human staff linked to this agent |
| **Forwarding Slots** | Call routing rules |
| **Schedule** | Business hours and backup agent |
| **Webhook** | Post-call notification URL |

## Basic settings

| Setting | Options | Description |
|---------|---------|-------------|
| **Name** | Free text | Display name (click to edit inline) |
| **Language** | de, en, fr, es | Agent's conversation language |
| **Formality** | formal (Sie), informal (Du) | How the agent addresses callers |
| **Active** | Toggle | Whether the agent accepts calls |

## Advanced settings

| Setting | Default | Description |
|---------|---------|-------------|
| **Allow greeting interruption** | true | Let callers speak before the greeting finishes |
| **Short calls auto-edit** | false | Automatically mark very short calls as done |
| **Track caller name** | true | Ask for and record the caller's name |
| **Transfer timeout** | 30s | How long to wait for a transfer (10-120 seconds) |

## Role-based access

Not all users see the same settings:

| Section | Super-Admin / Dev-Admin | Client-Admin | Client-Employee |
|---------|:-----------------------:|:------------:|:---------------:|
| Basic Info | Full | Limited | Read-only |
| Prompt | Full | Greeting only | Read-only |
| LLM | Full | Hidden | Hidden |
| Voice | Full | Full | Read-only |
| Tools | Full | Hidden | Hidden |
| Knowledge Base | Full | Hidden | Hidden |
| Employees | Full | Full | Read-only |
| Schedule | Full | Full | Read-only |

See [Roles & Permissions](/roles-and-permissions) for the complete access matrix.
