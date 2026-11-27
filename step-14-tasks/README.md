# Step 14: Long-Running Work with Tasks

Handle work that takes longer than a single request, such as reports, imports and batch jobs, using the MCP Tasks extension so the client can start, track and retrieve results later. The work itself keeps running on the Cloudflare Workflow you built in step 13, and Tasks becomes the standard MCP face for it.

> 🏬 **Stockroom's ask:** Stockroom wants its long jobs to appear as standard MCP tasks, so Claude can track progress and fetch the result without Stockroom's own status tools.

> **Note:** Tasks (`io.modelcontextprotocol/tasks`) is a Beta extension on its way into the core spec. Anthropic announced on November 24, 2026 that Tasks is generally available in Claude and Claude Code for Pro, Max and Ultra plans, and still rolling out to Team and Enterprise. Check the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix) for ChatGPT and other clients, and keep the step 13 tools as the fallback.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

> **Note:** The book covers the Tasks extension in one section of lesson 20.14 and as an optional lab task, not in a full lesson yet. Use the [MCP Tasks docs](https://modelcontextprotocol.io/extensions/tasks/overview) to fill the gaps.

- [20.14 How does a program sign in with no person?, section "What is the Tasks extension?"](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/machines-with-no-person/#what-is-the-tasks-extension). Task handles, `tasks/get` and `input_required`.
- [14.8 Async and await](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/async-and-await/). Waiting on long work.
- [20.1 What is an MCP extension?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/what-an-mcp-extension-is/). Tasks is in beta, and how fallbacks work.

**Official docs.** Keep these open while you build:

- [MCP: Tasks](https://modelcontextprotocol.io/extensions/tasks/overview). Task handles, `tasks/get` and the task lifecycle.
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix). Current Claude and ChatGPT support.
- [Cloudflare Workflows](https://developers.cloudflare.com/workflows/). The engine behind each task.

## Why this step matters

Real business operations don't always finish in seconds. In step 13 the model had to call your own status tool to check on a job. Tasks is the standard way to do the same thing, so hosts that support it can track the job, show its progress and fetch the result without custom tools.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../assets/diagram-tasks-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="../assets/diagram-tasks-light.png">
  <img src="../assets/diagram-tasks-light.png" alt="Claude starts generate_sales_report and gets a task handle at once. A Cloudflare Workflow runs the job while Claude keeps working. The task moves to input_required, an elicitation form asks the user to confirm an 18-month report, the answer returns through tasks/update, the Workflow resumes, and Claude receives the finished report." width="100%">
</picture>

## Why this technology in an MCP server: The MCP Tasks extension

- **What it is.** Tasks is the MCP extension for long-running work: a call returns a task handle right away, and the client checks it with `tasks/get` until the result is ready.
- **Why an MCP server uses it.** In step 13 the model had to use your own status tools. Tasks is the standard way to do the same thing, so any host that supports it can track progress and fetch results from any server without custom tools.
- **What it solves.** Every server inventing its own start-and-check tools, and long calls that time out or block the conversation.
- **Alternatives.** Custom status tools (kept as the fallback for clients without Tasks support) or progress notifications on a long-held request, which still times out.

## What you will learn

- Understand the task lifecycle: create, poll or notify, complete, retrieve the result
- Turn a slow tool into a task-based tool
- Back each task with the Cloudflare Workflow instance from step 13, so a task handle maps to a Workflow run
- Keep step 13's status tools as the fallback for clients that don't support Tasks yet
- Report progress and handle cancellation
- Pause a task for the user's answer (the `input_required` status, answered with `tasks/update`), using an elicitation form, for example to confirm a very large report before it runs
- Show task status and results with an MCP Apps widget

## What you will build

The step 13 `generate_sales_report` Workflow, now exposed as an MCP task: starting it returns a task handle (the Workflow instance ID, checked against the caller's company), the client reads progress and the finished report through `tasks/get`, cancelling the task terminates the Workflow run, and a `TaskStatus` widget shows progress and then the report. `start_sales_report` and `get_report_status` stay as the fallback for clients without Tasks support.

## Key concepts

- **Task handle**: An ID the client keeps for work that outlives a request, and checks with `tasks/get`.
- **Task lifecycle**: Working, input required, completed, failed or cancelled, each mapped from the Workflow instance's status.
- **Handle to Workflow run**: Each task ID is one Workflow instance, so any copy of the stateless Worker can answer for it.
- **Ownership check**: Every `tasks/get` and cancel verifies the task belongs to the caller's company, so a guessed ID reveals nothing.
- **Progress and cancellation**: Keeping the user informed and in control.
- **Fallback tools**: Plain start and status tools for clients that don't support Tasks yet.

## Outline

1. Why synchronous tools fall short
2. The Tasks extension
3. Implementing a task on Cloudflare
4. Progress, cancellation and errors
5. A task-status widget

## Prerequisites

- [Step 13](../step-13-cloudflare-workflows/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 13 code into `step-14-tasks/`.**

```text
Copy my step-13 code into step-14-tasks/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-14 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-14 from my step-13 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Check the Tasks extension's current client support, then plan.**

```text
Read the MCP Tasks extension docs linked in this step and explain the task lifecycle. Then plan how to expose the generate_sales_report Workflow from step 13 through the Tasks extension: how a task handle maps to a Workflow instance, how status and results are read, and how the step 13 status tools stay as a fallback.
```

**3. Turn the Workflow into a task.**

```text
Make generate_sales_report task-based: starting it creates a Workflow instance and returns a task handle right away, and tasks/get reads the instance's status. Keep start_sales_report and get_report_status working for clients without Tasks support.
```

**4. Add progress and cancellation.**

```text
Report progress from the Workflow's steps, support cancellation by terminating the Workflow instance, and return the saved report as the task's result. When a report would cover more than a year of orders, move the task to input_required with an elicitation form that states the size and asks the user to confirm, have the Workflow wait for the answer (step.waitForEvent), and continue or cancel when the client sends it with tasks/update. Check that the answer comes from the same company and user who started the task, and that this user is still a member of that company when the answer arrives (a task whose user was removed while it waited is cancelled with a clear reason).
```

**5. Show it in the UI.**

```text
Add a TaskStatus widget that shows progress and then the finished report.
```

**6. Test it.**

```text
Add tests that a task ID from another company is refused and that cancelling a task terminates its Workflow run. Then deploy and give me a test plan for starting, watching, cancelling and retrieving a report in each client, using the MCP Inspector where a client doesn't support Tasks yet.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-14-tasks/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: a task ID from another company is refused by `tasks/get` and by cancel
- [ ] Authority at run time: a test removes the user while the task waits in `input_required`, and the task is cancelled instead of continuing
- [ ] It works in claude.ai or Claude Desktop. If Claude doesn't support this extension yet, the rest of the server still works there and your NOTES.md records the support status from the [client support matrix](https://modelcontextprotocol.io/extensions/client-matrix)
- [ ] It works in ChatGPT (custom MCP server), with the same exception and NOTES.md entry if ChatGPT doesn't support this extension yet
- [ ] Tagged `step-14-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-14-done`, live server URL, and screenshots of a report task running and finishing (or a note on client support if it isn't available yet) in **Claude** and **ChatGPT**

## References

- [MCP: Tasks](https://modelcontextprotocol.io/extensions/tasks/overview)

---

← [Step 13: Durable Jobs with Cloudflare Workflows](../step-13-cloudflare-workflows/) · [Roadmap](../README.md#roadmap) · [Step 15: Events](../step-15-events/) →
