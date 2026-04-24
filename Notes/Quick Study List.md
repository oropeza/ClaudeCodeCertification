# Claude Certified Architect — Quick Study List

---

## DOMAIN 1: Agentic Architecture & Orchestration (27%)

### Agentic Loop (1.1)
- Loop runs while `stop_reason == "tool_use"`, exits on `"end_turn"`
- Messages API is stateless — send full conversation history every request
- Tool results go in a **user-role** message as `tool_result` blocks
- `pause_turn` = server-side loop hit its cap, re-send to continue
- Other stop reasons: `max_tokens`, `stop_sequence`, `refusal`

### Multi-Agent Orchestration (1.2)
- Hub-and-spoke: coordinator manages all inter-subagent communication
- Subagents have **isolated context** — no automatic inheritance from parent
- Coordinator does: task decomposition, delegation, result aggregation
- Risk: overly narrow decomposition → incomplete topic coverage

### Subagent Configuration (1.3)
- `Task` tool spawns subagents; `allowedTools` must include `"Task"`
- Context must be **explicitly passed** in the prompt — not inherited
- `AgentDefinition`: description, system prompt, tool restrictions, model
- Parallel subagents: emit multiple Task calls in a single coordinator turn
- Subagents **cannot spawn their own subagents** (no nesting)

### Workflow Enforcement (1.4)
- Hooks/programmatic gates for mandatory rules (deterministic)
- Prompt instructions for guidance (probabilistic — non-zero failure rate)
- Prerequisite gates: block downstream tools until prior steps complete
- Structured handoff summaries for human escalation

### Agent SDK Hooks (1.5)
- `PostToolUse`: normalize data formats before model processes them
- `PreToolUse`: block policy-violating actions, redirect to alternatives
- Hooks run in your process, **not in the context window** (no token cost)
- Hooks vs prompts: hooks = guaranteed compliance; prompts = best-effort

### Task Decomposition (1.6)
- **Prompt chaining**: fixed sequential steps, known in advance, predictable
- **Dynamic decomposition**: model generates subtasks at runtime based on findings
- Can I write the steps before seeing the input? Yes → chaining. No → dynamic.
- Split large code reviews: per-file local passes + cross-file integration pass

### Session Management (1.7)
- `--resume <session-name>` to continue a prior conversation
- `fork_session` for branching into different approaches from shared baseline
- Stale tool results → start fresh with structured summary instead of resuming
- Inform resumed sessions about specific file changes

---

## DOMAIN 2: Tool Design & MCP Integration (18%)

### Tool Descriptions (2.1)
- Descriptions are the **primary mechanism** for tool selection
- Include: input formats, example queries, edge cases, boundaries
- Overlapping descriptions → misrouting (rename/split to differentiate)
- System prompt keywords can create unintended tool associations

### Error Responses (2.2)
- `isError` flag for MCP tool failures
- Error types: transient, validation, business, permission
- Return: `errorCategory`, `isRetryable` boolean, human-readable description
- Distinguish access failures (retry decisions) from valid empty results

### Tool Distribution (2.3)
- Too many tools (18+) degrades selection reliability
- Scoped access: give agents only tools for their role (4-5 ideal)
- Agents with tools outside their specialization tend to misuse them
- `tool_choice`: `"auto"` / `"any"` / forced specific tool

### MCP Resources vs Tools (2.4)
- **Resources**: content catalogs, read-only data (visibility without tool calls)
- **Tools**: actions with side effects
- Use existing community MCP servers before building custom ones
- `.mcp.json` config: project vs user scope, environment variable expansion

### Built-in Tools (2.5)
- **Grep**: search file contents (function names, error messages, imports)
- **Glob**: find files by path pattern (`**/*.test.tsx`)
- **Read/Write**: full file operations; **Edit**: targeted text replacement
- Edit fails on non-unique text → fallback to Read + Write
- Build understanding incrementally: Grep → Read → trace, not read everything

---

## DOMAIN 3: Claude Code Configuration & Workflows (20%)

### CLAUDE.md Hierarchy (3.1)
- **User** (`~/.claude/`) → **Project** (repo root) → **Directory** (subdirectories)
- `@import` for modular organization
- Always-loaded context → keep concise, token-efficient

### Custom Commands & Skills (3.2)
- `.claude/commands/` = slash commands (project-scoped, version controlled)
- `.claude/skills/` with SKILL.md + frontmatter options
- `context: fork` isolates verbose output from main session
- `allowed-tools` restricts tool access during skill execution
- `argument-hint` prompts for required parameters
- Skills = on-demand invocation; CLAUDE.md = always-loaded

### Path-Specific Rules (3.3)
- `.claude/rules/` files with YAML `paths:` glob patterns
- Load only when editing matching files → saves tokens
- Better than subdirectory CLAUDE.md for conventions spanning many directories

