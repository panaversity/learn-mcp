# Step 21: Capstone: A Complete MCP App for a Real Client

Build one complete, client-ready MCP app with the simplest design that meets a client's needs. Every capstone has the same core: a multi-tenant remote MCP server on Cloudflare Workers with sign-in, business data in Neon with row-level security, screens chosen from intent, security, tests, production deploys and a price. On top of that core, you add only the extras the client actually needs (company SSO, a dedicated database for an enterprise customer, search by meaning, skills, Durable Objects, Queues, Workflows, Tasks, Events, self-serve billing, a directory listing and more), and you justify each one. The capstone grades your judgment, not how many technologies you use. This time you don't extend the sales server. You start Stockroom's next product from a brief (or a product for a client of your own), which is the job you'll do as an FDE or a founder, and the result is your portfolio piece.

> 🏬 **Stockroom's ask:** Stockroom's sales assistant is selling, and its customers now want inventory and purchasing too. This time Stockroom hands you a brief instead of prompts. (You may bring a client of your own instead.)

> **Note:** Some extensions (Skills over MCP, Tasks and Events) may still be rolling out in Claude or ChatGPT. Check the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix). If you choose one of them, make sure the app still works through plain tools where a client doesn't support them yet.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.5 When is a connector worth writing?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/built-in-custom-skill-or-connector/). Deciding what the client actually needs before building.
- [20.15 How do the pieces fit together?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/how-the-pieces-fit/). Core MCP plus extensions as one connector, the common mistakes, and (in its last section) where the check of who has paid goes when the app has screens.
- [18.9 How does one connector serve many companies?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/what-a-tenant-is/). A product serves many companies, and each company's people and data stay apart.
- [19.7 Shared tables, or a database per company?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/three-ways-to-keep-data-apart/). Choosing how tenants are separated in Neon.
- [Lab 20b, a full interactive app](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/lab-review-workspace/). A worked example of a full app, end to end.
- *Optional:* [17.10 How do you see what a connector did?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/watching-a-connector/). Knowing what the app is doing once real users arrive.

**Official docs.** Keep these open while you build:

