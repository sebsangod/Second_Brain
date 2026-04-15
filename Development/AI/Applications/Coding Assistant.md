---
aliases:
  - Coding Assistant
tags:
  - learning
---
**Date**: 19/mar/2026
**Update**: 19/mar/2026
**Sources**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

**Tags:** #LLM #AI #assistant #development

**Related:** [[Artificial Intelligence]], [[Large Language Models]], [[Claude]], [[Claude Code]], [[Opus]], [[Sonnet]], [[Haiku]]

---

## Description

A coding assistant is more than just a tool that writes code - **it's a sophisticated system that uses language models (`LLMs`) to tackle complex programming tasks.** Understanding how these assistants work behind the scenes will help you appreciate what makes a truly powerful coding companion.

---

## Key concepts

Understanding coding assistants comes down to a few essential points:

- Coding assistants use `LLMs` to complete different tasks
- `LLMs` need tools to handle most real-world programming tasks or to work a vast majority of tasks
- Not all `LLMs` use tools with the same skill level
- `Claude's` strong tool use enables better security, customization, and longevity in `Claude Code`

---

## Details

### How Coding Assistants Work

When you give a coding assistant a task, like fixing a bug based on an error message, it follows a process similar to how a human developer would approach the problem:

![[What_is_a_coding_assistant.png]]

1. **Gather context** - Understanding what the error refers to, which part of the codebase is affected, and what files are relevant
2. **Formulate a plan** - Deciding how to solve the issue, such as changing code and running tests to verify the fix
3. **Take action** - Actually implementing the solution by updating files and running commands

The key insight here is that **the first and last steps require the assistant to interact with the outside world** - reading files, fetching documentation, running commands, or editing code.


### The Tool Use Challenge

**`LLMs` by themselves can only process text and return text** (they can't actually read files or run commands). If you ask a standalone `LLM` to read a file, it will tell you it doesn't have that capability.

So how do coding assistants solve this problem? They use a clever system called _tool use_.


### How Tool Use Works

**When you send a request to a coding assistant, it automatically adds instructions to your message that teach the `LLM` how to request actions.** For example, it might add text like: "If you want to read a file, respond with 'ReadFile: name of file'"

Here's the complete flow:

1. You ask: "What code is written in the main.go file?"
2. The coding assistant adds tool instructions to your request
3. The `LLM` responds: "ReadFile: main.go"
4. The coding assistant reads the actual file and sends its contents back to the model
5. The `LLM` provides a final answer based on the file contents

This system allows `LLMs` to effectively "read files," "write code," and "run commands" **even though they're really just generating carefully formatted text responses**.


### Why Claude's Tool Use Matters

Not all `LLMs` are equally good at using tools. The Claude series of models (`Opus`, `Sonnet`, and `Haiku`) are particularly strong at understanding what tools do and using them effectively to complete complex tasks.

![[How_coding_assistants_work.png]]

![[Tool_use.png]]


### Benefits of Strong Tool Use

![[Strong_tool_usage.png]]

This tool-use capability is what transforms a simple text-generating model into a powerful coding assistant that can read your files, understand your codebase, and make meaningful changes to your projects.

---