### Plan Mode vs Direct Execution (3.4)
- **Plan mode**: complex tasks, multiple approaches, architectural decisions, multi-file
- **Direct execution**: simple, well-scoped, single-file changes

### Iterative Refinement (3.5)
- Interview pattern: surface design considerations before implementing
- Provide specific test cases with input/output for edge case handling
- Multiple interacting issues → single detailed message
- Independent issues → sequential iteration

### CI/CD Integration (3.6)
- `-p` / `--print` flag for non-interactive CI mode
- `--output-format json` + `--json-schema` for structured CI output
- Session context isolation: separate review instance from code generator
- Include prior review findings to avoid duplicate comments
- CLAUDE.md provides testing standards to CI-invoked Claude Code

---

## DOMAIN 4: Prompt Engineering & Structured Output (20%)

### Explicit Criteria (4.1)
- Specific criteria > vague instructions ("flag when X contradicts Y" > "check accuracy")
- "Be conservative" / "only report high-confidence findings" → does NOT improve precision
- High false positive rates undermine trust in ALL categories
- Define which issues to report vs skip by category, not confidence

### Few-Shot Prompting (4.2)
- **First-line fix** when instructions alone produce inconsistent output
- 2–4 targeted examples for ambiguous boundary cases
- Show reasoning: "This looks like X but is actually Y because Z"
- Enables generalization to novel patterns (not just memorizing examples)
- Reduces hallucination in extraction (show null for missing data)
- Wrap in `<example>` / `<examples>` XML tags

### Structured Output (4.3)
- `tool_use` + JSON schema = guaranteed schema compliance, no syntax errors
- **Does NOT prevent semantic errors** (wrong values, fabricated data)
- `tool_choice: "auto"` → model may return text instead
- `tool_choice: "any"` → must call a tool, can choose which
- `tool_choice: {"type": "tool", "name": "X"}` → forced specific tool
- Optional/nullable fields → prevent fabrication when data absent

