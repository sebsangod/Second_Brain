---
aliases:
  - MCP
tags:
  - learning
  - dev/backend
date: 2026-04-28
---
**Sources**: [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk), [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol/)

**Related:** [[Development/AI/Applications/MCP/MCP|MCP]], [[Python|Python]], [[uv]], [[Large Language Models|Large Language Models]], [[Decorators]], [[Claude]], [[Tools]], [[Development/Languages/Python/Libraries/MCP/Resources|Resources]], [[Prompts]]

---

## Description

The `MCP` allows applications to provide context for `LLMs` in a standardized way, separating the concerns of providing context from the actual `LLMs` interaction. This `Python` _SDK_ implements the full `MCP` specification, making it easy to:

---

## Key Concepts

- Build `MCP clients` that can connect to any `MCP server`
- Create `MCP servers` that expose resources, prompts and tools
- Use standard transports like stdio, SSE, and Streamable HTTP
- Handle all `MCP protocol` messages and lifecycle events

---

## Key Benefits

- No manual JSON schema writing required
- Type hints provide automatic validation
- Clear parameter descriptions help `Claude` understand tool usage
- Error handling integrates naturally with `Python` exceptions
- Tool registration happens automatically through decorators

The _MCP Python SDK_ **transforms tool creation from a complex schema-writing exercise into simple Python function definitions.** This approach makes it much easier to build and maintain MCP servers while ensuring `Claude` receives properly formatted tool specifications.

---

## Details

### Installation

```bash title:uv
$ uv add "mcp[cli]"
```


### Instantiation

```python title:mcp_server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("DocumentMCP", log_level="ERROR")

if __name__ == "__main__":
	mcp.run(transport="stdio")

```


### MCP Server Primitives

![[mcp_server_primitives.png]]


#### Tools: Model-Controlled

_Tools_ **are controlled entirely by** ``Claude``. The AI model decides when to call these functions, and the results are used directly by ``Claude`` to accomplish tasks.

_Tools_ **are perfect for giving** ``Claude`` **additional capabilities it can use autonomously.** When you ask ``Claude`` to "calculate the square root of 3 using JavaScript," it's ``Claude`` that decides to use a JavaScript execution tool to run the calculation.


#### Resources: App-Controlled

_Resources_ **are controlled by your application code**. Your app decides when to fetch resource data and how to use it - typically for UI elements or to add context to conversations.

In our project, we used resources in two ways:

- Fetching data to populate autocomplete options in the UI
- Retrieving content to augment prompts with additional context

Think of the "Add from Google Drive" feature in ``Claude's`` interface, the application code determines which documents to show and handles injecting their content into the chat context.


#### Prompts: User-Controlled

_Prompts_ **are triggered by user actions**. Users decide when to run these predefined workflows through UI interactions like button clicks, menu selections, or slash commands.

Prompts are ideal for implementing workflows that users can trigger on demand. In ``Claude's`` interface, those workflow buttons below the chat input are examples of _prompts_ - predefined, optimized workflows that users can start with a single click.


#### Choosing the Right Primitive

Here's a quick decision guide:

- **Need to give Claude new capabilities?** Use _tools_
- **Need to get data into your app for UI or context?** Use _resources_
- **Want to create predefined workflows for users?** Use _prompts_

You can see all three primitives in action in ``Claude's`` official interface. The workflow buttons demonstrate _prompts_, the Google Drive integration shows _resources_ in action, and when ``Claude`` executes code or performs calculations, it's using _tools_ behind the scenes.

These are high-level guidelines to help you choose the right primitive for your specific use case. Each serves a different part of your application stack: **tools serve the model, resources serve your app, and prompts serve your users.**

---

## Claude Sessions
