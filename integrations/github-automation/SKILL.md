---
name: github-automation
description: Commit, branch, push, and manage code in GitHub repos — both via local workspace git and direct GitHub API (Composio)
version: "1.1.0"
tags: [github, git, version-control, code, commit, push, branch]
category: productivity
tools:
  - name: workspace_git
    description: Run git operations (status, diff, add, commit, push, pull, log, branch, checkout, stash, show, blame, fetch) inside the workspace repo. Does NOT support `remote` — use workspace_exec for that.
  - name: workspace_exec
    description: Shell exec inside the repo dir — fallback for git ops not in workspace_git's enum (e.g., `git remote -v`, `git config`)
  - name: workspace_list_dir
    description: List directory contents — used to discover the cloned repo name under `repos/`
  - name: workspace_read_file
    description: Read files in the cloned repo
  - name: workspace_write_file
    description: Edit files in the cloned repo before committing
  - name: composio_execute
    description: Execute Composio GITHUB_* actions (for direct API operations — PRs, issues, remote commits)
---

# GitHub Automation Skill

Use this skill when the user wants to make code changes and put them into GitHub. Users often say it in plain English — **translate their intent to the right action**:

| User says                                                   | They mean                                      | Path to use           |
|-------------------------------------------------------------|------------------------------------------------|-----------------------|
| "commit this", "save this change", "commit the code"        | git add + git commit                           | `workspace_git`       |
| "push", "push the changes", "upload my code", "send to GitHub" | git push                                    | `workspace_git`       |
| "commit and push"                                           | add + commit + push (in sequence)              | `workspace_git`       |
| "pull", "get latest", "sync"                                | git pull                                       | `workspace_git`       |
| "new branch", "create a branch", "switch branch"            | git checkout -b / git checkout                 | `workspace_git`       |
| "open a PR", "raise a pull request"                         | GitHub pull request                            | `composio_execute`    |
| "create an issue"                                           | GitHub issue                                   | `composio_execute`    |
| "list my repos"                                             | list user's repositories                       | `composio_execute`    |
| "clone that repo"                                           | clone via workspace                            | (handled by Workspace Explorer — tell the user to use it) |

## Prerequisites

- **GitHub tool must be installed and connected** in Settings → Tools & Accounts. Without it, Composio actions return `ConnectedAccountNotFound` (code 1810).
- **For local git operations**, a repo must already be cloned into the workspace. Clones land under `repos/<repo-name>/`. If no repo is cloned, tell the user to open Workspace → Explorer → **Clone Repo**.
- **Git identity** is configured automatically via `.gitconfig` injected into the workspace root when the repo is cloned. If a `commit` fails with *"Please tell me who you are"*, the credentials weren't injected — tell the user to reconnect their GitHub integration rather than setting `user.name`/`user.email` yourself.
- **Composio action names**: this skill's action names (`GITHUB_CREATE_A_PULL_REQUEST`, etc.) match the Composio toolkit at time of writing. If you get `400 Invalid action` from Composio, the action name may have changed — surface the error to the user instead of guessing alternates.

## Two Paths — Pick The Right One

### Path A — Local repo (default for "commit / push / branch")

When a repo is cloned into the workspace, work against the **local copy** using `workspace_git`. This mirrors what a human developer does on their laptop. `workspace_git` auto-detects the repo dir (`repos/<name>/`) — you do not need to pass `cwd`.

### Path B — Direct GitHub API (for one-off remote ops)

When there is no local clone, or the user wants a pull request / issue / branch created remotely, use `composio_execute` with the appropriate `GITHUB_*` action.

---

## Path A — Local Git Workflows

### 1. Check what's changed (always do this first)

```
workspace_git operation="status"
```

Shows modified, untracked, and staged files. **Always run this before committing** so you know what you are about to commit.

### 2. Edit files

Use `workspace_write_file` with a path relative to the repo root. The executor auto-prefixes with the repo dir.

```
workspace_write_file path="src/index.js" content="..."
```

