---
aliases:
  - MCP
tags:
  - learning
  - dev/backend
date: 2026-05-01
---
**Sources**: [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol/)

**Related:** [[Development/Languages/Python/Libraries/SDKs/MCP/MCP|MCP]], [[Development/AI/Applications/MCP/MCP|MCP]], [[Pydantic|Pydantic]], [[Claude|Claude]]

---

## Details

### Server Side

#### Tool Definition with Decorators

The _SDK_ uses `decorators` to define tools. Instead of writing JSON schemas manually, you can use `Python` type hints and field descriptions. The _SDK_ automatically generates the proper schema that `Claude` can understand.


#### Creating a Document Reader Tool

The first tool reads document contents by ID. Here's the complete implementation:

```python title:mcp_server.py
from pydantic import Field

docs = {
	"deposition.md": "This deposition covers...",
	"report.pdf": "The report details the...",
	"financials.docx": "These financials outline...",
	"outlook.pdf": "This document presents the...",
	"plan.md": "The plan outlines the steps for...",
	"spec.txt": "These specifications define the...",
}

@mcp.tool(
	name="read_doc_contents",
	description="Read the contents of a document and return it as a string."
)
def read_document(
	doc_id: str = Field(description="Id of the document to read")
):
	if doc_id not in docs:
		raise ValueError(f"Doc with id {doc_id} not found")

	return docs[doc_id]

```

The `decorator` specifies the tool name and description, while the function parameters define the required arguments. The _Field_ class from `Pydantic` provides argument descriptions that help `Claude` understand what each parameter expects.


#### Building a Document Editor Tool

The second tool performs simple find-and-replace operations on documents:

```python title:mcp_server.py
@mcp.tool(
	name="edit_document",
	description="Edit a document by replacing a string in the documents content with a new string."
)
def edit_document(
	doc_id: str = Field(description="Id of the document that will be edited"),
	old_str: str = Field(description="The text to replace. Must match exactly, including whitespace."),
	new_str: str = Field(description="The new text to insert in place of the old text.")
):
	if doc_id not in docs:
		raise ValueError(f"Doc with id {doc_id} not found")

	docs[doc_id] = docs[doc_id].replace(old_str, new_str)

```

This tool takes three parameters: the document ID, the text to find, and the replacement text. The implementation includes error handling for missing documents and performs a straightforward string replacement.


#### Server _Inspector_

When building _MCP servers_, you need a way to **test your functionality without connecting to a full application.** The _Python MCP SDK_ includes a built-in browser-based _inspector_ that lets you **debug and test your server in real-time.**

To run the _inspector_ you must run:

```bash title:command
$ mcp dev mcp_server.py
```

Or

```bash title:command
$ uv run mcp dev mcp_server.py
```

##### Testing Your Tools

Once the _inspector_ is successfully connected to the target _MCP_, navigate to the _Tools section_ and click _List Tools_ to see all available tools from your server. When you select a tool, **the right panel shows its details and input fields.**

![[mcp_inspector_interface.png]]


##### Development Workflow

The _MCP Inspector_ becomes an essential part of your development process. Instead of writing separate test scripts or connecting to full applications, you can:

- Quickly iterate on tool implementations
- Test edge cases and error conditions
- Verify tool interactions and state management
- Debug issues in real-time

**This immediate feedback loop makes** _MCP server_ **development much more efficient and helps catch issues early in the development process.**


### Client Side

The _client_ is what allows our application code to communicate with the _MCP server_ and access its functionality.

The _MCP client_ consists of two main components:
- **MCP Client** - A **custom class** we create to make using the session easier
- **Client Session** - The actual **connection to the server** (part of the _MCP Python SDK_)


![[mcp_client_architecture.png]]

> [!warning] Warning
> The _client session_ **requires careful resource management. We need to properly clean up connections when we're done.** That's why we wrap it in our own class that handles all the cleanup automatically.


#### Custom Class to Manage Sessions

```python title:mcp_client.py
from contextlib import AsyncExitStack
from typing import Any

from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client


class MCPClient:
	def __init__(
		self,
		command: str,
		args: list[str],
		env: dict | None = None,
	):
		self._command = command
		self._args = args
		self._env = env
		self._session: ClientSession | None = None
		self._exit_stack: AsyncExitStack = AsyncExitStack()

	async def connect(self):
		server_params = StdioServerParameters(
			command=self._command,
			args=self._args,
			env=self._env,
		)
		stdio_transport = await self._exit_stack.enter_async_context(
			stdio_client(server_params)
		)
		_stdio, _write = stdio_transport
		self._session = await self._exit_stack.enter_async_context(
			ClientSession(_stdio, _write)
		)
		await self._session.initialize()

	def session(self) -> ClientSession:
		if self._session is None:
			raise ConnectionError(
				"Client session not initialized or cache not populated. "
				"Call connect_to_server first."
			)
		return self._session

	###############################################################################
	################################ Set of Tools #################################
	###############################################################################

	async def cleanup(self):
		await self._exit_stack.aclose()
		self._session = None

	async def __aenter__(self):
		await self.connect()
		return self

	async def __aexit__(self, exc_type, exc_val, exc_tb):
		await self.cleanup()

```


#### Core Client Functions

##### List Tools Function

This function gets all available tools from the MCP server:

```python title:mcp_client.py
async def list_tools(self) -> list[types.Tool]:
    result = await self.session().list_tools()
    return result.tools

```

It's straightforward - we access our session (the connection to the server), call the built-in _list_tools()_ method, and return the tools from the result.


##### Call Tool Function

This function executes a specific tool on the server:

```python title:mcp_client.py
async def call_tool(
    self, tool_name: str, tool_input: dict
) -> types.CallToolResult | None:
    return await self.session().call_tool(tool_name, tool_input)

```

We pass the tool name and input parameters (provided by `Claude`) to the _server_ and return the result.


#### Testing the Client

To test the previous custom session class, we must add the following snippet at the bottom of our file:

```python title:mcp_client.py
from asyncio import run, set_event_loop_policy, WindowsProactorEventLoopPolicy
from contextlib import AsyncExitStack
from sys import platform

async def main():
	async with MCPClient(
		# If using Python without UV, update command to 'python' and remove "run" from args.
		command="uv",
		args=["run", "mcp_server.py"],
	) as _client:
		result = await _client.list_tools()
		print(result)


if __name__ == "__main__":
	if platform == "win32":
		set_event_loop_policy(WindowsProactorEventLoopPolicy())
	run(main())

```

And run it directly to verify everything works:

```bash title:bash
$ uv run mcp_client.py
```

This will connect to your _MCP server_ and print out the available tools. You should see output showing your tool definitions, including descriptions and input schemas.

---

## Claude Sessions
