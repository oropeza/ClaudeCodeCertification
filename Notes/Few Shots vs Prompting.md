# CCA-F – Few-Shot vs Prompting Strategies

> **Core principle:** Concrete examples beat abstract instructions.  
> The exam tests *when* to use each strategy — not just that you know them.


| Topic                                                                                                             | Strategy                                                      |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| PR categorize critical severity<br>**Classification**                                                             | Criteria in prompt + examples                                 |
| Flag commits + criteria contradict actual behavior<br>*It's about behavior, not classification*                   | Specify explicit criteria                                     |
| Multiple tasks have issues, but the individual work<br>Pattern guidance                                           | Few shots                                                     |
| Ambiguous scenarios<br>Tool selection                                                                             | 4-6 examples + reasoning<br>Examples + reasons > decide rules |
| Existing detailed instructions + incorrect output<br>Pattern for output, more reliable than abstract instructions | Few shots                                                     |
| Case gaps complexity<br>Complexity                                                                                | Evaluator – Optimizert paths                                  |
| Multipart request, many tool call<br>Complexity                                                                   | Decompose concern and solve parallel                          |
| Multiple matches                                                                                                  | Additional identifier, no session                             |

---

## Decision Table

| Situation                                               | Strategy                                                     | Why                                                      |
| ------------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| Ambiguous scenarios with multiple valid interpretations | **Few-shot** (4–6 examples + reasoning)                      | Anchors behavior better than rules                       |
| Flag/classify with contradictory signals                | **Few-shot**                                                 | Shows the model what "correct" looks like under conflict |
| Multiple concerns in individual work                    | **Few-shot** (pattern guidance)                              | More reliable than abstract pattern descriptions         |
| Tool selection disambiguation                           | **Tool description** (input formats, edge cases, boundaries) | LLM uses description as primary signal — not examples    |
| Existing detailed instructions + wrong output           | **Few-shot**                                                 | Examples override ambiguous instruction interpretation   |
| Structured data extraction (contracts, invoices)        | **`tool_use` + JSON schema**                                 | Guarantees format; minimizes hallucination               |
| Output format must be strict JSON                       | **`tool_use` + schema** (not few-shot)                       | Schema enforces structure programmatically               |
| Explicit review criteria (PR severity, code quality)    | **Explicit criteria in prompt**                              | Few-shot alone won't define thresholds                   |
| Complex multi-step task                                 | **Task decomposition + subagents**                           | Breaks cognitive load; isolates context                  |
| Incorrect output after retry                            | **Validation-retry loop** (model sees its own error)         | Self-correction without human intervention               |
| Multi-pass review needed                                | **Multi-pass architecture**                                  | Separate passes for different concerns                   |
| Long-context inputs                                     | **Key facts block at top of context**                        | "Lost in the middle" effect is real                      |
| Code generation with verification                       | **TDD iteration** (failing test → implement → verify)        | Specific criteria beat vague instructions                |
|                                                         |                                                              |                                                          |

---

## Few-Shot: When ✅ vs When ❌

### ✅ Use Few-Shot When
- Behavior is **ambiguous** and rules alone don't resolve it
- Output **format or tone** needs to be shown, not described
- Criteria **contradict** or overlap (classification edge cases)
- Instructions are abstract and output keeps drifting
- **4–6 examples** can cover the decision space

### ❌ Don't Use Few-Shot When
- The root cause is **poor tool descriptions** → fix descriptions first (D2)
- You need **guaranteed structure** → use `tool_use` + JSON schema (D4.3)
- Task needs **explicit numeric criteria** (e.g., refund limits) → use hooks or schema constraints
- You have **18+ tools and wrong selection** → distribute tools to subagents (D2.3)
- Few-shot adds token overhead without fixing the actual issue

---

## Key Exam Patterns (from official scenarios)

### PR Categorization / Critical Severity
- ✅ Criteria in prompt + examples
- ❌ Classification without examples → model guesses

### Tool Selection Errors
- ✅ Expand tool descriptions (input formats, edge cases, disambiguation)
- ❌ Few-shot examples for tool routing → adds overhead, doesn't fix root cause

### Refund / Policy Enforcement
- ✅ `PostToolUse` hook to intercept and escalate
- ❌ "Do not process refunds over $500" in system prompt

### Multipart Requests
- ✅ Decompose concerns → solve in parallel (subagents)
- ❌ Treat as monolithic single-agent task

### Multiple Models / Agents
- ✅ Additional identifier per model, no shared session
- ❌ Shared global state or full conversation history across agents

### Incorrect Output Despite Instructions
- ✅ Few-shot (concrete beats abstract)
- ❌ More verbose abstract instructions

---

## Structured Output Hierarchy (D4.3)

From most to least guaranteed:

1. `tool_use` + JSON schema — guaranteed structure, preferred for extraction
2. Few-shot examples — format guidance, flexible but not enforced
3. XML tags — lightweight structuring, useful for sections
4. Natural language only — use when format doesn't matter

---

## Related Domains

- [[CCA-F – Domain 1 – Agentic Architecture]] (27%)
- [[CCA-F – Domain 2 – Tool Design & MCP]] (18%)
- [[CCA-F – Domain 3 – Claude Code]] (20%)
- [[CCA-F – Domain 5 – Context & Reliability]] (15%)