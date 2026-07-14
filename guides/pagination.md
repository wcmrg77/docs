---
title: "Pagination"
description: "Navigate large result sets with page-based pagination"
---

All list endpoints support page-based pagination. This keeps responses fast and predictable, even when you have thousands of records.

## Query parameters

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| `page` | integer | `1` | — | Page number (1-indexed) |
| `limit` | integer | `20` | `100` | Items per page |

## Response format

Every paginated response includes a `pagination` object:

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 157,
    "pages": 8
  }
}
```

| Field | Description |
|-------|-------------|
| `page` | Current page number |
| `limit` | Items per page |
| `total` | Total number of items across all pages |
| `pages` | Total number of pages |

## Examples

### First page (default)

```bash
curl -H "X-API-Key: tp_live_YOUR_KEY" \
  "https://api.talkpilot.io/v1/agents/{agentId}/calls"
```

### Specific page with custom limit

```bash
curl -H "X-API-Key: tp_live_YOUR_KEY" \
  "https://api.talkpilot.io/v1/agents/{agentId}/calls?page=3&limit=50"
```

### Fetch all pages

To retrieve all records, iterate through pages until `page >= pages`:

```javascript
async function fetchAllCalls(agentId) {
  const allCalls = [];
  let page = 1;
  let totalPages = 1;

  while (page <= totalPages) {
    const response = await fetch(
      `${BASE_URL}/v1/agents/${agentId}/calls?page=${page}&limit=100`,
      { headers: { "X-API-Key": API_KEY } }
    );
    const json = await response.json();

    allCalls.push(...json.data);
    totalPages = json.pagination.pages;
    page++;
  }

  return allCalls;
}
```

## Paginated endpoints

The following endpoints support pagination:

| Endpoint | Default order |
|----------|--------------|
| `GET /v1/agents` | Created date (newest first) |
| `GET /v1/agents/{agentId}/employees` | Name (alphabetical) |
| `GET /v1/agents/{agentId}/knowledge-base` | Created date (newest first) |
| `GET /v1/agents/{agentId}/calls` | Created date (newest first) |

**Note:** Forwarding slots (`GET /v1/agents/{agentId}/forwarding-slots`) and tools (`GET /v1/agents/{agentId}/tools`) return all records without pagination, as agents typically have a small number of these.

## Tips

- Use `limit=100` (the maximum) when you need to fetch large datasets to minimize the number of requests
- Cache `pagination.total` to show progress indicators in your UI
- Be aware that records may shift between pages if items are added or deleted during iteration
