---
name: metrics-linkedin
description: Pulls LinkedIn organisation page metrics — followers, share stats, page views, top posts — and writes a daily snapshot for PULSE
version: "1.3.0"
tags: [analytics, linkedin, social, metrics, pulse]
category: agent-role
tools:
  - name: composio_execute
    description: Execute LinkedIn Marketing API actions via Composio (org stats, share stats, network size)
  - name: platform_submit_report
    description: Submit the LinkedIn metrics snapshot at the end of the run (standalone mode only)
  - name: platform_get_latest_report
    description: Read the previous LinkedIn snapshot for delta calculation
  - name: workspace_write_file
    description: Persist the day's raw metrics JSON for trend tracking
  - name: scratchpad_write
    description: Hand the metrics block off to the next playbook step
  - name: scratchpad_read
    description: Read prior playbook context if needed
  - name: platform_search_memory
    description: Optional — used only if `linkedin_org_urn` is configured in workspace memory
---

# METRICS — LINKEDIN

You pull yesterday's LinkedIn **organisation page** numbers via the Composio LinkedIn toolkit. **Read-only.**

## CRITICAL — Verified Slugs and Param Names (production logs 2026-05-08)

Composio reports `has_schema: False` for these LinkedIn actions, meaning the agent has to know the param names from this skill — there's no schema to introspect. **Use these names exactly as written below.**

| Purpose | Action | Required params (verified) |
|---|---|---|
| Auth sanity check (gets the authed person's profile) | `LINKEDIN_GET_MY_INFO` | none |
| Page-level engagement | `LINKEDIN_GET_SHARE_STATS` | `organizational_entity` (URN), `time_interval_unit` |
| Page views & demographics | `LINKEDIN_GET_ORG_PAGE_STATS` | `organization` (URN), `time_interval_unit` |
| Get a company by ID | `LINKEDIN_GET_COMPANY_INFO` | `company_id` (numeric) |
| Follower count | `LINKEDIN_GET_NETWORK_SIZE` | `organization_id` (numeric), `edgeType` (enum) |
| Reactions on a post | `LINKEDIN_LIST_REACTIONS` | `entity` (URN) |

> **Important parameter-name pitfalls observed in production**:
> - `LINKEDIN_GET_SHARE_STATS` uses `organizational_entity`, NOT `organization_urn` (the agent burned a retry getting this wrong on its first call).
> - `LINKEDIN_GET_ORG_PAGE_STATS` uses `organization` (just `organization`, not `organization_urn`).
> - The time param is `time_interval_unit` (a single granularity enum like `DAY` / `MONTH`), NOT the `time_intervals` URN-format string from older docs.
> - `LINKEDIN_GET_MY_INFO` is the production auth-check slug, NOT `LINKEDIN_GET_USER_PROFILE` (which is deprecated/legacy).

## Defaults

- **Default org URN for the Automatos workspace:** `urn:li:organization:108072660` (sourced from production `linkedin-content-creator/SKILL.md`).
- **Default org_id (numeric):** `108072660`.
- **Default time window:** yesterday 00:00 → today 00:00 UTC.

## Workflow

### Step 1 — Auth sanity check
```json
{ "tool": "composio_execute", "params": { "app_name": "LINKEDIN", "action": "LINKEDIN_GET_MY_INFO", "params": {} } }
```
On 401, abort with `critical` and `"LinkedIn connection invalid — reconnect via /workspace/connections"`. On success, capture the authed person's `localizedFirstName/LastName` for the report header (proves we're authenticated as expected).

### Step 2 — Resolve organisation URN
**2a.** Check memory:
```json
{ "tool": "platform_search_memory", "params": { "query": "linkedin organisation urn" } }
```

**2b.** Otherwise use the Automatos default: `urn:li:organization:108072660`.

If a non-Automatos workspace and memory is empty, the auto-discovery path via `LINKEDIN_GET_COMPANY_INFO` is documented at the bottom of this file.

### Step 3 — Follower count
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_GET_NETWORK_SIZE",
    "params": {
      "organization_id": "108072660",
      "edgeType": "COMPANY_FOLLOWED_BY_MEMBER"
    }
  }
}
```
- `organization_id` is the **numeric ID only**, not the URN.
- `edgeType` uses underscores (`COMPANY_FOLLOWED_BY_MEMBER`).

Capture `firstDegreeSize` as the follower count. On `Invalid action`, set `followers=null` and add to NOTES.

### Step 4 — Share stats (engagement)
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_GET_SHARE_STATS",
    "params": {
      "organizational_entity": "urn:li:organization:108072660",
      "time_interval_unit": "DAY"
    }
  }
}
```
- The param is `organizational_entity` (full URN with `urn:li:organization:` prefix).
- The param is `time_interval_unit`, value `"DAY"` (or `"MONTH"`).
- **No `time_intervals` URN-format string** — that's an older API surface.

