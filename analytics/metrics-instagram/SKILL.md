---
name: metrics-instagram
description: Pulls Instagram account + post metrics — followers, reach, views, profile-link taps, top posts — and writes a daily snapshot for PULSE
version: "1.1.0"
tags: [analytics, instagram, social, metrics, pulse]
category: agent-role
tools:
  - name: composio_execute
    description: Execute Instagram Graph API actions via Composio (insights, media, user info)
  - name: platform_submit_report
    description: Submit the Instagram metrics snapshot at the end of the run (standalone mode only)
  - name: platform_get_latest_report
    description: Read the previous Instagram snapshot for delta calculation
  - name: workspace_write_file
    description: Persist the day's raw metrics JSON for trend tracking
  - name: scratchpad_write
    description: Hand the metrics block off to the next playbook step
  - name: scratchpad_read
    description: Read prior playbook context if needed
---

# METRICS — INSTAGRAM

You pull yesterday's Instagram numbers for a Business / Creator account via the Composio Instagram Graph API toolkit. One run = one daily snapshot. **Read-only.**

## CRITICAL

- **Use `composio_execute` directly. Never wrap in `platform_execute`.**
- **Account type matters.** Insights require a **Business** or **Creator** account linked to a Facebook Page. If `INSTAGRAM_GET_USER_INFO` returns `account_type: PERSONAL`, abort with status `critical`.
- **Use Graph API v22 metric names.** `impressions`, `profile_views`, `website_clicks` are deprecated — they will return empty. Use `views`, `views` (content) and `profile_links_taps` instead.
- **`metric` is an ARRAY, not a comma-separated string.** Pass `["views", "reach", ...]`. The Composio wrapper will reject CSVs.
- **Most metrics need `metric_type: "total_value"`.** Without it, `time_series` is the default and many metrics return empty.
- **Window = yesterday.** `since`/`until` are Unix timestamps (seconds, not ms). Yesterday 00:00 → today 00:00 UTC.

## Composio Actions Used

| Purpose | Action |
|---|---|
| Account profile + follower count (own account only) | `INSTAGRAM_GET_USER_INFO` |
| Account-level insights | `INSTAGRAM_GET_USER_INSIGHTS` |
| Recent posts list | `INSTAGRAM_GET_IG_USER_MEDIA` |
| Per-post engagement | `INSTAGRAM_GET_IG_MEDIA_INSIGHTS` |
| Publishing quota left | `INSTAGRAM_GET_IG_USER_CONTENT_PUBLISHING_LIMIT` |

`INSTAGRAM_GET_POST_INSIGHTS` is **deprecated** — do not use it.

## Valid User-Insights Metrics (v22)

| Metric | Period | metric_type | Notes |
|---|---|---|---|
| `views` | day | `total_value` | **Replaces `impressions` from v22.** |
| `reach` | day | `total_value` | |
| `accounts_engaged` | day | `total_value` | |
| `total_interactions` | day | `total_value` | |
| `likes` | day | `total_value` | |
| `comments` | day | `total_value` | |
| `shares` | day | `total_value` | |
| `saves` | day | `total_value` | |
| `replies` | day | `total_value` | |
| `follows_and_unfollows` | day | `total_value` | |
| `profile_links_taps` | day | `total_value` | **Replaces `website_clicks`.** Use breakdown=`contact_button_type` to split by link type. |
| `follower_count` | day | (default time_series) | Daily follower delta — only available for last 30 days. |
| `online_followers` | lifetime | (default) | |

**Deprecated — do NOT request:** `impressions`, `profile_views`, `website_clicks`, `email_contacts`, `phone_call_clicks`, `text_message_clicks`, `get_directions_clicks`.

## Workflow

### Step 1 — Resolve account
```json
{ "tool": "composio_execute", "params": { "app_name": "INSTAGRAM", "action": "INSTAGRAM_GET_USER_INFO", "params": { "ig_user_id": "me" } } }
```
Record `id`, `username`, `followers_count`, `follows_count`, `media_count`, `account_type`. Pass `ig_user_id="me"` — `followers_count`/`follows_count` are only populated when querying `me`. If `account_type` is not `BUSINESS` or `CREATOR`, abort with `critical`.

### Step 2 — Account insights (yesterday, total_value)
Compute `since` = yesterday 00:00 UTC unix-seconds, `until` = today 00:00 UTC unix-seconds.

```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "INSTAGRAM",
    "action": "INSTAGRAM_GET_USER_INSIGHTS",
    "params": {
      "ig_user_id": "{id from step 1}",
      "metric": ["views", "reach", "accounts_engaged", "total_interactions", "likes", "comments", "shares", "saves", "profile_links_taps"],
      "period": "day",
      "metric_type": "total_value",
      "since": "{yesterday 00:00 UTC unix}",
      "until": "{today 00:00 UTC unix}"
    }
  }
}
```
Each metric in the response has a `total_value.value`. If the response includes `composio_execution_message: "Note: Requested N metric(s) but received M"`, that is **expected** — Instagram silently drops a metric when there's no data. Treat missing metrics as `0`, not as an error. Add a one-line entry to NOTES listing which metrics were dropped.

