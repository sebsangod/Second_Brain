---
aliases:
  - Claude Code
tags:
  - learning
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code 101](https://anthropic.skilljar.com/claude-code-101)

**Related:** [[Claude Code]], [[Claude]]

---

## Description

One of the most useful features in ``Claude Code`` is the _CLAUDE.md_ file. **It gives** ``Claude Code`` **persistent memory about your project.**

---

## The Problem It Solves

When you open ``Claude Code`` without a _CLAUDE.md_ file, it starts fresh every time. It has to re-explore your codebase, figure out what dependencies are needed, and understand what features are already implemented. Sometimes it makes assumptions, which makes it harder to steer ``Claude`` in the right direction.

_CLAUDE.md_ solves this. It's a Markdown file you add to the root of your project, and ``Claude Code`` reads it automatically every time you start a session. **Think of it as an onboarding script for your codebase.** The contents of the _CLAUDE.md_ file are appended to your prompt.

___

## Example

Here's what a typical CLAUDE.md file looks like:

```markdown title:CLAUDE.md
# Project
This is a Next.js 15 app using the App Router, Tailwind, and Drizzle ORM.

# Commands
- Dev server: `pnpm dev`
- Run tests: `pnpm test`
- Lint: `pnpm lint`

# Code Style
- Use 2-space indentation
- Prefer named exports
- All API routes go in app/api/
- Use server actions instead of API routes where possible
```

It's straightforward. Now if you ask Claude Code to create a React component, it already knows to use Tailwind for styling and to follow your code conventions.

___

## _CLAUDE.md_ is for Teams

You can (and should) commit your _CLAUDE.md_ to version control so your team benefits from it. There's actually a hierarchy of memory files depending on who they're for:

- **Project-level CLAUDE.md** lives in the root directory of your project. Shared with the team.

- **User-level CLAUDE.md** lives in your configuration folder. This one is just for you and applies across all your projects. Put your personal preferences here.

---

## Tips

### Save corrections to memory.

If you find yourself correcting Claude repeatedly — like telling it to always use server actions instead of API routes — explicitly ask Claude to save that rule to memory. Next time you open the project, it'll know.

![[claude_md_file_tips.png]]


### Reference project docs.

If you have documentation in your project that you want Claude to reference, use the `@` symbol with the file path:

```markdown title:CLAUDE.md
## README.md

Please read if you need more info: @README.md
```


### Start without one.

We recommend starting a project without a CLAUDE.md file so you can see where you constantly have to course-correct the model. This keeps your CLAUDE.md compact and focused on only the necessary information. When you're ready, run `/init` to have Claude generate one for you.
___

## Claude Sessions
