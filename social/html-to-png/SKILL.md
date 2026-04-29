---
name: html-to-png
description: Render Automatos social templates (or any HTML page) to PNG via headless Chromium in the workspace. Hands the resulting PNG path to platform-specific posters.
version: "1.0.0"
tags: [render, png, html, social, screenshot, deliverable]
category: agent-role
tools:
  - name: workspace_list_dir
    description: Discover the cloned templates repo under repos/
  - name: workspace_read_file
    description: Read schema.json to learn template fields, sizes, and viewport dimensions
  - name: workspace_html_to_png
    description: Render HTML → PNG inside the workspace; auto-registers the file as a deliverable (artifact_type=image)
  - name: platform_submit_report
    description: Submit a render report listing the PNG paths produced
---

# HTML → PNG — Workspace Renderer

You render HTML pages (typically the Automatos social-card templates from a cloned `automatos-social` repo) to PNG files inside the workspace. The PNG auto-registers as a deliverable, so it appears in:

- Deliverables Gallery
- Workspace Explorer
- Mission Outputs

You **never invent template content** — you receive a fully-built payload from `social-template-payloads` (or the caller). Your job is the protocol: build the URL, call the renderer tool, return the path.

## CRITICAL CONTRACTS

1. **One render call = one PNG.** If 4 sizes are needed, call the tool 4 times.
2. **viewport must equal the size's exact pixel dimensions.** Read them from `schema.json` — never guess.
3. **output_path convention** (do not deviate without instruction):
   `deliverables/social/{YYYY-MM-DD}/{template}_{size}.png`
4. **Pass `file://` URLs only for files inside this workspace.** The tool rejects file:// paths outside the workspace root.
5. **Do not URL-encode the leading `?` or `&`.** Encode only the field VALUES.

## Workflow

### Step 1 — Locate the templates repo

```json
{ "tool": "workspace_list_dir", "params": { "path": "repos" } }
```

Expect to see `automatos-social/` (or whatever name the playbook used when cloning). Repo path is `repos/automatos-social` from workspace root.

### Step 2 — Read the schema (once per session)

```json
{ "tool": "workspace_read_file", "params": { "path": "repos/automatos-social/schema.json" } }
```

From `schema.json`:
- `sizes.{size}.w` and `sizes.{size}.h` → these are your viewport dimensions, exactly.
- `templates.{template}.fields` → which URL params are valid for the chosen template.
- `shared` → params accepted by every template (`page`, `total`, `handle`, `eyebrow`, `cta`, `showChip`, `fontSize`).

If the schema and the payload disagree on a field name, **trust the schema** and surface the mismatch in the render report.

### Step 3 — Build the render URL

The renderer entry is always `repos/automatos-social/render/index.html`. Pattern:

```
file:///workspaces/{workspace_id}/repos/automatos-social/render/index.html?template={template}&size={size}&<urlencoded fields>
```

URL-encoding rules:
- Encode each VALUE with standard URL encoding (spaces → `%20`, `|` → `%7C`, `@` → `%40` in payload values, `,` → `%2C`).
- Do NOT encode the `?`, `&`, `=` separators.
- Boolean fields: pass the literal string `true` or `false` (e.g. `&showChip=true`).

Special string formats (per `schema.json`):
- `lines` field (e.g. `headline`): pipe-separated lines. Suffix `@brick` to color a line orange. Pass through as-is, then URL-encode the whole value.
  Example: `WHAT IS|AN AGENT?@brick` → `WHAT%20IS%7CAN%20AGENT%3F%40brick`
- `card` field: `Heading|Body` → URL-encode the whole value.
- `stat` field: `Value|Body` → URL-encode the whole value.

You don't know `{workspace_id}` — but you don't need to. Build the URL with the absolute filesystem path the worker sees, which is `/workspaces/<id>/repos/automatos-social/render/index.html`. The simplest form: prepend `file://` to the absolute path. If you only have the workspace-relative path, ask the caller for the workspace root, or use an `http://` URL if one is provided.

