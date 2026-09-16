# DRG-Dev — wiring the agent in Automatos (session mode)

**What this is.** An Automatos agent whose runtime is Gerard's own Claude Code session. Automatos owns the board and the tickets (PRD-234 session mode, PRD-239 parity). The paired CLI host claims a ticket, launches `claude` in the agent's workspace folder with the agent's Description + Persona + attached skill bodies as an appended system prompt, supervises the session through hooks, and records the session's final message as the ticket result.

Files here (source of truth for the text; the Automatos rows are copies):

| File | Goes where |
|---|---|
| `PERSONA.md` | Agent form → Description, and Persona → Custom |
| `SKILL.md` | A workspace skill named `drg-dev`, attached to the agent |
| `README.md` | This guide |

A copy of all three also sits in `Automatos-AI-Platform/automatos-skills/engineering/drg-dev/` (that repo is PUBLIC — see section 2 before pushing).

## 1. Prerequisites (local edition only)

- Automatos local stack with `CLI_RUNTIME_ENABLED=true` (the backend refuses it unless `AUTH_EDITION=local`).
- `LOCAL_PROJECTS_DIR` in the stack `.env` set to a PARENT of the workspace, e.g. `/Users/gkavanagh/Development`; then `make up` and `make cli-host-install` so the host allows that folder. Settings → Session mode shows the state and hands you the two lines.
- `claude` installed and logged in with your own plan (`claude login`); host paired (`make cli-host PAIR=<code>`); host version 0.7.0 or later.

## 2. Create the skill first (so it can be attached at agent creation)

Ticket sessions run with `--setting-sources user --strict-mcp-config`, so a skill in the repo's `.claude/skills/` is never loaded (PRD-239 D2). The session sees only the Automatos Skill rows attached to the agent, rendered in full.

- **UI:** Agents → Skills → create from content, paste `SKILL.md` verbatim (name `drg-dev`, category development, type technical).
- **API:** `POST /api/workspaces/{workspace_id}/skills/create` with `{"name": "drg-dev", "content": "<SKILL.md verbatim>", "skill_type": "technical", "category": "development"}`. Frontmatter `description`, `tags` and `version` are read from the file; the whole file becomes the skill body the session sees.
- **Never commit this file to `AutomatosAI/automatos-skills`.** That repo is public; this skill carries Dr Green internals.
- **Budget:** all attached skill bodies together are capped at `CLI_SESSION_SKILLS_MAX_CHARS` (default 24,000 characters). The first skill always renders (truncated if it must); later ones fall back to name + description when they do not fit. Keep `drg-dev` the first (highest-priority) attachment. It is about 16,500 characters, so one more mid-sized skill fits beside it.
- **Scanner:** uploads pass the plugin security scanner. It rejects code-execution patterns (`subprocess`, `eval(`, `os.system(`, file-write calls) and prompt-injection phrases. The current file is clean; keep it that way when editing.

## 3. Create the agent

Agents → Create:

