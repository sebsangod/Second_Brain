---
aliases:
  - CLAUDE.md
tags:
  - dev/ai/llm
  - dev/backend
date: 2026-05-02
---
**Sources**: [Anthropic SDK](https://docs.anthropic.com/en/api/), [OpenAI SDK](https://platform.openai.com/docs/api-reference), [LangChain](https://python.langchain.com/docs/), [MCP Spec](https://modelcontextprotocol.io/)

**Related:** [[CLAUDE File]], [[Large Language Models]], [[AI Agent]], [[MCP]], [[FastAPI]], [[Pydantic]], [[Beanie]]

---

# AI / LLM Application Development

## Project
Python 3.12+ AI-powered application.
Stack: LLM provider SDKs + Pydantic v2 + async Python.
Common integrations: FastAPI (serving), MongoDB/Beanie (storage), vector DBs (retrieval).

## Commands
- Dev server: `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`
- Tests: `python -m pytest tests/ -v --tb=short`
- Tests (single): `python -m pytest tests/test_module.py::TestClass::test_method -v`
- Lint: `ruff check . --fix && ruff format .`
- Deps: `uv add <package>` / `uv remove <package>`

## Project Structure
```
project/
├── app/
│   ├── __init__.py
│   ├── main.py                  # Entrypoint (FastAPI or CLI)
│   ├── config.py                # Settings, API keys, model configs
│   │
│   ├── llm/                     # LLM interaction layer
│   │   ├── __init__.py
│   │   ├── client.py            # Provider client factory (Anthropic, OpenAI)
│   │   ├── prompts.py           # Prompt templates and builders
│   │   ├── schemas.py           # Request/response models for LLM calls
│   │   └── parsers.py           # Output parsing (structured, JSON, etc.)
│   │
│   ├── agents/                  # Agentic patterns
│   │   ├── __init__.py
│   │   ├── base.py              # Base agent loop
│   │   ├── tools.py             # Tool definitions and registry
│   │   └── orchestrator.py      # Multi-agent coordination
│   │
│   ├── retrieval/               # RAG / context retrieval
│   │   ├── __init__.py
│   │   ├── embeddings.py        # Embedding generation
│   │   ├── vector_store.py      # Vector DB interface (ABC)
│   │   ├── chunking.py          # Document chunking strategies
│   │   └── retriever.py         # Search + reranking logic
│   │
│   ├── domain/                  # Business logic (LLM-agnostic)
│   │   ├── __init__.py
│   │   └── models.py
│   │
│   ├── infrastructure/          # External services, DB, storage
│   │   ├── __init__.py
│   │   ├── database.py
│   │   └── storage.py
│   │
│   └── shared/
│       ├── __init__.py
│       ├── tokens.py            # Token counting utilities
│       └── utils.py
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_prompts.py
│   ├── test_parsers.py
│   └── test_agents.py
│
├── prompts/                     # Prompt files (version-controlled)
│   ├── system/
│   │   └── assistant.md
│   └── templates/
│       └── summarize.md
│
├── .env
├── .env.example
├── pyproject.toml
└── uv.lock
```

## LLM Client Patterns

### Provider abstraction
```python
from abc import ABC, abstractmethod
from pydantic import BaseModel

class LLMResponse(BaseModel):
    content: str
    model: str
    input_tokens: int
    output_tokens: int
    stop_reason: str

class LLMClient(ABC):
    @abstractmethod
    async def complete(
        self,
        messages: list[dict[str, str]],
        model: str,
        max_tokens: int = 1024,
        temperature: float = 0.0,
        **kwargs,
    ) -> LLMResponse: ...

class AnthropicClient(LLMClient):
    async def complete(self, ...) -> LLMResponse:
        response = await self.client.messages.create(...)
        return LLMResponse(...)

class OpenAIClient(LLMClient):
    async def complete(self, ...) -> LLMResponse:
        response = await self.client.chat.completions.create(...)
        return LLMResponse(...)
```
- Always abstract behind an interface — never scatter `anthropic.messages.create()` across the codebase
- Return a normalized response model — not raw provider objects
- Inject via dependency injection — swap providers without code changes

### API key management
```python
from pydantic_settings import BaseSettings

class AISettings(BaseSettings):
    anthropic_api_key: str = ""
    openai_api_key: str = ""
    default_model: str = "claude-sonnet-4-20250514"
    max_retries: int = 3
    request_timeout: int = 30
```
- All keys from `.env` — never hardcoded, never committed
- One settings class per concern — separate AI settings from DB settings

## Prompt Engineering

### Prompt management rules
- Store prompts as files in `prompts/` directory — not inline strings
- Version control prompts — they are code, treat them as such
- Use clear variable markers: `{variable_name}` — substitute with `.format()` or template engine
- System prompt: separate file, loaded once at startup
- Never concatenate user input directly into prompts — always parameterize

### Prompt structure (best practices)
```markdown
# System Prompt
Role definition → Context → Task → Constraints → Output format

# User Prompt
Context (if dynamic) → Specific instruction → Input data
```

### Structured output
```python
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    sentiment: Literal["positive", "negative", "neutral"]
    confidence: float
    summary: str
    key_topics: list[str]

# Force structured output via provider SDK or parsing
response = await client.complete(
    messages=[...],
    response_format=AnalysisResult,  # if supported
)
# Or parse manually
result = AnalysisResult.model_validate_json(response.content)
```
- Always use Pydantic models for LLM outputs — never parse raw strings manually
- Validate with `model_validate_json()` — handle `ValidationError` gracefully
- Include retry logic for malformed outputs — LLMs are non-deterministic

## Agentic Patterns

### Tool definition
```python
from pydantic import BaseModel, Field

class ToolParameter(BaseModel):
    name: str
    description: str
    type: str
    required: bool = True

class Tool(BaseModel):
    name: str
    description: str
    parameters: list[ToolParameter]
    handler: Callable  # async function that executes the tool
```
- Tools are Pydantic models — schema is auto-serializable to provider format
- Tool handlers are async functions — never blocking I/O in the agent loop
- Register tools in a registry — never hardcode in the agent loop

### Agent loop pattern
```python
async def agent_loop(
    client: LLMClient,
    tools: list[Tool],
    messages: list[dict],
    max_iterations: int = 10,
) -> str:
    for _ in range(max_iterations):
        response = await client.complete(messages=messages, tools=tools)

        if response.stop_reason == "end_turn":
            return response.content

        if response.stop_reason == "tool_use":
            tool_result = await execute_tool(response.tool_call, tools)
            messages.append({"role": "assistant", "content": response.raw})
            messages.append({"role": "user", "content": tool_result})

    raise MaxIterationsError(f"Agent did not converge in {max_iterations} steps")
```
- Always cap iterations — agents can loop forever
- Log every iteration: input tokens, output tokens, tool calls, elapsed time
- Append full conversation history — the LLM needs context from previous turns

### Multi-agent design
- One agent = one responsibility (researcher, planner, executor)
- Agents communicate via structured messages (Pydantic models), not raw strings
- Orchestrator decides routing — individual agents never call each other directly
- Each agent has its own system prompt, tools, and model config

## RAG (Retrieval-Augmented Generation)

### Pipeline
```
Document → Chunk → Embed → Store → Query → Retrieve → Rerank → Augment → Generate
```

### Chunking rules
- Chunk size: 500-1000 tokens (tune per use case)
- Overlap: 10-20% between chunks — prevents cutting context at boundaries
- Respect document structure: prefer splitting at paragraphs/sections, not mid-sentence
- Store metadata with chunks: `source`, `page`, `section`, `timestamp`

### Embedding conventions
- Use the same model for indexing and querying — mismatched models = poor retrieval
- Normalize embeddings if using cosine similarity
- Cache embeddings — never re-embed unchanged documents
- Log embedding dimensions and model version in metadata

### Vector store abstraction
```python
class VectorStoreInterface(ABC):
    @abstractmethod
    async def upsert(self, documents: list[Document]) -> None: ...

    @abstractmethod
    async def search(
        self, query: str, top_k: int = 5, filters: dict | None = None
    ) -> list[SearchResult]: ...
```
- Abstract behind interface — swap vector DBs (Chroma, Pinecone, Qdrant, pgvector) freely
- Always include `top_k` and metadata filters in search interface
- Return scored results: `SearchResult(content, score, metadata)`

### Context window management
- Count tokens before sending to LLM — never exceed model limits
- Reserve tokens: system prompt + response budget first, then fill with retrieved context
- Use `tiktoken` (OpenAI) or provider-specific tokenizer for accurate counts
- Truncate retrieved chunks by relevance score, not arbitrarily

## MCP (Model Context Protocol)

### When to use MCP
- Integrating external tools/data sources into agentic workflows
- Building reusable tool servers that multiple clients can consume
- Bridging LLM applications with databases, APIs, or filesystems

### MCP server structure
```python
# Expose tools via MCP protocol
# Tools: ListToolsRequest → ListToolsResult
# Execution: CallToolRequest → CallToolResult
```
- One MCP server per domain/service — not one monolithic server
- Transport: stdio for local, HTTP/WebSocket for remote
- Permission model: explicitly declare which tools a server exposes

## Testing AI Applications

### What is testable (and must be tested)
1. **Prompt construction**: given inputs, verify the assembled prompt is correct
2. **Output parsing**: given raw LLM text, verify Pydantic model parses correctly
3. **Tool execution**: given tool call params, verify the handler returns expected results
4. **Chunking logic**: given a document, verify chunks meet size/overlap constraints
5. **Embedding pipeline**: verify documents produce vectors of correct dimensionality
6. **Agent routing**: given a message, verify the correct tool/agent is selected
7. **Error handling**: malformed LLM output, API timeouts, rate limits

### What to mock
- **Always mock**: LLM API calls — they are non-deterministic, slow, and cost money
- **Always mock**: embedding API calls — same reasons
- **Never mock**: prompt assembly, output parsing, chunking, business logic

### Testing pattern
```python
@pytest.fixture
def mock_llm_client():
    client = AsyncMock(spec=LLMClient)
    client.complete.return_value = LLMResponse(
        content='{"sentiment": "positive", "confidence": 0.95}',
        model="test-model",
        input_tokens=100,
        output_tokens=50,
        stop_reason="end_turn",
    )
    return client

async def test_analysis_parses_positive_sentiment(mock_llm_client):
    result = await analyze(client=mock_llm_client, text="Great product!")
    assert result.sentiment == "positive"
    assert result.confidence > 0.9
```

### Evaluation (beyond unit tests)
- Build eval datasets: input + expected output pairs
- Track metrics: accuracy, latency, token cost per request
- Version prompts alongside eval results — know which prompt produced which score
- Run evals in CI on prompt changes — regressions are silent without measurement

## Cost and Performance

### Token optimization
- Cache repeated requests (same input → same output when temperature=0)
- Use cheaper models for simple tasks — reserve powerful models for complex reasoning
- Batch requests when possible — fewer API calls, lower overhead
- Stream responses for user-facing endpoints — perceived latency matters

### Observability
- Log every LLM call: model, input_tokens, output_tokens, latency_ms, cost
- Track cumulative cost per session/user/feature
- Alert on token budget thresholds — prevent runaway costs
- Use structured logging — JSON format, parseable by monitoring tools

## Anti-patterns (never do these)
1. Hardcoded API keys or model names in source code
2. Raw string manipulation for prompt building — use templates
3. Trusting LLM output without validation — always parse with Pydantic
4. No iteration cap on agent loops — infinite loops burn money
5. Mixing provider-specific code with business logic
6. Embedding with one model, querying with another
7. Ignoring token counts — context window overflow causes silent truncation
8. Synchronous LLM calls in async handlers — always `await`
9. No retry/backoff on API calls — rate limits and transient errors are normal
10. Testing against live LLM APIs in CI — mock at the client boundary

## Verification (AI-specific)
After every change:
1. `ruff check . --fix && ruff format .`
2. `python -m pytest tests/ -v --tb=short` — all tests pass (with mocked LLM)
3. Verify prompt files: no broken template variables, valid markdown
4. If prompt changed: run eval suite and compare metrics to baseline
5. Check token budget: verify no request exceeds model context window

---

## Claude Sessions
