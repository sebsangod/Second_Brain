---
aliases:
  - Claude Code
tags:
  - learning
  - dev/ai/llm
date: 2026-05-26
---
**Sources**: [Claude Code](https://claude.com/product/claude-code), [Claude Code 101](https://anthropic.skilljar.com/claude-code-101), [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action), [Exploring the .claude directory](https://code.claude.com/docs/en/claude-directory)

**Related:** [[Artificial Intelligence]], [[Large Language Models]], [[Claude]], [[Anthropic]], [[Context Window]], [[MCP servers]], [[NodeJS]], [[Python]], [[Bash]], [[TypeScript]], [[CI-CD]]

---

## Description

*Claude Code* is a **command-line AI assistant that uses** `LLMs` **to perform development tasks.**

_Claude Code_ is an agentic coding tool that understands your codebase, edits your files, runs commands, and integrates with your existing developer tools to help you get things done faster. It's available in your terminal, Visual Studio Code, the Claude Desktop app, on the web, and in JetBrains IDEs.

---

## Key commands

Write here...

---

## Usage examples

![[Optimization_task.png]]

![[Data_analysis_task.png]]

![[UI_styling_task.png]]

![[Github_integration.png]]

---

## Installation

Full setup directions can be found [here](https://code.claude.com/docs/en/quickstart).

In short, you'll need to do the following:

1. Install Claude Code:

```bash title:"MacOS Homebrew"
$ _brew install --cask claude-code_
```

```bash title:"MacOS, Linux and WSL"
$ _curl -fsSL https://claude.ai/install.sh | bash_
```

```bash title:"Windows CMD"
$ _curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd_
```

2. After installation, run _claude_ at your terminal. The first time you run this command you will be prompted to authenticate

>[!info] **NOTE:** if https://claude.ai/install.sh doesn't work, try with [this URL](https://storage.googleapis.com/claude-code-dist-86c565f3-f756-42ad-8dfa-d59b1c096819/claude-code-releases/bootstrap.sh)

If you're making use of AWS Bedrock or Google Cloud Vertex, there is some additional setup:

- Special directions for [AWS Bedrock](https://code.claude.com/docs/en/amazon-bedrock)
- Special directions for [Google Cloud Vertex](https://code.claude.com/docs/en/google-vertex-ai)

___

## Details

### The Agentic Loop

_Claude Code_ is best explained through the **agentic loop**:

1. You enter a prompt into _Claude Code_.
2. ``Claude`` gathers the context it needs by interacting with the model, which returns text or a tool call that _Claude Code_ can execute.
3. It takes action — for example, editing a file or running a command.
4. It verifies the results and determines whether they achieve what your prompt set out to do.
5. If they do, ``Claude`` finishes and waits for the next prompt. If they don't, it loops back and tries again until the results are complete and verifiable.

Throughout this loop, you can add context, interrupt, or steer the model to help guide it toward your goal.

![[agentic_loop.png]]


### Context

**Think of the context window as the amount of space** ``Claude`` **can hold in its memory.** Whenever you enter a prompt, ``Claude`` reads a file, runs a tool call, or receives a tool call result, it's all adding to the ``context window``. **Since there's a finite amount of space, it becomes important to optimize how you use it.**

``Claude`` has a ``context window`` that determines how much of your conversation, file contents, command outputs, and more it can store and reference. Once you reach that limit, _Claude Code_ compacts your conversation — automatically determining what it can remove or summarize to bring the context window back down to a usable size.


### Tools

Tools are the backbone of how agents work. Most AI assistants simply take text in and return text out. Tools let _Claude Code_ determine _when_ to execute code to get closer to completing a task. This could be a file-reading tool, a web search tool, or any number of other capabilities. _Claude Code_ uses semantic understanding to determine when to call a tool and how to use the output.

_Claude Code_ comes with a comprehensive set of built-in tools that handle common development tasks like reading files, writing code, running commands, and managing directories. But what makes _Claude Code_ truly powerful is how intelligently it combines these tools to tackle complex, multi-step problems.


### Tools available in Claude Code

![[Tools_with_claude_code.png]]

---

## Adding context

**When working with `Claude` on coding projects, context management is crucial**. Your project might have dozens or hundreds of files, but Claude only needs the right information to help you effectively. Too much irrelevant context actually decreases Claude's performance, so learning to guide it toward relevant files and documentation is essential.


### The _/init_ command

When you first start `Claude` in a new project, run the _/init_ command. This tells `Claude` to analyze your entire codebase and understand:

- The project's purpose and architecture
- Important commands and critical files
- Coding patterns and structure

![[command_init.png|1082]]

After analyzing your code, `Claude` creates a summary and writes it to a _CLAUDE.md_ file. When `Claude` asks for permission to create this file, you can either hit Enter to approve each write operation, or press Shift+Tab to let `Claude` write files freely throughout your session.


### The _CLAUDE.md_ File

The _CLAUDE.md_ file serves two main purposes:

- Guides `Claude` through your codebase, pointing out important commands, architecture, and coding style
- Allows you to give `Claude` specific or custom directions

This file gets included in every request you make to `Claude`, so **it's like having a persistent system prompt for your project**.

#### Location

Claude recognizes three different `CLAUDE.md` files in three common locations:

- **CLAUDE.md** - Generated with _/init_ (in the root of the project folders structure), committed to source control, shared with other engineers
- **CLAUDE.local.md** - Not shared with other engineers, contains personal instructions and customizations for `Claude`
- **~/.claude/CLAUDE.md** - Used with all projects on your machine, contains instructions that you want `Claude` to follow on all projects


### Adding Custom Instructions

**You can customize how `Claude` behaves by adding instructions to your _CLAUDE.md_ file**. For example, if `Claude` is adding too many comments to code, you can address this by updating the file.

Use the _#_ command to enter _memory mode_ - this lets you edit your _CLAUDE.md_ files intelligently. Just type something like:

```bash title:Claude_Code_bash
# Use comments sparingly. Only comment complex code.
```

`Claude` will merge this instruction into your _CLAUDE.md_ file automatically.


### File Mentions with _@_

When you need `Claude` to look at specific files, use the _@_ symbol followed by the file path. This automatically includes that file's contents in your request to `Claude`.

For example, if you want to ask about your authentication system and you know the relevant files, you can type:

```bash title:Claude_Code_bash
How does the auth system work? @auth
```

`Claude` will show you a list of auth-related files to choose from, then include the selected file in your conversation.


### Referencing Files in _CLAUDE.md_

You can also mention files directly in your _CLAUDE.md_ file using the same `@` syntax. This is particularly useful for files that are relevant to many aspects of your project.

For example, if you have a database schema file that defines your data structure, you might add this to your _CLAUDE.md_:

```bash title:Claude_Code_bash
The database schema is defined in the @prisma/schema.prisma file. Reference it anytime you need to understand the structure of data stored in the database.
```

When you mention a file this way, its contents are automatically included in every request, so `Claude` can answer questions about your data structure immediately without having to search for and read the schema file each time.

___

## Making changes

When working with `Claude` in your development environment, you'll often need to make changes to existing projects. This section covers practical techniques for implementing changes effectively, including visual communication with screenshots and leveraging `Claude's` advanced reasoning capabilities.


### Using Screenshots for Precise Communication

**One of the most effective ways to communicate with `Claude` is through screenshots**. When you want to modify a specific part of your interface, taking a screenshot helps `Claude` understand exactly what you're referring to.

To paste a screenshot into `Claude`, use _Ctrl + V_ (not _Cmd + V_ on macOS). This keyboard shortcut is specifically designed for pasting screenshots into the chat interface. Once you've pasted the image, you can ask `Claude` to make specific changes to that area of your application.


### Planning Mode

For more complex tasks that require extensive research across your codebase, you can enable _Planning Mode_. **This feature makes `Claude` do thorough exploration of your project before implementing changes.**

Enable _Planning Mode_ by pressing `Shift + Tab` twice (or once if you're already auto-accepting edits). In this mode, `Claude` will:

- Read more files in your project
- Create a detailed implementation plan
- Show you exactly what it intends to do
- Wait for your approval before proceeding

**This gives you the opportunity to review the plan and redirect `Claude` if it missed something important or didn't consider a particular scenario.**


### Thinking Modes

`Claude` offers different levels of reasoning through _thinking_ modes. **These allow `Claude` to spend more time reasoning about complex problems before providing solutions.**

The available thinking modes include:

- **Think**: Basic reasoning
- **Think more**: Extended reasoning
- **Think a lot**: Comprehensive reasoning
- **Think longer**: Extended time reasoning
- **Ultrathink**: Maximum reasoning capability

Each mode gives `Claude` progressively more tokens to work with, allowing for deeper analysis of challenging problems.


### When to Use Planning vs Thinking

These two features handle different types of complexity:

_Planning Mode_ is best for:

- Tasks requiring broad understanding of your codebase
- Multi-step implementations
- Changes that affect multiple files or components

_Thinking Mode_ is best for:

- Complex logic problems
- Debugging difficult issues
- Algorithmic challenges

You can combine both modes for tasks that require both breadth and depth. Just keep in mind that both features consume additional tokens, so there's a cost consideration for using them.

___

## Controlling context

When working with `Claude` on complex tasks, you'll often need to guide the conversation to keep it focused and productive. **There are several techniques you can use to control the flow of your conversation and help `Claude` stay on track.**

![[controlling_context.png]]


### Interrupting `Claude` with _Escape_

Sometimes `Claude` starts heading in the wrong direction or tries to tackle too much at once. **You can press the _Escape key_ to stop `Claude` mid-response, allowing you to redirect the conversation.**

This is particularly useful when you want `Claude` to focus on one specific task instead of trying to handle multiple things simultaneously. For example, if you ask `Claude` to write tests for multiple functions and it starts creating a comprehensive plan for all of them, you can interrupt and ask it to focus on just one function at a time.


### Combining _Escape_ with Memories

One of the most powerful applications of the escape technique is **fixing repetitive errors**. When `Claude` makes the same mistake repeatedly across different conversations, you can:

- Press _Escape_ to stop the current response
- Use the _#_ shortcut to add a memory about the correct approach
- Continue the conversation with the corrected information

This prevents `Claude` from making the same error in future conversations on your project.


### Rewinding Conversations

During long conversations, you might accumulate context that becomes irrelevant or distracting. For instance, if `Claude` encounters an error and spends time debugging it, that back-and-forth discussion might not be useful for the next task.

![[rewind_conversation.png]]

You can rewind the conversation by pressing _Escape_ twice. **This shows you all the messages you've sent, allowing you to jump back to an earlier point and continue from there.** This technique helps you:

- Maintain valuable context (like `Claude's` understanding of your codebase)
- Remove distracting or irrelevant conversation history
- Keep `Claude` focused on the current task

___

## SKILLS

*Skills* **are folders of instructions, scripts, and resources that** `Claude` **loads dynamically to improve performance on specialized tasks.** Think of them as expertise packages—they teach _`Claude`_ how to complete specific tasks in a repeatable way.


#### Types of Skills

There are two categories of _Skills_ you'll encounter:

- **Anthropic** _Skills_ are created and maintained by `Anthropic`. These include enhanced document creation capabilities for `Excel`, Word, PowerPoint, and PDF files. `Anthropic` _Skills_ are available to all paid users and _`Claude`_ invokes them automatically when relevant—you don't need to do anything special to use them.

- **Custom Skills** are ones you or your organization create for specialized workflows and domain-specific tasks. For example, you might create a _skill_ that applies your company's brand guidelines to presentations, structures meeting notes in a specific format, or executes your organization's data analysis workflows.


#### Enabling *Skills*

_Skills_ are currently available as a feature preview for users on Pro, Max, Team, and Enterprise plans. To use _Skills_, you'll need to have `Code` execution and file creation enabled, since _Skills_ require `Claude_'s` secure sandboxed computing environment to function.

Here's how to enable _Skills_:

1. Navigate to _Settings_ > _Capabilities_
2. Ensure that _Code execution and file creation_ is toggled on
3. Scroll to the _Skills_ section
4. Toggle individual _skills_ on or off as needed

For **Enterprise plans**, organization Owners must first enable both Code execution and _Skills_ in Admin settings before individual members can access them.

For **Team plans**, this feature preview is enabled by default at the organization level.

Once enabled, you'll see available _Skills_ listed in your settings, including `Anthropic's` built-in _Skills_ and any custom _Skills_ you've uploaded.


#### Using _Skills_ in practice

The beauty of _Skills_ is that you typically don't need to think about them. _`Claude`_ handles _skill_ selection automatically based on your request. Here are some examples of prompts that would invoke _Skills_:

- "Create an `Excel` spreadsheet tracking monthly expenses with formulas for totals"
- "Turn this meeting notes document into a PowerPoint presentation"
- "Generate a PDF report summarizing this data"
- "Build a financial model in `Excel` with scenario analysis"

When _`Claude`_ uses a _Skill_, you'll see it mentioned in _`Claude`_'s chain of thought as it works. The output will be a downloadable file you can save to your computer or directly to Google Drive.


#### Security considerations

Because _Skills_ can include executable code, it's important to use them thoughtfully:

- Only install custom _Skills_ from trusted sources
- `Anthropic's` built-in _Skills_ are tested and maintained by `Anthropic`
- Custom _Skills_ you upload are private to your individual account

If you're installing a custom _Skill_ from an external source, review its contents before use to understand what it does.


#### Creating custom _skills_

While `Anthropic's` built-in _Skills_ cover common document creation tasks, the real power of _Skills_ comes from creating your own. Custom _Skills_ let you teach _`Claude`_ your specific workflows, brand guidelines, and ways of working. So _`Claude`_ can apply that knowledge automatically whenever it's relevant.

The easiest way to create a custom _Skill_ is through conversation with _`Claude`_ itself. You don't need to write code or manually create files. _`Claude`_ handles the technical structure for you.

Here's how to create a Skill through conversation:

1. **Start a new chat** and tell _`Claude`_ what you want to create. For example: "I want to create a _skill_ for writing quarterly business reviews" or "I need a _skill_ that applies our brand guidelines to presentations."
2. **Answer** `Claude_'s` **questions.** _`Claude`_ will interview you about your workflow, asking things like: What should this _skill_ do? What makes good output for this type of work? Can you give examples of when you'd use this _skill_?
3. **Upload reference materials** if you have them. Templates, style guides, brand assets, or examples of work you're proud of all help _`Claude`_ understand exactly what you're looking for.
4. **Download your skill.** When finished, _`Claude`_ generates a downloadable ZIP file containing your properly structured _skill_.
5. **Upload to your settings.** Go to _Settings_ > _Capabilities_, scroll to the _Skills_ section, click "Upload _skill_", and select your ZIP file.

Your custom _Skill_ will appear in your _Skills_ list alongside `Anthropic's` built-in _Skills_. From that point forward, _`Claude`_ will automatically invoke it whenever you work on relevant tasks—no manual triggering needed.


#### Skills vs. Projects

You might be wondering—if both _skills_ and projects can be used to give more context to _`Claude`_, when should I use each? Think of it this way: **projects store knowledge, skills perform tasks**.

_Projects_ are knowledge hubs. They hold the reference materials _`Claude`_ needs to understand your work—project specs, meeting notes, research documents. When you upload files to a _project_, _`Claude`_ draws on that information across every conversation within that _project_.

_Skills_ are procedural machines. They encode **how** _`Claude`_ should execute a task—the specific steps, order of operations, and methodology you want followed every time. _Skills_ shine when you have repeatable workflows you want _`Claude`_ to run consistently.

The two features complement each other. A _skill_ can reference knowledge stored in a _project_. Your "customer call prep" _skill_ might pull from customer profiles uploaded to a project's knowledge base. The _project_ provides the **what** (information), the _skill_ provides the **how** (process).

|             | Projects                                                   | Skills                                                         |
| ----------- | ---------------------------------------------------------- | -------------------------------------------------------------- |
| Purpose     | Store knowledge _`Claude`_ references                      | Define processes _`Claude`_ executes                           |
| Best for    | Long-term context, reference materials, team collaboration | Repeatable workflows, multi-step tasks, consistent methodology |
| Example     | Customer hub, research buddy, feedback generator           | Brand guidelines, Blog drafting, PDF creation                  |
| Persistence | Knowledge available across all chats in the project        | Instructions applied when the skill is invoked                 |

___

## Claude Sessions