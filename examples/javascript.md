---
title: "JavaScript / TypeScript Examples"
description: "JavaScript and TypeScript code examples"
---

A lightweight wrapper around `fetch` for the TalkPilot API — no external dependencies required.

## Setup

```typescript
const BASE_URL = "https://api.talkpilot.io/v1";
const API_KEY = "tp_live_YOUR_KEY_HERE";

async function talkpilot(
  path: string,
  options: RequestInit = {}
): Promise<any> {
  const response = await fetch(`${BASE_URL}${path}`, {
    ...options,
    headers: {
      "X-API-Key": API_KEY,
      "Content-Type": "application/json",
      ...options.headers,
    },
  });

  // Handle rate limiting with automatic retry
  if (response.status === 429) {
    const retryAfter = parseInt(response.headers.get("Retry-After") || "60");
    console.log(`Rate limited. Retrying in ${retryAfter}s...`);
    await new Promise((r) => setTimeout(r, retryAfter * 1000));
    return talkpilot(path, options);
  }

  if (response.status === 204) return null;

  const body = await response.json();

  if (!response.ok) {
    const err = body.error;
    throw new Error(`[${err.code}] ${err.message} (${err.request_id})`);
  }

  return body;
}
```

## List agents

```typescript
const { data: agents, pagination } = await talkpilot("/agents");

console.log(`Found ${pagination.total} agents:`);
agents.forEach((agent) => {
  console.log(`- ${agent.name} (${agent.is_active ? "active" : "inactive"})`);
});
```

## Get agent details

```typescript
const agent = await talkpilot(`/agents/${agentId}`);

console.log(`Agent: ${agent.name}`);
console.log(`Model: ${agent.llm_provider}/${agent.llm_model}`);
console.log(`Voice: ${agent.voice_id}`);
console.log(`Active: ${agent.is_active}`);
```

## Update agent settings

```typescript
// Update prompt and greeting
const updated = await talkpilot(`/agents/${agentId}`, {
  method: "PATCH",
  body: JSON.stringify({
    prompt: "Du bist ein professioneller Kundenservice-Agent fuer Firma X.",
    greeting: "Willkommen bei Firma X, wie kann ich Ihnen helfen?",
  }),
});

console.log(`Agent updated: ${updated.name}`);
```

## Toggle vacation mode

```typescript
// Enable vacation mode
await talkpilot(`/agents/${agentId}`, {
  method: "PATCH",
  body: JSON.stringify({
    vacation_mode: true,
    vacation_end: "2026-04-01T00:00:00Z",
    vacation_notdienst: true,
  }),
});

// Disable vacation mode
await talkpilot(`/agents/${agentId}`, {
  method: "PATCH",
  body: JSON.stringify({ vacation_mode: false }),
});
```

## Manage employees

```typescript
// List employees for an agent
const { data: employees } = await talkpilot(
  `/agents/${agentId}/employees?status=anwesend`
);

// Create a new employee
const newEmployee = await talkpilot(`/agents/${agentId}/employees`, {
  method: "POST",
  body: JSON.stringify({
    name: "Max Mustermann",
    phone_number: "+491701234567",
    email: "max@example.com",
    status: "anwesend",
    active: true,
    get_mail: true,
  }),
});

// Update employee status (e.g., mark as on vacation)
await talkpilot(`/agents/${agentId}/employees/${employeeId}`, {
  method: "PATCH",
  body: JSON.stringify({ status: "urlaub" }),
});

// Delete an employee
await talkpilot(`/agents/${agentId}/employees/${employeeId}`, {
  method: "DELETE",
});
```

## Sync forwarding slots from external system

```typescript
// Replace all forwarding slots in one request
const result = await talkpilot(`/agents/${agentId}/forwarding-slots`, {
  method: "PUT",
  body: JSON.stringify({
    slots: [
      {
        slotnummer: "01",
        cases: "Rechnungsfragen, Zahlungsprobleme",
        slot_belegung: "Max Mustermann",
        prioritaet_1: "+491701234567",
        prioritaet_2: "+491709876543",
      },
      {
        slotnummer: "02",
        cases: "Technischer Support",
        slot_belegung: "Erika Musterfrau",
        prioritaet_1: "+491705555555",
      },
    ],
  }),
});

console.log(`Replaced ${result.replaced_count} slots with ${result.data.length} new slots`);
```

## Fetch calls with filtering

```typescript
// Recent calls from the last 7 days
const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString();

const { data: calls, pagination } = await talkpilot(
  `/agents/${agentId}/calls?from=${sevenDaysAgo}&limit=50`
);

console.log(`${pagination.total} calls in the last 7 days`);

// Get full call details with transcript
const call = await talkpilot(`/agents/${agentId}/calls/${callId}`);

console.log(`Customer: ${call.customer_name}`);
console.log(`Duration: ${call.duration}s`);
console.log(`Summary: ${call.call_summary}`);
if (call.transcript) {
  console.log(`Transcript: ${call.transcript}`);
}
```

