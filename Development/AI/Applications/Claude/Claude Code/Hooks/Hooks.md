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

_Hooks_ **allow you to run commands before or after `Claude` attempts to run a tool.** They're incredibly useful for implementing automated workflows like running code formatters after file edits, executing tests when files change, or blocking access to specific files.

---

## Key Benefits

This approach provides several advantages:

- **Proactive protection**: blocks access before sensitive data is read
- **Transparent operation**: `Claude` understands why the operation failed
- **Flexible matching**: works with multiple tools (_read_, _grep_, etc.)
- **Clear feedback**: provides meaningful error messages

While this specific example focuses on _.env_ files, the same pattern can protect any sensitive files or directories in your project. You can extend the logic to check for multiple file patterns or implement more sophisticated access controls based on your security requirements.

---

## Details

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

---

## Claude Sessions
