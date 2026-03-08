---
name: jira-automation
description: Search, create, edit issues and manage sprints in Jira
version: "1.0.0"
tags: [jira, project-management, issues]
category: productivity
tools:
  - name: composio_execute
    description: Execute Composio actions (Jira API operations)
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# Jira Automation Skill

## Prerequisites

Jira must be connected in **Settings > Integrations**. The agent must have the `composio_execute` tool assigned.

## Core Workflows

### 1. Search Issues (JQL)

```
composio_execute action="JIRA_SEARCH_ISSUES" params={
  "jql": "project = PROJ AND status = 'In Progress' AND assignee = currentUser() ORDER BY priority DESC",
  "maxResults": 20
}
```

**JQL reference:**
| Clause | Example | Notes |
|--------|---------|-------|
| `project` | `project = PROJ` | Project key |
| `status` | `status = "To Do"` | Quote multi-word values |
| `assignee` | `assignee = currentUser()` | Current user function |
| `reporter` | `reporter = "alice"` | By username |
| `priority` | `priority = High` | Highest/High/Medium/Low/Lowest |
| `type` | `type = Bug` | Bug/Story/Task/Epic/Sub-task |
| `sprint` | `sprint in openSprints()` | Active sprints |
| `labels` | `labels = backend` | Label match |
| `created` | `created >= -7d` | Relative dates: -1d, -2w, -1m |
| `updated` | `updated >= "2026-03-01"` | Absolute date |
| `text` | `text ~ "login error"` | Full-text search |
| `ORDER BY` | `ORDER BY created DESC` | Sort results |

**Pitfalls:**
- Multi-word status/field values must be quoted: `status = "In Progress"`, not `status = In Progress`.
- `maxResults` defaults low. Set explicitly for bulk queries.
- `text ~` is full-text search (fuzzy). Use `summary ~` or `description ~` for field-specific search.

### 2. Create Issue

```
composio_execute action="JIRA_CREATE_ISSUE" params={
  "project_key": "PROJ",
  "summary": "Fix login timeout on mobile",
  "issue_type": "Bug",
  "description": "Users on iOS report timeout after 30s...",
  "priority": "High",
  "assignee_id": "5f1234abc",
  "labels": ["mobile", "auth"]
}
```

**Pitfalls:**
- `project_key` is the short key (e.g., `PROJ`), not the full project name.
- `issue_type` must match exactly: `Bug`, `Story`, `Task`, `Epic`, `Sub-task`. Case-sensitive.
- `assignee_id` is the Jira account ID (not username). Resolve via `JIRA_FIND_USERS`.
- `description` uses Atlassian Document Format (ADF) for rich text, or plain string for simple text.

### 3. Edit / Update Issue

```
composio_execute action="JIRA_UPDATE_ISSUE" params={
  "issue_id_or_key": "PROJ-123",
  "summary": "Updated summary",
  "status": "In Review"
}
```

**Transition (status change):**
```
composio_execute action="JIRA_TRANSITION_ISSUE" params={
  "issue_id_or_key": "PROJ-123",
  "transition_id": "31"
}
```

**Pitfalls:**
- Status changes require **transitions**, not direct field updates. Use `JIRA_GET_ISSUE_TRANSITIONS` to get valid transition IDs first.
- `JIRA_UPDATE_ISSUE` works for fields like summary, description, priority, labels, assignee — but NOT status.
- Some fields are read-only or require specific permissions.

### 4. Sprint Management

```
// Get active sprints for a board
composio_execute action="JIRA_GET_SPRINTS" params={
  "board_id": "42"
}

// Move issue to sprint
composio_execute action="JIRA_MOVE_ISSUE_TO_SPRINT" params={
  "sprint_id": "128",
  "issue_keys": ["PROJ-123", "PROJ-124"]
}
```

**Pitfall:** `board_id` is required — get it from `JIRA_GET_ALL_BOARDS`. Sprint operations are board-scoped.

### 5. Comments

```
composio_execute action="JIRA_ADD_COMMENT" params={
  "issue_id_or_key": "PROJ-123",
  "body": "Deployed fix to staging. Ready for QA."
}
```

## ID Resolution Pattern

Many Jira actions need IDs rather than names:

1. **User name to account ID:** `JIRA_FIND_USERS` with `query` param → use `accountId`
2. **Transition name to ID:** `JIRA_GET_ISSUE_TRANSITIONS` for an issue → match by `name` → use `id`
3. **Board name to ID:** `JIRA_GET_ALL_BOARDS` → match by `name` → use `id`
4. **Sprint name to ID:** `JIRA_GET_SPRINTS` with board_id → match by `name` → use `id`

## Quick Reference

| Action | Key Params |
|--------|-----------|
| `JIRA_SEARCH_ISSUES` | jql, maxResults? |
| `JIRA_CREATE_ISSUE` | project_key, summary, issue_type, description? |
| `JIRA_UPDATE_ISSUE` | issue_id_or_key, (fields to update) |
| `JIRA_TRANSITION_ISSUE` | issue_id_or_key, transition_id |
| `JIRA_GET_ISSUE_TRANSITIONS` | issue_id_or_key |
| `JIRA_GET_ISSUE` | issue_id_or_key |
| `JIRA_ADD_COMMENT` | issue_id_or_key, body |
| `JIRA_FIND_USERS` | query |
| `JIRA_GET_ALL_BOARDS` | — |
| `JIRA_GET_SPRINTS` | board_id |
| `JIRA_MOVE_ISSUE_TO_SPRINT` | sprint_id, issue_keys |
| `JIRA_ASSIGN_ISSUE` | issue_id_or_key, assignee_id |

## Recipe Integration

- **Read inputs:** `scratchpad_read` to get issue keys, project info from upstream steps.
- **Export results:** `scratchpad_write` created issue keys, search results for downstream steps.
