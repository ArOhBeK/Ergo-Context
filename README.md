# ErgoScript Development Context Package

This package is a unified reference set for ErgoScript smart contract development on the Ergo blockchain.  
All files describe the same domain knowledge—extended UTXO (eUTXO) concepts, secure patterns, known issues, and best practices—but in different formats so you can use them with both humans and tools.

---

## 1. context.md

**Type:** Markdown (primary human-readable reference)

This is the main document you should read first. It includes:

- Links to core references (LangSpec, tutorials, examples, whitepaper)
- Overview of the eUTXO model on Ergo
- How boxes, registers (R4–R9), tokens, and scripts fit together
- Secure contract patterns (Commit–Reveal, perpetual token boxes, multi-stage flows, etc.)
- Known ErgoScript issues and common pitfalls
- Guidelines for safe state transitions
- High-level rules for how contracts should be designed and audited

**How to use it:**

- Keep `context.md` in your ErgoScript repo under `/context` or `/docs`.
- Read it when designing new contracts or reviewing existing ones.
- Treat it as the “house style guide” for ErgoScript in your project.

---

## 2. context.json

**Type:** JSON (machine-readable)

This file contains the same logical sections as `context.md`, but as structured JSON objects.

**How to use it:**

- Load it from scripts that assist in ErgoScript development.
- Use it in Node.js, Python, Rust, or any environment that prefers structured JSON.
- Build tools that can:
  - show known issues,
  - inject design patterns into templates,
  - or validate simple rules against your contracts.

Example (Node.js):

```js
const fs = require("fs");
const ctx = JSON.parse(fs.readFileSync("./context.json", "utf8"));
console.log(ctx.known_issues);
```

---

## 3. context.yaml

**Type:** YAML (config-style)

Same content as `context.json`, but in YAML. This is helpful for configuration-driven setups.

**How to use it:**

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

---

## 4. context.toml

**Type:** TOML (Rust-friendly configuration)

This file is aimed at Rust-based tooling and services that interact with Ergo or ErgoScript.

**How to use it:**

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

---

## 5. context.loop

**Type:** S-expression / LISP-like

A compact, tree-structured representation of the same information.

**How to use it:**

- For custom parsers or tools that like S-expression input.
- For symbolic or logic-based analysis (e.g., custom static analyzers).
- For experiments where you treat the context as structured, navigable data.

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

---

## 7. embeddings_input.txt

**Type:** Plain text (embedding-ready)

This file is structured as grouped text sections, formatted for bulk ingestion by any system that:

- builds semantic search over your docs,
- computes text embeddings,
- does keyword or similarity analysis.

**How to use it:**

- Feed directly into a search/embedding pipeline.
- Use it to build “search over contract guidelines” in your dev tools or dashboards.

---

## 8. rag_chunks.json

**Type:** JSON (pre-chunked reference sections)

This file breaks the context into small sections (“chunks”), each with:

- `id` — stable identifier (e.g., `core_references`, `known_issues`)
- `title` — human-friendly name
- `tags` — labels for filtering
- `text` — a short description of that topic

**How to use it:**

- Load these chunks into a search index or knowledge store.
- When a tool needs advice (e.g., on token handling or known issues), look up the relevant chunk by ID or tags.
- Use it as a routing table between contract topics and content.

Even if you’re not building a full RAG (Retrieval-Augmented Generation) system, you can still treat this as a “section index” of the context.

---

## 9. context.pdf

**Type:** PDF

A plain PDF rendering of the main context content.

**How to use it:**

- Share with other team members or auditors.
- Attach to documentation packages, audit reports, or specs.
- Print or review offline.

---

## 10. sphinx_docs/index.rst & sphinx_docs/context.rst

**Type:** reStructuredText (Sphinx documentation)

These are the starter files for building a Sphinx-powered documentation site for the context.

- `index.rst` — entry point / table of contents.
- `context.rst` — overview page describing what the context covers.

**How to use them:**

1. Ensure Sphinx is installed:
   ```bash
   pip install sphinx
   ```
2. Run:
   ```bash
   sphinx-build -b html sphinx_docs/ build/
   ```
3. Open `build/index.html` in your browser to see a browsable set of docs based on this package.

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

4. Document your contract decisions so others can review against this context.

### When auditing an existing contract

1. Open **`context.md`** and read the **Known Issues** and **Secure Patterns** sections side by side with the contract.

2. Check:
   - Are all registers (especially R4–R9) used as intended?
   - Are tokens preserved where they must be?
   - Are HEIGHT and time-based conditions used safely?
   - Are all paths protected by proper signatures or proofs?
   - Does every state transition obey the eUTXO expectations described here?

3. Use **`context.json`**, **`.yaml`**, or **`.toml`** if you are writing automated checks or static analysis tools to assist in the audit.

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

By keeping all of these files together, your ErgoScript projects always have:

- A single source of truth for how contracts **should** be designed.
- Multiple formats so that humans, scripts, tools, and services can all consume the same knowledge.
- A consistent way to reason about security, correctness, and maintainability across every contract you write or review.
