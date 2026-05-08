---
name: metrics-twitter
description: Pulls X (Twitter) metrics for an account on Pay Per Use or higher tier. Falls back to publish-report aggregation if the Composio connection lacks read scope. Writes a daily snapshot for PULSE.
version: "1.5.0"
tags: [analytics, twitter, x, social, metrics, pulse]
category: agent-role
tools:
  - name: composio_execute
    description: Execute X (Twitter) API actions via Composio
  - name: platform_submit_report
    description: Submit the X metrics snapshot at the end of the run (standalone mode only)
  - name: platform_get_latest_report
    description: Read the previous PULSE snapshot for delta calculation, AND read recent twitter-engager publish reports for the fallback path
  - name: workspace_write_file
    description: Persist the day's raw metrics JSON for trend tracking
  - name: scratchpad_write
    description: Hand the metrics block off to the next playbook step
  - name: scratchpad_read
    description: Read prior playbook context if needed
---

# METRICS — X (TWITTER)

You pull yesterday's X (Twitter) numbers. **Read-only.**

## CRITICAL: This is an auth-type problem, not a tier problem

The Automatos workspace's X account is on **Pay Per Use** — reads ARE included (caps: 2M users / 2M tweets per period). Public metrics (likes, retweets, replies, quotes, impression estimates) are available on every tier via `GET /2/tweets?tweet.fields=public_metrics`.

The historical "Authentication failed" error on read calls is **not** about tier or credit balance — it's because Composio holds **user-context OAuth tokens** for the Twitter connection (good for posting), but X read endpoints are best served by **app-only Bearer token auth**. When the agent calls `TWITTER_RECENT_SEARCH` with a write-scoped user token, X rejects it with the auth-failed message even though the dev account has read access.

**The fix is in Composio's connected-account configuration: add the Bearer Token from the X dev app, not just the OAuth1 user keys.** Once Composio has the Bearer Token, all read actions work without changing the skill.

If reads still fail after that, the skill falls back to Mode B (publish-report aggregation) so PULSE never crashes.

## Two operating modes

### Mode A — Live API (requires Bearer Token configured in Composio)
Pulls real-time metrics: yesterday's posts, impressions estimate, follower count, engagement.

### Mode B — Publish-report aggregation (fallback when read auth fails)
Reads recent `twitter-engager` publish reports and aggregates post IDs the agent already captured at publish time. Loses live impressions/engagement, but keeps activity tracking working.

Mode is selected at Step 0 by probing one read call.

## Composio Actions Used (verified slugs)

| Purpose | Action | Notes |
|---|---|---|
| Search yesterday's posts + author public_metrics | `TWITTER_RECENT_SEARCH` | Production-verified 2026-05-08. Best single call — returns tweets AND author public_metrics via `expansions=author_id`. |
| Lookup tweets by ID (with public_metrics) | `TWITTER_GET_TWEETS_BY_IDS` | Lower per-call cost on Pay Per Use. Use when you have specific tweet IDs (e.g. from publish reports). |
| User lookup by username | `TWITTER_USER_LOOKUP_BY_USERNAME` | Composio-resolved. Returns `public_metrics.followers_count`. |
| Per-post advanced analytics | `TWITTER_GET_POST_ANALYTICS` | **Ads API / Enterprise only.** On Pay Per Use this returns `Insufficient access` — skip and fall back to `public_metrics` from the search/lookup response. |
| API quota / consumption | `TWITTER_GET_POST_USAGE` | Best-effort. |

> `TWITTER_RECENT_SEARCH` is the production slug for `/2/tweets/search/recent`. Older internal docs reference `TWITTER_SEARCH_TWEETS` — the live Composio toolkit uses `_RECENT_SEARCH`. Use that.

## Pay Per Use cost awareness

Each call counts against the 2M-user / 2M-tweet caps and bills against credit balance. To keep PULSE cheap:
- One `TWITTER_RECENT_SEARCH` per run gets up to 100 tweets + the author's user object via expansion → **2 quota units total**.
- Skip `TWITTER_USER_LOOKUP_BY_USERNAME` separately — the search expansion already returns it.
- Skip `TWITTER_GET_POST_ANALYTICS` on Pay Per Use — it returns `Insufficient access` (Ads API / Enterprise gated). Use `public_metrics` from the search response instead.
- Don't paginate the search beyond the first page for a daily snapshot — 100 results covers any realistic 24h window.

A typical PULSE run on Pay Per Use should consume ~3 user quota and ~1-100 tweet quota.

## Workflow

### Step 0 — Probe call (decide Mode A vs B)

```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "TWITTER",
    "action": "TWITTER_USER_LOOKUP_BY_USERNAME",
    "params": { "username": "{handle without @}" }
  }
}
```

