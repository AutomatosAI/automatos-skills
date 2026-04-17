---
name: twitter-automation
description: Create posts, search tweets, manage bookmarks, and handle replies on Twitter/X
version: "1.0.0"
tags: [twitter, x, social-media, posts, content]
category: social-media
tools:
  - name: composio_execute
    description: Execute Composio actions
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# Twitter/X Automation

## Prerequisites

- Active Composio connection: `TWITTER` app linked to user's X account
- OAuth 2.0 with read/write permissions

## Core Workflows

### 1. Create a Post

```
composio_execute action="TWITTER_CREATION_OF_A_POST"
  params:
    text: "Launching our new feature today! Check it out."
```

### 2. Reply to a Tweet

```
composio_execute action="TWITTER_CREATION_OF_A_POST"
  params:
    text: "@user Great point! Here's our take..."
    reply_in_reply_to_tweet_id: "<tweet_id>"
```

### 3. Search Tweets

```
composio_execute action="TWITTER_SEARCH_TWEETS"
  params:
    query: "automatos AI"
    max_results: 10
```

Search supports operators: `from:user`, `has:links`, `is:reply`, `-is:retweet`, `lang:en`.

### 4. Bookmark a Post

```
composio_execute action="TWITTER_BOOKMARK_A_POST"
  params:
    tweet_id: "<tweet_id>"
```

### 5. Remove a Bookmark

```
composio_execute action="TWITTER_REMOVE_A_BOOKMARK"
  params:
    tweet_id: "<tweet_id>"
```

### 6. Delete a Post

```
composio_execute action="TWITTER_DELETE_A_POST_BY_ID"
  params:
    tweet_id: "<tweet_id>"
```

## Key Parameters

| Parameter | Format | Notes |
|-----------|--------|-------|
| `text` | String (max 280 chars) | Required for post creation |
| `tweet_id` | String | Numeric tweet ID |
| `query` | String | Supports search operators |
| `max_results` | Integer (10-100) | Default varies by endpoint |
| `reply_in_reply_to_tweet_id` | String | Parent tweet ID for replies |

## Recipe Pattern: Content Pipeline

```
1. scratchpad_read key="draft_content"       # Get content from upstream step
2. composio_execute action="TWITTER_CREATION_OF_A_POST" text=<content>
3. scratchpad_write key="tweet_id" value=<response.id>  # Pass to downstream
```

## Pitfalls

- **280 character limit:** Posts exceeding 280 chars are rejected. Validate length before posting.
- **Rate limits:** Tweet creation: 200 per 15-minute window. Search: 300 per 15-minute window. Hitting limits returns 429 errors.
- **Media uploads:** Media is a separate workflow — you cannot attach images inline with `TWITTER_CREATION_OF_A_POST`. Upload media first, then reference the `media_id`.
- **Poll creation:** Requires `poll_options` and `poll_duration_minutes` params — both must be present together.
- **Duplicate detection:** Twitter rejects identical tweets posted within a short window. Vary text or wait.

## Quick Reference

| Task | Action |
|------|--------|
| Post a tweet | `TWITTER_CREATION_OF_A_POST` |
| Reply to tweet | `TWITTER_CREATION_OF_A_POST` + `reply_in_reply_to_tweet_id` |
| Search tweets | `TWITTER_SEARCH_TWEETS` |
| Bookmark tweet | `TWITTER_BOOKMARK_A_POST` |
| Remove bookmark | `TWITTER_REMOVE_A_BOOKMARK` |
| Delete tweet | `TWITTER_DELETE_A_POST_BY_ID` |

## What NOT to Do

- Do NOT post text longer than 280 characters — it will be rejected
- Do NOT try to attach media directly in the post action — upload media separately first
- Do NOT spam identical tweets — Twitter's duplicate detection will block them
- Do NOT ignore rate limit responses (429) — back off and retry after the window resets
- Do NOT assume tweet IDs are integers in code — always treat as strings
