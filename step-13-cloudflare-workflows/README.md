# Step 13: Durable Jobs with Cloudflare Workflows

Some jobs take minutes, run in several stages, wait for a person, or must survive a crash. Your MCP server handles them with Cloudflare Workflows: the Workflow is defined and bound inside the same Worker that serves `/mcp`, and runs each job as a series of durable steps: each finished step is saved, a failed step is retried, and the job picks up where it left off. An MCP tool starts the job and returns a run ID straight away, and a second tool lets the model check on it. This uses plain tools, so it works in Claude and ChatGPT today, and step 14 puts the standard MCP Tasks extension on top of it.

> 🏬 **Stockroom's ask:** Year-end sales reports take minutes to build and wait for a manager's approval before they are published. Stockroom wants these jobs to survive crashes and let the model check on them.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

> **Note:** Chapters 12 to 20 of the book, which this bootcamp follows, don't cover durable execution or Cloudflare Workflows. The lessons below give the ideas this step relies on, and the [Cloudflare Workflows docs](https://developers.cloudflare.com/workflows/) teach the feature itself.

- [17.4 What if a different copy answers the next call?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/stateless-servers/). Why a job's progress can't live in the Worker's memory.
- [14.8 Async and await](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/async-and-await/). Waiting on long work in TypeScript.
- [20.14 How does a program sign in with no person?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/machines-with-no-person/). Long work that a client checks on later, which this step builds by hand and step 14 makes standard.
- *Optional:* [17.3 What should a tool say when it fails?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/errors-that-say-what-to-do-next/). Reporting a failed run so the model knows what to do.

**Official docs.** Keep these open while you build:

- [Cloudflare Workflows](https://developers.cloudflare.com/workflows/). Steps, retries, sleeping and waiting for events.
- [Workflows limits](https://developers.cloudflare.com/workflows/reference/limits/). Step counts, durations and running instances.
- [Workflows pricing](https://developers.cloudflare.com/workflows/reference/pricing/). What durable jobs cost, including the free plan.

## Why this step matters

Real client work includes jobs that don't fit in one request: a year-end report, a month-end close, an approval that waits for a manager. A Worker request that runs too long gets cut off, and anything kept in memory is lost. Workflows gives these jobs a durable home inside the MCP server's own Worker, with retries and waiting built in and nothing extra to host. From this step on, every long or multi-stage job the server runs is a Workflow. This bootcamp standardizes on Workflows for durable execution, and step 14 builds on it.

## Why this technology in an MCP server: Cloudflare Workflows

- **What it is.** Workflows is Cloudflare's durable execution engine. A job is written as named steps whose results are saved, so it survives crashes and redeploys, retries failed steps, and can sleep or wait for an outside event.
- **Why an MCP server uses it.** Some client jobs are multi-stage and slow, or wait for a person: a year-end report, an approval. A tool starts a Workflow and returns a run ID, and the model checks its status with another tool, so long jobs fit the request-and-reply shape of MCP.
- **What it solves.** Jobs that restart from scratch after a failure, approvals with nowhere to wait, and progress nobody can see.
- **Alternatives.** Chaining queue messages or building a state machine on Durable Object alarms by hand, or other engines such as Temporal and Inngest. This bootcamp standardizes on Workflows because they run inside the same Worker.

## What you will learn

- Explain durable execution: named steps whose results are saved, so a job survives crashes, retries and redeploys
- Write a Workflow class with `step.do` and bind it to your MCP server's Worker, so the server's tools can start and track it
- Start a Workflow instance from an MCP tool and return its run ID right away
- Let the model check progress and fetch the result with a status tool
- Pause a job with `step.sleep`, and wait for a person's approval with `step.waitForEvent`
- Make each step safe to retry (idempotent), and scope every run to the signed-in user
- Know the limits and pricing that matter (Workflows is included in the Workers free plan)

## What you will build

A `generate_sales_report` Workflow that reads a year of Neon sales data in pages, computes the summaries and saves the finished report to Neon. Three tools drive it: `start_sales_report` (returns a run ID), `get_report_status` (shows the current step and, when done, the report) and `approve_report` (sends the approval event that a final "publish" step waits for).

## Key concepts

- **Durable execution**: Running a job as saved steps, so a crash or redeploy resumes the job instead of restarting it.
- **Workflow instance**: One run of a Workflow, with its own ID and status that any copy of your Worker can look up.
- **Steps (`step.do`)**: A named unit of work whose result is saved. A step that already finished is not run again.
- **Retries and idempotency**: Failed steps are retried, so each step must give the same result if it runs twice.
- **Sleeping and waiting (`step.sleep`, `step.waitForEvent`)**: A paused instance uses no CPU while it waits for a time or an outside event.
- **Workflow, Queue or Durable Object**: A Workflow is one job with ordered steps, waits and a status to check. A Queue (step 12) is many independent pieces of work. A Durable Object (step 11) is small per-user state that must always be consistent.

## Outline

1. Why long jobs don't fit in a tool call
2. Durable execution and the Workflows model
3. Writing the report Workflow
4. Starting it from a tool and checking its status
5. Waiting for approval with `step.waitForEvent`
6. Retries, idempotency and per-user scoping
7. Limits, pricing and watching runs in the Cloudflare dashboard
8. Testing in Claude and ChatGPT

## Prerequisites

- [Step 12](../step-12-queues/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 12 code into `step-13-cloudflare-workflows/`.**

```text
Copy my step-12 code into step-13-cloudflare-workflows/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-13 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-13 from my step-12 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan the Workflow.**

```text
Read the Cloudflare Workflows docs (https://developers.cloudflare.com/workflows/) and explain durable execution, steps, retries, step.sleep and step.waitForEvent in plain words. Then plan a generate_sales_report Workflow for this server: its steps, what each step saves, and the tools that start it and check on it. Show me the plan before writing code.
```

**3. Write the Workflow.**

```text
Create the generate_sales_report Workflow class and bind it in the wrangler config. Split the work into named steps: read a year of orders from Neon in pages, compute totals by month, product and customer, and save the report to a reports table in Neon owned by the user. Make each step safe to retry, and explain why each one is idempotent.
```

**4. Drive it from tools.**

```text
Add start_sales_report, which creates a Workflow instance for the signed-in user and returns its run ID right away, and get_report_status, which returns the current step, any error and, when finished, a short summary of the report. Pass the step 7 tenant envelope as the Workflow's params, run every step's queries through the tenant router, and make sure a user can only see their own company's runs. Set the right tool annotations.
```

**5. Wait for approval.**

```text
Add a final publish step that waits for an approval event with step.waitForEvent and a sensible timeout. Add an approve_report tool that sends that event to the user's run, and only for an admin of the company (the role from step 4). Right before publishing, re-check in the database that the approver is still an admin of that company and the run's starter is still a member; if either has lost the right, end the run as refused with a clear reason. Explain what happens to the instance while it waits.
```

**6. Deploy and test.**

```text
Deploy, then give me prompts to try in Claude and ChatGPT that start a report, check on it until it finishes, and approve it. Show me where to watch the runs in the Cloudflare dashboard, and how to test that a failed step is retried.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-13-cloudflare-workflows/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: every Workflow run carries the tenant envelope, and status and approval tools refuse another company's runs
- [ ] Authority at run time: a test demotes the approver between approval and publish, and the report is not published
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-13-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-13-done`, live server URL, and screenshots of a report being started, checked until it finishes, and approved in **Claude** and **ChatGPT**

## References

- [Cloudflare Workflows](https://developers.cloudflare.com/workflows/)
- [Rules of Workflows](https://developers.cloudflare.com/workflows/build/rules-of-workflows/)
- [Workflows limits](https://developers.cloudflare.com/workflows/reference/limits/)
- [Workflows pricing](https://developers.cloudflare.com/workflows/reference/pricing/)

---

← [Step 12: Background Work with Queues](../step-12-queues/) · [Roadmap](../README.md#roadmap) · [Step 14: Long-Running Work with Tasks](../step-14-tasks/) →
