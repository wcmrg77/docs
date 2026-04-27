---
title: "Knowledge Base"
description: "Add, update, and remove knowledge base documents"
---

The knowledge base is a RAG (Retrieval-Augmented Generation) document store for an agent. Upload documents with information the agent should reference during calls — product details, FAQs, pricing, opening hours, etc.

## Document lifecycle

```
Create → pending → processing → completed (ready for search)
                               → error (check error_message)
```

When a document is created, it's automatically chunked and embedded for vector search. This process runs asynchronously — poll the document status to check when it's ready.

## Data model

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Document identifier |
| `agent_id` | uuid | Parent agent |
| `title` | string | Document title |
| `content` | string | Text content (null for file-based documents) |
| `source_type` | string | `manual`, `pdf`, `csv`, `txt`, `docx` |
| `source_url` | string | Storage URL (file uploads only) |
| `status` | string | `pending`, `processing`, `completed`, `error` |
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
| `status` | string | Filter by processing status (`pending`, `processing`, `completed`, `error`) |

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

Creates a text-based document. The content is automatically chunked and embedded.

**Note:** File uploads (PDF, DOCX, XLSX, PPTX) are only available through the Dashboard UI.

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

If the `content` field changes, the document is automatically re-chunked and re-embedded.

### Delete document

```
DELETE /v1/agents/{agentId}/knowledge-base/{documentId}
```

**Permission:** `kb:write` | Returns `204 No Content`

Deletes the document and all associated chunks and embeddings.

## Direct Chunk API

For advanced integrations, you can write pre-processed chunks directly to an agent's knowledge base — bypassing the automatic chunking pipeline. This is useful when your external system handles its own text splitting and embedding generation.

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
1. Create a document (POST /knowledge-base)         → get document_id
2. Generate chunks + embeddings in your system
3. Upload chunks (POST /knowledge-base/{id}/chunks)  → chunks stored
4. Update document status to "completed" (PATCH)     → agent can search
```

### Access control

Chunk operations are scoped to your organization. You can only write chunks for agents that belong to an organization you are a member of.

## Relationship to the KB tool

For the agent to actually use the knowledge base during calls, a tool of type `knowledge_base` must be configured and enabled on the agent. See [Tools — knowledge_base](/tools.md#knowledge_base).

The Dashboard automatically activates the KB tool when documents become ready and deactivates it when no ready documents remain.

## Related resources

- [Agents](/agents) — Parent resource
- [Tools](/tools) — The `knowledge_base` tool type
- [Knowledge Base](/product/knowledge-base) — Dashboard UI guide
