# Personal Notes

## Basic principles:

- Specificity beats generality
- Structure data at every boundary
- keep contexts isolated
- Programmatic enforcement for Guarantees
- Fix Root Causes
- Domain Knowledge

-

# Agents and context fork

For your CCA-F exam, remember that `context:fork` is about **execution isolation** - running skills in separate subagent contexts to keep the main conversation clean.

The fork returns its final output to the main conversation, but intermediate messages and history inside the fork remain isolated.

**Dynamic data injection**: The `!` syntax runs commands at invocation time 


# API and tool schemas

Include tool schemas in all requests where those tools are referenced

# Subagents

* Clean context, will not receive Claude.md unless explicit
* Subagents will NOT have access to skills unless you explicitly grant them in the `allowedTools` section of their agent config.
* The Task tool as the mechanism for spawning subagents, and **the requirement that allowedTools must include "Task" for a coordinator to invoke subagents**

# Explicit criteria vs FewShoot

> **Explicit criteria** = reduce false positives on objective measures  
> **Few-shot examples** = handle subjective/nuanced patterns

**Use this decision tree:**

```
Is the requirement objective & measurable?
(line counts, thresholds, formats, hard rules)
    ↓
  YES → Explicit Criteria
    ↓
   NO → Is it subjective, nuanced, or context-dependent?
         (style, tone, judgment calls, pattern recognition)
           ↓
         YES → Few-Shot Examples
```

This pattern appears across all domains.
If issue is complex, use another pattern
# Structured output

tool_use with JSON schema definitions guarantees structural compliance — the output will always match the defined schema. However, note that this only guarantees structure, not semantic correctness. System prompts and few-shot examples improve the likelihood of formatting, but don't guarantee it. Post-processing catches errors but doesn't prevent them.

- **Tool use with JSON schemas guarantees structure, not semantics** — API enforces correct types/fields (eliminates syntax errors), but Claude can still extract wrong values or make semantic mistakes (e.g., totals that don't match line items)
- **`tool_choice` controls output format:**
    - `"auto"` = might return text (no guarantee)
    - `"any"` = must use a tool, Claude picks which (guarantees structure)
    - `{"type": "tool", "name": "extract_metadata"}` = forces specific tool (guarantees exact schema)
- **Design schemas defensively:**
    - Mark fields as optional when data might be missing (prevents hallucination)
    - Add enum escape hatches like `"unclear"` or `"other"` + detail fields
    - Include format rules in descriptions ("ISO 8601", "no currency symbols")


# Conflict resolution: 
- Conflicting stats from credible sources → annotate with source attribution, don't pick one


# General Quick Study Points
 
- Implementation of hooks for enforcement (before or after tool usage) 
- System prompts and few-shot examples operate probabilistically 
- Prioritize correcting tool descriptions initially 
- All interactions are managed via the coordinator 
- Maintain structured error context 
- Subagents lack access to the shared context

- Use 'context: fork' when applying skills such as sub-agents
- Utilize .claude/rules/ for rules tied to specific paths
- Employ the Plan mode for handling complex modifications
- Run independent instances for review purposes
- Always include -p to enable non-interactive mode (ideal for CI/CD)
- Specify --output-format json and --json-schema for CI/CD workflows

**Prompt Engineering & Structured Output**

- Implement structured data consistently throughout
- Utilize tool_use alongside JSON Schema
- Set nullable: true to indicate missing information
- Retry operations only when facing resiliency problems
- It is acceptable to temporarily disable problematic categories
- For blocking workflows like pre-commit, always use synchronous API calls
- Reserve Batch API usage for non-blocking tasks such as investigations



- Key summaries should be placed at the beginning rather than lost in the middle.
- Organize data using structured blocks.
- Apply PostToolUse trim when appropriate.
- Consider using scratchpad files as an alternative.
- The /compact option is also available.