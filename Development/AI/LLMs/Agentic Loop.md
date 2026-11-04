---
aliases:
  - Agentic Loop
tags:
  - learning
  - dev/ai/llm
date: 2026-11-02
---
**Sources**: [Claude Code 101](https://academy.claude.com/courses/claude-code-101/how-claude-code-works)

**Related:** [[Large Language Model]], [[Prompt]], [[Context Window]], [[Agent]], [[Claude Code]]

---

## Description

1. You enter a prompt into a `LLM`.
2. The `LLM` gathers the context it needs by interacting with the model, which returns text or a tool call that `LLM` can execute.
3. It takes action — for example, editing a file or running a command.
4. It verifies the results and determines whether they achieve what your ``prompt`` set out to do.
5. If they do, the `LLM` finishes and waits for the next prompt. If they don't, it loops back and tries again until the results are complete and verifiable.

Throughout this loop, you can add context, interrupt, or steer the model to help guide it toward your goal.

![[agentic_loop.png]]

---

## Key concepts

### Context

An `LLM` has a ``context window`` that **determines how much** of your conversation, file contents, command outputs, and more **it can store and reference**. **Once you reach that limit, the** `LLM` **compacts your conversation** — automatically determining what it can remove or summarize to bring the context window back down to a usable size.

### Tools

_Tools_ **are the backbone of how** ``agents`` work. Most ``AI`` assistants simply take text in and return text out. _Tools_ let an `LLM` determine _when_ to execute code to get closer to completing a task. This could be a file-reading _tool_, a web search _tool_, or any number of other capabilities. The `LLM` uses semantic understanding to determine when to call a _tool_ and how to use the output.

#### Tools available in ``Claude Code``

![[Tools_with_claude_code.png]]

---

## Claude Sessions
