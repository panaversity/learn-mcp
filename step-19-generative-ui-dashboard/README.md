# Step 19: Generative UI: An Intent-Driven Sales Dashboard

Put the bootcamp's north star to work on the sales server. A user asks in plain language, for example "show me last quarter's sales forecast", and the model chooses and fills the right widget with live Neon data, in both Claude and ChatGPT.

> 🏬 **Stockroom's ask:** Stockroom's customers' managers don't want to learn a dashboard. They want to ask "how did we do last quarter?" and get the right screen. This is the feature Stockroom will sell on.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [20.10 Should the AI design the screen?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/screens-the-ai-designs/). Fixed, approved-parts and open screens. This is the core of generative UI.
- [20.15 How do the pieces fit together?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/how-the-pieces-fit/). Core MCP plus extensions as one connector, and common mistakes.
- [20.4 Who does a click act as?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/who-a-click-acts-as/). Who a widget action runs as.
- [20.7 What if the data holds page code?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/untrusted-text-on-the-screen/). Handling text you didn't write safely in a widget.
- [Lab 20b, a full interactive app](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/lab-review-workspace/). A worked example the size of this step.

**Official docs.** Keep these open while you build:

- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview). The standard behind every widget.
- [MCP Apps: build an app](https://modelcontextprotocol.io/extensions/apps/build). Tool-to-UI linking and widget messaging.
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix). What renders where.

## Why this step matters

This is the bootcamp's north star, Generative UI + MCP: an app built around intent instead of navigation. The interface comes out of the user's request, and MCP supplies the tools, data, skills and governance behind it. It is also the last feature the sales server gets: step 20 puts a price on it, and in the step 21 capstone you build a new product from the same parts.

## Why this technology in an MCP server: Generative UI

- **What it is.** Generative UI means the model chooses which screen to show, and fills it, from what the user asks, instead of the user navigating menus. Your server supplies a library of well-described widgets and the data behind them.
- **Why an MCP server uses it.** MCP is how the model reaches your tools, data and widgets, and MCP Apps is how the chosen screen reaches the chat. The server's job is to make the right screen easy to choose and safe to show.
- **What it solves.** Fixed dashboards that never quite match the question, and one-widget-per-tool designs that can't combine views.
- **Alternatives.** A fixed dashboard app, one fixed widget per tool (step 9), or letting the model generate its own HTML ("open screens" in book lesson 20.10), which is harder to keep safe.

## What you will learn

- Design a widget library that the model can compose from
- Map user intents to tools, skills and widgets
- Feed widgets from everything the server already has: Neon data, the draft order in each user's Durable Object, import status from the queue, and report runs from the Workflow
- Evaluate the result with real prompts in Claude and ChatGPT
- Reflect on where generative UI helps, and where a fixed UI is better

## What you will build

A complete, deployed sales-insights server where the dashboard is generated from the user's intent.

## Key concepts

- **Intent over navigation**: No menus. The request drives the interface.
- **Composable widgets**: Small, well-described components the model can choose from.
- **Governance**: Auth, annotations and confirmations keep generative UI safe.

## Outline

1. The dashboard brief and architecture
2. Building the widget library
3. Intent-to-UI mapping with tools and skills
4. Putting it together
5. Evaluation in Claude and ChatGPT
6. Retrospective: where generative UI helps, and where a fixed UI is better

## Prerequisites

- [Step 18](../step-18-deploy-and-test/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 18 code into `step-19-generative-ui-dashboard/`.**

```text
Copy my step-18 code into step-19-generative-ui-dashboard/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-19 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-19 from my step-18 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and design the dashboard.**

```text
Design the dashboard: an intent-driven sales dashboard where the user asks in plain language and the model chooses and fills the right widgets with live Neon data. Map ten example intents to the tools, skills and widgets each one needs.
```

**3. Build the widget library.**

```text
Refactor the widgets into a small composable library (KPI card, table, chart, form, report) with consistent props and theming, and descriptions that help the model pick between them.
```

**4. Wire intents to the UI.**

```text
Update the tools and the monthly-sales-review skill so each example intent produces the right combination of widgets. Add any missing tools.
```

**5. Evaluate.**

```text
Write an evaluation script of 20 realistic prompts with the expected tool and widget for each. Help me run them in Claude and ChatGPT and record the results.
```

**6. Reflect and polish.**

```text
Summarize where generative UI worked well and where a fixed UI would have been better, then write it up in step-19-generative-ui-dashboard/NOTES.md (leave the bootcamp's README.md unchanged).
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-19-generative-ui-dashboard/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: every dashboard intent and widget is scoped to the caller's company
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-19-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-19-done`, live server URL, and screenshots of two different intents producing two different dashboard widgets in **Claude** and **ChatGPT**

## References

- [Generative UI + MCP: The New Stack for AI-Native Applications (ODSC)](https://odsc.medium.com/generative-ui-mcp-the-new-stack-for-ai-native-applications-b62af183f2aa)
- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview)

---

← [Step 18: Deploying and Testing on claude.ai, Claude Desktop and ChatGPT](../step-18-deploy-and-test/) · [Roadmap](../README.md#roadmap) · [Step 20: Billing and Distribution: Charging for Your Server and Listing It](../step-20-billing-and-distribution/) →
