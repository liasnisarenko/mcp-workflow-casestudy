# Architecture: How the stack is wired

A more detailed walkthrough of the MCP-orchestrated multi-brand operations stack.

---

## The three layers

```
+-------------------------------------------------------------+
|                    OPERATING LAYER                           |
|  Claude (Cowork desktop) + Claude in Chrome (browser agent)  |
|  - Composes tools per task                                   |
|  - Holds the safety-rail rules                               |
|  - Produces drafts, queues, reports                          |
+-------------------------------------------------------------+
                              |
                              | MCP connectors
                              v
+-------------------------------------------------------------+
|                     DATA LAYER                               |
|  Notion (CRM, KB, content calendar)                          |
|  Asana (PM, critical-path planning)                          |
|  Cloudflare (DNS, hosting, Pages)                            |
|  Shopify (commerce)                                          |
|  Google Drive (source-of-truth files)                        |
|  Adobe Creative Cloud (assets)                               |
|  Canva (brand templates)                                     |
+-------------------------------------------------------------+
                              |
                              | publish actions
                              v
+-------------------------------------------------------------+
|                  CUSTOMER-FACING LAYER                       |
|  YouTube (channel, published videos)                         |
|  Meta (live ads, posts)                                      |
|  Gmail (sent email)                                          |
|  MailerLite (sent campaigns)                                 |
|  Eventbrite (live event listings)                            |
|  Shopify (live storefront)                                   |
|                                                              |
|  The HUMAN is the gatekeeper for everything in this layer.   |
+-------------------------------------------------------------+
```

The key boundary is between the data layer and the customer-facing layer. The AI operates freely in the data layer (read, write, transform, draft). At the customer-facing boundary, the AI hands off to the human.

---

## How "tools per task, not per venture" works in practice

A typical task: launch a new event for one of the brands. Here's the per-task composition.

| Step | Tool used | Why this tool, not another |
|------|-----------|----------------------------|
| 1. Project scaffolding | Asana | Has the 85-task critical-path template. |
| 2. Brand assets pull | Google Drive | Brand books and logos live here. |
| 3. Promo graphics | Canva + Firefly | Canva for template-driven layout, Firefly for backgrounds matching the brand palette. |
| 4. Event listing | Eventbrite (browser agent) | The only practical way to drive Eventbrite is the browser. |
| 5. Landing page | Cloudflare Pages | Subdomain hosting, single-HTML deployment, fast. |
| 6. Launch email | MailerLite | Audience is already segmented here. |
| 7. Paid social | Meta (manual publish) | The agent stages creative, the human publishes. |
| 8. Public posts | Instagram (manual publish) | Same pattern as Meta. |

The same agent walks the human through all 8 steps in a single working session. No platform-to-platform integrations - the agent is the integration.

---

## The CRM design

Notion holds the relationship graph for every venture, in one workspace, using a shared structure:

- **Contacts** (people, with multi-select tags for which ventures they touch)
- **Companies** (organizations, with reverse-links to contacts)
- **Opportunities** (active deals or prospects, linked to contacts and companies)
- **Interactions** (every meaningful touchpoint, linked to opportunities)
- **Saved views** per venture, filtering the same databases by tag

This replaced four separate spreadsheets and a half-dozen DM threads. The AI queries across the whole graph for tasks like "who in my network is well-placed to refer this opportunity" or "show me every contact I haven't touched in 90 days."

The architectural choice that makes this work: **one shared structure across ventures, filtered per-venture by tag**. Not one database per venture. The shared structure means cross-venture intelligence is cheap.

---

## How safety rails are factored

Every agent task includes a shared "rails" preamble. The preamble lives in a single document, gets pasted (or referenced) at the top of every working session. Updating it once updates the behavior across every workflow.

The current preamble covers:

1. Never send email. Drafts only.
2. Never publish to a customer-facing surface (YouTube, Meta, IG, Eventbrite, Shopify storefront). Drafts only.
3. Never connect new accounts, payment methods, or third-party integrations. Stop and ask.
4. Never delete CRM records. Mark as archived instead.
5. Never modify records held by a parallel session.
6. Never click "Allow", "Accept", "Connect", or "Agree" in any browser flow without explicit human confirmation.
7. If a field is ambiguous or missing, ask. Do not guess.

These rules are tool-agnostic. They apply equally to a browser agent on Eventbrite, an AI run on Notion, or a Cowork session generating files.

---

## What I'd do differently if I started over

Three honest notes from running this in production.

**One.** The shared safety-rail preamble should have been formalized earlier. For the first 2-3 months I was rewriting safety rules per prompt. Consolidating them into a single document I include in every task would have saved a lot of "wait, I think the agent just..." moments.

**Two.** Browser-automation agents are not yet reliable enough for unattended use on customer-facing surfaces. The 9 AM standup runs unattended because everything it touches is in the data layer. Anything that reaches the customer-facing layer is run interactively, with me watching.

**Three.** Notion's MCP behavior changes with platform updates. I budget a half-day per month to validate the workflows still work as expected. The cost is real but small relative to the leverage.