### Step 3 — Daily follower delta (separate call, time_series)
`follower_count` only works on `period=day` with the default `time_series` mode and only for the last 30 days. Call it on its own:
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "INSTAGRAM",
    "action": "INSTAGRAM_GET_USER_INSIGHTS",
    "params": {
      "ig_user_id": "{id from step 1}",
      "metric": ["follower_count"],
      "period": "day",
      "since": "{yesterday 00:00 UTC unix}",
      "until": "{today 00:00 UTC unix}"
    }
  }
}
```
Read `values[0].value` for yesterday's gain. If empty, fall back to comparing `followers_count` from Step 1 against the previous PULSE report.

### Step 4 — Recent posts (last 7 days)
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "INSTAGRAM",
    "action": "INSTAGRAM_GET_IG_USER_MEDIA",
    "params": {
      "ig_user_id": "{id from step 1}",
      "fields": "id,caption,media_type,media_url,permalink,timestamp,like_count,comments_count",
      "limit": 25
    }
  }
}
```
**Always pass `ig_user_id`** — the API rejects the call without it. Filter client-side to posts with `timestamp` in the last 7 days. Sort by `like_count + comments_count` to find the top 3.

### Step 5 — Per-post insights (top 3 only)
For each top-3 media id:
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "INSTAGRAM",
    "action": "INSTAGRAM_GET_IG_MEDIA_INSIGHTS",
    "params": {
      "ig_media_id": "{media_id}",
      "metric": ["views", "reach", "saved", "total_interactions"]
    }
  }
}
```
For Reels (`media_type=VIDEO` or `REELS`) some metrics differ. If a 400 mentions an invalid metric, retry with `["ig_reels_video_view_total_time", "reach", "saved", "total_interactions"]`. For Stories use `["replies", "navigation", "follows", "profile_visits"]`.

> **Account follower minimum:** Per-post insights require the account to have ≥1,000 followers AND media published in the last 2 years. If the account is below the threshold, skip Step 5 and add a NOTES line.

### Step 6 — Publishing limit
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "INSTAGRAM",
    "action": "INSTAGRAM_GET_IG_USER_CONTENT_PUBLISHING_LIMIT",
    "params": { "ig_user_id": "{id from step 1}" }
  }
}
```
Capture `quota_total - quota_usage` as remaining (typically 100/day, but this is workspace-specific).

### Step 7 — Compare with yesterday's snapshot
```json
{ "tool": "platform_get_latest_report", "params": { "agent_name": "PULSE" } }
```
Pull the previous `metrics.instagram` block. Compute deltas: `followers Δ`, `views Δ`, `reach Δ`, `profile_links_taps Δ`. First run → `n/a`.

### Step 8 — Persist + hand off

**Playbook mode (default — when this skill runs as a playbook step):**
```json
{ "tool": "scratchpad_write", "params": { "key": "instagram_metrics", "value": "{full metrics JSON below}" } }
```
Skip `platform_submit_report` — the synthesis step submits the consolidated report.

**Standalone mode (when this skill is invoked directly):**
Submit the report directly — do NOT write a JSON file to the workspace (paths get rooted inside the active repo).
```json
{
  "tool": "platform_submit_report",
  "params": {
    "title": "Instagram daily metrics",
    "report_type": "metrics_snapshot",
    "status": "ok | warning | critical",
    "summary": "e.g. '+2 followers, 412 views (+18%), 3 posts'",
    "metrics": "{see metrics JSON below}",
    "content": "{Output Format below}"
  }
}
```

**Metrics JSON shape** (for both modes):
```json
{
  "platform": "instagram",
  "username": "{username}",
  "account_type": "BUSINESS|CREATOR",
  "followers": 0,
  "followers_delta": 0,
  "views": 0,
  "reach": 0,
  "accounts_engaged": 0,
  "total_interactions": 0,
  "profile_links_taps": 0,
  "posts_yesterday": 0,
  "posts_7d": 0,
  "publishing_limit_remaining": 0,
  "top_post_url": null,
  "top_post_engagement": 0,
  "status": "success | error",
  "notes": "any dropped metrics, account-size skips, or warnings"
}
```

## Output Format (standalone-mode report content)

```
INSTAGRAM — {YYYY-MM-DD}
────────────────────────────
Account:           @{username} ({account_type})
Followers:         {n} ({+/-n} d/d)
Posts on file:     {media_count}
Publishing left:   {n}/100 today

YESTERDAY
  Views:           {n} ({+/-}% d/d)   [v22 — was "impressions"]
  Reach:           {n} ({+/-}% d/d)
  Accounts Eng.:   {n}
  Interactions:    {n}
  Profile Taps:    {n}                [v22 — was "website_clicks"]

TOP POSTS (last 7d, by engagement)
  1. {permalink} — {likes}❤ {comments}💬 reach {n}
  2. {permalink} — {likes}❤ {comments}💬 reach {n}
  3. {permalink} — {likes}❤ {comments}💬 reach {n}

NOTES
  {dropped metrics, follower-threshold skips, etc.}
────────────────────────────
```

## Status Rules

- **OK** — All calls succeeded, account is BUSINESS/CREATOR.
- **WARNING** — Multiple metrics dropped, publishing quota <5, account <1k followers (per-post insights skipped), or followers dropped >2% d/d.
- **CRITICAL** — Account is PERSONAL, auth revoked, or `INSTAGRAM_GET_USER_INFO` returned 401/403.

## What NOT To Do

- Do NOT request `impressions`, `profile_views`, or `website_clicks` — deprecated in v22.
- Do NOT pass `metric` as a comma-separated string — it must be an array.
- Do NOT request `follower_count` in the same call as `total_value` metrics — it has different period rules. Separate call.
- Do NOT call deprecated `INSTAGRAM_GET_POST_INSIGHTS`.
- Do NOT request per-post insights for every post — only the top 3 by engagement.
- Do NOT post, comment, like, or modify anything — read-only.
- Do NOT fabricate `followers_delta` if there's no previous report — write `null`.
- Do NOT swallow Composio errors. If `successful: false`, surface the `error` field in NOTES.
- Do NOT store IG access tokens in the workspace file — Composio handles auth.
