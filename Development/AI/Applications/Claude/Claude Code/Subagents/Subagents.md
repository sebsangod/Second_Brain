---
aliases:
  - Claude Code
tags:
  - learning
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code 101](https://anthropic.skilljar.com/claude-code-101)

**Related:** [[Claude]], [[Claude Code]], [[Context Window]]

---

## Description

_Subagents_ are specialized assistants that `Claude Code` can delegate tasks to. Think of them as focused helpers: **each one runs in its own conversation context window, does its work, and returns a summary to the main thread.** The intermediate steps (all the file reads, searches, and tool calls) stay isolated and never clutter your main conversation.

``Claude`` **can delegate tasks to subagents that break them down and run component tasks in parallel, improving your** ``context management``. Each _subagent_ operates in its own isolated context window.

---

## Key Takeaways

_Subagents_ give you **three main benefits:**

- **They break work into focused pieces**, letting each _subagent_ concentrate on a specific task
- They keep your `main context window` clean by isolating all the intermediate work
- They bring back just the information you need as a concise summary

Whether you're using the built-in _subagents_ or creating your own, they're a practical way to get more out of longer `Claude Code` sessions. The less noise in your main context, the longer and more effectively you can work.

---

## Details

### Why _Subagents_ Matter

Every time you chat with `Claude Code`, you're adding to the `main context window`. **Every tool call, every file read, every search result gets stored there.** That space is finite, and once it fills up, `Claude` starts losing track of earlier parts of the conversation.

_Subagents_ solve this by spinning up a separate `context window`. The _subagent_ receives two things:

- **A custom system prompt** from your configuration file that defines the subagent's role and behavior
- **A task description** written by the parent agent based on what you asked for

The _subagent_ then works on its own. It reads files, runs searches, edits code or whatever it needs to do. **When it's done, only a summary comes back to your main conversation.** The entire _subagent_ conversation is **then discarded.**

This means your main context stays clean. You get the answer without all the noise of the journey it took to find it. The tradeoff is that you lose visibility into how the _subagent_ reached its conclusions.


### A Practical Example

Say you're exploring an unfamiliar codebase and you want to know which service handles refunds. Without a _subagent_, `Claude` might read 15 files, run several searches, and trace through multiple function calls. All of that fills your `context window`, even though you only needed one fact.

With a _subagent_, the experience is much cleaner. You ask the question, the Explore _subagent_ spins up, does all that digging in its own context, and hands back a focused answer.

Your `main context window` only records the question and the summary, not the 15 files that were read along the way.


### Built-in _Subagents_

`Claude Code` ships with several built-in _subagents_ you can use right away:

- **General purpose subagent:** for multi-step tasks that require both exploration and action
- **Explore:** for fast searching and navigation of codebases
- **Plan:** used during plan mode for research and analysis of your codebase before presenting a plan


### Custom _Subagents_

Beyond the built-in options, you can create your own _subagents_ with custom system prompts and tool access. This lets you define specialized agents tailored to your workflow -- a code reviewer, a test writer, a documentation generator, or anything else you need.


### Creating Your Own Subagent

Subagents are defined in Markdown files with YAML frontmatter. The easiest way to get started is to let Claude generate one for you. Run:

```bash title:bash
$ /agents
```

Then select "Create new agent." You'll walk through steps including choosing the scope of the agent, defining its purpose, selecting the tools it has access to, and even picking a color for it.

Claude will generate a name, description, and prompt for the subagent. This also tells Claude when to call the subagent based on the prompts you give it.

## Further Customization

Subagents can be customized further. Here are some highlights:

- **Persistent memory** lets your subagent retain memory across conversations. This is great if you're using it consistently on the same projects.
- **Preload skills** into subagents by adding the `skill` key and listing skills by name. Note that unlike skills in your main conversation, the entire skill is loaded into context here.


## Tips

### Making `Claude` Use Your _Subagent_ Automatically

If you want `Claude` to delegate tasks to the _subagent_ without you explicitly asking, include the word **"proactively"** in the description field. For example:

```markdown title:".claude/agents/code-quality-reviewer"
___
...
description: Proactively suggest running this agent after major code changes...
...
___

...
```

You can also add example conversations to the description to help `Claude` understand specific scenarios where the _subagent_ should be used. **The more concrete your examples, the better** `Claude` **gets at knowing when to delegate.**


### Defining an **Output Format**

The single most important improvement you can make to a _subagent_ is defining an **output format** in its system prompt. This does two things:

- **It creates natural stopping points:** the _subagent_ knows it's done when it has filled in each section of the format.
- **It prevents the subagent from running too long:** Without a defined output, _subagents_ struggle to decide when enough research has been done and tend to run much longer than necessary.

Here's an example of a structured output format for a code review _subagent_:

```markdown title:".claude/agents/code-quality-reviewer"
...

Provide your review in a structured format:
1. Summary: Brief overview of what you reviewed and overall assessment
2. Critical Issues: Any security vulnerabilities, data integrity risks, or logic errors that must be fixed immediately
3. Major Issues: Quality problems, architecture misalignment, or significant performance concerns
4. Minor Issues: Style inconsistencies, documentation gaps, or minor optimizations
5. Recommendations: Suggestions for improvement, refactoring opportunities, or best practices to apply
6. Approval Status: Clear statement of whether the code is ready to merge/deploy or requires changes

...
```

This format gives the _subagent_ a clear checklist to work through. Once every section is filled in, the _subagent_ knows it can stop.


### Reporting Obstacles

When a subagent discovers a workaround during its work -- like solving a dependency issue or finding that a certain command needs particular flags -- those details need to appear in the summary it returns. If they don't, the main thread has to rediscover the same solutions on its own, which wastes time and tokens.

The kinds of things you want surfaced include:

- Setup issues or environment quirks
- Workarounds discovered during the task
- Commands that needed special flags or configuration
- Dependencies or imports that caused problems

The way to get this information is to explicitly ask for it in the output format. Adding an "Obstacles Encountered" section to your output template surfaces this information reliably.

```markdown title:".claude/agents/code-quality-reviewer"
...

Provide your review in a structured format:
...
7. Obstacles Encountered: Report any obstacles encountered during the review process. This can be: setup issues, workarounds discovered or environment quirks. Report commands that needed a special flag or configuration. Report dependencies or imports that caused problems.
   
...

```


### Limiting Tool Access

Not every _subagent_ needs access to every tool. Think about what a _subagent_ actually needs to do, and only give it the tools required for that job. This does two things: it prevents unintended side effects, and it makes each _subagent's_ role clearer when you have several of them.

Here's how to think about tool access for common _subagent_ types:

- **Research / read-only subagent** -- Only needs `Glob`, `Grep`, and `Read`. Cannot accidentally modify files.
- **Code reviewer** -- Needs `Bash` access to run `git diff` and see what changed, but still doesn't need `Edit` or `Write`.
- **Styling / code modification agent** -- This is where you give `Edit` and `Write` access, because the subagent's job is to actually change your code.


### Putting It All Together

Effective _subagents_ share four characteristics:

1. **Specific descriptions:** The description controls when the _subagent_ is launched and what instructions it receives. Write it to steer both.
2. **Structured output:** Define an output format in the system prompt so the _subagent_ knows when it's done and returns information the main thread can use.
3. **Obstacle reporting:** Include a section in the output format for workarounds, quirks, and problems so the main thread doesn't have to rediscover them.
4. **Limited tool access:** Only give a _subagent_ the tools it actually needs. Read-only for research, bash for reviewers, edit/write only for agents that should change code.

**Each of these patterns is simple on its own, but together they turn a** _subagent_ **from something that vaguely tries to help into a focused, predictable worker that finishes on time and reports back clearly.**


## Using _subagents_ effectively

You know how to create _subagents_ and design them well. Now the question is: **when do they actually help, and when do they get in the way?** The difference comes down to one thing: **whether the intermediate work matters to your main thread.**


### When _subagents_ shine

_Subagents_ **work best when the exploration is separate from the execution.** If each step in a task depends on what the previous step discovered, you want that work in your main thread. But **if you just need an answer and don't care about the journey, delegate it.**

_Subagents_ **excel at tasks where:**

- You need a result, not a play-by-play of how it was found
- The exploratory work would clutter your main thread's context
- The task benefits from a fresh perspective or a custom system prompt


### Research tasks

Research is the classic _subagent_ use case. Consider investigating how authentication works in an unfamiliar codebase. Your main thread needs to know _where_ the JWT is validated, but it doesn't need to see every file that was searched along the way.

A research _subagent_ can read dozens of files, trace through function calls, and explore different code paths. All that exploration stays in the _subagent's_ context. Your main thread receives a clean summary like:

```bash title:bash
JWT validation happens in middleware/auth.js line 42,
called from the Express router in route/api.js
```

The _subagent_ did the heavy lifting. Your main thread gets exactly what it needs to move forward.


### Code Reviews

`Claude` reviews code more effectively when the code is presented as being authored by someone else. If you built a feature over many turns with your main thread, asking that same thread to review it often produces weak feedback. `Claude` was involved in creating it, so it has trouble seeing it with fresh eyes.

A reviewer _subagent_ sees the changes in a separate context. It runs _git diff_, reads the modified files, and applies its specialized review criteria without the history of how the code was written. This separation also lets you encode project-specific review standards in the _subagent's_ system prompt, ensuring consistent review criteria across the team.


### Custom System Prompts

`Claude Code's` default system prompt emphasizes concise, code-focused responses. That works great for coding, but not for everything.

Here are two cases where a custom system prompt makes the _subagent_ genuinely better than the main thread:

- **Copywriting subagent:** Give it instructions about tone, audience, and style. `Claude Code's` default prompt tends toward concise technical writing, which really isn't what you want for a landing page or email campaign. A copywriting _subagent_ can have completely different instructions about voice and structure.
- **Styling subagent:** Point it at your design system files. When the _subagent_ runs, those files load into its context automatically, so it knows your color variables, spacing conventions, and component patterns before it even starts writing any CSS.


### When _Subagents_ Hurt

The overhead of launching a _subagent_ -- losing visibility into its work and compressing its findings into a summary -- **only makes sense when the** _subagent_ **does something the main thread can't.** There are three common anti-patterns to watch out for.


#### Expert Claims

_Subagents_ that claim expertise rarely help. **Prompts like "you are a Python expert" or "you are a Kubernetes specialist" add no value because** `Claude` **already has that knowledge.** There's nothing a so-called expert _subagent_ can do that your main thread can't do directly.


#### Sequential Pipelines

Sequential _subagent_ pipelines create problems. Consider a three-agent flow: one to reproduce a bug, one to debug it, and one to fix it. Pipelines work when tasks are truly independent. They **fail when each step depends on discoveries from the previous step**. And bug fixing almost always does. Information gets lost in the handoff between _agents_.


#### Test Runners

Test runner _subagents_ tend to hide information you need. When tests fail, you want the full output to diagnose issues. A _subagent_ that returns "tests failed" forces you to create additional debug scripts to get details that would have been visible in direct output. Testing has shown that the test runner pattern performed worse among all configurations.


### The Decision Rule

When you're deciding whether to use a _subagent_, ask yourself one question: **does the intermediate work matter?**

If the answer is no (you just need the final result), delegate it to a _subagent_. If the answer is yes (you need to see and react to what's happening along the way) keep it in your main thread.

Use _subagents_ for:

- Research and exploration
- Code reviews
- Tasks that need a custom system prompt

Avoid _subagents_ for:

- "Expert" personas that don't add real capability
- Multi-step pipelines where each step depends on the last

---

## Claude Sessions
