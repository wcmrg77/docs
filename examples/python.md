---
title: "Python Examples"
description: "Python code examples with the requests library"
---

Using the `requests` library for the TalkPilot API.

## Setup

```bash
pip install requests
```

```python
import requests
import os
import time

BASE_URL = os.environ.get("TALKPILOT_BASE_URL", "https://api.talkpilot.io/v1")
API_KEY = os.environ.get("TALKPILOT_API_KEY", "tp_live_YOUR_KEY_HERE")

HEADERS = {
    "X-API-Key": API_KEY,
    "Content-Type": "application/json",
}


def talkpilot(path, method="GET", data=None):
    """Make a request to the TalkPilot API with automatic retry on rate limiting."""
    url = f"{BASE_URL}{path}"
    response = requests.request(method, url, headers=HEADERS, json=data)

    # Auto-retry on rate limiting
    if response.status_code == 429:
        retry_after = int(response.headers.get("Retry-After", 60))
        print(f"Rate limited. Retrying in {retry_after}s...")
        time.sleep(retry_after)
        return talkpilot(path, method, data)

    if response.status_code == 204:
        return None

    body = response.json()

    if not response.ok:
        error = body.get("error", {})
        raise Exception(
            f"[{error.get('code')}] {error.get('message')} "
            f"(request_id: {error.get('request_id')})"
        )

    return body
```

## List agents

```python
result = talkpilot("/agents")

print(f"Found {result['pagination']['total']} agents:")
for agent in result["data"]:
    status = "active" if agent["is_active"] else "inactive"
    print(f"  - {agent['name']} ({status})")
```

## Get agent details

```python
agent = talkpilot(f"/agents/{agent_id}")

print(f"Agent: {agent['name']}")
print(f"Model: {agent['llm_provider']}/{agent['llm_model']}")
print(f"Voice: {agent['voice_id']}")
print(f"Active: {agent['is_active']}")
```

## Update agent settings

```python
# Update prompt and greeting
updated = talkpilot(f"/agents/{agent_id}", method="PATCH", data={
    "prompt": "Du bist ein professioneller Kundenservice-Agent fuer Firma X.",
    "greeting": "Willkommen bei Firma X, wie kann ich Ihnen helfen?",
})

print(f"Agent updated: {updated['name']}")
```

## Toggle vacation mode

```python
# Enable vacation mode
talkpilot(f"/agents/{agent_id}", method="PATCH", data={
    "vacation_mode": True,
    "vacation_end": "2026-04-01T00:00:00Z",
    "vacation_notdienst": True,
})

# Disable vacation mode
talkpilot(f"/agents/{agent_id}", method="PATCH", data={
    "vacation_mode": False,
})
```

## Manage employees

```python
# List present employees
result = talkpilot(f"/agents/{agent_id}/employees?status=anwesend")
employees = result["data"]

# Create a new employee
new_employee = talkpilot(f"/agents/{agent_id}/employees", method="POST", data={
    "name": "Max Mustermann",
    "phone_number": "+491701234567",
    "email": "max@example.com",
    "status": "anwesend",
    "active": True,
    "get_mail": True,
})

print(f"Created employee: {new_employee['name']} (ID: {new_employee['id']})")

# Update employee status
talkpilot(f"/agents/{agent_id}/employees/{employee_id}", method="PATCH", data={
    "status": "urlaub",
})

# Delete an employee
talkpilot(f"/agents/{agent_id}/employees/{employee_id}", method="DELETE")
```

## Sync forwarding slots

```python
# Replace all forwarding slots in one request
result = talkpilot(f"/agents/{agent_id}/forwarding-slots", method="PUT", data={
    "slots": [
        {
            "slotnummer": "01",
            "cases": "Rechnungsfragen, Zahlungsprobleme",
            "slot_belegung": "Max Mustermann",
            "prioritaet_1": "+491701234567",
            "prioritaet_2": "+491709876543",
        },
        {
            "slotnummer": "02",
            "cases": "Technischer Support",
            "slot_belegung": "Erika Musterfrau",
            "prioritaet_1": "+491705555555",
        },
    ],
})

print(f"Replaced {result['replaced_count']} slots with {len(result['data'])} new slots")
```

## Fetch calls with filtering

```python
from datetime import datetime, timedelta

# Calls from the last 7 days
seven_days_ago = (datetime.utcnow() - timedelta(days=7)).isoformat() + "Z"

result = talkpilot(f"/agents/{agent_id}/calls?from={seven_days_ago}&limit=50")

print(f"{result['pagination']['total']} calls in the last 7 days")

for call in result["data"]:
    print(f"  [{call['created_at'][:10]}] {call.get('customer_name', 'Unknown')} "
          f"- {call.get('call_summary', 'No summary')[:60]}...")
```

## Fetch all pages

