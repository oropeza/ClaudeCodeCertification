
# Exam Feedback

**Date:** 2026-04-06

---

## General Observations

- Each exam attempt presents different scenarios — do not expect the same questions.
- The hardest domain for me was **Domain 5: Context Management & Reliability**, particularly:
  - Context degradation over long conversations
  - Partitioning research across context boundaries

---

## Topics That Appeared on the Exam

### MCP Tool Selection
- Questions tested when and how to select the correct MCP tool, and how tool metadata is fetched and used by the model.

### Handling Multi-Question Prompts
- When a prompt asks too many things at once, valid strategies include: answering questions one by one, persisting intermediate state in a "database", or assuming context already defined in the prompt.

### Validation Error Types
- Key distinction: **schema syntax errors** (malformed JSON — eliminated by `tool_use`) vs. **semantic validation errors** (values don't sum correctly, field placed in the wrong location — not caught by schema alone).

### Fixing Semantically Incorrect JSON
- If generated JSON is syntactically valid but semantically wrong, the correct approach is a **follow-up self-correction request** that includes: the original document, the failed extraction, and the specific validation errors — letting the model fix its own output.

### Self-Correction Validation Flows
- Design extraction schemas to include derived fields like `calculated_total` alongside `stated_total` to detect discrepancies, and add `conflict_detected` boolean flags for inconsistent source data.

### Confidence Ratings for Uncertain Values
- When some JSON fields are not reliably generated, one approach is adding a **confidence score** per field so users can identify which values need manual review.

### Section Being Ignored in Prompt Output
- A question explored why one section of a document was consistently ignored while all others were processed correctly — likely a context positioning or prompt structure issue.

### Conversational Refinement (Stateful Adjustments)
- Scenario: a user progressively refines a request ("I want a $10M house" → "I want a $20M house"). The system must correctly track and apply the latest user intent without carrying over stale state.

### Document Extraction with Amendments
- Scenario: extracting data from a contract that contains amendments (e.g., initial budget $10M, later amended to $20M). The extraction must resolve which value is authoritative — the final amendment wins.

---

## Key Takeaway

> **Always prefer the simplest solution that solves the problem.** Avoid over-engineering.
