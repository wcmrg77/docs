---
title: "Call Management"
description: "View, filter, and manage call records"
---

The **Anrufe** (Calls) page is where you review, manage, and process call records.

## Call list

All calls are displayed in a filterable, sortable table (desktop) or card view (mobile).

### Filtering

| Filter | Description |
|--------|-------------|
| **Search** | Search by customer name, phone number, or call summary |
| **Status** | All / Success / Failed |
| **Done** | All / Done / Not done |
| **Employee** | Filter by assigned employee (or "unassigned") |
| **Agent** | Filter by agent |
| **Date range** | Custom start and end date |

### Column customization

Customize which columns are visible and in what order:

1. Click the **column settings** icon in the toolbar
2. Toggle columns on/off
3. Drag columns to reorder them
4. Click **Reset** to restore defaults

Your column preferences are saved locally and persist between sessions.

## Call actions

### Mark as done

Click the checkbox or toggle to mark a call as processed. If your organization has **auto-delete** enabled, done calls are automatically moved to the trash.

### Assign employee

Use the dropdown in the employee column to assign a call to a specific employee. Select "unassigned" to remove the assignment.

### Add notes

Click the note icon to open the note dialog. Notes are free text — use them for follow-up reminders, action items, or additional context.

### Report errors

Click the error icon to flag a call with an issue. Select the error type and add a description.

### Forward by email

Click **Forward** in the call detail sheet to send this single call by email to the employee
assigned to it. The email uses the same layout as the automatic post-call notification:
caller name, number, date, callback number, subject, summary, an audio link and the transcript.

Rules:

- The action only appears when **Forward calls by email** is enabled for the organization
  (Settings → Organization).
- The recipient is always exactly the assigned employee — never a distribution list. Without an
  assignment, or when the assigned employee has no email address on file, the button stays
  disabled and says why.
- Read-only roles (`dev_employee`) cannot forward.
- The audio link is freshly signed and expires after one hour; the recording is never attached.

## Call detail sheet

Click on any call to open the detail side panel:

| Section | Content |
|---------|---------|
| **Header** | Customer name, status, date/time |
| **Customer info** | Name (editable), request/subject (editable) |
| **Call status** | Success/failed dropdown, disconnection reason |
| **Employee** | Assignment dropdown |
| **Duration** | Call length |
| **Summary** | AI-generated call summary |
| **Transcript** | Full conversation transcript (expandable) |
| **Audio** | Call recording player |
| **Notes** | Add or edit notes |

Navigate between calls using the up/down arrows in the detail sheet.

## CSV export

Export your filtered calls as a CSV file:

1. Apply your desired filters
2. Click the **CSV Export** button in the toolbar
3. The file downloads with all visible columns

Filename format: `anrufe_YYYY-MM-DD_HH-mm.csv`

## Bulk operations

### Delete multiple calls

1. Click the **delete mode** button in the toolbar
2. Select calls using the checkboxes
3. Click **Delete selected** to move them to the trash
4. Confirm the action

## Trash (Papierkorb)

Deleted calls are soft-deleted — they're moved to the trash and can be recovered.

| Action | Where |
|--------|-------|
| **View trash** | Settings > Papierkorb |
| **Restore** | Click restore icon on a trashed call |
| **Permanently delete** | Click delete icon on a trashed call |
| **Empty trash** | Delete all trashed calls at once |

### Auto-delete

Enable in Settings > Organization: when a call is marked as done, it's automatically moved to the trash.

## Via API

Read call records programmatically — see [Calls API](/api/calls). The API is read-only; call management actions (done, assign, notes) are Dashboard-only.
