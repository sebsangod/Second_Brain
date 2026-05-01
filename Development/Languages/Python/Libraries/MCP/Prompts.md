---
aliases:
  - MCP
tags:
  - learning
  - dev/backend
date: 2026-05-01
---
**Sources**: [Introduction to Model Context Protocol](https://anthropic.skilljar.com/introduction-to-model-context-protocol/)

**Related:** [[Development/Languages/Python/Libraries/MCP/MCP|MCP]], [[Development/AI/Applications/MCP/MCP|MCP]], [[Claude|Claude]], [[Prompt Engineering]], [[Commands]], [[Decorators]], [[Pydantic|Pydantic]]

---

## Description

_Prompts_ in _MCP servers_ **let you define pre-built, high-quality instructions that clients can use instead of writing their own prompts from scratch.**

Think of them as **carefully crafted templates that give better results** than what users might come up with on their own.

The goal is to provide _prompts_ that are so well-crafted and tested that **users prefer them over writing their own instructions from scratch**.

---

## Key Benefits

- **Consistency:** Users get reliable results every time
- **Expertise:** You can encode domain knowledge into _prompts_
- **Reusability:** Multiple client applications can use the same _prompts_
- **Maintenance:** Update _prompts_ in one place to improve all clients

_Prompts_ work best when they're specialized for your _MCP server's_ domain. A document management server might have _prompts_ for formatting, summarizing, or analyzing documents. A data analysis server might have prompts for generating reports or visualizations.

---

## Details

### Why Use Prompts?

Here's the key insight: users can already ask ``Claude`` to do most tasks directly. For example, a user could type "reformat the report.pdf in markdown" and get decent results. But they'll **get much better results if you provide a thoroughly tested, specialized prompt that handles edge cases and follows best practices.**

As the _MCP server_ author, you can spend time crafting, testing, and evaluating _prompts_ that work consistently across different scenarios. **Users benefit from this expertise without having to become** ``prompt engineering`` **experts themselves**.

![[mpc_prompt_example.png]]


### How Prompts Work

![[mcp_how_prompts_work.png]]

_Prompts_ **define a set of user and assistant messages that clients can use.** They should be high-quality, well-tested, and relevant to your _MCP server's_ purpose. The workflow is:

- Write and evaluate a _prompt_ relevant to your server's functionality
- Define the _prompt_ in your _MCP server_ using the _@mcp.prompt_ decorator
- Clients can request the _prompt_ at any time
- Arguments provided by the client become keyword arguments in your _prompt_ function
- The function returns formatted messages ready for the AI model

This system creates reusable, parameterized _prompts_ that maintain consistency while allowing customization through variables. It's particularly useful for complex workflows where you want to ensure the AI receives properly structured instructions every time.


### Building a Format ``Command``

Let's implement a practical example: a format ``command`` that converts documents to markdown. Users will type _/format doc_id_ and get back a professionally formatted markdown version of their document.

The workflow looks like this:

- User types _/_ to see available commands
- They select _format_ and specify a document ID
- ``Claude`` uses your pre-built prompt to read and reformat the document
- The result is clean markdown with proper headers, lists, and formatting


#### Defining _Prompts_

_Prompts_ use a similar ``decorator pattern`` to tools and resources:

```python title:mcp_server.py
from mcp.server.fastmcp.prompts import base


@mcp.prompt(
	name="format",
	description="Rewrites the contents of the document in Markdown format."
)
def format_document(
	doc_id: str = Field(description="Id of the document to format")
) -> list[base.Message]:
    prompt = f"""
Your goal is to reformat a document to be written with markdown syntax.

The id of the document you need to reformat is:
<document_id>
{doc_id}
</document_id>

Add in headers, bullet points, tables, etc as necessary. Feel free to add in structure.
Use the 'edit_document' tool to edit the document. After the document has been reformatted...
"""

	return [
		base.UserMessage(prompt)
	]

```

The function returns a list of messages that get sent directly to ``Claude``. You can include multiple user and assistant messages to create more complex conversation flows.


### Accessing _Resources_ from Client Side

#### Implementing List Prompts

The _list_prompts_ method is straightforward. It calls the session's list prompts function and returns the prompts:

```python title:mcp_client.py
async def list_prompts(self) -> list[types.Prompt]:
    result = await self.session().list_prompts()
    return result.prompts
```


### Getting Individual Prompts

The _get_prompt_ method is more interesting because **it handles variable interpolation.** When you request a _prompt_, you provide arguments that get passed to the _prompt_ function as **keyword arguments**:

```python title:mcp_client.py
async def get_prompt(self, prompt_name, args: dict[str, str]):
    result = await self.session().get_prompt(prompt_name, args)
    return result.messages
```

For example, if your server has a _format_document_ _prompt_ that expects a _doc_id_ parameter, the arguments dictionary would contain _{"doc_id": "plan.md"}_. **This value gets interpolated into the prompt template.**

---

## Claude Sessions
