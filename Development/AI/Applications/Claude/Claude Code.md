---
aliases:
  - Claude Code
tags:
  - learning
---
**Date**: 19/mar/2026
**Update**: 22/mar/2026
**Sources**: [Claude Code](https://claude.com/product/claude-code), [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

**Tags:** #development #LLM #AI

**Related:** [[Artificial Intelligence]], [[Large Language Models]], [[Claude]], [[MCP servers]], [[Playwright]], [[NodeJS]], [[Python]], [[Bash]], [[TypeScript]], [[CI-CD]]

---

## Description

Claude Code is a **command-line AI assistant that uses`LLMs` to perform development tasks.**

---

## Key commands

Write here...

---

## Details

Claude Code comes with a comprehensive set of built-in tools that handle common development tasks like reading files, writing code, running commands, and managing directories. But what makes Claude Code truly powerful is how intelligently it combines these tools to tackle complex, multi-step problems.


### Tools available in Claude Code

![[Tools_with_claude_code.png]]

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

```bash title:MacOS_Homebrew
$ _brew install --cask claude-code_
```

```bash title:MacOS,Linux,WSL
$ _curl -fsSL https://claude.ai/install.sh | bash_
```

```bash title:Windows_CMD
$ _curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd_
```

2. After installation, run _claude_ at your terminal. The first time you run this command you will be prompted to authenticate

>[!info] **NOTE:** if https://claude.ai/install.sh doesn't work, try with [this URL](https://storage.googleapis.com/claude-code-dist-86c565f3-f756-42ad-8dfa-d59b1c096819/claude-code-releases/bootstrap.sh)

If you're making use of AWS Bedrock or Google Cloud Vertex, there is some additional setup:

- Special directions for [AWS Bedrock](https://code.claude.com/docs/en/amazon-bedrock)
- Special directions for [Google Cloud Vertex](https://code.claude.com/docs/en/google-vertex-ai)

___

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


### When to Use These Techniques

These conversation control techniques are particularly valuable during:

- Long-running conversations where context can become cluttered
- Task transitions where previous context might be distracting
- Situations where `Claude` repeatedly makes the same mistakes
- Complex projects where you need to maintain focus on specific components

By using _escape_, double-tap _escape_, _/compact_, and _/clear_ strategically, you can keep `Claude` focused and productive throughout your development workflow. **These aren't just convenience features. They're essential tools for maintaining effective AI-assisted development sessions.**

___

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

___

## `MCP servers` with Claude Code

**You can extend Claude Code's capabilities by adding `MCP (Model Context Protocol) servers`. These servers run either remotely or locally on your machine and provide `Claude` with new tools and abilities it wouldn't normally have.**

One of the most popular MCP servers is `Playwright`, which gives `Claude` the ability to control a web browser. This opens up powerful possibilities for web development workflows.

![[playwright_mcp.png]]


### Installing the Playwright MCP Server

To add the `Playwright` server to Claude Code, run this command in your terminal (not inside Claude Code):

```bash title:bash
$ claude mcp add playwright npx @playwright/mcp@latest
```

This command does two things:

- Names the MCP server "playwright"
- Provides the command that starts the server locally on your machine


### Managing Permissions

When you first use MCP server tools, `Claude` will ask for permission each time. If you get tired of these permission prompts, you can pre-approve the server by editing your settings.

Open the _.claude/settings.local.json_ file and add the server to the allow array:

```json title:.claude/settings.local.json
{
    "other_stuff": {},

    "permissions": {
        "allow": ["mcp__playwright"],
        "deny": []
    },

    "other_stuff": {},
}
```

Note the double underscores in `mcp__playwright`. This allows `Claude` to use the `Playwright` tools without asking for permission every time.


### Practical Example: Improving Component Generation

Here's a real-world example of how the `Playwright` `MCP server` can improve your development workflow. Instead of manually testing and tweaking prompts, you can have `Claude`:

1. Open a browser and navigate to your application
2. Generate a test component
3. Analyze the visual styling and code quality
4. Update the generation prompt based on what it observes
5. Test the improved prompt with a new component

For instance, you might ask `Claude` to:

```bash title:Claude
> Navigate to localhost:3000, generate a basic component, review the styling, and update the generation prompt at @src/lib/prompts/generation.tsx to produce better components going forward.
```

Claude will use the browser tools to interact with your app, examine the generated output, and then modify your prompt file to encourage more original and creative designs.


### Results and Benefits

In practice, this approach can lead to significantly better results. Instead of generic purple-to-blue gradients and standard Tailwind patterns, `Claude` might update prompts to encourage:

- Warm sunset gradients (orange-to-pink-to-purple)
- Ocean depth themes (teal-to-emerald-to-cyan)
- Asymmetric designs and overlapping elements
- Creative spacing and unconventional layouts

The key advantage is that `Claude` can see the actual visual output, not just the code, which allows it to make much more informed decisions about styling improvements.


### Exploring Other `MCP Servers`

`Playwright` is just one example of what's possible with `MCP servers`. The ecosystem includes servers for:

- Database interactions
- API testing and monitoring
- File system operations
- Cloud service integrations
- Development tool automation

Consider exploring `MCP servers` that align with your specific development needs. They can transform `Claude` from a code assistant into a comprehensive development partner that can interact with your entire toolchain.

___

## Hooks

_Hooks_ **allow you to run commands before or after `Claude` attempts to run a tool.** They're incredibly useful for implementing automated workflows like running code formatters after file edits, executing tests when files change, or blocking access to specific files.


### How Hooks Work

To understand _hooks_, let's first review the normal flow when you interact with Claude Code. When you ask `Claude` something, your query gets sent to the `Claude` model along with tool definitions. `Claude` might decide to use a tool by providing a formatted response, and then Claude Code executes that tool and returns the result.

_Hooks_ **insert themselves into this process, allowing you to execute code just before or just after the tool execution happens.**

![[hooks.png]]

There are two types of _hooks_:

- **PreToolUse** _hooks_: Run **before** a tool is called
- **PostToolUse** _hooks_: Run **after** a tool is called

![[hooks2.png]]


### _Hooks_ Configuration

_Hooks_ are defined in Claude settings files. You can add them to:

- **Global**: _~/.claude/settings.json_ (affects all projects)
- **Project**: _.claude/settings.json_ (shared with team)
- **Project (not committed)**: _.claude/settings.local.json_ (personal settings)

You can write _hooks_ by hand in these files or use the _/hooks_ command inside Claude Code.

```json title:.claude/settings.local.json
{
    "other_stuff": {},

    "hooks": {
        "PreToolUse": [ // Runs before a tool is called
            {
                "matcher": "Read|Grep",
                "hooks": [
                    {
                        "type": "command",
                        "command": "node /home/hooks/read_hook.js"
                    }
                ]
            }
        ],
        "PostToolUse": [ // Runs after a tool is called
            {
                "matcher": "Write|Edit|MultiEdit",
                "hooks": [
                    {
                        "type": "command",
                        "command": "node /home/hooks/edit_hook.js"
                    }
                ]
            }
        ],
    },

    "other_stuff": {},
}
```


### _PreToolUse Hooks_

_PreToolUse hooks_ run before a tool is executed. They include a matcher that specifies which tool types to target (as in the example above).

Before the _Read_ tool is executed, this configuration runs the specified command. Your command receives details about the tool call `Claude` wants to make, and you can:

- Allow the operation to proceed normally
- Block the tool call and send an error message back to `Claude`


### _PostToolUse Hooks_

_PostToolUse hooks_ run after a tool has been executed. Here's an example that triggers after write, edit, or multi-edit operations (as in the example above).

Since the tool call has already occurred, _PostToolUse hooks_ can't block the operation. However, they can:

- Run follow-up operations (like formatting a file that was just edited)
- Provide additional feedback to `Claude` about the tool use


### Practical Applications

Here are some common ways to use _hooks_:

- **Code formatting**: Automatically format files after `Claude` edits them
- **Testing**: Run tests automatically when files are changed
- **Access control**: Block `Claude` from reading or editing specific files
- **Code quality**: Run linters or type checkers and provide feedback to `Claude`
- **Logging**: Track what files `Claude` accesses or modifies
- **Validation**: Check naming conventions or coding standards

**The key insight is that** _hooks_ **let you extend Claude Code's capabilities by integrating your own tools and processes into the workflow.** _PreToolUse hooks_ give you control over what `Claude` can do, while _PostToolUse hooks_ let you enhance what `Claude` has done.

___

### Building a _Hook_

Creating a _hook_ involves four main steps:

![[building_hooks_steps.png]]

1. **Decide on a** _PreToolUse_ **or** _PostToolUse_ _hook_: _PreToolUse hooks_ can prevent tool calls from executing, while _PostToolUse hooks_ run after the tool has already been used
2. **Determine which type of tool calls you want to watch for**: You need to specify exactly which tools should trigger your _hook_
3. **Write a command that will receive the tool call**: This command gets JSON data about the proposed tool call via standard input
4. **If needed, command should provide feedback to `Claude`**: Your command's exit code tells `Claude` whether to allow or block the operation


#### Available Tools

Claude Code provides several built-in tools that you can monitor with _hooks_:

![[tools.png]]

To see exactly which tools are available in your current setup, you can ask `Claude` directly for a list. This is especially useful since the available tools can change when you add custom `MCP servers`.

```bash title:Claude
> List out the names of all the tools you have access to, bullet point list.
```


#### Tool Call Data Structure

When your _hook_ command executes, `Claude` sends JSON data through standard input containing details about the proposed tool call:

![[tool_data_call.png]]

```json
{
    "session_id": "2d6a1e4d-6...",
    "transcript_path": "/Users/sg/...",
    "hook_event_name": "PreToolUse",
    "tool_name": "Read",
    "tool_input": {
        "file_path": "/code/queries/.env"
    }
}
```

Your command reads this JSON from standard input, parses it, and then decides whether to allow or block the operation based on the tool name and input parameters


#### Exit Codes and Control Flow

Your _hook_ command communicates back to `Claude` through exit codes:

![[hooks_exit_codes.png]]

- **Exit Code 0**: Everything is fine, allow the tool call to proceed
- **Exit Code 2**: Block the tool call (_PreToolUse hooks_ only)

When you exit with code 2 in a _PreToolUse hook_, any error messages you write to standard error will be sent to `Claude` as feedback, explaining why the operation was blocked.


#### Example Use Case

A common use case is preventing `Claude` from reading sensitive files like _.env_ files. Since both the _Read_ and _Grep_ tools can access file contents, you'd want to monitor both tool types and check if they're trying to access restricted file paths.

This approach gives you complete control over `Claude's` file system access while providing clear feedback about why certain operations are restricted.

___

### Implementing a _hook_

Let's build a custom _hook_ to prevent `Claude` from reading sensitive files like _.env_. This is a practical example of how _hooks_ can protect your environment variables and other confidential data during development sessions.


#### Setting Up the _Hook_ Configuration

First, we need to configure our hook in the settings file. Open your _.claude/settings.local.json_ file and locate the hooks section. We'll create a _PreToolUse hook_ since we want to intercept tool calls before they execute.

The configuration requires two key pieces:

- **Matcher**: specifies which tools to watch for
- **Command**: the script that runs when those tools are called

For the matcher, we want to catch both _read_ and _grep_ operations that might access the _.env_ file:

```
"matcher": "Read|Grep"
```

The pipe symbol (_|_) acts as an _OR_ operator, so this will trigger on either tool. For the command, we'll point to a `Node.js` script:

```
"command": "node ./hooks/read_hook.js"
```


#### Understanding Tool Call Data

**When** `Claude` **attempts to use a tool, your** _hook_ **receives detailed information about that call through standard input as JSON.** This data includes:

- Session ID and transcript path
- _Hook_ event name (_PreToolUse_ in our case)
- Tool name (_Read_, _Grep_, etc.)
- Tool input parameters, including the file path

Your _hook_ script processes this data and can either allow the operation to continue or block it by exiting with a specific code.


#### Implementing the _Hook_ Script

##### Using `NodeJS`

The _hook_ script needs to read the tool call data from standard input and check if `Claude` is trying to access the _.env_ file. Here's the core logic:

```javascript title:read_hook.js
async function main() {
    const chunks = [];
    for await (const chunk of process.stdin) {
        chunks.push(chunk);
    }

    const toolArgs = JSON.parse(Buffer.concat(chunks).toString());

    // Extract the file path Claude is trying to read
    const readPath = toolArgs.tool_input?.file_path || toolArgs.tool_input?.path || "";

    // Check if Claude is trying to read the .env file
    if (readPath.includes('.env')) {
        console.error("You cannot read the .env file");
        process.exit(2);
    }
}
```

The script checks for _.env_ in the file path and blocks the operation if found. When you exit with _code 2_, `Claude` receives an error message and understands the operation was blocked by a _hook_.


##### Using `Python`

Claude Code's _hooks_ work with every executable script with access to _stdin_ values and returns _stdout_ / _stderr_ values too. So the `Python` equivalent to the previous `NodeJS` _hook_ script is:

```json title:.claude/settings.local.json
{
    "other_stuff": {},

    "hooks": {
        "PreToolUse": [ // Runs before a tool is called
            {
                "matcher": "Read|Grep",
                "hooks": [
                    {
                        "type": "command",
                        "command": "python /home/hooks/read_hook.py"
                    }
                ]
            }
        ]
    },

    "other_stuff": {},
}
```

```python title:read_hook.py
#!/usr/bin/env python3

from json import loads
from sys import exit, stdin, stderr


def main():
    raw = stdin.read()
    tool_args = loads(raw)

    # Extract the file path Claude is trying to read
    read_path = (
        tool_args.get("tool_input", {}).get("file_path")
        or tool_args.get("tool_input", {}).get("path")
        or ""
    )

    # Check if Claude is trying to read the .env file
    if ".env" in read_path:
        print(f"You cannot read the .env file: {stderr}")
        exit(2)


if __name__ == "__main__":
    main()
```

**You must give execution permissions to the script:**

```bash title:bash
chmod +x /home/hooks/read_hook.py
```


##### Using Another

Claude Code simply passes the tool's context through _stdin_ by a JSON and reads the result, so the important thing for **any language** are the exit codes:

* **exit 0**: allows the tool call
* **exit 1**: raises an error during the _hook_ execution (tool call is not blocked but it raises an error)
- **exit 2**: blocks the tool call (the message within _stderr_ is shown to the user and sent to Claude Code to analyze)

With this, you can use `NodeJS`, `Python`, `Bash`, or any other language that can run as a process. Here is an example using `Bash`:

```bash title:read_hook.sh
#!/bin/bash
input=$(cat)
path=$(echo "$input" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('file_path',''))")

if [[ "$path" == *".env"* ]]; then
  echo "You cannot read the .env file" >&2
  exit 2
fi
```


#### Testing Your Hook

**After saving your configuration and** _hook_ **script, restart Claude Code for the changes to take effect.** Then test it by asking `Claude` to read your _.env_ file.

When `Claude` attempts the read operation, your hook will intercept it and return an error message. `Claude` will recognize that the operation was blocked and explain this to you, often mentioning that a read hook prevented access to the file.

The same protection works for _grep_ operations - if `Claude` tries to search within the _.env_ file, the hook will block that as well.


#### Key Benefits

This approach provides several advantages:

- **Proactive protection**: blocks access before sensitive data is read
- **Transparent operation**: `Claude` understands why the operation failed
- **Flexible matching**: works with multiple tools (_read_, _grep_, etc.)
- **Clear feedback**: provides meaningful error messages

While this specific example focuses on _.env_ files, the same pattern can protect any sensitive files or directories in your project. You can extend the logic to check for multiple file patterns or implement more sophisticated access controls based on your security requirements.

___

## _SDKs_

The Claude Code _SDK_ lets you run Claude Code programmatically from within your own applications and scripts. It's available for `TypeScript`, `Python`, and via the _CLI_, giving you the same Claude Code functionality you use at the terminal but integrated into larger workflows.

![[sdk.png]]

**The** _SDK_ **runs the exact same Claude Code you're already familiar with.** It has access to all the same tools and will use them to complete whatever task you give it. This makes it particularly powerful for automation and integration scenarios.


### Key Features

- Runs Claude Code programmatically
- Same Claude Code functionality as the terminal version
- Inherits all settings from Claude Code instances in the same directory
- Read-only permissions by default
- Most useful as part of larger pipelines or tools


### Basic Usage

Here's a simple `TypeScript` example that asks `Claude` to analyze code for duplicate queries:

```typescript title:integration.ts
import { query } from "@anthropic-ai/claude-code";

const prompt = "Look for duplicate queries in the ./src/queries dir";

for await (const message of query({ prompt, })) {
    console.log(JSON.stringify(message, null, 2));
}
```

When you run this code, you'll see the raw conversation between your local Claude Code and the `Claude` language model, message by message. The final message contains `Claude's` complete response.

### Permissions and Tools

**By default, the** _SDK_ **only has read-only permissions.** It can read files, search directories, and perform grep operations, but **it cannot write, edit, or create files.**

To enable write permissions, you can add the _allowedTools_ option to your query:

```typescript title:integration.ts
for await (
    const message of query({
        prompt,
        options: {
            allowedTools: ["Edit"]
        }
    })
) {
    console.log(JSON.stringify(message, null, 2));
}
```

Alternatively, you can configure permissions in your settings file within the _.claude_ directory for project-wide access.

### Practical Applications

The Claude Code _SDK_ shines when integrated into larger development workflows. Consider using it for:

- Git hooks that automatically review code changes
- Build scripts that analyze and optimize code
- Helper commands for code maintenance tasks
- Automated documentation generation
- Code quality checks in `CI/CD` pipelines

The _SDK_ essentially lets you add `AI-powered` intelligence to any part of your development process where programmatic access would be valuable.

## Claude Sessions

- [[2026-04-13 - Qué Es Python Explicamelo Para Entenderlo Como Fuera La]] — 13/abr/2026

___
