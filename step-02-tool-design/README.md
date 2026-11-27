# Step 02: Designing Good Tools

Learn to design tools that models use correctly. That means clear names and descriptions, tight input schemas, structured outputs, sensible granularity, and annotations that tell the host which tools are safe.

> 🏬 **Stockroom's ask:** Stockroom's first feature is account notes for its customers' sales reps ("called Ali at Karachi Foods, wants a price list on Friday"). Before any of it is real data, Stockroom wants notes tools the model calls correctly every time, because a wrong call in front of a customer's staff costs trust.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.2 How does the model pick the right tool?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/a-tool-is-its-description/). The model chooses tools from their names, descriptions and schemas.
- [17.3 What should a tool say when it fails?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/errors-that-say-what-to-do-next/). Writing errors the model can recover from.
- [14.5 What a schema holds, and what it lets through](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/schema-rules/). Validating input with Zod-style schemas.
- [17.4 What if a different copy answers the next call?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/stateless-servers/). Why tools on Workers must not keep state between calls.
- [17.7 What if a record tells the AI what to do?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/untrusted-output/). What a tool should and should not return.

**Official docs.** Keep these open while you build:

- [MCP: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools). Structured output, annotations and tool errors in the spec.
- [MCP security best practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices). What a tool should never trust or return.
- [Cloudflare Workers KV](https://developers.cloudflare.com/kv/). Where this step stores the notes.
- [Cloudflare Workers: limits](https://developers.cloudflare.com/workers/platform/limits/). CPU, memory and request limits your tools must fit in.

## Why this step matters

A server is only as good as the model's ability to use it. Poorly described or overly broad tools lead to wrong calls, wasted tokens and confused users. Tool design matters more than anything else in this bootcamp, and the later steps (database, UI, tasks) all depend on it.

## Why this technology in an MCP server: Good tool design, and Workers KV for shared state

- **What it is.** Tools are the functions your server offers, each described by a name, a description and an input schema. Workers KV is Cloudflare's key-value store, readable from every copy of your Worker.
- **Why an MCP server uses it.** The model never reads your code. It chooses and calls tools only from their names, descriptions and schemas, so tool design *is* the interface of an MCP server. And because Cloudflare may run many copies of your Worker, any data that must survive between tool calls needs a shared store, not the Worker's memory.
- **What it solves.** Well-designed tools mean fewer wrong calls, fewer wasted tokens and safer confirmations, since annotations tell the host which tools change data. KV keeps the notes alive across requests and copies.
- **Alternatives.** For storage: D1 or Postgres (more than a notes demo needs, and used for real data from step 5), or Durable Objects when data must be strongly consistent (step 11). There is no alternative to good tool design.

## What you will learn

- Write tool names and descriptions that help the model pick the right tool
- Use precise input schemas (enums, required fields, formats) to prevent bad calls
- Return structured content alongside human-readable text
- Choose the right granularity: when to split a tool and when to combine
- Use tool annotations (`readOnlyHint`, `destructiveHint` and friends) so hosts can make safe permission choices
- Return useful, recoverable errors instead of throwing opaque failures
- Keep the server stateless: store data where every copy of the Worker can read it, never in the Worker's memory

## What you will build

We refactor the step 1 server into a small "notes" toolset (`list_notes`, `get_note`, `create_note`, `delete_note`) stored in Cloudflare Workers KV, then compare a badly designed version against a well-designed one.

## Key concepts

- **Descriptions are prompts**: The model reads them to decide what to call.
- **Schemas are guardrails**: Constrain inputs so invalid calls fail early and clearly.
- **Structured output**: Machine-readable results the model and the UI can both use.
- **Annotations**: Hints about side effects that hosts use for confirmation prompts.
- **Stateless servers**: Cloudflare may run many copies of your Worker, so anything kept in memory can vanish between calls. The notes go in Workers KV, a key-value store every copy can read. KV is eventually consistent (a change can take up to a minute to reach other regions), which is fine for notes. Business data moves to Postgres in step 5.

## Outline

1. Anatomy of a tool definition
2. Naming and describing tools
3. Input schema patterns
4. Structured content vs. text content
5. Tool annotations and safety
6. Error handling
7. Before/after: redesigning a messy toolset
8. Testing tool selection in Claude and ChatGPT

## Prerequisites

- [Step 1](../step-01-hello-world/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 1 code into `step-02-tool-design/`.**

```text
Copy my step-01 code into step-02-tool-design/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-02 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds.
```

**2. Review how the tools are defined today.**

```text
Read the project in step-02-tool-design/ and summarize how the tools are defined today. Then use the build-mcp-server skill's tool-design guidance to list what makes a well-designed MCP tool.
```

**3. Build the notes toolset.**

```text
Replace the demo tools with a notes toolset: list_notes, get_note, create_note and delete_note. Store the notes in a Workers KV namespace bound as NOTES, not in the Worker's memory, and explain why memory would fail once the server is online. Give me the Wrangler command to create the namespace and add the binding to the wrangler config. Use precise input schemas (required fields, string limits, enums where it makes sense) and descriptions that tell the model when to use each tool.
```

**4. Add structured output and annotations.**

```text
Make each tool return structured content alongside a short human-readable text summary. Add tool annotations: mark the read tools readOnlyHint and delete_note destructiveHint. Explain how Claude and ChatGPT use these hints.
```

**5. Make errors useful.**

```text
Review error handling. When a note isn't found or the input is invalid, return a clear, recoverable error message the model can act on instead of throwing.
```

**6. Compare a bad design with a good one.**

```text
Create a deliberately badly designed version of one tool in a separate file (vague name, loose schema, no description), and write a short markdown comparison explaining why the good version gets called correctly more often.
```

**7. Deploy and test tool selection.**

```text
Deploy, then give me five realistic prompts to try in Claude and ChatGPT that check the model picks the right notes tool each time.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-02-tool-design/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-02-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-02-done`, live server URL, and screenshots of the model choosing the right notes tool in **Claude** and **ChatGPT**

## References

- [MCP specification](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro)
- [Built-in skill: `build-mcp-server` (see `.claude/skills/`)](../.claude/skills/build-mcp-server/SKILL.md)

---

← [Step 01: Hello World: A Minimal Remote MCP Server on Cloudflare](../step-01-hello-world/) · [Roadmap](../README.md#roadmap) · [Step 03: Resources and Prompts: The Other Two MCP Primitives](../step-03-resources-and-prompts/) →
