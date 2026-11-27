# Step 18: Deploying and Testing on claude.ai, Claude Desktop and ChatGPT

Take the server to production on Cloudflare and verify it properly across all target clients: separate environments, a custom domain, secrets per environment, a CI pipeline that runs the step 17 tests and gates each deploy, safe releases with rollbacks, and a cross-client test plan.

> 🏬 **Stockroom's ask:** Stockroom is going live with paying customers. It needs staging and production, safe releases with a rollback, and proof that everything works in each AI app its customers use.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.8 How do you put a connector online safely?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/securing-a-remote-server/). Access keys, Origin checks and staging.
- [Field guide F6, Deploying to Cloudflare Workers](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f6-deploying-to-cloudflare-workers). Deploying step by step.
- [17.6 How do you trust a connector an agent wrote?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/specify-generate-verify/). Testing what the agent built.
- [20.16 How do today's products do this?, "How to show your connector's screen in your app"](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/how-todays-tools-realize-it/#how-to-show-your-connectors-screen-in-your-app) and [MCP Apps field guide F1, Testing a page](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/field-guide-mcp-apps/#f1-testing-a-page). What Claude and ChatGPT support today, and how to connect and test in each, step by step.

**Official docs.** Keep these open while you build:

- [Wrangler: environments](https://developers.cloudflare.com/workers/wrangler/environments/). Separate dev, staging and production.
- [Cloudflare Workers: secrets](https://developers.cloudflare.com/workers/configuration/secrets/). Secrets per environment.
- [Workers Logs](https://developers.cloudflare.com/workers/observability/logs/workers-logs/). Viewing your structured tool-call logs.
- [Workers metrics and analytics](https://developers.cloudflare.com/workers/observability/metrics-and-analytics/). Requests, errors and CPU time.
- [Cloudflare: GitHub Actions](https://developers.cloudflare.com/workers/ci-cd/external-cicd/github-actions/). Running tests and deploys in CI.
- [Cloudflare: rollbacks](https://developers.cloudflare.com/workers/versions-and-deployments/rollbacks/). Undoing a bad release.
- [Neon: branching](https://neon.com/docs/introduction/branching). A database branch per environment.
- [MCP security best practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices). The release security checklist.

## Why this step matters

A server that works on your laptop isn't done. This step covers what it takes to run reliably for real users in Claude and ChatGPT, and how to catch client-specific differences before your users do.

## Why this technology in an MCP server: environments, CI/CD and safe releases

- **What it is.** Separate dev, staging and production environments with Wrangler, each with its own secrets, D1, Neon branch and bindings; a CI/CD pipeline that tests and deploys; versioned releases with rollbacks; and a manual test plan for each client.
- **Why an MCP server uses it.** An MCP server is used by real people through several hosts, and Claude and ChatGPT don't behave identically. Clients paying for your server expect it to stay up, to be changed safely and to be fixed quickly when something breaks.
- **What it solves.** Releasing untested changes straight to clients, not knowing why a tool call failed, and client-specific bugs that only users discover.
- **Alternatives.** Other CI/CD services, Cloudflare's own Git integration (Workers Builds), or deploying by hand from a laptop, which doesn't scale past one developer and one client.

## What you will learn

- Set up separate dev, staging and production environments with Wrangler
- Manage secrets and configuration safely
- Run the step 17 tests and automated evals in a CI/CD pipeline that gates every deploy
- Release safely with versions, gradual rollout and rollbacks
- Run a manual test plan in claude.ai, Claude Desktop and ChatGPT
- Watch production with the logs and traces from step 17 after each release

## What you will build

A production deployment of the sales server on a custom domain, staging and production environments, a CI/CD pipeline that tests before it deploys, a release checklist with rollback steps, and a cross-client test plan.

## Key concepts

- **Environments**: Isolated configs and databases per stage.
- **CI/CD**: Tests run automatically on every push, and deploys happen only when they pass.
- **Rollback**: Returning production to the previous version in one step.
- **Cross-client testing**: Same server, different hosts. Verify behavior in each.

## Outline

1. Environments and custom domains
2. Secrets and configuration
3. CI/CD with the step 17 tests
4. Releases and rollbacks
5. Cross-client manual test plan
6. Release checklist

## Prerequisites

- [Step 17](../step-17-testing-and-observability/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 17 code into `step-18-deploy-and-test/`.**

```text
Copy my step-17 code into step-18-deploy-and-test/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-18 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-18 from my step-17 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Plan the production setup.**

```text
Plan a production setup for this server on Cloudflare: separate dev, staging and production environments, custom domain, secrets, separate D1 and Neon databases per environment, and the Durable Object, Queues and Workflow from steps 11 to 13 bound in each environment.
```

**3. Configure environments.**

```text
Implement the environments in the wrangler config and add npm scripts to deploy each one. Make sure no secret is committed.
```

**4. Add CI/CD.**

```text
Extend the CI workflow from step 17 so every push runs the tests, merges to main deploy to staging, and a manual approval deploys to production. Before that approval, run the step 17 automated eval script against staging (with the Anthropic API key as a CI secret) and block the release if the pass rate drops.
```

**5. Run the cross-client test plan.**

```text
Write a manual test plan as a markdown checklist covering every tool and widget in claude.ai, Claude Desktop and ChatGPT, then walk me through running it.
```

**6. Prepare the release.**

```text
Create a release checklist covering versioning, changelog, gradual rollout, rollback and what to watch in the logs and traces after each release. Then do a practice rollback.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-18-deploy-and-test/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Tenant isolation: the cross-tenant suite passes against staging before each production release, and no tenant data or settings are shared between environments
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Project defence recorded (up to five minutes: it works, one tool end to end, one tenant-isolation decision, one change you made; see [SUBMITTING.md](../SUBMITTING.md#10-defend-your-project)) and linked in the form
- [ ] Tagged `step-18-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-18-done`, live server URL, and screenshots of your production server passing the cross-client test plan in **Claude** and **ChatGPT**

## References

- [Cloudflare: Model Context Protocol](https://developers.cloudflare.com/agents/model-context-protocol/)
- [Extensions client support matrix](https://modelcontextprotocol.io/extensions/client-matrix)

---

← [Step 17: Testing, Evals and Observability](../step-17-testing-and-observability/) · [Roadmap](../README.md#roadmap) · [Step 19: Generative UI: An Intent-Driven Sales Dashboard](../step-19-generative-ui-dashboard/) →
