---
title: "Knowledge Base"
description: "Manage company knowledge and grant it to agents"
---

The knowledge base gives your agents access to information they can reference during calls — product details, FAQs, pricing, opening hours, and more.

## How it works

1. You upload documents **once for the whole organization** under **Unternehmenswissen** (Company Knowledge)
2. TalkPilot automatically splits them into chunks and creates vector embeddings
3. On an agent's detail page you tick which documents that agent may use
4. During a call, when the agent needs information, it searches only the documents it was granted
5. The most relevant chunks are included in the agent's context

This is called **RAG** (Retrieval-Augmented Generation) — the agent retrieves relevant information and uses it to generate accurate answers.

<Note>
Documents belong to the **organization**, not to a single agent. The same PDF can serve any number of agents without being uploaded, chunked or maintained more than once. Editing it once updates it everywhere.
</Note>

## Adding documents

Documents are added on the **Unternehmenswissen** page (sidebar). They are stored for the organization you are currently working in.

### Manual text

1. Go to **Unternehmenswissen** (Company Knowledge)
2. Click **Dokument hinzufuegen** (Add Document)
3. Enter a title and paste your text content
4. Click **Speichern** (Save)

The document is queued for processing and becomes available within seconds.

### File upload

Upload files by dragging them into the upload area or clicking to browse.

| Format | Extension |
|--------|-----------|
| PDF | `.pdf` |
| Word | `.docx` |
| Excel | `.xlsx` |
| PowerPoint | `.pptx` |

Files are automatically parsed, text-extracted, and indexed.

## Granting documents to an agent

1. Open the agent's detail page > **Wissensdatenbank** (Knowledge Base)
2. You see every document of your organization with a checkbox
3. Tick a document — the agent can now search it; untick it and the agent loses access

Ticking and unticking only changes that one agent. Removing the tick does **not** delete the document; it stays in Company Knowledge and keeps working for every other agent that has it ticked.

## Document status

After uploading, documents go through a processing pipeline:

| Status | Meaning |
|--------|---------|
| **Pending** | Document is queued for processing |
| **Processing** | Text is being extracted, chunked, and embedded |
| **Ready** | Document is available for agent search |
| **Failed** | Processing error — check the error message |

Status changes appear live — no page reload needed.

## Knowledge Base tool

For the agent to actually search the knowledge base during calls, a **Knowledge Base** tool must be configured:

- **Auto-activation:** The tool is automatically enabled when the first document granted to this agent becomes ready
- **Auto-deactivation:** The tool is automatically disabled when the agent has no ready document left — either because the tick was removed or because the document was deleted

You can configure the tool's behavior:

| Setting | Default | Description |
|---------|---------|-------------|
| **Top K** | 5 | Number of chunks to retrieve |
| **Similarity threshold** | 0.7 | Minimum relevance (0 = any, 1 = exact match) |
| **Bridging sentence** | — | What the agent says while searching (e.g., "Einen Moment...") |

## Managing documents

- **Edit:** Click on a manual text document in Company Knowledge to update its content. Editing triggers automatic re-processing — every agent that has it ticked gets the new version.
- **Delete:** Remove a document and all its chunks/embeddings. It disappears from every agent that had it ticked.

**Via API:** You can create and manage text documents programmatically. File uploads are Dashboard-only. See [Knowledge Base API](/api/knowledge-base).

## Tips

- Keep documents focused on one topic each — this improves search accuracy
- Use clear, factual language — avoid ambiguous phrasing
- Update documents when information changes (pricing, hours, etc.) — once, in Company Knowledge
- Check document status after upload to confirm successful processing
- Only tick what an agent really needs — fewer, more relevant documents produce better answers