| Field | Value |
|---|---|
| Name | `DRG-Dev` |
| Category | the engineering / development one |
| Description | `PERSONA.md` section 1 |
| Persona | Custom → `PERSONA.md` section 2 |
| Runtime section → runtime | Claude Code session (`configuration.runtime = "cli"`, `provider = "claude"`) |
| Runtime section → model | blank (the CLI's default), or an alias your login has (`fable`, `opus`, `sonnet`) |
| Runtime section → Workspace folder | `/Users/gkavanagh/Development/Dr-Green-Cannexis` |
| Runtime section → worktree per ticket | **OFF.** The folder is an umbrella; its root repo tracks almost nothing, so a worktree of it would be empty (PRD-239 S6b). Sessions then run in the folder itself and `cd` into the repo they change. |
| Skills | attach `drg-dev`, priority first |
| Team / Job title | Engineering / Senior Developer, Dr Green |

The verdict line under the Workspace folder field must read as browsable and allowed; if it says the folder is outside the host's roots, fix `LOCAL_PROJECTS_DIR` (section 1).

### Bash allowlist (no form field — set once through the agents API)

The host judges every shell command per segment against a prefix allowlist: the built-in defaults (`git status/diff/log/show/branch/add/commit/stash/restore`, `git checkout -b`, `git switch -c`, `ls`, `cat`, `head`, `tail`, `grep`, `rg`, `find`, `pwd`, and the usual test/build verbs, which the skill forbids by rule) plus whatever `configuration.allowed_tools` adds. `git push`, remote edits, `gh pr create/merge/edit`, `sudo` and `rm -rf /` are never allowed; everything else not on the list is HELD in the approvals inbox until you answer. Recommended additions, sent as a shallow-merged configuration update to the agent:

```json
{"configuration": {"allowed_tools": ["cd", "git switch", "git checkout", "git fetch", "git pull --ff-only", "git ls-tree", "git worktree", "sed -n", "gh run list", "gh run view", "gh pr view", "gh pr list", "gh pr checks", "gh pr diff"]}}
```

Why `cd`: `cd dr-green-backend && git status` is judged segment by segment, so `cd` must be allowed. Do NOT add `git -C`: the allowlist matches prefixes, and `git -C <repo> push` would slip past the push guard. `aws`, `railway`, Docker and the test/build commands stay off the list on purpose; a held command is the intended stop.

Configuration keys the runtime reads: `runtime`, `provider`, `model`, `working_directory`, `worktree_per_ticket`, `allowed_tools`.

## 4. Filing a ticket

Board → new task → assign `DRG-Dev`. On the local edition, filing a ticket assigned to the agent, dragging it to In Progress, or Run Now IS your approval; the host claims it on its next poll (the ticket says "no host online" if the host is down). Auto can file one too ("have DRG-Dev …"); it writes the same four-part contract. Use the template the skill expects:

```
OBJECTIVE: <one sentence, the outcome, not the activity>
REPO: <dr-green-backend | dr-green-admin | dr-green-dapp | drg-wp-plugins | budstack-saas>
OUTPUT: <branch + commits | PRD in docs/prd | runbook | report>
CONTEXT: <PRD path · memory hook · handoff · PR numbers · log lines>
BOUNDARIES: <what not to touch · consumers that must keep working · flags that stay dark>
DEFINITION OF DONE: <checkable statements>
```

Paste memory entries the ticket depends on into CONTEXT: a ticket session can read only inside the workspace folder, and the memory directory is outside it (the index still auto-loads).

**What comes back.** The session's final message, in the skill's summary format, becomes the ticket result; the ticket lands in review (or done, per its review mode). A held or denied command during the run also parks it in review. Integration is yours: push the branch, open the PR, dispatch the deploy.

**Working with the agent live.** Pick `DRG-Dev` in the chat's agent menu: the Runtime Canvas opens its Claude Code session in the workspace folder (no worktree, no hook gate, your normal permission mode). Same persona and skill, your rules.

## 5. Maintaining

- The session prompt is rendered from the DB rows on every claim. Edit the skill content (update route or UI) or the agent's persona and the next ticket gets it. Keep these files in step; they are the text's source of truth.
- Section 3 of the skill duplicates deploy mechanics for admin, dapp and wp-plugins because those repos have no `CLAUDE.md`. Good first ticket: write those three `CLAUDE.md` files, then shrink the skill's section 3 to pointers.
- A BudStacks twin (`BDS-Dev`) is the same persona with the lane flipped plus a `bds-dev` skill. Do not widen `DRG-Dev` to own `budstack-saas`; that is the two-hats rule.
- Bump `version` in the frontmatter when the rules change; it is stored on the skill row.

## 6. Size check

Run from the workspace root: `wc -c .claude/automatos/drg-dev/SKILL.md` — keep the total of all attached skills under 24,000 characters.
