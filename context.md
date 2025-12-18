# ErgoScript Contract Development Context File
# Purpose: Provide LLMs/Agents with a compact, high-signal reference layer
# Style: JSON/LOOP-like section structure for fast parsing and contextual grounding

--------------------------------------------------------------------------------
## 1. CORE_REFERENCES
{
  "LangSpec": "https://github.com/ergoplatform/sigmastate-interpreter/blob/develop/docs/LangSpec.md",
  "LangSpec_local": "./local_files/LangSpec.md",
  "ErgoScript_PDF": "https://storage.googleapis.com/ergo-cms-media/docs/ErgoScript.pdf",
  "ErgoScript_PDF_local": "./local_files/ErgoScript.pdf",
  "AdvancedErgoScript_Tutorial": "https://storage.googleapis.com/ergo-cms-media/docs/AdvancedErgoScriptTutorial.pdf",
  "AdvancedErgoScript_Tutorial_local": "./local_files/AdvancedErgoScriptTutorial.pdf",
  "ErgoScript_By_Example": "https://github.com/ergoplatform/ergoscript-by-example",
  "ErgoScript_By_Example_local": "./local_files/ergoscript-by-example-main",
  "Whitepaper": "https://storage.googleapis.com/ergo-cms-media/docs/ErgoScript.pdf",
  "Whitepaper_local": "./local_files/ErgoScript.pdf"
}
--------------------------------------------------------------------------------

## 2. AGENT_REFERENCE
{
  "Ergo_Agent_PR_2242": "https://github.com/ergoplatform/ergo/pull/2242/files",
  "Usage": "Defines how off-chain agents interact with on-chain scripts, state transitions, and node evaluation.",
  "Agent_Guidelines": [
    "Agents compose transactions based on script constraints.",
    "Agents maintain protocol invariants defined by contracts.",
    "Agents ensure safe state transitions according to LangSpec."
  ]
}
--------------------------------------------------------------------------------

## 3. EUTXO_MODEL
{
  "Definition": "Ergo uses the extended UTXO model where each box contains value, tokens, registers, and a guarding script.",
  "Key_Concepts": [
    "Box = state + value + tokens + registers",
    "ErgoScript = predicate that must evaluate TRUE to spend",
    "State transition = old boxes spent → new boxes created",
    "Registers R4–R9 carry contract state",
    "Data Inputs provide read-only state"
  ],
  "Design_Guide": [
    "Ensure deterministic state transitions",
    "Verify all tokens / ERG accounted for in OUTPUTS",
    "Use registers for versioning, commitments, and state"
  ]
}
--------------------------------------------------------------------------------

## 4. KNOWN_ERGOSCRIPT_ISSUES
{
  "Categories": [
    "Unvalidated data inputs",
    "Token loss / unpreserved tokens",
    "OR-branch bypass paths",
    "Missing signature requirements",
    "Point-at-infinity / Sigma-protocol edge cases",
    "Missing commit-reveal (front-running)",
    "Incorrect HEIGHT comparisons",
    "Cyclic script hash dependencies",
    "Scripts too complex hitting cost limit"
  ],
  "Reference_Details": "Use this section to check all contract logic against historical & theoretical flaws."
}
--------------------------------------------------------------------------------

## 5. SECURE_PATTERNS_INDEX
{
  "CommitReveal": "Use hashing commitment in stage1, reveal in stage2",
  "PerpetualToken": "Enforce token preservation via OUTPUTS.exists(...) + script hash match",
  "MultiStage_Workflow": "Validate next state via OUTPUT.propositionBytes.blake2b256 == expected_hash",
  "Multisig": "Use sigmaProp(pkA && pkB) or atLeast(n, keys)",
  "EmergencyRefund": "Use HEIGHT > timeout && ownerKey",
  "StateContinuation": "Require next box to preserve registers and tokens",
  "OracleAuth": "Require oracle NFT or expected box ID"
}
--------------------------------------------------------------------------------

