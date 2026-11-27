# Step 10: Skills over MCP

Ship *know-how* along with tools. Skills over MCP lets a server provide packaged instructions (skills) that teach the model how to use the server's tools well for specific tasks. Because the skills live on the server, you update them whenever you update the tools, and users install nothing. And because your server already knows who is calling and which company they belong to, it can do something a plugin can't: serve a shared playbook to everyone *and* a skill built for each company.

> 🏬 **Stockroom's ask:** Every Stockroom customer sells a little differently. Stockroom wants one shared sales-review playbook for everyone, plus each customer's own targets, terms and sign-off rules, served only to that customer.

> **Note:** Skills over MCP (`io.modelcontextprotocol/skills`) is a Beta extension. Anthropic announced public beta support in Claude and Claude Code on November 24, 2026: on by default for Pro, Max and Ultra plans, enabled by an admin on Team plans, with Enterprise to follow. ChatGPT has partial support. Check the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix) for the latest status.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [16.1 From a prompt you repeat to a skill](https://agentfactory.panaversity.org/building-ai-workers/skills-and-plugins/from-a-prompt-to-a-skill/). What a skill is.
- [20.11 How does a skill travel with the connector?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/skills-that-travel-with-the-server/). The Skills over MCP extension.
- [20.12 Who approves a skill from a server?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/approving-a-skill-from-a-server/). How trust and approval work.
- *Optional:* [16.9 Installing is a trust decision](https://agentfactory.panaversity.org/building-ai-workers/skills-and-plugins/installing-is-a-trust-decision/). Why adding a skill is a trust decision.

**Official docs.** Keep these open while you build:

- [MCP: Skills over MCP](https://modelcontextprotocol.io/extensions/skills/overview). How a server provides skills to a client.
- [Skills extension specification: security considerations](https://github.com/modelcontextprotocol/ext-skills/blob/main/specification/beta/skills.mdx#security-considerations). The approval, origin and cache rules a client applies before it loads a server's skill.
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix). Current Claude and ChatGPT support.

## Why this step matters

Tools say *what* a model can do, and skills say *how* to do a job well. Serving skills from the MCP server keeps that expertise versioned next to the tools it describes, so every connected client gets the same playbook.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../assets/diagram-skills-over-mcp-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="../assets/diagram-skills-over-mcp-light.png">
  <img src="../assets/diagram-skills-over-mcp-light.png" alt="Two users from different companies call skills/list on the same MCP server. Each gets the shared monthly-sales-review skill plus their own company-playbook skill, served alongside the server's tools and resources. Skills load only when needed, after the user approves them, and per-company answers are cached as private." width="100%">
</picture>

## Why this technology in an MCP server: Skills over MCP

- **What it is.** A skill is packaged know-how: instructions (and optional files) that teach the model how to do a job well. The Skills over MCP extension lets your server provide skills alongside its tools.
- **Why an MCP server uses it.** Tools say *what* the model can do. Skills say *how* to do a client's job with them, such as "prepare the monthly sales review". Serving skills from the server keeps that playbook versioned next to the tools, and every connected client gets the same one.
- **What it solves.** Long, procedural tool descriptions that bloat the model's context, and inconsistent results because each user prompts differently. Skills load only when needed (progressive disclosure). And because the server authenticates every caller, it can serve each company a skill tailored to it, such as its own targets and sign-off steps.
- **Alternatives.** Putting the procedure in tool descriptions (costly in context), MCP prompts (templates the user picks), or asking users to install a skill in their own client separately.

## What you will learn

- Understand what a skill is (instructions plus optional resources) and when the model loads one
- Expose skills from the server using the Skills over MCP extension
- Write skills that orchestrate several tools, for example "prepare a monthly sales review"
- Serve skills per company: a shared skill for every tenant plus a company-specific skill, listed and served only to that company's users
- Understand why a client asks the user to approve a skill before loading it, and keep text a company admin writes from acting as instructions
- Decide what belongs in a tool description versus a skill
- Version and test skills

## What you will build

A shared `monthly-sales-review` skill that guides the model through querying Neon, choosing the right widgets and writing a summary, plus a `company-playbook` skill that the server builds for each company from its settings in Neon (its sales targets, product terms and sign-off steps). A company admin edits those settings through an admin-only `update_company_playbook` tool, and each company sees only its own playbook.

## Key concepts

- **Skills**: Packaged procedural knowledge the model can load on demand.
- **Progressive disclosure**: Short metadata up front, with the full instructions only when needed.
- **Tools + skills**: Capabilities plus a playbook for using them.
- **Per-company skills**: Skills built for the signed-in company and listed only to its users, something a server that knows its callers can do and an installed plugin can't.
- **Skill approval**: Skill text steers the model, so the client checks a skill and may ask the user to approve it, and approves it again when its content changes.
- **Private caching**: Skill lists and contents that depend on the caller are returned with `cacheScope: "private"`, so no shared cache hands one company's skill to another.

## Outline

1. What Skills over MCP adds
2. Defining and serving a skill
3. Writing effective skill instructions
4. Per-company skills from an authenticated server
5. Approval, and keeping company text as data
6. Skills that drive generative UI
7. Testing skills

## Prerequisites

- [Step 9](../step-09-generative-ui-mcp-apps/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 9 code into `step-10-skills-over-mcp/`.**

```text
Copy my step-09 code into step-10-skills-over-mcp/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-10 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-10 from my step-09 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Check the extension's current client support (see the note at the top of this page), then plan.**

```text
Read the Skills over MCP extension docs linked in this step and summarize how a server provides skills to a client: skills/list, skills/get, reading skill files with resources/read, and how the client approves a skill before loading it. Then plan how to add a shared monthly-sales-review skill and a per-company company-playbook skill to this server.
```

**3. Write the skill.**

```text
Create a monthly-sales-review skill. It should guide the model to pull the right data with get_sales_summary and list_orders, choose the right widgets, and write a short narrative summary with recommendations.
```

**4. Serve it.**

```text
Expose the skill from the server using the Skills over MCP extension, and keep it in a versioned folder next to the tools it uses.
```

**5. Add a per-company skill.**

```text
Add a company-playbook skill that the server builds for the caller's company from its settings in Neon (sales targets, product terms and sign-off steps), using the step 7 tenant context and never a company ID from input. Add an admin-only update_company_playbook tool with tight length limits that stores those settings as plain data, and render them into the skill as quoted data under a clear heading, never as instructions to the model. Return per-company skill lists and contents with cacheScope private, and make the skill's version change whenever its settings change, so the client asks for approval again. Add tests that one company never sees another company's playbook through skills/list, skills/get or resources/read, and that a member can't call the admin tool.
```

**6. Refine tools vs. skills.**

```text
Review the tool descriptions and the skill together. Move procedural how-to guidance out of tool descriptions and into the skill, and keep tool descriptions about what each tool does.
```

**7. Test it.**

```text
Deploy and give me prompts that should trigger each skill in each client, signed in as users from two different companies. Tell me how to confirm the skill was actually used, what the approval prompt looks like, and how to check the skills with the MCP Inspector where a client doesn't support them yet.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-10-skills-over-mcp/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: the shared skill holds no company's data and uses only company-scoped tools, each company's playbook is listed and served only to that company (tested through `skills/list`, `skills/get` and `resources/read`), and per-company results are never cached as public
- [ ] It works in claude.ai or Claude Desktop. If Claude doesn't support this extension yet, the rest of the server still works there and your NOTES.md records the support status from the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix)
- [ ] It works in ChatGPT (custom MCP server), with the same exception and NOTES.md entry if ChatGPT doesn't support this extension yet
- [ ] Project defence recorded (up to five minutes: it works, one tool end to end, one tenant-isolation decision, one change you made; see [SUBMITTING.md](../SUBMITTING.md#10-defend-your-project)) and linked in the form
- [ ] Tagged `step-10-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-10-done`, live server URL, and screenshots of the monthly-sales-review skill and a company playbook in use (or a note on client support if it isn't available yet) in **Claude** and **ChatGPT**

## References

- [MCP: Skills over MCP](https://modelcontextprotocol.io/extensions/skills/overview)

---

← [Step 09: Generative UI with MCP Apps](../step-09-generative-ui-mcp-apps/) · [Roadmap](../README.md#roadmap) · [Step 11: Per-User State with Durable Objects](../step-11-durable-objects/) →