### 3. Stage changes

```
workspace_git operation="add" args="."            # stage everything
workspace_git operation="add" args="src/index.js" # stage one file
```

### 4. Commit

```
workspace_git operation="commit" args="-m \"Add rate limiting middleware\""
```

**Commit message rules:**
- Start with a verb: `Add`, `Fix`, `Update`, `Remove`, `Refactor`.
- One-line summary, ≤ 72 chars.
- If the user didn't give a message, infer one from the changes — don't ask unless it's ambiguous.

### 5. Push

```
workspace_git operation="push"
```

If push is rejected with "non-fast-forward", pull first:

```
workspace_git operation="pull" args="--rebase"
workspace_git operation="push"
```

### 6. Branching

```
workspace_git operation="checkout" args="-b feat/new-feature"  # create and switch
workspace_git operation="checkout" args="main"                 # switch existing
workspace_git operation="branch"                               # list branches
```

### 7. The "commit and push" sequence (most common user ask)

```
1. workspace_git operation="status"                    # see what's changed
2. workspace_git operation="add" args="."              # stage all changes
3. workspace_git operation="commit" args="-m \"...\""  # commit with message
4. workspace_git operation="push"                      # push to remote
```

Run these **in order**, checking each result. If a step fails, stop and report the error — do not continue.

### 8. Viewing diffs (before committing)

```
workspace_git operation="diff"                     # unstaged changes
workspace_git operation="diff" args="--staged"     # staged changes
```

---

## Path B — Direct GitHub API (Composio)

Use these when there is no local clone or for remote-only operations. All require `owner` and `repo` (e.g. `"AutomatosAI"` and `"automatos-ai"`).

### Create or Update a File (single-file commit)

```
composio_execute app_name="GITHUB" action="GITHUB_CREATE_OR_UPDATE_FILE_CONTENTS" params={
  "owner": "AutomatosAI",
  "repo":  "automatos-ai",
  "path":  "docs/README.md",
  "message": "Update README",
  "content": "<base64-encoded file content>",
  "branch":  "main",
  "sha":     "<existing file sha — REQUIRED for updates, omit for create>"
}
```

**Pitfalls:**
- `content` **must be base64-encoded**. Plain text will fail.
- For *updates*, you must first fetch the file's current `sha`:
  ```
  composio_execute app_name="GITHUB" action="GITHUB_GET_REPOSITORY_CONTENT" params={
    "owner": "...", "repo": "...", "path": "...", "ref": "main"
  }
  # → response includes "sha": "abc123..." — pass that as sha in the next call
  ```
