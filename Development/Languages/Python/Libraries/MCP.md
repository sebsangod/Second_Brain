---
aliases:
  - MCP
tags:
  - learning
  - dev/backend
date: 2026-04-28
---
**Sources**: [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)

**Related:** [[Development/AI/Applications/MCP/MCP|MCP]], [[Python|Python]], [[uv]], [[Large Language Models|Large Language Models]]

---

## Description

The `MCP` allows applications to provide context for `LLMs` in a standardized way, separating the concerns of providing context from the actual `LLMs` interaction. This `Python` _SDK_ implements the full `MCP` specification, making it easy to:

---

## Key concepts

- Build `MCP clients` that can connect to any `MCP server`
- Create `MCP servers` that expose resources, prompts and tools
- Use standard transports like stdio, SSE, and Streamable HTTP
- Handle all `MCP protocol` messages and lifecycle events

---

## Details

### Installation

```bash title:uv
$ uv add "mcp[cli]"
```


### Instanciation

---

## Claude Sessions
