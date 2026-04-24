# Coding Assistant

![](../Attachments/Pasted%20image%2020260319234750.png)

1. **Gather context** - Understanding what the error refers to, which part of the codebase is affected, and what files are relevant
2. **Formulate a plan** - Deciding how to solve the issue, such as changing code and running tests to verify the fix
3. **Take action** - Actually implementing the solution by updating files and running commands

1 and 3 require tools

---
Understanding what tools do and using them effectively to complete complex tasks.
## Benefits of Strong Tool Use

- **Tackles harder tasks** - Claude can combine different tools to handle complex work and will use tools it hasn't seen before
- **Extensible platform** - You can easily add new tools to Claude Code, and Claude will adapt to use them as your workflow evolves
- **Better security** - Claude Code can navigate codebases without requiring indexing, which often means not sending your entire codebase to external servers

## Key Takeaways

Understanding coding assistants comes down to a few essential points:

- Coding assistants use language models to complete different tasks
- Language models need tools to handle most real-world programming tasks
- Not all language models use tools with the same skill level
- Claude's strong tool use enables better security, customization, and longevity in Claude Code

# Context


/init -> Generates CLAUDE.md

- The project's purpose and architecture
- Important commands and critical files
- Coding patterns and structure

![](../Attachments/Pasted%20image%2020260328110501.png)

```
# Memory management
@ Mention a file
```

## Thinking Modes

Claude offers different levels of reasoning through "thinking" modes. These allow Claude to spend more time reasoning about complex problems before providing solutions.

The available thinking modes include:

- "Think" - Basic reasoning
- "Think more" - Extended reasoning
- "Think a lot" - Comprehensive reasoning
- "Think longer" - Extended time reasoning
- "Ultrathink" - Maximum reasoning capability

Each mode gives Claude progressively more tokens to work with, allowing for deeper analysis of challenging problems.

## Planning vs thinking

**Planning Mode** is best for:

- Tasks requiring broad understanding of your codebase
- Multi-step implementations
- Changes that affect multiple files or components

**Thinking Mode** is best for:

- Complex logic problems
- Debugging difficult issues
- Algorithmic challenges



## Context Management Commands


* Combining Escape with Memories
* Rewinding Conversations (double esc)

### /compact

The `/compact` command summarizes your entire conversation history while preserving the key information Claude has learned. 

### /clear

The `/clear` command completely removes the conversation history, giving you a fresh start. 


## Custom commands

Automate repetitive tasks you run frequently
Custom commands can accept arguments using the `$ARGUMENTS` placeholder. This makes them much more flexible and reusable.

## Key Benefits

- **Automation** - Turn repetitive workflows into single commands
- **Consistency** - Ensure the same steps are followed every time
- **Context** - Provide Claude with specific instructions and conventions for your project
- **Flexibility** - Use arguments to make commands work with different inputs


# MCP Server

You can extend Claude Code's capabilities by adding MCP (Model Context Protocol) servers. These servers run either remotely or locally on your machine and provide Claude with new tools and abilities it wouldn't normally have.

# Github integration

### Mention Action

You can mention Claude in any issue or pull request using `@claude`. When mentioned, Claude will:

- Analyze the request and create a task plan
- Execute the task with full access to your codebase
- Respond with results directly in the issue or PR

### Pull Request Action

Whenever you create a pull request, Claude automatically:

- Reviews the proposed changes
- Analyzes the impact of modifications
- Posts a detailed report on the pull request