- For *creates*, omit `sha` entirely (don't pass empty string).

### Create a Branch

```
# 1. Get the SHA of the source branch tip
composio_execute app_name="GITHUB" action="GITHUB_GET_A_REFERENCE" params={
  "owner": "...", "repo": "...", "ref": "heads/main"
}

# 2. Create the new ref from that SHA
composio_execute app_name="GITHUB" action="GITHUB_CREATE_A_REFERENCE" params={
  "owner": "...", "repo": "...",
  "ref":   "refs/heads/feat/new-feature",
  "sha":   "<sha from step 1>"
}
```

### Create a Pull Request

```
composio_execute app_name="GITHUB" action="GITHUB_CREATE_A_PULL_REQUEST" params={
  "owner": "AutomatosAI",
  "repo":  "automatos-ai",
  "title": "Add rate limiting",
  "head":  "feat/rate-limiting",   # source branch
  "base":  "main",                 # target branch
  "body":  "## Summary\n...\n## Test plan\n- [ ] ..."
}
```

### Create an Issue

```
composio_execute app_name="GITHUB" action="GITHUB_CREATE_AN_ISSUE" params={
  "owner": "...", "repo": "...",
  "title": "Bug: login returns 500 on empty password",
  "body":  "Steps to reproduce:..."
}
```

### List User's Repos

```
composio_execute app_name="GITHUB" action="GITHUB_LIST_REPOSITORIES_FOR_THE_AUTHENTICATED_USER" params={
  "per_page": 30, "page": 1
}
```

---

## Resolving Owner + Repo

If the user says "commit to my repo" without naming it:

1. **Prefer local clone context.** If a repo is cloned in the workspace:
   - Use `workspace_list_dir path="repos"` to see the repo directory name.
   - If you need the GitHub `owner/repo` string (e.g., for a Composio PR action), run `workspace_exec command="git remote -v"` and parse the origin URL. **`workspace_git` does not support the `remote` operation** — use `workspace_exec` for it.
   - For Path A (local git ops), you do not need owner/repo at all — `workspace_git` auto-detects the repo dir.
2. **Otherwise ask** — "Which repo? I can see these in your account: …" and list repos from `GITHUB_LIST_REPOSITORIES_FOR_THE_AUTHENTICATED_USER`.
3. **Don't guess** owner/repo — Composio returns `400 Invalid request data` for wrong values, and silent failures waste tokens.

### Multiple repos cloned

If `repos/` contains more than one directory, `workspace_git` will silently pick the first one (alphabetical). To target a specific repo:
- Use `workspace_exec` with an explicit `cwd`: `workspace_exec command="git status" cwd="repos/<repo-name>"`.
- Or check first: `workspace_list_dir path="repos"` → confirm with the user which repo they meant.

## Quick Reference

| Intent                      | Tool / Action                              | Key Params / Args                          |
|-----------------------------|--------------------------------------------|--------------------------------------------|
| Check changes               | `workspace_git` op=status                  | —                                          |
| Stage files                 | `workspace_git` op=add                     | args="." or args="<path>"                  |
| Commit                      | `workspace_git` op=commit                  | args="-m \"message\""                      |
| Push                        | `workspace_git` op=push                    | —                                          |
| Pull                        | `workspace_git` op=pull                    | args="--rebase" (recommended)              |
| New branch                  | `workspace_git` op=checkout                | args="-b <branch-name>"                    |
| Switch branch               | `workspace_git` op=checkout                | args="<branch-name>"                       |
| View diff                   | `workspace_git` op=diff                    | args="--staged" for staged only            |
| List repos                  | `GITHUB_LIST_REPOSITORIES_FOR_THE_AUTHENTICATED_USER` | per_page, page             |
| Get file (for sha)          | `GITHUB_GET_REPOSITORY_CONTENT`            | owner, repo, path, ref                     |
| Remote commit (one file)    | `GITHUB_CREATE_OR_UPDATE_FILE_CONTENTS`    | owner, repo, path, message, content(b64), branch, sha? |
| Get branch ref              | `GITHUB_GET_A_REFERENCE`                   | owner, repo, ref="heads/main"              |
| Create branch ref           | `GITHUB_CREATE_A_REFERENCE`                | owner, repo, ref, sha                      |
| Open PR                     | `GITHUB_CREATE_A_PULL_REQUEST`             | owner, repo, title, head, base, body       |
| Create issue                | `GITHUB_CREATE_AN_ISSUE`                   | owner, repo, title, body                   |

## What NOT To Do

- **Don't call a GitHub action with empty `params`** — always supply at least `owner`, `repo`, and the action-specific fields. Composio returns `400 Invalid request data provided` for empty/partial params.
- **Don't call remote Composio actions when a local clone exists** — use `workspace_git`. Faster, cheaper, and matches how the user actually works.
- **Don't skip `git status`** before committing. You need to see what's about to land.
- **Don't commit without a message** — always pass `args="-m \"...\""`. Empty commits get rejected.
- **Don't `git add .` in a repo you haven't inspected** — check `status` first so you don't stage secrets or junk.
- **Don't force push** (`push --force`) unless the user explicitly asks. It rewrites history and can destroy teammates' work.
- **Don't invent owner/repo** — resolve from remote origin or ask.
- **Don't skip base64 encoding** on `GITHUB_CREATE_OR_UPDATE_FILE_CONTENTS` — the raw string will fail.
