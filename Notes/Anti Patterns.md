# Claude Certified Architect — Anti-Patterns Reference

> Every anti-pattern below is something the exam guide explicitly warns against,
> either in the task statements, sample questions, or preparation exercises.
> The exam distractors are built from these — knowing them is knowing what NOT to pick.

---

## Domain 1: Agentic Architecture & Orchestration

### Agentic Loop Anti-Patterns
- **Parsing natural language to detect loop completion** — e.g., checking if Claude's response contains "DONE" or "I'm finished." Use `stop_reason` instead.
- **Setting arbitrary iteration caps as the primary stopping mechanism** — iteration limits are safety nets, not flow control. The loop should exit on `stop_reason: "end_turn"`.
- **Checking for assistant text content as a completion indicator** — Claude can emit text AND tool calls in the same turn. Presence of text ≠ task complete.
- **Not sending full conversation history on each API call** — the Messages API is stateless. Omitting prior turns causes Claude to lose context.
- **Skipping the assistant's `tool_use` turn in the message array** — role alternation (user/assistant/user) must be preserved. You can't send a `tool_result` without the preceding `tool_use` turn.

### Multi-Agent Anti-Patterns
- **Assuming subagents inherit the coordinator's context** — they don't. Context must be explicitly passed in the Task tool's prompt string.
- **Overly narrow task decomposition** — the coordinator splits a broad topic into too-specific subtasks, missing entire domains (e.g., only visual arts for "creative industries").
- **Always routing through the full pipeline** — the coordinator should dynamically select which subagents to invoke based on query complexity, not blindly run all of them.
- **Subagents communicating directly with each other** — all inter-subagent communication should route through the coordinator for observability and error handling.
- **Giving step-by-step procedural instructions to the coordinator** — specify research goals and quality criteria instead, to enable subagent adaptability.

### Workflow Enforcement Anti-Patterns
- **Relying on prompt instructions alone for mandatory business rules** — prompts have a non-zero failure rate. Use hooks/programmatic prerequisites for deterministic compliance.
- **No structured handoff when escalating to humans** — the human agent lacks access to the conversation transcript. Compile a structured summary (customer ID, root cause, recommended action).

---

## Domain 2: Tool Design & MCP Integration

### Tool Description Anti-Patterns
- **Minimal or missing tool descriptions** — descriptions are the primary mechanism Claude uses for tool selection. Minimal descriptions → unreliable selection among similar tools.
- **Overlapping tool descriptions** — near-identical descriptions for different tools (e.g., `analyze_content` vs `analyze_document`) cause misrouting. Rename and differentiate.
- **System prompt keywords overriding tool descriptions** — keyword-sensitive instructions in the system prompt can create unintended tool associations that bypass well-written descriptions.
- **One generic tool instead of purpose-specific tools** — a single `analyze_document` tool is harder for Claude to use correctly than three specific tools: `extract_data_points`, `summarize_content`, `verify_claim_against_source`.

### Error Handling Anti-Patterns
- **Uniform error responses** — returning generic "Operation failed" for all errors prevents the agent from making appropriate recovery decisions. Return structured metadata (error category, retryable boolean, description).
- **Silently suppressing errors** — returning empty results marked as success when a tool actually failed. Hides critical information from the coordinator.
- **Terminating the entire workflow on a single failure** — one subagent's timeout shouldn't kill the whole research pipeline. Implement local recovery; propagate only unrecoverable errors.
- **Not distinguishing error types** — treating transient errors (timeouts), validation errors (bad input), business errors (policy violations), and permission errors identically. Each requires a different recovery strategy.
- **Wasting retries on non-retryable errors** — without a `isRetryable` flag, the agent may retry business rule violations or permission denials endlessly.

### Tool Distribution Anti-Patterns
- **Giving an agent too many tools** — 18 tools degrades selection reliability. Scope to 4-5 per agent role.
- **Giving agents tools outside their specialization** — a synthesis agent with web search tools will attempt searches instead of synthesizing. Only the search agent should have search tools.
- **Building custom MCP servers for standard integrations** — use existing community MCP servers (e.g., for Jira) before writing custom ones. Reserve custom servers for team-specific workflows.

