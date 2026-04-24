# Subagents

Subagents are specialized assistants that Claude Code can delegate tasks to.

Subagents spin up a separate context window. The subagent receives two things:

- **A custom system prompt** from your configuration file that defines the subagent's role and behavior
- **A task description** written by the parent agent based on what you asked for


## Built-in Subagents

Claude Code ships with several built-in subagents you can use right away:

- **General purpose subagent** -- for multi-step tasks that require both exploration and action
- **Explore** -- for fast searching and navigation of codebases
- **Plan** -- used during plan mode for research and analysis of your codebase before presenting a plan
- Custom Subagent

Benefits:

- They break work into focused pieces, letting each subagent concentrate on a specific task
- They keep your main context window clean by isolating all the intermediate work
- They bring back just the information you need as a concise summary


Scope of your subagent:

- **Project-level** -- available only in the current project
- **User-level** -- shared across all projects on your machine


## The Config File

Once creation is complete, the subagent config file is saved into your project (typically at `.claude/agents/your-agent-name.md`). Here is what a typical subagent config looks like:


```
---
name: code-quality-reviewer
description: Use this agent when you need to review recently written or modified code for quality, security, and best practice compliance.
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch
model: sonnet
color: purple
---

You are an expert code reviewer specializing in quality assurance, security best practices, and
adherence to project standards. Your role is to thoroughly examine recently written or modified code
and identify issues that could impact reliability, security, maintainability, or performance.
```


## Effective subagents

1. **Specific descriptions** -- The description controls when the subagent is launched and what instructions it receives. Write it to steer both.
2. **Structured output** -- Define an output format in the system prompt so the subagent knows when it's done and returns information the main thread can use.
3. **Obstacle reporting** -- Include a section in the output format for workarounds, quirks, and problems so the main thread doesn't have to rediscover them.
4. **Limited tool access** -- Only give a subagent the tools it actually needs. Read-only for research, bash for reviewers, edit/write only for agents that should change code.



### Subagents excel in:

* Research tasks
* Code reviews
* Custom system prompts: Copywriting subagent, Styling subagent

### Subagent hurts:

* Expert claims
* Sequential pipelines
* Test runners
