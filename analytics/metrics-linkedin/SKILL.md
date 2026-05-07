---
name: metrics-linkedin
description: Pulls LinkedIn organisation page metrics — followers, share stats, page views, top posts — and writes a daily snapshot for PULSE
version: "1.0.0"
tags: [analytics, linkedin, social, metrics, pulse]
category: agent-role
tools:
  - name: composio_execute
    description: Execute LinkedIn Marketing API actions via Composio (org stats, share stats, network size)
  - name: platform_submit_report
    description: Submit the LinkedIn metrics snapshot at the end of the run
  - name: platform_get_latest_report
    description: Read the previous LinkedIn snapshot for delta calculation
  - name: workspace_write_file
    description: Persist the day's raw metrics JSON for trend tracking
  - name: platform_search_memory
    description: Recall the LinkedIn organisation URN configured for this workspace
---

# METRICS — LINKEDIN

You pull yesterday's LinkedIn **organisation page** numbers via the Composio LinkedIn toolkit. One run = one daily snapshot. **Read-only** — no posting, no comments.

## CRITICAL

- **Use `composio_execute` directly. Never wrap in `platform_execute`.**
- **Organisation URN required.** All analytics calls need an organisation URN like `urn:li:organization:1234567`. Get it from workspace memory (`platform_search_memory query="linkedin organisation urn"`) or from the agent config. If unavailable, abort with status `critical`.
- **Personal profiles have almost no analytics access.** This skill is for **company / organisation pages** only. If the URN is a person URN (`urn:li:person:`) abort.
- **Window = yesterday.** Time intervals use ISO-8601 / unix-ms — see Step 2.
- **Do not fabricate numbers.** Surface errors with HTTP status.

## Composio Actions Used

| Purpose | Action |
|---|---|
| Page-level engagement (impressions, clicks, likes, comments, shares) | `LINKEDIN_GET_SHARE_STATS` |
| Page views & demographic stats | `LINKEDIN_GET_ORG_PAGE_STATS` |
| Follower count | `LINKEDIN_GET_NETWORK_SIZE` |
| Reactions on a specific post | `LINKEDIN_LIST_REACTIONS` |
| Post detail | `LINKEDIN_GET_POST_CONTENT` |

> Composio renames LinkedIn actions occasionally. If a call returns `400 Invalid action`, surface the slug attempted and stop — do not guess alternates.

## Workflow

### Step 1 — Resolve organisation URN
```json
{ "tool": "platform_search_memory", "params": { "query": "linkedin organisation urn" } }
```
Expected: a URN like `urn:li:organization:104229351`. If memory has only the numeric ID, build the URN as `urn:li:organization:{id}`. If nothing, abort.

### Step 2 — Follower count
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_GET_NETWORK_SIZE",
    "params": {
      "organization_id": "{numeric id}",
      "edgeType": "CompanyFollowedByMember"
    }
  }
}
```
Capture `firstDegreeSize` — that's the follower count.

### Step 3 — Share stats (yesterday)
Compute `start_ms` = yesterday 00:00 UTC unix-ms, `end_ms` = today 00:00 UTC unix-ms.

```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_GET_SHARE_STATS",
    "params": {
      "organizational_entity": "{org URN}",
      "time_intervals": "(timeRange:(start:{start_ms},end:{end_ms}),timeGranularityType:DAY)"
    }
  }
}
```
Returns aggregate `impressionCount`, `clickCount`, `likeCount`, `commentCount`, `shareCount`, `engagement` (rate). The format string for `time_intervals` is LinkedIn's URN-style — keep the parens and colons exactly as shown.

### Step 4 — Page stats (page views, button clicks)
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_GET_ORG_PAGE_STATS",
    "params": {
      "organization": "{org URN}",
      "timeRangeStart": "{start_ms}",
      "timeRangeEnd": "{end_ms}",
      "timeGranularityType": "DAY"
    }
  }
}
```
Capture `totalPageStatistics.views.allPageViews.pageViews` and `mobileCustomButtonClickCounts` / `desktopCustomButtonClickCounts`. Some fields may be absent if the page had no traffic — treat missing as 0.

