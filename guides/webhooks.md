---
title: "Webhooks"
description: "Receive real-time notifications for calls and status changes"
---

TalkPilot uses webhooks to notify your systems about events in real-time. There are two webhook types: **post-call webhooks** and **employee status webhooks**.

## Post-call webhooks

Configured per agent via the `post_call_webhook_url` field. After every completed call, TalkPilot sends a POST request to this URL with call data.

### Configuration

Set the webhook URL on the agent:

```bash
curl -X PATCH -H "X-API-Key: $TP_KEY" -H "Content-Type: application/json" \
  -d '{"post_call_webhook_url": "https://your-app.com/webhooks/talkpilot"}' \
  "$TP_BASE/agents/{agentId}"
```

Or configure it in the Dashboard under **Agenten > [Agent] > Webhook**.

### Payload

The webhook sends a JSON payload with the call details:

```json
{
  "event": "call.completed",
  "call": {
    "id": "call-uuid",
    "call_id": "lk_call_abc123",
    "customer_name": "Hans Meier",
    "customer_request": "Frage zu Rechnung #1234",
    "phone_number": "+491701234567",
    "duration": 180,
    "done": false,
    "general_success": true,
    "disconnection_reason": "user_hangup",
    "call_summary": "Kunde fragte nach Rechnung #1234...",
    "transcript": "[{\"role\":\"agent\",\"text\":\"...\"}]",
    "employee": null,
    "agent_id": "agent-uuid",
    "created_at": "2026-03-22T10:30:00Z"
  }
}
```

### Handling webhooks

```javascript
// Express.js example
app.post("/webhooks/talkpilot", (req, res) => {
  const { event, call } = req.body;

  if (event === "call.completed") {
    // Create a ticket in your CRM
    await createTicket({
      customerName: call.customer_name,
      subject: call.customer_request,
      summary: call.call_summary,
      phone: call.phone_number,
    });
  }

  // Respond quickly — process asynchronously
  res.status(200).json({ received: true });
});
```

```python
# Flask example
@app.route("/webhooks/talkpilot", methods=["POST"])
def handle_webhook():
    data = request.json
    event = data.get("event")

    if event == "call.completed":
        call = data["call"]
        # Queue for async processing
        task_queue.enqueue(process_call, call)

    return {"received": True}, 200
```

## Employee status webhooks

Configured in the user's profile settings (Dashboard > Settings > Webhooks). When an employee's forwarding rule or status changes in the Dashboard, a webhook notification is sent.

### Payload

```json
{
  "event": "employee.forwarding_created",
  "employee": {
    "name": "Max Mustermann",
    "phone_number": "+491701234567",
    "email": "max@example.com"
  },
  "agents": ["Rezeption Bot", "Support Bot"],
  "substitute": "Erika Musterfrau",
  "responsibilities": "Rechnungsfragen, Zahlungsprobleme",
  "start_date": "2026-03-25",
  "reasons": ["Urlaub"]
}
```

## Best practices

1. **Respond with 200 quickly** — Process webhook data asynchronously. TalkPilot expects a response within a few seconds.

2. **Handle duplicates** — In rare cases, a webhook may be sent more than once. Use the `call.id` to deduplicate.

3. **Validate the source** — Verify that webhook requests come from TalkPilot by checking the source IP or implementing a shared secret in the URL (e.g., `https://your-app.com/webhooks/talkpilot?secret=YOUR_SECRET`).

4. **Log request IDs** — Store the call ID for troubleshooting and correlation with the TalkPilot Dashboard.

5. **Handle failures gracefully** — If your endpoint is down, webhook deliveries are not retried. Use the [Calls API](/api/calls) to catch up on missed events.

## Related resources

- [Agents API](/api/agents) — Configure `post_call_webhook_url`
- [CRM Integration](/use-cases/crm-integration) — End-to-end webhook integration guide
