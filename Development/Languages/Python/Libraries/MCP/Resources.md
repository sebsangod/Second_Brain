---
aliases:
  - MCP
tags:
  - learning
  - dev/backend
date: 2026-04-29
---
**Sources**: [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol/)

**Related:** [[Development/Languages/Python/Libraries/MCP/MCP|MCP]], [[Development/AI/Applications/MCP/MCP|MCP]]

---

## Description

_Resources_ in _MCP servers_ **allow you to expose data to clients, similar to GET request handlers in a typical HTTP server.** They're perfect for scenarios where you need to fetch information rather than perform actions.

_Resources_ provide a clean way to expose read-only data from your _MCP server_, making it easy for clients to fetch information without the complexity of tool calls.

___

## Details

### How _Resources_ Work

_Resources_ **follow a request-response pattern**. When your client needs data, it sends a _ReadResourceRequest_ with a _URI_ to identify which resource it wants. The _MCP server_ processes this request and returns the data in a _ReadResourceResult_.

![[mcp_how_resources_work.png]]

The flow looks like this: your code requests a resource from the _MCP client_, which forwards the request to the _MCP server_. The server processes the _URI_, runs the appropriate function, and returns the result.


### Types of _Resources_

There are two types of _resources_:

#### Direct _Resources_

_Direct resources_ have static _URIs_ that never change. They're perfect for operations that don't need parameters.

```python title:mcp_server.py
@mcp.resource("docs://documents", mime_type="application/json")
def list_docs() -> list[str]:
    return list(docs.keys())

```


#### Templated _Resources_

_Templated resources_ include parameters in their _URIs_. The _Python SDK_ automatically parses these parameters and passes them as keyword arguments to your function.

```python title:mcp_server.py
@mcp.resource("docs://documents/{doc_id}", mime_type="text/plain")
def fetch_doc(doc_id: str) -> str:
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")

    return docs[doc_id]

```


### Implementation Details

_Resources_ **can return any type of data**: strings, JSON, binary data, etc. Use the _mime_type_ parameter to give clients a hint about what kind of data you're returning:

- _application/json_ for structured data
- _text/plain_ for plain text
- _application/pdf_ for binary files

The _MCP Python SDK_ **automatically serializes your return values.** You don't need to manually convert objects to JSON strings - just return the data structure and let the _SDK_ handle serialization.


### Accessing _Resources_ from Client Side

#### Implementing _Resource_ Reading

To enable _resource_ access in your _MCP client_, you need to implement a _read_resource_ function.

The core function makes a request to the _MCP server_ and processes the response based on its _MIME type_:

```python title:mcp_client.py
from json import loads
from pydantic import AnyUrl

...

###############################################################################
################################ Set of Tools #################################
###############################################################################
async def read_resource(self, uri: str) -> Any:
    result = await self.session().read_resource(AnyUrl(uri))
    resource = result.contents[0]

    if isinstance(resource, types.TextResourceContents):
        if resource.mimeType == "application/json":
            return loads(resource.text)

    return resource.text

```


#### Understanding the Response Structure

When you request a resource, the server returns a result with a _contents_ list. We access the first element since we typically only need one resource at a time. The response includes:

- The actual content (text or data)
- A MIME type that tells us how to parse the content
- Other metadata about the resource


#### Content Type Handling

The function checks the _MIME type_ to determine how to process the content:

- If it's _application/json_, parse the text as JSON and return the parsed object
- Otherwise, return the raw text content

This approach handles both structured data (like JSON) and plain text documents seamlessly.

> [!tip] Testing Resource Access
> Once implemented, you can test the _resource_ functionality through your CLI application. When you type "@" followed by a resource name, the system will:
> 
>1. Show available _resources_ in an autocomplete list
>2. Let you select a _resource_ using arrow keys and space
>3. Include the _resource_ content directly in your prompt
>4. **Send everything to the AI model without requiring additional tool calls**
>
>This creates a much smoother user experience compared to having the AI model make separate tool calls to access document contents. **The** _resource_ **content becomes part of the initial context, allowing for immediate responses about the data.**

---

## Claude Sessions