## 6. RESOURCE_PATHS
{
  "Syntax_and_Semantics": "LangSpec.md",
  "Syntax_and_Semantics_local": "./local_files/LangSpec.md",
  "eUTXO_Basics": "ErgoScript_PDF",
  "eUTXO_Basics_local": "./local_files/ErgoScript.pdf",
  "Examples": "ergoscript-by-example",
  "Examples_local": "./local_files/ergoscript-by-example-main",
  "Advanced_Patterns": "AdvancedErgoScriptTutorial.pdf",
  "Advanced_Patterns_local": "./local_files/AdvancedErgoScriptTutorial.pdf",
  "Audit_Examples": "./audit_examples",
  "Audit_Examples_GluonW_AI_Audit_Review_PDF": "./audit_examples/Gluon W ErgoScript Contract AI Audit Review - Proton Docs.pdf",
  "Audit_Guide": "KNOWN_ERGOSCRIPT_ISSUES + SECURE_PATTERNS_INDEX"
}
--------------------------------------------------------------------------------

## 7. AGENT_BEHAVIOR_RULES
{
  "When_Designing_Contracts": [
    "Reference LangSpec for syntax & typing",
    "Use eUTXO model constraints to structure state",
    "Apply secure patterns from SECURE_PATTERNS_INDEX",
    "Check against KNOWN_ERGOSCRIPT_ISSUES"
  ],
  "When_Auditing_Contracts": [
    "Verify all spend paths",
    "Check invariants across state transitions",
    "Verify token & ERG conservation",
    "Validate signature gating and authentication",
    "Ensure predictable and correct HEIGHT usage",
    "Detect bypass via OR branches"
  ]
}
--------------------------------------------------------------------------------

## 8. LLM_USAGE_NOTES
{
  "Goal": "Provide safe, correct, secure ErgoScript development & auditing.",
  "Instruction": "Use this file as the primary index. All contextual lookup should be routed through CORE_REFERENCES, EUTXO_MODEL, SECURE_PATTERNS_INDEX, KNOWN_ERGOSCRIPT_ISSUES, and AUDIT_EXAMPLES."
}
--------------------------------------------------------------------------------

## 9. AUDIT_EXAMPLES (LLM output reviewed by human)
{
  "Folder": "./audit_examples",
  "Files": [
    {
      "Title": "Gluon W ErgoScript Contract AI Audit Review (Proton Docs PDF)",
      "Path": "./audit_examples/Gluon W ErgoScript Contract AI Audit Review - Proton Docs.pdf"
    }
  ],
  "Purpose": "Use these as calibration examples for where an LLM audit can be correct, incorrect, or partially correct, and what kinds of details it commonly misses.",
  "Observed_LLM_Pitfalls_From_Example": [
    "Missing that a token id check exists when it is referenced indirectly (e.g., inside a tuple like oracleBoxPoolNFT).",
    "Over-recommending extra identity checks (e.g., ergoTree bytes) when a singleton NFT already uniquely identifies the box in common patterns.",
    "Producing valid findings but missing contextual cancellation/invariants (e.g., _MinFee cancels out when included on both input and output sides; singleton token amount is known to be 1 for a referenced box).",
    "Not accounting for type constraints in ErgoScript (e.g., Coll[] indexing requires Int; a Long->Int downcast may be unavoidable).",
    "Flagging theoretical numeric truncation issues without considering fixed bounds/constants (e.g., BUCKETS = 14).",
    "Missing micro-optimizations and best-practice tradeoffs (e.g., && can be smaller than allOf(); getVar is off-chain supplied and may be intentionally flexible for multiple UIs)."
  ],
  "How_To_Use": [
    "Treat LLM audit findings as hypotheses; verify against LangSpec and the actual on-chain constraints.",
    "When a finding depends on off-chain transaction construction, explicitly ask: can a different backend violate this and still satisfy the script?",
    "When in doubt, cross-check your current audit output against these examples and call out uncertainty."
  ]
}
--------------------------------------------------------------------------------
