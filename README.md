# Automatos Skills

Agent skills for the [Automatos AI Platform](https://github.com/AutomatosAI/automatos-ai). Each skill is a `SKILL.md` file that gets injected into an agent's system prompt, teaching it **how** to perform a specific job.

## What's a Skill?

A skill is a document — instructions, process guidance, formats, and examples. When an agent reads it, it understands how to do a task without the recipe prompt micromanaging every tool call.

**Without skills:** Recipe prompts are 50+ lines of "call this tool, format it like this, export to scratchpad with this key..."

**With skills:** Recipe prompts become simple intent — "Create Jira tickets for P0/P1 bugs from the QA report."

## Available Skills

### Agent Roles (The Automatos Team)

| Agent | Skill | Description |
|-------|-------|-------------|
| **SENTINEL** | `sentinel/` | Infrastructure watchdog — API health, error spikes, deploy status, cost anomalies. Heartbeat every 15min. |
| **PATCHER** | `bug-fixer/` | End-to-end bug fixing: read ticket → find code → write failing test → minimal fix → verify → commit → open draft PR. |
| **SCOUT** | `scout/` | Lead intelligence — qualify signups, score prospects (1-10), draft outreach, track pipeline. |
| **HARPER** | `harper/` | Content machine — LinkedIn posts, changelog updates, tweets from real git history and platform data. |
| **ECHO** | `echo/` | Customer support — classify messages, auto-reply FAQs, create bug tickets, flag churn signals. |
| **ATLAS** | `atlas/` | Business intelligence — weekly cost/revenue analysis, usage trends, anomaly detection, optimization recommendations. |
| **FORGE** | `forge/` | Recipe builder — natural language → multi-step workflows with agent assignments and triggers. |
| **ORACLE** | `oracle/` | Knowledge curator — audit RAG health, reprocess failures, sync Google Drive, prune stale content. |
| **RALLY** | `rally/` | Community growth — map relevant communities, identify creators and builders, score outreach opportunities, grow ecosystem authentically. |

### DevOps Pipeline (Existing)

| Skill | Directory | Description |
|-------|-----------|-------------|
| **QA Engineer** | `qa-engineer/` | Run tests, classify failures by severity (P0–P3), correlate with server logs, produce structured JSON reports. |
| **Jira Admin** | `jira-admin/` | Create, update, transition, and comment on Jira issues. Priority mapping, ticket lifecycle, data handoff. |

### Integration Skills (Composio)

| Skill | Directory | Description |
|-------|-----------|-------------|
| **Gmail** | `gmail-automation/` | Send, reply, search, labels, drafts, attachments via Composio Gmail toolkit. |
| **Slack** | `slack-automation/` | Messages, search, threads, channels, reactions, scheduling via Composio Slack toolkit. |
| **Jira** | `jira-automation/` | JQL search, create/edit issues, sprints, comments via Composio Jira toolkit. |
| **Google Calendar** | `google-calendar-automation/` | Events, availability, attendees, recurring events via Composio Calendar toolkit. |
| **LinkedIn** | `linkedin-automation/` | Text posts, article shares, image posts, comments via Composio LinkedIn toolkit. |
| **Twitter/X** | `twitter-automation/` | Posts, search, bookmarks, replies via Composio Twitter toolkit. |
| **TikTok** | `tiktok-automation/` | Video/photo upload, publish, status polling via Composio TikTok toolkit. |

## SKILL.md Format

Each skill uses YAML frontmatter + Markdown:

```yaml
---
name: skill-name
description: What this skill teaches the agent
version: "1.0.0"
tags: [relevant, searchable, tags]
tools:
  - name: TOOL_NAME
    description: What this tool does
---

# Skill Name

Instructions, process, formats, examples...
```

### Required Frontmatter Fields

- `name` — Unique skill identifier (must match across re-uploads for upsert)
- `description` — Short summary shown in skill listings

### Optional Frontmatter Fields

- `version` — Semver string (default: `1.0.0`)
- `tags` — Array of searchable tags
- `tools` — Tool schemas the agent should know about
- `category` — Grouping category (default: `general`)
- `skill_type` — Type classification (default: `custom`)

## Uploading to Automatos

### Via Git Import (Admin)

1. Go to **Marketplace > Skills** in the Automatos dashboard
2. Click **Upload / Import** and provide this repo's Git URL
3. The skill loader recursively finds all `SKILL.md` files and indexes them
4. Assign skills to agents in the **Agent Builder > Capabilities** tab

### Re-uploading / Updates

The loader uses **upsert** — it matches on skill `name` + source. Re-importing the same repo updates existing skills in-place. No duplicates are created. Version field is updated from frontmatter.

## Assigning Skills to Agents

Skills are assigned per-agent (1–3 skills is the sweet spot for focused agents):

- **Jira Admin Agent** → `jira-admin` skill
- **QA Test Engineer** → `qa-engineer` skill
- **Bug Fixer Agent** → `bug-fixer` skill

The agent's system prompt automatically includes all assigned skill content at activation time.

## Writing New Skills

1. Create a new directory: `my-skill/`
2. Add a `SKILL.md` with YAML frontmatter and markdown instructions
3. Focus on **how** — the process, formats, conventions, and guardrails
4. Keep it under ~2000 tokens (~8KB) for prompt efficiency
5. Include a "What NOT to Do" section to prevent common mistakes
6. Push and re-import — existing skills update, new ones are added

## Recipe Prompt Comparison

### Before (without skills)
```
Create JIRA tickets for P0 and P1 issues from the QA report.
Read scratchpad key "qa_report" for the issues array.
For each issue where severity is P0 or P1, call JIRA_CREATE_ISSUE:
  project_key: "PILOT"
  summary: "[Nightly Test] " + issue.title
  issue_type: "Bug"
  priority_name: "Highest" (P0) or "High" (P1)
  labels: ["auto-test", "nightly-run", issue.category]
  description: test nodeid, error snippet, root cause hint, severity
...
```

### After (with jira-admin skill)
```
Create Jira bug tickets in PILOT for all P0 and P1 bugs from the qa_report.
Label them "auto-test" and "nightly-run". Export ticket keys to scratchpad.
```

## License

Internal — AutomatosAI
