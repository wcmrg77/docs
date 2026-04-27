---
title: "Tools Configuration"
description: "Set up HTTP requests, call transfers, and other agent tools"
---

Tools extend what your agent can do during a call — from looking up customer data in your CRM to transferring calls to the right department.

## Adding a tool

1. Open the agent detail page
2. Scroll to the **Tools** section
3. Click **Tool hinzufuegen** (Add Tool)
4. Select the tool type from the dropdown
5. Configure the tool settings
6. Save

## Tool types

### HTTP Request

Make API calls to external systems during a conversation. Common uses: CRM lookup, ticket creation, appointment booking.

| Setting | Description |
|---------|-------------|
| **URL** | Target API endpoint |
| **Method** | GET, POST, PUT, or DELETE |
| **Headers** | Key-value pairs (e.g., Authorization) |
| **Query Parameters** | Key-value URL parameters |
| **Body** | JSON body (for POST/PUT) |
| **Timeout** | Max wait time in seconds |
| **Speak during execution** | Agent talks while waiting for the response |
| **Speak after execution** | Agent reads the result to the caller |

**Security:** URLs pointing to internal/private IP addresses are blocked (SSRF protection).

### Transfer Call

Transfer the call to a human agent or external number.

| Setting | Description |
|---------|-------------|
| **Phone number** | Target number |
| **Transfer mode** | **Cold** (immediate handoff) or **Warm** (agent stays on line briefly) |
| **Play hold music** | Music while transferring |

### Monitored Transfer

Like Transfer Call, but the AI stays on the line to monitor the transfer via SIP INVITE.

| Setting | Description |
|---------|-------------|
| **Phone number** | Target number |
| **Timeout** | How long to wait for the transfer (seconds) |

### End Call

Terminate the call with a goodbye message.

| Setting | Description |
|---------|-------------|
| **Goodbye message** | What the agent says before hanging up |

### Extract Variable

Extract structured data from the conversation (email, phone number, etc.).

| Setting | Description |
|---------|-------------|
| **Variable name** | Internal identifier |
| **Variable type** | email, phone, name, number, date, or custom |
| **Prompt text** | What the agent asks to collect the data |
| **Example format** | Example shown to the AI (e.g., `name@example.com`) |
| **Confirmation** | Ask the caller to confirm the extracted value |

### Play Tone / Press Digit

Play DTMF tones — useful when the agent needs to navigate an IVR system.

| Setting | Description |
|---------|-------------|
| **Digit** | DTMF digit to play |
| **Duration** | Tone duration in milliseconds |
| **Volume** | Tone volume (0-1) |

### Knowledge Base Search

Search the agent's knowledge base documents. See [Knowledge Base](/knowledge-base) for document management.

| Setting | Description |
|---------|-------------|
| **Top K** | Number of document chunks to retrieve |
| **Similarity threshold** | Minimum relevance score (0-1) |
| **Bridging sentence** | What the agent says while searching |

## Priority

Each tool has a priority number. Lower numbers = higher priority. When the AI decides to use a tool, priority helps resolve conflicts between similar tools.

## Enable / Disable

Toggle individual tools on or off without deleting them. Disabled tools are not available to the AI during calls.

## Who can edit

Only **Super-Admin** and **Dev-Admin** can configure tools. This section is hidden for Client-Admin and Client-Employee users.
