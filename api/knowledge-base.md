---
title: "Knowledge Base"
description: "Add, update, and remove knowledge base documents"
---

The knowledge base is a RAG (Retrieval-Augmented Generation) document store. Upload documents with information an agent should reference during calls — product details, FAQs, pricing, opening hours, etc.

## Ownership model

Documents belong to an **organization**, not to a single agent. Which agent may use which document is a separate grant (`kb_agent_documents`), managed in the Dashboard under **Unternehmenswissen** (Company Knowledge) and on the agent's detail page.

The endpoints stay addressed per agent (`/v1/agents/{agentId}/knowledge-base`) and return everything that agent can search, with a `scope` field:

| `scope` | Meaning | Write access |
|---|---|---|
| `agent` | Created by this agent (through the API or its detail page); automatically granted to it | PATCH, DELETE, chunk endpoints |
| `organization` | Company knowledge uploaded in the Dashboard and granted to this agent | read-only — write operations return `403 FORBIDDEN`; manage it under Unternehmenswissen |

A document created through the API is assigned to the agent's organization **and** granted to that agent, so it is also visible in Company Knowledge and can be granted to further agents from there.

## Document lifecycle

```
Create → pending → processing → ready (searchable)
                               → error / failed (check error_message)
```

When a document is created, the processing pipeline picks it up (`pending`), chunks and embeds the `content` for vector search and sets the status to `ready`. This runs asynchronously — poll the document status to check when it's done. The status cannot be set through the API.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Document identifier |
| `agent_id` | uuid | Creating agent; null for organization-level documents |
| `org_id` | uuid | Owning organization |
| `scope` | string | `agent` or `organization` — see above |
| `title` | string | Document title |
| `content` | string | Text content (null for file-based documents) |
| `source_type` | string | `manual` for documents created via the API; Dashboard uploads carry their file type or `website` |
| `source_url` | string | Source URL (Dashboard uploads and website imports only) |
| `status` | string | `pending`, `processing`, `ready`, `error`, `failed` |
| `chunk_count` | integer | Number of chunks created for embedding |
| `error_message` | string | Error details (if status is `error`) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |

## Endpoints

### List documents

```
GET /v1/agents/{agentId}/knowledge-base
```

**Permission:** `kb:read` | **Pagination:** yes

**Filters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by processing status (`pending`, `processing`, `ready`, `error`, `failed`) |

### Get document

```
GET /v1/agents/{agentId}/knowledge-base/{documentId}
```

**Permission:** `kb:read`

### Create document

```
POST /v1/agents/{agentId}/knowledge-base
```

**Permission:** `kb:write`

Creates a text-based document (`source_type: manual`). The content is chunked and embedded by the processing pipeline.

**Note:** File uploads and website imports are only available through the Dashboard UI.

```bash
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "title": "Oeffnungszeiten",
    "content": "Montag - Freitag: 08:00 - 18:00 Uhr\nSamstag: 09:00 - 14:00 Uhr\nSonntag: geschlossen"
  }' \
  "$TP_BASE/agents/{agentId}/knowledge-base"
```

### Update document

```
PATCH /v1/agents/{agentId}/knowledge-base/{documentId}
```

**Permission:** `kb:write`

Updatable: `title`, `content`. If `content` changes, the status goes back to `pending` and the pipeline re-chunks and re-embeds the document. Other fields (including `status`) are ignored.

### Delete document

```
DELETE /v1/agents/{agentId}/knowledge-base/{documentId}
```

**Permission:** `kb:write` | Returns `204 No Content`

Deletes the document and all associated chunks and embeddings.

## Direct Chunk API

For advanced integrations, you can write pre-processed chunks directly to a document — replacing what the automatic pipeline produced. This is useful when your external system handles its own text splitting and embedding generation.

### Chunk data model

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | uuid | auto | Chunk identifier (auto-generated) |
| `document_id` | uuid | yes | Parent document ID |
| `agent_id` | uuid | yes | Agent this chunk belongs to |
| `content` | string | yes | The chunk text content |
| `embedding` | vector(1536) | no | OpenAI `text-embedding-3-small` compatible vector |
| `chunk_index` | integer | yes | Position within the document (0-based) |
| `token_count` | integer | no | Estimated token count of the chunk |
| `metadata` | jsonb | no | Custom metadata (default: `{}`) |
| `created_at` | datetime | auto | Creation timestamp |

<Note>
The `embedding` field expects a 1536-dimensional vector compatible with OpenAI's `text-embedding-3-small` model. Chunks without embeddings will not appear in vector search results.
</Note>

### List chunks

```
GET /v1/agents/{agentId}/knowledge-base/{documentId}/chunks
```

**Permission:** `kb:read` | **Pagination:** yes

Returns all chunks for a specific document.

### Create chunks (batch)

```
POST /v1/agents/{agentId}/knowledge-base/{documentId}/chunks
```

**Permission:** `kb:write`

Insert one or more pre-processed chunks. The parent document must exist and belong to the agent.

```bash
curl -X POST -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{
    "chunks": [
      {
        "content": "Produkt A kostet 29,99 EUR pro Monat...",
        "embedding": [0.012, -0.034, 0.056, ...],
        "chunk_index": 0,
        "token_count": 150,
        "metadata": {"source": "pricing-system", "version": "2.1"}
      },
      {
        "content": "Produkt B kostet 49,99 EUR pro Monat...",
        "embedding": [0.023, -0.045, 0.067, ...],
        "chunk_index": 1,
        "token_count": 120,
        "metadata": {"source": "pricing-system", "version": "2.1"}
      }
    ]
  }' \
  "$TP_BASE/agents/{agentId}/knowledge-base/{documentId}/chunks"
```

### Delete all chunks for a document

```
DELETE /v1/agents/{agentId}/knowledge-base/{documentId}/chunks
```

**Permission:** `kb:write` | Returns `204 No Content`

Removes all chunks for the given document. Useful before re-uploading updated chunks.

### Typical external integration flow

```
1. Create a document with a short placeholder content (POST /knowledge-base) → document_id
2. Poll GET /knowledge-base/{id} until status is "ready" (the pipeline has processed the placeholder)
3. Generate chunks + embeddings in your system
4. Delete the pipeline's chunks (DELETE /knowledge-base/{id}/chunks)
5. Upload your chunks (POST /knowledge-base/{id}/chunks) → searchable immediately
```

Do not `PATCH` the document's `content` afterwards — that re-triggers the pipeline, which rebuilds the chunks from `content`. The document status is managed by the pipeline and cannot be set via the API.

### Access control

Chunk operations are scoped to your organization. You can only write chunks for agents that belong to an organization you are a member of.

### Retrieval

During a call the agent searches the chunks of every document it has been granted — the documents it created through the API plus everything ticked for it in Company Knowledge. Chunks of documents the agent was not granted are never returned.

## Relationship to the KB tool

For the agent to actually use the knowledge base during calls, a tool of type `knowledge_base` must be configured and enabled on the agent. See [Tools — knowledge_base](/api/tools#knowledge_base).

The Dashboard automatically activates the KB tool when a document granted to the agent becomes ready and deactivates it when the agent has no ready document left.

## Related resources

- [Agents](/api/agents) — Parent resource
- [Tools](/api/tools) — The `knowledge_base` tool type
- [Knowledge Base](/product/knowledge-base) — Dashboard UI guide
