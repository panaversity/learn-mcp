# Step 04: Adding Authorization with Better Auth and D1

Protect the server so that only signed-in users can use it, and so that every tool call knows *who* is calling. We use OAuth with Better Auth, and store users and sessions in Cloudflare D1. We chose Better Auth partly because it supports single sign-on (SSO) through its open-source `@better-auth/sso` plugin, so a client's staff can later sign in with the company account they already have. Every user also belongs to a company (a *tenant*) with a role, because the servers you build for clients serve whole teams, and from here on every step keeps each company's data apart.

> 🏬 **Stockroom's ask:** Stockroom sells to companies, not individuals. Each customer company's staff must sign in and see only their own notes, admins need more rights than members, and Stockroom's bigger prospects already ask whether their staff can sign in with their company accounts (SSO).

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [18.1 Who is calling?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/who-is-calling/). Why a shared key is not enough.
- [18.2 How does signing in become a token?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/sign-in-in-one-picture/). The parties in OAuth and the flow that starts with the first 401.
- [18.3 Where does the app send you to sign in?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/finding-the-door/). How clients discover where to sign in (protected-resource discovery).
- [18.7 What may each person do?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/permission-per-tool/). Scopes per tool, 403 responses and step-up.
- [18.9 How does one connector serve many companies?](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/what-a-tenant-is/). A product serves many companies, and each company's people and data stay apart.
- [Field guide F3, Better Auth from zero](https://agentfactory.panaversity.org/building-ai-workers/securing-connectors/field-guide-securing-connectors/#f3-better-auth-from-zero). Read F2 for D1 as well. Its "Going further: sign in with your company (SSO)" section explains how SSO and the connector's sign-in work together.
- *Optional:* [20.13 How does a company decide which employees connect?](https://agentfactory.panaversity.org/building-ai-workers/mcp-apps/employees-through-their-company/). Enterprise-Managed Authorization: the company's identity provider decides which employees may use a connector, so nobody clicks a consent screen.

**Official docs.** Keep these open while you build:

- [MCP: Authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization). The OAuth rules every remote MCP server follows.
- [MCP: Authorization server discovery](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/authorization-server-discovery). The metadata clients look for after a 401.
- [MCP: Client registration](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration). How Claude and ChatGPT register with your auth server. Client ID Metadata Documents are now the preferred way; Dynamic Client Registration is deprecated in this spec version but still allowed for older clients.
- [MCP: Authorization extensions](https://modelcontextprotocol.io/extensions/auth/overview). What the official extensions add on top of the core spec: client credentials for machines, and Enterprise-Managed Authorization for companies.
- [MCP: Enterprise-Managed Authorization](https://modelcontextprotocol.io/extensions/auth/enterprise-managed-authorization). How a company's identity provider (such as Okta) authorizes your server once for all its staff.
- [Better Auth: MCP plugin](https://better-auth.com/docs/plugins/mcp). Better Auth's support for MCP servers.
- [Better Auth: OAuth provider plugin](https://better-auth.com/docs/plugins/oauth-provider). Turning Better Auth into the OAuth server clients sign in to.
- [Better Auth: SSO plugin](https://better-auth.com/docs/plugins/sso). Signing people in with their company's identity provider over OIDC or SAML.
- [Better Auth: organization plugin](https://better-auth.com/docs/plugins/organization). Companies, members and roles, so every user belongs to a tenant.
- [Cloudflare D1: Wrangler commands](https://developers.cloudflare.com/workers/wrangler/commands/d1/). Creating the auth database and running migrations.
- [Cloudflare Workers: secrets](https://developers.cloudflare.com/workers/configuration/secrets/). Storing auth secrets outside your code.

## Why this step matters

Real servers act on behalf of real users. Claude and ChatGPT both support OAuth when connecting to a remote MCP server. Getting auth right early means every later step (business data, UI, tasks) can safely be per-user.

## Why this technology in an MCP server: OAuth with Better Auth and Cloudflare D1

- **What it is.** OAuth is the sign-in standard that MCP's authorization spec is built on. Better Auth is a TypeScript library that can act as the OAuth server for your MCP server. D1 is Cloudflare's SQLite database, where Better Auth keeps users, sessions and registered clients. Its open-source `@better-auth/sso` plugin adds single sign-on with a company's own identity provider (such as Okta or Microsoft Entra ID) over OIDC or SAML.
- **Why an MCP server uses it.** When a user adds your server as a connector, Claude and ChatGPT run the OAuth sign-in for them and then send a token with every request. That token tells your tools **who** is calling, which is what lets the server keep each user's data apart and decide what each user may do.
- **What it solves.** A shared API key can't tell one person from another, can't be limited per user, and is hard to revoke (book lesson 18.1). OAuth gives each user their own revocable access, which clients and their security teams expect.
- **Why SSO matters for client work.** Enterprise clients usually require their staff to sign in with the company account their IT team manages, so access starts and ends with employment. Better Auth gives you that with an open-source plugin, inside your own Worker and database, instead of a paid identity service. SSO proves who the person is; your server still decides what they may do.
- **Alternatives.** Hosted identity providers such as Auth0, Clerk, WorkOS or Stytch, or Cloudflare's own OAuth provider library for Workers. We use Better Auth because it supports SSO through an open-source plugin, runs inside your Worker, stores its data in your own database and is what the book teaches.

## What you will learn

- Understand how MCP authorization works: the server as an OAuth resource server, discovery metadata, and tokens on every request
- Set up Better Auth on Cloudflare Workers, with Cloudflare D1 as its database
- Run the OAuth flow when a user adds the connector in Claude or ChatGPT
- Read the authenticated user inside a tool handler
- Apply per-user authorization rules to tools
- Put every user in a company (a tenant) with a role, using Better Auth's organization plugin, so later steps can keep each company's data apart
- Know what the MCP authorization extensions add on top of the core spec, including Enterprise-Managed Authorization, where an admin authorizes the server once for the whole company
- Optionally, let a client company sign in with its own identity provider through Better Auth's SSO plugin

## What you will build

The notes server from steps 2 and 3, now behind sign-in. Every user belongs to a company with a role (`member` or `admin`), and a first sign-in creates a personal company. Notes stay in Workers KV, with every key prefixed by the company and the owner's user ID (`org:<orgId>:user:<userId>:note:<noteId>`), so each user sees only their own notes. A new admin-only `list_company_members` tool shows a role check at work.

## Key concepts

- **OAuth 2.1 for MCP**: How hosts discover your auth server and obtain tokens.
- **Better Auth**: A TypeScript auth framework that provides the sign-in, session and OAuth provider pieces.
- **Cloudflare D1**: SQLite at the edge, used here only for auth data (users, sessions, clients).
- **Identity in tool calls**: Turning a validated token into a user ID, company ID and role your tools can trust.
- **Tenant**: A customer company. One server serves many tenants, and no tenant ever sees another's people or data.
- **Role**: What a member may do inside their company, such as `member` or `admin`, checked by the server on every call.
- **Single sign-on (SSO)**: Signing in with the account a person's company already manages, over OIDC or SAML. It proves who they are, and your server still decides what they may do.
- **Enterprise-Managed Authorization**: An official, stable MCP extension, built with Okta and others, for a world where agents run without someone clicking a consent button. The company's identity provider gives the AI app a pass (an ID-JAG) that your auth server trades for an ordinary token, so an admin decides once which employees may use the server.
- **Client registration**: How an AI app identifies itself to your auth server. The 2026-07-28 spec prefers Client ID Metadata Documents (the client's ID is a URL to its own description) over Dynamic Client Registration.

## Outline

1. How Claude and ChatGPT authenticate to remote MCP servers
2. Creating the D1 database and binding it to the Worker
3. Configuring Better Auth
4. Exposing the OAuth metadata MCP clients expect
5. Protecting the `/mcp` endpoint
6. Companies, roles and the user's identity in tools
7. Connecting and signing in from Claude and ChatGPT
8. Security checklist

## Prerequisites

- [Step 3](../step-03-resources-and-prompts/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)).

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 3 code into `step-04-auth/`.**

```text
Copy my step-03 code into step-04-auth/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-04 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds.
```

**2. Switch to plan mode and plan the auth.**

```text
Read the build-mcp-server skill's auth reference. Then plan how to add OAuth to this server using Better Auth, with Cloudflare D1 as Better Auth's database, so that Claude and ChatGPT can sign users in when they add the connector. Show me the plan and any choices I need to make.
```

**3. Create the D1 database.**

```text
Tell me the Wrangler commands to create a D1 database for auth, then add the binding to the wrangler config and generate Better Auth's schema migrations for D1.
```

**4. Configure Better Auth.**

```text
Set up Better Auth in the Worker with the D1 database, and enable what an MCP server needs to act as an OAuth-protected resource, including the discovery metadata clients look for. Add the organization plugin so every user belongs to a company with a role (member or admin), and create a personal company on a user's first sign-in. Explain each config option you set. For client registration, support Client ID Metadata Documents if Better Auth does, keep Dynamic Client Registration only as the fallback the spec still allows, and tell me which one Claude and ChatGPT use when they connect.
```

**5. Protect the MCP endpoint.**

```text
Require a valid token on /mcp, return the correct unauthorized response with discovery info when it's missing, and make the signed-in user's ID, the company they chose when they connected (read from the verified token, never from a tool argument) and their current role (looked up in the membership table on every request) available inside tool handlers.
```

**6. Make the notes per-user.**

```text
Change the notes tools so each user only sees and changes their own notes, by prefixing every Workers KV key with the company and user ID (org:<orgId>:user:<userId>:note:<noteId>) and listing only that prefix. Add an admin-only list_company_members tool that returns a clear 403-style error for members. Add quick tests that one user can't read another user's note and that a member can't call the admin tool.
```

**7. Deploy and sign in from each client.**

```text
Set the needed secrets with Wrangler, deploy, and give me a checklist for connecting and signing in from claude.ai, Claude Desktop and ChatGPT.
```

**8. Run a security review.**

```text
Review the auth setup for security issues: token validation, redirect URIs, secret handling and CORS. List anything to fix and fix it.
```

**9. Optional: add single sign-on for a client company.** You'll need a test identity provider, such as a free Okta developer account or Microsoft Entra ID.

```text
Read Better Auth's SSO plugin docs and the "sign in with your company (SSO)" section of the book's field guide F3. Add the @better-auth/sso plugin so a company can register its own OIDC (or SAML) identity provider and its staff sign in with their company accounts when they connect the server from Claude or ChatGPT. Keep our own rules for what each signed-in user may do, and explain how SSO and the connector's OAuth sign-in fit together. Then read the Enterprise-Managed Authorization docs linked in this step and write in NOTES.md how it differs from SSO, and what our auth server would need to accept an ID-JAG: check it against the identity provider's published keys, issuer, audience and expiry, map its subject (and email, for existing accounts) to a user and company, then issue our own token. Check whether Better Auth supports it yet, and build it only if it does.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (custom MCP server)

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-04-auth/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (custom MCP server)
- [ ] Project defence recorded (up to five minutes: it works, one tool end to end, one tenant-isolation decision, one change you made; see [SUBMITTING.md](../SUBMITTING.md#10-defend-your-project)) and linked in the form
- [ ] Tagged `step-04-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-04-done`, live server URL, and screenshots of signing in and using your own notes in **Claude** and **ChatGPT**

## References

- [MCP: Authorization extensions](https://modelcontextprotocol.io/extensions/auth/overview)
- [MCP: Enterprise-Managed Authorization](https://modelcontextprotocol.io/extensions/auth/enterprise-managed-authorization)
- [Better Auth](https://better-auth.com/docs)
- [Cloudflare D1](https://developers.cloudflare.com/d1/)

---

← [Step 03: Resources and Prompts: The Other Two MCP Primitives](../step-03-resources-and-prompts/) · [Roadmap](../README.md#roadmap) · [Step 05: Relational Databases and Normal Forms with Neon](../step-05-relational-databases/) →