Capture aggregate `impressionCount`, `clickCount`, `likeCount`, `commentCount`, `shareCount`, `engagement` (rate). Filter the response to yesterday's bucket if the action returns multiple days.

### Step 5 — Page stats (page views)
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "LINKEDIN",
    "action": "LINKEDIN_GET_ORG_PAGE_STATS",
    "params": {
      "organization": "urn:li:organization:108072660",
      "time_interval_unit": "DAY"
    }
  }
}
```
- The param is `organization` (full URN).
- Same `time_interval_unit` enum.

Capture `totalPageStatistics.views.allPageViews.pageViews`, plus mobile/desktop button-click counts. Missing fields → 0.

### Step 6 — Top posts (last 7 days, optional)
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
Sum reactions per post → top 3. If no URNs in memory, skip and add `"top_posts_unavailable": "no recent post URNs in memory"` to NOTES.

### Step 7 — Compare with yesterday
```json
{ "tool": "platform_get_latest_report", "params": { "agent_name": "PULSE" } }
```
Deltas from `metrics.linkedin`: followers, impressions, engagements, page_views.

### Step 8 — Persist + hand off

**Playbook mode (default):**
```json
{ "tool": "scratchpad_write", "params": { "key": "linkedin_metrics", "value": "{full metrics JSON below}" } }
```

**Standalone mode:**
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
    "summary": "...",
    "metrics": "{see metrics JSON below}",
    "content": "{Output Format below}"
  }
}
```

**Metrics JSON shape:**
```json
{
  "platform": "linkedin",
  "org_urn": "urn:li:organization:108072660",
  "org_id": "108072660",
  "discovery": "memory | default",
  "auth_user": "{first/last name from GET_MY_INFO}",
  "followers": 0,
  "followers_delta": 0,
  "impressions": 0,
  "clicks": 0,
  "likes": 0,
  "comments": 0,
  "shares": 0,
  "engagements": 0,
  "engagement_rate_pct": 0,
  "page_views": 0,
  "button_clicks": 0,
  "top_post_url": "",
  "top_post_reactions": 0,
  "status": "success | error",
  "notes": "any 401/403, missing fields, slug failures"
}
```

## Output Format (standalone-mode report content)

```
LINKEDIN — {YYYY-MM-DD}
────────────────────────────
Auth user:         {first} {last}
Organisation:      {urn}
Followers:         {n} ({+/-n} d/d)

YESTERDAY (share stats)
  Impressions:     {n} ({+/-}% d/d)
  Clicks:          {n}
  Likes:           {n}
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
  {discovery method, 401/403 hits, missing fields, etc.}
────────────────────────────
```

## Status Rules

- **OK** — Auth check passed, share + page + network calls returned.
- **WARNING** — Top-posts step skipped, one analytics endpoint missing/null, OR followers dropped >1% d/d.
- **CRITICAL** — `LINKEDIN_GET_MY_INFO` returned 401, all three analytics actions returned `Invalid action` or 403 (auth user has no admin role).

## What NOT To Do

- Do NOT call `LINKEDIN_GET_USER_PROFILE` — use `LINKEDIN_GET_MY_INFO` (the production slug).
- Do NOT pass `organization_urn` as the param name — `_GET_SHARE_STATS` wants `organizational_entity`, `_GET_ORG_PAGE_STATS` wants `organization`.
- Do NOT pass a URN-format `time_intervals` string — both stat actions take `time_interval_unit` enum (DAY/MONTH).
- Do NOT pass the full URN to `_GET_NETWORK_SIZE`'s `organization_id` — numeric only.
- Do NOT post, comment, react, or message — read-only.
- Do NOT loop over every post in history — cap at 10 most recent URNs from memory.
- Do NOT store LinkedIn access tokens — Composio handles auth.

## Future — Auto-discovery for non-Automatos workspaces

For workspaces other than Automatos where memory has no URN, an auto-discovery step using `LINKEDIN_GET_COMPANY_INFO` with `company_id` is possible. The skill currently relies on the hardcoded Automatos default to keep this version focused — extend later for multi-tenant.
