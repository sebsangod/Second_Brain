---
aliases:
  - Skills
tags:
  - learning
  - dev/ai/llm
date: 2026-04-14
---
**Sources**: [Claude Code Skills](https://code.claude.com/docs/en/skills)

**Related:** [[Claude]], [[Claude Code]], [[CLAUDE.md File]], [[Context Window]]

---

The rule of thumb is simple:
<h1 style="text-align: center;">If you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written.</h1>

___

## Description

_Skills_ are **folders of instructions and resources** that `Claude Code` can discover and use **to handle tasks more accurately.** Each skill lives in a _SKILL.md_ file with a name and description in its _frontmatter_.

>[!important] `Claude` **loads only skill names and descriptions at startup**.

>[!warning] Priority for name conflicts
>**Enterprise → Personal → Project → Plugins**

___

## Details

**The description is how** `Claude` **decides whether to use the skill.** When you ask `Claude` to review a PR, it matches your request against available _skill_ descriptions and finds the relevant one. `Claude` reads your request, compares it to all available _skill_ descriptions, and activates the ones that match.

Below the _frontmatter_, **you write the actual instructions**: your review checklist, formatting preferences, or whatever `Claude` needs to know for that task.

>[!note] Frontmatter
>The _frontmatter_ is the header of the file, separated by "---".

![[skills_preview.png]]


### Where Skills Live

You can store skills in different places depending on who needs them:

- **Personal skills** go in _~/.claude/skills_ (your home directory). **These follow you across all your projects.** Your commit message style, your documentation format, how you like code explained.
- **Project skills** go in _.claude/skills_ **inside the root directory of your repository Anyone who clones the repo gets these skills automatically.** This is where team standards live, like your company's brand guidelines, preferred fonts, and colors for web design.

>[!note]
>On Windows, personal skills live in _C:/Users/[your-user]/.claude/skills_.


### Skills vs. `CLAUDE.md` vs. Slash Commands

`Claude Code` has several ways to customize behavior. _Skills_ are unique because they're automatic and task-specific. Here's how they compare:

| **Skills**                                                                                                                                                                                                                                                                                         | **CLAUDE.md**                                                                                                                     | **Slash commands**                                                                                                |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Load on demand when they match your request.<br><br>`Claude` only loads the name and description initially, so they don't fill up your entire `context window`.<br><br>Your PR review checklist doesn't need to be in context when you're debugging — it loads when you actually ask for a review. | Files load into every conversation.<br><br>If you want `Claude` to always use TypeScript's strict mode, that goes in `CLAUDE.md`. | Require you to explicitly type them. Skills don't.<br><br>`Claude` applies them when it recognizes the situation. |


### Creating a Skill

In this section, a skill that teaches `Claude` how to write PR descriptions in a consistent format will be created. **Since it's a personal skill, it lives in your home directory and works across all your projects.**

First, create a directory for your skill inside the skills folder:

```bash title:bash
$ mkdir -p ~/.claude/skills/pr-description
```

>[!warning] The directory name **MUST** match the skill name

Then create a _SKILL.md_ file inside that directory. The file has two parts separated by _frontmatter_ dashes:

```markdown title:SKILL.md
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR, writing a PR, or when the user asks to summarize changes for a pull request.
---

When writing a PR description:

1. Run `git diff main...HEAD` to see all changes on this branch
2. Write a description following this format:

## What
One sentence explaining what this PR does.

## Why
Brief context on why this change is needed

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
```


#### Writing Effective Descriptions

**Be explicit with your instructions.** If someone told you "your job is to help with docs," you wouldn't know what to do, and `Claude` thinks the same way.

**A good description answers two questions:**

1. What does the skill do?
2. When should Claude use it?

If your _skill_ isn't triggering when you expect it to, try adding more **keywords** that match how you actually phrase your requests. The description is what `Claude` uses to decide whether a _skill_ is relevant, so **the language matters.**


### Metadata Fields

The agent _skills_ open standard supports several fields in the _SKILL.md frontmatter_. Two are required, and the rest are optional:

- **name** (required) — Identifies your skill. Use lowercase letters, numbers, and hyphens only. Maximum 64 characters. Should match your directory name.
- **description** (required) — Tells Claude when to use the skill. Maximum 1,024 characters. This is the most important field because Claude uses it for matching.
- **allowed-tools** (optional) — Restricts which tools Claude can use when the skill is active: (Read, Grep, Glob, Bash)
- **model** (optional) — Specifies which Claude model to use for the skill.


### Progressive Disclosure

_Skills_ share `Claude's` `context window` with your conversation. When `Claude` activates a skill, it loads the contents of that _SKILL.md_ into context. But sometimes you need references, examples, or utility scripts that the skill depends on.

**Cramming everything into one 2,000-line file has two problems: it takes up a lot of context window space, and it's not fun to maintain.**

_Progressive disclosure_ **solves this. Keep essential instructions in** _SKILL.md_ **and put detailed reference material in separate files that** `Claude` **reads only when needed.**

The open standard suggests organizing your skill directory with:

- _scripts/_ — Executable code
- _references/_ — Additional documentation
- _assets/_ — Images, templates, or other data files

Then in _SKILL.md_, link to the supporting files with clear instructions about when to load them:

![[skill_resource_reference.png]]

In this example, `Claude` reads _architecture-guide.md_ only when someone asks about system design. If they're asking where to add a component, it never loads that file. **It's like having a table of contents in the** `context window` **rather than the entire document.**

A good rule of thumb: **keep SKILL.md under 500 lines**. If you're exceeding that, consider whether the content should be split into separate reference files.

___

## Claude Sessions