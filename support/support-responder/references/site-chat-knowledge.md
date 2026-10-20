<!--
Source text for the knowledge document in the workspace that serves the site chat. The site
bot answers from it through search_knowledge: site chats are widget turns, and widget turns
don't render skills (automatos-ai F155), so this document and the agent's persona text
(site-chat-persona.md) are what the site bot reads, not SKILL.md. The platform does not read
this file: the document is replaced in that workspace by hand.
Facts: the public facts sheet of 19 Oct 2026. Change this file, site-chat-persona.md and
SKILL.md "Public facts" together.
-->

# Automatos: public facts for the site chat (19 October 2026)

## What is Automatos?

Automatos AI is the brand. Automatos OS is the open-source engine. Automatos Studio is the hosted app. Automatos Academy (free training) and Automatos Markets (a read-only crypto market-intelligence product) are outlets of the same family. Automatos Markets is not "the marketplace": the marketplace is the catalogue of agents, Playbooks, skills, plugins and packages inside Studio. Automatos is built in Lisbon.

## Where is Automatos based, and who backs it?

Automatos AI is built in Lisbon. It is bootstrapped. To get in touch, use the contact form at https://automatos.app/contact. We reply within two working days.

## Is Auto a person?

No. Auto is Automatos's AI assistant. The site chat is an AI, and it says so.

## Can I use Automatos today?

The local edition is free and open source (Apache 2.0), available today from GitHub: https://github.com/AutomatosAI/automatos-ai. It runs on your own keys or your Claude Code / CLI subscription.

Hosted Studio opens as a beta in November 2026. Until then, sign-up is through the waitlist: https://studio.automatos.app/#waitlist. In short: on your own machine today, hosted from November 2026. Existing beta users sign in at https://studio.automatos.app/login.

## How much does Automatos cost?

The local edition is free and open source, on your own keys or your Claude Code / CLI subscription. Hosted Studio opens as a beta in November with three plans: Own keys at $9 a month, Pro at $29 and Business at $99. That is beta pricing, not charged until billing opens. Own keys and Pro are the same platform: on Own keys the AI runs on your own keys and your provider bills you, while Pro includes monthly AI credits. Business adds seats, agents, video render minutes, voice and a larger credit allowance. New accounts get a one-time trial of 500 credits; a credit is one US cent of AI usage at our prices. During the beta, once the trial is used, the AI runs on your own Anthropic, OpenAI or OpenRouter key on every plan, and your provider bills you for it. Business is by invite during the beta. Included credits and top-ups start when billing opens. Enterprise is coming soon: contact us.

## What is in each plan?

New accounts start with a one-time 500-credit trial. A credit is one US cent of AI usage at Automatos prices. Beta pricing: not charged until billing opens. During the beta every plan runs on the trial and then on your own AI keys; monthly credits on Pro and Business start when billing opens. The local edition is free and open source (Apache 2.0), on your own keys or your Claude Code / CLI subscription.

Every plan: branded documents and templates, socials on five networks, and the marketplace's packages, Playbooks and plugins.

- **Own keys: $9 a month.** Everything in Pro, on your own Anthropic, OpenAI or OpenRouter keys. Your provider bills you for the AI.
- **Pro: $29 a month.** 5 seats, 20 agents, 60 minutes of video render a month, with monthly AI credits included, so you don't need keys of your own.
- **Business: $99 a month.** 25 seats, unlimited agents, 240 minutes of video render a month, voice, and a larger monthly credit allowance. By invite during the beta: contact Automatos.
- **Enterprise: coming soon.** Contact Automatos.

## What is the difference between Own keys and Pro?

They are the same service: Own keys is everything in Pro. On Own keys you bring your own AI keys and your provider bills you for the AI. On Pro, monthly AI credits are included.

## Are there annual plans, discounts, top-ups or refunds?

Not today. Billing is not open yet during the beta. Details are published when billing opens.

## What about Enterprise?

Enterprise is coming soon. The same OS in your own cluster, with a licence, committed credits and consultancy. We're building it now. Tell us what you need: https://automatos.app/contact.

## Do you have SOC 2, ISO 27001, SSO, SLAs or a dedicated cluster?

Not today. SOC 2, ISO 27001, SSO (SAML, SCIM, OIDC), uptime or response SLAs, dedicated clusters, a DPA or BAA, EU residency pinning or a Frankfurt region, 24/7 support, a named engineer or account manager, and a Compliance Pack are not available. A Helm chart exists and is tested in CI. If you need any of these, tell us what you need through the contact form.

## What is in the Studio marketplace?

Hosted Studio's marketplace has 100+ ready-made agents, 20+ Playbooks, 400+ skills, 70+ plugins and 3 packages: Socials, Shopify Management and Shopify Development. Among them is the open automatos-skills library of 140 skills.

## What agents do I start with?

A new hosted workspace starts with Auto and a Brand Designer. Auto builds the rest of your team during onboarding, from a package or from scratch. A fresh local install starts with five agents: Auto, a Brand Designer and a starter team of Researcher, Writer and Analyst.

## Which apps and models does Automatos connect to?

In every edition: 1,000+ apps through Composio (on the local edition, you bring your own Composio key), and 400+ models through OpenRouter, plus direct keys for OpenAI, Anthropic, Google, DeepSeek and others. Agents also have 200+ built-in platform actions.

## Which chat channels and social networks are supported?

Chat channels: Telegram, Slack, Discord and WhatsApp, plus a generic webhook. Social networks: LinkedIn, X, Instagram, TikTok and YouTube.

## Does Automatos work with Shopify?

Hosted Studio's marketplace has 12 Shopify agent templates in two packages: Shopify Management and Shopify Development. A Shopify App Store app is next; it is not available yet.

## Are there workspace templates?

Workspace templates are coming. They are not available yet.

## Does Automatos improve itself?

Automatos does not rewrite its own Playbooks or change itself unattended.

## What is Automatos doing about the EU AI Act?

EU AI Act work is in progress. Human approval before anything publishes or spends, risk classes with matching oversight, and an append-only audit log are in the product today. Article 50: In the product, always on: every chat widget shows an “AI” badge and an opening notice, every in-app chat shows “Auto is an AI” and pauses if a theme hides it, channel bots say they are an AI and for whom before their first reply, and Auto Live voice says so at the start of the call. The videos, images and audio Automatos renders carry the IPTC “AI-generated” mark and a signed C2PA manifest, and anyone can check a file on the /verify page without an account. You still add, as the deployer, the deep-fake disclosure where you publish (Art. 50(4)) and a notice for any emotion recognition or biometric categorisation you run (Art. 50(3)). More: https://automatos.app/eu-ai-act.

Dates: Article 50 transparency has applied since 2 Aug 2026. 2 Dec 2026 is the Article 50(2) marking deadline for generative systems already on the market before 2 Aug 2026. Annex III high-risk applies 2 Dec 2027 and Annex I high-risk 2 Aug 2028 (Regulation (EU) 2026/1744).

## Where is my data stored?

Documents and vectors are stored on AWS in Ireland (eu-west-1). We don't state a region for anything else, and EU residency pinning isn't available today.

## When and where can I meet Automatos?

Product Hunt launch: Tuesday 3 November 2026. Web Summit Lisbon: 10–12 November 2026, with opening night on 9 November. Automatos is at the ALPHA stand (AI/Machine Learning track).

## How do I contact the team?

Use the contact form at https://automatos.app/contact. We reply within two working days.
