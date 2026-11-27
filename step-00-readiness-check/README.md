# Step 00: Are You Ready? Readiness Check

Before you build anything, check that you have the seven skills every later step assumes: Git and GitHub, the terminal, reading TypeScript, HTTP and JSON, SQL basics, keeping secrets safe, and working with Claude in a Claude project. This is a self-check, not a module. Each skill has a short test. If you pass a test, skip its reading. If you don't, it sends you to the exact sections of [The AI Agent Factory](https://agentfactory.panaversity.org/) that teach it, and to an outside reference only where the book doesn't cover the topic. Study them, then take the test again. You finish by setting up your fork and recording your results, so even this step ends with something you did, not just something you read.

> 🏬 **Stockroom's ask:** Stockroom, the client you work for in this bootcamp (see [Your client: Stockroom](../README.md#your-client-stockroom)), gives every new engineer a day to set up and prove they can work in its repo before handing over the first ticket. This step is that day.

## How it works

You don't need to be fluent in any of these skills. Claude writes most of the code. You need to **read** what it writes, **check** it, and **notice** when something is wrong.

Do these four things in order. If you already have the skills, it takes about an hour.

1. **Set up.** Fork and clone the repo, then set up your Claude project ([Build it with Claude Code](#build-it-with-claude-code), instructions 1 and 2).
2. **Take the seven checks below**, one at a time. Each one says what it tests, gives a short test, and tells you how to know if you passed.
3. **Save your results.** Claude writes them into `NOTES.md`, and you commit, tag and push in GitHub Desktop (instructions 3 and 4).
4. **Close any gaps.** For each check you didn't pass, read the book sections listed under it, then take its test again.

How long the reading takes if you don't pass a check:

| Check | Book sections | Time |
| --- | --- | --- |
| [1. Git and GitHub](#1-git-and-github) | 15.1, 15.2 | 1 to 2 hours |
| [2. The terminal](#2-the-terminal) | 12.2, 12.7, 13.1, 13.2 | 1 to 2 hours |
| [3. Reading TypeScript](#3-reading-typescript) | 13.1, 13.3, 13.6, 14.2, 14.4, 14.5, 14.8, 14.9 | 6 to 10 hours |
| [4. HTTP, JSON and APIs](#4-http-json-and-apis) | 14.7 | 1 to 2 hours |
| [5. SQL basics](#5-sql-basics) | 19.2, 19.3 | 2 to 3 hours |
| [6. Keeping secrets safe](#6-keeping-secrets-safe) | 14.7, 17.8, field guide F0, F1, F6 | 1 to 2 hours |
| [7. Working with Claude in a Claude project](#7-working-with-claude-in-a-claude-project) | 12.1, 12.3, 12.6, 12.8, 16.1 | 1 to 2 hours |

Didn't pass most of them? Work through chapters 12 to 15 of [The AI Agent Factory](https://agentfactory.panaversity.org/) first. Plan on two to three weeks of part-time study. Every later step will go faster for it.

## The seven checks

### 1. Git and GitHub

**You're ready if you can** use [GitHub Desktop](https://desktop.github.com/download/) to fork a repo, clone your fork, read a diff before you accept it, commit a change, add a tag and push them. This bootcamp uses GitHub Desktop for your own Git work, beside Claude. Claude never commits or pushes here: that is your job, so a person reads every change before it reaches GitHub (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)).

**Test.** Fork this repo and clone your fork in GitHub Desktop (build instruction 1). After Claude writes your `NOTES.md` (build instruction 3), do the rest yourself in GitHub Desktop (build instruction 4):

1. In the **Changes** tab, read the diff of `NOTES.md` and say in one sentence what changed.
2. Commit it to `main` with the summary `step-00: readiness check`.
3. In the **History** tab, right-click that commit, choose **Create Tag…**, and name it `step-00-done`.
4. Click **Push origin**, which sends the commit and the tag to your fork.

You pass if you can find both on your fork's GitHub page: the commit in the commit history, and the tag under **Tags**.

**If not, study these book sections, then take the test again:**

- [15.1 Git through the agent, "What does git keep?"](https://agentfactory.panaversity.org/building-ai-workers/code-review/git-through-the-agent/#what-does-git-keep). Commits, branches and tags (a tag is a fixed name for one commit).
- [15.1, "How do I read a diff?"](https://agentfactory.panaversity.org/building-ai-workers/code-review/git-through-the-agent/#how-do-i-read-a-diff). Reading a change before you accept it.
- [15.1, "Which commands need a pause?"](https://agentfactory.panaversity.org/building-ai-workers/code-review/git-through-the-agent/#which-commands-need-a-pause). Which git commands to approve with care, such as push.
- [15.2 GitHub and the pull request, "Where does a team keep the good copy?"](https://agentfactory.panaversity.org/building-ai-workers/code-review/github-and-the-pull-request/#where-does-a-team-keep-the-good-copy). Remotes, pushing, and what GitHub adds on top of git. (In this bootcamp you never open a pull request to the bootcamp repo.)

**Outside the book** (it works with git through the agent, not GitHub Desktop): [GitHub Desktop: Cloning and forking repositories](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-and-forking-repositories-from-github-desktop), [Committing and reviewing changes](https://docs.github.com/en/desktop/making-changes-in-a-branch/committing-and-reviewing-changes-to-your-project-in-github-desktop) and [Managing tags](https://docs.github.com/en/desktop/managing-commits/managing-tags-in-github-desktop).

### 2. The terminal

**You're ready if you can** open a terminal, move into a folder, run a command, and read its output and errors. You know what an environment variable is.

**Test.** Open a terminal (Terminal on macOS, PowerShell on Windows) and move into your fork's folder with `cd`. (In GitHub Desktop, **Repository → Open in Terminal**, or **Open in Command Prompt** on Windows, opens one already in that folder, but try `cd` yourself first.) Then:

1. Run `node --version`, `npm --version`, `git --version` and `npx wrangler --version`. Each should print a version number (Claude uses `git` to read your changes, so install Git even though you use GitHub Desktop), and Node.js must be **version 24 or later**. The first time, `npx` asks to download Wrangler (`Ok to proceed? (y)`): that is normal, so answer `y`. If a command fails, read the error and say what is missing.
2. Set an environment variable and read it back: `export STOCKROOM_ENV=dev` then `echo $STOCKROOM_ENV` on macOS or Linux, or `$env:STOCKROOM_ENV = "dev"` then `echo $env:STOCKROOM_ENV` in PowerShell. You should see `dev`. Then open a new terminal window and run the `echo` again. Say why it now prints nothing.

**If not, study these book sections, then take the test again:**

- [13.2 The workbench, "What will the agent ask me to approve?"](https://agentfactory.panaversity.org/building-ai-workers/programming-fundamentals/the-workbench/#what-will-the-agent-ask-me-to-approve). What `node`, `npm` and `npx` do, and the commands the agent runs for you.
- [12.2 The workspace](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/the-workspace/). Which folder Claude Code works in, and where its commands run.
- [12.7 The same agent outside the app, "Where else can it run?"](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/outside-the-app/#where-else-can-it-run). The terminal as one of the places an agent runs.
- [13.1 You read, the agent writes, "How do I set up to try this?"](https://agentfactory.panaversity.org/building-ai-workers/programming-fundamentals/you-read-the-agent-writes/#how-do-i-set-up-to-try-this). Installing Git and Node.js, and asking the agent to check what you already have.
- [14.7 JSON and REST APIs, "Where does the key go?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/#where-does-the-key-go). What an environment variable is.
- [Field guide F0, Keys on your computer](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f0-keys-on-your-computer). Read from the paragraph that starts "Ask the agent for the full path" to the one that starts "Then start the AI app": opening the Terminal app, moving into a folder with `cd`, setting an environment variable, and why a new terminal window doesn't have it. The rest of F0 is about the book's lab.

**Outside the book** (it doesn't show installing Wrangler on its own): [Install and update Wrangler](https://developers.cloudflare.com/workers/wrangler/install-and-update/). In every step from Step 01 on, Wrangler is installed with the step's other packages, so this is only for the version check.

### 3. Reading TypeScript

**You're ready if you can** read a short TypeScript function and say what it does, what types it takes and returns, and what happens when something goes wrong.

**Test.** Read this function, which looks a lot like the tools you will write, and answer the three questions below it without running it.

```ts
import { z } from "zod";

const Input = z.object({
  sku: z.string().min(1),
  quantity: z.number().int().positive(),
});

export async function checkStock(
  raw: unknown,
  apiUrl: string,
): Promise<{ ok: boolean; message: string }> {
  const parsed = Input.safeParse(raw);
  if (!parsed.success) {
    return { ok: false, message: "sku and a positive whole quantity are required" };
  }
  const res = await fetch(`${apiUrl}/stock/${encodeURIComponent(parsed.data.sku)}`);
  if (!res.ok) {
    return { ok: false, message: `Stock service answered ${res.status}` };
  }
  const { onHand } = (await res.json()) as { onHand: number };
  return { ok: onHand >= parsed.data.quantity, message: `${onHand} on hand` };
}
```

1. What does `checkStock({ sku: "A-1", quantity: 2.5 }, url)` return, and why?
2. Why is there an `await` before `fetch`?
3. What does it return if the stock service answers 404?

<details>
<summary>Check your answers</summary>

1. `{ ok: false, message: "sku and a positive whole quantity are required" }`, because the schema requires a whole number (`int()`), so validation fails before any request is sent.
2. `fetch` returns a promise. `await` waits for the response before the next line uses it.
3. `{ ok: false, message: "Stock service answered 404" }`, because `res.ok` is false for any status outside 200 to 299.

</details>

**If not, study these book sections, then take the test again:**

Chapters 13 and 14 teach TypeScript and Python side by side. Follow the TypeScript parts; Python is optional for this bootcamp.

- [13.1 You read, the agent writes, "Why predict before I run?"](https://agentfactory.panaversity.org/building-ai-workers/programming-fundamentals/you-read-the-agent-writes/#why-predict-before-i-run). The predict-then-check habit this test uses.
- [13.3 Values, types and variables](https://agentfactory.panaversity.org/building-ai-workers/programming-fundamentals/values-types-variables/). Strings, numbers and why a value's type matters.
- [13.6 Functions, "What does a signature promise?"](https://agentfactory.panaversity.org/building-ai-workers/programming-fundamentals/functions/#what-does-a-signature-promise). Reading a function's inputs and return type.
- [14.2 Types you write down](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/types-you-write-down/). Object types and exact values, like `{ ok: boolean; message: string }`.
- [14.4 Data models and schemas, "What is a schema?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/data-models-and-schemas/#what-is-a-schema). Zod schemas, and what `safeParse` returns (`success`, true or false). Question 1 of the test.
- [14.5 What a schema holds, "What rules can a schema hold?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/schema-rules/#what-rules-can-a-schema-hold). Rules such as whole numbers and minimums, like `.int().positive()`.
- [14.8 Async and await, "What does `await` do?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/async-and-await/#what-does-await-do). Question 2 of the test.
- [14.9 Reading SDK-shaped code](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/reading-sdk-code/). Reading code built on a library, like the MCP SDK.

### 4. HTTP, JSON and APIs

**You're ready if you can** name the parts of an HTTP request and response (method, URL, headers, status code, body), read a JSON body, and explain what an `Authorization: Bearer` header is for.

**Test.** Run `curl -i https://api.github.com/repos/panaversity/learn-mcp` in a terminal (or ask Claude to run it). On Windows, type `curl.exe` instead of `curl`, because in Windows PowerShell `curl` runs a different command. Point out the status code, the `content-type` header, and the value of `"default_branch"` in the JSON body. Then say what status code you would expect if the repo didn't exist.

<details>
<summary>Check your answers</summary>

The status line reads `HTTP/2 200` (or `HTTP/1.1 200 OK`): 200 means it worked. `content-type` is `application/json; charset=utf-8`, so the body is JSON. `"default_branch"` is `"main"`. A repo that doesn't exist gives `404 Not Found`. (If you see `403` with a message about a rate limit, GitHub has limited requests from your network for the hour. Try again later or from another network.)

</details>

**If not, study these book sections, then take the test again:**

- [14.7 JSON and REST APIs, "What are the parts of a request?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/#what-are-the-parts-of-a-request). Method, URL, headers and body.
- [14.7, "What is JSON?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/#what-is-json). Reading a JSON body.
- [14.7, "What should the client do for each status code?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/#what-should-the-client-do-for-each-status-code). 200, 404, 401 and the rest.
- [14.7, "Where does the key go?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/#where-does-the-key-go). The `Authorization: Bearer` header.

### 5. SQL basics

**You're ready if you can** read a `SELECT` with `WHERE`, `JOIN` and `GROUP BY`, and explain what a primary key and a foreign key are. You don't need to design a schema yet: step 5 teaches that.

**Test.** Given these two tables, predict the result of the query.

```text
customers                 orders
id | name                 id | customer_id | total
---+---------------       ---+-------------+------
 1 | Karachi Foods        10 |           1 |   300
 2 | Lahore Traders       11 |           1 |   200
                          12 |           2 |    50
```

```sql
SELECT c.name, SUM(o.total) AS sales
FROM customers c
JOIN orders o ON o.customer_id = c.id
WHERE o.total > 100
GROUP BY c.name;
```

<details>
<summary>Check your answer</summary>

One row: `Karachi Foods | 500`. `WHERE` removes order 12 (total 50) before the rows are grouped, so Lahore Traders has no orders left and doesn't appear. Karachi Foods' two orders, 300 and 200, add up to 500. `orders.customer_id` is a foreign key pointing at the primary key `customers.id`.

</details>

**If not, study these book sections, then take the test again:**

- [19.2 What are tables, rows and keys?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/tables-rows-and-keys-from-zero/). Tables, primary keys and foreign keys.
- [19.3 How do you read SQL an agent wrote? "Block 2. A join gives one row for every match"](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/reading-sql/#block-2-a-join-gives-one-row-for-every-match). `JOIN`, as in the test.
- [19.3, "Block 3. Count rows, or count values"](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/reading-sql/#block-3-count-rows-or-count-values). `GROUP BY` and `sum`.

### 6. Keeping secrets safe

**You're ready if you can** say where an API key goes on your computer, where it goes once deployed, how git is told to ignore it, and what to do if you commit one by mistake.

**Test.** Open this repo's `.gitignore` and find the lines that keep `.dev.vars` out of git. Then answer: a Worker reads `NEON_URL` locally from which file, and in production from which command? You pushed a real key to your public fork an hour ago. What do you do first?

<details>
<summary>Check your answers</summary>

Locally from `.dev.vars` (ignored by git; commit `.dev.vars.example` with placeholders instead). In production from a secret set with `wrangler secret put NEON_URL`. If you pushed a real key, **rotate it first**: make a new key and revoke the old one. Deleting the commit isn't enough, because your fork is public and the key may already have been copied.

</details>

**If not, study these book sections, then take the test again:**

- [14.7 JSON and REST APIs, "Where does the key go?"](https://agentfactory.panaversity.org/building-ai-workers/structuring-programs/json-and-rest-apis/#where-does-the-key-go). Keys belong in environment variables, never in code or a commit.
- [Field guide F0, Keys on your computer](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#f0-keys-on-your-computer). Keys in files git ignores, out of the chat and out of the agent's sight.
- [Field guide F1, "The words you will meet"](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#the-words-you-will-meet). What `.dev.vars` is.
- [17.8 How do you put a connector online safely?, "Where do the secrets live?"](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/securing-a-remote-server/#where-do-the-secrets-live). Secrets once the server is deployed.
- [Field guide F6, "Secrets"](https://agentfactory.panaversity.org/building-ai-workers/connectors-and-mcp/field-guide-connectors/#secrets). Setting a deployed Worker's secrets with Wrangler.

**Outside the book** (it covers making new keys after a leak, in F0 and chapter 18, but not cleaning a key out of a public repo's history): [GitHub: Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository).

### 7. Working with Claude in a Claude project

**You're ready if you can** set up a Claude project with your local folder and your GitHub fork, start a thread in it, read a plan before approving it, approve or refuse a permission request, keep Claude from committing, and use a skill.

**Test.** Build instruction 2 below. You pass if:

- your project has your local `learn-mcp` folder, your GitHub fork, and instructions that tell Claude never to commit or push;
- in a new thread, Claude proposes a plan before acting, and you approve or refuse each command it asks to run;
- you start the `build-mcp-server` skill yourself (type `/` in the prompt box and pick it, or ask Claude to use it) and ask which deployment model fits a server that Claude and ChatGPT reach over the internet. It should answer (a remote server over HTTP) without editing any files;
- you ask Claude which permissions, folders and accounts it can use in this thread, and its answer matches your settings (an instruction is not a permission, so you check what it can really reach; see [How we work](../README.md#how-we-work-local-first-human-in-the-loop), item 6);
- at the end, GitHub Desktop shows no commit that you didn't make yourself.

If projects aren't available on your account yet, take the same test in the **Code** tab of Claude Desktop, with your `learn-mcp` folder open and the same "never commit" rule in your first message.

**If not, study these book sections, then take the test again:**

- [12.1 From a chat to a coding agent](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/from-chat-to-coding-agent/). What changes when the AI can run commands and edit files.
- [12.3 Permissions, and plan before you act, "How do I read a plan?"](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/permissions-and-plan/#how-do-i-read-a-plan). Plan mode, and what to check before you approve.
- [12.6 Many sessions at once, "Which kind of helper do I use?"](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/many-sessions/#which-kind-of-helper-do-i-use). What a thread is, and how it differs from a subagent or a side chat.
- [12.8 How today's tools realize it, "The details, tool by tool"](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/both-ai-vendors/#the-details-tool-by-tool). Claude projects and their threads, and what a thread loads (`CLAUDE.md`, but not the repo's skills). Read the Claude Code part.
- [12.8, "Practice: the workspace"](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/both-ai-vendors/#practice-the-workspace). Opening a folder in the Code tab of Claude Desktop, the fallback (follow the Claude Code part).
- [12.8, "Practice: permissions, and a plan"](https://agentfactory.panaversity.org/building-ai-workers/agentic-coding/both-ai-vendors/#practice-permissions-and-a-plan). Plan mode and permission requests.
- [16.1 From a prompt you repeat to a skill, "Can I start a skill myself?"](https://agentfactory.panaversity.org/building-ai-workers/skills-and-plugins/from-a-prompt-to-a-skill/#can-i-start-a-skill-myself). What a skill is, and the two ways it starts: the agent picks it, or you type its name.

**Outside the book** (it doesn't show the redesigned projects or the Claude app's skill settings): Anthropic's [Projects redesigned](https://claude.com/resources/articles/projects-redesigned), for what a project, its coordinator and its threads are, and [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude), for turning skills on and uploading your own under **Customize → Skills**.

## Why this step matters

Every later step assumes these seven skills and moves fast. A learner who can't read a diff, or who pastes a key into a public fork, gets stuck in step 1 or 4 for reasons that have nothing to do with MCP. An hour here saves days later, and it tells you exactly which book lessons to read instead of all of them.

## What you will build

Your fork of the bootcamp, cloned locally and added, with your GitHub fork, to a Claude project that tells Claude never to commit, and a `NOTES.md` in `step-00-readiness-check/` that records your tool versions and, for each of the seven checks, whether you passed it or what you read. Committed, tagged `step-00-done` and pushed to your fork with GitHub Desktop.

## Prerequisites

- The tools and accounts listed in the [main README](../README.md#prerequisites), including [GitHub Desktop](https://desktop.github.com/download/), signed in to your GitHub account. You'll create the Cloudflare and Neon accounts in steps 1 and 5 if you don't have them yet.

## Build it with Claude Code

**1. Fork and clone the repo with GitHub Desktop.** Follow [Set up: fork, clone and sync](../README.md#set-up-fork-clone-and-sync) in the main README. That is the first part of check 1 (Git and GitHub).

**2. Set up your Claude project and start a thread.** Follow [How we work](../README.md#how-we-work-local-first-human-in-the-loop), steps 2 and 3: create a project, add your local `learn-mcp` folder and your GitHub fork, write the project instructions (including "never commit or push"), and add the bootcamp's skills to your Claude account: install Anthropic's `mcp-server-dev` plugin under **Customize → Plugins**, or upload the skills under **Customize → Skills** (step-by-step in [Built-in skills](../README.md#built-in-skills)). Then start a thread and paste this prompt. That is check 7 in practice.

```text
I'm starting the CCDV-F Bootcamp. Before we change anything, check my setup: run node --version, npm --version, git --version and npx wrangler --version, and tell me if any of them is missing, or if Node.js is older than version 24. Then tell me which of the bootcamp's skills (build-mcp-server, build-mcp-app, build-mcpb) you can use here and what each one is for, in one sentence each. Don't edit any files yet, and never commit or push.
```

> 💡 **Tip: keep your computer awake.** When Claude works in your local folder, it works on your computer, so a long run or a scheduled task stops if the computer goes to sleep. In Claude Desktop, open **Settings → Desktop app** and turn on **Keep computer awake**. Two limits: closing a laptop's lid still puts it to sleep, and cloud sessions don't need this setting. For a job that must keep running with the lid closed, use a cloud session on your GitHub fork instead ([How we work](../README.md#how-we-work-local-first-human-in-the-loop), step 5).

**3. Record your results.**

```text
Create step-00-readiness-check/NOTES.md (leave README.md alone, because it belongs to the bootcamp). Add my tool versions from the last check, then a table with one row for each of the seven checks in step-00-readiness-check/README.md, with columns "Check", "Passed?" and "What I read". I'll tell you my answers for each row. Don't commit or push: I'll do that myself in GitHub Desktop.
```

**4. Commit, tag and push in GitHub Desktop.** Do this yourself, not through Claude: it is the rest of check 1. In the **Changes** tab, read the diff of `NOTES.md`, then commit to `main` with the summary `step-00: readiness check`. In the **History** tab, right-click the commit and choose **Create Tag…** to add `step-00-done`. Then click **Push origin**.

**5. Close the gaps.** For any check you didn't pass, read the linked lessons, try its test again, update the row in `NOTES.md`, and commit and push the change in GitHub Desktop. Then go to Step 01.

## Submission checklist

Step 00 isn't graded and has no form entry. Before you start Step 01, make sure that:

- [ ] Your fork exists, is cloned with GitHub Desktop, and is added, with your GitHub fork, to a Claude project whose instructions say never to commit or push
- [ ] `node` (version 24 or later), `npm`, `git` and `npx wrangler` all print a version
- [ ] You passed all seven checks, or read the lessons for the ones you didn't and passed them on a second try
- [ ] `step-00-readiness-check/NOTES.md` is committed in GitHub Desktop and pushed to your fork, with the tag `step-00-done`

## References

- [The AI Agent Factory, chapters 12 to 20](https://agentfactory.panaversity.org/)
- [SUBMITTING.md](../SUBMITTING.md), for how you'll submit every step from Step 01 on

---

[Roadmap](../README.md#roadmap) · [Step 01: Hello World: A Minimal Remote MCP Server on Cloudflare](../step-01-hello-world/) →
