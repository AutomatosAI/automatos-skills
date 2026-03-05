# Automatos Skills

Agent skills for the [Automatos AI Platform](https://github.com/AutomatosAI/automatos-ai). Each skill is a `SKILL.md` file that gets injected into an agent's system prompt, teaching it **how** to perform a specific job.

## What's a Skill?

A skill is a document — instructions, process guidance, formats, and examples. When an agent reads it, it understands how to do a task without the recipe prompt micromanaging every tool call.

**Without skills:** Recipe prompts are 50+ lines of "call this tool, format it like this, export to scratchpad with this key..."

**With skills:** Recipe prompts become simple intent — "Create Jira tickets for P0/P1 bugs from the QA report."

## Available Skills

| Skill | Directory | Description |
|-------|-----------|-------------|
| **Jira Admin** | `jira-admin/` | Create, update, transition, and comment on Jira issues. Priority mapping (P0→Highest), ticket lifecycle, data handoff between recipe steps. |
| **QA Engineer** | `qa-engineer/` | Run tests, classify failures by severity (P0–P3), correlate with server logs, produce structured JSON reports for downstream agents. |
| **Bug Fixer** | `bug-fixer/` | End-to-end bug fixing: read ticket → find code → write failing test → minimal fix → verify → commit → open draft PR. |

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
