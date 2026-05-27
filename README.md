# MCP Workflow Case Study: Multi-Brand Operations Stack

How I run multiple concurrent brands (art advisory, interior design, event production, a podcast line) out of a single MCP-orchestrated AI stack. This is a real working setup, not a reference architecture.

The case study covers what tools are connected, how they compose per task, the unattended workflows that run every morning, and the tradeoffs I made along the way.

---

## TL;DR

**Problem.** Running 4 concurrent ventures with a 1-person operating core. Each venture has its own CRM, content calendar, ad accounts, and client comms. The traditional answer is a project manager, an executive assistant, and a marketing coordinator. None of those were in the budget.

**Approach.** Connect Claude to a curated stack of MCP-enabled platforms. Compose the tools per task, not per venture. Treat the AI as the operating layer, the platforms as the data layer.

**Outcome.** A single operating cadence across 4 brands. One morning standup that pulls from every tracker. Multi-brand content workflows that pass between design, social, and email tools without me handling files manually. A CRM architecture in Notion that replaced four separate spreadsheets and a half-dozen DM threads.

---

## The stack

| Platform | Used for | Why MCP matters here |
|----------|----------|----------------------|
| **Notion** | CRM, knowledge base, content calendar | Multi-database structures with linked records. AI queries across the whole graph at once. |
| **Asana** | Project management, critical-path planning | Multi-section projects with owner tags. AI updates statuses and surfaces blockers. |
| **Calendly** | Booking, intake flows | AI updates availability and reads intake form responses to prep call briefs. |
| **Cloudflare** | DNS, hosting, Pages | AI deploys landing pages and configures subdomains for new ventures. |
| **Shopify** | Commerce (selected ventures) | Product creation, inventory, order summaries. |
| **Adobe Creative Cloud** | Firefly, Photoshop, Premiere Pro | Image and video generation, batch retouching, social variations. |
| **Canva** | Design and brand templates | Brand-template-driven asset creation for fast social posts. |
| **YouTube** | Channel and SEO | AI authors metadata, manages channel architecture. Stop-and-confirm before publish. |
| **Meta** | Paid social and audience insights | Audit and reporting (not auto-posting). |
| **Gmail** | Outbound and intake | Drafts only. Human sends. |
| **Google Drive** | Source-of-truth files | AI reads briefs, exports decks, organizes brand-asset folders. |
| **MailerLite** | Email campaigns | Newsletter drafts and audience segmentation. |

The defining choice: **the AI never holds the trust-critical action**. It doesn't send email, doesn't publish posts, doesn't charge cards. It drafts, queues, and reports. The human action is always the last one.

---

## How the tools compose per task

The stack is wide but the per-task composition is narrow. Example workflows:

### Daily morning standup (unattended, scheduled)

**Composition:** Asana + Notion + iPhone Notes (via export) + Calendar

**What runs.** At 9 AM every morning, the agent pulls today's Asana tasks across every project, diffs against a personal-baseline Notes file (so we can tell what's new), reads calendar events, and produces a one-page brief:

- Top 5 tasks ranked by critical-path impact
- Anything new added since yesterday
- Blockers flagged (tasks waiting on someone else)
- Meetings today with the brief-doc link for each one

Sent to me as the first thing I read with morning coffee. No manual trigger.

### Event production sprint (cross-tool)

**Composition:** Asana + Eventbrite + Canva + Adobe Creative Cloud + Meta + MailerLite

**What runs.** When we launch a new event, the agent walks through:

1. Spin up an Asana project from a saved template (85-task, 9-section critical-path structure).
2. Draft the Eventbrite listing using a governance-grade prompt (see [governance-prompts repo](https://github.com/liasnisarenko/governance-prompts)).
3. Generate promotional graphics in Canva from a brand template + Firefly background variants.
4. Draft the launch email in MailerLite.
5. Stage the Meta ad creative for the human to review and publish.

Each platform contributes one slice. The AI composes the slices.

### CRM-driven outreach (sales)

**Composition:** Notion + Gmail + Calendly

**What runs.** A Notion linked-database structure (clients, leads, opportunities, contacts) holds the relationship graph. When I prep for a sales call, the agent pulls the contact's record, all linked opportunities, the latest interaction log, and the latest call notes, and produces a one-page brief. Drafts a follow-up email post-call, queued for me to send.

---

## Architecture decisions

### One AI orchestrator, many platforms

The platforms don't talk to each other directly. The AI is the integration layer. This is intentional - direct platform-to-platform integrations are brittle and expensive to maintain. AI-mediated workflows are flexible: when a platform changes its UI, the prompt adapts. When a new platform joins the stack, you write one prompt, not ten integrations.

### Tools per task, not per venture

There is no Snisarenko-Gallery stack and Snisarenko-Interiors stack. There's one stack, and the per-task composition pulls whatever tools the task needs. This keeps the cognitive load low and lets ventures share workflows. A content-launch workflow looks the same whether it's a gallery exhibition or an event company campaign - only the brand inputs change.

### Governance separated from execution

Every workflow has two layers:

- **Execution prompts** - the steps the agent takes (open Asana, pull tasks, group by project, etc.).
- **Safety rails** - what the agent must not do (auto-publish, auto-DM, auto-pay).

The safety rails are reusable across workflows. They live in a separate prompt fragment that gets included in every agent task. Centralizing them means I update them in one place when a new failure mode appears.

### The human holds the publish button, always

Every workflow ends with the agent producing a draft, queue, or report - never a published artifact. The human action (clicking Send, Publish, Post, Charge) is always the last action. This is the single highest-leverage design choice in the whole stack and the reason I trust it in live business use.

---

## What this case study is meant to show

If you're hiring an AI Ops person, this is what the work actually looks like. It's not training models. It's not writing code. It's:

- Designing the right composition of tools per task.
- Writing prompts that stay safe in live environments.
- Building unattended workflows that run reliably without supervision.
- Separating draft work (cheap, agent-owned) from publish actions (expensive, human-owned).
- Making the system maintainable for a small team or solo operator.

The artifacts in this repo are what I'd hand a new collaborator on day one to get them oriented.

---

## Files in this repo

| File | What's in it |
|------|--------------|
| [`architecture.md`](architecture.md) | A more detailed walkthrough of how the stack is wired together. |
| [`daily-standup-workflow.md`](daily-standup-workflow.md) | The full prompt and trigger setup for the 9 AM unattended standup. |

---

## License

MIT. Borrow patterns, copy structures, adapt to your stack. If you build something interesting on top, I'd love to hear about it.

---

*Maintained by [Lia Snisarenko](https://github.com/liasnisarenko). AI Workflow Operator based in Los Angeles.*
