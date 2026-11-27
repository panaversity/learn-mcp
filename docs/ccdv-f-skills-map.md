# CCDV-F Skills Map: Where This Bootcamp Practises Each Exam Objective

This map lines up the bootcamp's 21 steps with the 24 objectives of Anthropic's **Claude Certified Developer – Foundations (CCDV-F) exam guide, version 1.1** (effective December 7, 2026). Panaversity's PCDV-F mirrors the same objectives. The objective numbers, names and domain weights come from the guide. The scope lines are our short summaries, so read the [official exam guide](https://anthropic-partners.skilljar.com/path/claude-certified-developer-foundations) for the exact wording.

**Read this honestly.** This bootcamp builds one kind of product, an MCP app, so it practises some objectives in depth and others not at all. It is strong supporting preparation for CCDV-F, **not** complete preparation. For the objectives marked *Not covered*, study the later chapters of Part III of [The AI Agent Factory](https://agentfactory.panaversity.org/), which cover agents, the Agent SDK, the Messages API and model choice.

**Coverage** means:

- **Practised**: you build it and a submission checklist item checks it.
- **Partly**: you use it in some steps, but not every element the guide names.
- **Not covered**: this bootcamp doesn't teach it.

**Evidence** is what a grader can see in your fork that shows you did it.

## Domain 1: Agents and Workflows (17%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 1.1 Architecture and Runtime Selection | Choosing a workflow or an agent, and where it runs | Partly | 13, 21 | NOTES.md on why a Workflow (fixed steps) fits, and the capstone's architecture choices | No agentic architectures, subagents or managed agent runtimes |
| 1.2 Agent SDK Construction and Hooks | Building agents and hooks with the Claude Agent SDK | Not covered | | | Study the book's Agent SDK chapter |
| 1.3 Agent Pattern Diagnosis | Finding the missing pattern in a misbehaving agent | Not covered | | | Study the book's agent-loop chapters |

## Domain 2: Applications and Integration (20%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 2.1 Messages API Mechanics | Messages, tool use, streaming, caching, structured output, stop reasons, batch | Partly | 17 | The eval script that calls the Claude API with your MCP server attached | Only one API call pattern; no streaming, vision, thinking, caching or batch |
| 2.2 Application Boundary and Surface Behavior | How each surface treats instructions, and what crosses the boundary | Partly | 9, 10, 18, 19 | Text fallback for every widget, `cacheScope: "private"` on caller-specific results, tested in Claude and ChatGPT | Covers MCP surfaces, not every Claude surface |
| 2.3 Configuration and SDLC Integration | CLAUDE.md, settings, pinning, CI, environments | Partly | 0, 17, 18 | CLAUDE.md and AGENTS.md in the repo, pinned dependencies, staging and production, CI | No model version pinning or prompt versioning |
| 2.4 Integration Resilience | Finding defects in failing integration code, retries and fallbacks | Partly | 2, 12, 13, 17 | Recoverable tool errors, retries and a dead-letter queue, retried Workflow steps | Not Anthropic API error semantics or streaming events |

## Domain 3: Claude Code (8%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 3.1 Claude Code Operation | Surfaces, sessions, CLAUDE.md, settings, permission modes | Practised | 0, every step | Step 00 check 7 (projects, plans, permissions), every step built in a Claude project thread | Headless and streaming modes aren't used |
| 3.2 Claude Code Customization and Extension | Choosing and authoring a skill, plugin, hook, subagent or command | Partly | 0, 10 | Using the bundled skills; serving skills from your server in step 10 | You don't author or package a Claude Code plugin or hook |

## Domain 4: Eval, Testing, and Debugging (10%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 4.1 Evaluation Design | Golden sets, regression tests, judges, metrics | Practised | 8, 17, 21 | The retrieval question set in CI, the tool-and-screen eval set, the capstone's 25-prompt evaluation | LLM-as-judge calibrated against human labels isn't required |
| 4.2 Failure-Mode Diagnosis | Reading a trace to find the cause, integration or model | Partly | 17 | Traces and structured logs, NOTES.md on a failure you diagnosed | Model failure modes (refusal, drift) are touched, not taught |

## Domain 5: Model Selection and Optimization (12%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 5.1 Generation Mechanics and Model Options | Tokens, context, sampling, thinking and effort | Not covered | | | Study the book's model chapters |
| 5.2 Model Tier Selection and Release Planning | Choosing a model tier, and regression evals across releases | Partly | 17 | The eval set, which you can rerun on a new model | No tier comparison |
| 5.3 Cost Diagnosis and Optimization | Finding the cost driver: caching, batching, token budgets | Not covered | 20 | Step 20 prices infrastructure, not model tokens | Study the book's cost lessons |

## Domain 6: Prompt and Context Engineering (15%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 6.1 Context Management for Long-Running Sessions | Compaction, pruning, subagents, fresh sessions | Partly | 2, 6 | Small, paginated, summarized tool results that keep the model's context small | No session or agent context management |
| 6.2 Prompt Revision | Revising a prompt to close a gap an eval found | Partly | 2, 17 | Tool descriptions revised after the eval set shows a wrong choice | Tool descriptions only, not system prompts |
| 6.3 Structured Output and Defensive Parsing | Strict schemas, validation, defensive parsing | Partly | 2, 16 | Zod schemas on every tool input, structured content in every result | Parsing model output isn't required |
| 6.4 Grounding and Source Citation | Answers grounded in documents, cited, or "not found" | Practised | 8 | Search that cites document and line or abstains below a threshold, measured in CI | |

## Domain 7: Security and Safety (8%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 7.1 Prompt Injection and Jailbreak Defense | Layered mitigations for agents that read untrusted content | Practised | 2, 15, 16, 19 | The injected-order-note test, untrusted text marked in results and escaped in widgets | |
| 7.2 Deterministic Guardrails and Least Privilege | Enforcement the model can't talk its way around | Practised | 4, 6, 7, 12 to 16 | Row-level security, per-tool scopes, sealed elicitation confirmations, run-time authority re-checks | Claude Code hooks aren't used |
| 7.3 API Key and Credential Handling | Keys server-side, environment separation, rotation | Practised | 0, 16, 17, 20 | `.dev.vars` and Wrangler secrets, a rotation note for every secret, test-mode Stripe keys, separate bootcamp accounts | Anthropic workspace and admin keys only in step 17 |

## Domain 8: Tools and MCP (10%)

| Objective | Scope, in short | Coverage | Steps | Evidence | Gap |
| --- | --- | --- | --- | --- | --- |
| 8.1 Tool Design and Response Shaping | Schema, description, small results, errors, approvals | Practised | 2, 3, 16, 20 | Tight schemas, annotations, recoverable errors, elicitation before costly actions | |
| 8.2 MCP Server Development and Integration | Building, deploying and connecting an MCP server, with auth | Practised | 1, 3, 4, 18 | A live `/mcp` server with tools, resources and prompts, OAuth, working in Claude and ChatGPT | Remote HTTP only; local stdio servers aren't built |
| 8.3 Capability Mechanism Selection | Built-in tool, custom tool, skill or MCP server, and why | Practised | 10, 20, 21 | The capstone's justified extras, and the step 10 choice of a served skill | |

## Summary

| Domain | Weight | This bootcamp |
| --- | --- | --- |
| 8. Tools and MCP | 10% | Practised in depth |
| 7. Security and Safety | 8% | Practised in depth |
| 6. Prompt and Context Engineering | 15% | Grounding practised; the rest partly |
| 4. Eval, Testing, and Debugging | 10% | Evaluation practised; diagnosis partly |
| 3. Claude Code | 8% | Operation practised; customization partly |
| 2. Applications and Integration | 20% | Partly |
| 1. Agents and Workflows | 17% | Mostly not covered |
| 5. Model Selection and Optimization | 12% | Mostly not covered |

About a third of the exam's weight (Domains 7 and 8, plus parts of 3, 4 and 6) is practised in depth here. Plan your study for Domains 1, 2 and 5 from the book before you sit the exam.

*Checked against exam guide v1.1 on November 27, 2026. If Anthropic publishes a new version, recheck every row.*
