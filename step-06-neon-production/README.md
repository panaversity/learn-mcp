# Step 06: Production Data Tools on Neon: Hyperdrive, Writes and Row-Level Security

> 🚧 **Under development.** This step is being written. The plan below shows what it will cover, and the code and instructions are coming soon.

Your server already reads a well-designed schema from Neon. Now make its data access ready for real clients: fast pooled connections through Cloudflare Hyperdrive, a Neon branch per environment, list tools that page and summarize, a write tool that validates and runs in a transaction, and row-level security so the database itself keeps each company's rows apart.

> 🏬 **Stockroom's ask:** Stockroom's first paying customers are coming on. Their staff will page through thousands of orders and record new ones, and they must never see another customer's rows, even if one of your queries has a bug.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [19.17 What if the database is slow or asleep?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/a-database-behind-a-stateless-server/). Connecting a Worker to Postgres without exhausting connections.
- [19.8 What if a query forgets its company filter?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/row-level-security/). Letting the database enforce whose rows a query may see.
- [19.9 What if the last call's company is still set?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/one-company-per-transaction/). Setting the owner for one transaction only, so a pooled connection never leaks it.
- [Field guide F3, Hyperdrive from zero](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/field-guide-business-data-and-retrieval/#f3-hyperdrive-from-zero). Pooling Postgres connections for Workers.
- [19.10 What can still leak when the rows are safe?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/where-the-guarantee-stops/). Seven things that happen outside row-level security (keys, error messages, roles that skip it, exports, caches, logs) and the rule that closes each.
- [19.12 How do you prove the data stays apart?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/proving-the-data-stays-apart/). A data matrix that tries every table from another company, with planted bugs it must catch.

**Official docs.** Keep these open while you build:

- [Cloudflare Hyperdrive with Neon](https://developers.cloudflare.com/hyperdrive/examples/connect-to-postgres/postgres-database-providers/neon/). Setting up Hyperdrive for a Neon database.
- [Neon: connection pooling](https://neon.com/docs/connect/connection-pooling). How Neon pools connections, and when to use it.
- [Neon: branching](https://neon.com/docs/introduction/branching). A database branch per environment.
- [Neon: roles](https://neon.com/docs/manage/roles). A least-privilege role for the server.
- [PostgreSQL: row security policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html). How Postgres row-level security works.

## Why this step matters

A schema that works for one person testing is not yet safe for a client. Real users bring many concurrent requests, large result sets, writes that must not half-succeed, and the risk that one bug in a `WHERE` clause shows one user another user's orders. This step adds the production habits that every later step (widgets, imports, workflows and reports) relies on.

## Why this technology in an MCP server: Hyperdrive, least-privilege roles and row-level security

- **What it is.** Hyperdrive pools and speeds up Postgres connections from Workers. A least-privilege role can only do what the server needs. Row-level security (RLS) makes Postgres itself filter every query to the current company's rows.
- **Why an MCP server uses it.** An MCP server answers many short, concurrent tool calls from many users, and each call may open a database connection. Pooling keeps that fast and stops Neon running out of connections. Because tools act on behalf of a signed-in user, the database should enforce that user's boundary too, not just your code.
- **What it solves.** Slow or failing calls under load, oversized tool results that overflow the model's context (solved with pagination and summaries), half-finished writes (solved with transactions), and one missing `WHERE` clause leaking another user's orders (solved with RLS).
- **Alternatives.** Neon's own connection pooler or its HTTP driver without Hyperdrive, and filtering only in application code without RLS, which works until the first bug.

## What you will learn

- Connect through Cloudflare Hyperdrive, and explain why pooling matters for Workers
- Use a separate Neon branch for development and production
- Use two database roles: an owner role that runs migrations, and a connector role that can't change the schema and has no `BYPASSRLS`
- Build list tools with filters and cursor pagination, and summary tools that fit the model's context
- Build a write tool that validates input and runs in a transaction
- Force row-level security on every business table, with `USING` (which rows a query may see) and `WITH CHECK` (which rows it may write), failing closed when no company is set
- Set the company with `set_config('app.tenant_id', $1, true)` inside each tool call's transaction, from the token only (book lesson 19.9)
- Close the gaps row-level security doesn't cover: the company in every key, identical "not found" errors, no `SECURITY DEFINER` functions, Hyperdrive query caching off, and no rows in logs (book lesson 19.10)
- Prove it with a data matrix that tries every table from another company, plus planted bugs it must catch

## What you will build

The step 5 sales server, production-ready: `list_orders` (filters and cursor pagination), `get_sales_summary` (totals by month, product or customer) and `create_order` (validated, transactional and scoped to the signed-in user), all running through Hyperdrive as a least-privilege role, with row-level security policies on every business table.

## Key concepts

- **Connection pooling**: Reusing a few database connections across many short-lived Worker requests.
- **Branch per environment**: Development and production each get their own Neon branch, so experiments never touch real data.
- **Least privilege**: The server's role can read and write business rows, but can't change the schema.
- **Transactions**: A group of writes that all succeed or all fail.
- **Row-level security**: Database policies that filter every query to the current company's rows, as a second guard behind your tool code.
- **Forced RLS**: `FORCE ROW LEVEL SECURITY` makes the policies apply even to the table's owner, so no role you connect as can skip them by accident.
- **`USING` and `WITH CHECK`**: `USING` hides other companies' rows from reads, updates and deletes; `WITH CHECK` refuses writing a row for another company.
- **Failing closed**: `current_setting('app.tenant_id', true)` returns null when no company is set, so a query with no company sees no rows.
- **Data matrix**: A test that acts as one company against every table and every route, and must find nothing of another company's.
- **Context budgeting**: Return pages and summaries, not whole tables.

## Planned outline

1. From "it works" to production data access
2. Hyperdrive and pooled connections
3. Branches per environment and a least-privilege role
4. List tools with pagination
5. Summary tools
6. A transactional write tool
7. Row-level security
8. Testing in Claude and ChatGPT

## Prerequisites

- [Step 5](../step-05-relational-databases/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

> These prompts are a draft and will be refined as the step is developed.

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 5 code into `step-06-neon-production/`.**

```text
Copy my step-05 code into step-06-neon-production/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, submission/, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-06 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-06 from my step-05 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan the production changes.**

```text
Read the Neon and Hyperdrive docs linked in this step. Plan how to make this server's data access production-ready: Hyperdrive in front of Neon, a Neon branch for development and one for production, a least-privilege database role for the server, paginated list tools, a transactional write tool and row-level security. Show me the plan before writing code.
```

**3. Add Hyperdrive, branches and a least-privilege role.**

```text
Use this step's step-06 Neon branch as its production branch, create a dev branch from it, and create a role for the server that can read and write the business tables but not change the schema. Set up Hyperdrive in front of the production branch, bind it in the wrangler config, and keep using the dev branch locally. Explain what pooling changes. Use two roles: the owner role runs migrations, and the connector role the Worker uses can read and write business rows but can't change the schema and has no BYPASSRLS. Turn Hyperdrive's query caching off for this database, and explain why a cache that doesn't see the company setting could hand one company another's rows (book lesson 19.9).
```

**4. Build the list and summary tools.**

```text
Add list_orders, with filters (date range, customer, product) and cursor pagination, and get_sales_summary, with totals by month, product or customer. Reuse the queries behind answer_business_question where they fit. Keep responses small enough for the model's context.
```

**5. Build a write tool.**

```text
Add create_order with strict validation. Insert the order and its order_items in one transaction, copying the current product price into unit_price as designed in step 5. Set the right tool annotations, and return a clear, recoverable error when validation fails.
```

**6. Add row-level security.**

```text
Add row-level security to every business table, as book lessons 19.8 and 19.9 teach: ENABLE and FORCE row level security, and a policy for the connector role with USING (org_id = current_setting('app.tenant_id', true)) and the same WITH CHECK, so a query with no company set sees nothing. Add one forTenant(ctx, fn) helper that opens a transaction, calls set_config('app.tenant_id', $1, true) with the company from the verified token as a parameter, runs the tool's queries and commits, and make every tool use it. Then close the gaps from lesson 19.10: make every "not found" identical whether the row is missing or belongs to another company, make sure the connector role can't reach a SECURITY DEFINER function, and keep rows out of logs and errors. Finally add a data matrix test (lesson 19.12) that acts as one company against every table and tool, and plant three bugs (a tool with no WHERE clause, a policy without WITH CHECK, a query outside forTenant) that the matrix must catch before you remove them.
```

**7. Deploy and test.**

```text
Run the migrations against the production branch, deploy, and give me prompts to try in Claude and ChatGPT that page through orders, ask for summaries and create an order. Show me how to confirm the requests went through Hyperdrive.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (developer mode)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-06-neon-production/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Row-level security is forced on every business table, and the data matrix passes and caught each planted bug
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (developer mode)
- [ ] Screenshots of paging through orders and creating an order in **Claude** and **ChatGPT** are saved in `step-06-neon-production/submission/`, with anything private cropped out
- [ ] Tagged `step-06-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the step form: your fork's URL, this step and its live server URL (see [SUBMITTING.md](../SUBMITTING.md#8-submit-the-step))

## References

- [Neon documentation](https://neon.com/docs)

---

← [Step 05: Relational Databases and Normal Forms with Neon](../step-05-relational-databases/) · [Roadmap](../README.md#roadmap) · [Step 07: Multi-Tenant MCP Servers and Databases](../step-07-multi-tenancy/) →