- [MCP specification (2026-07-28)](https://modelcontextprotocol.io/specification/2026-07-28). The protocol, for every design decision you make.
- [MCP extensions overview](https://modelcontextprotocol.io/extensions/overview). All the extensions your app uses, in one place.
- [Better Auth: organization plugin](https://better-auth.com/docs/plugins/organization). Companies, members and roles for a multi-tenant app.
- [Neon: branching](https://neon.com/docs/introduction/branching). A separate branch for the new product and for testing.
- [Cloudflare Workers: pricing](https://developers.cloudflare.com/workers/platform/pricing/). Your running costs, for pricing the product.
- [Neon: plans](https://neon.com/docs/introduction/plans). Database costs, for pricing the product.

## Why this step matters

Every earlier step taught one technology on the same sales server, with the prompts written for you. Clients don't arrive with prompts. They arrive with a business problem, and you decide which parts to use and how they fit. The capstone proves you can take a brief, design the product, build it with Claude Code, ship it to the AI apps the client already uses, and explain your choices. It is the piece you show to clients and employers, and it pulls together what you practised for the PCDV-F and CCDV-F exams.

## Why this technology in an MCP server: The right ones, together

- **What it is.** The capstone combines the bootcamp's core technologies with the extras a real client needs. The requirements table below marks each feature as core or extra, and maps it to the technology and step that teaches it.
- **Why an MCP server uses it.** Each technology solves a different problem for an MCP server: Workers make it reachable, resources and prompts give users data and starting points, OAuth says who is calling, Neon holds the business data, pgvector searches it by meaning, KV caches it, MCP Apps shows it, skills teach the model the job, Durable Objects keep exact per-user state, Queues and Workflows run long work, Tasks and Events give that work a standard face, security hardening, tests and environments keep it safe and reliable, and billing and directory listing make it a product.
- **What it solves.** A real client's problem, end to end, in the AI apps they already use.
- **Alternatives.** Use the simplest piece that meets each requirement, and say why in NOTES.md. An extra you add without a client need costs points, because clients pay for a product they can run, not for a tour of the bootcamp. A client on another cloud may need other hosting or databases, but the MCP design (tools, auth, widgets, skills, tasks and events) carries over unchanged.

## What you will learn

- Turn a client brief into an architecture, choosing the right Cloudflare service and MCP feature for each requirement (the rules in [AGENTS.md](../AGENTS.md))
- Build a multi-tenant MCP server from scratch, where each company's users and data stay apart
- Combine the core technologies with only the extras a client needs, into one product that works in Claude and ChatGPT
- Plan fallbacks for extensions a client doesn't support yet
- Evaluate the product with realistic prompts, and document it for a client handover
- Present the product: a short demo, an architecture diagram and a price you could charge

## What you will build

**Stockroom Inventory**: Stockroom's next product, an AI-native inventory and purchasing app for the same small distributors and retailers. Their staff run the stockroom by asking Claude or ChatGPT, and the right screen appears in the chat. You may pick another client and domain instead (for example a clinic's appointments or a contractor's jobs), as long as your product meets this table. Build every **Core** row. Then pick at least two **Extra** rows that your client needs, and for each one write in NOTES.md the need, why something simpler isn't enough, and what it costs to run. For each extra you leave out, write one line on why the client doesn't need it.

| Requirement | Core or extra | Stockroom feature | Technology (step) |
| --- | --- | --- | --- |
| Remote server, good tools | Core | Tools for products, stock levels, suppliers and purchase orders, with annotations and recoverable errors | Workers, tool design (1, 2) |
| Reference data and starter prompts | Extra | Supplier terms and the product catalog as resources, and prompts such as "weekly stock check" | Resources and prompts (3) |
| Sign-in, many companies | Core | Each company has its own staff, roles (staff, manager) and data | Better Auth + D1, organization plugin (4), per-company row-level security (6) |
| Company sign-in (SSO) | Extra | A company's staff sign in with the company's own identity provider, for a client whose customers ask for it | Better Auth SSO plugin (4) |
| Tenant isolation everywhere | Core | Shared tables with row-level security, per-company settings, tenant-scoped keys, objects, messages, runs and events, and a cross-tenant test suite | Multi-tenant architecture (7) |
| Enterprise tier | Extra | A dedicated Neon database for one enterprise customer, behind the tenant router, for a client that sells to companies who require it | Multi-tenant architecture (7) |
| Business data | Core | Products, stock movements, suppliers and purchase orders, in a schema normalized to 3NF | Relational design + Neon Postgres (5, 6) |
| Search by meaning | Extra (core if the product answers questions from documents) | Supplier terms, return rules and the product catalog searchable by meaning, citing document and line or abstaining, measured by a question set in CI | pgvector retrieval (8) |
| Fast, slightly stale data | Extra | Supplier catalog and settings cached for quick lookups | Workers KV (2 to 4, 7) |
| Screens in the chat | Core | Stock table, reorder form, purchase-order card and a KPI dashboard, chosen by the model from intent | MCP Apps + generative UI (9, 19) |
| Know-how | Extra | A `weekly-reorder-review` skill that checks low stock, proposes orders and explains them, plus a per-company playbook (reorder rules and approvers) served only to that company | Skills over MCP (10) |
| Consistent per-user state | Extra | A purchase-order draft built over several messages, and per-user rate limits | Durable Objects (11) |
| Background work | Extra | Bulk import of a supplier price list or stock count from CSV, with failed rows reported | Queues (12) |
| Multi-step job with waiting | Extra | Purchase-order approval: manager approves, the order is sent, the app waits for the delivery confirmation and sends reminders | Workflows (13) |
| Long-running work, standard face | Extra | A month-end stock valuation report that clients can track | Tasks, backed by a Workflow (14) |
| Telling the client when something happens | Extra | `stock.low`, `po.approved` and `delivery.received` events | Events (15) |
| Safe against attack | Core | A threat model, token audience checks, least-privilege tools, elicitation confirmations before destructive actions, and an injection test on supplier and order notes | Security hardening (16) |
| Proven and observable | Core | Unit and integration tests, an eval set for tool and screen choice, structured logs and traces | Testing, evals and observability (17) |
| Production | Core | Staging and production environments, CI/CD and a release checklist with rollback | Deploy and test (18) |
| A price | Core | A monthly price with the Cloudflare and Neon costs behind it, in CLIENT.md and NOTES.md | Billing and distribution (20) |
| Self-serve plans and billing | Extra | Plans per company with usage limits checked on every call, and Stripe test-mode checkout, for a product sold to many companies (a bespoke build for one client is usually invoiced instead) | Billing and distribution (20) |
| Directory listing | Extra | A directory submission pack, for a product that customers should find in Claude's and ChatGPT's directories | Billing and distribution (20) |

## Key concepts

- **Brief to architecture**: Map each client requirement to the simplest technology that meets it, and write down why.
- **Core and extras**: The core is what every multi-tenant MCP product needs. An extra earns its place only when a client need calls for it.
- **Multi-tenancy**: One deployment serves many companies, and every query, object name and queue message carries the company and user it belongs to.
- **Graceful fallback**: When a client doesn't support an extension, plain tools still get the job done.
- **Evaluation**: A fixed set of realistic prompts with the expected tools and screens, run in each client before every release.
- **Handover**: Documentation, a demo and a test plan that let a client run and trust what you built.

## Outline

1. The client brief and choosing a product
2. Architecture: mapping requirements to technologies
3. A fresh project, tenants and sign-in
4. Data model and core tools
5. Screens and generative UI
6. The extras the client needs, and why
7. Production deploy, evaluation and fallbacks
8. Demo, documentation and pricing
9. Validating with a client

## Prerequisites

- [Step 20](../step-20-billing-and-distribution/), and every step before it
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)). The prompts here are shorter than in earlier steps on purpose: you make more of the decisions.

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 20 code into `step-21-capstone-mcp-app/` as a starting skeleton.**

```text
Copy my step-20 code into step-21-capstone-mcp-app/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-21 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Don't change any features yet.
```

**2. Switch to plan mode and design the product.**

```text
Read step-21-capstone-mcp-app/README.md and the technology rules in AGENTS.md. I'm building Stockroom Inventory (or tell me if I name another product). Propose the architecture: tenants and roles, Neon tables, the tools with their annotations, and the widgets for every Core row. Then interview me about the client's needs and propose which Extra rows to add, at least two, with the need, why something simpler isn't enough and the running cost for each, and one line for each extra we leave out. If we choose Skills, Tasks or Events, include a fallback for clients that don't support them yet. Write the agreed design to step-21-capstone-mcp-app/NOTES.md.
```

**3. Start the product on a clean base.**

```text
Remove the sales-specific code, keeping the auth, Workers, widget build and test setup. Create a new D1 database for Stockroom Inventory's sign-in and a new Neon project or branch for its data, add tenants and roles (and single sign-on with Better Auth's SSO plugin only if we chose that extra), and make every query and tool scoped to the signed-in user's company.
```

**4. Build the data model and core tools.**

```text
Create the Neon schema and seed data for two sample companies, then build the core tools for products, stock, suppliers and purchase orders. If we chose search by meaning, make supplier terms and the catalog searchable with pgvector, citing or abstaining, with a retrieval question set. Add tests that one company can never see another's data.
```

**5. Build the screens.**

```text
Build the stock table, reorder form, purchase-order card and KPI dashboard widgets, and write tool and widget descriptions so the model picks the right screen from the user's intent.
```

**6. Add only the extras you chose.**

```text
Read the extras we chose in NOTES.md. Build each one the way its step taught it, scoped to the caller's company, with tests. Don't add anything that isn't on that list. If building an extra shows it isn't needed after all, tell me so we can drop it and update NOTES.md.
```

**7. Ship and evaluate.**

```text
Write SECURITY.md with a threat model and a test for each risk, add unit and integration tests, structured logs and traces, then set up staging and production environments with CI/CD. Then write an evaluation of 25 realistic prompts from a stockroom worker and a manager, with the expected tools and screens, and help me run it in claude.ai, Claude Desktop and ChatGPT and record the results in NOTES.md.
```

**8. Make it a product and prepare the handover.**

```text
If we chose self-serve billing, add plans per company with usage limits checked on every call and Stripe test-mode checkout outside the chat, with step 20's billing tests. If we chose directory listing, write DISTRIBUTION.md with the submission pack. Then add to NOTES.md: an architecture diagram in Mermaid, setup steps for a new client, known limits and fallbacks, a three-minute demo script, and a suggested monthly price with your reasoning.
```

**9. Validate it with a client.**

```text
Help me write CLIENT.md for a real small business, or a classmate playing one: discovery notes from a 30-minute interview about how they track stock today, a one-page proposal with scope and what is out of scope, a price with the costs behind it, notes from a live demo with their feedback, and five acceptance tests in their words, each marked passed or failed after they try the app.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-21-capstone-mcp-app/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: at least two sample companies (plus a dedicated-database company if you chose the enterprise tier), and the cross-tenant suite proves they never see each other's data
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Every Core row is implemented, and at least two Extra rows, each justified in NOTES.md (with a fallback where a client doesn't support an extension yet)
- [ ] NOTES.md has the architecture, evaluation results, setup steps, demo script and price
- [ ] CLIENT.md has the discovery notes, proposal, price, demo feedback and acceptance test results
- [ ] Project defence recorded (up to ten minutes: it works, one tool end to end, one tenant-isolation decision, one change you made; see [SUBMITTING.md](../SUBMITTING.md#10-defend-your-project)) and linked in the form
- [ ] Tagged `step-21-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-21-done`, live server URL, screenshots of at least three different screens chosen from intent in **Claude** and **ChatGPT**, and a link to a demo video of up to three minutes

## References

- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview)
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix)
- [Cloudflare: Model Context Protocol](https://developers.cloudflare.com/agents/model-context-protocol/)
- [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/), [Queues](https://developers.cloudflare.com/queues/) and [Workflows](https://developers.cloudflare.com/workflows/)

---

← [Step 20: Billing and Distribution: Charging for Your Server and Listing It](../step-20-billing-and-distribution/) · [Roadmap](../README.md#roadmap)
