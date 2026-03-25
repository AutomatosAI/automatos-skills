# Automatos SKILL.md Author Guide

Every agent in Automatos is driven by a `SKILL.md` file. This guide defines the format, quality bar, and conventions.

## File Location

```
automatos-skills/{slug}/SKILL.md
```

The directory name (`{slug}`) becomes the skill identifier. Use lowercase kebab-case: `frontend-developer`, `sre`, `deal-strategist`.

## Frontmatter (YAML)

```yaml
---
name: sentinel
description: DevOps watchdog that monitors infrastructure health, error spikes, and LLM costs
version: "2.0.0"
tags: [devops, monitoring, uptime, infrastructure, alerting]
category: agent-role
tools:
  - name: platform_get_system_health
    description: Check overall platform service health and uptime
  - name: platform_get_logs
    description: Retrieve recent logs filtered by severity or keyword
---
```

### Required Fields

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Human-readable name (e.g. "sentinel", "Frontend Developer") |
| `description` | string | One sentence — what this agent does |
| `version` | string | Semver, quoted (e.g. `"1.0.0"`) |
| `tags` | list | 3-5 relevant tags as flow list `[a, b, c]` |
| `category` | string | Always `agent-role` |
| `tools` | list of objects | Each item has `name` (tool function name) and `description` (what the tool does for this agent) |

### Tools Format

Each tool entry has two fields:

```yaml
tools:
  - name: platform_submit_report
    description: Submit status report after each heartbeat cycle
  - name: composio_execute
    description: Execute HubSpot CRM actions for lead management
```

Tool names MUST be real Automatos tools (see Tool Reference below). Each skill should use 3-6 tools.

## Body Sections

### 1. Identity (required)

One heading + 1-2 sentences establishing who this agent is in the Automatos context.

```markdown
# SENTINEL — Infrastructure Health Monitor

You are the infrastructure watchdog for the Automatos platform. Your job is to detect problems before users do.
```

### 2. Workflow (required)

Numbered steps with `json` code blocks showing exact tool calls with realistic parameters.

```markdown
## Workflow

### Step 1: Platform Health Check
```json
{ "tool": "platform_get_system_health" }
```
Record service statuses and response times. Flag any service returning non-200.

### Step 2: Error Detection
```json
{ "tool": "platform_get_logs", "params": { "severity": "error", "limit": 50 } }
```
Count errors in the last 15 minutes.
```

Every step must reference a real tool call. No generic "analyze and implement" steps.

### 3. Output Format (required)

A structured template showing what the agent's output looks like.

```markdown
## Output Format

```
SENTINEL STATUS REPORT — {timestamp}
────────────────────────────
API Health:        {OK|WARN|CRITICAL} — {detail}
Error Rate:        {OK|WARN|CRITICAL} — {count} errors
────────────────────────────
Action Required:   {None | list of actions}
```
```

### 4. What NOT To Do (required)

3-5 anti-patterns specific to the role.

```markdown
## What NOT To Do

- Do not attempt to fix issues — report them only.
- Do not skip any tool call or report "UNKNOWN" for any check.
- Do not guess root causes. State observations only.
```

## Quality Bar

- **60-100 lines** total (frontmatter + body). Dense, actionable, no filler.
- Every line must be actionable or structural.
- No references to external platforms (Cursor, OpenClaw, Qwen, etc.).
- Agent identity should be Automatos-specific.
- Tool calls use realistic parameters — copy the style from reference skills.

## Tool Reference

### Platform Tools

| Tool | Description | Key Params |
|------|-------------|------------|
| `platform_get_system_health` | Service health + response times | — |
| `platform_get_logs` | App logs by severity | `severity`, `limit` |
| `platform_get_llm_usage` | Token usage + cost metrics | — |
| `platform_get_cost_breakdown` | Detailed cost analysis | — |
| `platform_workspace_stats` | Workspace metrics | — |
| `platform_submit_report` | Submit status/audit report | `title`, `report_type`, `status`, `content`, `metrics`, `summary` |
| `platform_get_latest_report` | Read previous report | `agent_name` |
| `platform_create_task` | Create board task | `title`, `description`, `priority`, `status` |
| `platform_list_tasks` | List board tasks | `status` |
| `platform_board_summary` | Board state overview | — |
| `platform_search_memory` | Search workspace knowledge | `query` |
| `platform_search_chat_history` | Search past conversations | `query` |
| `platform_query_loki_logs` | LogQL query | `query` |
| `platform_publish_blog_post` | Publish content | `title`, `content` |
| `platform_schedule_task` | Schedule recurring work | `schedule`, `task` |

### Workspace Tools

| Tool | Description | Key Params |
|------|-------------|------------|
| `workspace_read_file` | Read file | `path` |
| `workspace_write_file` | Write file | `path`, `content` |
| `workspace_list_dir` | List directory | `path` |
| `workspace_grep` | Regex search | `pattern`, `path`, `include`, `max_results` |
| `workspace_exec` | Run command | `command`, `cwd`, `timeout` |
| `workspace_git` | Git operations | `operation`, `args` |

### Composio (External Services)

| Tool | Description | Apps |
|------|-------------|------|
| `composio_execute` | Execute external action | HUBSPOT, GMAIL, LINKEDIN, TWITTER, GOOGLE_SHEETS, GOOGLE_ANALYTICS, JIRA, GITHUB, SLACK |

## Reference Skills

- **Sentinel** (`sentinel/SKILL.md`) — Monitoring/DevOps pattern with platform health tools
- **Scout** (`scout/SKILL.md`) — Sales/outreach pattern with CRM and composio tools

Read both before writing any new skill.