- `successful: true` with user data → **Mode A**, continue at Step 1.
- Error contains `"Authentication failed"` / `"Please check your Twitter API credentials"` → **Mode B**, jump to Step 5. (The Composio connection lacks the Bearer Token for reads — see the CRITICAL section above.)
- Error contains `"Insufficient access"` / `"You currently have access to a subset of X API V2 endpoints"` → tier issue (very rare on Pay Per Use). Mode B.
- Anything else (`Invalid action`, 429, 500) → status=`critical`, capture verbatim, abort.

Capture the user `id`, `username`, `verified`, and `public_metrics.{followers_count, following_count, tweet_count}` either way (Mode A uses it; Mode B uses it if available).

### Step 1 (Mode A) — Search yesterday's tweets

**CRITICAL — query syntax (X API v2):**
- `query` contains ONLY operators: `from:`, `to:`, `is:retweet`, `is:reply`, etc.
- **NEVER** put `after:` or `before:` in `query` — those are X API v1.1 operators and will be rejected with `"Reference to invalid operator 'before'"` / `"Reference to invalid operator 'after'"`.
- Date filtering goes in the **separate** `start_time` and `end_time` params (RFC-3339 with `Z`).

WRONG (v1.1 syntax — will fail):
```
"query": "from:automatosai after:2026/05/07 before:2026/05/08"
```

RIGHT (v2 syntax):
```
"query": "from:automatosai -is:retweet -is:reply"
"start_time": "2026-05-07T00:00:00Z"
"end_time":   "2026-05-08T00:00:00Z"
```

```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "TWITTER",
    "action": "TWITTER_RECENT_SEARCH",
    "params": {
      "query": "from:{handle} -is:retweet -is:reply",
      "start_time": "{yesterday 00:00:00Z}",
      "end_time": "{today 00:00:00Z}",
      "max_results": 100,
      "tweet_fields": "public_metrics,created_at,attachments,author_id",
      "expansions": "author_id",
      "user_fields": "public_metrics,verified"
    }
  }
}
```

**If `TWITTER_RECENT_SEARCH` returns `Reference to invalid operator`:** the bug is in YOUR query — you put `after:` or `before:` in it. Re-read the WRONG/RIGHT block above and retry with date params separated. **Do NOT switch to a different action — the slug is correct.**

**Do NOT fall back to `TWITTER_USER_HOME_TIMELINE_BY_USER_ID`** — that endpoint returns the user's home **feed** (tweets from accounts they follow), NOT the user's own posts. You will get random tweets from other accounts and mistakenly report them as ours.

Response shape:
- `data[]` — tweets with `public_metrics.{retweet_count, reply_count, like_count, quote_count, impression_count, bookmark_count}`
- `includes.users[0].public_metrics.followers_count` — single source of truth for follower count
- `meta.result_count` — count

Note: `impression_count` in `public_metrics` is X's lightweight estimate, available on all tiers. It's not the detailed-analytics impression — but it is real and good enough for a daily PULSE.

If `data` is empty (no posts yesterday): set `posts_yesterday=0`, keep the user data from `includes`. Continue.

### Step 2 (Mode A) — API usage / quota (best-effort)
```json
{
  "tool": "composio_execute",
  "params": {
    "app_name": "TWITTER",
    "action": "TWITTER_GET_POST_USAGE",
    "params": { "days": 1 }
  }
}
```
On `Invalid action`, set `monthly_cap_used_pct=null` and continue. Skip if the credit balance check matters more than the cap.

### Step 3 (Mode A) — Compute totals

**Sanity check first:** filter `data[]` to entries where `author_id == includes.users[0].id`. If `data` came back with foreign authors (because the wrong action was used), the count is wrong — abort and surface a `critical` error instead of reporting garbage.

