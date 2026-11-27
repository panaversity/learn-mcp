# Step 05: Relational Databases and Normal Forms with Neon

Give your remote MCP server a real business database, and learn to design one well while you do it. The server connects to Neon Postgres, its migrations create a sales schema that you take from a messy spreadsheet through the first three normal forms (1NF, 2NF and 3NF), and new read-only tools let Claude and ChatGPT explore the schema, answer business questions from it and show why normalization matters. Auth stays in D1, and the KV notes tools from steps 2 to 4 are replaced by sales data.

> 🏬 **Stockroom's ask:** Notes aren't enough. Stockroom's customers want to ask about their sales ("who bought the most last month?"), and their history arrives as a messy spreadsheet. You design a proper sales database in Neon that the model can answer from.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [19.1 Why move the data out of the code?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/why-a-database-now/). Why business data moves into a database server, separate from the sign-in data.
- [19.2 What are tables, rows and keys?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/tables-rows-and-keys-from-zero/). Tables, columns, primary and foreign keys, and the three shapes of query.
- [19.3 How do you read SQL an agent wrote?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/reading-sql/). Reading SQL before you trust it, including SQL a coding agent writes.
- [19.5 Where does each fact belong?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/third-normal-form-in-detail/). Normal forms from zero, then 3NF in detail, and when to keep a copy on purpose.
- [Field guide F5, The SQL in detail](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/field-guide-business-data-and-retrieval/#f5-the-sql-in-detail). The SQL reference, with first, second and third normal form side by side.
- [Field guide F1, Postgres from zero](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/field-guide-business-data-and-retrieval/#f1-postgres-from-zero) and [F2, Neon from zero](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/field-guide-business-data-and-retrieval/#f2-neon-from-zero). Setting up Postgres on Neon.
- [19.6 How do you change tables that are in use?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/changing-the-tables-safely/). Numbered migrations, applied once and in order.
- *Optional:* [18.8 How do you read the sign-in server's records?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/tables-rows-and-relationships/). The same ideas at reading depth, if you want a gentler start.

**Official docs.** Keep these open while you build:

- [Neon: architecture overview](https://neon.com/docs/introduction/architecture-overview). What serverless Postgres is.
- [Neon: branching](https://neon.com/docs/introduction/branching). A throwaway copy of your database for trying designs and migrations.
- [Neon: roles](https://neon.com/docs/manage/roles). The database users the server connects as.
- [Cloudflare Workers: secrets](https://developers.cloudflare.com/workers/configuration/secrets/). Keeping the Neon connection string out of git.
- [PostgreSQL: data definition](https://www.postgresql.org/docs/current/ddl.html). Tables, constraints and keys in Postgres itself.
- [PostgreSQL: queries](https://www.postgresql.org/docs/current/queries.html). `SELECT`, joins and grouping.

## Why this step matters

Almost every client server you build as an FDE has a database behind it, and a badly designed one causes bugs no tool description can fix: a customer's city that differs between orders, a product renamed in one place but not another, totals that don't add up. Normal forms are the simple rules that keep each fact in one place. Designing the schema inside your MCP server, and watching the model answer questions from it, shows you exactly how the design shapes what your tools can do.

## Why this technology in an MCP server: A relational database (Neon Postgres) and normal forms

- **What it is.** A relational database stores data in tables linked by keys and answers questions with SQL. Neon is serverless Postgres with database branching and a driver that works from Workers. Normal forms are the design rules that keep each fact in one place.
- **Why an MCP server uses it.** Most client servers exist to let a model work with a business's data. The questions users ask ("top products last quarter", "best customers") are joins, filters and totals, which SQL answers exactly. A normalized schema means every tool returns the same, consistent answer, and fixed SQL behind each tool keeps the model from running queries you didn't write.
- **What it solves.** A flat spreadsheet repeats facts, so the same customer can have two addresses and totals drift. Normalization removes those contradictions before a model ever reads the data.
- **Alternatives.** D1 (SQLite) works for smaller data, and we keep it for sign-in. Supabase or another hosted Postgres works the same way. Document databases are weaker at the joins and totals business questions need.

## What you will learn

- Explain tables, rows, primary keys and foreign keys, and how tables relate (one-to-many, many-to-many)
- Connect your Worker to Neon with a driver that works on Cloudflare, with the connection string as a Wrangler secret
- Apply **first normal form** (one value per cell, no repeating groups)
- Apply **second normal form** (every fact depends on the whole key)
- Apply **third normal form** (no fact depends on another non-key column)
- Know when to keep a copy on purpose, such as the price at the time of sale
- Write the design as numbered migrations, with constraints that enforce the rules, run from the server project
- Expose the schema through safe, read-only tools with fixed SQL, never SQL written by the model

## What you will build

The step 4 server, now backed by Neon. Its migrations load a flat spreadsheet of a year's sales (one row per order line, repeating the customer's details and the product's name, category and price) into a `staging` schema, then create the normalized 3NF design: `customers`, `categories`, `products`, `orders` and `order_items`, each row owned by a company (`org_id`, the tenant from step 4). Three read-only tools sit on top:

- `describe_schema`: the tables, columns and keys, so the model can explain the design.
- `answer_business_question`: one of ten named questions (monthly revenue, top products, best customers and so on), each a fixed, parameterized query scoped to the signed-in user's company.
- `compare_designs`: runs the same question against the flat table and the normalized tables, and shows the anomalies the flat design causes.

## Key concepts

- **Primary key**: The column (or columns) that identifies one row, such as `order_id`.
- **Foreign key**: A column that points to a row in another table, such as `orders.customer_id`.
- **First normal form (1NF)**: Each cell holds one value, and there are no repeating groups like "product1, product2".
- **Second normal form (2NF)**: In a table with a composite key, every other column depends on the whole key, not part of it.
- **Third normal form (3NF)**: No column depends on another non-key column, so each fact is stored once, in the table it describes.
- **A copy on purpose**: Storing `unit_price` on each order line breaks 3NF deliberately, because the sale price must not change when the catalog price does.
- **Migrations**: Numbered SQL files that change the schema once, in order, so every database ends up the same.
- **Fixed SQL behind tools**: Tools run queries you wrote and reviewed, with parameters, so the model can never run arbitrary SQL.

## Outline

1. Why a relational database, and connecting the Worker to Neon
2. Tables, rows and keys
3. The flat sales spreadsheet and its anomalies
4. First, second and third normal form
5. The copy we keep on purpose
6. Constraints, migrations and seed data
7. Read-only tools over the schema
8. Testing in Claude and ChatGPT

## Prerequisites

- [Step 4](../step-04-auth/)
- The tools and accounts listed in the [main README](../README.md#prerequisites), including a Neon account

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)). In this step, ask Claude to explain every SQL statement, and predict each result before you run it.

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 4 code into `step-05-relational-databases/`.**

```text
Copy my step-04 code into step-05-relational-databases/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-05 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds.
```

**2. Create a Neon project in the Neon console, then switch to plan mode and plan the change.**

```text
Plan how to add Neon Postgres to this MCP server as its business database, keeping D1 for auth only and replacing the KV notes tools (and the NOTES binding) with sales data. Recommend a Postgres driver that works on Cloudflare Workers and a simple migration runner that lives in this project, and explain why. Create a Neon branch named step-05 for this step, and a dev branch from it to experiment on, so each later step can get its own branch and never break this one while it is graded.
```

**3. Connect the server to Neon.**

```text
Store the Neon connection string as a Wrangler secret (and in .dev.vars for local runs), connect to Neon from the Worker, and add an npm script that applies the numbered SQL files in db/migrations/ in order and records which ones ran. Explain tables, rows, primary keys and foreign keys using what we create.
```

**4. Load the flat spreadsheet.**

```text
Write the first migration: a staging.flat_sales table with a year of realistic sales, one row per order line, repeating the customer's name, email and city and the product's name, category and price on every row, with a few rows that put several products in one cell. Run it on my dev branch, then show me, with queries, an update anomaly, an insert anomaly and a delete anomaly in this table.
```

**5. Normalize to 1NF, 2NF and 3NF.**

```text
Walk me through normalizing the flat table one form at a time. For 1NF, 2NF and 3NF in turn: state the rule, point to the exact columns that break it, and write the next migration that creates the new tables and moves the data. Keep unit_price on order_items as a copy on purpose and explain why. End with customers, categories, products, orders and order_items, each row owned by an org_id (the company from step 4), with the company in every key, as book lesson 19.7 teaches (for example a primary key of (org_id, order_id) and foreign keys that include org_id, so no row can point at another company's row), plus NOT NULL, CHECK and UNIQUE constraints. Load the sample data into a demo company, and add a small seed script that copies it into my own company so my signed-in account sees it. Record the walkthrough and a Mermaid ER diagram in step-05-relational-databases/NOTES.md.
```

**6. Add the read-only tools.**

```text
Add three read-only tools, each scoped to the signed-in user's company: describe_schema (tables, columns and keys), answer_business_question (an enum of ten named questions such as monthly_revenue and top_products, each a fixed parameterized query) and compare_designs (runs one question against staging.flat_sales and the normalized tables and reports the differences and anomalies). Never accept SQL from the model. Set readOnlyHint and write descriptions that tell the model when to use each tool.
```

**7. Deploy and test.**

```text
Apply the migrations to a fresh Neon branch, set the secret, deploy, and give me prompts to try in Claude and ChatGPT: one that asks the model to explain the schema, three business questions, and one that asks why the old spreadsheet gave a customer two different cities.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-05-relational-databases/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only, and no Neon connection string)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] The migrations run cleanly on a fresh Neon branch
- [ ] NOTES.md has the 1NF, 2NF and 3NF walkthrough and the ER diagram
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-05-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-05-done`, live server URL, and screenshots of the model explaining your schema and answering a business question in **Claude** and **ChatGPT**

## References

- [Neon documentation](https://neon.com/docs)
- [PostgreSQL documentation](https://www.postgresql.org/docs/current/)

---

← [Step 04: Adding Authorization with Better Auth and D1](../step-04-auth/) · [Roadmap](../README.md#roadmap) · [Step 06: Production Data Tools on Neon: Hyperdrive, Writes and Row-Level Security](../step-06-neon-production/) →