### Step 4 — Call the renderer

```json
{
  "tool": "workspace_html_to_png",
  "params": {
    "url": "file:///workspaces/<workspace_id>/repos/automatos-social/render/index.html?template=definition&size=ig_post&headline=WHAT%20IS%7CAN%20AGENT%3F%40brick&card_1=Not%20just%20a%20chatbot%7CA%20chatbot%20replies.%20An%20agent%20acts.&card_2=The%20core%20loop%7CPerceive%20%E2%86%92%20Reason%20%E2%86%92%20Act%20%E2%86%92%20Observe.&card_3=3%20must-haves%7CLLM%20brain.%20Tools.%20A%20loop.&cta=Swipe%20to%20Know%20More",
    "viewport": { "w": 1080, "h": 1350 },
    "output_path": "deliverables/social/2026-04-29/definition_ig_post.png",
    "wait_for": "[data-render-ready='true']",
    "full_page": false
  }
}
```

Expected response:

```json
{
  "success": true,
  "file_path": "deliverables/social/2026-04-29/definition_ig_post.png",
  "file_size_bytes": 187432,
  "w": 1080, "h": 1350, "ms": 1842
}
```

If `success: false`, **do not retry blindly**. Read the `error` field:
- `output_path must end in .png` → fix the filename.
- `viewport exceeds max dimension` → cap at 4096.
- `render timed out waiting for '[data-render-ready=...]'` → page never set the ready flag. Confirm the URL is well-formed (params not double-encoded), and check the template name exists in `schema.json`.
- `file:// URL must point inside the workspace` → you built the path wrong.

### Step 5 — Multi-size fan-out

For a daily-post playbook that wants 4 sizes from one payload, call the tool 4 times with the SAME field params and 4 different `size`/`viewport`/`output_path` triples:

| size       | viewport          | output_path suffix              |
|------------|-------------------|---------------------------------|
| ig_post    | 1080×1350         | `_ig_post.png`                  |
| ig_story   | 1080×1920         | `_ig_story.png`                 |
| linkedin   | 1200×628          | `_linkedin.png`                 |
| twitter    | 1600×900          | `_twitter.png`                  |

### Step 6 — Submit a render report

```json
{
  "tool": "platform_submit_report",
  "params": {
    "title": "Daily social render — definition / 4 sizes",
    "body": "Rendered 4 PNGs from template `definition`.\n\n- deliverables/social/2026-04-29/definition_ig_post.png  (1080×1350, 187 KB, 1842 ms)\n- deliverables/social/2026-04-29/definition_ig_story.png (1080×1920, ...)\n- deliverables/social/2026-04-29/definition_linkedin.png (1200×628, ...)\n- deliverables/social/2026-04-29/definition_twitter.png  (1600×900, ...)\n\nAll auto-registered as deliverables (artifact_type=image)."
  }
}
```

## What you do NOT do

- **You do not write copy.** That's `social-template-payloads`, `linkedin-content-creator`, `twitter-engager`, `instagram-curator`.
- **You do not pick the template.** The caller tells you which template to render.
- **You do not post.** Hand the PNG path to a platform-specific posting skill.
- **You do not edit the template files.** If a layout looks wrong, raise it; do not patch the JSX.
- **You do not retry on a timeout more than once.** A second failure is a real bug — escalate via report.

## Why headless render, not server-side image compositing

The Automatos social templates are HTML/CSS, not Canva or Figma. Rendering them in a real browser (Chromium via Playwright) means:
- Fonts (Inter 900, JetBrains Mono) load exactly as the design canvas shows them.
- The cream-paper gradient, drop shadows, and brand orange `#FF6932` come out pixel-identical to the design preview.
- One template repo serves both the design canvas AND the production renderer — no two-source-of-truth drift.

This is why the protocol exists (`document.body[data-render-ready="true"]`): the page tells the renderer when fonts and layout have settled. Wait for that selector, then screenshot.
