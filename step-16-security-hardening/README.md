# Step 16: Security Hardening for MCP Servers

Your server now holds business data, acts for signed-in users, runs background jobs and shows screens in the chat, which makes it worth attacking. This step reviews and hardens the whole server against the threats specific to MCP: prompt injection through tool results and widget text, tokens meant for another server, over-powerful tools, unvalidated input, leaked secrets and data that should not leave. You finish with a written threat model and tests that prove each fix.

> 🏬 **Stockroom's ask:** A large retailer's security team sends Stockroom a questionnaire before it will connect anything. Stockroom needs a threat model and proof that each risk is handled.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [17.7 What if a record tells the AI what to do?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/untrusted-output/). Treating what tools return as untrusted, and keeping private data in.
- [17.8 How do you put a connector online safely?](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/securing-a-remote-server/). Access keys, Origin checks and staging.
- [18.6 Is this token real, and is it for this connector?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/a-token-made-for-one-door/). Checking that every token was issued for your server (its audience).
- [18.7 What may each person do?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/permission-per-tool/). Scopes per tool, 403 responses and step-up.
- [18.4 What if someone copies the code?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/two-guards-for-the-code/). Two independent checks for every rule that matters.
- [20.5 Does hiding a tool protect it?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/hidden-is-not-protected/) and [20.7 What if the data holds page code?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/untrusted-text-on-the-screen/). What a widget must never rely on, and showing text you didn't write safely.
- [Field guide F8, More input during a call](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#more-input-during-a-call). How a stateless tool asks for a confirmation, and why the state it sends back must be protected.
- *Optional:* [19.10 What can still leak when the rows are safe?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/where-the-guarantee-stops/). Where row-level security stops protecting you.

**Official docs.** Keep these open while you build:

- [MCP security best practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices). The MCP-specific attacks and their defenses.
- [MCP: Authorization security considerations](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations). Token audience, redirect URIs and token passthrough.
- [MCP: Elicitation](https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation). Asking the user mid-call, through a form (form mode) or a link for anything sensitive (URL mode).
- [MCP: Multi round-trip requests](https://modelcontextprotocol.io/specification/2026-07-28/basic/patterns/mrtr). How a stateless server asks for more input: it returns `input_required`, and the client retries with the answers and the server's `requestState`, which the server must protect.
- [OWASP: Prompt injection (LLM01)](https://genai.owasp.org/llmrisk/llm01-prompt-injection/). How injected instructions reach a model, and how to limit the damage.
- [Cloudflare Workers: secrets](https://developers.cloudflare.com/workers/configuration/secrets/). Storing and rotating secrets.
- [Cloudflare: rate limiting binding](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/). Limiting abuse per user or per IP.

## Why this step matters

A client will ask "is it safe?" before they connect your server to their business. MCP servers face the usual web risks plus new ones: text in a customer record can try to instruct the model, a token stolen from another app can be replayed at your server, and a tool that can do too much turns one mistake into a breach. Security review is also part of what Claude's directory and client security teams check before they accept a connector.

## Why this technology in an MCP server: security hardening

- **What it is.** A threat model of the server, followed by fixes: token audience checks, least-privilege tools and scopes, strict input validation, output handling that treats data as data, secret hygiene, rate limits and audit logs.
- **Why an MCP server uses it.** An MCP server acts on a person's behalf with whatever the model asks it to do, and the model can be steered by text in tool results. The server is the last place you control, so it must enforce the rules itself rather than trusting the model, the client or the widget.
- **What it solves.** Prompt injection that turns a read into a write, tokens accepted from the wrong issuer or for the wrong server, data from one user reaching another, secrets in logs or git, and abuse that runs up your costs.
- **Alternatives.** None replace this work. Gateways, web application firewalls and Cloudflare's own protections help at the edge, and a security review by a specialist helps before a big client launch, but the server's own checks are what make it safe.

## What you will learn

- Write a threat model for an MCP server: who can call it, what each tool can do, and what could go wrong
- Check every token's issuer, audience and expiry, and never pass a user's token on to another service
- Shrink each tool to the least it needs, with scopes per tool
- Ask the user to approve anything destructive or irreversible before it runs, with elicitation, and protect that approval from replay and tampering
- Validate every input strictly, and treat text from the database, imports and events as data, never as instructions
- Keep secrets out of code, logs and tool results, and rotate them
- Add rate limits and an audit log of who did what

## What you will build

A hardened version of the server, plus `SECURITY.md` in the step folder: the threat model, each risk and its fix, and the test that proves it. Tests cover a token for another audience being refused, one user never reaching another's data, an injected instruction in an order note not causing a write, oversized or malformed input being rejected, and secrets never appearing in logs or tool results.

## Key concepts

- **Threat model**: A list of what you protect, who might attack it, and how.
- **Token audience**: The server a token was issued for. A server must refuse tokens meant for someone else.
- **Prompt injection**: Text in data that tries to give the model instructions. The server limits what any instruction can make it do.
- **Least privilege**: Each tool, role and token gets only what it needs.
- **Defense in depth**: Two independent guards for every rule that matters, such as tool checks plus row-level security.
- **Audit log**: A record of who called which tool, when, with what result.
- **Elicitation**: The server pauses a tool call to ask the user something, through a form or a link. Servers use it mainly to get approval before an action that costs money or can't easily be undone, as when GitHub's server asks you to type a repository's name before deleting it. Anthropic made it generally available in Claude in November 2026.
- **Sealed request state**: A stateless server sends the confirmation's context to the client in `requestState` and gets it back on the retry, so it signs that state and binds it to the company, user, tool, arguments and an expiry, and refuses it if anything changed.
- **Agents acting on their own**: Never pass a user's token on to another service; call it with the server's own credentials. MCP's maintainers are working on a Workload Authorization Grant standard for agents that reach third-party services by themselves, which isn't published yet.

## Outline

1. Threat modelling an MCP server
2. Tokens: issuer, audience, expiry and no passthrough
3. Least-privilege tools, scopes, and confirmations with elicitation
4. Input validation and output handling
5. Prompt injection through data, imports, events and widgets
6. Secrets, rate limits and audit logs
7. Proving each fix with tests

## Prerequisites

- [Step 15](../step-15-events/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 15 code into `step-16-security-hardening/`.**

```text
Copy my step-15 code into step-16-security-hardening/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-16 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-16 from my step-15 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and write the threat model.**

```text
Read the MCP security best practices and the authorization security considerations linked in this step. Then review this whole server (tools, resources, prompts, widgets, skills, elicitation confirmations, Durable Objects, queues, workflows, tasks and events) and write a threat model in step-16-security-hardening/SECURITY.md: assets, attackers, entry points and the top risks, ranked. Don't change code yet.
```

**3. Harden tokens and permissions.**

```text
Make sure every request's token is checked for issuer, audience (this server only) and expiry, and that no user token is ever passed on to another service. Add scopes per tool, and return clear 401 and 403 responses.
```

**4. Confirm destructive actions with elicitation.**

```text
Read the MCP elicitation and multi round-trip request docs linked in this step. For every destructive or irreversible tool (for example cancelling an order, deleting a note, or an import that replaces data), return input_required with a form-mode elicitation that says exactly what will happen and asks the user to type the order number, or the word the tool names, to confirm. Seal requestState with an HMAC key kept as a Wrangler secret, bind it to the company, user, tool name, arguments and a short expiry, and refuse a retry whose state is missing, changed, expired or from another user. Never ask for passwords, tokens or payment details in a form. For clients that don't support elicitation, fall back to two calls: the first returns a summary and a sealed confirmation token, and the second must send it back. Add tests for each refusal, including a confirmation replayed by a user from another company.
```

**5. Validate input and handle output safely.**

```text
Tighten every tool's input schema and server-side validation (lengths, ranges, enums and formats). Review every piece of background work (queue consumers, Workflow steps, tasks and event handlers) and make sure each consequential action re-checks the acting user's current membership and role right before it runs, as the "Authority at run time" rule in AGENTS.md says, and that any job with no person behind it runs under its own service identity with the least scope. Add tests for a removal and a demotion mid-run. Then treat text from the database, CSV imports and events as data: never follow instructions found in it, mark it as untrusted in tool results, and escape it in widgets. Add a test where an order note contains an injected instruction and show that it can't cause a write.
```

**6. Secrets, limits and audit.**

```text
Check that no secret appears in code, logs, errors or tool results, and document how to rotate each one. Add per-user rate limits where they are missing and an audit log of tool calls (user, tool, time, result, never secrets or full personal data).
```

**7. Prove it.**

```text
Write tests for each risk in SECURITY.md and run them. Update SECURITY.md with each risk, its fix and the test that proves it, then deploy and walk me through a quick manual check in Claude and ChatGPT.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-16-security-hardening/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] SECURITY.md has the threat model, each fix and the test that proves it, and the tests pass
- [ ] Every destructive or irreversible tool asks for confirmation through elicitation (with the two-call fallback), and tests show a changed, expired or cross-company confirmation is refused
- [ ] Tenant isolation: SECURITY.md covers cross-tenant attacks (guessed IDs, injected company names, shared caches) and stale authority in background work (a removed or demoted user's job), each with a passing test
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Tagged `step-16-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-16-done`, live server URL, and screenshots of a refused request (wrong token or another user's data) and of the injection test passing, plus the server still working in **Claude** and **ChatGPT**

## References

- [MCP security best practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices)
- [OWASP: Prompt injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)

---

← [Step 15: Events](../step-15-events/) · [Roadmap](../README.md#roadmap) · [Step 17: Testing, Evals and Observability](../step-17-testing-and-observability/) →