---

## Domain 3: Claude Code Configuration & Workflows

### CLAUDE.md Anti-Patterns
- **Putting everything in CLAUDE.md** — it's always-loaded, consuming tokens on every request. Use skills (on-demand) and path-specific rules (conditional) for content that isn't universally needed.
- **Using subdirectory CLAUDE.md for conventions spanning many directories** — conventions that apply to files spread across the codebase (e.g., all test files) belong in `.claude/rules/` with glob patterns, not directory-specific CLAUDE.md files.

### CI/CD Anti-Patterns
- **Using the same Claude session to generate AND review code** — session context isolation matters. The same session that wrote the code is less effective at reviewing its own changes. Use an independent review instance.
- **Running Claude Code in CI without the `-p` flag** — without `--print` mode, Claude Code waits for interactive input, hanging the pipeline.
- **Not including prior review findings when re-running reviews** — leads to duplicate comments on issues that were already flagged. Include prior findings and instruct Claude to report only new or still-unaddressed issues.
- **Test generation without providing existing test files** — Claude may suggest duplicate test scenarios already covered by the suite. Provide existing tests in context.

### Plan Mode Anti-Patterns
- **Using plan mode for simple, well-scoped changes** — plan mode adds overhead. A single-file bug fix with a clear stack trace should use direct execution.
- **Using direct execution for architectural decisions** — complex tasks with multiple valid approaches, multi-file modifications, or infrastructure tradeoffs need plan mode first.

### Skills Anti-Patterns
- **Not using `context: fork` for verbose skills** — skills that produce large discovery output (codebase analysis, brainstorming) pollute the main session's context. Fork isolates them.
- **Not restricting tool access in skills** — a migration skill that only needs file write access shouldn't have Bash permissions. Use `allowed-tools` in frontmatter.

---

## Domain 4: Prompt Engineering & Structured Output

### Prompt Precision Anti-Patterns
- **Vague instructions for precision** — "be conservative," "only report high-confidence findings," or "check that comments are accurate" do NOT improve precision. Define specific categorical criteria instead.
- **Confidence-based filtering** — telling the model to filter by its own confidence level is unreliable. Define which issue categories to report vs skip.
- **Leaving high false-positive categories enabled** — they undermine developer trust in ALL categories, including accurate ones. Temporarily disable them while improving prompts.

### Few-Shot Anti-Patterns
- **Adding more detailed instructions when output is inconsistent** — if instructions already exist and output varies, the fix is examples, not longer instructions.
- **Too many examples of the same pattern** — over-specifying risks pattern-matching to superficial features. Use 2-4 diverse examples that teach the underlying principle.
- **Few-shot examples without reasoning** — for ambiguous cases, showing only input → output is insufficient. Include why one action was chosen over the plausible alternative.
- **Not covering varied document structures** — examples only from one format type (e.g., only academic papers) cause failures on other formats (e.g., news articles). Include both.

