# Step 03: Resources and Prompts: The Other Two MCP Primitives

Tools are only one of the three things an MCP server can offer. **Resources** are data with a fixed address (a URI) that the host app can read and attach to the conversation, and **prompts** are ready-made message templates a person picks, often from a slash menu. In this step your notes server publishes each note as a resource and offers one prompt, "summarize my notes", so you learn when a feature should be a tool, a resource or a prompt. Prompts are still part of the MCP spec, but few servers use them today, and MCP's maintainers say they have mostly fallen out of use. So this step keeps them small, and step 10 shows the modern way to ship reusable know-how: skills served over MCP. Resources matter more than ever, because widgets (`ui://`) and skills (`skill://`) are both built on them.

> 🏬 **Stockroom's ask:** Reps want to attach an account's notes to a chat, and managers want one ready-made way to ask for a summary of the week's notes. That is a resource and a prompt, not another tool.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.1 How does an AI app ask a connector for help?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/mcp-in-one-picture/). Where tools, resources and prompts sit between the AI app and the server.
- [Field guide F8, The protocol in detail](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f8-the-protocol-in-detail). Resources (data with a fixed name that the host decides to use) and prompts (templates a person picks).
- [17.2 How does the model pick the right tool?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/a-tool-is-its-description/). The model chooses tools; compare that with who chooses resources and prompts.
- *Optional:* [20.11 How does a skill travel with the connector?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/skills-that-travel-with-the-server/). A later use of resources: skills are served as `skill://` resources and read with `resources/read`.

**Official docs.** Keep these open while you build:

- [MCP: Resources](https://modelcontextprotocol.io/specification/2026-07-28/server/resources). Resource URIs, templates, `resources/list` and `resources/read`.
- [MCP: Prompts](https://modelcontextprotocol.io/specification/2026-07-28/server/prompts). Prompt templates, arguments and `prompts/get`.
- [MCP: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools). For comparing the three primitives.

## Why this step matters

Putting everything behind tools works, but it isn't always the best design. Reference data the user wants to attach (a policy, a note, a price list) reads better as a resource, and a task users repeat ("prepare a weekly summary") can be a prompt they pick, although for most procedures a skill (step 10) now does that job better. Knowing all three primitives lets you design servers that feel natural in Claude and ChatGPT, and later steps reuse resources for widgets (`ui://`) and skills (`skill://`).

## Why this technology in an MCP server: resources and prompts

- **What it is.** Resources are read-only data a server exposes at a URI, such as `note://42`. Prompts are named message templates with arguments that a person chooses to run.
- **Why an MCP server uses it.** The three primitives are controlled by different parties: the **model** chooses tools, the **app or user** chooses resources to attach, and the **user** chooses prompts. Matching each feature to the right primitive gives the user control where they expect it.
- **What it solves.** Tools that only fetch static data, long instructions users have to retype, and context the user wants to add on purpose rather than leaving it to the model.
- **Alternatives.** A read-only tool instead of a resource (works everywhere, but the model decides when to use it), or a skill instead of a prompt (step 10), which is now the usual choice for reusable procedures. Client support for resources and prompts varies, so check how each client shows them.

## What you will learn

- Explain the three MCP primitives and who controls each one
- Publish notes as resources, with a resource template for `note://{id}`
- Add one prompt with an argument (`summarize_notes`), and explain why skills have largely taken over the job prompts were meant for
- Notify clients when the resource list changes
- Decide whether a feature should be a tool, a resource or a prompt
- See how Claude and ChatGPT show resources and prompts today

## What you will build

The step 2 notes server, now also listing each note as a resource (`note://{id}`, with the note's text as its content) and offering one prompt, `summarize_notes` (optionally for a tag). The notes tools stay, so the same data is reachable all three ways.

## Key concepts

- **Resource**: Read-only data at a URI that the host app or user chooses to attach.
- **Resource template**: A URI pattern such as `note://{id}` that covers many resources.
- **Prompt**: A named message template, with arguments, that a person picks to start a task.
- **Who controls what**: The model picks tools, the app or user picks resources, and the user picks prompts.
- **Prompts today**: Still in the spec and not deprecated, but rarely used. Servers now ship reusable procedures as skills over MCP (step 10), which the model loads when it needs them.
- **List-changed notification**: A message telling clients that the list of resources or prompts has changed.

## Outline

1. The three MCP primitives
2. Notes as resources and a resource template
3. One prompt, and why skills have taken over
4. Telling clients when lists change
5. Tool, resource or prompt?
6. Testing in Claude and ChatGPT

## Prerequisites

- [Step 2](../step-02-tool-design/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 2 code into `step-03-resources-and-prompts/`.**

```text
Copy my step-02 code into step-03-resources-and-prompts/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-03 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds.
```

**2. Switch to plan mode and plan the change.**

```text
Read the build-mcp-server skill's resources-and-prompts reference and the MCP resources and prompts docs linked in this step. Explain the three MCP primitives and who controls each. Then plan how this notes server should expose notes as resources and offer one prompt, summarize_notes.
```

**3. Publish notes as resources.**

```text
Expose each note as a resource with a note://{id} resource template, list them with resources/list, return the note text from resources/read, and send a list-changed notification (through the handler's notify controls) when a note is created or deleted. Explain why the notification only reaches clients with an open subscriptions/listen stream on the same handler, and why that is fine for this step.
```

**4. Add a prompt.**

```text
Add one prompt, summarize_notes, with an optional tag argument. Write its messages so they tell the model to use the notes tools and resources. Then explain in NOTES.md why MCP prompts are rarely used today, and how a skill served over MCP (step 10) differs: who picks it, when it loads, and why it works better across clients.
```

**5. Tool, resource or prompt?**

```text
Review the whole server and, in NOTES.md, explain for each feature whether it is a tool, a resource or a prompt and why. Suggest anything that should move.
```

**6. Deploy and test.**

```text
Deploy, then tell me how to attach a note as a resource and run the prompt in claude.ai, Claude Desktop and ChatGPT, and how to check with the MCP Inspector if a client doesn't show resources or prompts yet.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)
- The MCP Inspector, for clients that don't show resources or prompts yet

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-03-resources-and-prompts/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server), or your NOTES.md records how ChatGPT shows resources and prompts today
- [ ] Tagged `step-03-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-03-done`, live server URL, and screenshots of a note attached as a resource and a prompt being run in **Claude** and **ChatGPT** (or the MCP Inspector)

## References

- [MCP: Resources](https://modelcontextprotocol.io/specification/2026-07-28/server/resources)
- [MCP: Prompts](https://modelcontextprotocol.io/specification/2026-07-28/server/prompts)
- [Built-in skill: `build-mcp-server` (see `.claude/skills/`)](../.claude/skills/build-mcp-server/SKILL.md)

---

← [Step 02: Designing Good Tools](../step-02-tool-design/) · [Roadmap](../README.md#roadmap) · [Step 04: Adding Authorization with Better Auth and D1](../step-04-auth/) →
