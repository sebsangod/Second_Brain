---
aliases:
  - LLM
tags:
  - learning
  - dev/ai/llm
date: 2026-04-26
---
**Sources**: [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action), [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol/)

**Related:** [[Claude]], [[Claude Code]], [[Skills]], [[Playwright]], [[HTTP]], [[WebSocket]]

---

## Description

_Model Context Protocol (MCP)_ is a communication layer that provides `Claude` with **context and tools without requiring you to write a bunch of tedious integration code.** Think of it as a way to shift the burden of tool definitions and execution away from your server to specialized MCP servers.

_MCP_ **is an open standard that lets** ``Claude Code`` **connect to external tools and data sources**. When you ask a question, ``Claude`` automatically understands when it should use those tools to better handle your query.

**A lot of your context lives outside your codebase — in databases, productivity apps, or public repositories.** _MCP_ bridges that gap.

![[mcp_basic_architecture.png]]

When you first encounter _MCP_, you'll see diagrams showing the basic architecture: an _MCP Client_ (your server) connecting to _MCP Servers_ that contain tools, prompts, and resources. Each MCP Server acts as an interface to some outside service.

> [!warning]
> _MCP servers_ load all of their available tools into context by default, even when you're not using them. **If you have servers configured for things unrelated to the current project, consider turning them off.** You can also try ``Skills``, which work similarly to _MCP_ servers but don't load everything into context upfront.

---

## Key concepts

_MCP Servers_ **provide access to data or functionality implemented by outside services. They act as specialized interfaces that expose tools, prompts, and resources in a standardized way.**

---

## Details

### The Problem _MCP_ Solves

Let's say you're building a chat interface where users can ask `Claude` about their GitHub data. A user might ask "What open pull requests are there across all my repositories?" To handle this, `Claude` needs tools to access GitHub's API.

**GitHub has massive functionality** - repositories, pull requests, issues, projects, and tons more. Without _MCP_, **you'd need to create an incredible number of tool schemas and functions to handle all of GitHub's features.**

![[mcp_problem_it_solves_example.png]]

This means writing, testing, and maintaining all that integration code yourself. **That's a lot of effort and ongoing maintenance burden.**

### How _MCP_ Works

_MCP_ **shifts this burden by moving tool definitions and execution from your server to dedicated** _MCP servers_. Instead of you authoring all those GitHub tools, **an** _MCP Server_ **for GitHub handles it.**

![[mcp_github_server.png]]

**The** _MCP Server_ **wraps up tons of functionality around GitHub and exposes it as a standardized set of tools.** Your application connects to this _MCP server_ instead of implementing everything from scratch.

In our GitHub example, the _MCP Server_ for GitHub contains tools like _get_repos()_ and connects directly to GitHub's API. Your server communicates with the _MCP server_, which handles all the GitHub-specific implementation details.

![[mcp_problem_it_solves_example2.png]]


### _MCP Clients_

The _MCP client_ serves as the **communication bridge between your server and** _MCP servers_. It's your access point to all the tools that an _MCP server_ provides, handling the message exchange and protocol details so your application doesn't have to.


#### **Transport Agnostic** Communication

One of _MCP's_ key strengths is being transport agnostic: a fancy way of saying **the client and server can communicate over different protocols** depending on your setup.

The most common setup runs both the _MCP client and server_ on the same machine, communicating through **standard input/output**. But you can also connect them over:
- `HTTP`
- `WebSockets`
- Various other network protocols


### _MCP_ Message Types

**Once connected, the client and server exchange specific message types defined in the MCP specification**. The main ones you'll work with are:

- **ListToolsRequest** / **ListToolsResult**: The client asks the server "what tools do you provide?" and gets back a list of available tools.

- **CallToolRequest** / **CallToolResult:** The client asks the server to run a specific tool with given arguments, then receives the results.


### How It All Works Together

The example below shows **how a user query flows through the entire system**: from your server, through the _MCP client_, to external services like GitHub, and back to `Claude`.

Let's say a user asks "What repositories do I have?" Here's the step-by-step flow:

1. **User Query:** The user submits their question to your server
2. **Tool Discovery:** Your server needs to know what tools are available to send to `Claude`
3. **List Tools Exchange:** Your server asks the _MCP client_ for available tools
4. **MCP Communication:** The _MCP client_ sends a _ListToolsRequest_ to the _MCP server_ and receives a _ListToolsResult_
5. **Claude Request:** Your server sends the user's query plus the available tools to `Claude`
6. **Tool Use Decision:** `Claude` decides it needs to call a tool to answer the question
7. **Tool Execution Request:** Your server asks the _MCP client_ to run the tool `Claude` specified
8. **External API Call:** The _MCP client_ sends a _CallToolRequest_ to the _MCP server_, which makes the actual GitHub API call
9. **Results Flow Back:** GitHub responds with repository data, which flows back through the _MCP server_ as a _CallToolResult_
10. **Tool Result to Claude:** Your server sends the tool results back to `Claude`
11. **Final Response:** `Claude` formulates a final answer using the repository data
12. **User Gets Answer:** Your server delivers `Claude's` response back to the user

![[mcp_full_flow.png]]


### _MCP servers_ with `Claude Code`

**You can extend** `Claude Code's` capabilities by adding _MCP servers_. These servers run either remotely or locally on your machine and provide `Claude` with new tools and abilities it wouldn't normally have.**

One of the most popular _MCP servers_ is `Playwright`, which gives `Claude` the ability to control a web browser. This opens up powerful possibilities for web development workflows.

![[playwright_mcp.png]]


#### Installing the Playwright MCP Server

To add the `Playwright` server to Claude Code, run this command in your terminal (not inside Claude Code):

```bash title:bash
$ claude mcp add playwright npx @playwright/mcp@latest
```

This command does two things:

- Names the MCP server "playwright"
- Provides the command that starts the server locally on your machine


#### Managing Permissions

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


#### Practical Example: Improving Component Generation

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


#### Results and Benefits

In practice, this approach can lead to significantly better results. Instead of generic purple-to-blue gradients and standard Tailwind patterns, `Claude` might update prompts to encourage:

- Warm sunset gradients (orange-to-pink-to-purple)
- Ocean depth themes (teal-to-emerald-to-cyan)
- Asymmetric designs and overlapping elements
- Creative spacing and unconventional layouts

The key advantage is that `Claude` can see the actual visual output, not just the code, which allows it to make much more informed decisions about styling improvements.


#### Exploring Other `MCP Servers`

`Playwright` is just one example of what's possible with `MCP servers`. The ecosystem includes servers for:

- Database interactions
- API testing and monitoring
- File system operations
- Cloud service integrations
- Development tool automation

Consider exploring `MCP servers` that align with your specific development needs. They can transform `Claude` from a code assistant into a comprehensive development partner that can interact with your entire toolchain.

---

## Claude Sessions
