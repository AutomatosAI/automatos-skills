# Automatos Skills

Agent skills for the [Automatos AI Platform](https://github.com/AutomatosAI/automatos-ai). Each skill is a `SKILL.md` file — instructions, process, formats, and guardrails — that gets injected into an agent's system prompt, teaching it **how** to do a job without the recipe prompt micromanaging every tool call.

> **128 skills indexed.** Physically organised into 16 top-level group directories. Signature Automatos agents (SENTINEL, SCOUT, HARPER, ECHO, ATLAS, FORGE, ORACLE, RALLY, PATCHER) live under [`team/`](./team/).

---

## At a glance

| Domain | Dir | Count |
|---|---|---|
| [The Automatos Team](#the-automatos-team) — signature named agents | [`team/`](./team/) | 9 |
| [Shopify & Commerce](#shopify--commerce) | [`shopify/`](./shopify/) | 21 |
| [Software Engineering](#software-engineering) | [`engineering/`](./engineering/) | 11 |
| [Quality, Security & Compliance](#quality-security--compliance) | [`quality/`](./quality/) | 9 |
| [DevOps, SRE & Infrastructure](#devops-sre--infrastructure) | [`devops/`](./devops/) | 6 |
| [Product & Project Management](#product--project-management) | [`product/`](./product/) | 8 |
| [Design & UX](#design--ux) | [`design/`](./design/) | 11 |
| [Content & Editorial](#content--editorial) | [`content/`](./content/) | 7 |
| [Social Media](#social-media) | [`social/`](./social/) | 12 |
| [Marketing & Growth](#marketing--growth) | [`marketing/`](./marketing/) | 7 |
| [Sales & Revenue](#sales--revenue) | [`sales/`](./sales/) | 8 |
| [Analytics & BI](#analytics--bi) | [`analytics/`](./analytics/) | 5 |
| [People & Operations](#people--operations) | [`people-ops/`](./people-ops/) | 3 |
| [Research & Knowledge](#research--knowledge) | [`research/`](./research/) | 2 |
| [Support](#support) | [`support/`](./support/) | 1 |
| [Integrations (Composio)](#integrations-composio) | [`integrations/`](./integrations/) | 8 |
| **Total** | | **128** |

Cross-references (e.g. `scout` shows up under both Team and Sales; `platform-management` shows up under both Research and Platform/Meta) are listed once in their home directory and linked from the other section.

---

## The Automatos Team

The signature named agents — each one has a persona, a beat, and a home in the platform UI. All live under [`team/`](./team/).

| Agent | Dir | What they do |
|---|---|---|
| **SENTINEL** | [`team/sentinel/`](./team/sentinel/) | Infrastructure watchdog — API health, error spikes, deploy status, cost anomalies. Heartbeat every 15 min. |
| **PATCHER** | [`team/bug-fixer/`](./team/bug-fixer/) | End-to-end bug fixing — read ticket → find code → write failing test → minimal fix → verify → commit → draft PR. |
| **SCOUT** | [`team/scout/`](./team/scout/) | Lead intelligence — qualify signups, score prospects, draft outreach, track pipeline. |
| **HARPER** | [`team/harper/`](./team/harper/) | Content machine — turns platform activity into LinkedIn posts, changelogs, social content. |
| **ECHO** | [`team/echo/`](./team/echo/) | Customer support — classify messages, auto-reply FAQs, create bug tickets, flag churn signals. |
| **ATLAS** | [`team/atlas/`](./team/atlas/) | Business intelligence — weekly cost/revenue, usage trends, anomaly detection, optimisation recs. |
| **FORGE** | [`team/forge/`](./team/forge/) | Recipe builder — natural language → multi-step workflows with agent assignments and triggers. |
| **ORACLE** | [`team/oracle/`](./team/oracle/) | Knowledge curator — RAG freshness, reprocess failures, Drive sync, prune stale content. |
| **RALLY** | [`team/rally/`](./team/rally/) | Community growth — map communities, identify creators, score outreach, grow ecosystem authentically. |

---

## Shopify & Commerce

Everything for the Automatos ↔ Shopify product surface. See also: [`automatos-shopify/docs/PRDS/`](../automatos-shopify/docs/PRDS/) for the build roadmap.

### Storefront-facing widgets
- [`shopify/shopify-support/`](./shopify/shopify-support/) — answers shopper questions, looks up orders, explains returns, escalates to humans
- [`shopify/shopify-product-expert/`](./shopify/shopify-product-expert/) — answers technical product questions on PDP, cites reviews, compares alternatives
- [`shopify/shopify-merchandiser/`](./shopify/shopify-merchandiser/) — personal shopping assistant, conversational recommendations, cross-sells
- [`shopify/shopify-gift-concierge/`](./shopify/shopify-gift-concierge/) — 4-question gift quiz that finds the right product from the catalog
- [`shopify/shopify-review-analyst/`](./shopify/shopify-review-analyst/) — summarises reviews with pros, cons, sentiment, quality signals
- [`shopify/shopify-seo-content/`](./shopify/shopify-seo-content/) — long-tail blog posts, meta tags, publishing cadence

### Admin-facing agents
- [`shopify/shopify-business-analyst/`](./shopify/shopify-business-analyst/) — daily morning briefs: sales, traffic, top products, inventory risks, 3 actions
- [`shopify/shopify-ops-manager/`](./shopify/shopify-ops-manager/) — day-to-day store ops — inventory, orders, customers, pricing, vendor coordination
- [`shopify/shopify-inventory-watchdog/`](./shopify/shopify-inventory-watchdog/) — scans stock daily, flags stockout risk, reorder recs
- [`shopify/shopify-inventory-management/`](./shopify/shopify-inventory-management/) — reorder points, dead-stock analysis, replenishment recs
- [`shopify/shopify-order-triage/`](./shopify/shopify-order-triage/) — fulfillment SLA monitoring, delayed shipments, fraud signals, returns workflow
- [`shopify/shopify-pricing-strategy/`](./shopify/shopify-pricing-strategy/) — margin calc, markdown recommendations, bundle pricing, rules
- [`shopify/shopify-returns-handling/`](./shopify/shopify-returns-handling/) — policy decision tree, rate tracking, quality flagging
- [`shopify/shopify-customer-retention/`](./shopify/shopify-customer-retention/) — lifecycle segmentation, CLV, churn detection, re-engagement playbooks
- [`shopify/shopify-peak-season/`](./shopify/shopify-peak-season/) — Black Friday prep, real-time monitoring, post-peak analysis
- [`shopify/shopify-supplier-management/`](./shopify/shopify-supplier-management/) — lead times, POs, SLA tracking, cost optimisation
- [`shopify/shopify-seo-ecommerce/`](./shopify/shopify-seo-ecommerce/) — product/collection SEO, meta tags, Schema.org, content plans

### Build tools
- [`shopify/shopify-app-architect/`](./shopify/shopify-app-architect/) — Shopify Functions, Partner API, payment apps, OAuth, GraphQL
- [`shopify/shopify-storefront-dev/`](./shopify/shopify-storefront-dev/) — Liquid themes, Storefront API, Hydrogen
- [`shopify/shopify-extension-builder/`](./shopify/shopify-extension-builder/) — Admin, Checkout, Customer Account, POS extensions with Polaris

### Commerce (non-Shopify-specific)
- [`shopify/cross-border-ecommerce/`](./shopify/cross-border-ecommerce/) — localisation, multi-market strategy, international sales optimisation

---

## Software Engineering

- [`engineering/backend-architect/`](./engineering/backend-architect/) — backend systems, APIs, data models, scalability reviews
- [`engineering/frontend-developer/`](./engineering/frontend-developer/) — frontend components, UI bugs, client-side performance
- [`engineering/mobile-app-builder/`](./engineering/mobile-app-builder/) — iOS + Android build/test/deploy
- [`engineering/ai-engineer/`](./engineering/ai-engineer/) — AI/ML pipelines, prompt chains, model integrations
- [`engineering/data-engineer/`](./engineering/data-engineer/) — data pipelines, quality validation, ETL
- [`engineering/database-optimizer/`](./engineering/database-optimizer/) — PostgreSQL query tuning, indexes, schema review
- [`engineering/embedded-firmware-engineer/`](./engineering/embedded-firmware-engineer/) — C/C++ for microcontrollers
- [`engineering/senior-developer/`](./engineering/senior-developer/) — production-quality feature delivery with tests + docs
- [`engineering/software-architect/`](./engineering/software-architect/) — system architecture, trade-offs, ADRs
- [`engineering/solidity-smart-contract-engineer/`](./engineering/solidity-smart-contract-engineer/) — Solidity review, authoring, auditing
- [`engineering/rapid-prototyper/`](./engineering/rapid-prototyper/) — fast MVPs to validate ideas

---

## Quality, Security & Compliance

- [`quality/code-reviewer/`](./quality/code-reviewer/) — correctness, security, maintainability review
- [`quality/api-tester/`](./quality/api-tester/) — endpoint validation, contract checks, failure reports
- [`quality/qa-engineer/`](./quality/qa-engineer/) — test runs, P0–P3 classification, log correlation, JSON reports
- [`quality/performance-benchmarker/`](./quality/performance-benchmarker/) — response times, baseline comparisons
- [`quality/accessibility-auditor/`](./quality/accessibility-auditor/) — WCAG violations, ARIA, keyboard traps
- [`quality/security-engineer/`](./quality/security-engineer/) — code audits, auth flow review, secure practices
- [`quality/threat-detection-engineer/`](./quality/threat-detection-engineer/) — threat hunting, detection rules, log coverage audits
- [`quality/compliance-auditor/`](./quality/compliance-auditor/) — policy violations, license issues, regulatory gaps
- [`quality/legal-compliance-checker/`](./quality/legal-compliance-checker/) — legal/regulatory compliance scans

---

## DevOps, SRE & Infrastructure

- [`devops/devops-automator/`](./devops/devops-automator/) — CI/CD pipelines, infra configs, deployment workflows
- [`devops/sre/`](./devops/sre/) — SLOs, reliability, incident investigation, toil reduction
- [`devops/infrastructure-maintainer/`](./devops/infrastructure-maintainer/) — infra health, config checks, maintenance tasks
- [`devops/incident-response-commander/`](./devops/incident-response-commander/) — production incident triage, diagnosis, post-mortems
- [`devops/git-workflow-master/`](./devops/git-workflow-master/) — branching, conflict resolution, release automation
- [`devops/jira-workflow-steward/`](./devops/jira-workflow-steward/) — Jira-linked commits, structured PRs, release-safe branches
- See also: [`team/sentinel/`](./team/sentinel/), [`quality/threat-detection-engineer/`](./quality/threat-detection-engineer/)

---

## Product & Project Management

- [`product/manager/`](./product/manager/) — strategic PM: roadmap, stakeholders, outcome delivery
- [`product/project-manager-senior/`](./product/project-manager-senior/) — spec → task breakdown with dependencies, scope, risk
- [`product/project-shepherd/`](./product/project-shepherd/) — cross-functional tracking, blockers, stakeholder alignment
- [`product/sprint-prioritizer/`](./product/sprint-prioritizer/) — impact/effort/urgency scoring for sprint plans
- [`product/studio-operations/`](./product/studio-operations/) — daily efficiency, process health, tool adoption
- [`product/studio-producer/`](./product/studio-producer/) — multi-project portfolios, resources, timelines
- [`product/feedback-synthesizer/`](./product/feedback-synthesizer/) — conversations + memory → prioritised product insights
- [`product/experiment-tracker/`](./product/experiment-tracker/) — hypotheses, results, data-driven recommendations

---

## Design & UX

- [`design/ui-designer/`](./design/ui-designer/) — UI markup/component review, component specs, visual consistency
- [`design/ux-architect/`](./design/ux-architect/) — information architecture, user flows, navigation
- [`design/ux-researcher/`](./design/ux-researcher/) — research plans, feedback synthesis, findings reports
- [`design/automatos-design/`](./design/automatos-design/) — Automatos brand identity reference (colours, type, style)
- [`design/brand-guardian/`](./design/brand-guardian/) — content + UI brand consistency audit
- [`design/whimsy-injector/`](./design/whimsy-injector/) — personality, micro-interactions, empty states, playful copy
- [`design/visual-storyteller/`](./design/visual-storyteller/) — visual narratives for presentations, dashboards, marketing
- [`design/inclusive-visuals-specialist/`](./design/inclusive-visuals-specialist/) — representation, accessibility, cultural sensitivity
- [`design/image-prompt-engineer/`](./design/image-prompt-engineer/) — AI image prompts, libraries, style guides
- [`design/carousel-design-system/`](./design/carousel-design-system/) — Instagram carousel layout architecture (1080×1350)
- [`design/carousel-growth-engine/`](./design/carousel-growth-engine/) — multi-slide social post design for engagement

---

## Content & Editorial

- [`content/content-creator/`](./content/content-creator/) — editorial calendars, brand-voice drafting, publishing
- [`content/technical-writer/`](./content/technical-writer/) — developer docs, API refs, onboarding guides
- [`content/book-co-author/`](./content/book-co-author/) — book planning, chapter drafting, voice consistency
- [`content/document-generator/`](./content/document-generator/) — structured reports, proposals, templates as workspace files
- [`content/executive-summary-generator/`](./content/executive-summary-generator/) — agent reports + platform data → exec summaries
- [`content/seo-specialist/`](./content/seo-specialist/) — site structure audits, on-page optimisation, ranking tracking
- [`content/ai-citation-strategist/`](./content/ai-citation-strategist/) — optimise content to be cited by LLMs and AI search

---

## Social Media

### Channel specialists
- [`social/instagram-curator/`](./social/instagram-curator/) — visual grid planning, captions, cohesive posting
- [`social/linkedin-content-creator/`](./social/linkedin-content-creator/) — professional posts, engagement, thought leadership
- [`social/tiktok-strategist/`](./social/tiktok-strategist/) — video concepts, scripts, trend tracking
- [`social/twitter-engager/`](./social/twitter-engager/) — tweets, threads, real-time engagement
- [`social/reddit-community-builder/`](./social/reddit-community-builder/) — subreddit identification, authentic posts/comments
- [`social/podcast-strategist/`](./social/podcast-strategist/) — show concepts, episode outlines, guest research
- [`social/short-video-editing-coach/`](./social/short-video-editing-coach/) — editing, scripts, format tips for Reels/Shorts/TikTok

### Cross-channel ops
- [`social/social-brand-voice/`](./social/social-brand-voice/) — tone, terminology, claims vs editorial standards
- [`social/social-media-strategist/`](./social/social-media-strategist/) — content calendars, engagement analysis, cross-platform
- [`social/social-ops/`](./social/social-ops/) — daily workflows, copy + design payload coordination, publish readiness
- [`social/social-production-workflow/`](./social/social-production-workflow/) — intake → QA → design → approval → publish
- [`social/social-template-payloads/`](./social/social-template-payloads/) — structured, field-mapped content for Canva/HTML templates

---

## Marketing & Growth

- [`marketing/growth-hacker/`](./marketing/growth-hacker/) — experiments, metrics, conversion funnel optimisation
- [`marketing/app-store-optimizer/`](./marketing/app-store-optimizer/) — ASO: listing audits, keyword research, optimised copy
- [`marketing/trend-researcher/`](./marketing/trend-researcher/) — market + competitive intelligence reports
- [`marketing/ppc-strategist/`](./marketing/ppc-strategist/) — paid media audits, budget optimisation, performance reporting
- [`marketing/creative-strategist/`](./marketing/creative-strategist/) — ad copy, testing frameworks, creative performance
- [`marketing/auditor/`](./marketing/auditor/) — paid-media audit: account structure, tracking, bidding, waste
- [`marketing/behavioral-nudge-engine/`](./marketing/behavioral-nudge-engine/) — contextual nudges to reduce friction + boost engagement

---

## Sales & Revenue

- [`sales/engineer/`](./sales/engineer/) — pre-sales technical specialist: demos, objections, PoC scoping
- [`sales/deal-strategist/`](./sales/deal-strategist/) — MEDDPICC qualification, competitive positioning, win plans
- [`sales/discovery-coach/`](./sales/discovery-coach/) — discovery-call transcript review, rep coaching
- [`sales/outbound-strategist/`](./sales/outbound-strategist/) — signal-based outbound, personalised multi-channel sequences
- [`sales/proposal-strategist/`](./sales/proposal-strategist/) — RFP / proposal architecture with win themes
- [`sales/account-strategist/`](./sales/account-strategist/) — post-sale expansion mapping, retention risk
- [`sales/pipeline-analyst/`](./sales/pipeline-analyst/) — pipeline health, forecast accuracy, at-risk deals
- [`sales/coach/`](./sales/coach/) — rep performance review, call pattern analysis, development plans
- See also: [`team/scout/`](./team/scout/) — lead intelligence & qualification (Automatos Team)

---

## Analytics & BI

- [`analytics/analytics-reporter/`](./analytics/analytics-reporter/) — workspace analytics from platform metrics + costs + usage
- [`analytics/finance-tracker/`](./analytics/finance-tracker/) — LLM spend + platform cost monitoring, budget alerts
- [`analytics/autonomous-optimization-architect/`](./analytics/autonomous-optimization-architect/) — perf benchmarking, config tuning, cost savings
- [`analytics/ai-data-remediation-engineer/`](./analytics/ai-data-remediation-engineer/) — data quality anomaly detection + automated fixes
- [`analytics/google-analytics/`](./analytics/google-analytics/) — GA4 manager: live traffic, reports, conversions, audiences
- See also: [`team/atlas/`](./team/atlas/) — BI weekly reporting (Automatos Team)

---

## People & Operations

- [`people-ops/recruitment-specialist/`](./people-ops/recruitment-specialist/) — job postings, candidate screening, hiring pipeline
- [`people-ops/supply-chain-strategist/`](./people-ops/supply-chain-strategist/) — procurement, vendors, inventory, logistics
- [`people-ops/developer-advocate/`](./people-ops/developer-advocate/) — technical content, community triage, DevEx tracking

---

## Research & Knowledge

- [`research/web-research/`](./research/web-research/) — Composio Search-powered web research + synthesis
- [`research/platform-management/`](./research/platform-management/) — the full kitchen-sink platform ops skill (marketplace, agents, playbooks, heartbeats, board, scheduling, governance, LLMs, workspace, command centre)
- See also: [`team/oracle/`](./team/oracle/) — knowledge curator / RAG health (Automatos Team)

---

## Support

- [`support/support-responder/`](./support/support-responder/) — first-line support, past-conversation search, KB lookup
- See also: [`team/echo/`](./team/echo/) — customer support & classification (Automatos Team)

---

## Integrations (Composio)

Thin skills — each teaches the agent how to use one external Composio-backed toolkit. Composable into any agent role.

- [`integrations/gmail-automation/`](./integrations/gmail-automation/) — send, search, label, draft Gmail
- [`integrations/slack-automation/`](./integrations/slack-automation/) — messages, search, threads, channels, reactions
- [`integrations/jira-automation/`](./integrations/jira-automation/) — JQL search, create/edit issues, sprints, comments
- [`integrations/google-calendar-automation/`](./integrations/google-calendar-automation/) — events, availability, attendees, recurring
- [`integrations/linkedin-automation/`](./integrations/linkedin-automation/) — posts, article shares, image posts, comments
- [`integrations/twitter-automation/`](./integrations/twitter-automation/) — posts, search, bookmarks, replies
- [`integrations/tiktok-automation/`](./integrations/tiktok-automation/) — video/photo upload, two-phase publish workflow

### Admin-facing operational skills (Jira)
- [`integrations/jira-admin/`](./integrations/jira-admin/) — create, update, transition, comment, priority mapping, lifecycle

---

## Housekeeping

There's an empty `skills/` directory at the repo root — a stale leftover, safe to delete. Not removed yet because anything pinned to historical paths might break. Flag for cleanup in the next PR.

**Directory reorg note (2026-04-17):** All 128 skills were physically moved from the repo root into 16 top-level group directories via `git mv` (history preserved). If you have tooling, bookmarks, or the Automatos loader pointing at old flat paths (`./sentinel/SKILL.md`, etc.), update to the new nested paths (`./team/sentinel/SKILL.md`). The loader recursively walks the tree so it'll pick up skills regardless — but explicit path references in code/docs need updating.

---

## SKILL.md format (quick reference)

Each skill is `{group}/{slug}/SKILL.md` with YAML frontmatter + Markdown body. See [SKILL-GUIDE.md](./SKILL-GUIDE.md) for the full author guide including required frontmatter, tool reference, and quality bar.

> **Important:** the skill's identity comes from the `name:` field in frontmatter, **not** the directory path. The physical reorg did not change any `name:` values, so nothing needs re-registering in the Automatos marketplace — the loader upserts by `name`.

```yaml
---
name: my-skill
description: One-sentence summary of what this skill teaches
version: "1.0.0"
tags: [relevant, searchable, tags]
category: agent-role
tools:
  - name: platform_submit_report
    description: What this tool does for this agent
---

# My Skill — One-Line Identity

...instructions, process, output format, anti-patterns...
```

**Quality bar:** 60–100 lines, every line actionable, real tool calls with realistic params, Automatos-native identity (no Cursor/Qwen/etc. references).

---

## Uploading to Automatos

1. **Marketplace > Skills** in the Automatos dashboard.
2. **Upload / Import** → paste this repo's Git URL.
3. Loader recursively finds every `SKILL.md`, indexes them, upserts by `name`.
4. Assign skills to agents in **Agent Builder > Capabilities** (1–3 skills per agent is the sweet spot).

Re-importing updates in place — the loader matches on `name` + source, no duplicates.

---

## Writing a new skill

1. Pick the group directory that fits (e.g. `engineering/`, `shopify/`, `sales/`). Create a new group only if your skill genuinely doesn't fit any existing one.
2. `mkdir <group>/my-skill && touch <group>/my-skill/SKILL.md`
3. Follow [SKILL-GUIDE.md](./SKILL-GUIDE.md) — frontmatter, workflow steps with real tool calls, output format, anti-patterns.
4. Keep it under ~2000 tokens (~8 KB).
5. Reference pattern: read [`team/sentinel/SKILL.md`](./team/sentinel/SKILL.md) (monitoring pattern) + [`team/scout/SKILL.md`](./team/scout/SKILL.md) (sales/outreach pattern) before writing your own.
6. Push → re-import. Existing skills update, new ones land in the marketplace.

---

## Recipe prompt: before vs after

**Before (no skill assigned):**
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

**After (`jira-admin` skill assigned):**
```
Create Jira bug tickets in PILOT for all P0 and P1 bugs from the qa_report.
Label them "auto-test" and "nightly-run". Export ticket keys to scratchpad.
```

The skill owns the mechanical detail; the recipe owns the intent.

---

## License

Internal — AutomatosAI
