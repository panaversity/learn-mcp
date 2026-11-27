# Step 12: Background Work with Queues

Some work is too big or too slow for one tool call but splits into many small, independent pieces: importing thousands of orders, sending notifications, recalculating summaries. Your MCP server hands that work to Cloudflare Queues. A tool puts messages on a queue and answers straight away, and the same Worker that serves `/mcp` consumes the messages in batches in the background, with retries and a dead-letter queue for messages that keep failing.

> 🏬 **Stockroom's ask:** New customers arrive with years of orders in a CSV file. Stockroom wants imports that run in the background, retry what fails and report bad rows, while the chat keeps answering.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

> **Note:** The book doesn't cover Cloudflare Queues yet. The lessons below give the ideas this step relies on: work that is safe to repeat, and errors that say what to do next. Use the [Queues docs](https://developers.cloudflare.com/queues/) for the feature itself.

- [17.4 What if a different copy answers the next call?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/stateless-servers/). Why background work can't live in the Worker's memory.
- [17.3 What should a tool say when it fails?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/errors-that-say-what-to-do-next/). Reporting rows that failed so the model and the user can fix them.
- [19.17 What if the database is slow or asleep?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/a-database-behind-a-stateless-server/). Writing to Neon from many short-lived consumers.
- [19.6 How do you change tables that are in use?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/changing-the-tables-safely/). Applying each change once, which is the same idea as an idempotent consumer.
- [14.8 Async and await](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/async-and-await/). Work that finishes after the reply.

**Official docs.** Keep these open while you build:

