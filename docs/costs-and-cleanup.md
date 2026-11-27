# Costs and Cleanup: What This Bootcamp Uses, What Can Cost Money, and What to Delete

This guide helps you keep the bootcamp's spending predictable. It lists every service the 21 steps use, the plan you need, what could start to cost money, and what you may delete once a step is graded. Prices and free allowances change, so each row links the official page: check it before you upgrade anything.

For AI tools themselves (which Claude plan, cheaper coding agents, API costs), read [Affordable Agentic AI Development](affordable-agentic-ai.md).

## The rule: no paid resource without your yes

- Stay on free plans unless a step says otherwise. Every step in this bootcamp is designed to fit the free plans of Cloudflare and Neon, and Stripe runs in test mode only.
- If you ever upgrade (for example to Cloudflare's Workers Paid plan), set a budget or usage alert first, and write down why in that step's `NOTES.md`.
- Never let Claude create a paid resource, change a plan or enter payment details. Those choices are yours, in each product's dashboard (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop), item 6).

## What each service costs you

Checked on November 27, 2026.

| Service | Steps | Plan you need | What could cost money | Official page |
| --- | --- | --- | --- | --- |
| Claude | All | A paid plan (Pro or higher) for Claude Code and Claude projects | Your subscription; usage limits can slow you down, not charge you more | [Claude pricing](https://claude.com/pricing) |
| ChatGPT | All (testing) | A plan and, on a work account, a workspace that allows adding a custom MCP server | Your subscription, if your plan needs one | [Add a custom MCP server](https://developers.openai.com/api/docs/guides/custom-mcp-server) |
| Cloudflare Workers | 1 to 21 | Workers Free | Nothing on the free plan: a request over the daily limit fails instead of being billed | [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/) |
| Workers KV | 2 to 4, 7 on | Workers Free | Nothing on the free plan; heavy writes hit the daily limit | [KV pricing](https://developers.cloudflare.com/kv/platform/pricing/) |
| D1 (sign-in data) | 4 on | Workers Free | Nothing on the free plan at bootcamp size | [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/) |
| Neon Postgres | 5 on | Neon Free | Nothing on the free plan; a project over its limits pauses or refuses new branches. **Watch the branch limit (below)** | [Neon plans](https://neon.com/docs/introduction/plans) |
| Hyperdrive | 6 on | Workers Free (limited daily use) | Nothing on the free plan; over the daily limit, queries fail until 00:00 UTC | [Hyperdrive pricing](https://developers.cloudflare.com/hyperdrive/platform/pricing/) |
| Workers AI (embeddings) | 8 on | Workers Free (a daily free allowance of Neurons) | Nothing on the free plan; going past the daily allowance needs Workers Paid | [Workers AI pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/) |
| Durable Objects | 11 on | Workers Free (SQLite-backed objects only) | Nothing on the free plan at bootcamp size. Use the SQLite storage backend, the default for new classes | [Durable Objects pricing](https://developers.cloudflare.com/durable-objects/platform/pricing/) |
| Queues | 12 on | Workers Free | Nothing on the free plan at bootcamp size | [Queues pricing](https://developers.cloudflare.com/queues/platform/pricing/) |
| Workflows | 13 on | Workers Free | Nothing on the free plan at bootcamp size. Free-plan runs keep their state for 3 days | [Workflows pricing](https://developers.cloudflare.com/workflows/reference/pricing/) |
| Claude API (evals) | 17 on | An Anthropic API account with a small prepaid balance | A few cents to a few dollars per eval run, depending on the model and the size of your eval set | [Claude pricing](https://claude.com/pricing) |
| Stripe | 20, and 21 if you choose billing | A free Stripe account, **test mode only** | Nothing in test mode. Never switch to live keys in this bootcamp | [Stripe testing](https://docs.stripe.com/testing) |

The one real cost is your Claude subscription, plus a little for the Claude API from step 17. Everything else fits free plans if you follow the steps.

## The Neon branch limit

Each step from 5 to 20 uses its own Neon branch (`step-05` to `step-20`), so a later migration never breaks an earlier step that is still being graded. That is 16 branches, plus `main`, and **Neon's Free plan allows 10 branches per project** ([Neon plans](https://neon.com/docs/introduction/plans)). So you will need to delete old branches as you go:

1. Once a step's grade is in, you may delete its branch (and stop its Worker, below). Keep the branch of any step still waiting for a grade.
2. In practice, keep only the last few steps' branches. For example, when you create `step-12`, delete `step-05` to `step-08` if they are graded.
3. Never delete `main` or the branch your current step is using, and never delete a branch before its step is graded.
4. Delete a branch yourself in the Neon Console, on its **Branches** page, not by asking Claude: a deleted branch can't be restored.

If you'd rather not delete anything, start a second Neon project at step 13 and create the remaining branches there.

## Cleanup after grading

| Resource | When you may delete it | How |
| --- | --- | --- |
| A step's Worker (`learn-mcp-step-NN`) | After that step's grade is in. Keep at least your latest step and your capstone live, because they are your portfolio | In the Cloudflare dashboard, open the Worker and delete it from its settings |
| A step's Neon branch | After that step's grade is in | In the Neon Console, as above |
| Queues and dead-letter queues | After the steps that use them (12 on) are graded and you have deleted their Workers | In the Cloudflare dashboard, on the Queues page |
| Test D1 databases and KV namespaces | After the steps that use them are graded | In the Cloudflare dashboard, on the D1 and KV pages |
| Stripe test data | Any time | In the Stripe dashboard, in test mode |
| `.dev.vars` and test keys | Never commit them; rotate any key you shared by mistake | See Step 00, check 6 |

Tags in your fork stay forever, even after you delete a Worker or a branch, so your submitted code is never lost.

## Before your capstone

Your capstone is meant to be shown to clients, so keep it live after grading. Its Worker, its Neon project (or branch) and its D1 database are the only resources you need to keep long term.
