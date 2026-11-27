# Step 11: Per-User State with Durable Objects

Some state can't wait a minute to be consistent and doesn't belong in the business database: a draft order a user builds over several tool calls, or a count of how many times they called a write tool this minute. Your MCP server keeps that kind of state in Cloudflare Durable Objects. A Durable Object is one named object (for example, one per user) with its own storage, and every request for that name reaches the same object, so reads and writes are strongly consistent. The objects are defined and bound inside the same Worker that serves `/mcp`, and the server's tools call them.

> 🏬 **Stockroom's ask:** Reps build an order over several messages, and one customer's script once hammered the write tools. Stockroom wants a draft order that survives between messages and an exact per-user rate limit.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

> **Note:** The book mentions Durable Objects only in passing (in the connectors field guide), not in a lesson yet. The lessons below give the ideas this step relies on. Use the [Durable Objects docs](https://developers.cloudflare.com/durable-objects/) for the feature itself.

- [17.4 What if a different copy answers the next call?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/stateless-servers/). Why the Worker keeps nothing in memory, and why state needs a shared place every copy can reach.
- [17.9 How do you charge for a connector?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/charging-for-a-connector/). A plan check on every call, which needs an exact per-user count.
- [18.11 Can one company's answer reach another by accident?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/one-order-slip-per-request/). Nothing on a shared note, and where call counts for rate limits belong.
- *Optional:* [19.9 What if the last call's company is still set?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/one-company-per-transaction/). Keeping one user's state from leaking into another's.

**Official docs.** Keep these open while you build:

- [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/). Naming, storage, consistency and alarms.
- [Cloudflare: rate limiting binding](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/). Cloudflare's built-in alternative for simple rate limits, to compare with your own.
- [Cloudflare: MCP handler API](https://developers.cloudflare.com/agents/model-context-protocol/apis/handler-api/). Why the stateless handler keeps no session, and where cross-request data should live instead (a Durable Object is one of the answers).

## Why this step matters

Client servers need limits and multi-call flows: "no more than 20 orders a minute", "build the order, review it, then submit". Workers KV is too slow to agree with itself for this, and Neon is the wrong place for short-lived drafts and counters. Durable Objects give each user their own consistent little store right next to your server. Your `/mcp` handler stays stateless, as Cloudflare's handler docs recommend: the Durable Object holds application state behind the verified user ID, not an MCP session.

## Why this technology in an MCP server: Cloudflare Durable Objects

- **What it is.** A Durable Object is a named instance of a class, such as one per user, with its own storage. Every request for that name reaches the same object, so its reads and writes are strongly consistent, and it can set alarms.
- **Why an MCP server uses it.** The Worker is stateless and KV can be briefly out of date, but some MCP state must be exact: a draft order built over several tool calls, a rate limit, a user's event subscriptions. A Durable Object gives each user that exact state right next to your server. Cloudflare's handler docs point to Durable Objects for exactly this: durable data behind an authenticated handle, while the MCP handler itself stays stateless.
- **What it solves.** Lost or conflicting state across tool calls, and limits that two parallel calls could both slip past.
- **Alternatives.** Rows in Postgres with locking (slower and adds database load), an external store such as Redis, or Cloudflare's built-in rate-limiting binding for simple limits.

## What you will learn

- Explain what a Durable Object is: one named instance per key, with its own storage and strongly consistent reads and writes
- Define a Durable Object class and bind it to your MCP server's Worker
- Route each signed-in user to their own object, and call it from tool handlers
- Build a multi-call flow (a draft order) whose state lives in the object
- Enforce a per-user rate limit for write tools, with a clear error the model can act on
- Use an alarm to clean up stale state, and know when Durable Objects are the wrong choice

## What you will build

A `UserState` Durable Object, one per signed-in user, that holds a draft order and a rate-limit counter. New tools `add_to_draft`, `view_draft`, `clear_draft` and `submit_draft` build an order over several calls, and `submit_draft` writes it to Neon through `create_order`. Every write tool checks the user's rate limit first, and an alarm clears drafts left untouched for a day.

## Key concepts

- **Durable Object**: A class whose instances each have a unique name, private storage and a single place where they run.
- **Strong consistency per key**: All requests for one name reach one object, so two calls can't both read a stale count.
- **Routing by ID**: Turning the signed-in user's ID into the object's name, so users never share an object.
- **Alarms**: A timer an object sets for itself, used here to expire stale drafts.
- **Right tool for the job**: Durable Objects for small per-key state, Neon for business records, KV for data that can be a little stale.

## Outline

1. Where state is allowed in a stateless server
2. Durable Objects in one picture
3. Defining and binding the `UserState` object
4. A draft order built over several tool calls
5. Per-user rate limits
6. Alarms for clean-up
7. A stateless handler with stateful objects behind it
8. Testing in Claude and ChatGPT

## Prerequisites

- [Step 10](../step-10-skills-over-mcp/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 10 code into `step-11-durable-objects/`.**

```text
Copy my step-10 code into step-11-durable-objects/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-11 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-11 from my step-10 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan the Durable Object.**

```text
Read the Cloudflare Durable Objects docs (https://developers.cloudflare.com/durable-objects/) and explain Durable Objects in plain words: naming, storage, consistency and alarms. Then plan a UserState Durable Object for this MCP server, one per signed-in user, holding a draft order and a rate-limit counter. Confirm that the /mcp handler stays stateless (createMcpHandler) and explain how keeping state in a Durable Object differs from the old McpAgent pattern of one Durable Object per MCP session.
```

**3. Create and bind the object.**

```text
Create the UserState Durable Object class, add its binding and migration to the wrangler config, and add a helper that gets the signed-in user's object with the step 7 tenantObjectName helper (org:<orgId>:user:<userId>). Explain why the name must come from the verified company and user IDs and never from tool input, and add a test that two users with the same user ID in different companies never share an object.
```

**4. Build the draft order tools.**

```text
Add add_to_draft, view_draft, clear_draft and submit_draft. Keep the draft in the user's UserState object. submit_draft should validate the draft, create the order in Neon with the existing create_order logic, and clear the draft only after the order is saved. Set the right tool annotations.
```

**5. Add a per-user rate limit.**

```text
Add a rate limit in UserState: at most 20 write-tool calls per user per minute. Check it at the start of every write tool, and when it's exceeded return a clear, recoverable error that tells the model when to try again.
```

**6. Clean up with an alarm.**

```text
Set an alarm in UserState that clears a draft left untouched for 24 hours. Explain what happens to the object when no requests arrive.
```

**7. Deploy and test.**

```text
Deploy, then give me prompts to try in Claude and ChatGPT that build a draft order over several messages, submit it, and trigger the rate limit. Show me where to see the Durable Object in the Cloudflare dashboard.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-11-durable-objects/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: every Durable Object is named from the company and user (`org:<orgId>:user:<userId>`), never from tool input, and the cross-tenant suite still passes
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-11-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-11-done`, live server URL, and screenshots of a draft order built over several messages and submitted, plus the rate-limit error, in **Claude** and **ChatGPT**

## References

- [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/)
- [Cloudflare: Model Context Protocol](https://developers.cloudflare.com/agents/model-context-protocol/)

---

← [Step 10: Skills over MCP](../step-10-skills-over-mcp/) · [Roadmap](../README.md#roadmap) · [Step 12: Background Work with Queues](../step-12-queues/) →
