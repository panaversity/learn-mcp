# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Cursor and others) working in this repository. Read it before making changes. Where it conflicts with a direct instruction from a maintainer, the maintainer wins.

## What this repo is

**Build and Sell MCP Apps: The CCDV-F Bootcamp Part I** (repo `learn-mcp`) is a hands-on bootcamp that teaches developers to build **remote MCP servers** in small, incremental steps. It is a companion to [The AI Agent Factory, Second Edition](https://agentfactory.panaversity.org/) by Panaversity.

- **Name:** call this program a **bootcamp** ("The CCDV-F Bootcamp Part I"; its sequel on self-improving AI Workers, under development, is "Build and Sell Self Improving AI Workers: The CCDV-F Bootcamp Part II"), not a course, workbook or lab. "Lab" is reserved for the book's in-chapter exercises (for example Lab 20b). Use "repo" only for the Git repository itself (forking, the repo root, files).
- **Teaching principle: learn by building.** This bootcamp teaches a hands-on way of learning. The book gives the concepts, and the bootcamp is where learners practise them. Every step must end with something the learner builds, deploys and tests themselves in Claude (and ChatGPT). Don't add steps or sections that are only reading or explanation; link the book for theory and keep the step focused on building.
- **North star:** *Generative UI + MCP*. Apps are built around user intent rather than navigation. The model decides which interface to show at runtime, and the MCP server supplies the tools, data, skills and governance behind it.
- **Why it exists:** this bootcamp is hands-on preparation for the CCDV-F exam (Claude Certified Developer: Foundations) and its Panaversity mirror, PCDV-F, and for monetizing these skills in the international market. It is part of [The FDE 2027 Master Plan](https://agentfactory.panaversity.org/fde-2027-master-plan) in the book. The book teaches two products, MCP Apps and self-improving AI Workers, and **this bootcamp covers MCP Apps only**, so keep AI Worker topics out; they belong in Part II. Learners should come away able to build MCP servers and apps for paying international clients, as FDEs or for their own startups, and ready for CCDV-F. Favour realistic, client-ready examples over toy ones.
- **Companion book:** this bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Each step's `## Learn first` section links the book lessons (URL pattern `https://agentfactory.panaversity.org/building-ai-workers/<chapter>/<lesson>/`). The book is updated continuously, so check that links still resolve when you edit a step, and keep the step and the book consistent.
- **Book edition:** link only to the current (second) edition at `agentfactory.panaversity.org`. **Never use or link `agentfactory1.panaversity.org`**, which is the deprecated first edition, and never link a page that redirects there (old `/docs/...` paths do). Check that every book link loads directly before adding it.
- **Book chapters:** the bootcamp draws only on **chapters 12 to 20** of the book. `Learn first` links must come from those chapters. Where they don't cover a technology (for example Durable Objects, Queues, Workflows or the Events extension), say so in a note and link the official docs instead. Links in the root README that aren't lessons (the PCAO-F appendix, the FDE plan, the certification roadmaps) may stay.
- **Audience:** learners who have already passed PCAO-F (Panaversity Certified Associate: Foundations) and now prepare for CCDV-F (and its Panaversity mirror, PCDV-F), and who follow the steps in a Claude project (see **How learners work** below). Write instructions for that workflow, and mention the **Code** tab of Claude Desktop, and the `claude` CLI, only as fallbacks. Every change should make the bootcamp easier to follow, not just make the code work.
- **Status:** every step is ready to build, which means its learning instructions are complete. It does not mean a reference implementation has been independently built and tested, so never claim that (the roadmap's **Instructions** column says "Ready to build" for this reason). Each step folder has a README (what to build, the book lessons, the prompts and the submission checklist) and **no starter or solution code**: learners write the code themselves, with Claude, in their own fork, and submit it. Don't add solution code to this repo, and don't describe the code as "coming soon".
- **How learners work (workflow standard):** local first, with a human in the loop. Learners clone their fork locally with GitHub Desktop, create a Claude project, add the local folder and their GitHub fork to it, and work in the project's threads by default. Claude edits files and runs commands locally, and is told never to commit or push. The learner reviews, commits, tags and pushes in GitHub Desktop, and opens any pull request in GitHub Desktop or on github.com, only inside their own fork. For long-running work while their computer is off, they start a cloud session on their GitHub fork and review its branch themselves. Because a rule in a prompt is not a permission, the README also tells learners to approve commands one at a time, use bootcamp-only Cloudflare, Neon and Stripe (test mode) accounts, and keep production credentials off the machine they build on. The root README's "How we work" section is the learner-facing version; keep the two in sync.

## Target clients

- **Primary testing:** claude.ai and Claude Desktop.
- **Must also work in:** ChatGPT, connected as a custom MCP server in **ChatGPT Plugins** (formerly called developer mode).
- **Out of scope:** anything that needs a self-hosted chat front end, such as AG-UI, CopilotKit or A2UI renderers. Don't add these.

To stay portable, **use only standard MCP and MCP Apps APIs** (`@modelcontextprotocol/ext-apps`). Don't use client-specific APIs (for example ChatGPT-only `window.openai` calls or Claude-only behaviour) unless a step explicitly teaches the difference, and even then keep a standard fallback.

The Skills over MCP, Tasks and Events extensions stay in the roadmap even where client support is still rolling out. Don't remove those steps. Note the support status in the step instead.

## Tech stack

| Area | Choice |
| --- | --- |
| Language | TypeScript (strict mode) |
| TypeScript tooling | TypeScript 7 for type-checking only (`noEmit`), while Wrangler bundles the code with esbuild. Types come from `wrangler types` (a `typecheck` script runs `wrangler types && tsc`), with `@types/node` installed and the tsconfig using `moduleResolution: "bundler"` and `isolatedModules`. Each step sets `compatibility_date` to the day it is created, and adds no `nodejs_compat` flag, because Node.js compatibility is on by default from 2026-08-04 |
| Development | Claude in a Claude project (threads) on the learner's local clone; the Code tab of Claude Desktop as a fallback |
| Git and GitHub | GitHub Desktop for the learner's own Git work (clone, diff, commit, tag, push, sync). Claude may read git status and diffs, but never commits or pushes |
| Hosting | Cloudflare Workers (deployed with Wrangler) |
| MCP transport | Streamable HTTP at `/mcp` |
| MCP server | Cloudflare's stateless `createMcpHandler` (`agents/mcp/server`) with MCP SDK v2, created once at module level. Never the deprecated `McpAgent` class |
| Search by meaning | pgvector on Neon in the same tables and under the same row-level security, `bge-m3` embeddings on Workers AI (`@cf/baai/bge-m3`, 1,024 dimensions) via the `AI` binding, from step 8 on. No separate vector database |
| Tenancy | Shared tables with row-level security by default, dedicated Neon databases for an enterprise tier, one tenant context per request (step 7) |
| Auth | Better Auth (OAuth), from step 4 on, with the organization plugin for companies (tenants) and roles. Chosen because it supports single sign-on through its open-source `@better-auth/sso` plugin (OIDC and SAML), which enterprise clients expect |
| Notes store | Cloudflare Workers KV, for the notes example in steps 2 to 4 |
| Auth database | Cloudflare D1, used only for auth data |
| Business database | Neon serverless Postgres, from step 5 on (Hyperdrive and row-level security from step 6) |
| UI | MCP Apps widgets built with React, from step 9 on |
| Per-user state | Cloudflare Durable Objects, from step 11 on |
| Background work | Cloudflare Queues, from step 12 on |
| Durable jobs | Cloudflare Workflows, from step 13 on (also the engine behind MCP Tasks in step 14) |
| Evals | An automated eval script through the Claude API's MCP connector, from step 17 on, plus a manual pass in each client |
| Payments | Stripe in test mode, step 20 |

Don't introduce another language, host, auth library or database without a maintainer asking for it.

## Repository layout

```
.
├── AGENTS.md                 # this file
├── CLAUDE.md                 # imports AGENTS.md, because Claude project threads read CLAUDE.md
├── README.md                 # bootcamp overview and roadmap
├── SUBMITTING.md             # how students fork, tag and submit
├── LICENSE                   # MIT
├── .gitignore                # keeps node_modules, .wrangler and .dev.vars out of forks
├── .claude/skills/           # vendored Anthropic mcp-server-dev skills (do not edit)
├── assets/                  # README hero images (rendered at 2x from assets/src/*.svg) and step diagrams, all as light and dark PNGs
├── docs/                     # student guides: affordable-agentic-ai.md, costs-and-cleanup.md, ccdv-f-skills-map.md
└── step-NN-<name>/           # one folder per step
    └── README.md             # what to build, Claude Code prompts, references (no solution code)
```

### Step folders

- Name them `step-NN-<descriptive-name>`: a two-digit number plus a short kebab-case name (e.g. `step-01-hello-world`). Never rename an existing step folder without asking, because the README links and the book reference these paths.
- The current steps are:
  0. `step-00-readiness-check` (titled "Step 00: Are You Ready? Readiness Check"): a short self-check (Git and GitHub, terminal, reading TypeScript, HTTP and JSON, SQL basics, secrets, working with Claude in a Claude project) with a test per skill, so learners can test out. Not graded and not counted in the "21 steps"; it ends with the learner's fork set up and a `NOTES.md` of results, and it points to book lessons (chapters 12 to 20) and official docs rather than teaching the skills itself. Keep it short and don't grow it into a module
  1. `step-01-hello-world`: minimal remote MCP server on Cloudflare
  2. `step-02-tool-design`: designing good tools
  3. `step-03-resources-and-prompts`: resources and prompts, the other two MCP primitives
  4. `step-04-auth`: Better Auth + D1, with companies (tenants) and roles through the organization plugin
  5. `step-05-relational-databases`: the server connects to Neon; relational design, SQL and 1NF to 3NF, with read-only schema tools
  6. `step-06-neon-production`: production data tools: Hyperdrive, paginated and transactional tools, row-level security
  7. `step-07-multi-tenancy`: multi-tenant architecture: isolation models, tenant context, shared and dedicated databases, tenant-scoped helpers, cross-tenant tests
  8. `step-08-semantic-search-pgvector`: search by meaning with pgvector on Neon: passages, bge-m3 embeddings on Workers AI, tenant-scoped search that cites or abstains, a retrieval question set in CI
  9. `step-09-generative-ui-mcp-apps`: React widgets via MCP Apps
  10. `step-10-skills-over-mcp`: Skills over MCP: a shared skill plus per-company skills
  11. `step-11-durable-objects`: per-user state (draft orders, rate limits) with Durable Objects
  12. `step-12-queues`: background bulk imports with Queues
  13. `step-13-cloudflare-workflows`: durable jobs with Cloudflare Workflows
  14. `step-14-tasks`: long-running work with Tasks, backed by the step 13 Workflow
  15. `step-15-events`: Events
  16. `step-16-security-hardening`: threat model and hardening (tokens, least privilege, elicitation confirmations, injection, secrets, audit)
  17. `step-17-testing-and-observability`: tests in the Workers runtime, evals, logs and traces
  18. `step-18-deploy-and-test`: production deploy and cross-client testing
  19. `step-19-generative-ui-dashboard`: intent-driven sales dashboard (generative UI)
  20. `step-20-billing-and-distribution`: Stripe plans, usage limits, and directory listing
  21. `step-21-capstone-mcp-app`: capstone, a complete client-ready MCP app built from a required core plus justified extras
- **Every step builds on the remote MCP server.** Each step adds to the same deployed server and is tested through it in Claude and ChatGPT. No standalone exercises: database design, SQL, background jobs and everything else are learned by building them into the server and exposing them through its tools.
- **Each step is self-contained and starts from a copy of the previous step's code.** Never import code across step folders. A learner must be able to `cd` into any step and run it on its own.
- Each step has its own `package.json`, `wrangler` config, `tsconfig.json` and lockfile. There is no root workspace.
- **The client: Stockroom.** Every step is framed as work for one fictional client, **Stockroom**, a startup that sells AI-native back-office software to small distributors and retailers (its customers are the tenants), with the learner as its FDE. Steps 1 to 20 build Stockroom's sales assistant, and the step 21 capstone builds its next product, Stockroom Inventory. Each step README has a one- or two-sentence `> 🏬 **Stockroom's ask:**` blockquote right after the summary paragraph, giving the business reason for the step. Keep it when editing a step, keep it a light touch (no extra sections), and keep it consistent with the root README's "Your client: Stockroom" section.
- Keep the running example consistent: a notes server stored in Workers KV (steps 2 to 4, per-user from step 4) that grows into a sales-data server on Neon (step 5 onward) and ends in the generative UI dashboard (step 19). The step 21 capstone is a new product (Stockroom Inventory by default) built from a required core plus at least two extras the student justifies; keep its requirements table (with its Core or extra column) in sync when a step is added or removed. Never keep data in a Worker's memory between requests (book lesson 17.4).
- Each step deploys as its own Worker named `learn-mcp-step-NN` (e.g. `learn-mcp-step-03`), so earlier steps stay live while they are graded.
- **One Neon branch per step.** From step 5, each step uses its own Neon branch (`step-NN`), created from the previous step's branch by the copy prompt in steps 6 to 20, so a later migration never breaks an earlier step that is still being graded. The capstone (step 21) starts a new Neon project or branch.
- Each step's first Claude Code prompt copies the previous step's code into the new folder, skipping the bootcamp's `README.md`, `NOTES.md`, `node_modules/`, `.wrangler/`, `dist/` and `.dev.vars`, and renames the Worker. Keep that prompt when editing a step.
- Prompts must never ask Claude to write or edit a step's `README.md`. Learner write-ups go in `NOTES.md` in the step folder.

## Multi-tenancy

Multi-tenancy is a first-class theme of this bootcamp, because the servers learners build for clients serve many companies. Keep these rules in every step:

- **Where the tenant comes from.** The company (`orgId`), user, role and tier are resolved once per request by step 7's tenant module. The company comes only from the verified token, chosen when the user connects; there is no switch-company tool. Membership and role are re-checked against the database on every request, so removal or demotion takes effect on the next call. No tool, resource or prompt ever takes a company or user ID as input.
- **Data.** Every business row carries `org_id`. Shared-tier companies use shared tables with row-level security, set per transaction (step 6); enterprise companies may get a dedicated Neon database behind the tenant router (step 7). Tools never know which tier they use, and migrations run across every tenant database.
- **Names.** KV keys, Durable Object names, cache entries and log fields start with the company (`org:<orgId>:...`), built with the tenant helpers.
- **Background work.** Every queue message, Workflow run, task and event carries the tenant envelope (`orgId`, `userId`, `requestId`), and the handler re-checks it before touching data.
- **Authority at run time.** Background work can run long after the request that started it, and the user may have been removed or demoted since. So a handler acting for a user re-checks that user's current membership and role in the database right before each consequential action (a write, a send, a payment, an approval), never trusting the role captured when the work began. If the right is gone, it stops, records a clear reason and does nothing. Work that runs with no person behind it (a scheduled job) runs under its own service identity with the least scope it needs (book lesson 20.14), and the audit log says so. Steps 12 to 16 test a removal and a demotion mid-run.
- **Skills and caching.** Per-company skills (step 10) are listed and served only to that company, and any list or read result that depends on the caller is returned with `cacheScope: "private"`, so no shared cache hands one company's answer to another.
- **Confirmations.** An elicitation's `requestState` is sealed and bound to the company and user who started the call, and a retry from anyone else is refused.
- **Per-company config, sign-in and billing.** Settings and limits are per company (cached in KV), SSO providers are linked to a company's organization, and plans and usage are counted per company (step 20).
- **Proof.** From step 7, every step keeps a cross-tenant test suite passing, and every step README from step 8 on has a `Tenant isolation:` item in its submission checklist. A cross-tenant leak fails a step outright (see the rubric in `SUBMITTING.md`).

## Working in a step

Run commands from inside the step folder, not the repo root. The steps will typically use:

```bash
cd step-NN-<name>
npm install
npm run dev          # local Worker (wrangler dev)
npm test             # tests, once a step has them
npm run deploy       # wrangler deploy
```

If a step's `package.json` defines different scripts, follow that file. When you create a step, provide `dev`, `test`, `deploy` and (from step 9) `build:widgets` scripts so the commands stay the same across steps.

- **Local testing:** use the MCP Inspector against the local `/mcp` endpoint before deploying.
- **Secrets:** use `wrangler secret put` and `.dev.vars` for local values. Never commit secrets, `.dev.vars`, connection strings or tokens. Commit a `.dev.vars.example` with placeholder values instead.
- **Databases:** D1 migrations go through Wrangler. Neon schema changes go through migration files committed in the step. Never run destructive SQL against a shared database without asking.
- **Deploying:** only deploy when asked. A deploy publishes a public URL.

## Coding conventions

- TypeScript in strict mode, ES modules, `async`/`await`. No `any` without a comment explaining why.
- Validate every tool input with a schema (e.g. Zod), and keep input schemas tight: required fields, enums, length limits.
- Treat tool names, descriptions and schemas as prompts. They are written for the model, so make them clear and specific.
- Return structured content alongside a short text summary.
- Set tool annotations correctly (`readOnlyHint`, `destructiveHint`, etc.).
- Return clear, recoverable errors rather than throwing opaque ones.
- Ask before anything costly or irreversible. Destructive tools and actions that spend money confirm with the user through elicitation (form mode for a confirmation, URL mode for payment or anything sensitive), using the stateless `input_required` pattern with a `requestState` sealed by HMAC and bound to company, user, tool, arguments and an expiry, plus a plain two-call fallback for clients without elicitation (steps 16 and 20).
- Follow the 2026-07-28 spec. Don't use what it removed (protocol sessions, the `initialize` handshake) or deprecated (Sampling, Roots, the Logging feature, the HTTP+SSE transport), even when older tutorials show them: use OpenTelemetry for logs and traces, and prefer Client ID Metadata Documents over Dynamic Client Registration. Prompts are still in the spec but rarely used, so teach skills (step 10) for reusable procedures and keep prompts to step 3.
- Durable Objects, Queues and Workflows are part of the MCP server: define and bind them in the same Worker that serves `/mcp`, and drive them from MCP tools (and through Tasks from step 14). Never run long work inside a single tool call or keep state in memory. Pick the store or runtime by the job:
  - **Workers KV**: small, read-mostly data that can be up to a minute stale (the notes in steps 2 to 4, config, caches).
  - **D1**: auth data only.
  - **Neon**: business records, the source of truth, with row-level security forced on every business table: `USING` and `WITH CHECK` on `current_setting('app.tenant_id', true)`, the company set per transaction with `set_config(..., $1, true)` through one helper, a connector role without `BYPASSRLS`, the company in every key, and Hyperdrive query caching off (book chapter 19).
  - **pgvector** (step 8 on): passages and embeddings in Neon under the same policies. Search tools cite document and line, abstain below a threshold, and are measured by a retrieval question set in CI.
  - **Durable Objects** (step 11 on): small per-key state that must be strongly consistent, such as drafts, rate limits, quotas, live sessions and subscriptions. Name each object with the tenant helper from the verified company and user IDs (`org:<orgId>:user:<userId>`), or from the company ID alone for company-wide state, never from tool input. Not a business database.
  - **Queues** (step 12 on): many independent pieces of background work or fan-out. Delivery is at least once, so every consumer write must be idempotent, with a dead-letter queue for repeated failures.
  - **Workflows** (step 13 on): one job with ordered steps, waits (`step.sleep`, `step.waitForEvent`), per-step retries and a status the model can check.
- Scope every data access to the authenticated user and their company (from step 4 onward). Business rows in Neon carry an `org_id`, and row-level security enforces it per transaction from step 6.
- Keep responses small enough for a model's context: paginate and summarize.
- Widgets (step 9+) are React, bundled into self-contained `ui://` resources. Respect host theming and keep payloads small.
- Prefer clarity over cleverness. This is teaching code, so a learner should be able to read every file. Add short comments that explain *why*, not *what*.
- Keep dependencies minimal and pin versions in each step.

## Bundled skills

`.claude/skills/` contains Anthropic's `mcp-server-dev` skills, vendored unchanged under Apache 2.0:

- `build-mcp-server`: the entry point for deployment model, tool design, auth and references.
- `build-mcp-app`: MCP Apps widgets, iframe sandbox and payload budgeting.
- `build-mcpb`: packaging local servers as bundles. This bootcamp mostly builds remote servers, so it is rarely needed.

Use these skills when building steps. **Don't edit the vendored files.** Note that the vendored `build-mcp-server` Cloudflare reference (`references/deploy-cloudflare-workers.md`, last verified upstream 2026-03) still shows the deprecated `McpAgent` class. This bootcamp uses `createMcpHandler` instead, and the step prompts say so explicitly; keep them that way until upstream catches up. To update them, follow `.claude/skills/README.md`.

## Writing step READMEs

Every step's `README.md` follows the same structure, so keep it:

1. `# Step NN: <Title>`
2. A one-paragraph summary
3. `> 🏬 **Stockroom's ask:** …`, the business reason for the step in one or two sentences
4. An optional client-support note (for extensions still rolling out)
5. `## Learn first` (book lessons to study before the step, each with a one-line reason, then an **Official docs** list of deep links to the Cloudflare, Neon, Better Auth and MCP pages the step needs, each with a one-line reason)
6. `## Why this step matters`
7. `## Why this technology in an MCP server: <technology>` (bullets: **What it is**, **Why an MCP server uses it**, **What it solves**, **Alternatives**)
8. `## What you will learn` (bullets)
9. `## What you will build`
10. `## Key concepts` (bold term, then a one-sentence definition)
11. `## Outline` (numbered)
12. `## Prerequisites` (link the previous step and the main README)
13. `## Build it with Claude Code` (numbered instructions, each followed by a copy-paste prompt in a ```` ```text ```` block)
14. `## Testing targets`
15. `## Submission checklist`
16. `## References`
17. Footer navigation: previous step · Roadmap · next step

Step 00 (the readiness check) is the one exception to this structure: it has the summary, Stockroom's ask, a `## How it works` section (the order of work and a study-time table), the seven checks (each with "You're ready if", a test and "If not, read"), then `## Why this step matters`, `## What you will build`, `## Prerequisites`, `## Build it with Claude Code`, `## Submission checklist`, `## References` and the footer.

Style:
- Write plain, friendly English for learners, in full sentences. Explain jargon the first time it appears.
- Prompts are written as the learner would type them to Claude Code. They are specific and self-contained, and they ask Claude to plan or explain before writing big changes.
- The first instruction in each step tells the learner to start a new thread for the step in their bootcamp's Claude project and to work inside the step folder. Don't give terminal-only keyboard shortcuts. Describe UI actions in the Claude app instead, generically where the projects UI may still change.
- **Tooling standard for Git:** learners do routine Git work (clone, read a diff, commit, tag, push, sync) in GitHub Desktop, beside Claude Code in Claude Desktop. Write those instructions as GitHub Desktop actions (for example, tag with **History**, right-click the commit, **Create Tag…**, then **Push origin**), with git commands at most as a collapsed command-line fallback. Prompts never ask Claude to commit or push; they end with the learner reviewing and committing in GitHub Desktop.
- Link to official docs (modelcontextprotocol.io, developers.cloudflare.com, better-auth.com/docs, neon.com/docs) rather than copying them. Prefer deep links to the exact page, and check each one loads.
- Steps carry no status banner. If a step is ever withdrawn or reworked, say so in the **Instructions** column of the roadmap table in the root `README.md`.

## Keeping things in sync

When you add, rename or reorder a step, update all of these together:
- the step folder name
- the roadmap table in `README.md`
- the previous/next links in the neighbouring step READMEs
- the step list in this file
- the step count badge in `assets/src/hero-light.svg` and `assets/src/hero-dark.svg`, then re-render both PNGs (2560×1280)
- the milestones table and the CCDV-F skills map in `README.md`
- the capstone requirements table in step 21
- `docs/ccdv-f-skills-map.md` (its Steps column) and `docs/costs-and-cleanup.md` (its Steps column and the Neon branch count)

## Students and submissions

This repo is a student assignment. Students **fork** it, work in their fork, and submit through a form (see `SUBMITTING.md`).

- **Never open pull requests to `panaversity/learn-mcp` on a student's behalf**, and never suggest it. Student work stays in their fork.
- If you are helping a student in their fork, don't commit or push: they review, commit and push in GitHub Desktop. Only if they explicitly ask you to, push to *their fork's* `origin`, never anywhere else.
- Each finished step is tagged `step-NN-done` (e.g. `step-01-done`) and pushed to the fork.
- Students deploy to their own Cloudflare and Neon accounts. Never hard-code a maintainer's account IDs, URLs or secrets.
- Students shouldn't edit bootcamp-owned files (`README.md`, `AGENTS.md`, `SUBMITTING.md`, `.claude/skills/`, step `README.md` files), because that causes conflicts when they sync their fork. Their notes go in `NOTES.md` inside the step folder.
- Every step README ends with a `## Submission checklist`, placed before `## References`. Keep it when editing a step.
- At the end of each milestone (steps 4, 8, 10, 15, 18 and 21) students record a **project defence** (`SUBMITTING.md` section 10): a short screen recording explaining one tool end to end and one tenant-isolation decision in their own words. It is a pass-or-fail gate in the rubric, and those steps' submission checklists have a matching item. If milestones change, update the list in `SUBMITTING.md`, the root README and the affected checklists together.
- Grading follows the rubric in `SUBMITTING.md` (section 9). Keep it consistent with the steps' submission checklists, and until the maintainer supplies the form link, keep section 8's statement that graded submissions open later and that the link will be added there. Never invent a link.

## Git

- Make small, focused commits with descriptive messages, e.g. `step-03: add Better Auth with D1`.
- Don't commit build output, `node_modules/`, `.wrangler/`, `.dev.vars` or secrets.
- Don't rewrite published history.

## Useful references

- MCP specification: https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro
- MCP 2026-07-28 key changes: https://modelcontextprotocol.io/specification/2026-07-28/changelog
- MCP extensions overview: https://modelcontextprotocol.io/extensions/overview
- Extensions client support matrix: https://modelcontextprotocol.io/extensions/client-matrix
- MCP Apps: https://modelcontextprotocol.io/extensions/apps/overview
- MCP Apps SDK: https://github.com/modelcontextprotocol/ext-apps
- Cloudflare MCP docs: https://developers.cloudflare.com/agents/model-context-protocol/
- Better Auth: https://better-auth.com/docs
- Neon: https://neon.com/docs