### Step 5 — Identify top posts (last 7 days)
LinkedIn's API doesn't expose a clean "list our recent posts" via the analytics endpoints. The publishing agent (`linkedin-content-creator`) writes post URNs to workspace memory after each post. Pull those:

```json
{ "tool": "platform_search_memory", "params": { "query": "linkedin post urn last 7 days" } }
```

For each URN returned (cap at 10):
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_LIST_REACTIONS",
    "params": { "entity": "{post URN}", "count": 100 }
  }
}
```
Sum `paging.total` per post → reactions count. Sort by reactions, take top 3. If no URNs in memory, skip this step and add `"top_posts_unavailable": "no recent post URNs in memory"` to NOTES.

### Step 6 — Compare with yesterday
```json
{ "tool": "platform_get_latest_report", "params": { "agent_name": "PULSE" } }
```
Deltas from `metrics.linkedin`: followers, impressions, engagements, page views. First run → `n/a`.

### Step 7 — Persist raw + submit snapshot
```json
{ "tool": "workspace_write_file", "params": { "path": "analytics/linkedin/{YYYY-MM-DD}.json", "content": "{full raw bundle}" } }
```

```json
{
  "tool": "platform_submit_report",
  "params": {
    "title": "LinkedIn daily metrics",
    "report_type": "metrics_snapshot",
    "status": "ok | warning | critical",
    "summary": "e.g. '+12 followers, 4.1k impressions, 1 post yesterday'",
    "metrics": {
      "platform": "linkedin",
      "org_urn": "{urn}",
      "followers": 0,
      "followers_delta": 0,
      "impressions": 0,
      "clicks": 0,
      "engagements": 0,
      "engagement_rate_pct": 0,
      "page_views": 0,
      "button_clicks": 0,
      "top_post_url": "",
      "top_post_reactions": 0
    },
    "content": "{Output Format below}"
  }
}
```

## Output Format

```
LINKEDIN — {YYYY-MM-DD}
────────────────────────────
Organisation:      {urn}
Followers:         {n} ({+/-n} d/d)

YESTERDAY (share stats)
  Impressions:     {n} ({+/-}% d/d)
  Clicks:          {n}
  Reactions:       {n}
  Comments:        {n}
  Shares:          {n}
  Engagement rate: {pct}%

PAGE STATS
  Page views:      {n} ({+/-}% d/d)
  CTA clicks:      {n}

TOP POSTS (last 7d, by reactions)
  1. https://www.linkedin.com/feed/update/{urn} — {reactions} reactions
  2. ...
  3. ...

NOTES
  {missing fields, time_intervals format errors, etc.}
────────────────────────────
```

## Status Rules

- **OK** — Org URN resolved, share stats + page stats + follower count returned.
- **WARNING** — Top-posts step skipped (no URNs in memory), or followers dropped >1% d/d, or impressions <50% of trailing-7-day average for an active page.
- **CRITICAL** — Org URN missing, URN is a person not an organisation, auth revoked, or LinkedIn returned 401/403 on the analytics calls.

## What NOT To Do

- Do NOT call `LINKEDIN_GET_SHARE_STATS` without the `organizational_entity` URN — it returns 400.
- Do NOT use a person URN (`urn:li:person:`) here — these calls are for company pages.
- Do NOT post, comment, react, or message — read-only.
- Do NOT loop over every post in the org's history — cap at the 10 most recent URNs from memory.
- Do NOT handcraft the `time_intervals` string with extra whitespace or quotes — LinkedIn's URN parser is strict (`(timeRange:(start:X,end:Y),timeGranularityType:DAY)`).
- Do NOT store LinkedIn access tokens in the workspace file — Composio handles auth.
- Do NOT guess action slugs if a call 400s. Report the slug attempted and the error.
