# Submitting Your Work

This bootcamp is also an assignment. You do all your work in **your own fork** of this repository, and you submit each finished step through the submission form (it [opens later](#8-submit-the-step); build and tag your steps in the meantime).

> ⛔ **Do not open pull requests to this repository.** Pull requests to `panaversity/learn-mcp` are not how you submit, and they will be closed without review. Your work stays in your fork.

## 1. Fork the repository

1. Sign in to GitHub and open https://github.com/panaversity/learn-mcp.
2. Click **Fork**, then choose your own account as the owner.
3. Keep the repository name `learn-mcp`, and leave **Copy the `main` branch only** checked.

Your fork is public, like the original. Never commit secrets (see [Keep secrets out of git](#7-keep-secrets-out-of-git)).

## 2. Clone your fork

Clone **your fork**, not the original. This bootcamp uses [GitHub Desktop](https://desktop.github.com/download/) for your own Git work, beside Claude, and Claude never commits or pushes for you.

In [GitHub Desktop](https://desktop.github.com/download/), signed in to your GitHub account, choose **File → Clone Repository**, pick `<your-username>/learn-mcp` on the **GitHub.com** tab, choose where to save it, and click **Clone**. When GitHub Desktop asks how you plan to use this fork, choose **For my own purposes**, because you never send work back to the bootcamp repo.

<details>
<summary>Prefer the command line?</summary>

```bash
git clone https://github.com/<your-username>/learn-mcp.git
cd learn-mcp
git remote add upstream https://github.com/panaversity/learn-mcp.git
git remote -v   # origin = your fork, upstream = the bootcamp repo
```

</details>

Then set up your Claude project: add your cloned `learn-mcp` folder (the repo root) and your GitHub fork to it, and tell it never to commit or push. [How we work](README.md#how-we-work-local-first-human-in-the-loop) in the main README has the steps.

## 3. Keep your fork in sync

New steps and fixes are added to the bootcamp repo over time. Pull them into your fork before starting each step.

1. **On GitHub:** open your fork and click **Sync fork → Update branch**.
2. **In GitHub Desktop:** click **Fetch origin**, then **Pull origin**, to bring the update to your computer.

<details>
<summary>Prefer the command line?</summary>

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

</details>

To avoid merge conflicts when you sync:

- **Don't edit the bootcamp's own files**: `README.md`, `AGENTS.md`, `SUBMITTING.md`, `.claude/skills/`, and the `README.md` inside each step folder.
- Put your own notes in a `NOTES.md` inside the step folder instead.

## 4. Do the work in each step folder

- Start with the readiness check in [`step-00-readiness-check/`](step-00-readiness-check/). It isn't graded, but it tells you which book lessons to read before you build.
- Then work through the steps in order, starting with [`step-01-hello-world/`](step-01-hello-world/).
- Follow the **Build it with Claude Code** section in each step's README, working in a thread of your bootcamp's Claude project.
- All your code for a step goes inside that step's folder. Each step starts from a copy of your previous step's code.
- Commit often, with clear messages. In GitHub Desktop, read the diff in the **Changes** tab, write a summary such as `step-01: add greet and get_time tools`, click **Commit to main**, then **Push origin**. Claude doesn't commit for you (your project's instructions tell it not to), so every commit is one you have read.

## 5. Deploy to your own accounts

Every student deploys to **their own** accounts:

- **Cloudflare:** your own Workers deployment (`*.workers.dev` or your own domain). Each step is its own Worker named `learn-mcp-step-NN`, so deploying step 2 doesn't replace step 1
- **Neon:** your own project, from step 5, with one branch per step (`step-05`, `step-06` …). Each step's first prompt creates its branch, so a later step's migrations never break an earlier step that is still being graded. Once a step's grade is in you may delete its branch, which you'll need to do on Neon's Free plan (see [Costs and cleanup](docs/costs-and-cleanup.md))
- **Claude and ChatGPT:** connect your deployed server as a custom connector in claude.ai or Claude Desktop, and in ChatGPT as a custom MCP server

Keep your server running until your step has been graded.

## 6. Tag each finished step

When a step is finished, deployed and tested, tag the commit and push the tag in GitHub Desktop:

1. Open the **History** tab, right-click the step's last commit, and choose **Create Tag…**.
2. Name it `step-01-done` and click **Create Tag**.
3. Click **Push origin**. It sends the tag to your fork with any unpushed commits.

Check the tag appears under **Tags** on your fork's GitHub page.

Use the tag names `step-01-done`, `step-02-done` … `step-21-done`. The grader reviews the code at your tag, so make sure everything for the step is committed before you tag.

Tags are final: never move or delete a tag you have submitted, because the grader may already be reviewing it. If you need to fix something after tagging, commit the fix, add a new tag with a version suffix (for example `step-01-done-v2`) the same way, push it, and submit that tag in the form.

## 7. Keep secrets out of git

- Store secrets with `wrangler secret put`, and keep local values in `.dev.vars`. The repo's `.gitignore` already keeps `.dev.vars`, `node_modules/` and `.wrangler/` out of git, so don't remove those lines.
- Never commit `.dev.vars`, API keys, database connection strings or tokens. Commit a `.dev.vars.example` with placeholder values instead.
- If you commit a secret by accident, **rotate it right away** (create a new key or password). Deleting the commit isn't enough, because your fork is public.

## 8. Submit the step

Submit each step through the form:

**📝 Submission form: graded submissions are not open yet. They will open later, and the link to the form will be added here.**

> You don't have to wait for the form to start building. Keep each finished step tagged and pushed, keep its server running, and submit every finished step once the form opens. When a step's checklist says "Submitted in the form", that is the one item to leave until then.

For each step, the form asks for:

| Field | Example |
| --- | --- |
| Your fork's URL | `https://github.com/<your-username>/learn-mcp` |
| Step tag | `step-01-done` |
| Live MCP server URL | `https://learn-mcp-step-01.<your-subdomain>.workers.dev/mcp` |
| Screenshot: Claude | The step's tool or widget working in claude.ai or Claude Desktop |
| Screenshot: ChatGPT | The same thing working in ChatGPT |
| Defence video (steps 4, 8, 10, 15, 18 and 21 only) | A link to your recorded project defence (see [Defend your project](#10-defend-your-project)) |

Each step's README ends with a **Submission checklist** listing exactly what that step needs.

## 9. How each step is graded

Each step is graded out of 10. A step passes at 7, and the grader reviews the code at your `step-NN-done` tag and the live server URL you submit.

| Criterion | Points | What the grader checks |
| --- | --- | --- |
| Works in Claude | 2 | The step's tools (and widgets, from step 9) work in claude.ai or Claude Desktop, as your screenshot shows |
| Works in ChatGPT | 2 | The same in ChatGPT as a custom MCP server. For an extension a client doesn't support yet, your NOTES.md records the support status and the fallback works |
| Builds what the step asks | 2 | Everything in the step's **What you will build** and **Submission checklist** is there, inside the remote MCP server |
| Design and code quality | 1 | Clear tool names, descriptions and schemas, correct annotations, recoverable errors, no state kept in the Worker's memory, readable code |
| Security and tenant isolation | 2 | No secrets in git, inputs validated, every data access scoped to the signed-in user and their company (from step 4), and from step 7 the cross-tenant suite passes and the step's tenant-isolation checklist item is met |
| NOTES.md | 1 | A short explanation, in your own words, of what you built, the choices you made and how to run it |

**Project defence (steps 4, 8, 10, 15, 18 and 21):** these steps end a milestone, and each needs a recorded defence (section 10) as well as the points above. The defence is pass or fail. A step that needs one doesn't pass without it, however many points it scores, and it doesn't pass if the defence shows you can't explain your own code. A failed defence can be re-recorded once.

**Automatic fail for the step:** any way for one company to reach another company's data (or, from step 20, to change another company's plan or usage), a committed secret that wasn't rotated, a server that can't be reached at the submitted URL, or code copied from another student.

**Capstone (step 21)** is graded out of 20: the 10 points above, plus up to 10 more: every Core row of the requirements table (3), extras that are justified and kept simple, with no technology added without a client need (2), the evaluation results (2), CLIENT.md with the client's acceptance tests (1), the architecture and handover notes (1) and the demo video (1). It passes at **14 of 20**, with at least 7 of the first 10. The automatic fails above apply to the capstone too, whatever its total: a cross-tenant leak, an unrotated committed secret, an unreachable server, copied code, or (if you built billing) a way for one company to get another's plan or be charged for another's usage. The capstone also needs its project defence, which is separate from the three-minute demo video: the demo shows the product to a client, and the defence explains the code to the grader.

## 10. Defend your project

Claude Code writes much of your code, so the bootcamp checks that you understand what it wrote. At the end of each milestone (steps 4, 8, 10, 15, 18 and the step 21 capstone) you record a short **project defence**: a screen recording, with your voice, of you walking through your own code. It is the same conversation a client or an interviewer will have with you.

**Length:** up to five minutes (up to ten for the capstone). **Where:** any link the grader can open without signing in, such as an unlisted YouTube video or a shared Loom or Google Drive file. Link it in the form with the step.

Cover these four things, in your own words:

1. **It works.** Show the step running in Claude or ChatGPT, in under a minute.
2. **One tool, end to end.** Open the code at your tag and follow one tool call from its input schema through auth and tenant context to the data it reads or writes, and back to what the model sees.
3. **One tenant-isolation decision.** Pick one decision that keeps one company's data away from another's, and explain where the company comes from (never from tool input), what stops another company's data from coming back, and which test proves it. In step 4, this is how a user's company is chosen and how it scopes their notes' keys.
4. **One change you made.** Something Claude proposed that you changed, refused or fixed, and why.

Talk, don't read: a script written by an AI and read aloud doesn't count as a defence. The grader may also ask any student for a short live defence on a call, at random or when the code and the recording don't match.

## Getting help

- Re-read the step's README and ask Claude (in a thread of your Claude project) to explain whatever is unclear.
- Ask your instructor or classmates through the bootcamp's usual help channel. Don't use pull requests for questions.
