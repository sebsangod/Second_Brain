---
aliases:
  - Context Window
tags:
  - learning
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code 101]([https://anthropic.skilljar.com/claude-code-101](https://academy.claude.com/es/courses/claude-code-101/context-management))

**Related:** [[Large Language Model]], [[Claude]], [[Claude Code]], [[Agent]], [[CLAUDE File]], [[Prompt]], [[Development/AI/LLMs/MCP/MCP|MCP]],[[Skills]], [[Subagents]]

---

## Description

Think of the _context window_ as **the amount of space an** ``AI Agent`` (like Claude) **can hold in its memory**. Every time you enter a prompt, Claude reads a file, makes a tool call, or receives a result from a tool call—all of that adds to the _context window_. **Since there is a finite amount of space, it’s important to optimize how you use it**.

![[context_window.webp]]

---

## Details

### What Happens When the Context Fills

When you approach the limit, **the** _context window_ **automatically collapses**. **Collapsing the window summarizes important details and removes unnecessary results from tool calls to free up space**. Keep in mind that **this process may potentially result in the loss of details**.

### Commands

You can run the **compaction manually** using the _/compact_ command. This **compacts everything up to that point**. It's useful when you want to free up context space while retaining a record of what you've worked on previously.

If you want to start completely from scratch without any history from the previous session, run _/clear_. **This deletes everything**.

![[context window-1791779610850.webp]]

#### When to Use Which

A general rule:

Use _/compact_ **when you're working on a specific task and are nearing the context limit but need to continue**. It's important to keep the context relevant to your current task.

Use _/clear_ **when you want to start a new task**. You don’t want the previous conversation to bias the new one.

> [!TIP]
For things you want ``Claude`` to remember between sessions, put them in your ``CLAUDE.md`` file so it doesn’t have to rediscover them from scratch.

### Tips for Saving Context Space
**Be specific.** A vague prompt may seem shorter, but it actually takes up more context in the long run. Without clear instructions, Claude is forced to explore your codebase further and come up with its own reasoning—which takes up much more context space than a detailed ``prompt``.

**Manage your** ``MCP`` **servers**. ``MCP`` servers load all their available tools into the context by default, even when you’re not using them. If you have servers configured for tasks unrelated to the current project, consider disabling them. You can also try ``Skills``, which work similarly to ``MCP`` servers but don’t load everything into the context upfront.

**Use** ``subagents``. ``Subagents`` run in parallel with your main agent but have a completely separate _context window_. For tasks where you only need the answer—such as “Where are the authentication endpoints located?”—a subagent does the work and returns only a summary to your main agent, keeping your main context clean.

---

## Claude Sessions
