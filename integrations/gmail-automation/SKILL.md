---
name: gmail-automation
description: Send, search, label, and draft emails via Gmail
version: "1.0.0"
tags: [gmail, email, communication]
category: productivity
tools:
  - name: composio_execute
    description: Execute Composio actions (Gmail API operations)
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# Gmail Automation Skill

## Prerequisites

Gmail must be connected in **Settings > Integrations**. The agent must have the `composio_execute` tool assigned.

## Core Workflows

### 1. Send Email

```
composio_execute action="GMAIL_SEND_EMAIL" params={
  "recipient_email": "user@example.com",
  "subject": "Subject line",
  "message_body": "<p>HTML body</p>",
  "cc": "cc@example.com",          // optional
  "bcc": "bcc@example.com"         // optional
}
```

**Pitfalls:**
- `message_body` is HTML — use `<p>`, `<br>`, `<a href>` tags. Plain text renders without line breaks.
- No attachment support via this action. Use `GMAIL_CREATE_EMAIL_DRAFT` + manual send for attachments.

### 2. Reply to Email

Two-step: find the message, then reply.

```
composio_execute action="GMAIL_FETCH_MESSAGE_BY_MESSAGE_ID" params={
  "message_id": "<message-id>"
}

composio_execute action="GMAIL_REPLY_TO_THREAD" params={
  "thread_id": "<thread_id from fetch>",
  "message_body": "<p>Reply body</p>",
  "recipient_email": "original-sender@example.com"
}
```

**Pitfalls:**
- `message_id` is the Gmail API ID (numeric string), NOT the RFC `Message-ID` header.
- You must extract `thread_id` from the fetched message — replies without it create a new thread.
- `recipient_email` is required even for replies.

### 3. Search Email

```
composio_execute action="GMAIL_FETCH_EMAILS" params={
  "query": "from:boss@company.com after:2026/03/01 has:attachment",
  "max_results": 10
}
```

**Query syntax reference (Gmail search operators):**
| Operator | Example | Notes |
|----------|---------|-------|
| `from:` | `from:alice@co.com` | Sender |
| `to:` | `to:team@co.com` | Recipient |
| `subject:` | `subject:invoice` | Subject line |
| `after:` / `before:` | `after:2026/03/01` | Date format: YYYY/MM/DD |
| `has:attachment` | — | Has attachments |
| `is:unread` | — | Unread only |
| `label:` | `label:important` | By label |
| `in:` | `in:sent` | Mailbox folder |
| `newer_than:` | `newer_than:2d` | Relative date (d/m/y) |

**Pitfalls:**
- Default `max_results` is low. Set explicitly for bulk searches.
- Returns metadata only — use `GMAIL_FETCH_MESSAGE_BY_MESSAGE_ID` for full body.
- Date format is `YYYY/MM/DD` with slashes, not dashes.

### 4. Manage Labels

```
// List existing labels
composio_execute action="GMAIL_LIST_LABELS" params={}

// Create new label
composio_execute action="GMAIL_CREATE_LABEL" params={
  "label_name": "Project/Alpha"
}
```

**Pitfall:** Nested labels use `/` separator — `"Project/Alpha"` creates `Alpha` under `Project`.

### 5. Apply Labels to Messages

```
composio_execute action="GMAIL_ADD_LABEL_TO_EMAIL" params={
  "message_id": "<message-id>",
  "label_id": "<label-id>"
}
```

**Pitfall:** Requires the label **ID** (not name). Get IDs from `GMAIL_LIST_LABELS` first.

### 6. Create Draft

```
composio_execute action="GMAIL_CREATE_EMAIL_DRAFT" params={
  "recipient_email": "user@example.com",
  "subject": "Draft subject",
  "message_body": "<p>Draft body</p>"
}
```

## Quick Reference

| Action | Key Params |
|--------|-----------|
| `GMAIL_SEND_EMAIL` | recipient_email, subject, message_body |
| `GMAIL_REPLY_TO_THREAD` | thread_id, recipient_email, message_body |
| `GMAIL_FETCH_EMAILS` | query, max_results |
| `GMAIL_FETCH_MESSAGE_BY_MESSAGE_ID` | message_id |
| `GMAIL_LIST_LABELS` | — |
| `GMAIL_CREATE_LABEL` | label_name |
| `GMAIL_ADD_LABEL_TO_EMAIL` | message_id, label_id |
| `GMAIL_REMOVE_LABEL_FROM_EMAIL` | message_id, label_id |
| `GMAIL_CREATE_EMAIL_DRAFT` | recipient_email, subject, message_body |

## Recipe Integration

- **Read inputs:** `scratchpad_read` to get recipients, content from upstream steps.
- **Export results:** `scratchpad_write` message IDs, thread IDs, or search results for downstream steps.
