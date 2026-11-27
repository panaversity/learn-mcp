# Step 20: Billing and Distribution: Charging for Your Server and Listing It

Turn the server into something you can sell. You add plans and payment with Stripe, check every tool call against the company's plan and usage limits on the server, and prepare the server for listing in Claude's Connectors Directory and in ChatGPT's plugin directory: the review checklist, privacy policy, screenshots and test account a reviewer needs.

> 🏬 **Stockroom's ask:** Stockroom needs revenue: a plan for each customer company, usage limits that protect its costs, and listings in Claude's and ChatGPT's directories so new customers can find it.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.9 How do you charge for a connector?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/charging-for-a-connector/). Value on the server, a plan check on every call, and payment outside the AI.
- [20.15 How do the pieces fit together?, "Where does the check of who has paid go?"](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/how-the-pieces-fit/#where-does-the-check-of-who-has-paid-go). The paid-plan check stays on the server, and no screen holds a checkout.
- [17.5 When is a connector worth writing?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/built-in-custom-skill-or-connector/). What a client will actually pay for.
- [20.16 How do today's products do this?, "Product reference"](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/how-todays-tools-realize-it/#product-reference). Where MCP Apps run today and each client's rules, including Claude's directory badge and the screenshots a submission needs.
- [MCP Apps field guide F2, The interactive-connector checklist](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/field-guide-mcp-apps/#f2-the-interactive-connector-checklist). The checks to pass before you publish.

**Official docs.** Keep these open while you build:

- [Stripe Checkout](https://docs.stripe.com/payments/checkout) and [Stripe subscriptions](https://docs.stripe.com/billing/subscriptions/overview). Taking payment for plans, on Stripe's own pages.
- [MCP: Elicitation, URL mode](https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation#url-mode-elicitation-requests). Sending the user to a page such as Stripe Checkout, so payment details never pass through the chat.
- [Stripe webhooks](https://docs.stripe.com/webhooks). Updating a company's plan when a payment succeeds or a subscription ends.
- [Claude: directory or custom connector](https://claude.com/docs/connectors/building/directory-vs-custom). When to list publicly and when to share a custom connector.
- [Claude: submitting to the Connectors Directory](https://claude.com/docs/connectors/building/submission) and [review criteria](https://claude.com/docs/connectors/building/review-criteria). What Anthropic checks.
- [OpenAI: upload and submit your plugin](https://developers.openai.com/plugins/deploy/submission). How ChatGPT's plugin directory (which replaced its app directory) accepts a plugin built on your MCP server.

## Why this step matters

Skills only turn into income when someone pays. Most clients will pay you to build and run a server for them, and some products are sold to many users at once. Either way, you need plans, limits that protect your costs, and a way for users to find and connect the server. Directory listing also signals to clients that the server passed a review.

## Why this technology in an MCP server: billing and directory listing

- **What it is.** Stripe for plans, checkout and subscriptions; a plan and usage check on the server for every tool call, using the Durable Object counters you already have; and the materials Claude's and ChatGPT's directories require.
- **Why an MCP server uses it.** The model and the chat window can't be trusted to enforce payment, so the server decides on every call whether this company's plan allows it. Payment itself happens on Stripe's pages, outside the chat, as the book recommends. Directories are how most users will discover and add a connector.
- **What it solves.** Unpaid use that costs you money, users who can't find or trust your server, and rejected submissions because a privacy policy, test account or screenshots were missing.
- **Alternatives.** Other payment providers such as Paddle or Lemon Squeezy, invoicing a client directly for a private server, or sharing a custom connector URL instead of a public listing.

## What you will learn

- Design plans (for example free, pro and team) with usage limits that match your costs
- Take payment with Stripe Checkout and keep the company's plan in sync with Stripe webhooks
- Check the company's plan and usage on every tool call, with a clear message and upgrade link when a limit is reached
- Keep checkout out of the chat and out of widgets
- Ask the user to approve a cost before the server spends it, with elicitation: a form that states the cost, or a URL-mode link to Stripe Checkout so card details never pass through the chat
- Prepare a directory submission: review checklist, privacy policy, terms, test account and screenshots
- Choose between a public listing and a custom connector for a single client

## What you will build

Three plans stored in Neon, a Stripe Checkout link and webhook handler that update the company's plan, per-plan monthly limits counted per company in a Durable Object named from the company ID and checked on every tool call, a `get_my_plan` tool and widget that show usage and an upgrade link that opens Stripe outside the chat, and a `DISTRIBUTION.md` with the completed directory checklist and the submission materials for Claude and ChatGPT.

## Key concepts

- **Plan check on every call**: The server, not the model or the widget, decides if a call is allowed.
- **Usage metering**: Counting calls or work per company per period, here in a Durable Object named from the company ID, so one company's usage never counts against another's.
- **Checkout outside the chat**: Payment happens on Stripe's own page, reached by a link.
- **Webhooks**: Stripe tells your server when a payment succeeds or a subscription changes.
- **Cost confirmation**: Before an action that costs money, the server asks the user through elicitation, as Supabase's server does before it creates a paid project. Payment itself happens at a URL (URL mode), never in a form.
- **Directory review**: The checks a client's directory runs before listing your server.

## Outline

1. What to charge for, and plan design
2. Stripe Checkout and subscriptions
3. Webhooks and keeping the plan in sync
4. Plan and usage checks on every tool call
5. Showing usage and upgrades without a checkout in the chat
6. Billing attack tests: the wrong company, the wrong person, repeated and late events, races and cancellations
7. Preparing for Claude's and ChatGPT's directories
8. Testing in Claude and ChatGPT

## Prerequisites

- [Step 19](../step-19-generative-ui-dashboard/)
- The tools and accounts listed in the [main README](../README.md#prerequisites), plus a free [Stripe](https://stripe.com/) account in test mode

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 19 code into `step-20-billing-and-distribution/`.**

```text
Copy my step-19 code into step-20-billing-and-distribution/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-20 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-20 from my step-19 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and design the plans.**

```text
Read book lesson 17.9 and the "Where does the check of who has paid go?" section of lesson 20.15 (both linked in this step). Propose three plans for this sales server with monthly limits that cover my Cloudflare and Neon costs, and plan how the server checks the plan on every tool call, where usage is counted, and how payment happens outside the chat.
```

**3. Add Stripe.**

```text
Using Stripe in test mode, create the products and prices, add a Checkout link per plan, and add a webhook endpoint to the Worker that verifies Stripe's signature and updates the company's plan in Neon. Create each Checkout session on the server for the signed-in company, only for its owner or admin, and record the session ID against that company in Neon; when the webhook arrives, find the company from that stored record, never from metadata or anything the client sent. Process each Stripe event once (store its event ID) and ignore an event older than the plan change you already applied. Store the Stripe keys as Wrangler secrets.
```

**4. Check the plan on every call.**

```text
Count each company's monthly usage in a Durable Object named from the company ID (org:<orgId>) and check it, with the company's plan, at the start of every tool call. When a limit is reached, return a clear, recoverable message with an upgrade link that opens outside the chat. Add a get_my_plan tool and a small widget showing plan and usage, with no checkout inside it. When the user asks to upgrade from the chat, return input_required with a URL-mode elicitation that opens the Stripe Checkout page (never a form asking for card details), and treat the plan as changed only when Stripe's webhook says so. Before an action that uses a large share of the plan, such as a year-long report, confirm with a form-mode elicitation that states how much of the monthly limit it will use. Reuse the sealed requestState helper from step 16, and keep the plain upgrade link as the fallback for clients without elicitation.
```

**5. Prove the billing can't be tricked.**

```text
Add billing tests that must pass before this step is done: a member who isn't an owner or admin tries to upgrade the company and is refused; a webhook for a Checkout session recorded against company A can't change company B's plan, even when its metadata names B; the same event delivered twice changes the plan once; an older event arriving after a newer one doesn't undo it; twenty tool calls at once can't go past the usage limit (the Durable Object counts them one at a time); and a cancellation while a report or import is queued stops the work at its next run-time check instead of finishing on a plan the company no longer has. Run them and show me the results.
```

**6. Prepare the directory submissions.**

```text
Read Claude's directory submission and review-criteria docs and the build-mcp-app skill's directory checklist, and OpenAI's plugin submission guide. Create DISTRIBUTION.md with a completed checklist, a privacy policy and terms draft, a test account plan for reviewers, and the screenshots each directory needs. Tell me what's still missing before I could submit.
```

**7. Deploy and test.**

```text
Deploy, then walk me through a test-mode purchase, hitting a limit and upgrading, and checking get_my_plan, in Claude and ChatGPT.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)
- Stripe test mode

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-20-billing-and-distribution/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only, and no Stripe keys)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] Stripe runs in test mode only, and the webhook verifies signatures
- [ ] Billing tests pass: a non-admin upgrade is refused, a session for one company can't change another's plan, duplicate and out-of-order events are handled, concurrent calls can't pass the limit, and a cancellation stops queued work
- [ ] An upgrade from the chat goes through a URL-mode elicitation to Stripe Checkout (or the link fallback), and a costly action asks for confirmation first
- [ ] DISTRIBUTION.md has the directory checklist and submission materials (you don't have to actually submit)
- [ ] Tenant isolation: plans and usage limits are per company, and one company's usage never counts against another's
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-20-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-20-done`, live server URL, and screenshots of a usage limit being reached and of get_my_plan after a test-mode upgrade in **Claude** and **ChatGPT**

## References

- [Stripe documentation](https://docs.stripe.com/)
- [Claude: Connectors Directory submission](https://claude.com/docs/connectors/building/submission)
- [Built-in skill: `build-mcp-app` directory checklist (see `.claude/skills/`)](../.claude/skills/build-mcp-app/references/directory-checklist.md)

---

← [Step 19: Generative UI: An Intent-Driven Sales Dashboard](../step-19-generative-ui-dashboard/) · [Roadmap](../README.md#roadmap) · [Step 21: Capstone: A Complete MCP App for a Real Client](../step-21-capstone-mcp-app/) →
