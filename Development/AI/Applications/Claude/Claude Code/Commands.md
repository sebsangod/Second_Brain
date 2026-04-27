---
aliases:
  - Claude Code
tags:
  - learning
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

**Related:** [[Claude Code]]

---

## Description

Write here...

---

## Key concepts

Write here...

---

## Details

### Context Management Commands

`Claude` provides several commands to help manage conversation context effectively:


#### _/compact_

The _/compact_ command summarizes your entire conversation history while preserving the key information `Claude` has learned. This is ideal when:

- `Claude` has gained valuable knowledge about your project
- You want to continue with related tasks
- The conversation has become long but contains important context

Use compact when `Claude` has learned a lot about the current task and you want to maintain that knowledge as it moves to the next related task.


#### _/clear_

The _/clear_ command completely removes the conversation history, giving you a fresh start. This is most useful when:

- You're switching to a completely different, unrelated task
- The current conversation context might confuse `Claude` for the new task
- You want to start over without any previous context


#### _/context_

To check the state of your context, run the `/context` command. You'll get a high-level overview of your context size, the categories taking up the most space, and a visual graphic showing the breakdown.

![[context_command.png]]


#### When to Use These Techniques

These conversation control techniques are particularly valuable during:

- Long-running conversations where context can become cluttered
- Task transitions where previous context might be distracting
- Situations where `Claude` repeatedly makes the same mistakes
- Complex projects where you need to maintain focus on specific components

By using _escape_, double-tap _escape_, _/compact_, and _/clear_ strategically, you can keep `Claude` focused and productive throughout your development workflow. **These aren't just convenience features. They're essential tools for maintaining effective AI-assisted development sessions.**

A general rule of thumb:

- **Use `/compact`** when you're working on a specific feature and running up against the context limit but need to continue. Keeping the context relevant to your current feature is important.
- **Use `/clear`** when you want to start a new feature. You don't want the previous conversation to introduce bias into something new. For things you want Claude to remember across sessions, put them in your ``CLAUDE.md`` file so it doesn't have to rediscover things from scratch.


## Custom commands

Claude Code comes with built-in commands that you can access by typing a forward slash, but **you can also create your own custom commands to automate repetitive tasks you run frequently.**

### Creating Custom Commands

To create a custom command, you need to set up a specific folder structure in your project:

1. Find the _.claude_ folder in your project directory
2. Create a new directory called _commands_ inside it
3. Create a new markdown file with your desired command name (like _audit.md_)

The filename becomes your command name - so _audit.md_ creates the _/audit_ command.


### Example: Audit Command

Here's a practical example of a custom command that audits project dependencies for vulnerabilities:

```markdown title:audit.md
Your goal is to update any vulnerable dependencies.

Do the following:

1. Run `npm audit` to find vulnerable installed packages
2. Run `npm audit fix` to apply updates
3. Run tests to verify the updates didn't break anything
```

After creating your command file, you must restart Claude Code for it to recognize the new command.


### Commands with Arguments

Custom commands can accept arguments using the _$ARGUMENTS_ placeholder. This makes them much more flexible and reusable.

For example, a _write_tests.md_ command might contain:

```markdown title:write_tests.md
Write comprehensive tests for: $ARGUMENTS

Testing conventions:
* Use Vitests with React Testing Library
* Place test files in a __tests__ directory in the same folder as the source file
* Name test files as [filename].test.ts(x)
* Use @/ prefix for imports

Coverage:
* Test happy paths
* Test edge cases
* Test error states
```

You can then run this command with a file path:

_/write_tests the use-auth.ts file in the hooks directory_

The arguments don't have to be file paths - they can be any string you want to pass to give Claude context and direction for the task.


### Key Benefits

- **Automation**: Turn repetitive workflows into single commands
- **Consistency**: Ensure the same steps are followed every time
- **Context**: Provide `Claude` with specific instructions and conventions for your project
- **Flexibility**: Use arguments to make commands work with different inputs

Custom commands are particularly useful for project-specific workflows like running test suites, deploying code, or generating boilerplate following your team's conventions.

---

## Claude Sessions
