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

#### _/init_

**When working with `Claude` on coding projects, context management is crucial**. Your project might have dozens or hundreds of files, but ``Claude`` only needs the right information to help you effectively. Too much irrelevant context actually decreases ``Claude's`` performance, so learning to guide it toward relevant files and documentation is essential.

When you first start `Claude` in a new project, run the _/init_ command. This tells `Claude` to analyze your entire codebase and understand:

- The project's purpose and architecture
- Important commands and critical files
- Coding patterns and structure

![[command_init.png|1082]]

After analyzing your code, `Claude` creates a summary and writes it to a _CLAUDE.md_ file. When `Claude` asks for permission to create this file, you can either hit Enter to approve each write operation, or press Shift+Tab to let `Claude` write files freely throughout your session.


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

### Code Commands

#### _/diff_

**A diff is the before-and-after of a change**: the lines removed and the lines added, file by file. The _/diff_ command opens an interactive viewer of your uncommitted changes in that form, and it can also show what each of ``Claude's`` turns changed. Use the up and down arrows to move between files and _Enter_ to open one.
  
The most important things that deserve a second look every time are:

- **Changes you didn't ask for.** A config value that was edited while ``Claude`` was in the file or a rewritten helper method that you didn't mention.
- **Tests that got weaker.** If the project you're working in contains tests, identify any that were skipped, deleted, or loosened until they passed.
- **New packages and hard-coded values.** A dependency that was added for only one function, a URL or a key written straight into the code.

#### _/code-review_

The Explore → Plan → Code → Commit lesson said to have a second reviewer on a change before you commit it. A long session carries everything it has read and decided. That's the context you learned to manage in the previous lesson, and it's exactly the history you don't want in a reviewer. _/code-review_ is that second reviewer: **it reviews the change in a clean context, with none of your session's history, and reports what it finds**. It edits nothing unless you ask it to.

The review runs in the background, anywhere from seconds to a few minutes, and counts against your usage like any other task, **so save it for changes that deserve a second look**. The findings arrive in your conversation when it finishes. You can also ask in plain words, and ``Claude`` can start the same review from the request. If ``Claude`` answers inline instead of starting a review, run the command yourself.

### _/goal_

Goal sets a completion condition. You describe what "done" looks like, and Claude keeps working across turns until a fast evaluator confirms those conditions are met. It won't just stop the first time it thinks it's finished.

For example:
```bash title:bash
/goal all tests in src/billing pass, and the type checker reports zero errors
```

To cancel it, run _/goal clear_. One important constraint: the evaluator only reads the transcript. So your condition has to be checkable from the output Claude actually produces, like the results of a test run.

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

## Custom

### Distribute

#### Description

Custom `command` that reads the last transcription created by the `hook` `Obsidian Syncing` and its related notes, and distributes the relevant content into one or more existing or new notes.

#### Instructions

Read the latest unprocessed Claude Code session transcript from my Obsidian vault  
and distribute its content into the relevant notes using MCPVault MCP tools.

##### Steps

1. **Find the transcript.**
    
    - Use `list_directory` on `Journal/Claude Sessions/` → find the most recent date folder.
    - Use `list_directory` on that folder → list .md files.
    - Use `get_frontmatter` on the most recent file to confirm `distributed: false`.
    - If $ARGUMENTS is provided, filter by filename match instead.
    - Use `read_note` to get the full transcript content.
2. **Discover target notes.**
    
    - Parse the `Related:` wikilinks from the transcript header — these are the primary targets.
    - Extract 3–5 key technical topics from the conversation body.
    - Run `search_notes` with those topics to find additional relevant notes not in Related:.
    - Combine both sources, de-duplicate, cap at 10 total (prioritize Related: first).
3. **Read target notes in batch.**
    
    - Use `read_multiple_notes` with all target paths (up to 10 at once).
    - If more than 10, make a second call for the remainder.
4. **Write the Summary section in the transcript.**
    
    - Synthesize a 2–4 sentence summary of what was discussed and decided.
    - Use `patch_note` to replace `Write here...` inside `## Summary` with the summary text.
5. **Distribute content into each target note.**  
    For each note, add only what's new and relevant:
    
    - **Existing sections** (Description, Details, Key concepts, etc.) →  
        use `patch_note` to append new information at the end of the right section.
    - **New sections needed** → use `patch_note` to insert them before `## Claude Sessions`.
    - **Stub notes** (`#claude-generated`, body is `Write here...`) →  
        use `patch_note` to replace the stub with a proper overview.
    - **Code examples** → add in fenced code blocks with language tags.
6. **Mark as distributed.**
    
    - Use `update_frontmatter` on the transcript to set `distributed: true`.
7. **Show a summary** of what was changed in each note.
    

##### Rules

- NEVER remove or overwrite existing content — only add.
- Match each note's style: header levels, list formats, wikilink conventions, language.
- Don't duplicate information already present in the note.
- Keep additions concise — reference notes, not essays.
- Preserve all frontmatter, tags, and `## Claude Sessions` sections untouched.
- When using `patch_note`, use a string long enough to be unique in the file.
- If `read_multiple_notes` needs more than 10 files, make two calls.

---

## Claude Sessions
