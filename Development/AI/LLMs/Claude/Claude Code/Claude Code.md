---
aliases:
  - Claude Code
tags:
  - learning
  - dev/ai/llm
date: 2026-05-26
---
**Sources**: [Claude Code](https://claude.com/product/claude-code), [Claude Code 101](https://anthropic.skilljar.com/claude-code-101), [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action), [Exploring the .claude directory](https://code.claude.com/docs/en/claude-directory)

**Related:** [[Artificial Intelligence]], [[Large Language Model]], [[Claude]], [[Context Window]], [[CLAUDE File]], [[Development/AI/LLMs/MCP/MCP|MCP]], [[NodeJS]], [[Python]], [[Bash]], [[TypeScript]], [[CI-CD]]

---

## Description

*Claude Code* is a **command-line AI assistant that uses** `LLMs` **to perform development tasks.**

_Claude Code_ is an agentic coding tool that understands your codebase, edits your files, runs commands, and integrates with your existing developer tools to help you get things done faster. It's available in your terminal, Visual Studio Code, the Claude Desktop app, on the web, and in JetBrains IDEs.

---

## Usage examples

![[Optimization_task.png]]

![[Data_analysis_task.png]]

![[UI_styling_task.png]]

![[Github_integration.png]]

---

## Installation

### Terminal (recommended)

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

### Visual Studio Code

  
Open your Extensions panel and search for "Claude Code." Look for the extension by Anthropic with the blue verification check. Hit install.

After installation, you may need to restart VS Code. Once it's running, open the command palette with `Ctrl/Cmd + Shift + P` and search for "Claude Code Open in New Tab." You can also click the Claude logo if it's visible in your sidebar.

![[Claude Code-1793680703146.webp]]

---

## Details

### Permission Modes

- **Manual:** ``Claude`` asks for explicit permission before editing a file or running a shell command.
- **Auto-accept:** Files are edited without asking, and common filesystem commands like _mkdir_, _mv_, and _rm_ run without asking too. Other commands still require approval.
- **Plan mode:** ``Claude`` reads files and runs shell commands to explore, without editing your source files, and compiles a plan of action before starting any work.
- **Auto mode:** ``Claude`` works without routine permission prompts while a classifier checks most actions in the background, targeted at blocking actions that are irreversible, destructive, or aimed outside your environment. When something is blocked, ``Claude`` usually finds a safer approach or asks you for the go-ahead.

### Making changes

When working with `Claude` in your development environment, you'll often need to make changes to existing projects. This section covers practical techniques for implementing changes effectively, including visual communication with screenshots and leveraging `Claude's` advanced reasoning capabilities.


#### Using Screenshots for Precise Communication

**One of the most effective ways to communicate with `Claude` is through screenshots**. When you want to modify a specific part of your interface, taking a screenshot helps `Claude` understand exactly what you're referring to.

To paste a screenshot into `Claude`, use _Ctrl + V_ (not _Cmd + V_ on macOS). This keyboard shortcut is specifically designed for pasting screenshots into the chat interface. Once you've pasted the image, you can ask `Claude` to make specific changes to that area of your application.


#### Planning Mode

For more complex tasks that require extensive research across your codebase, you can enable _Planning Mode_. **This feature makes `Claude` do thorough exploration of your project before implementing changes.**

Enable _Planning Mode_ by pressing `Shift + Tab` twice (or once if you're already auto-accepting edits). In this mode, `Claude` will:

- Read more files in your project
- Create a detailed implementation plan
- Show you exactly what it intends to do
- Wait for your approval before proceeding

**This gives you the opportunity to review the plan and redirect `Claude` if it missed something important or didn't consider a particular scenario.**


#### Thinking Modes

`Claude` offers different levels of reasoning through _thinking_ modes. **These allow `Claude` to spend more time reasoning about complex problems before providing solutions.**

The available thinking modes include:

- **Think**: Basic reasoning
- **Think more**: Extended reasoning
- **Think a lot**: Comprehensive reasoning
- **Think longer**: Extended time reasoning
- **Ultrathink**: Maximum reasoning capability

Each mode gives `Claude` progressively more tokens to work with, allowing for deeper analysis of challenging problems.


#### When to Use Planning vs Thinking

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

#### Controlling context

When working with `Claude` on complex tasks, you'll often need to guide the conversation to keep it focused and productive. **There are several techniques you can use to control the flow of your conversation and help `Claude` stay on track.**

![[controlling_context.png]]


#### Interrupting `Claude` with _Escape_

Sometimes `Claude` starts heading in the wrong direction or tries to tackle too much at once. **You can press the _Escape key_ to stop `Claude` mid-response, allowing you to redirect the conversation.**

This is particularly useful when you want `Claude` to focus on one specific task instead of trying to handle multiple things simultaneously. For example, if you ask `Claude` to write tests for multiple functions and it starts creating a comprehensive plan for all of them, you can interrupt and ask it to focus on just one function at a time.


#### Combining _Escape_ with Memories

One of the most powerful applications of the escape technique is **fixing repetitive errors**. When `Claude` makes the same mistake repeatedly across different conversations, you can:

- Press _Escape_ to stop the current response
- Use the _#_ shortcut to add a memory about the correct approach
- Continue the conversation with the corrected information

This prevents `Claude` from making the same error in future conversations on your project.


#### Rewinding Conversations

During long conversations, you might accumulate context that becomes irrelevant or distracting. For instance, if `Claude` encounters an error and spends time debugging it, that back-and-forth discussion might not be useful for the next task.

![[rewind_conversation.png]]

You can rewind the conversation by pressing _Escape_ twice. **This shows you all the messages you've sent, allowing you to jump back to an earlier point and continue from there.** This technique helps you:

- Maintain valuable context (like `Claude's` understanding of your codebase)
- Remove distracting or irrelevant conversation history
- Keep `Claude` focused on the current task

### Skills vs. Projects

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