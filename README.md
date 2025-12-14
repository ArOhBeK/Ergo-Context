# ErgoScript Development Context Package

This package is a unified reference set for ErgoScript smart contract development on the Ergo blockchain.  
All files describe the same domain knowledge—extended UTXO (eUTXO) concepts, secure patterns, known issues, and best practices—but in different formats so you can use them with both humans and tools, including LLMs and autonomous agents.

---

## 1. context.md

**Type:** Markdown (primary human-readable reference)

This is the main document you should read first. It includes:

- Links + local/offline paths to core references (LangSpec, tutorials, examples, whitepaper) via `./local_files/`
- Overview of the eUTXO model on Ergo
- How boxes, registers (R4–R9), tokens, and scripts fit together
- Secure contract patterns (Commit–Reveal, perpetual token boxes, multi-stage flows, etc.)
- Known ErgoScript issues and common pitfalls
- Guidelines for safe state transitions
- High-level rules for how contracts should be designed and audited

Core reference files are mirrored under `local_files/` so air-gapped or localized LLM setups can load the same source material without internet access. In `context.json`, `context.yaml`, and `context.toml`, use `core_references_local` for offline paths.

**How to use it (developers):**

- Keep `context.md` in your ErgoScript repo under `/context` or `/docs`.
- Read it when designing new contracts or reviewing existing ones.
- Treat it as the “house style guide” for ErgoScript in your project.

**How to use it (LLMs/agents):**

- Load the entire file into a **system prompt** or “developer context” for an LLM-powered assistant that helps write or review ErgoScript.
- Use it as the **base layer of domain knowledge**, and then add the user’s specific contract or question on top.

---

## 2. context.json

**Type:** JSON (machine-readable)

This file contains the same logical sections as `context.md`, but as structured JSON objects.

**How to use it (tools):**

- Load it from scripts that assist in ErgoScript development.
- Use it in Node.js, Python, Rust, or any environment that prefers structured JSON.
- Build tools that can:
  - show known issues,
  - list secure patterns,
  - or verify simple rules against your contracts.

Example (Node.js):

```js
const fs = require("fs");
const ctx = JSON.parse(fs.readFileSync("./context.json", "utf8"));
console.log(ctx.known_issues);
```

**How to use it (LLMs/agents):**

- Let an agent read `context.json` at startup to build an internal map of:
  - `known_issues` to check against contracts,
  - `secure_patterns` to recommend during code generation,
  - `eutxo_model` principles to validate state transitions.
- Agents can call into this JSON instead of relying only on natural-language context.

---

## 3. context.yaml

**Type:** YAML (config-style)

Same content as `context.json`, but in YAML. This is helpful for configuration-driven setups.

**How to use it (CI/CD & infra):**

- Reference in CI/CD pipelines (e.g., GitHub Actions, GitLab CI) to:
  - enforce that contracts meet basic rules,
  - gate deployment with simple checks.
- Use with tools or services that natively consume YAML config.

Example (pseudo-use):

```yaml
known_issues:
  - "unvalidated data inputs"
  - "token loss / unpreserved tokens"
```

**How to use it (agents):**

- Use `context.yaml` as the **config file** for an ErgoScript agent, for example:
  - list which patterns to enforce,
  - define which issues to warn or fail on,
  - define severity levels per issue.

---

## 4. context.toml

**Type:** TOML (Rust-friendly configuration)

This file is aimed at Rust-based tooling and services that interact with Ergo or ErgoScript.

**How to use it (Rust tools):**

- Load it in Rust CLIs, backend services, or analysis tools that help with contract generation or verification.
- Use it to keep a consistent config across tools without manually duplicating settings.

Example (Rust):

```rust
use serde::Deserialize;
use std::fs;

#[derive(Deserialize)]
struct Context {
    known_issues: Vec<String>,
}

let data = fs::read_to_string("context.toml")?;
let ctx: Context = toml::from_str(&data)?;
println!("{:?}", ctx.known_issues);
```

**How to use it (Rust-based agents):**

- A Rust agent can load `context.toml` at startup to:
  - access known issues,
  - enforce secure patterns,
  - and drive linting/audit logic over contract source or ErgoTree.

---

## 5. context.loop

**Type:** S-expression / LISP-like

A compact, tree-structured representation of the same information.

**How to use it:**

- For custom parsers or tools that like S-expression input.
- For symbolic or logic-based analysis (e.g., custom static analyzers).
- For experiments where you treat the context as structured, navigable data.

**How to use it (agents):**

- Logic-based or symbolic agents can parse `context.loop` to reason over the structure of:
  - patterns,
  - issues,
  - and eUTXO principles as a tree rather than raw text.

---

## 6. keyword_index.txt

**Type:** Plain text keywords

A flat list of important terms and phrases pulled from the context, such as:

- ErgoScript
- LangSpec
- eUTXO
- Commit–Reveal
- Token Preservation
- HEIGHT validation
- Registers R4–R9
- State transition invariants

**How to use it:**

- Seed search indices or autocomplete systems.
- Use as a quick reference for naming conventions and core concepts.
- Drive tagging or filtering in developer tools.

**How to use it (LLMs/agents):**

- Use these as **boosted terms** or “must-know concepts” when designing prompts.
- Use to build a **vector index** of related documents (e.g., LangSpec, tutorials) and map them to these keywords.

---

## 7. embeddings_input.txt

**Type:** Plain text (embedding-ready)

This file is structured as grouped text sections, formatted for bulk ingestion by any system that:

- builds semantic search over your docs,
- computes text embeddings,
- does keyword or similarity analysis.

**How to use it (LLMs & RAG):**

