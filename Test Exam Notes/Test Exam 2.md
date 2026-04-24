# Notes on Test Exam Questions (Second)

## Tool Naming & Disambiguation

* Semantic overlap: Rename tools to eliminate ambiguity (e.g., extract_web_results instead of generic names)
* Tool descriptions are primary: LLMs rely heavily on tool descriptions for selection — this is the first diagnostic area when wrong tools are chosen
* Expand descriptions: Include input formats, example queries, edge cases, and boundaries to help LLMs distinguish similar tools

### Structured Data from Agents

* Reduce token bloat: Have upstream agents return structured data (key facts, citations, relevance scores) instead of verbose reasoning and raw content
* Preserves essential info: Cuts volume at the source while maintaining what matters for synthesis

## CLI Flags for Structured Output

--output-format json + --json-schema: Enforces structured output at CLI level — guarantees parseable JSON with required fields
Enables automation: Structured output can be reliably parsed and posted programmatically (e.g., GitHub API inline comments)

## Consistency in Automated Reviews

* Explicit severity criteria: Include concrete code examples for each severity level in prompts to eliminate ambiguity
* Reduces trust erosion: Predictable, consistent ratings help teams know what requires immediate attention
* Disable high-noise categories: Temporarily turn off categories with high false positives (style, naming, docs) to rebuild trust while improving prompts

## Precise Criteria vs. Vague Instructions

Bad: "Check that comments are accurate and up-to-date"
Good: "Flag comments only when claimed behavior contradicts actual code behavior"
Effect: Eliminates false positives on acceptable patterns and false negatives on real issues

## Batch API Limitation

No mid-request tool execution: Asynchronous fire-and-forget model prevents iterative tool-calling workflows — can't intercept, execute, and return results for continued analysis

> Investigate other BATH API Limitations 


## Few-Shot Prompting for Tool Selection

* Target ambiguous scenarios: Use 4-6 examples focused on edge cases where wrong tool selection happens
* Include reasoning: Show why one tool was chosen over plausible alternatives — teaches comparative decision-making
* For multi-concern requests: Add examples demonstrating correct decomposition and tool sequencing when a single message requires multiple tools
* Most effective when: The agent already handles individual cases well but struggles with ambiguity or compound requests

   
## Wrong!!!

## Question 1: Few-Shot Prompting for Consistent Output Format

**Context**: Agent produces variable output formats despite instructions
### Key Points:

- **Few-shot examples are the most effective technique** for achieving consistent output formatting when abstract instructions fail
- **Provide 3-4 concrete examples** showing the exact desired format (issue, location, specific fix)
- **Pattern recognition > abstract rules**: LLMs learn better from concrete patterns than from descriptions of what to do

### Why This Works:

- Gives the model a **template to imitate** rather than interpret
- More reliable than declarative instructions like "format output as X"
- Directly shows what "consistent" looks like in practice

### Wrong Answer Likely Was:

- Adding more detailed instructions (doesn't help if model already struggles with abstract guidance)
- Schema validation (catches problems after generation, doesn't prevent them)

**Category**: Claude Code for CI/CD — Few-Shot Prompting

## Question 2: Skills vs CLAUDE.md Scope

**Context**: When to use Skills vs CLAUDE.md for different coding standards and workflows

### Key Points:

- **CLAUDE.md = Always-on, universal context**
    - Content loaded for **every conversation**
    - Ideal for: coding standards, testing conventions, architectural principles
    - Always applied without needing explicit invocation
- **Skills = On-demand, task-specific workflows**
    - Invoked when Claude detects **trigger keywords**
    - Ideal for: PR reviews, deployments, migrations, specific task workflows
    - Activated only when relevant

### Decision Framework:

- **Use CLAUDE.md for**: Standards that should guide all code generation
- **Use Skills for**: Specialized workflows that don't apply to every interaction

### Why This Matters:

- Prevents context bloat (Skills only load when needed)
- Ensures consistent application of core standards (CLAUDE.md always present)
- Right tool for right scope

**Category**: Code Generation with Claude Code — Skills vs CLAUDE.md Scope


## Question 3: Self-Evaluation Pattern (Evaluator-Optimizer)

**Context**: Inconsistent explanation completeness in customer support responses

### Key Points:

- **Self-critique step = Evaluator-Optimizer pattern**
- Agent evaluates its **own draft** against specific criteria before presenting
- Criteria examples: policy context, timelines, next steps, completeness checklist

### Why This is Most Effective:

- **Catches case-specific gaps** that vary across different scenarios
- Works without human review (automated quality gate)
- Addresses **root cause**: variable completeness across complex, unique cases
- Prevents issues at generation time rather than catching them later

### Pattern Structure:

1. Generate draft response
2. Self-evaluate against criteria
3. Revise if gaps found
4. Present final response

### Wrong Answer Likely Was:

- Adding more examples (doesn't help with case-by-case variation)
- Human review step (not scalable, defeats automation purpose)

**Category**: Customer Support Agent — Self-Evaluation Patterns

## Question 4: Multi-Step Workflow Orchestration

**Context**: Agent makes redundant API calls when handling multi-concern customer requests

### Key Points:

- **Decompose request into distinct concerns** (billing + shipping + returns)
- **Investigate in parallel** with **shared customer context**
- Single customer context fetch → multiple parallel investigations → unified synthesis

### Why This is Most Effective:

- **Eliminates redundant data fetching**: Reuses context across all concerns
- **Reduces total tool calls**: Parallelization instead of sequential processing
- **Addresses both core issues**: redundancy AND efficiency

### Workflow Structure:

```
1. Fetch customer context (once)
2. Identify distinct concerns
3. Parallel tool calls (each uses shared context)
4. Synthesize unified resolution
```

### Wrong Answer Likely Was:

- Sequential processing (still redundant, slower)
- Caching (helps but doesn't address orchestration inefficiency)
- Single tool that handles everything (not always possible/flexible)

**Category**: Customer Support Agent — Multi-Step Workflow Orchestration

## Question 5: Ambiguous Result Handling

**Context**: Multiple customer matches cause 15% error rate in customer identification

### Key Points:

- **Ask user for additional identifier** when multiple matches occur
- Additional identifiers: email, phone, order number, account ID
- **User has definitive knowledge** of their own identity

### Why This is Most Effective:

- **Eliminates guessing**: No heuristics or probability-based selection
- **One extra conversational turn** is minimal cost vs. 15% error rate
- Most reliable disambiguation method
- User can provide exact identifier that maps to one record

### Wrong Answers Likely Were:

- Using heuristics (recent activity, account age) — still guessing, maintains errors
- Presenting all matches — confusing UX, requires user to understand system IDs
- Defaulting to most recent — arbitrary, maintains error rate

### Principle:

**When ambiguity exists, clarify with the authoritative source (the user) rather than guess**

**Category**: Customer Support Agent — Ambiguous Result Handling


## Summary of Common Themes in Wrong Answers:

1. **Few-shot > instructions** for format consistency
2. **Right scope for right context** (CLAUDE.md vs Skills)
3. **Self-evaluation prevents quality gaps** before they reach users
4. **Parallel + shared context = efficiency** in multi-step workflows
5. **Clarify with authority (user) > guess** when data is ambiguous

These all reflect **proactive problem prevention** rather than reactive fixes.