### Structured Output Anti-Patterns
- **Assuming JSON schema via tool_use prevents all errors** — it eliminates syntax errors only. Semantic errors (fabricated values, wrong field placement, totals that don't sum) still require validation.
- **Using `tool_choice: "auto"` when structured output is required** — `"auto"` allows the model to return text instead of a tool call. Use `"any"` or forced tool selection.
- **Making all schema fields required** — when source documents may not contain certain information, required fields cause the model to fabricate values. Use optional/nullable fields.
- **Not adding `"unclear"` or `"other"` enum values** — if the real-world data doesn't fit the predefined categories, Claude is forced to pick the wrong one. Add extensibility options.

### Validation Anti-Patterns
- **Retrying when information is absent from the source** — retries work for format/structural errors the model can self-correct. If the data simply doesn't exist in the document, retries just waste tokens.
- **Not including the specific validation error in the retry prompt** — a generic "try again" retry is far less effective than appending the exact error (e.g., "line items sum to $847 but stated_total is $900").
- **Not distinguishing semantic errors from schema errors** — schema errors are eliminated by `tool_use`. Semantic errors (values in wrong fields, inconsistent totals) require different solutions: validation loops, self-correction flows.

### Batch Processing Anti-Patterns
- **Using Message Batches API for blocking workflows** — 50% cost savings but up to 24-hour processing with no latency SLA. Unsuitable for pre-merge checks where developers wait.
- **Expecting multi-turn tool calling in batch mode** — the Batches API doesn't support it. Each batch item is a single request/response.
- **Not using `custom_id` for result correlation** — without it, you can't match batch responses to their original requests.

---

## Domain 5: Context Management & Reliability

### Context Preservation Anti-Patterns
- **Progressive summarization of critical values** — summarizing "the customer paid $847.50 on March 15" into "the customer made a recent purchase" loses actionable data. Extract specific facts into a persistent "case facts" block.
- **Placing critical information in the middle of long inputs** — "lost in the middle" effect: models attend best to beginning and end. Place key findings at the start and organize with explicit section headers.
- **Letting verbose tool outputs accumulate** — a 40-field order lookup when only 5 fields are relevant wastes tokens across all subsequent turns. Trim before accumulating.
- **Not passing complete conversation history** — the API is stateless. Omitting turns causes coherence breakdown.

### Escalation Anti-Patterns
- **Sentiment-based escalation** — customer frustration doesn't reliably indicate case complexity or need for human intervention. Use explicit criteria instead.
- **Self-reported confidence scores as escalation triggers** — the model's confidence in its own assessment is an unreliable proxy for actual difficulty.
- **Investigating before honoring explicit human requests** — when a customer says "I want to speak to a person," don't attempt resolution first. Escalate immediately.
- **Not escalating when policy is ambiguous** — if the customer's request falls in a gap the policy doesn't address (e.g., competitor price matching when policy only covers own-site adjustments), escalate rather than guess.
- **Heuristic selection with multiple customer matches** — when a lookup returns multiple results, don't pick the most likely one. Ask for additional identifiers.

### Error Propagation Anti-Patterns
- **Generic error statuses** — "search unavailable" hides whether there were partial results, what was attempted, and what alternatives exist. Return structured error context.
- **Silently suppressing subagent errors** — returning empty results as success when the subagent actually failed. The coordinator can't recover from what it doesn't know about.
- **Terminating the entire workflow on a single subagent failure** — other subagents may have produced useful partial results. Implement local recovery and propagate only unresolvable errors with context.
- **Not distinguishing access failures from valid empty results** — a search timeout (access failure → retry) is fundamentally different from a search that succeeded but found nothing (valid empty → don't retry).

### Codebase Exploration Anti-Patterns
- **Extended single-session exploration without compaction** — context degrades; the model starts referencing "typical patterns" instead of specific code found earlier. Use `/compact` or delegate to subagents.
- **Not using scratchpad files** — key findings discovered during exploration are lost when context fills up. Write them to persistent files and reference them later.
- **Reading all files upfront** — don't dump the entire codebase into context. Build understanding incrementally: Grep for entry points → Read specific files → trace imports.
- **Resuming sessions with stale tool results** — if code was modified since the last session, prior tool results are outdated. Start fresh with a structured summary or inform the session about specific changes.

### Human Review Anti-Patterns
- **Trusting aggregate accuracy metrics** — 97% overall accuracy may mask 60% accuracy on a specific document type or field. Validate per segment.
- **Automating before validating by document type and field** — consistent performance across all segments must be verified before reducing human review.
- **Not calibrating confidence scores** — field-level confidence scores are only useful after calibrating against labeled validation sets. Raw scores are not meaningful thresholds.

### Provenance Anti-Patterns
- **Losing source attribution during summarization** — when findings are compressed without claim-source mappings, downstream agents can't verify or cite.
- **Arbitrarily selecting one value from conflicting sources** — two credible sources reporting different statistics should both be preserved with source attribution and annotated as conflicting.
- **Not requiring temporal metadata** — without publication/collection dates, temporal differences between sources can be misinterpreted as contradictions.
- **Not annotating coverage gaps** — the final output should indicate which topic areas are well-supported by evidence and which have gaps due to unavailable sources.
