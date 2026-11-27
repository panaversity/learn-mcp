# Step 09: Generative UI with MCP Apps

Give the server a face. Tools return interactive React widgets (tables, charts, forms) that render inside the chat in Claude and ChatGPT, and the model picks which widget to show based on what the user asked.

> 🏬 **Stockroom's ask:** Customers' staff find long text tables hard to read in a chat. Stockroom wants real screens (an orders table, a sales chart, an order form) that appear in Claude and ChatGPT when someone asks.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [20.1 What is an MCP extension?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/what-an-mcp-extension-is/). Extensions, their stages and fallbacks.
- [20.2 When does a screen beat text?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/when-a-screen-beats-text/). When a widget is worth it.
- [20.3 How does a tool get a screen?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/how-an-mcp-app-works/). The `ui://` resource, the tool pointer, the sandbox and messaging.
- [20.6 What can the page reach?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/the-sandbox/). What the iframe can and cannot do.
- [20.9 What if the AI app shows no screens?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/a-text-answer-behind-every-screen/). Making the same widget work in Claude and ChatGPT.

**Official docs.** Keep these open while you build:

- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview). The standard for interactive UI in MCP.
- [MCP Apps: build an app](https://modelcontextprotocol.io/extensions/apps/build). Registering `ui://` resources and linking them to tools.
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix). Which clients render MCP Apps today.

## Why this step matters

Text is a poor interface for data. MCP Apps is the open standard for interactive UI in MCP: the server ships an HTML/JS app as a `ui://` resource, and the host renders it in a sandboxed frame. Building with the standard `@modelcontextprotocol/ext-apps` APIs, and nothing client-specific, lets one server work in both Claude and ChatGPT.

## Why this technology in an MCP server: MCP Apps (interactive UI in the chat)

- **What it is.** MCP Apps is the MCP extension for interactive UI. The server registers an HTML/JS app as a `ui://` resource, a tool points to it, and the host renders it in a sandboxed frame inside the chat. We build the apps with React.
- **Why an MCP server uses it.** Text is a poor way to show a table of orders, a chart or a form. MCP Apps lets a tool return a real interface in Claude and ChatGPT, built once with the standard APIs, and lets the widget call tools back through the host.
- **What it solves.** Users get screens they can read and act on without leaving the chat, and you ship one UI that works in both major clients instead of building a separate web app.
- **Alternatives.** Client-specific APIs such as ChatGPT-only `window.openai` calls (we avoid them to stay portable), markdown tables or images in text replies, or a separate web app that users have to switch to.

## What you will learn

- Explain the MCP Apps model: UI resources, linking a tool to its UI, and the sandboxed iframe
- Build React widgets and bundle them as `ui://` resources
- Pass tool results into a widget, and let the widget call tools back
- Place this step among the three ways to make a screen in book lesson 20.10 (fixed, from approved parts, open): here the model picks one fixed screen per request and fills it with data; step 19 lets it compose from approved parts
- Keep widgets portable across Claude and ChatGPT
- Respect payload size, theming and accessibility inside the chat

## What you will build

Sales widgets for the step 8 server, each showing only the signed-in company's data: an orders table, a sales chart and an order-entry form. The model picks whichever fits the request.

## Key concepts

- **UI resources (`ui://`)**: HTML/JS bundles that the server registers and the host renders.
- **Tool to UI linkage**: How a tool tells the host which UI to show with its result.
- **Widget to host messaging**: Widgets can call tools and send messages back through the host.
- **Fixed, approved-parts and open screens (book lesson 20.10)**: This step builds fixed screens that the model selects. Composing a screen from approved parts comes in step 19. Open screens, where the model writes its own HTML, are out of scope because they are hard to keep safe.

## Outline

1. How MCP Apps works
2. Setting up React and a bundler for widgets
3. Registering a UI resource
4. Linking tools to widgets
5. Two-way communication between widget and server
6. Letting the model choose the widget
7. Portability checklist for Claude and ChatGPT
8. Testing in both clients

## Prerequisites

- [Step 8](../step-08-semantic-search-pgvector/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 8 code into `step-09-generative-ui-mcp-apps/`.**

```text
Copy my step-08 code into step-09-generative-ui-mcp-apps/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-09 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-09 from my step-08 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan the widgets.**

```text
Use the build-mcp-app skill. Plan how to add MCP Apps widgets to this server using React and the standard @modelcontextprotocol/ext-apps APIs only, with nothing Claude-specific or ChatGPT-specific, so it works in both. Propose the build setup for bundling widgets.
```

**3. Set up the widget build.**

```text
Set up a React widget build that outputs self-contained bundles the server can serve as ui:// resources. Add a script that builds the widgets before deploying.
```

**4. Build the first widget.**

```text
Build an OrdersTable widget and link it to list_orders, so the table renders in the chat with the tool's result. Follow the skill's guidance on payload size and theming.
```

**5. Add more widgets.**

```text
Add a SalesChart widget for get_sales_summary and an OrderForm widget that calls create_order from inside the widget. Show how the widget calls the tool back through the host.
```

**6. Let the model choose.**

```text
Improve the tool and widget descriptions so the model picks the table, chart or form based on what the user asks. Give me prompts that should each produce a different widget.
```

**7. Test portability.**

```text
Deploy, then give me a test checklist for Claude (web and Desktop) and ChatGPT covering rendering, theming, tool callbacks and error states. Note anything that behaves differently between clients.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-09-generative-ui-mcp-apps/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: widgets show only the signed-in company's data, and every tool a widget calls re-checks the caller's company
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-09-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-09-done`, live server URL, and screenshots of a React widget (table, chart or form) rendering in the chat in **Claude** and **ChatGPT**

## References

- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview)
- [SEP-1865: MCP Apps](https://modelcontextprotocol.io/seps/1865-mcp-apps-interactive-user-interfaces-for-mcp)
- [`ext-apps` SDK](https://github.com/modelcontextprotocol/ext-apps)
- [Built-in skill: `build-mcp-app` (see `.claude/skills/`)](../.claude/skills/build-mcp-app/SKILL.md)

---

← [Step 08: Search by Meaning with pgvector: Answers That Cite or Abstain](../step-08-semantic-search-pgvector/) · [Roadmap](../README.md#roadmap) · [Step 10: Skills over MCP](../step-10-skills-over-mcp/) →