```python
def fetch_all(path):
    """Fetch all items across all pages."""
    all_items = []
    page = 1
    total_pages = 1
    separator = "&" if "?" in path else "?"

    while page <= total_pages:
        result = talkpilot(f"{path}{separator}page={page}&limit=100")
        all_items.extend(result["data"])
        total_pages = result["pagination"]["pages"]
        page += 1

    return all_items


# Usage
all_calls = fetch_all(f"/agents/{agent_id}/calls?from=2026-03-01T00:00:00Z")
print(f"Fetched {len(all_calls)} calls total")
```

## Add knowledge base content

```python
doc = talkpilot(f"/agents/{agent_id}/knowledge-base", method="POST", data={
    "title": "Preisliste 2026",
    "content": """Produkt A: 29,99 EUR/Monat
Produkt B: 49,99 EUR/Monat
Produkt C: 99,99 EUR/Monat
Alle Preise zzgl. MwSt.""",
})

print(f"Document created: {doc['title']} (status: {doc['status']})")
```

## Upload pre-processed chunks with embeddings

```python
import openai

openai_client = openai.OpenAI()


def get_embeddings(texts):
    """Generate embeddings using OpenAI text-embedding-3-small."""
    response = openai_client.embeddings.create(
        model="text-embedding-3-small",
        input=texts,
    )
    return [item.embedding for item in response.data]


def upload_chunks(agent_id, title, chunks_data):
    """
    Upload pre-processed chunks to a TalkPilot agent's knowledge base.

    chunks_data: list of dicts with 'content' and optional 'metadata'
    """
    # 1. Create parent document
    doc = talkpilot(f"/agents/{agent_id}/knowledge-base", method="POST", data={
        "title": title,
        "content": "",
    })
    doc_id = doc["id"]

    # 2. Generate embeddings
    texts = [c["content"] for c in chunks_data]
    embeddings = get_embeddings(texts)

    # 3. Build chunk payloads
    chunks = []
    for i, (chunk, embedding) in enumerate(zip(chunks_data, embeddings)):
        chunks.append({
            "content": chunk["content"],
            "embedding": embedding,
            "chunk_index": i,
            "token_count": len(chunk["content"].split()),
            "metadata": chunk.get("metadata", {}),
        })

    # 4. Upload chunks
    result = talkpilot(
        f"/agents/{agent_id}/knowledge-base/{doc_id}/chunks",
        method="POST",
        data={"chunks": chunks},
    )
    print(f"Uploaded {result['inserted_count']} chunks for '{title}'")

    # 5. Mark document as completed
    talkpilot(
        f"/agents/{agent_id}/knowledge-base/{doc_id}",
        method="PATCH",
        data={"status": "completed"},
    )

    return doc_id


# Usage
upload_chunks(agent_id, "Product Catalog", [
    {"content": "Produkt A kostet 29,99 EUR pro Monat...", "metadata": {"category": "pricing"}},
    {"content": "Produkt B kostet 49,99 EUR pro Monat...", "metadata": {"category": "pricing"}},
    {"content": "Produkt C kostet 99,99 EUR pro Monat...", "metadata": {"category": "pricing"}},
])
```

## Complete script: HR status sync

```python
#!/usr/bin/env python3
"""
Sync employee statuses from your HR system to TalkPilot.
Run as a cron job or scheduled task.
"""

import os
import requests
import time

BASE_URL = os.environ["TALKPILOT_BASE_URL"]
API_KEY = os.environ["TALKPILOT_API_KEY"]
AGENT_ID = os.environ["TALKPILOT_AGENT_ID"]
HEADERS = {"X-API-Key": API_KEY, "Content-Type": "application/json"}


def talkpilot(path, method="GET", data=None):
    url = f"{BASE_URL}{path}"
    response = requests.request(method, url, headers=HEADERS, json=data)
    if response.status_code == 429:
        time.sleep(int(response.headers.get("Retry-After", 60)))
        return talkpilot(path, method, data)
    if response.status_code == 204:
        return None
    body = response.json()
    if not response.ok:
        raise Exception(f"API error: {body['error']['message']}")
    return body


def get_hr_statuses():
    """Replace this with your actual HR system integration."""
    return [
        {"name": "Max Mustermann", "status": "anwesend"},
        {"name": "Erika Musterfrau", "status": "urlaub"},
    ]


def sync():
    hr_employees = get_hr_statuses()
    tp_result = talkpilot(f"/agents/{AGENT_ID}/employees")
    tp_employees = tp_result["data"]

    updated = 0
    for hr_emp in hr_employees:
        match = next((e for e in tp_employees if e["name"] == hr_emp["name"]), None)
        if match and match["status"] != hr_emp["status"]:
            talkpilot(
                f"/agents/{AGENT_ID}/employees/{match['id']}",
                method="PATCH",
                data={"status": hr_emp["status"]},
            )
            print(f"Updated {match['name']}: {match['status']} -> {hr_emp['status']}")
            updated += 1

    print(f"Sync complete. {updated} employee(s) updated.")


if __name__ == "__main__":
    sync()
```