- [Cloudflare Queues](https://developers.cloudflare.com/queues/). Producers, consumers, batching, retries and dead-letter queues.
- [Cloudflare Workers: limits](https://developers.cloudflare.com/workers/platform/limits/). Why a big import can't run in one request.

## Why this step matters

Clients often arrive with data to load: a spreadsheet of last year's orders, a product catalog, a customer list. A tool that tries to insert thousands of rows in one call times out and leaves a half-finished import. A queue turns the job into many small messages that are processed in the background, retried when they fail and set aside when they keep failing, while the tool answers in a second.

## Why this technology in an MCP server: Cloudflare Queues

- **What it is.** A queue holds messages. Your tool (the producer) sends them and your Worker (the consumer) processes them later in batches, with retries and a dead-letter queue for messages that keep failing.
- **Why an MCP server uses it.** A tool call should answer in seconds, but some work (importing thousands of rows, sending notifications) takes much longer. A queue lets the tool answer at once and do the work in the background, at a pace the database can handle.
- **What it solves.** Timeouts and half-finished imports, lost work when something fails, and bursts that overwhelm Neon.
- **Alternatives.** `ctx.waitUntil` for short follow-up work (no retries, limited time), Workflows when the job has ordered steps (step 13), or external queues such as Amazon SQS.

## What you will learn

- Explain producers, consumers, batches and at-least-once delivery
- Create a queue and bind it to your MCP server's Worker as both producer and consumer
- Send messages from a tool and answer straight away with an import ID
- Consume messages in batches and write them to Neon idempotently, so a retried message never creates a duplicate
- Configure retries and a dead-letter queue, and report failed rows clearly
- Choose between a Queue and a Workflow (step 13) for a given job

## What you will build

A `bulk_import_orders` tool that accepts CSV data, validates its header, splits the rows into batches and puts them on an `order-imports` queue, then returns an import ID. The Worker's queue consumer inserts each batch into Neon for the signed-in user, keyed by import ID and row number so retries are safe. A `get_import_status` tool reports rows done, rows failed and why, read from an `imports` table in Neon.

> **Looking ahead: file uploads.** In this step the CSV travels as text inside the tool call, so it passes through the model's context and has to stay small. MCP's [File Uploads extension](https://github.com/modelcontextprotocol/experimental-ext-file-uploads) is an experimental draft (November 2026) in which the server gives the client an upload target, typically a pre-signed URL, and the client sends the file there directly over HTTPS, so the bytes never pass through the model. Anthropic has said Claude will support it soon, first with a few launch partners. Keep this step's size limit, and note in NOTES.md how `bulk_import_orders` would change once clients support uploads, for example a pre-signed upload to Cloudflare R2 under an `org:<orgId>:` key.

## Key concepts

- **Producer and consumer**: The tool sends messages, and the Worker's `queue()` handler processes them later.
- **Batches**: The consumer receives several messages at once, which keeps database round trips down.
- **At-least-once delivery**: A message can arrive more than once, so every write must be idempotent.
- **Retries and dead-letter queue**: Failed messages are retried, then moved aside for inspection instead of being lost.
- **Queue or Workflow**: Queues suit many independent pieces of work. A Workflow (step 13) suits one job with ordered steps, waits and a status to check.

## Outline

1. Work that doesn't fit in a tool call
2. Queues in one picture
3. Creating and binding the queue
4. Sending import batches from a tool
5. An idempotent consumer that writes to Neon
6. Retries, the dead-letter queue and failed rows
7. Reporting import status to the model
8. Testing in Claude and ChatGPT

## Prerequisites

- [Step 11](../step-11-durable-objects/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 11 code into `step-12-queues/`.**

```text
Copy my step-11 code into step-12-queues/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-12 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-12 from my step-11 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan the import.**

```text
Read the Cloudflare Queues docs (https://developers.cloudflare.com/queues/) and explain producers, consumers, batching, at-least-once delivery, retries and dead-letter queues in plain words. Then plan a bulk order import for this MCP server: the tools, the queue and its dead-letter queue, the message shape, and how the consumer stays idempotent.
```

**3. Create and bind the queues.**

```text
Give me the Wrangler commands to create an order-imports queue and an order-imports-dlq dead-letter queue. Bind order-imports to this Worker as both producer and consumer, with sensible batch size, retries and the dead-letter queue, and explain each setting.
```

**4. Send work from a tool.**

```text
Add bulk_import_orders. It takes CSV text, checks the header and size limits, records a new import in an imports table in Neon for the signed-in user's company, splits the rows into batches of 50 and sends them to the queue, each message wrapped in the step 7 tenant envelope, then returns the import ID right away. Set the right tool annotations.
```

**5. Consume it safely.**

```text
Add the Worker's queue consumer. For each batch, validate the rows, read the company and user from the message's tenant envelope, check that the import belongs to that company, re-check in the database that the user who started it is still a member of that company with a role that may import orders (stop the import and mark it refused, with a reason, if not), insert valid orders through the step 7 tenant router (so row-level security applies), and record failed rows with a reason. Use the import ID and row number so a retried message never creates a duplicate order. Explain which errors should retry the message and which should not.
```

**6. Report status.**

```text
Add get_import_status, which returns rows done, rows failed with reasons, and whether the import is finished, for the signed-in user's imports only. Keep the response small, summarizing failures if there are many.
```

**7. Deploy and test.**

```text
Deploy, then give me a sample CSV with a few deliberately bad rows and prompts to try in Claude and ChatGPT that import it and check on it until it finishes. Show me how to look at the queue and the dead-letter queue in the Cloudflare dashboard.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-12-queues/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: every queue message carries the tenant envelope, the consumer re-checks it before writing, and an import can't write into another company
- [ ] Authority at run time: a test removes the user mid-import, and the remaining batches are refused instead of written
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-12-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-12-done`, live server URL, and screenshots of a bulk import being started and its status (including failed rows) in **Claude** and **ChatGPT**

## References

- [Cloudflare Queues](https://developers.cloudflare.com/queues/)

---

← [Step 11: Per-User State with Durable Objects](../step-11-durable-objects/) · [Roadmap](../README.md#roadmap) · [Step 13: Durable Jobs with Cloudflare Workflows](../step-13-cloudflare-workflows/) →
