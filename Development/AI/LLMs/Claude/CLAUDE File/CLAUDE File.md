---
aliases:
  - CLAUDE File
tags:
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code 101](https://academy.claude.com/es/courses/claude-code-101/the-claude-md-file)

**Related:** [[Claude Code]], [[Hooks]]

---

## Description

One of the most useful features in ``Claude Code`` is the _CLAUDE.md_ file. **It gives** ``Claude Code`` **persistent memory about your project.**

The _CLAUDE.md_ file serves two main purposes:

- Guides Claude through your codebase, pointing out important commands, architecture, and coding style
- Allows you to give Claude specific or custom directions

This file gets included in every request you make to Claude, so **it's like having a persistent system prompt for your project**.

---

## Details

### The Problem It Solves

When you open ``Claude Code`` without a _CLAUDE.md_ file, it starts fresh every time. It has to re-explore your codebase, figure out what dependencies are needed, and understand what features are already implemented. Sometimes it makes assumptions, which makes it harder to steer Claude in the right direction.

_CLAUDE.md_ solves this. It's a ``Markdown`` file you add to the root of your project, and ``Claude Code`` reads it automatically every time you start a session. **Think of it as an onboarding script for your codebase.** The contents of the _CLAUDE.md_ file are appended to your prompt.

### First, ask if _CLAUDE.md_ is even the right tool

Before you write a rule, ask whether it belongs in _CLAUDE.md_ at all. **Some rules are guidance, and some rules are hard lines that must never be crossed**. Those are two different jobs.

Take a rule like "never push to main." If you put that in _CLAUDE.md_, you're hoping Claude reads it and respects it. Most of the time it will. But "most of the time" isn't good enough for something that dangerous. **A hard rule like that belongs in a pre-tool-use** ``hook`` **instead**.

The difference matters. A ``hook`` is code that runs before Claude takes an action, and it can actually block the action. So even if Claude does try to push to main, the hook stops it. That's real enforcement, not a polite request. **Move your hard rules to** ``hooks`` **and let** _CLAUDE.md_ **handle the softer conventions**.

___

### _CLAUDE.md_ is for Teams

You can (and should) commit your _CLAUDE.md_ to version control so your team benefits from it. There's actually a hierarchy of memory files depending on who they're for:

- **Project-level CLAUDE.md** lives in the root directory of your project. Shared with the team.

- **User-level CLAUDE.md** lives in your configuration folder. This one is just for you and applies across all your projects. Put your personal preferences here.

### Location

Claude recognizes three different `CLAUDE.md` files in three common locations:

- **CLAUDE.md** - Generated with _/init_ (in the root of the project folders structure), committed to source control, shared with other engineers
- **CLAUDE.local.md** - Not shared with other engineers, contains personal instructions and customizations for Claude
- **~/.claude/CLAUDE.md** - Used with all projects on your machine, contains instructions that you want Claude to follow on all projects


### Adding Custom Instructions

**You can customize how Claude behaves by adding instructions to your _CLAUDE.md_ file**. For example, if Claude is adding too many comments to code, you can address this by updating the file.

Use the _#_ command to enter _memory mode_ - this lets you edit your _CLAUDE.md_ files intelligently. Just type something like:

```bash title:Claude_Code_bash
# Use comments sparingly. Only comment complex code.
```

Claude will merge this instruction into your _CLAUDE.md_ file automatically.


### File Mentions with _@_

When you need Claude to look at specific files, use the _@_ symbol followed by the file path. This automatically includes that file's contents in your request to Claude.

For example, if you want to ask about your authentication system and you know the relevant files, you can type:

```bash title:Claude_Code_bash
How does the auth system work? @auth
```

Claude will show you a list of auth-related files to choose from, then include the selected file in your conversation.


### Referencing Files in _CLAUDE.md_

You can also mention files directly in your _CLAUDE.md_ file using the same `@` syntax. This is particularly useful for files that are relevant to many aspects of your project.

For example, if you have a database schema file that defines your data structure, you might add this to your _CLAUDE.md_:

```bash title:Claude_Code_bash
The database schema is defined in the @prisma/schema.prisma file. Reference it anytime you need to understand the structure of data stored in the database.
```

When you mention a file this way, its contents are automatically included in every request, so Claude can answer questions about your data structure immediately without having to search for and read the schema file each time.

___

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

We recommend starting a project without a _CLAUDE.md_ file so you can see where you constantly have to course-correct the model. This keeps your _CLAUDE.md_ compact and focused on only the necessary information. When you're ready, run `/init` to have Claude generate one for you.

### Keep the file under revision

Your _CLAUDE.md_ file is never finished. **Treat it like living code that keeps getting edited**.

When Claude does the wrong thing, don't just sigh and fix it by hand. Treat it as a bug report against your _CLAUDE.md_ file. You can even tell Claude directly: "add that to the _CLAUDE.md_ file," and it'll write the rule for you. That way the file gets better every time something goes wrong.

### The bottom line

**Treat your** _CLAUDE.md_ **like production code**. If you can't justify a line, delete it. To keep the file lean and followable:

2. Move hard rules to ``hooks``, where they're actually enforced.
3. Organize long files with imports (just remember they don't reduce context).
4. Make every rule specific and checkable, and name the replacement.
5. Spend your emphasis budget on the few rules that matter most.
6. Keep revising the file whenever Claude gets something wrong.

The whole idea is simple. The leaner the file, the more of it Claude follows.

___

## Example

Here's what a typical _CLAUDE.md_ file looks like:

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

It's straightforward. Now if you ask ``Claude Code`` to create a React component, it already knows to use Tailwind for styling and to follow your code conventions.

___

## Claude Sessions