### Validation & Retry (4.4)
- Retry-with-error-feedback: append specific validation errors on retry
- Retries are **ineffective** when info is absent from source (vs format errors)
- `detected_pattern` field for tracking false positive patterns
- Semantic errors (values don't sum) vs schema syntax errors (eliminated by tool_use)

### Batch Processing (4.5)
- Message Batches API: **50% cost savings**, up to 24-hour processing
- No guaranteed latency SLA → unsuitable for blocking workflows
- `custom_id` for request/response correlation
- No multi-turn tool calling support in batch mode

---

## DOMAIN 5: Context Management & Reliability (15%)

### Context Preservation (5.1)
- Progressive summarization risks: loses numbers, dates, percentages
- **Lost in the middle**: models attend better to beginning and end
- Tool results accumulate tokens disproportionately (40+ fields, only 5 relevant)
- Must send complete conversation history each API request
- Trim verbose tool outputs; extract structured "case facts" block

### Escalation Patterns (5.2)
- Escalate when: customer demands human, policy gaps, no progress
- Honor explicit human requests **immediately** (don't investigate first)
- Sentiment-based escalation and self-reported confidence = **unreliable**
- Multiple customer matches → ask for more identifiers, don't guess

### Error Propagation (5.3)
- Structured error context: failure type, what was attempted, partial results, alternatives
- Access failures (timeout) ≠ valid empty results (query succeeded, no matches)
- Generic error statuses hide context from coordinator
- Anti-patterns: suppress errors silently OR terminate entire workflow on single failure

### Large Codebase Exploration (5.4)
- Context degrades in long sessions → "typical patterns" instead of specific findings
- Use **scratchpad files** to persist findings across context boundaries
- Delegate verbose exploration to subagents; main agent coordinates
- `/compact` to reduce context during extended sessions
- Crash recovery: structured state exports (manifests) loaded on resume

### Human Review & Confidence (5.5)
- 97% aggregate accuracy may mask poor performance on specific document types
- Stratified random sampling for measuring error rates
- Field-level confidence scores calibrated with labeled validation sets
- Validate accuracy **by document type and field** before automating

### Information Provenance (5.6)
- Source attribution lost during summarization → use structured claim-source mappings
- Conflicting stats from credible sources → annotate with source attribution, don't pick one
- Require publication/collection dates to prevent temporal confusion
- Coverage gap annotations: which topics are well-supported vs which have gaps

---

---

# EXAM TRAPS

> These are the counterintuitive answers or common misconceptions the exam exploits.

---

## Domain 1 Traps

**Trap: "The agent sometimes skips a required step. Add stronger prompt instructions."**
Wrong. If compliance must be guaranteed (identity verification before refund), use **hooks/programmatic gates**, not prompts. Prompts have a non-zero failure rate.

**Trap: "The agent terminates when Claude's response contains the word DONE."**
Wrong. This is parsing natural language signals. Use `stop_reason` field — it's the authoritative signal. Text content + tool calls can coexist in the same turn.

**Trap: "Set max iterations to 5 as the primary stopping mechanism."**
Wrong. Arbitrary iteration caps are safety nets, not flow control. The loop should exit because Claude signals `end_turn`.

**Trap: "The final report misses topics. Fix the synthesis agent."**
Wrong. If the coordinator's decomposition was too narrow, the problem is the **coordinator**, not downstream agents that executed correctly within their scope (exam Question 7).

**Trap: "Subagents automatically inherit the coordinator's context."**
Wrong. Subagent context must be **explicitly passed** in the prompt. They start with a fresh conversation.

**Trap: "The task looks complex, so use dynamic decomposition."**
Not necessarily. If the steps are **predictable** regardless of input (e.g., review every PR for security, style, test coverage), use prompt chaining. Dynamic decomposition is for when you can't know the steps until you start.

---

## Domain 2 Traps

**Trap: "Two tools with similar names route incorrectly. Add routing instructions to the system prompt."**
Wrong. **Rename the tools** and update descriptions to eliminate overlap. System prompt keywords can create *more* unintended associations.

**Trap: "Give the agent access to all 18 tools for maximum flexibility."**
Wrong. 18 tools degrades selection reliability. Scope to **4-5 tools** per agent role.

**Trap: "Tool returned no results — it failed."**
Not necessarily. Distinguish **access failures** (timeout, service unavailable → retryable) from **valid empty results** (query succeeded, nothing matched → don't retry).

**Trap: "Return generic error: 'Operation failed'."**
Wrong. Generic errors prevent intelligent recovery. Return structured error metadata: category, retryable boolean, description.

---

## Domain 3 Traps

**Trap: "Test conventions apply to test files across the repo. Use a subdirectory CLAUDE.md."**
Wrong. Test files are spread across many directories. Use **`.claude/rules/`** with glob patterns (`**/*.test.tsx`) for path-specific rules.

**Trap: "Claude Code reviewing its own generated code finds fewer issues."**
Correct observation — but the fix is using a **separate review instance** with fresh context, not the same session.

**Trap: "Always use plan mode for safety."**
Wrong. Plan mode adds overhead for simple tasks. **Direct execution** is correct for well-scoped, single-file changes.

**Trap: "Put everything in CLAUDE.md for easy access."**
Wrong. CLAUDE.md is always-loaded and costs tokens every request. Use **skills** for on-demand workflows and **rules** for conditional loading.

---

## Domain 4 Traps

**Trap: "Output format is inconsistent. Make the instructions more detailed."**
Wrong. If detailed instructions already exist and output is still inconsistent, **add few-shot examples**. Instructions describe; examples show.

**Trap: "'Be conservative' will reduce false positives."**
Wrong. Vague confidence instructions don't improve precision. Define **specific categorical criteria** for what to flag vs skip.

**Trap: "JSON schema via tool_use prevents all extraction errors."**
Wrong. It prevents **syntax** errors only. Semantic errors (fabricated values, wrong field placement, totals that don't add up) still require few-shot examples + validation.

**Trap: "Retry the extraction when the data isn't in the document."**
Wrong. Retries only work for **format/structural errors** the model can self-correct. If the information is absent from the source, retries just waste tokens.

**Trap: "Use tool_choice: 'auto' to guarantee structured output."**
Wrong. `"auto"` lets the model return text instead of calling a tool. Use `"any"` to guarantee a tool call, or force a specific tool.

**Trap: "Required schema fields ensure completeness."**
Wrong. Required fields can cause the model to **fabricate values** to satisfy the schema. Use **optional/nullable fields** when data may be absent.

---

## Domain 5 Traps

**Trap: "97% overall extraction accuracy means the system is ready for production."**
Wrong. Aggregate accuracy can mask poor performance on **specific document types or fields**. Validate accuracy per segment before deploying.

**Trap: "Summarize everything to save context tokens."**
Risky. Progressive summarization **loses specific values**: numbers, dates, percentages, stated expectations. Extract critical facts into a separate persistent block.

**Trap: "Put the most important information in the middle of the prompt."**
Wrong. Models attend best to the **beginning and end** of long inputs (lost-in-the-middle effect). Place key findings at the start.

**Trap: "The customer is upset, escalate immediately."**
Wrong. Sentiment-based escalation is unreliable. Escalate when: customer explicitly requests human, policy is ambiguous, or agent can't make progress. Offer resolution first if the issue is within capability.

**Trap: "Two sources disagree on a statistic. Pick the more recent one."**
Wrong. **Annotate both values with source attribution.** Don't arbitrarily select — conflicting data should be preserved with provenance.

**Trap: "The subagent encountered an error, terminate the whole workflow."**
Wrong. Implement **local recovery** for transient failures. Propagate to coordinator only what can't be resolved locally, including partial results and what was attempted.

**Trap: "Search returned an error → return empty results to the coordinator."**
Wrong. Silently suppressing errors (returning empty results as success) hides critical context. Return **structured error context** so the coordinator can make informed decisions.
