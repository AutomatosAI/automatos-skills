---
name: linkedin-automation
description: Create posts, share articles, publish image posts, and comment on LinkedIn
version: "1.0.0"
tags: [linkedin, social-media, professional, content, networking]
category: social-media
tools:
  - name: composio_execute
    description: Execute Composio actions
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# LinkedIn Automation

Designed for the HARPER content agent and general LinkedIn content workflows.

## Prerequisites

- Active Composio connection: `LINKEDIN` app linked to user's LinkedIn account
- Use **v2 actions only** — v1 actions are deprecated and may fail silently

## Core Workflows

### 1. Create a Text Post

```
composio_execute action="LINKEDIN_CREATE_A_POST_V2"
  params:
    text: "We just shipped our unified skills architecture.\n\nHere's what we learned building it:\n\n1. Keep interfaces simple\n2. Let agents compose skills\n3. Test with real workflows, not mocks\n\nWhat patterns have you found useful?\n\n#AI #Engineering #Automation"
    visibility: "PUBLIC"
```

### 2. Share an Article

```
composio_execute action="LINKEDIN_CREATE_A_POST_V2"
  params:
    text: "Great read on the future of AI agents in enterprise workflows."
    visibility: "PUBLIC"
    article_url: "https://example.com/article"
    article_title: "AI Agents in 2026"
    article_description: "A deep dive into agent architectures"
```

### 3. Create an Image Post

```
composio_execute action="LINKEDIN_CREATE_AN_IMAGE_POST_V2"
  params:
    text: "Our architecture diagram for the new skills system."
    visibility: "PUBLIC"
    image_url: "https://example.com/diagram.png"
    image_alt_text: "Skills architecture diagram showing agent-tool relationships"
```

### 4. Comment on a Post

```
composio_execute action="LINKEDIN_CREATE_A_COMMENT_ON_A_POST"
  params:
    post_urn: "urn:li:share:7000000000000000000"
    text: "This resonates. We've seen similar patterns in our platform."
```

### 5. Get User Profile

```
composio_execute action="LINKEDIN_GET_USER_PROFILE"
```

Use this to retrieve the authenticated user's profile URN, needed for some operations.

## Recipe Pattern: HARPER Content Pipeline

```
1. scratchpad_read key="content_brief"           # Topic + key points from upstream
2. Draft post text (agent generates copy)
3. composio_execute action="LINKEDIN_CREATE_A_POST_V2" text=<draft>
4. scratchpad_write key="linkedin_post_urn" value=<response.urn>
```

## Key Parameters

| Parameter | Format | Notes |
|-----------|--------|-------|
| `text` | String (max 3000 chars) | Post body with newlines and hashtags |
| `visibility` | Enum | `PUBLIC`, `CONNECTIONS` |
| `article_url` | URL string | For link shares |
| `image_url` | URL string | Publicly accessible image URL |
| `image_alt_text` | String | Accessibility text for images |
| `post_urn` | URN string | `urn:li:share:123` or `urn:li:ugcPost:123` |

## LinkedIn Content Tips

- **Hook in the first line:** LinkedIn truncates after ~210 chars with "...see more". Front-load value.
- **Use line breaks:** Dense paragraphs get skipped. One idea per line, blank lines between sections.
- **Hashtags at the end:** 3-5 max, placed after the main content. More than 5 looks spammy.
- **Call to action:** End with a question or prompt — LinkedIn's algorithm favors comment engagement.
- **No external links in text:** LinkedIn deprioritizes posts with links. Put the link in the first comment instead, or use the `article_url` param for proper link cards.

## Pitfalls

- **Always use v2 actions:** `LINKEDIN_CREATE_A_POST_V2` not `LINKEDIN_CREATE_A_POST`. V1 is deprecated.
- **3000 character limit:** Posts exceeding 3000 chars are rejected. Count carefully — newlines count.
- **Image posts need URN resolution:** `LINKEDIN_CREATE_AN_IMAGE_POST_V2` requires a publicly accessible image URL. Private/authenticated URLs will fail.
- **Rate limits are aggressive:** ~100 posts per day, but LinkedIn may throttle well before that. Comments have separate limits.
- **Post URN format:** Comments require the full URN (`urn:li:share:...`). Passing just the numeric ID will fail.
- **Formatting:** LinkedIn does NOT support markdown. No bold, italic, or headers. Use Unicode characters or CAPS for emphasis if needed.
- **Visibility default:** Always set `visibility` explicitly — omitting it may default to connections-only.

## Quick Reference

| Task | Action |
|------|--------|
| Text post | `LINKEDIN_CREATE_A_POST_V2` |
| Article share | `LINKEDIN_CREATE_A_POST_V2` + `article_url` |
| Image post | `LINKEDIN_CREATE_AN_IMAGE_POST_V2` |
| Comment on post | `LINKEDIN_CREATE_A_COMMENT_ON_A_POST` |
| Get profile | `LINKEDIN_GET_USER_PROFILE` |

## What NOT to Do

- Do NOT use v1 actions (`LINKEDIN_CREATE_A_POST`) — they are deprecated and unreliable
- Do NOT exceed 3000 characters — the API will reject the post
- Do NOT put more than 5 hashtags — LinkedIn's algorithm penalizes hashtag stuffing
- Do NOT use private image URLs for image posts — they must be publicly accessible
- Do NOT pass bare IDs to comment actions — always use full URN format
- Do NOT embed external links in post text — use `article_url` param or post the link as a first comment
