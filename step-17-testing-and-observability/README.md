# Step 17: Testing, Evals and Observability

Prove the server works, keep proving it on every change, and see what it does once real users arrive. You add automated tests that run the Worker the way Cloudflare does, integration tests that call the server over MCP, an **eval set** that checks the model picks the right tool and widget for realistic requests in Claude and ChatGPT, and structured logs, traces and metrics on Cloudflare.

> 🏬 **Stockroom's ask:** Stockroom ships changes every week and can't afford a reworded tool description that silently breaks a customer's workflow. It wants tests, evals and logs that catch it first.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.6 How do you trust a connector an agent wrote?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/specify-generate-verify/). Writing the checks before trusting what a coding agent built.
- [17.10 How do you see what a connector did?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/watching-a-connector/). What to log and measure once the server is live.
- [Field guide F7, Seeing the server's spans](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f7-seeing-the-servers-spans). Tracing tool calls on Cloudflare.
- [18.13 How do you prove companies stay apart?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/proving-the-sign-in-keeps-companies-apart/). Tests that prove isolation, not just assume it.
- [18.16 Who did what, for which company?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/watching-who-did-what/). Logs that answer "who did this?".
- [MCP Apps field guide F1, Testing a page](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/field-guide-mcp-apps/#f1-testing-a-page). Testing widgets.
- *Optional:* [16.8 Watching it run: telemetry](https://agentfactory.panaversity.org/building-ai-workers/skills-and-plugins/watching-it-run/). Telemetry for skills.

**Official docs.** Keep these open while you build:

- [Cloudflare Workers: Vitest integration](https://developers.cloudflare.com/workers/testing/vitest-integration/). Running tests inside the Workers runtime, with bindings.
- [MCP Inspector](https://modelcontextprotocol.io/docs/2026-07-28/tools/inspector). Calling the server by hand while you debug.
- [Workers Logs](https://developers.cloudflare.com/workers/observability/logs/workers-logs/). Storing and searching structured logs.
- [Workers traces](https://developers.cloudflare.com/workers/observability/traces/) and [custom spans](https://developers.cloudflare.com/workers/observability/traces/custom-spans/). Following one tool call through the database, queues and workflows.
- [Workers metrics and analytics](https://developers.cloudflare.com/workers/observability/metrics-and-analytics/). Requests, errors and CPU time.
- [Claude API: MCP connector](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector). Calling your remote MCP server from the Claude API, so the eval set can run automatically.

## Why this step matters

By now the server has many moving parts, and a change to one tool description can make the model stop choosing it. Tests catch broken code, evals catch broken behaviour, and logs and traces tell you what happened when a client reports a problem. These are also what let you change a client's server with confidence, which is what they are paying you for.

## Why this technology in an MCP server: tests, evals and observability

- **What it is.** Unit and integration tests run with Vitest inside the Workers runtime; an eval set of realistic prompts with the expected tool and widget for each; structured logs, traces and metrics in Cloudflare.
- **Why an MCP server uses it.** An MCP server has two kinds of correctness: the code must do the right thing, and the model must choose the right tool from your descriptions. Ordinary tests cover the first, and evals cover the second: an automated run through the Claude API on every release, plus a manual pass in each client. Observability covers what neither can predict.
- **What it solves.** Regressions shipped to clients, tools the model silently stops using after a description change, and bug reports you can't reproduce.
- **Alternatives.** Jest or another test runner (Vitest is what Cloudflare supports for the Workers runtime), hosted eval tools, and monitoring services such as Sentry, Datadog or any OpenTelemetry backend.

## What you will learn

- Run unit tests in the Workers runtime with real bindings (KV, D1, Durable Objects, Queues)
- Write integration tests that call the server over MCP, signed in as test users
- Build an eval set of realistic prompts with expected tools and widgets, run it automatically through the Claude API's MCP connector, and run a manual pass in Claude and ChatGPT
- Log every tool call as structured data, without secrets or full personal data
- Trace a tool call across Neon, queues and workflows, and read metrics
- Run tests in CI on every push

## What you will build

A test suite (unit, integration and isolation tests), an `evals/` folder with 30 prompts and their expected tools and widgets plus a results table per client, structured logging and tracing for every tool call, a CI workflow that runs the tests, and a short `OBSERVABILITY.md` on where to look when something breaks.

## Key concepts

- **Unit test**: Checks one function on its own.
- **Integration test**: Calls the server the way a client does, over MCP.
- **Eval**: A realistic prompt with the expected tool or widget, run against a real model to check behaviour.
- **Structured log**: A log line as data (tool, user, duration, result) that you can search and count.
- **Trace**: The path of one request through every service it touched.

## Outline

1. What to test in an MCP server
2. Unit tests in the Workers runtime
3. Integration and isolation tests over MCP
4. Evals for tool and widget selection
5. Structured logs, traces and metrics
6. CI on every push

## Prerequisites

- [Step 16](../step-16-security-hardening/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 16 code into `step-17-testing-and-observability/`.**

```text
Copy my step-16 code into step-17-testing-and-observability/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-17 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-17 from my step-16 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan the testing.**

```text
List what this server needs tested: each tool, resource, prompt, widget, Durable Object, queue consumer, workflow, task and event, plus auth and per-user isolation. Propose a test plan with unit tests, integration tests over MCP and an eval set, and tell me what each one catches that the others don't.
```

**3. Add unit and integration tests.**

```text
Set up Vitest with Cloudflare's Workers integration so tests run in the Workers runtime with our bindings. Write unit tests for the core logic and integration tests that call the server over MCP as two test users, including tests that prove one user can't see the other's data.
```

**4. Build the eval set.**

```text
Create evals/ with 30 realistic prompts from a sales manager and staff, each with the expected tool, arguments and widget. Write an npm run evals script that sends each prompt to the Claude API with my staging server attached through the MCP connector, checks which tool the model called, and writes a results table (it needs an Anthropic API key in .dev.vars and costs a little per run). Then help me run a manual pass in claude.ai, Claude Desktop and ChatGPT, record pass or fail per client, and suggest description changes for any that fail.
```

**5. Add observability.**

```text
Log every tool call as structured JSON (tool, user ID, duration, result, error type, never secrets or full personal data), enable Workers Logs and traces, add custom spans around database, queue and workflow calls, and write OBSERVABILITY.md explaining where to look when something breaks.
```

**6. Run it in CI and deploy.**

```text
Add a CI workflow that runs the tests on every push. Deploy, trigger a few tool calls from Claude and ChatGPT, and show me the logs, a trace and the metrics for them in the Cloudflare dashboard.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)
- Vitest in the Workers runtime, and CI

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-17-testing-and-observability/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tests pass in CI, and evals/ has results from the automated run and from the manual pass in both clients
- [ ] Tenant isolation: the cross-tenant suite runs in CI on every push
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-17-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-17-done`, live server URL, and screenshots of passing CI, your eval results, and a trace of a tool call from **Claude** or **ChatGPT**

## References

- [Cloudflare Workers: testing](https://developers.cloudflare.com/workers/testing/vitest-integration/)
- [Cloudflare Workers: observability](https://developers.cloudflare.com/workers/observability/logs/workers-logs/)

---

← [Step 16: Security Hardening for MCP Servers](../step-16-security-hardening/) · [Roadmap](../README.md#roadmap) · [Step 18: Deploying and Testing on claude.ai, Claude Desktop and ChatGPT](../step-18-deploy-and-test/) →
