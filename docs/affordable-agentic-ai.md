# Affordable Agentic AI Development

*A model-independent strategy for students and software developers. Student and faculty guide, version 3, 26 November 2026.*

> [!NOTE]
> **How this fits the bootcamp.** This bootcamp is standardized on Claude Code in Claude Desktop, which needs a paid Claude plan (see [What it costs](../README.md#what-it-costs)). This guide helps you keep your wider AI spending low: what is free, what is worth paying for, and how to compare the options before you buy. The prices, plans and models below change often, so check each provider's live pricing before you pay.

## Executive summary

Start with free tools. Pay for additional capacity only when a real learning or development workload warrants it. ChatGPT Free includes Codex; Go provides a low-cost general AI subscription with Codex access. OpenCode offers a free, open-source harness with a choice of model providers. Claude Pro remains useful for integrated knowledge work and coding.

**Student funding policy: Panaversity does not provide AI subscriptions, accounts, API credits or cloud computing. Students independently select, purchase and manage their own services and pay all associated charges. This guide is not an institutional procurement or subsidy plan.**

| **Option**                | **Monthly cost** | **Saving vs \$20** |
|---------------------------|------------------|--------------------|
| ChatGPT Free + Codex      | \$0              | 100%               |
| ChatGPT Go + Codex        | ~\$5.06\*        | 74.7%              |
| OpenCode + model APIs     | \$3–\$8 target   | 60–85%             |
| OpenCode Go               | \$10             | 50%                |
| Claude Pro / ChatGPT Plus | \$20 reference   | Baseline           |

\*All prices are in USD. Go’s approximately \$5.06 is an illustrative regional price from the earlier guide, not a verified current checkout quote. Taxes, payment fees, hardware, hosting and overages are excluded. API targets are untested scenarios; features and usage allowances are not equivalent. \[1–5\]

### The purchasing principle

Choose the lowest-cost option that completes your coursework reliably. Do not purchase multiple subscriptions by default. Keep a free fallback and upgrade only after observing real usage interruptions, model-quality limitations or a need for managed cloud execution.

## 1. Architecture: model, harness and runtime

| **Layer** | **Examples**                    | **Role**                           |
|-----------|---------------------------------|------------------------------------|
| Interface | Claude Desktop, Codex, OpenCode | User interaction                   |
| Harness   | Claude Code, Codex, OpenCode    | Context, tools, permissions, loops |
| Runtime   | Laptop, container, VM           | Executes tools and tasks           |
| Gateway   | OpenRouter, LiteLLM, Merge      | Routes and translates requests     |
| Model     | Claude, GPT, DeepSeek, Qwen     | Reasoning and generation           |

### Claude Desktop after the Chat and Cowork merger

The Chat and Code tabs do not mean Cowork capabilities have disappeared. Unified Claude combines conversation and knowledge-work features. Eligible scheduled tasks execute remotely; local files, apps and connectors can still require the desktop computer to be available. \[6, 7\]

Standard path: Claude Desktop → Anthropic-managed harness/runtime → Claude model. Third-party path: Claude Desktop → local execution environment → compatible model endpoint or gateway. The second path does not automatically preserve the managed VM, cross-device synchronization, cloud memory or scheduling. Third-party configuration is documented, but full non-Claude compatibility and individual deployment eligibility are not established here. \[8–11\]

### OpenCode as an alternative harness

OpenCode supports multiple providers, MCP, skills, permissions and persistent sessions. Students can connect directly to supported models or use a gateway. Its coding-oriented experience does not automatically reproduce Cowork’s document workflow, business connectors or organizational memory. \[12\]

For always-on execution, a student may run a compatible OpenCode server on a personal Linux VM. Secure authentication, network access, workspace isolation, backups, scheduling and event triggers need separate configuration. A remote Claude Desktop VM would also need supported OS and sandbox compatibility. Neither deployment has been tested for this guide; hosting is an extra personal expense.

## 2. Choosing an inference gateway

An Anthropic-compatible gateway accepts the Messages API. Translation to another provider may preserve basic messages without preserving every tool, reasoning, streaming or model-discovery feature. Test the complete agent loop, not merely a chat response.

| **Gateway / endpoint**      | **Hosting**            | **What to verify**                                                                        |
|-----------------------------|------------------------|-------------------------------------------------------------------------------------------|
| OpenRouter                  | Managed                | Claude Code guide; Desktop routing via supported configuration; model/tool compatibility. |
| LiteLLM                     | Self-hosted or managed | Desktop guide; deployment and model mappings.                                             |
| Merge Gateway               | Managed                | Desktop integration and model catalog.                                                    |
| Cloudflare AI Gateway       | Managed                | Desktop guide uses Anthropic endpoint; cross-model translation not established.           |
| Portkey / Vercel / Requesty | Varies                 | Candidate gateways; full Desktop support unverified.                                      |
| DeepSeek direct             | Provider API           | Native Anthropic-format endpoint; full Desktop compatibility unverified.                  |

DeepSeek publishes `https://api.deepseek.com/anthropic` as its Anthropic-format base URL. A separate gateway is not always necessary for a compatible client. \[5, 8–11, 13\]

### When to use a gateway

Use direct provider APIs for simplicity and transparent billing. Use a gateway when switching models, managing multiple keys, comparing costs or adding centralized routing is valuable. A gateway can introduce fees, another data processor, latency and operational dependency. It does not supply a cloud execution VM.

### Security and student ownership

Use a personal account and personal API key. Keep secrets outside source control, review data retention and processing regions, restrict tool permissions and avoid confidential client data. Where available, enable hard spending caps, disable automatic recharge and check billing frequently. A budget alert alone is not a hard cap.

## 3. Comparing platforms and plan limits

| **Capability**           | **Claude Pro**    | **ChatGPT Free / Go** | **ChatGPT Plus** | **OpenCode**                  |
|--------------------------|-------------------|-----------------------|------------------|-------------------------------|
| General chat             | Yes               | Yes                   | Yes              | Coding-oriented               |
| Agentic coding           | Yes               | Local Codex           | Codex            | Yes                           |
| Managed cloud coding     | Eligible          | No                    | Eligible         | Self-host separately          |
| Knowledge work           | Unified Claude    | Chat tools            | Work eligible    | Customize                     |
| Model choice             | Claude by default | OpenAI plan           | OpenAI plan      | Many providers                |
| MCP / skills             | Supported\*       | Codex\*               | Codex\*          | Supported\*                   |
| Open-source harness      | No                | Codex CLI             | Codex CLI        | Yes                           |
| Local open-weight models | 3P setup\*        | Not via plan          | Not via plan     | Supported                     |
| Base billing             | \$20              | \$0 / local Go price  | \$20 reference   | Free harness; inference extra |

\*Feature availability depends on client version, configuration and plan. Managed cloud access may depend on rollout. \[1, 2, 6–12\]

### Why pay for ChatGPT Go rather than Free?

Both plans include Codex and ordinary text chat, subject to safeguards. Go increases eligible upload, image generation and data-analysis allowances, provides longer memory and includes additional eligible features. It offers reasoning access but not every premium model. API consumption is billed separately. \[2, 14\]

Codex access is not unlimited coding. Go has a monthly allowance; Plus has shorter and weekly usage windows. Exact task capacity and the Free-to-Go coding improvement are not quantified here. Check the account’s usage meter and available models before paying. Free and Go do not include Codex Cloud. Paid instant resets are not offered on Go. \[2, 15–17\] Do not purchase Go solely for Codex until you have tested Free and determined whether the additional allowance justifies the price.

## 4. OpenCode Go: what \$10 buys

OpenCode Go is optional. The current plans are Go (\$10), Go Plus (\$40) and Go Teams (\$20 per member, minimum two). The paid models draw from a shared allowance, rather than separate monthly budgets for every model. \[4\]

| **Selected Go model** | **5-hour value** | **Weekly value** | **Monthly value** |
|-----------------------|------------------|------------------|-------------------|
| DeepSeek V4.1 Flash   | \$20             | \$50             | \$100             |
| GLM 5.3 Flash         | \$12             | \$30             | \$60              |
| Qwen3.7 Plus          | \$12             | \$30             | \$60              |
| MiniMax M3.1 Flash    | \$4              | \$10             | \$20              |
| DeepSeek V4 Pro       | \$3              | \$7.50           | \$15              |
| Kimi K3.1 Pro         | \$3              | \$7.50           | \$15              |

Values are illustrative metered usage equivalents based on the public Console examples, not withdrawable credits or guaranteed request counts. Five-hour and weekly ceilings are respectively 20% and 50% of the model’s monthly value. The weekly reset is Monday 00:00 UTC; monthly reset follows the paid period. Confirm live limits in the Console, as model lists and allowances change. \[4, 18\]

Example: using \$10 equivalent of a model with a \$100 monthly value consumes 10% of the shared allowance. Using \$3 equivalent on a \$15 model consumes another 20%. Switching models does not reset the allowance.

### Avoid unexpected charges

Check the Console usage meters. If a limit is reached, wait for the reset or switch to an available free model. Enable extra paid usage only deliberately; it can take total spend beyond \$10. Some models require opting into additional processing regions or data policies. Review these before activation.

### Who should choose it?

A student with frequent coding tasks who wants a predictable base subscription may prefer Go. A light user may spend less through direct API billing. Compare actual tasks per dollar rather than assuming any advertised request estimate translates into completed assignments.

## 5. Model API economics

The table uses direct DeepSeek off-peak pricing \[5\] and OpenCode Console pay-as-you-go rates \[19\] for other models. All rates are USD per million tokens. The regular workload is 50M input and 5M output with 80% cache-read hits. Cache-write surcharges, where applicable, are excluded; actual costs may be higher.

| **Model**      | **Input** | **Cache read** | **Output** | **Regular cost** |
|----------------|-----------|----------------|------------|------------------|
| DeepSeek Flash | \$0.15    | \$0.003        | \$0.6      | \$4.62           |
| DeepSeek Pro   | \$0.66    | \$0.022        | \$1.98     | \$17.38          |
| GLM 5.3 Flash  | \$0.15    | \$0.03         | \$0.5      | \$5.20           |
| Qwen3.8 Flash  | \$0.15    | \$0.016        | \$0.47     | \$4.49           |
| MiniMax M3     | \$0.3     | \$0.06         | \$1.2      | \$11.40          |
| Kimi K2.7 Code | \$0.95    | \$0.19         | \$4        | \$37.10          |

Formula: input millions × \[(1 − cache-hit share) × uncached rate + cache-hit share × cached rate\] + output millions × output rate. For Flash, the regular cost is \$4.62. DeepSeek peak pricing doubles the direct rates; in Pakistan, peak windows are 06:00–09:00 and 11:00–15:00 on weekdays, excluding Chinese public holidays. \[5\]

| **Monthly scenario** | **Light** | **Regular** | **Heavy**  |
|----------------------|-----------|-------------|------------|
| Input / output       | 10M / 1M  | 50M / 5M    | 200M / 20M |
| Flash off-peak       | \$0.92    | \$4.62      | \$18.48    |
| Flash all peak       | \$1.85    | \$9.24      | \$36.96    |
| Pro off-peak         | \$3.48    | \$17.38     | \$69.52    |
| Claude Pro base      | \$20      | \$20        | \$20\*     |

\*Heavy usage may exceed subscription allowances. These scenarios are hypothetical, not measured student averages. Qwen cache writes, for example, are priced separately; the simplified model does not include them. Model quality and tool reliability must be tested, not inferred from price.

### Candidate routing strategy

Try Flash-class models for routine edits; test MiniMax, Qwen and GLM on agent loops; escalate difficult architecture or debugging to stronger models such as DeepSeek Pro, Kimi or Claude. These are test candidates, not benchmark-backed rankings.

## 6. Sensitivity and financial impact

A coding agent repeatedly reads context, but cache hits are not guaranteed. The table holds the regular workload fixed and varies the input cache-hit share.

| **Cache-hit share** | **Flash off-peak** | **Flash all peak** | **Saving vs \$20 (off-peak)** |
|---------------------|--------------------|--------------------|-------------------------------|
| 0%                  | \$10.50            | \$21.00            | 47.5%                         |
| 50%                 | \$6.82             | \$13.65            | 65.9%                         |
| 80%                 | \$4.62             | \$9.24             | 76.9%                         |
| 95%                 | \$3.52             | \$7.04             | 82.4%                         |

At 80% cache hits and 25% of otherwise identical billed usage at peak rates, estimated Flash cost is \$5.78. At an 80% cache-hit rate, the regular workload remains below \$20 even at peak pricing. Lower cache-hit rates, higher consumption, retries and additional charges can push costs above \$20; zero cache hits at peak pricing cost \$21.

### Annual expenditure paid by students

| **Option**                 | **Per student/year** | **10,000 students/year** |
|----------------------------|----------------------|--------------------------|
| Claude Pro monthly billing | \$240                | \$2,400,000              |
| OpenCode Go                | \$120                | \$1,200,000              |
| ChatGPT Go (~\$5.06)       | \$60.72              | \$607,200                |
| OpenCode APIs (\$5 target) | \$60                 | \$600,000                |
| Free Codex                 | \$0                  | \$0                      |

At \$5/month, savings versus \$20 are \$15/month or \$180/year per student (75%). Across 10,000 students paying independently for 12 months, hypothetical aggregate savings are \$1.8 million. This is not a Panaversity budget saving, an enrollment forecast or a promise of equivalent access. Claude annual billing may lower its effective monthly cost; check current price. \[1\]

### Why not share a Claude account?

Three students splitting \$20 would nominally pay \$6.67 each, but Anthropic’s consumer terms prohibit credential sharing. Shared allowances, memory, files, connected apps and scheduled tasks also create privacy and assessment problems. Collaborate through shared repositories, not shared accounts or API keys. \[20\]

## 7. Setup guide: start without unnecessary spending

### A. ChatGPT Free or Go with Codex

1. Create your own account at chatgpt.com. Start on Free; upgrade to Go in the account menu only if justified.
2. Install the desktop app, CLI or IDE extension using the official Codex guide \[2, 21\].
3. Sign in with ChatGPT, open a practice repository and request a small change.
4. Review the diff, run tests and monitor Settings → Usage & billing or CLI /usage. Do not confuse API-key billing with subscription usage.

### B. OpenCode with DeepSeek or another API

1. Install OpenCode from opencode.ai/docs/ using the instructions for your OS and version.
2. Create a personal provider account and add only a small balance.
3. In OpenCode, use /connect to select a supported provider and supply the key, then /models to select a model; if needed, follow the custom endpoint configuration guide.
4. Open a test repository, approve only necessary tools, review changes and check provider usage after each session. \[5, 12, 22\]

### C. OpenCode Go

1. Sign into the OpenCode Console and purchase the personal Go plan.
2. Copy your personal key, connect OpenCode Go via /connect and choose a model via /models.
3. Check allowance consumption and keep extra usage disabled until you explicitly need it. \[4\]

### D. Claude Desktop with OpenRouter (experimental)

1. Verify Desktop third-party inference eligibility and install the supported version.
2. Create your own OpenRouter key and set a small budget.
3. Follow the vendor’s Desktop configuration instructions (developer mode and Gateway, if available); use the documented endpoint and model identifiers, not guessed URLs.
4. Restart as instructed, test chat, file edits, tool calls and MCP separately. The Claude Code environment-variable setup is not necessarily the Desktop setup. \[8, 13, 27\] OpenRouter’s Ori documentation also describes desktop routing; this does not establish that Unified Chat/Cowork and cloud features work with every model.

If third-party Desktop configuration is unavailable, use OpenCode directly or the documented Claude Code gateway integration. Do not patch the app or bypass account controls. None of these setup flows was executed for this guide.

## 8. Local models and a progressive learning path

### OpenCode with Ollama: no recurring API charge?

Install Ollama, choose a supported open-weight model with a license suitable for your use, download a quantized variant, and connect OpenCode to the local Ollama provider following its provider guide. Start with a small model, run a tool-calling test, and increase size only if hardware and quality permit. \[12, 23\]

A 7–8B model quantized to 4 bits requires roughly 4 GB for weights alone; actual RAM/VRAM needs are higher because of runtime overhead and context. A CPU can run suitable quantized models, but speed may be inadequate for long agent loops. Larger models and context windows require more memory. Local inference avoids provider token fees, not electricity, hardware, maintenance or time costs. Never assume a local small model matches a hosted frontier model.

| **Stage**        | **Personal monthly budget** | **Recommended approach**                                             |
|------------------|-----------------------------|----------------------------------------------------------------------|
| 1\. Free entry   | \$0                         | Free Codex; optional local model if suitable hardware already exists |
| 2\. Affordable   | \$3–\$8 target              | Go for broader chat tools or OpenCode APIs after testing             |
| 3\. Regular      | \$10                        | OpenCode Go if allowances fit                                        |
| 4\. Professional | \$20+                       | Claude Pro or ChatGPT Plus if premium features justify cost          |

These are alternatives, not mandatory sequential purchases. Students may remain on a free tier indefinitely if it meets their needs. Panaversity teaches portable concepts: prompting, agent loops, harness engineering, MCP, skills, evaluations, secure tools and human review.

### A one-month volunteer pilot

Compare Free Codex, Go Codex, OpenCode with a \$5 API target, OpenCode Go and Claude Pro. Recruit students who already have or independently choose to buy access. Use the same small feature, failing-test debugging, refactor, MCP task and documentation assignment. Keep repositories, task instructions and time budgets comparable.

Record actual personal expenditure, accepted tasks, test pass rates, human corrections, elapsed time and limit interruptions. Cost per accepted task = total expense / accepted tasks; report unsuccessful tasks separately. Evaluate local and cloud workflows separately. Publish results before claiming cost or quality parity.

## 9. Free hosted models and direct API decisions

Start with Free Codex and try free hosted models in OpenCode before buying any plan. In OpenCode, open /models and select a currently available free model; availability may change. OpenRouter also offers free model variants and a free router. Connect using /connect, select OpenRouter, and explicitly select a free model. A free router may choose different models over time. \[24–26\]

OpenRouter lists a baseline free allowance of 50 requests per day; other free offerings and provider-specific limits may differ. A single coding task can make many requests. Expect queues, throttling, model changes and incomplete tool support. Read the applicable provider data policy before sending code; never send secrets, private student records or confidential client repositories to an unreviewed endpoint. Free hosted inference is not a guaranteed production service.

### Direct API versus OpenCode Go

At the illustrative DeepSeek Flash off-peak rate and 80% cache hits, direct API costs \$0.92 for light usage, \$4.62 for regular usage and \$18.48 for heavy usage. Compared with Go’s \$10 base fee, direct billing is cheaper by \$9.08 or \$5.38 for the first two scenarios; Go could save \$8.48 in the heavy scenario if its actual model-specific allowances and task quality suffice. The two services may use different rates and limits, so this is not a like-for-like token entitlement comparison.

At the regular scenario’s 10:1 input/output ratio and 80% cache hits, \$10 buys approximately 108.2M input and 10.8M output tokens at the stated off-peak direct rate. Peak pricing, retries and lower cache hits reduce that capacity. Start with a small prepaid balance; choose a subscription only when your measured workload justifies it.

### Payment considerations for students in Pakistan

Check whether your own payment method supports international recurring payments. Compare the final USD checkout amount, applicable taxes, bank or card fees and exchange-rate exposure. Avoid unofficial resellers or shared credentials. Cancel unused subscriptions and disable automatic recharge where possible. Panaversity does not purchase or subsidize any of these services.

## 10. A one-minute purchasing decision

1. Start with Free Codex and available free OpenCode models. If coursework is completed reliably, stay free.
2. If general chat, uploads or analysis limits are the problem, consider ChatGPT Go after checking the personal price and Codex allowance.
3. If coding capacity or model choice is the problem, test direct Chinese model APIs with a \$3–\$8 target.
4. If repeated API expenditure approaches \$10, compare OpenCode Go’s actual allowance and interruptions before subscribing.
5. If quality, advanced knowledge work or managed cloud execution is essential, compare Claude Pro and ChatGPT Plus. Buy only the features you need.

### Pilot acceptance criteria (proposed, not measured)

Use the same tasks and time limits across groups. Aim for at least 90% of assigned tasks accepted and at least 95% of automated tests passed, with a target personal monthly expenditure below \$8 for the low-cost arm. Compare elapsed time with the Claude Pro arm; record corrections, retries and limit interruptions. Require zero critical security violations. These targets are hypotheses, not results or guarantees.

Report median and range of spend, time and accepted-task cost by arm; include failed or abandoned tasks and distinguish subscription cost from API, infrastructure and payment charges. Record exact model versions, cache rates and allowance windows. Volunteers may use tools they already own; participation does not require purchasing a paid plan. Do not extrapolate a small pilot to all students without reporting its limitations.

## 11. References and verification notes

1. Claude pricing — <https://claude.com/pricing>
2. Codex plan access and setup — <https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan>
3. Pakistan App Store reference — <https://apps.apple.com/pk/app/chatgpt/id6448311069>
4. OpenCode Go plans and limits — <https://opencode.ai/docs/console/go>
5. DeepSeek rates and endpoints — <https://api-docs.deepseek.com/quick_start/pricing/>
6. Cowork scheduling — <https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork>
7. Cowork web, desktop, mobile — <https://support.claude.com/en/articles/15520349-use-claude-cowork-on-web-desktop-and-mobile>
8. Claude third-party setup (original reference) — <https://claude.com/docs/third-party/claude-desktop/in-app-configuration>
9. LiteLLM Desktop setup — <https://docs.litellm.ai/docs/proxy/client_setup/claude_desktop>
10. Merge Desktop setup — <https://docs.merge.dev/merge-gateway/coding-agents/claude-desktop>
11. Cloudflare Desktop setup — <https://developers.cloudflare.com/ai-gateway/integrations/coding-agents/claude-desktop/>
12. OpenCode providers — <https://opencode.ai/docs/providers/>
13. OpenRouter agent integration — <https://openrouter.ai/docs/guides/coding-agents/claude-code-integration>
14. ChatGPT Go features — <https://help.openai.com/en/articles/11989085-what-is-chatgpt-go>
15. Codex allowance resets — <https://help.openai.com/en/articles/20001498-how-resets-work-in-codex>
16. Paid reset eligibility — <https://help.openai.com/en/articles/20001507-paid-weekly-work-and-codex-rate-limit-resets>
17. ChatGPT plan pricing — <https://chatgpt.com/pricing>
18. OpenCode Console allowance examples — <https://opencode.ai/console>
19. OpenCode pay-as-you-go rates — <https://opencode.ai/docs/console/models/>
20. Anthropic consumer terms — <https://www.anthropic.com/legal/consumer-terms>
21. Codex developer documentation — <https://developers.openai.com/codex/>
22. OpenCode installation — <https://opencode.ai/docs/>
23. Ollama — <https://ollama.com/>
24. OpenCode models and free model selection — <https://opencode.ai/docs/models/>
25. OpenRouter free model catalog — <https://openrouter.ai/collections/free-models>
26. OpenRouter pricing and free rate limits — <https://openrouter.ai/pricing>
27. OpenRouter Ori desktop setup — <https://learn.openrouter.ai/courses/get-started-with-ori/connect-your-agent>

Verification: Version 3 rechecked selected documentation for Codex access and Go resets, OpenCode free models and Go limits, OpenRouter free pricing and Ori desktop routing on 26 November 2026. Other rates, feature claims and setup links are carried forward from Version 2; verify live provider documentation and personal checkout before purchasing. No installation, model compatibility, workload or outcome tests were performed.
