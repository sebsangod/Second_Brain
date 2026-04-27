---
aliases:
  - LLM
tags:
  - learning
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

**Related:** [[Claude]], [[Claude Code]], [[Skills]]****

---

## Description

_Model Context Protocol (MCP)_ **is an open standard that lets** ``Claude Code`` **connect to external tools and data sources**. When you ask a question, ``Claude`` automatically understands when it should use those tools to better handle your query.

**A lot of your context lives outside your codebase — in databases, productivity apps, or public repositories.** _MCP_ bridges that gap.

> [!warning]
> _MCP servers_ load all of their available tools into context by default, even when you're not using them. **If you have servers configured for things unrelated to the current project, consider turning them off.** You can also try ``Skills``, which work similarly to _MCP_ servers but don't load everything into context upfront.

---

## Key concepts

Write here...

---

## Details

### `MCP servers` with Claude Code

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

---

## Claude Sessions