## Fetch all pages

```typescript
async function fetchAll<T>(path: string): Promise<T[]> {
  const allItems: T[] = [];
  let page = 1;
  let totalPages = 1;

  while (page <= totalPages) {
    const response = await talkpilot(`${path}${path.includes("?") ? "&" : "?"}page=${page}&limit=100`);
    allItems.push(...response.data);
    totalPages = response.pagination.pages;
    page++;
  }

  return allItems;
}

// Usage
const allCalls = await fetchAll(`/agents/${agentId}/calls?from=2026-03-01T00:00:00Z`);
console.log(`Fetched ${allCalls.length} calls total`);
```

## Add knowledge base content

```typescript
const doc = await talkpilot(`/agents/${agentId}/knowledge-base`, {
  method: "POST",
  body: JSON.stringify({
    title: "Preisliste 2026",
    content: `
      Produkt A: 29,99 EUR/Monat
      Produkt B: 49,99 EUR/Monat
      Produkt C: 99,99 EUR/Monat
      Alle Preise zzgl. MwSt.
    `.trim(),
  }),
});

console.log(`Document created: ${doc.title} (status: ${doc.status})`);
```

## Upload pre-processed chunks with embeddings

```typescript
import OpenAI from "openai";

const openai = new OpenAI();

async function getEmbeddings(texts: string[]): Promise<number[][]> {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: texts,
  });
  return response.data.map((item) => item.embedding);
}

interface ChunkInput {
  content: string;
  metadata?: Record<string, unknown>;
}

async function uploadChunks(
  agentId: string,
  title: string,
  chunksData: ChunkInput[]
) {
  // 1. Create parent document
  const doc = await talkpilot(`/agents/${agentId}/knowledge-base`, {
    method: "POST",
    body: JSON.stringify({ title, content: "" }),
  });
  const docId = doc.id;

  // 2. Generate embeddings
  const texts = chunksData.map((c) => c.content);
  const embeddings = await getEmbeddings(texts);

  // 3. Build chunk payloads
  const chunks = chunksData.map((chunk, i) => ({
    content: chunk.content,
    embedding: embeddings[i],
    chunk_index: i,
    token_count: chunk.content.split(/\s+/).length,
    metadata: chunk.metadata ?? {},
  }));

  // 4. Upload chunks
  const result = await talkpilot(
    `/agents/${agentId}/knowledge-base/${docId}/chunks`,
    {
      method: "POST",
      body: JSON.stringify({ chunks }),
    }
  );
  console.log(`Uploaded ${result.inserted_count} chunks for "${title}"`);

  // 5. Mark document as completed
  await talkpilot(`/agents/${agentId}/knowledge-base/${docId}`, {
    method: "PATCH",
    body: JSON.stringify({ status: "completed" }),
  });

  return docId;
}

// Usage
await uploadChunks(agentId, "Product Catalog", [
  { content: "Produkt A kostet 29,99 EUR...", metadata: { category: "pricing" } },
  { content: "Produkt B kostet 49,99 EUR...", metadata: { category: "pricing" } },
]);
```

## Node.js example (complete script)

```typescript
// talkpilot-sync.ts
// Sync employee statuses from your HR system to TalkPilot

const BASE_URL = process.env.TALKPILOT_BASE_URL!;
const API_KEY = process.env.TALKPILOT_API_KEY!;
const AGENT_ID = process.env.TALKPILOT_AGENT_ID!;

async function talkpilot(path: string, options: RequestInit = {}) {
  const res = await fetch(`${BASE_URL}${path}`, {
    ...options,
    headers: { "X-API-Key": API_KEY, "Content-Type": "application/json", ...options.headers },
  });
  if (res.status === 204) return null;
  const body = await res.json();
  if (!res.ok) throw new Error(`[${body.error.code}] ${body.error.message}`);
  return body;
}

// Simulated HR data
const hrEmployees = [
  { name: "Max Mustermann", status: "anwesend" },
  { name: "Erika Musterfrau", status: "urlaub" },
];

async function syncStatuses() {
  const { data: employees } = await talkpilot(`/agents/${AGENT_ID}/employees`);

  for (const hrEmployee of hrEmployees) {
    const match = employees.find((e: any) => e.name === hrEmployee.name);
    if (match && match.status !== hrEmployee.status) {
      await talkpilot(`/agents/${AGENT_ID}/employees/${match.id}`, {
        method: "PATCH",
        body: JSON.stringify({ status: hrEmployee.status }),
      });
      console.log(`Updated ${match.name}: ${match.status} -> ${hrEmployee.status}`);
    }
  }
}

syncStatuses().catch(console.error);
```
