# Step 01: Hello World: A Minimal Remote MCP Server on Cloudflare

Build and deploy the smallest useful remote MCP server: two small tools, written in TypeScript, running on Cloudflare Workers, and connected to Claude and ChatGPT.

> 🏬 **Stockroom's ask:** Meet your client. Throughout this bootcamp you work for **Stockroom**, a fictional startup that sells AI-native back-office software to small distributors and retailers, as its Forward Deployed Engineer (see [Your client: Stockroom](../README.md#your-client-stockroom)). Its first ask is small: Stockroom wants proof that its customers can reach a server of yours from Claude and ChatGPT before it pays for anything bigger. A live `/mcp` URL with two working tools is that proof.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [12.1 From a chat to a coding agent](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/from-chat-to-coding-agent/). How to drive Claude Code to build the server.
- [14.7 JSON and REST APIs](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/). MCP is JSON over HTTP, so you'll read a lot of requests and responses.
- [17.1 How does an AI app ask a connector for help?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/mcp-in-one-picture/). Client, server, tool call and reply in one diagram.
- [Field guide F1, Cloudflare from zero](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f1-cloudflare-from-zero-and-running-the-servers-on-your-computer). What a Worker is and how to run one locally.
- [Field guide F2, The MCP Inspector](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f2-the-mcp-inspector). Test the server before connecting a client.

**Official docs.** Keep these open while you build:

- [MCP specification (2026-07-28)](https://modelcontextprotocol.io/specification/2026-07-28). The protocol itself: hosts, clients, servers and messages.
- [MCP: Key changes in 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28/changelog). Why the protocol is now stateless, and which older features (sessions, the `initialize` handshake, sampling) to ignore when a tutorial still shows them.
- [MCP: Streamable HTTP transport](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http). How a remote server talks to clients at `/mcp`.
- [MCP: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools). Tool names, descriptions and input schemas.
- [MCP Inspector](https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector). Testing your server before connecting a client.
- [Cloudflare Workers: get started](https://developers.cloudflare.com/workers/get-started/guide/). Creating, running and deploying a Worker.
- [Cloudflare Workers: TypeScript](https://developers.cloudflare.com/workers/languages/typescript/). Generating types with `wrangler types` and setting up `tsconfig.json`.
- [Cloudflare: Model Context Protocol](https://developers.cloudflare.com/agents/model-context-protocol/). Cloudflare's guide to remote MCP servers.
- [Cloudflare: MCP handler API (`createMcpHandler`)](https://developers.cloudflare.com/agents/model-context-protocol/apis/handler-api/). The stateless handler this bootcamp uses to serve `/mcp`. (The older `McpAgent` class is deprecated, so don't follow tutorials that use it.)
- [OpenAI: add a custom MCP server](https://developers.openai.com/api/docs/guides/custom-mcp-server). Connecting your server in ChatGPT Plugins (formerly developer mode) and calling its tools with `@`.
- [Cloudflare: workers.dev routes](https://developers.cloudflare.com/workers/configuration/routing/workers-dev/). Where your public `/mcp` URL comes from.

## Why this step matters

Everything else in this bootcamp builds on a server that is reachable over the internet. Before adding auth, databases or UI, we want the full loop working end to end: write a tool, deploy it, connect a client, and watch the model call it.

## Why this technology in an MCP server: Cloudflare Workers and Streamable HTTP

- **What it is.** Cloudflare Workers run your TypeScript on Cloudflare's global network. Streamable HTTP is the MCP transport that lets a client talk to a server over an ordinary HTTPS URL, here `/mcp`.
- **Why an MCP server uses it.** claude.ai, Claude Desktop connectors and ChatGPT all connect to **remote** MCP servers by URL. A remote server is installed once and used by every client and every user, with nothing to install on their computers. Workers give you that URL with no server to manage, start instantly, scale down to zero when idle, and have a free plan, which suits a server that may get ten calls a day or ten thousand.
- **What it solves.** A local (stdio) server only runs on one person's computer and can't be reached from claude.ai on the web or from ChatGPT. A remote server can, so it can serve paying clients.
- **Alternatives.** Node.js on a container or VM (Fly.io, Render, Google Cloud Run), or serverless functions on Vercel or AWS Lambda. For tools that must touch a user's own files or apps, a local server packaged as an MCPB bundle. The book and this bootcamp standardize on Cloudflare.

## What you will learn

- Explain what an MCP server is, and how a *remote* server (Streamable HTTP) differs from a *local* one (stdio)
- Scaffold a TypeScript MCP server for Cloudflare Workers
- Define your first tools, each with a name, description and typed input schema
- Run the server locally and inspect it with the MCP Inspector
- Deploy it to Cloudflare and get a public `/mcp` URL
- Add the server as a custom connector in claude.ai and Claude Desktop, then in ChatGPT as a custom MCP server, and call the tool from a chat

## What you will build

A server with a `greet` tool and a second `get_time` tool (so you see the model choose between tools), deployed to `*.workers.dev` as a Worker named `learn-mcp-step-01`.

## Key concepts

- **Hosts, clients and servers**: Claude or ChatGPT is the host. It runs an MCP client that connects to our server.
- **Transports**: Remote servers speak Streamable HTTP. Local servers usually use stdio.
- **Tools**: Functions the model can call, each described by a name, a description and a JSON Schema for its inputs.
- **Capabilities on every request**: Since the 2026-07-28 spec there is no handshake when a client connects. Each request carries the protocol version and the client's capabilities in its `_meta`, and a client can ask the server what it supports with `server/discover`. New features, such as MCP Apps, Skills, Tasks and Events, arrive as optional *extensions* that both sides must declare.
- **A stateless server**: We serve `/mcp` with Cloudflare's `createMcpHandler` and MCP SDK v2. Two things have different lifetimes here. The handler is created once, when the Worker starts. For every request, the handler calls your factory function, which builds a fresh MCP server, registers the tools and answers that one request. The MCP spec (2026-07-28) removed protocol sessions and the `Mcp-Session-Id` header altogether, so no session is kept and any copy of the Worker can answer any request, just like an ordinary web service behind a load balancer. Anything that must last between calls goes in a store (KV, D1, Neon or a Durable Object), never in memory.

## Outline

1. Project setup: Node, Wrangler and the Cloudflare account
2. Scaffolding the Worker and the MCP server
3. Writing the `greet` and `get_time` tools
4. Testing locally with the MCP Inspector
5. Deploying with `wrangler deploy`
6. Connecting from claude.ai and Claude Desktop
7. Connecting from ChatGPT (custom MCP server)
8. Troubleshooting common connection errors

## Prerequisites

- [Step 00: Are You Ready? Readiness Check](../step-00-readiness-check/), passed or tested out
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, and make sure Claude plans before it writes code (the prompt asks it to). All code for this step goes in `step-01-hello-world/`.**

```text
Use the build-mcp-server skill. I want a minimal remote MCP server in TypeScript, deployed to Cloudflare Workers over Streamable HTTP, with no auth yet. Serve /mcp with Cloudflare's stateless createMcpHandler (from agents/mcp/server) and MCP SDK v2, following the handler API docs linked in this step, not the deprecated McpAgent class. Create the handler once at module level, and give it a factory that builds a new McpServer and registers the tools on every request. Nothing about a request may be kept in module scope. Ask me any questions you need, then propose a plan and a file layout before writing code.
```

**2. Approve the plan, then let Claude scaffold the project.**

```text
Scaffold the project in step-01-hello-world/: package.json, tsconfig, wrangler config and the server entry point. Name the Worker learn-mcp-step-01 and expose the MCP endpoint at /mcp. Pin TypeScript 7 and use it only to type-check (noEmit), because Wrangler bundles the code. In tsconfig, use moduleResolution "bundler" and isolatedModules, and list ./worker-configuration.d.ts and node in types (install @types/node). Set compatibility_date in the wrangler config to today's date, with no nodejs_compat flag, because Node.js compatibility is on by default for dates from 2026-08-04. Add a typecheck script that runs wrangler types && tsc. Leave the existing README.md in that folder unchanged, because it belongs to the bootcamp. Explain each file in one or two sentences.
```

**3. Add the first tools.**

```text
Add two tools: `greet`, which takes a required `name` string and returns a friendly greeting, and `get_time`, which takes an optional IANA `timezone` and returns the current time. Write clear descriptions and input schemas.
```

**4. Run it locally and inspect it.**

```text
Start the server locally and tell me how to open the MCP Inspector against it. Then list the tools and call each one so we can see the responses.
```

**5. Prove the server keeps no state between requests.**

```text
Add a test with Vitest and the Workers test pool that sends 20 concurrent tools/call requests with different names to greet and checks that each response has its own name. Then explain what would break if the McpServer were created once at module level instead of inside the factory.
```

**6. Deploy to Cloudflare.**

```text
Walk me through logging in to Wrangler and deploying this Worker. After deploying, tell me the exact MCP URL to use in clients.
```

**7. Connect from Claude and ChatGPT. This part happens in their settings, not in Claude Code.**

```text
Give me step-by-step instructions to add this server as a custom connector in claude.ai, Claude Desktop and ChatGPT (as a custom MCP server), and a test prompt for each that should make the model call `greet`.
```

**8. Review, then commit in GitHub Desktop.**

```text
Review the code for anything unclear or unnecessary. Then write a short NOTES.md in step-01-hello-world/ explaining how to run, test and deploy this server (don't touch README.md, which belongs to the bootcamp). Don't commit: I'll read the diff and commit it myself in GitHub Desktop.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-01-hello-world/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-01-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-01-done`, live server URL, and screenshots of the model calling `greet` in **Claude** and **ChatGPT**

## References

- [MCP: Getting started](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro)
- [Cloudflare: Model Context Protocol](https://developers.cloudflare.com/agents/model-context-protocol/)

---

← [Step 00: Are You Ready? Readiness Check](../step-00-readiness-check/) · [Roadmap](../README.md#roadmap) · [Step 02: Designing Good Tools](../step-02-tool-design/) →
