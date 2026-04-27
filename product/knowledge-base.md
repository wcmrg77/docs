---
title: "Knowledge Base"
description: "Upload documents and manage the knowledge base"
---

The knowledge base gives your agent access to information it can reference during calls — product details, FAQs, pricing, opening hours, and more.

## How it works

1. You upload documents (text or files)
2. TalkPilot automatically splits them into chunks and creates vector embeddings
3. During a call, when the agent needs information, it searches the knowledge base
4. The most relevant chunks are included in the agent's context

This is called **RAG** (Retrieval-Augmented Generation) — the agent retrieves relevant information and uses it to generate accurate answers.

## Adding documents

### Manual text

1. Go to agent detail > **Wissensdatenbank** (Knowledge Base)
2. Click **Dokument hinzufuegen** (Add Document)
3. Enter a title and paste your text content
4. Click **Speichern** (Save)

The document is immediately processed and available within seconds.

### File upload

Upload files by dragging them into the upload area or clicking to browse.

| Format | Extension |
|--------|-----------|
| PDF | `.pdf` |
| Word | `.docx` |
| Excel | `.xlsx` |
| PowerPoint | `.pptx` |

Files are automatically parsed, text-extracted, and indexed.

## Document status

After uploading, documents go through a processing pipeline:

| Status | Meaning |
|--------|---------|
| **Uploading** | File is being uploaded to storage |
| **Processing** | Text is being extracted, chunked, and embedded |
| **Ready** | Document is available for agent search |
| **Failed** | Processing error — check the error message |

## Knowledge Base tool

For the agent to actually search the knowledge base during calls, a **Knowledge Base** tool must be configured:

- **Auto-activation:** The tool is automatically enabled when the first document becomes ready
- **Auto-deactivation:** The tool is automatically disabled when the last document is deleted

You can configure the tool's behavior:

| Setting | Default | Description |
|---------|---------|-------------|
| **Top K** | 5 | Number of chunks to retrieve |
| **Similarity threshold** | 0.7 | Minimum relevance (0 = any, 1 = exact match) |
| **Bridging sentence** | — | What the agent says while searching (e.g., "Einen Moment...") |

## Managing documents

- **Edit:** Click on a manual text document to update its content. Editing triggers automatic re-processing.
- **Delete:** Remove a document and all its chunks/embeddings.

**Via API:** You can create and manage text documents programmatically. File uploads are Dashboard-only. See [Knowledge Base API](/api/knowledge-base).

## External integration (Direct Chunk API)

If you manage your own chunking and embedding pipeline (e.g., via n8n, LangChain, or a custom ETL), you can write pre-processed chunks directly to an agent's knowledge base — bypassing TalkPilot's automatic processing.

This is useful for:
- **CRM / ERP sync** — keep product data, pricing, or customer info always up-to-date
- **Custom chunking** — control chunk size and overlap for your specific use case
- **External embedding models** — use your own embeddings (must be `text-embedding-3-small` compatible, 1536 dimensions)

Typical flow:

```
1. Create a document via API          → get document_id
2. Generate chunks + embeddings       → in your system
3. Upload chunks via Direct Chunk API → stored in KB
4. Set document status to "completed" → agent can search
```

For full endpoint documentation, request/response examples, and the chunk data model, see the [Direct Chunk API reference](/api/knowledge-base#direct-chunk-api).

## Tips

- Keep documents focused on one topic each — this improves search accuracy
- Use clear, factual language — avoid ambiguous phrasing
- Update documents when information changes (pricing, hours, etc.)
- Check document status after upload to confirm successful processing
