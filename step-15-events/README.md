# Step 15: Events

Let the server tell the client when something happens, such as a new order or a crossed threshold, using the MCP Events extension.

> 🏬 **Stockroom's ask:** Managers want to hear when something happens (a new order, a monthly target crossed) instead of asking every hour. Stockroom wants events that the AI apps its customers use can react to.

> **Note:** Events (`io.modelcontextprotocol/events`) is a Beta extension with three ways to deliver an event: the client polls, the server pushes, or the server calls a webhook. MCP's maintainers say they don't yet know which will win, so plan for all three. As of November 2026 the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix) lists no Events support in Claude yet and webhook-only support in ChatGPT. Check it before you test, and keep the `list_recent_events` fallback.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

> **Note:** The book doesn't cover the Events extension yet (the Chapter 20 [field guide F0](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/field-guide-mcp-apps/#f0-the-extensions-and-versions-this-chapter-was-checked-against) lists it as "Not covered"). The lessons below are the nearest background. Rely on the [MCP Events docs](https://modelcontextprotocol.io/extensions/events/overview) for the extension itself.

- [Field guide: change notifications (`subscriptions/listen`, `listChanged`)](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#change-notifications). How servers tell clients that something changed, and the core-MCP mechanism Events builds beyond.
- [20.14 How does a program sign in with no person?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/machines-with-no-person/). Work that runs without a person watching, the setting where events matter most.
- [17.7 What if a record tells the AI what to do?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/untrusted-output/). What an event payload may and may not carry.
- [20.1 What is an MCP extension?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/what-an-mcp-extension-is/). Where Events sits among the MCP extensions.

**Official docs.** Keep these open while you build:

- [MCP: Events](https://modelcontextprotocol.io/extensions/events/overview). How clients subscribe and servers emit events.
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix). Current Claude and ChatGPT support.
- [Cloudflare Queues](https://developers.cloudflare.com/queues/). Publishing events without blocking a tool call.

## Why this step matters

Up to now, the model only learns things when it asks. Events let the server push meaningful changes, which opens the door to agents that react to the business instead of waiting to be prompted.

## Why this technology in an MCP server: The MCP Events extension

- **What it is.** Events is the MCP extension for server-initiated notifications about things that happen in the business, such as a new order or a crossed threshold.
- **Why an MCP server uses it.** Until now the model only learned something when it asked. Events let your server tell the client when something changes, which opens the way to agents that react to the business instead of waiting for a prompt.
- **What it solves.** Polling (the model calling a tool again and again to see if anything changed) and missed changes.
- **Alternatives.** Polling tools, webhooks to other systems, or notifications outside MCP such as email or Slack. Core MCP's list-changed notifications only cover changes to tool or resource lists, not business events.

## What you will learn

- Understand the Events extension and how clients subscribe
- Emit events from the server when data changes in Neon, publishing them through a Queue (step 12) and keeping each user's subscriptions in their Durable Object (step 11)
- Design event payloads that are useful to a model
- Combine events with tasks and widgets

## What you will build

Two events on the sales server: `order.created` (order ID, total and customer name, never full personal data) and `sales.threshold_reached` (month, total and threshold). Clients subscribe per company; subscriptions and the last threshold sent live in each user's `UserState` Durable Object; events go out through a Queue so a slow client never blocks `create_order`; and every event has an ID so a client that receives it twice can ignore the copy. A `list_recent_events` tool returns the same events for clients that don't support the extension yet.

## Key concepts

- **Events**: Server-initiated notifications about domain changes.
- **Subscriptions**: How a client chooses what it wants to hear about.
- **Delivery by poll, push or webhook**: The ways a client can receive events, and which ones each client supports.
- **Event ID and deduplication**: Every event carries a unique ID, so delivering it twice does no harm.
- **Event design**: Small, meaningful and actionable payloads that carry IDs, not full records or personal data.
- **Fallback**: A plain tool that returns recent events, for clients without the extension.

## Outline

1. What the Events extension provides
2. Emitting events from a Worker
3. Subscriptions and delivery
4. Event-driven workflows with tasks and widgets

## Prerequisites

- [Step 14](../step-14-tasks/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 14 code into `step-15-events/`.**

```text
Copy my step-14 code into step-15-events/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-15 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-15 from my step-14 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Check the Events extension's current client support, then plan.**

```text
Read the MCP Events extension docs linked in this step and explain how clients subscribe and how servers emit events (poll, push and webhook). Then plan event support for this server: the event names and payloads, where subscriptions live, how events are delivered through a queue, how duplicates are avoided, and a list_recent_events fallback tool for clients that don't support Events.
```

**3. Define events.**

```text
Define two events, order.created and sales.threshold_reached, with small, actionable payloads. Document each in the code.
```

**4. Emit them.**

```text
Emit order.created whenever create_order succeeds, and sales.threshold_reached when monthly sales cross a configurable threshold. Publish events through a queue like the one from step 12 so a slow client never blocks a tool call, and keep each user's subscriptions and threshold state in their UserState Durable Object from step 11. Carry the step 7 tenant envelope on every event, and deliver it only to subscribers in the same company.
```

**5. Connect to tasks and widgets.**

```text
Show how an event can lead the model to start a report task or show a widget, and add an example flow.
```

**6. Test it.**

```text
Add list_recent_events as the fallback, and tests that one company never receives another company's events and that a duplicate delivery is ignored. Then deploy and give me a way to trigger each event and confirm the client received it, using the MCP Inspector where a client doesn't support Events yet.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-15-events/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: events and subscriptions never cross companies, and the test for it passes
- [ ] It works in claude.ai or Claude Desktop. If Claude doesn't support this extension yet, the rest of the server still works there and your NOTES.md records the support status from the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix)
- [ ] It works in ChatGPT (custom MCP server), with the same exception and NOTES.md entry if ChatGPT doesn't support this extension yet
- [ ] Project defence recorded (up to five minutes: it works, one tool end to end, one tenant-isolation decision, one change you made; see [SUBMITTING.md](../SUBMITTING.md#10-defend-your-project)) and linked in the form
- [ ] Tagged `step-15-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-15-done`, live server URL, and screenshots of an event being received (or a note on client support if it isn't available yet) in **Claude** and **ChatGPT**

## References

- [MCP: Events](https://modelcontextprotocol.io/extensions/events/overview)

---

← [Step 14: Long-Running Work with Tasks](../step-14-tasks/) · [Roadmap](../README.md#roadmap) · [Step 16: Security Hardening for MCP Servers](../step-16-security-hardening/) →