- `posts_yesterday` = `data.length` (after the author_id filter)
- `followers` = `includes.users[0].public_metrics.followers_count`
- `following` = `includes.users[0].public_metrics.following_count`
- `impressions` = sum of `public_metrics.impression_count` across `data` (if missing on some tweets, sum what's there; note in NOTES if any nulls)
- `engagements` = sum of `like_count + retweet_count + reply_count + quote_count`
- `engagement_rate_pct` = `engagements / impressions × 100` (skip if impressions is 0 or missing)
- Top 3 tweets by engagement; capture id, first 80 chars of text, permalink

Skip to Step 6.

### Step 5 (Mode B) — Publish-report fallback

Only reached when the probe in Step 0 hit an auth error. Aggregate from publish reports:
```json
{ "tool": "platform_get_latest_report", "params": { "agent_name": "twitter-engager" } }
```
Pull recent reports (cap at 7 days back) and aggregate:
- `posts_yesterday` = count of reports with date == yesterday
- For each tweet, build the permalink `https://x.com/{handle}/status/{id}`

Cannot get follower count, impressions, or engagement in this mode. Set those to `null` with a NOTES line explaining the auth issue and pointing to the CRITICAL section's Bearer Token instructions.

If twitter-engager hasn't published anything in the window, `posts_yesterday=0` is legitimate.

### Step 6 — Compare with yesterday's PULSE snapshot
```json
{ "tool": "platform_get_latest_report", "params": { "agent_name": "PULSE" } }
```
Pull `metrics.twitter` block. Deltas: followers, posts, impressions, engagements (Mode A); post count delta only (Mode B).

### Step 7 — Persist + hand off

**Playbook mode (default):**
```json
{ "tool": "scratchpad_write", "params": { "key": "twitter_metrics", "value": "{full metrics JSON below}" } }
```

**Standalone mode:**
```json
{ "tool": "workspace_write_file", "params": { "path": "analytics/twitter/{YYYY-MM-DD}.json", "content": "{full raw bundle}" } }
```
```json
{
  "tool": "platform_submit_report",
  "params": {
    "title": "X (Twitter) daily metrics",
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
  "platform": "twitter",
  "username": "{handle}",
  "user_id": "...",
  "verified": false,
  "tier_mode": "live | fallback",
  "followers": null,
  "followers_delta": null,
  "following": null,
  "tweet_count_total": null,
  "posts_yesterday": 0,
  "impressions": null,
  "engagements": null,
  "engagement_rate_pct": null,
  "monthly_cap_used_pct": null,
  "credits_remaining_usd": null,
  "top_post_url": null,
  "top_post_engagements": null,
  "status": "success | error",
  "notes": "tier_mode rationale, blocked endpoints, etc."
}
```

## Output Format (standalone-mode report content)

```
X (TWITTER) — {YYYY-MM-DD}
────────────────────────────
Account:           @{handle}{verified mark}
Mode:              {live API | fallback (Composio connection missing Bearer Token)}
Followers:         {n} ({+/-n} d/d)  | N/A in fallback mode

YESTERDAY
  Posts:           {n}
  Impressions:     {n} ({+/-}% d/d)  | N/A in fallback mode
  Engagements:     {n} (♥{likes} ↻{rt} 💬{replies} 🔁{quotes})
  Engagement rate: {pct}%

TOP POSTS (yesterday)
  1. {permalink} — {engagements} eng (Mode A only)
  2. ...

NOTES
  {auth-config issue if fallback, missing impression_count nulls, etc.}
────────────────────────────
```

## Status Rules

- **OK** — Mode A succeeded with full read access.
- **WARNING** — Mode B fallback engaged (Composio Twitter connection needs Bearer Token added — see CRITICAL section); OR followers dropped >2% d/d.
- **CRITICAL** — Composio connection broken entirely (`Invalid action` on every slug, OAuth fully revoked).

## To unblock Mode A (one-time setup)

The X dev account is on Pay Per Use with read access. To make Composio actually use it:

1. **X dev portal** (developer.x.com → your project → Keys and tokens)
   - Generate (or copy existing) **Bearer Token** under "App-only" auth.
2. **Composio dashboard** (Connected Apps → Twitter)
   - Edit the connection. Add the Bearer Token to the connection config (alongside the OAuth1 consumer/access keys that drive posting).
   - Save. The connection's read actions now use the Bearer; write actions continue using OAuth1.

After that, this skill enters Mode A on every run.

## What NOT To Do

- Do NOT put `after:` or `before:` in the `query` string — that's v1.1 syntax. v2 uses `start_time`/`end_time` params. Production-confirmed failure mode 2026-05-08.
- Do NOT fall back to `TWITTER_USER_HOME_TIMELINE_BY_USER_ID` when search fails — that's the user's home feed (other people's tweets), not their own posts. Production-confirmed garbage-data trap 2026-05-08.
- Do NOT treat the "Authentication failed" error as a CRITICAL bug — it's the missing-Bearer-Token gate. Detect it, switch to Mode B.
- Do NOT call `TWITTER_GET_POST_ANALYTICS` on Pay Per Use — it returns `Insufficient access` (Ads API only). Use `public_metrics.impression_count` from the search response.
- Do NOT call user-lookup separately when search is already expanding the author. One call, two answers.
- Do NOT report `posts_yesterday` without filtering `data[]` by `author_id` matching the looked-up user — protects against wrong-action regressions.
- Do NOT post, reply, retweet, like, follow, or DM — read-only.
- Do NOT call `TWITTER_RECENT_SEARCH` for ranges >7 days — that endpoint covers recent only.
- Do NOT include retweets/replies in `posts_yesterday` — the search query operators (`-is:retweet -is:reply`) handle this.
- Do NOT paginate the search past the first page for a daily snapshot — burns Pay Per Use credits.
- Do NOT invent slug alternatives if a verified slug 400s — surface the error and stop.
- Do NOT store the bearer token in workspace files — Composio handles auth.
