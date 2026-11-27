# Step 07: Multi-Tenant MCP Servers and Databases

Almost every server you build for a client, and every product you sell, serves many companies from one deployment. Each company (a *tenant*) has its own people, settings and data, and no tenant may ever see another's. Steps 4 to 6 gave every user a company, put an `org_id` on every row and turned on row-level security. This step makes multi-tenancy the architecture of the whole server: you choose an isolation model for each kind of customer, carry one tenant context through every tool call, give each company its own settings and sign-in, build tenant-scoped helpers that every later step reuses for keys, Durable Objects, queue messages, Workflow runs, tasks and events, and prove isolation with a cross-tenant test suite.

> 🏬 **Stockroom's ask:** Stockroom is signing many small distributors on a shared plan and one large retailer that insists on its own database and its own sign-in. One deployment must serve them all, and a single leak between them would end Stockroom's business.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [18.9 How does one connector serve many companies?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/what-a-tenant-is/). A product serves many companies, and each company's people and data stay apart.
- [18.10 Which company is this request for?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/where-the-tenant-comes-from/). The company comes from the verified token, never from what the model sends.
- [18.12 What can still leak after the token is checked?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/what-else-carries-the-company/). Keys, caches, logs and background work must carry the company too.
- [18.14 How does a company come in, change and leave?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/a-companys-life/). Creating a company, inviting and removing members, and closing it.
- [19.7 Shared tables, or a database per company?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/three-ways-to-keep-data-apart/). Shared tables, a schema per company or a database per company, and when to use each.
- [19.9 What if the last call's company is still set?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/one-company-per-transaction/). Setting the company for one transaction only, so a pooled connection never leaks it.
- [19.11 How does each company keep its own rules and data?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/each-companys-own-rules-and-data/). Per-company settings and rules without per-company code.
- [19.12 How do you prove the data stays apart?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/proving-the-data-stays-apart/). Tests that try to cross the line, and must fail.
- [Field guide F9, The multi-tenant sign-in checklist](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/field-guide-securing-connectors/#f9-the-multi-tenant-sign-in-checklist) and [F7, The multi-tenant data checklist](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/field-guide-business-data-and-retrieval/#f7-the-multi-tenant-data-checklist). The checks you will turn into tests.
- *Optional:* [18.13 How do you prove companies stay apart?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/proving-the-sign-in-keeps-companies-apart/) and [19.10 What can still leak when the rows are safe?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/where-the-guarantee-stops/). What row-level security does not cover.

**Official docs.** Keep these open while you build:

- [Neon: multitenancy](https://neon.com/docs/guides/multitenancy). Shared tables versus a database or project per tenant on Neon, with the trade-offs.
- [PostgreSQL: row security policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html). The shared-schema guard you added in step 6.
- [PostgreSQL: schemas](https://www.postgresql.org/docs/current/ddl-schemas.html). What a schema per tenant would mean.
- [Better Auth: organization plugin](https://better-auth.com/docs/plugins/organization). Companies, members, roles, invitations and the active company.
- [Better Auth: SSO plugin](https://better-auth.com/docs/plugins/sso). Linking a company's own identity provider to its organization.
- [Cloudflare Hyperdrive with Neon](https://developers.cloudflare.com/hyperdrive/examples/connect-to-postgres/postgres-database-providers/neon/). One Hyperdrive config per database, including a dedicated tenant database.
- [Cloudflare Workers KV](https://developers.cloudflare.com/kv/). Where per-company settings are cached.

## Why this step matters

Clients rarely want a server for one person. A distributor wants one for every branch, a SaaS founder wants one for every customer, and an enterprise wants its own sign-in and sometimes its own database. Getting tenancy wrong is the most expensive bug an FDE can ship: one company seeing another's orders ends the contract. Getting it right early, with one tenant context and helpers every later feature must use, means Durable Objects, queues, workflows, tasks, events and billing in later steps are multi-tenant from the first line.

## Why this technology in an MCP server: multi-tenant architecture

- **What it is.** One deployment that serves many companies, with a chosen isolation model for their data (shared tables with row-level security, a schema per company, or a database per company), a tenant context taken from the verified token, and tenant-scoped names for everything the server stores or sends.
- **Why an MCP server uses it.** An MCP server acts for signed-in people at many companies, and the model can be steered by any text it reads. The company must therefore come from the token, never from tool input, and every store and message must carry it, so no tool call, background job or event can cross from one company to another.
- **What it solves.** Data leaks between customers, per-customer forks of the code, enterprise customers who need their own sign-in or their own database, and noisy neighbours that slow everyone down.
- **Alternatives.** A separate deployment per customer (simple isolation, but every update ships many times and costs grow per customer), or a database per tenant for everyone (strong isolation, harder migrations and higher cost). Most products use shared tables by default and a dedicated database only for customers who pay or must comply.

## What you will learn

- Compare the three isolation models (shared schema with row-level security, schema per tenant, database or Neon project per tenant) on isolation, cost, migrations, noisy neighbours and compliance, and choose one per kind of customer
- Resolve one tenant context (company ID, user ID, role, isolation tier) from the verified token at the start of every request, and pass it to every tool, resource and prompt
- Route each company's queries to the shared database or to its own dedicated database, behind one data layer the tools don't need to know about
- Give each company its own settings and limits, cached in Workers KV under company-scoped keys
- Link a company's own identity provider to its organization, so its staff land in the right company
- Build tenant-scoped helpers for KV keys, Durable Object names and a tenant envelope for queue messages, Workflow runs, tasks and events, used in every later step
- Handle a company's life: create, invite, work in another company by reconnecting, remove a member and close a company
- Prove isolation with a cross-tenant test suite that tries every way across the line

## What you will build

The step 6 sales server, made multi-tenant by design:

- `TENANCY.md`: a decision record comparing the three isolation models, with the server's choice: shared schema and row-level security for most companies, and a dedicated Neon database for an "enterprise" tier.
- A `tenant` module that resolves `{ orgId, userId, role, tier }` once per request from the token and hands it to every handler, plus helpers: `tenantKey()` for KV, `tenantObjectName()` for Durable Objects, and a `TenantEnvelope` type for queue messages, Workflow params, tasks and events.
- A tenant router: shared-tier companies go through the shared Hyperdrive config with row-level security, and an enterprise company goes through its own Hyperdrive config to its own Neon database, with the same migrations applied to both.
- Company tools: `list_my_companies`, `get_company_settings`, `update_company_settings` (admin only, cached in KV under `org:<orgId>:settings`), `invite_member` and `remove_member` (admin only).
- Optional: per-company single sign-on, linking one company's OIDC provider to its organization.
- `tests/tenancy/`: a cross-tenant suite with two companies (one shared, one dedicated) that tries guessed order IDs, resource URIs, KV keys, a tool with its `WHERE` clause removed, switching to a company you don't belong to, and access after removal. Every attempt must fail.

## Key concepts

- **Tenant**: A customer company. One server serves many tenants.
- **Isolation model**: How tenants' data is kept apart: shared tables with row-level security (pool), a schema per tenant (bridge) or a database per tenant (silo).
- **Tenant context**: The company, user, role and tier resolved from the verified token once per request, never from tool input.
- **Tenant router**: The one place that turns a tenant context into the right database connection.
- **Tenant-scoped key**: A name that starts with the company, such as `org:<orgId>:settings`, so two companies can never share an entry.
- **Tenant envelope**: The company and user carried with every background message, Workflow run, task and event, and re-checked when it is handled.
- **Noisy neighbour**: One tenant's heavy use slowing everyone, handled with per-company limits and, if needed, a dedicated tier.
- **Cross-tenant test**: A test that acts as one company and tries to reach another's data, and passes only if it fails.

## Outline

1. Tenants, and the three ways to keep their data apart
2. Choosing an isolation model per kind of customer
3. One tenant context per request
4. A tenant router: shared and dedicated databases
5. Per-company settings, limits and sign-in
6. Tenant-scoped helpers for keys, objects, messages, runs and events
7. A company's life: invite, switch, remove, close
8. Proving isolation with cross-tenant tests
9. Testing in Claude and ChatGPT

## Prerequisites

- [Step 6](../step-06-neon-production/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 6 code into `step-07-multi-tenancy/`.**

```text
Copy my step-06 code into step-07-multi-tenancy/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-07 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-07 from my step-06 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and write the tenancy decision record.**

```text
Read the Neon multitenancy guide and the book lessons linked in this step. Compare shared schema with row-level security, schema per tenant, and database (or Neon project) per tenant on isolation, cost, migrations, noisy neighbours, backups and compliance. Recommend shared schema with RLS for most companies and a dedicated Neon database for an enterprise tier, and write the comparison and decision to step-07-multi-tenancy/TENANCY.md. Then list every place this server touches company data (tools, resources, prompts, KV, logs) and plan how each gets the tenant context. Don't change code yet.
```

**3. One tenant context per request.** Book lesson 18.14 notes that a removed person's token keeps working until it runs out (up to 10 minutes in the book's example). This step closes that gap by checking membership and role against the database on every request.

```text
Add a tenant module that resolves { orgId, userId, role, tier } once per request and passes it to every tool, resource and prompt handler. The company comes only from the verified token: the user picks it when they connect, and a token speaks for one company for its whole life. On every request, check that the user is still a member of that company and read their current role from the membership table, so a removed member or a demoted admin loses access on the next call. Membership decides which company's data a call may touch; the role decides what the call may do there. Add negative tests: a removed member's token, a demoted admin calling an admin tool, and a tool argument that names another company. No tool may accept an org_id or user_id as input. Add helpers tenantKey(ctx, ...parts) for KV keys, tenantObjectName(ctx) for Durable Object names, and a TenantEnvelope type ({ orgId, userId, requestId }) for later queue messages, Workflow params, tasks and events, with unit tests.
```

**4. Route shared and dedicated tenants.**

```text
Add a tenants table (org_id, tier, database binding name) and a tenant router that returns the right database client: shared-tier companies use the shared Hyperdrive config with the company set for one transaction (as in step 6), and an enterprise company uses its own Hyperdrive config pointing at its own Neon database. Create that dedicated database, apply the same numbered migrations to both, and add an npm script that runs migrations across every tenant database. Make sure no tool code knows which tier it is using.
```

**5. Company settings, members and switching.**

```text
Add list_my_companies (which tells the user to reconnect to work in another company; no tool changes the company a token speaks for), get_company_settings and an admin-only update_company_settings (currency, fiscal year start, sales threshold), stored in Neon with RLS and cached in Workers KV under tenantKey(ctx, "settings"), with the cache cleared on update. Add admin-only invite_member and remove_member using Better Auth's organization plugin, and make a removed member lose access on their next call. Set the right annotations and return clear 403-style errors for members.
```

**6. Optional: per-company single sign-on.**

```text
Using Better Auth's SSO plugin, link one company's OIDC identity provider to its organization, so staff who sign in through it land in that company with the member role. Explain how this differs from the step 4 SSO setup and what happens when someone belongs to two companies.
```

**7. Prove isolation.**

```text
Create tests/tenancy/ with two companies, one shared-tier and one dedicated, and tests that act as company A and try to reach company B through every tool, resource and prompt: guessed order IDs, note and resource URIs, KV keys, a query with its WHERE clause removed (RLS must still block it), switching to a company the user doesn't belong to, and access after removal. Every attempt must fail, and the suite must run with npm test. Record the results in TENANCY.md.
```

**8. Deploy and test.**

```text
Deploy, then give me prompts to try in Claude and ChatGPT as two users in different companies: list orders, change settings as a member (refused) and as an admin, reconnect as a different company, and ask for another company's order by ID (refused). Show me how to confirm which database each company's queries reached.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)
- The cross-tenant suite in `tests/tenancy/`

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-07-multi-tenancy/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only, and no Neon connection strings)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] TENANCY.md has the isolation-model comparison, your decision and the test results
- [ ] Tenant isolation: the cross-tenant suite passes for a shared-tier and a dedicated-tier company, and no tool accepts a company or user ID as input
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-07-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-07-done`, live server URL, and screenshots of two users in different companies each seeing only their own data, and of a cross-company request being refused, in **Claude** and **ChatGPT**

## References

- [Neon: multitenancy](https://neon.com/docs/guides/multitenancy)
- [Better Auth: organization plugin](https://better-auth.com/docs/plugins/organization)
- [PostgreSQL: row security policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)

---

← [Step 06: Production Data Tools on Neon: Hyperdrive, Writes and Row-Level Security](../step-06-neon-production/) · [Roadmap](../README.md#roadmap) · [Step 08: Search by Meaning with pgvector: Answers That Cite or Abstain](../step-08-semantic-search-pgvector/) →
