---
name: slack-automation
description: Send messages, search, manage threads and channels in Slack
version: "1.0.0"
tags: [slack, messaging, communication]
category: productivity
tools:
  - name: composio_execute
    description: Execute Composio actions (Slack API operations)
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# Slack Automation Skill

## Prerequisites

Slack must be connected in **Settings > Integrations**. The agent must have the `composio_execute` tool assigned.

## Core Workflows

### 1. Send Message

```
composio_execute action="SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL" params={
  "channel": "C0123ABCDEF",
  "text": "Hello team!"
}
```

**Pitfalls:**
- `channel` requires the **channel ID**, not the name. Resolve names first with `SLACK_LIST_CHANNELS`.
- For DMs, use `SLACK_SENDS_A_DIRECT_MESSAGE_TO_A_SLACK_USER` with `user` param instead.
- Text supports Slack mrkdwn: `*bold*`, `_italic_`, `` `code` ``, `>quote`, `<url|link text>`.
- Mention users with `<@U0123ABCDEF>` (user ID), not `@username`.

### 2. Send Direct Message

```
composio_execute action="SLACK_SENDS_A_DIRECT_MESSAGE_TO_A_SLACK_USER" params={
  "user": "U0123ABCDEF",
  "text": "Hey, quick update on the project."
}
```

**Pitfall:** Requires **user ID**. Resolve from `SLACK_LIST_USERS` or `SLACK_SEARCH_USERS`.

### 3. Search Messages

```
composio_execute action="SLACK_SEARCH_MESSAGES" params={
  "query": "from:@alice in:#engineering deployment",
  "count": 20
}
```

**Search modifiers:**
| Modifier | Example | Notes |
|----------|---------|-------|
| `from:` | `from:@alice` | By sender |
| `in:` | `in:#general` | By channel (use #name) |
| `has:` | `has:link`, `has:reaction` | Content type |
| `before:` / `after:` | `after:2026-03-01` | Date (YYYY-MM-DD) |
| `on:` | `on:2026-03-07` | Exact date |
| `during:` | `during:march` | Month/year |

### 4. Thread Replies

```
composio_execute action="SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL" params={
  "channel": "C0123ABCDEF",
  "text": "Replying in thread",
  "thread_ts": "1709856000.000100"
}
```

**Pitfalls:**
- `thread_ts` is the timestamp of the parent message — get it from message responses or search results.
- Same action as sending a message, just with `thread_ts` added.
- To also post to channel, add `"reply_broadcast": true`.

### 5. Channel Management

```
// List channels
composio_execute action="SLACK_LIST_CHANNELS" params={
  "types": "public_channel",
  "limit": 100
}

// Get channel info
composio_execute action="SLACK_GET_CHANNEL_INFO" params={
  "channel": "C0123ABCDEF"
}

// Create channel
composio_execute action="SLACK_CREATE_CHANNEL" params={
  "name": "project-alpha",
  "is_private": false
}
```

**Pitfalls:**
- Channel names: lowercase, no spaces, max 80 chars. Use hyphens.
- `SLACK_LIST_CHANNELS` paginates — check `response_metadata.next_cursor` for more results.

### 6. Reactions

```
composio_execute action="SLACK_ADD_REACTION" params={
  "channel": "C0123ABCDEF",
  "timestamp": "1709856000.000100",
  "name": "white_check_mark"
}
```

**Pitfall:** Emoji `name` without colons — use `thumbsup` not `:thumbsup:`.

### 7. Schedule Message

```
composio_execute action="SLACK_SCHEDULE_MESSAGE" params={
  "channel": "C0123ABCDEF",
  "text": "Reminder: standup in 15 minutes",
  "post_at": 1709942400
}
```

**Pitfall:** `post_at` is a Unix timestamp (seconds), not a date string. Must be in the future.

## ID Resolution Pattern

Most Slack actions need IDs, not names. Common resolution flow:

1. **Channel name to ID:** `SLACK_LIST_CHANNELS` → find by `name` → use `id`
2. **User name to ID:** `SLACK_LIST_USERS` or `SLACK_SEARCH_USERS` → use `id`
3. **Cache IDs** via `scratchpad_write` to avoid repeated lookups in recipes.

## Quick Reference

| Action | Key Params |
|--------|-----------|
| `SLACK_SENDS_A_MESSAGE_TO_A_SLACK_CHANNEL` | channel, text, thread_ts? |
| `SLACK_SENDS_A_DIRECT_MESSAGE_TO_A_SLACK_USER` | user, text |
| `SLACK_SEARCH_MESSAGES` | query, count? |
| `SLACK_LIST_CHANNELS` | types?, limit? |
| `SLACK_GET_CHANNEL_INFO` | channel |
| `SLACK_CREATE_CHANNEL` | name, is_private? |
| `SLACK_ADD_REACTION` | channel, timestamp, name |
| `SLACK_SCHEDULE_MESSAGE` | channel, text, post_at |
| `SLACK_LIST_USERS` | limit? |
| `SLACK_SET_CHANNEL_TOPIC` | channel, topic |

## Recipe Integration

- **Read inputs:** `scratchpad_read` to get channel IDs, message content from upstream steps.
- **Export results:** `scratchpad_write` message timestamps, channel IDs for downstream steps.
