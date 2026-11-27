# Step 08: Search by Meaning with pgvector: Answers That Cite or Abstain

> 🚧 **Under development.** This step is being written. The plan below shows what it will cover, and the code and instructions are coming soon.

Clients' questions are often about their documents, not their tables: "can I give a 15% discount to a new customer?", "what's our returns rule for opened items?", "do we sell anything for cold storage?". Searching for the same words misses most of these, because people don't use the policy's exact words. In this step your MCP server searches by **meaning** with **pgvector**, the Postgres add-on for embeddings, inside the same Neon database, the same tables and the same row-level security as the rest of the business data. Every answer cites the document and line it came from, the search **abstains** when nothing is close enough, and a fixed question set measures retrieval so a change that makes it worse fails the build.

> 🏬 **Stockroom's ask:** Customers keep asking questions their tables can't answer: discount rules, return policies, what the catalog says about cold storage. Stockroom wants answers from each customer's own documents, with the source cited, and an honest "not covered" when nothing fits.

## Learn first

This bootcamp is the hands-on companion to [The AI Agent Factory](https://agentfactory.panaversity.org/). Study these lessons from the book **before** you start building this step:

- [19.13 How does a search find a rule by its meaning?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/retrieval-from-zero/). Passages, embeddings, vectors, similarity search and an index, with every search kept inside one company.
- [19.14 Where does a search by meaning go wrong?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/embeddings-and-pgvector-in-depth/). Cosine distance, what embeddings miss, search by words beside search by meaning, and what a vector costs.
- [19.15 What if the policies do not answer the question?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/answers-that-cite-or-abstain/). Citations, a threshold, and saying plainly when the documents don't answer.
- [19.16 How do you know the search still works?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/measuring-retrieval/). A fixed question set that fails the build before a user sees a wrong citation.
- [Field guide F4, Embeddings and vectors from zero](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/field-guide-business-data-and-retrieval/#f4-embeddings-and-vectors-from-zero). The book's model (`bge-m3` on Workers AI, 1,024 dimensions), its free allowance, and a test embedder for CI.
- *Refresh:* [19.8 What if a query forgets its company filter?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/row-level-security/) and [19.9 What if the last call's company is still set?](https://agentfactory.panaversity.org/building-ai-workers/business-data-and-retrieval/one-company-per-transaction/). Passages and embeddings live under the same rules as every other row.

**Official docs.** Keep these open while you build:

- [Neon: the pgvector extension](https://neon.com/docs/extensions/pgvector). Enabling pgvector, the `vector` type, distance operators and indexes on Neon.
- [Cloudflare Workers AI: bge-m3](https://developers.cloudflare.com/workers-ai/models/bge-m3/). The embeddings model the book uses, called from the Worker.
- [Cloudflare Workers AI: bindings](https://developers.cloudflare.com/workers-ai/configuration/bindings/). The `AI` binding your Worker uses to embed text.
- [Cloudflare Workers AI: pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/). The daily free allowance and what embedding costs beyond it.
- [PostgreSQL: full-text search](https://www.postgresql.org/docs/current/textsearch.html). Search by words, for codes and names that embeddings miss.

## Why this step matters

A model that answers a policy question from memory, or from the closest wrong rule, can cost a client real money: a discount that wasn't allowed, a refund that shouldn't have been given. Retrieval that cites its source lets a person check the answer in seconds, and abstaining when nothing fits turns "a confident wrong answer" into "the policies don't cover this, ask your manager". Doing it inside Postgres, under the same row-level security, means search by meaning adds no new way for one company to see another's documents. This is the most-requested feature in client MCP work, and the one most often built unsafely.

## Why this technology in an MCP server: pgvector and retrieval on Neon

- **What it is.** An embeddings model turns text into a list of numbers (a vector) so that texts with similar meaning get similar numbers. pgvector stores those vectors in a Postgres column and finds the nearest ones by cosine distance (`<=>`), with an index once the table grows. Retrieval splits documents into short passages, embeds each one, and returns the passages closest to the question.
- **Why an MCP server uses it.** The model can only answer from what your tools return. A search tool that returns the right passages, with their document and line, lets the model answer from the client's own documents and cite them, in Claude and ChatGPT alike.
- **What it solves.** Questions phrased in the user's words rather than the document's ("PO" for "purchase order"), answers no one can check, confident answers to questions the documents don't cover, and a separate vector database with its own copy of every company's data.
- **Alternatives.** A dedicated vector database (another service, another copy of the data and another place to enforce tenancy), search by words only (misses meaning), or putting whole documents in the prompt (costly, and the citation points at nothing). The book keeps vectors in Postgres with pgvector, so one database and one set of rules covers both.

## What you will learn

- Explain passages, embeddings, dimensions, cosine distance and nearest-neighbour search, and predict what a search returns before you run it
- Split documents into one passage per rule, each stored with its document and line number, so every answer can cite exactly where it came from
- Embed passages and questions with the same model (`bge-m3` on Workers AI), and check every vector has the expected 1,024 dimensions
- Store passages and their embeddings in Neon under the same forced row-level security as every other table, and search only the signed-in company's passages
- Return cited passages, and abstain below a similarity threshold instead of offering the closest wrong passage
- Measure retrieval with a fixed question set ("found in the top three", and abstains where it should), and fail the build when it gets worse
- Know when to add search by words (codes, SKUs, names) beside search by meaning, and when to add a vector index, using `EXPLAIN`

## What you will build

The step 7 multi-tenant sales server, now with search by meaning:

- A migration that enables pgvector and adds `policy_documents` and `policy_passages` (`org_id`, document, line, text, `embedding vector(1024)`), plus an embedding column on `products` for their descriptions, all with forced row-level security and the company in every key.
- Sales policies for two companies (discounts, returns, credit terms, each a short list of numbered rules, with one company's discount limit lower than the other's), loaded by `npm run embed`.
- `search_sales_policy` (read-only): returns up to three passages with document, line and similarity, or `abstained: true` with a message telling the model to say the policies don't cover it.
- `find_products` (read-only): finds products by meaning from their descriptions ("something for keeping vaccines cold"), scoped to the company.
- `add_policy_document` (admin only): splits a new document into one passage per rule, embeds it through the Worker's `AI` binding and stores it, replacing the previous version of that document.
- `evals/retrieval.json`: twelve questions, six per company (five the policies answer and one they don't), each with the expected document and line or `abstain`. `npm run eval:retrieval` reports found-in-top-three and abstention, and CI fails if either drops.
- A deterministic test embedder for unit tests and CI, so tests need no network, and the real model for the measured run.

## Key concepts

- **Passage**: One short piece of a document, such as one rule, stored as one row with its document and line number.
- **Embedding**: A list of numbers a model makes from text, so that texts with similar meaning get similar numbers. Stored in a `vector` column.
- **Dimensions**: The length of that list. The column type must match the model, here `vector(1024)`.
- **Cosine distance**: How far apart two embeddings point, written `<=>` in pgvector. Smaller means closer in meaning.
- **Threshold and abstain**: The lowest similarity the search accepts. Below it, the tool says it found nothing close enough.
- **Citation**: The document and line returned with every passage, so a person can check the answer.
- **Question set**: A fixed list of questions with the passage that should come back, or `abstain`, run on every change.
- **Hybrid search**: Search by words and by meaning, merged. Worth adding only when the question set shows it helps, for example with product codes.
- **Vector index (HNSW)**: An index that finds near neighbours fast on large tables, at the cost of exactness. Check with `EXPLAIN` whether it is used. pgvector uses it only when the query orders by the distance operator and has a `LIMIT`.

## Planned outline

1. Search by words versus search by meaning
2. Passages, embeddings and pgvector on Neon
3. Embedding with Workers AI, and a test embedder
4. Tenant-scoped search under row-level security
5. Citations, the threshold and abstaining
6. Products by meaning, and when to add search by words
7. Measuring retrieval with a question set in CI
8. Indexes, `EXPLAIN` and cost
9. Testing in Claude and ChatGPT

## Prerequisites

- [Step 7](../step-07-multi-tenancy/)
- The tools and accounts listed in the [main README](../README.md#prerequisites)

## Build it with Claude Code

Every step is built with **Claude in a thread of your bootcamp's Claude project**, working on your local clone (see [How we work](../README.md#how-we-work-local-first-human-in-the-loop)). Follow the instructions in order and paste each prompt into the thread. Read what Claude proposes before you approve it, because understanding the code is the point. Claude never commits or pushes: you review the changes, then commit and push them in GitHub Desktop. The prompts use Anthropic's `build-mcp-server` and `build-mcp-app` skills (see [Built-in skills](../README.md#built-in-skills)). As in step 5, predict what each query and search returns before you run it.

> These prompts are a draft and will be refined as the step is developed.

**1. Start a new thread for this step in your bootcamp's Claude project, then copy your step 7 code into `step-08-semantic-search-pgvector/`.**

```text
Copy my step-07 code into step-08-semantic-search-pgvector/ so I can build on it. Copy only the source and config files: skip README.md (this folder already has the bootcamp's README, which must not change), NOTES.md, node_modules/, .wrangler/, dist/ and .dev.vars. Then rename the Worker in the wrangler config to learn-mcp-step-08 so it deploys alongside my earlier steps instead of replacing them, run npm install, and confirm the project still builds. Then create a Neon branch named step-08 from my step-07 branch and point this step's database settings (the connection string in .dev.vars and as a secret, and Hyperdrive once the server uses it) at the new branch, so my earlier steps keep working while they are graded.
```

**2. Switch to plan mode and plan retrieval.**

```text
Read the book lessons 19.13 to 19.16 and field guide F4 linked in this step, and the Neon pgvector docs. Explain passages, embeddings, cosine distance, the threshold and abstaining in plain words. Then plan search by meaning for this server: the tables and migration, one passage per rule, the embeddings model (bge-m3 on Workers AI, 1,024 dimensions), how every search stays inside the signed-in company under row-level security (including the dedicated-tier company from step 7), the tools, and the question set. Don't write code yet.
```

**3. Add pgvector and the passages.**

```text
Write a migration that enables the vector extension and creates policy_documents and policy_passages (org_id, document_id, line, content, embedding vector(1024)) with the company in every key, and adds a description embedding column to products. Force row-level security on the new tables with USING and WITH CHECK policies, exactly like step 6. Apply it to every tenant database with the step 7 migration script.
```

**4. Embed the documents.**

```text
Add the AI binding to the wrangler config and an embed() helper that calls @cf/baai/bge-m3 and fails loudly unless every vector has 1,024 numbers. Add a deterministic test embedder selected by an environment variable, so unit tests and CI need no network. Write short sales policies for two companies (discounts, returns, credit terms, as numbered rules, with a lower discount limit for one company), and an npm run embed script that splits them into one passage per rule, embeds them and loads them for each company. Explain why the question and the passages must use the same model.
```

**5. Search that cites or abstains.**

```text
Add search_sales_policy: embed the question, find the three nearest passages for the signed-in company only (inside the step 6 transaction with the company set, never from tool input), and return each with document, line and similarity. Below a threshold, return no passages, abstained: true and a message telling the model to say the policies don't cover it. Add find_products, which searches product descriptions by meaning for the company. Both read-only, with descriptions that tell the model to cite what it uses. Add a test that one company's search never returns another company's passage.
```

**6. Let admins add documents.**

```text
Add an admin-only add_policy_document tool that splits a document into one passage per rule, embeds the passages through the AI binding, and replaces the previous version of that document for the company in one transaction. Set the right annotations, and return a clear error for members and for documents that are too large.
```

**7. Measure retrieval.**

```text
Create evals/retrieval.json with twelve questions, six per company: five that the policies answer and one off-topic question, each with the expected document and line or "abstain". Add npm run eval:retrieval to report found-in-the-top-three and correct abstentions, run it with the real model, and tune the threshold. Make CI run it with the test embedder and fail if either number drops. Then try search by words beside search by meaning for three product-code questions, and add hybrid search only if the question set shows it helps.
```

**8. Indexes, cost, deploy and test.**

```text
Show me with EXPLAIN whether a search uses a vector index at this size, add an HNSW index and explain when it starts to matter. Make sure the search orders by the distance operator with a LIMIT, because pgvector only uses the index for that shape of query. Explain why the company filter, applied after an approximate index scan, can return fewer than three passages, and whether to turn on pgvector's iterative index scans. Estimate the storage and Workers AI cost per thousand passages. Then deploy and give me prompts to try in Claude and ChatGPT as users of two companies: a policy question in the user's own words, a question the policies don't answer (it must abstain), a product search by meaning, and the same question asked by each company getting each company's own rule.
```

## Testing targets

- claude.ai and Claude Desktop (primary)
- ChatGPT (developer mode)
- The retrieval question set, in CI

## Submission checklist

Submit through your **fork**, following [SUBMITTING.md](../SUBMITTING.md). ⛔ Do not open a pull request to the bootcamp repo.

- [ ] Your fork is synced with the bootcamp repo
- [ ] All code for this step is in `step-08-semantic-search-pgvector/` and pushed to your fork
- [ ] No secrets are committed (`.dev.vars.example` only, and no Neon connection strings or API tokens)
- [ ] The server is deployed to your own Cloudflare account and the `/mcp` URL works
- [ ] `evals/retrieval.json` runs in CI, and NOTES.md records found-in-the-top-three, abstentions and the threshold you chose
- [ ] Tenant isolation: passages and embeddings are under forced row-level security, and a test proves one company's search never returns another company's passage
- [ ] It works in claude.ai or Claude Desktop
- [ ] It works in ChatGPT (developer mode)
- [ ] Project defence recorded (up to five minutes: it works, one tool end to end, one tenant-isolation decision, one change you made; see [SUBMITTING.md](../SUBMITTING.md#10-defend-your-project)) and linked in the form
- [ ] Tagged `step-08-done` and pushed (in GitHub Desktop: **History**, right-click the commit, **Create Tag…**, then **Push origin**)
- [ ] Submitted in the form: fork URL, tag `step-08-done`, live server URL, and screenshots of a cited policy answer and an abstention in **Claude** and **ChatGPT**

## References

- [Neon: the pgvector extension](https://neon.com/docs/extensions/pgvector)
- [pgvector on GitHub](https://github.com/pgvector/pgvector)
- [Cloudflare Workers AI: bge-m3](https://developers.cloudflare.com/workers-ai/models/bge-m3/)

---

← [Step 07: Multi-Tenant MCP Servers and Databases](../step-07-multi-tenancy/) · [Roadmap](../README.md#roadmap) · [Step 09: Generative UI with MCP Apps](../step-09-generative-ui-mcp-apps/) →