- Run `embeddings_input.txt` through your embedding model.
- Store the resulting vectors in a vector database (or basic index).
- At query time, retrieve the most relevant sections and feed them into the LLM as additional context alongside the user’s contract or question.

This is the simplest “single-file” starting point for a Retrieval-Augmented Generation (RAG) pipeline around ErgoScript knowledge.

---

## 8. rag_chunks.json

**Type:** JSON (pre-chunked reference sections)

This file breaks the context into small sections (“chunks”), each with:

- `id` — stable identifier (e.g., `core_references`, `known_issues`)
- `title` — human-friendly name
- `tags` — labels for filtering
- `text` — a short description of that topic

**How to use it (tools):**

- Load these chunks into a search index or knowledge store.
- When a tool needs advice (e.g., on token handling or known issues), look up the relevant chunk by ID or tags.

**How to use it (LLMs/agents, full RAG setup):**

1. **Ingest:**  
   - For each chunk in `rag_chunks.json`, embed the `text`.  
   - Store `{id, title, tags, text, vector}` in your vector store.

2. **Query:**  
   - When the user asks a question (e.g. “Is this auction contract safe?”), embed the question.  
   - Retrieve the top-k most similar chunks from the vector store (e.g. `eutxo_model`, `known_issues`, `secure_patterns`).

3. **Compose prompt:**  
   - Build a prompt like:  
     - System: high-level instructions (e.g. “You are an ErgoScript contract auditor...”).  
     - Context: include the retrieved `text` chunks from `rag_chunks.json`.  
     - User: include the contract or question.

4. **Answer:**  
   - Let the LLM respond, grounded in those specific, retrieved chunks of context, instead of guessing.

This is the **recommended way** to integrate this package with an LLM-powered ErgoScript assistant or contract-auditing agent.

---

## 9. context.pdf

**Type:** PDF

A plain PDF rendering of the main context content.

**How to use it:**

- Share with other team members or auditors.
- Attach to documentation packages, audit reports, or specs.
- Print or review offline.

Agents don’t typically use this directly, but it’s useful for humans collaborating with them.

---

## 10. sphinx_docs/index.rst & sphinx_docs/context.rst

**Type:** reStructuredText (Sphinx documentation)

These are the starter files for building a Sphinx-powered documentation site for the context.

- `index.rst` — entry point / table of contents.
- `context.rst` — overview page describing what the context covers.

**How to use them (docs):**

1. Ensure Sphinx is installed:
   ```bash
   pip install sphinx
   ```
2. Run:
   ```bash
   sphinx-build -b html sphinx_docs/ build/
   ```
3. Open `build/index.html` in your browser to see a browsable set of docs based on this package.

**How to use them (LLMs):**

- If you push these docs to a public site (e.g., ReadTheDocs), you can also mirror that content into your RAG store alongside `rag_chunks.json`, giving the LLM richer, linked context.

---

## Putting It All Together For Ergo Development

### When designing a new ErgoScript contract

1. Read **`context.md`** to understand:
   - how a box’s value, tokens, registers, and script interact,
   - how state should transition between boxes,
   - which patterns are recommended (e.g., commit–reveal, token-preserving boxes).

2. Refer to the **known issues** section (from any format):
   - unvalidated data inputs,
   - OR-branch bypasses,
   - missing signature checks,
   - incorrect use of `HEIGHT`,
   - token loss or value mis-accounting.

3. Follow the **secure patterns** for:
   - handling refunds,
   - continuing state across multiple boxes,
   - enforcing correct token and ERG flows.

4. If you are using an LLM assistant:
   - Load `context.md` (or relevant chunks from `rag_chunks.json`) into the system context.
   - Provide your intended contract behavior and let the model suggest an implementation that aligns with these rules.

---

### When auditing an existing contract

1. Open **`context.md`** and read the **Known Issues** and **Secure Patterns** sections side by side with the contract.

2. Check:
   - Are all registers (especially R4–R9) used as intended?
   - Are tokens preserved where they must be?
   - Are HEIGHT and time-based conditions used safely?
   - Are all paths protected by proper signatures or proofs?
   - Does every state transition obey the eUTXO expectations described here?

3. If you are using an LLM as an audit assistant:
   - Provide the contract text as user content.
   - Load relevant `rag_chunks.json` entries (e.g. `known_issues`, `secure_patterns`, `eutxo_model`) as context.
   - Ask the model explicitly: “List potential vulnerabilities according to the known issues and patterns provided above.”

4. Use **`context.json`**, **`.yaml`**, or **`.toml`** if you are writing automated checks or static analysis tools to assist in the audit.

---

## Suggested Project Layout

You can lay these files out in a repository like this:

```text
/context
  context.md
  context.json
  context.yaml
  context.toml
  context.loop
  keyword_index.txt

/local_files
  LangSpec.md
  ErgoScript.pdf
  AdvancedErgoScriptTutorial.pdf
  ergoscript-by-example-main/

/docs
  context.pdf
  sphinx_docs/
    index.rst
    context.rst

/analysis
  rag_chunks.json
  embeddings_input.txt
```

This keeps human-facing docs, configs, and machine-oriented assets organized but synchronized.

---

## Goal of This Package

By keeping all of these files together and wiring them into your ErgoScript tooling and LLM/agent stack, your projects always have:

- A single source of truth for how contracts **should** be designed.
- Multiple formats so that humans, scripts, tools, and services can all consume the same knowledge.
- A consistent way to reason about security, correctness, and maintainability across every contract you write or review.
- A ready-made backbone for any ErgoScript-aware LLM assistant or autonomous contract-auditing agent.
