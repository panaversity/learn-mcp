# Build and Sell MCP Apps: The CCDV-F Bootcamp Part I

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/hero-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="assets/hero-light.png">
  <img src="assets/hero-light.png" alt="Build and Sell MCP Apps, the CCDV-F Bootcamp Part I: Claude and ChatGPT connect to your multi-tenant MCP server on Cloudflare Workers, backed by Neon with row-level security, pgvector, Durable Objects, Queues and Workflows" width="100%">
</picture>

Welcome, readers of [The AI Agent Factory](https://agentfactory.panaversity.org/)! If you have finished your study for [PCAO-F](https://agentfactory.panaversity.org/appendices/exams/associate-foundations/overview/) (Panaversity Certified Associate: Foundations), congratulations: you already know how to work with AI agents and manage the work they do. Now it's time to build. This bootcamp is your hands-on preparation for **CCDV-F** (Claude Certified Developer: Foundations), Anthropic's developer exam, and for **PCDV-F** (Panaversity Certified Developer: Foundations), the Panaversity exam that mirrors it. Just as important, it teaches you to monetize these skills in the international market: building MCP apps for paying clients as a **Forward Deployed Engineer (FDE)** or for your own startup.

> [!TIP]
> **💰 On a budget?** Start with our student guide **[Affordable Agentic AI Development](docs/affordable-agentic-ai.md)**: which AI tools are free, what is worth paying for, and how to compare costs before you buy. See also [What it costs](#what-it-costs) for this bootcamp.

MCP turns two! Anthropic released the Model Context Protocol in November 2024. What started as a way to connect AI apps to tools and data is now becoming the standard for how agents communicate and orchestrate work. MCP also standardizes interactive apps that render inside the chat, through the MCP Apps extension.

MCP's second year was about agents. The [2026-07-28 spec](https://modelcontextprotocol.io/specification/2026-07-28/changelog) made the protocol stateless, with no sessions, so a server runs behind any load balancer, and new features now arrive as optional extensions that clients adopt one at a time. On MCP's second birthday (November 24, 2026), Anthropic made Tasks and Elicitation generally available in Claude, put Skills over MCP into public beta, and announced file uploads. In this bootcamp we learn to build **remote MCP servers** that use these capabilities, one small, incremental step at a time.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/diagram-mcp-timeline-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="assets/diagram-mcp-timeline-light.png">
  <img src="assets/diagram-mcp-timeline-light.png" alt="Two years of MCP: spec releases from 2024-11-05 to 2026-07-28, project milestones, and the official extensions MCP Apps, Enterprise-Managed Authorization, Skills over MCP beta and Events beta. This bootcamp targets spec 2026-07-28." width="100%">
</picture>

Our north star is **Generative UI + MCP**: AI-native apps built around intent rather than navigation. The model decides what interface to show at runtime, and MCP supplies the tools, data and context behind it. MCP Apps is the open standard that carries that UI into the chat, in both Claude and ChatGPT. ([Generative UI + MCP: The New Stack for AI-Native Applications](https://odsc.medium.com/generative-ui-mcp-the-new-stack-for-ai-native-applications-b62af183f2aa), ODSC, 2026)

This bootcamp is the hands-on companion to the book [The AI Agent Factory, Second Edition](https://agentfactory.panaversity.org/). **You learn by building.** The book teaches the concepts, and this bootcamp is where you practise them: every step ends with something you build, deploy and test yourself in Claude and ChatGPT. Each step's README starts with a **Learn first** section that links the book lessons to study before you build that step. Those lessons all come from **chapters 12 to 20** of the book. Steps 5 to 8 follow chapter 19 (Postgres, row-level security and pgvector on Neon) closely. Where those chapters don't cover a technology yet (Durable Objects, Queues, Workflows and the Events extension), the step says so and links the official docs instead. The book's [Practise in a bootcamp](https://agentfactory.panaversity.org/how-to-use-this-book/#practise-in-a-bootcamp) section explains why a bootcamp is worth your time and where this one fits beside the book.

## Why this bootcamp

- **A hands-on way of learning.** You don't learn MCP here by reading or watching. You read the concepts in the book, then build them into a real, deployed MCP server, one small step at a time. By the end, everything you know, you have built yourself.
- **Hands-on preparation for CCDV-F.** This bootcamp is hands-on preparation for **CCDV-F** (Claude Certified Developer: Foundations), and for **PCDV-F** (Panaversity Certified Developer: Foundations), which mirrors it. You don't just read about MCP servers: you build, deploy and test one in Claude, step by step. It practises the exam's MCP, security and evaluation objectives in depth, but not the whole exam: the [CCDV-F skills map](docs/ccdv-f-skills-map.md) shows what it covers and what to study in the book besides. See [Certification and Portfolio Roadmaps](https://agentfactory.panaversity.org/certification-and-portfolio-roadmaps).
- **Skills you can monetize in the international market.** Building MCP servers and MCP Apps is work clients around the world pay for today. Every step builds toward a client-ready product, and the capstone is a portfolio piece you can show to clients and employers, whether you work as an FDE or build your own startup.
- **Part of the FDE 2027 Master Plan.** It is one piece of [The FDE 2027 Master Plan](https://agentfactory.panaversity.org/fde-2027-master-plan) covered in the book, which trains Forward Deployed Engineers (FDEs) and the next generation of AI startup founders.
- **MCP Apps, one of the book's two products.** The book teaches you to build two kinds of products: **MCP Apps** and **AI Workers that improve themselves**. This bootcamp covers **MCP Apps**. Self-improving AI Workers get their own bootcamp, **Build and Sell Self Improving AI Workers: The CCDV-F Bootcamp Part II**, which is under development.

### Coming from the first bootcamp

If you took [Put AI Workers to Work: The CCAO-F Bootcamp](https://github.com/panaversity/learn-ai-at-work), you already met each problem this bootcamp solves, as the person managing AI work. Here you build the engineering behind it:

| In the first bootcamp you learned to | Here you build | Steps |
| --- | --- | --- |
| Give AI approved company knowledge | Resources and search by meaning that cites or abstains | 3, 8 |
| Define what AI may do (the authority envelope) | Sign-in, roles and per-tool permissions | 4, 16 |
| Teach company procedures | Skills served to each company | 10 |
| Keep each company's information apart | Tenant isolation, with row-level security | 4 to 7 |
| Check AI's work | Automated tests and evals | 8, 17 |
| Ask a person before anything risky happens | Elicitation confirmations | 14, 16, 20 |
| Delegate long-running jobs | Queues, Workflows and Tasks | 12, 13, 14 |
| Diagnose a failed run | Logs, traces and audit records | 16, 17 |
| Manage an AI Worker | The tools and data it works through | every step |

## Your client: Stockroom

Every step in this bootcamp is client work, so every step has a business reason. Your client is **Stockroom**, a fictional startup that sells AI-native back-office software to small distributors and retailers, and it has hired you as its Forward Deployed Engineer. Stockroom's customers are companies, each with its own staff and data, which is why everything you build serves many companies at once and keeps them apart.

- **Steps 1 to 20** build Stockroom's first product, a sales assistant that its customers' staff use from Claude and ChatGPT. It starts as one tool, becomes account notes for sales reps, grows into each customer's sales data on Neon, and ends as an intent-driven sales dashboard that Stockroom charges for.
- **Step 21**, the capstone, builds Stockroom's next product, an inventory and purchasing app, from a brief instead of ready-made prompts. You may bring a client of your own instead.

Each step's README opens with **🏬 Stockroom's ask**: what the client needs from that step, and why. Read it first, because it is the reason behind every decision in the step.

## Tech stack

| Area | Choice |
| --- | --- |
| Language | TypeScript |
| Development | Claude in a Claude project (threads) in [the Claude app](https://claude.com/download), working on your local clone; the **Code** tab of Claude Desktop as a fallback |
| Git and GitHub | GitHub Desktop: you review, commit and push, never Claude |
| Hosting | Cloudflare (Workers) |
| MCP server | Cloudflare's stateless [`createMcpHandler`](https://developers.cloudflare.com/agents/model-context-protocol/apis/handler-api/) with MCP SDK v2 |
| Auth | [Better Auth](https://better-auth.com/), with companies and roles through its [organization](https://better-auth.com/docs/plugins/organization) plugin and single sign-on (SSO) through its open-source [`@better-auth/sso`](https://better-auth.com/docs/plugins/sso) plugin |
| Notes store (steps 2 to 4) | Cloudflare Workers KV |
| Auth database | Cloudflare D1 |
| Business database | [Neon](https://neon.com/) (Postgres), with forced row-level security |
| Search by meaning | [pgvector](https://neon.com/docs/extensions/pgvector) on Neon, with `bge-m3` embeddings on [Workers AI](https://developers.cloudflare.com/workers-ai/) |
| Per-user state | [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/) |
| Background work | [Cloudflare Queues](https://developers.cloudflare.com/queues/) |
| Durable jobs | [Cloudflare Workflows](https://developers.cloudflare.com/workflows/) |
| Payments | [Stripe](https://docs.stripe.com/) in test mode (step 20) |
| Target clients | claude.ai and Claude Desktop (primary testing), plus ChatGPT |

## Getting started

### Prerequisites

- [Node.js](https://nodejs.org/) 24 or later (an LTS release) and npm
- [GitHub Desktop](https://desktop.github.com/download/), signed in to your GitHub account. You'll use it to clone, commit, tag and push, beside Claude Code (which may also run git itself, with your approval). Install [Git](https://git-scm.com/downloads) too, because Claude Code uses it.
- [Claude Desktop](https://claude.com/download), signed in with a Pro plan or higher, which includes Claude Code and Claude projects. You'll work in a Claude project with your local folder added (see [How we work](#how-we-work-local-first-human-in-the-loop)).
- A [Cloudflare](https://dash.cloudflare.com/sign-up) account
- A [Neon](https://neon.com/) account
- The same Claude account, to connect and test your servers in Claude Desktop and claude.ai
- A ChatGPT account that can [add a custom MCP server](https://developers.openai.com/api/docs/guides/custom-mcp-server) (in **ChatGPT Plugins** on the web; it used to be called developer mode), to check your servers work there too

#### What it costs

- **Claude:** Claude Code in Claude Desktop needs a paid Claude plan (Pro or higher).
- **ChatGPT:** adding a custom MCP server depends on your ChatGPT plan and, on a work account, your workspace's policies. Check [OpenAI's page](https://developers.openai.com/api/docs/guides/custom-mcp-server) before you start.
- **Cloudflare and Neon:** every step fits the free plans. Neon's Free plan allows 10 branches per project and the bootcamp uses one per step from step 5, so delete a step's branch once it is graded. The **[Costs and cleanup](docs/costs-and-cleanup.md)** guide lists every service, what could cost money and what you may delete after grading.
- **Workers AI:** step 8's embeddings fit the daily free allowance for small documents.
- **Anthropic API:** from step 17, the automated eval script calls the Claude API, which costs a little per run.
- **Stripe:** test mode (step 20) is free.

> [!TIP]
> **Keep your AI costs low.** Read the student guide **[Affordable Agentic AI Development](docs/affordable-agentic-ai.md)**. It covers free and low-cost options (Codex, OpenCode, direct model APIs and local models) and how to compare them before you pay.

### How we work: local first, human in the loop

This bootcamp follows the way Panaversity's own team works: Claude does most of the work on your computer, and you decide what reaches GitHub.

1. **Clone your fork to your computer** with GitHub Desktop ([Set up](#set-up-fork-clone-and-sync) below).
2. **Make a Claude project for the bootcamp.** In the Claude app, create a project, add your local `learn-mcp` folder to it, and add your GitHub fork (`<your-username>/learn-mcp`) as its repository. A project has a coordinator you brief in its main chat, threads that do the work, and settings for its instructions, connectors, plugins and model ([Projects redesigned](https://claude.com/resources/articles/projects-redesigned), Anthropic). In the project's instructions, write: *"Work in my local clone of learn-mcp. Read AGENTS.md first. Never commit or push: I review, commit and push in GitHub Desktop."*
3. **Work in threads.** Start a thread in the project for each step and paste the step's prompts into it. Claude edits the files in your local folder and runs commands on your computer after you approve them.
4. **You commit, not Claude.** Read the changes, then commit, tag and push them yourself in GitHub Desktop. If you use a branch in your own fork, open its pull request in GitHub Desktop or on github.com, inside your fork only, never to the bootcamp repo. Nothing reaches GitHub until a person has read it: that is the human in the loop.
5. **Long jobs while your computer is off.** Work in your local folder stops when your computer sleeps or the lid is closed. For a long-running task, start a cloud session on your GitHub fork instead. It runs on Anthropic's computers, and its work comes back to your fork as a branch that you review, pull and merge yourself. Plugins you installed in the app don't load in a cloud session, but the skills still work there, because the session runs on a copy of your fork, and the fork carries them in [`.claude/skills/`](.claude/skills/). Start it as a cloud session on the fork itself, not as a thread of your project, because project threads don't load a repo's skills.
6. **Make "Claude never commits" a boundary, not just a request.** A rule in a prompt is not a permission. Claude can only do what your computer lets it do, so limit what it can reach. Approve commands one at a time and refuse any `git push`, and any `wrangler deploy` or `wrangler secret put` that the step's prompt didn't ask for. Use Cloudflare, Neon and Stripe accounts made for the bootcamp, with Stripe in test mode, and never sign in to a client's or an employer's production account on the computer you build on. Keep keys in `.dev.vars`, never in a prompt or chat. At the start of each step, ask Claude which permissions and accounts it can use, and check its answer against your settings. It's the same least-privilege rule you build into your servers in step 16.

> **Good to know.** Claude projects are in beta for Pro and Max subscribers and are still rolling out; if you don't have them yet, join the waitlist linked from Anthropic's [Projects redesigned](https://claude.com/resources/articles/projects-redesigned) article. Anthropic's article doesn't give menu names, so the wording here describes the steps in general terms. Threads work directly on the local folder you add to the project, on your computer. (Anthropic's article was written when threads ran only in the cloud; local folders came after it.) Threads don't load the repo's bundled skills on their own (book lesson [12.8](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/both-ai-vendors/#the-details-tool-by-tool)), so add them to your Claude account once, under **Customize → Plugins** or **Customize → Skills** (see [Built-in skills](#built-in-skills)). If you don't have projects yet, open your local `learn-mcp` folder in the **Code** tab of Claude Desktop instead. Everything else stays the same, including committing in GitHub Desktop.

### Set up: fork, clone and sync

This bootcamp is also an assignment, and you do all your work in **your own fork** of this repo.

> ⛔ **Do not open pull requests to this repository.** Your work stays in your fork, and you submit through the submission form. Pull requests to `panaversity/learn-mcp` will be closed without review.

**1. Fork the repo.** Sign in to GitHub, open [panaversity/learn-mcp](https://github.com/panaversity/learn-mcp), and click **Fork** (top right). Choose your own account as the owner, keep the name `learn-mcp`, and click **Create fork**.

**2. Clone your fork** (not the original). In [GitHub Desktop](https://desktop.github.com/download/), signed in to your GitHub account, choose **File → Clone Repository**, pick `<your-username>/learn-mcp` on the **GitHub.com** tab, choose where to save it, and click **Clone**. When GitHub Desktop asks how you plan to use this fork, choose **For my own purposes**, because you never send work back to the bootcamp repo.

**3. Set up your Claude project.** Create a project, add your cloned `learn-mcp` folder (the repo root, not a step folder) and your GitHub fork, and add the project instructions, as in [How we work](#how-we-work-local-first-human-in-the-loop).

> Prefer the terminal? Running `claude` in the repo root gives you a Claude Code session on the same folder, but this bootcamp's instructions assume a Claude project. Claude still doesn't commit.

**4. Keep your fork in sync.** New steps and fixes are added to the bootcamp over time. Before starting each step, click **Sync fork → Update branch** on your fork's GitHub page, then click **Fetch origin** and **Pull origin** in GitHub Desktop to bring the update to your computer.

To avoid merge conflicts, don't edit the bootcamp's own files (`README.md`, `AGENTS.md`, `SUBMITTING.md`, `.claude/skills/`, or the `README.md` in each step folder). Put your own notes in a `NOTES.md` inside the step folder.

**5. Work, tag and submit.** Build each step in its folder, deploy it to your own Cloudflare (and Neon) accounts, commit and tag it in GitHub Desktop (in **History**, right-click the commit, choose **Create Tag…** and name it `step-01-done`, then click **Push origin**), and submit it through the form. **[SUBMITTING.md](SUBMITTING.md)** has the full details: tagging, keeping secrets out of your public fork, and exactly what to submit for each step.

### Built-in skills

The bootcamp's prompts use Anthropic's MCP server development skills:

- **`build-mcp-server`**: the starting point. It helps you choose a deployment model and design your tools.
- **`build-mcp-app`**: adds interactive UI (forms, pickers, dashboards) that renders in the chat.
- **`build-mcpb`**: packages a local MCP server as an installable bundle.

They live in [`.claude/skills/`](.claude/skills/). In the **Code** tab of Claude Desktop they load automatically when you open the repo root. A thread in a Claude project doesn't load a repo's skills (book lesson [12.8](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/both-ai-vendors/#the-details-tool-by-tool)), so add them to your Claude account once.

**Add the skills to your Claude account** (so every thread has them):

1. In **Settings → Capabilities**, make sure **Code execution and file creation** is on (in newer versions of the app it is called **Create and edit files**). Skills need it.
2. **Easiest: install the plugin.** Open **Customize → Plugins**, find Anthropic's `mcp-server-dev` plugin on the **Discover** tab, and install it. Its three skills then appear under **Customize → Skills**, labelled with the plugin's name, and turn on and off with the plugin.
3. **Or upload them yourself.** Zip each skill's folder from [`.claude/skills/`](.claude/skills/) (for example `build-mcp-server/`). Then open **Customize → Skills**, click **+**, choose **+ Create skill**, then **Upload a skill**, and upload the ZIP. Repeat for each skill, and turn each one on.

Then type `/` in a thread's prompt box: you should see `build-mcp-server`, `build-mcp-app` and `build-mcpb`. Anthropic's help pages: [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude) and [Skills overview](https://claude.com/docs/skills/overview). Menu names are from those pages as of November 2026 and may change.

**In a cloud session** (for long jobs, [How we work](#how-we-work-local-first-human-in-the-loop) step 5), installed plugins don't load, but you don't need them: the session works on a copy of your fork, and the skills come with it from `.claude/skills/`.

## Roadmap

Each step lives in its own numbered directory and builds on the one before it. Each folder holds a README that tells you what to build and gives you the prompts to build it with Claude. There is no starter or solution code to download: **you write the code yourself**, with Claude, in your own fork, and that code is what you submit.

**What "Ready to build" means.** The **Instructions** column says a step's learning instructions are complete: what to build, the book lessons, the prompts and the submission checklist. It does not say that a reference implementation of the step has been independently built and tested. The bootcamp team is building the steps alongside the first cohort, and tools and vendor APIs change, so if a prompt or a link doesn't work as written, tell your instructor and sync your fork to pick up fixes.

Start with **Step 00: Are You Ready? Readiness Check**, a short self-check of the skills every step assumes (Git and GitHub, the terminal, reading TypeScript, HTTP and JSON, SQL basics, secrets and working with Claude in a Claude project). If you pass its seven checks, you go straight to step 1.

| Step | Directory | Topic | Instructions |
| --- | --- | --- | --- |
| 0 | [`step-00-readiness-check/`](step-00-readiness-check/) | Are you ready? Readiness check: seven short tests, with book sections for any you don't pass. Not graded | ✅ Ready to build |
| 1 | [`step-01-hello-world/`](step-01-hello-world/) | Hello world: a minimal remote MCP server on Cloudflare | ✅ Ready to build |
| 2 | [`step-02-tool-design/`](step-02-tool-design/) | Designing good tools | ✅ Ready to build |
| 3 | [`step-03-resources-and-prompts/`](step-03-resources-and-prompts/) | Resources and prompts: the other two MCP primitives, and when to use each instead of a tool | ✅ Ready to build |
| 4 | [`step-04-auth/`](step-04-auth/) | Adding authorization with Better Auth and D1: sign-in, companies (tenants) and roles | ✅ Ready to build |
| 5 | [`step-05-relational-databases/`](step-05-relational-databases/) | Relational databases with Neon: the server connects to Neon, and you design its sales schema through the first three normal forms, with read-only tools over it | ✅ Ready to build |
| 6 | [`step-06-neon-production/`](step-06-neon-production/) | Production data tools on Neon: Hyperdrive, paginated and transactional tools, and row-level security | ✅ Ready to build |
| 7 | [`step-07-multi-tenancy/`](step-07-multi-tenancy/) | Multi-tenant MCP servers and databases: isolation models, tenant context in every call, shared and dedicated databases, and cross-tenant tests | ✅ Ready to build |
| 8 | [`step-08-semantic-search-pgvector/`](step-08-semantic-search-pgvector/) | Search by meaning with pgvector: tenant-scoped retrieval on Neon, answers that cite or abstain, and a question set in CI | ✅ Ready to build |
| 9 | [`step-09-generative-ui-mcp-apps/`](step-09-generative-ui-mcp-apps/) | Generative UI with MCP Apps: fixed React widgets that tools return, with the model selecting which one to show | ✅ Ready to build |
| 10 | [`step-10-skills-over-mcp/`](step-10-skills-over-mcp/) | Skills over MCP: a shared playbook plus skills built for each company, served from your signed-in server | ✅ Ready to build |
| 11 | [`step-11-durable-objects/`](step-11-durable-objects/) | Per-user state with Durable Objects: draft orders and rate limits that stay consistent | ✅ Ready to build |
| 12 | [`step-12-queues/`](step-12-queues/) | Background work with Queues: bulk imports processed in batches, with retries | ✅ Ready to build |
| 13 | [`step-13-cloudflare-workflows/`](step-13-cloudflare-workflows/) | Durable jobs with Cloudflare Workflows: multi-step, retryable work that a tool starts and the model checks on | ✅ Ready to build |
| 14 | [`step-14-tasks/`](step-14-tasks/) | Long-running work with Tasks | ✅ Ready to build |
| 15 | [`step-15-events/`](step-15-events/) | Events | ✅ Ready to build |
| 16 | [`step-16-security-hardening/`](step-16-security-hardening/) | Security hardening: threat model, token audience, least-privilege tools, elicitation confirmations, prompt injection, secrets and audit | ✅ Ready to build |
| 17 | [`step-17-testing-and-observability/`](step-17-testing-and-observability/) | Testing, evals and observability: tests in the Workers runtime, a tool-selection eval set, logs and traces | ✅ Ready to build |
| 18 | [`step-18-deploy-and-test/`](step-18-deploy-and-test/) | Deploying and testing on claude.ai, Claude Desktop and ChatGPT | ✅ Ready to build |
| 19 | [`step-19-generative-ui-dashboard/`](step-19-generative-ui-dashboard/) | Generative UI: an intent-driven dashboard, where the model composes a screen from approved widgets and fills it with Neon data | ✅ Ready to build |
| 20 | [`step-20-billing-and-distribution/`](step-20-billing-and-distribution/) | Billing and distribution: Stripe plans, usage limits on every call, and listing in Claude's and ChatGPT's directories | ✅ Ready to build |
| 21 | [`step-21-capstone-mcp-app/`](step-21-capstone-mcp-app/) | Capstone: a complete, client-ready MCP app (Stockroom, or your own client) built from the bootcamp's core plus the extras the client needs | ✅ Ready to build |

### Milestones and pacing

The 21 steps fall into six milestones, after the step 00 readiness check (about an hour if you pass every check; if you don't, its reading takes 1 to 10 hours per check, see its study-time table). Times are rough estimates for a learner working with Claude Code, including reading the linked lessons. Steps 10, 14 and 15 are lighter, because they teach extensions that clients are still adopting (Claude now supports Tasks and, in beta, Skills over MCP, while Events is still rolling out); build them anyway, with the plain-tool fallback each step describes.

| Milestone | Steps | Estimated time | What you can do at the end |
| --- | --- | --- | --- |
| 1. A remote MCP server | 1 to 4 | 18 to 22 hours | Ship a signed-in MCP server, with companies and roles, to Claude and ChatGPT |
| 2. Multi-tenant business data and retrieval | 5 to 8 | 28 to 36 hours | Design a 3NF schema, force row-level security, serve many companies from shared and dedicated databases, and search their documents by meaning with citations |
| 3. Screens and know-how | 9 and 10 | 10 to 12 hours | Return React widgets in the chat and ship skills with the server |
| 4. State and long work | 11 to 15 | 18 to 24 hours | Handle drafts, rate limits, bulk imports, durable jobs, tasks and events, all tenant-scoped |
| 5. Production | 16 to 18 | 16 to 20 hours | Harden, test, evaluate, observe and release the server safely |
| 6. A product | 19 to 21 | 35 to 50 hours | Build an intent-driven dashboard, charge each company for it, and deliver a complete client app |

The whole bootcamp is about 125 to 164 hours. You don't have to finish it to get value from it, so there are three completion levels:

| Level | Steps | Estimated time | You can show |
| --- | --- | --- | --- |
| Foundation | 1 to 4 | 18 to 22 hours | A signed-in remote MCP server with companies and roles, working in Claude and ChatGPT |
| Professional | 1 to 10 | 56 to 70 hours | A multi-tenant server with forced row-level security, cited search and MCP Apps widgets |
| Advanced | 1 to 21 | 125 to 164 hours | A tested, deployed MCP app with a commercial proposal |

**Multi-tenancy runs through the whole bootcamp.** Every user belongs to a company from step 4, every business row carries it in its keys from step 5, the database enforces it with forced row-level security from step 6, step 7 makes it the server's architecture, step 8 keeps search by meaning inside it, and from step 8 on every step's checklist includes a tenant-isolation check.

### What this bootcamp practises for CCDV-F

The bootcamp is hands-on preparation, not a copy of the exam. For each of the exam guide's 24 objectives, the **[CCDV-F skills map](docs/ccdv-f-skills-map.md)** shows whether this bootcamp practises it, in which steps, what evidence in your fork shows it, and what is left for you to study in the book. The table below is the short version. **Direct** rows practise objectives in the exam guide's Tools and MCP, Security and Safety, and evaluation domains. **Supporting** rows build the skills behind them. **FDE** rows are for client work, not the exam. Finishing the bootcamp does not by itself mean you are ready for the exam: check yourself against the official guide.

| Skill | Kind | Steps |
| --- | --- | --- |
| Building remote MCP servers and connecting them to Claude | Direct | 1, 18 |
| Designing tools, resources and prompts the model uses well | Direct | 2, 3, 19 |
| Securing a server: OAuth, roles, token audience, prompt injection | Direct | 4, 6, 16 |
| Asking the user with elicitation before costly or irreversible actions | Direct | 14, 16, 20 |
| Multi-tenant servers and databases, with row-level security | Supporting | 4 to 7, then every later step |
| Working with business data safely | Supporting | 5, 6, 7 |
| Retrieval: search by meaning, citations, abstaining and measuring | Direct | 8 |
| Interactive UI with MCP Apps | Direct | 9, 19 |
| Skills, Tasks and Events extensions | Supporting | 10, 14, 15 |
| Background and long-running work | Supporting | 11, 12, 13, 14 |
| Testing, evals and observability | Direct | 8, 17, 18 |
| Driving Claude Code to plan, build and review | Direct | every step |
| Turning a server into a product for clients | FDE | 20, 21 |

### How each step works

Every step is hands-on: you read the linked book lessons, then build the step yourself, deploy it, and test it in Claude and ChatGPT. A step is done only when it works in a real client, not when you have read about it.

Every step is built with Claude in your bootcamp's Claude project (see [How we work](#how-we-work-local-first-human-in-the-loop)). Each step's README has a **Build it with Claude Code** section with numbered instructions and copy-paste prompts:

1. Start a new thread for the step in your Claude project.
2. Let Claude propose a plan before it writes code.
3. Paste the prompts in order, and review what Claude proposes before approving it.
4. Review the changes, then commit and push them yourself in GitHub Desktop. Claude never commits.
5. Deploy and test in Claude and ChatGPT at the end of each step.
6. At the end of each milestone, record a short **project defence**: a walkthrough of your own code and one tenant-isolation decision, in your own words (see [SUBMITTING.md](SUBMITTING.md#10-defend-your-project)).

Each step's code lives in its own folder and starts from a copy of the previous step.

## Static and generative UI in Claude and ChatGPT

We test everything in claude.ai and Claude Desktop, and we also make it work in ChatGPT. To stay portable, every UI we build ships as a standard **[MCP App](https://modelcontextprotocol.io/seps/1865-mcp-apps-interactive-user-interfaces-for-mcp)**: an HTML/JS app that the host renders in a sandboxed frame in the chat. We use the standard MCP Apps APIs (`@modelcontextprotocol/ext-apps`) rather than any client-specific API, so one server works in both clients.

We use the three kinds of screen from book lesson [20.10 Should the AI design the screen?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/screens-the-ai-designs/):

| Kind of screen | What it means | Where |
| --- | --- | --- |
| Fixed | We build each React widget ahead of time. The model selects one from the user's request and the tool fills it with data | Step 9 |
| Approved parts | We build a library of small approved widgets. The model composes a screen from them, and the server checks every part and every field before it is shown | Steps 19 and 21 |
| Open | The model writes its own HTML | Not used, because it is hard to keep safe |

## Official documentation

The book teaches the ideas, and these official docs are the reference for the exact APIs. Each step's **Learn first** section also links the specific doc pages it needs.

**MCP**
- [Getting started](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro), the [specification](https://modelcontextprotocol.io/specification/2026-07-28) (version 2026-07-28) and its [key changes](https://modelcontextprotocol.io/specification/2026-07-28/changelog)
- [Extensions overview](https://modelcontextprotocol.io/extensions/overview) and [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix)
- [MCP Apps](https://modelcontextprotocol.io/extensions/apps/overview), [Authorization extension](https://modelcontextprotocol.io/extensions/auth/overview), [Skills over MCP](https://modelcontextprotocol.io/extensions/skills/overview), [Tasks](https://modelcontextprotocol.io/extensions/tasks/overview) and [Events](https://modelcontextprotocol.io/extensions/events/overview)
- [MCP Inspector](https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector)

**Cloudflare** ([all docs](https://developers.cloudflare.com/))
- [Workers](https://developers.cloudflare.com/workers/) and [Model Context Protocol on Cloudflare](https://developers.cloudflare.com/agents/model-context-protocol/)
- [Workers KV](https://developers.cloudflare.com/kv/), [D1](https://developers.cloudflare.com/d1/) and [Hyperdrive](https://developers.cloudflare.com/hyperdrive/)
- [Durable Objects](https://developers.cloudflare.com/durable-objects/), [Queues](https://developers.cloudflare.com/queues/) and [Workflows](https://developers.cloudflare.com/workflows/)

**Neon** ([all docs](https://neon.com/docs))
- [Architecture overview](https://neon.com/docs/introduction/architecture-overview), [connection pooling](https://neon.com/docs/connect/connection-pooling) and [branching](https://neon.com/docs/introduction/branching)

**Better Auth** ([all docs](https://better-auth.com/docs))
- [MCP plugin](https://better-auth.com/docs/plugins/mcp), [OAuth provider plugin](https://better-auth.com/docs/plugins/oauth-provider), [SSO plugin](https://better-auth.com/docs/plugins/sso) and [organization plugin](https://better-auth.com/docs/plugins/organization)

## License

MIT. See [LICENSE](LICENSE).
