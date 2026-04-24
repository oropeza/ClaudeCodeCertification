# Notes on Test Exam Questions (First)

- **For ambiguous tool selection, add 4–6 worked examples** showing _why_ one tool was chosen over alternatives — reasoning examples beat declarative rules for nuanced decisions.
- **Watch for keyword-sensitive instructions in system prompts** — terms like "account" can unintentionally trigger incorrect tool-selection patterns.
- **Extract key facts (amounts, dates, IDs) into a structured "case facts" block** outside summarized history — summaries are lossy, but a persistent block keeps precise details available across turns.
- **Use `stop_reason` to control your agentic loop:** `"tool_use"` → send tool results back; `"end_turn"` → stop the loop.
- **Use a self-critique step (evaluator-optimizer)** to catch variable, case-specific gaps in responses — few-shot examples only help with predictable, repeatable patterns.
- **Block downstream tools programmatically until a prerequisite completes** (e.g., require a verified customer ID from get_customer) — this guarantees deterministic sequencing, regardless of LLM behavior. 
- **Replace vague instructions with precise criteria** (e.g., "flag only when a comment contradicts actual code") to eliminate false positives and false negatives.
- **Define severity levels with concrete code examples** so the model has clear reference points, making classifications consistent and predictable. *Clasification*
- **Batch API can't support iterative tool-calling** — its fire-and-forget model has no way to intercept mid-request, execute a tool, and return results for continued reasoning.
- **Use 3–4 few-shot examples to lock down output format** — concrete examples of the exact structure you want (issue, location, fix) beat abstract formatting instructions.
- **A coordinator agent sees everything** — it observes all interactions, handles errors consistently, and controls what information each subagent receives.
