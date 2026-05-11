---
aliases:
  - Agno
tags:
  - learning
  - dev/ai
date: 2026-05-08
---
**Sources**: [Agno](https://www.agno.com/agent-framework), [docs](https://docs.agno.com/), [sdk](https://docs.agno.com/sdk/introduction)

**Related:** [[Artificial Intelligence]], [[Python]], [[FastAPI]], [[AI Agent]], [[RBAC]], [[WebSocket]], [[Database]], [[PostgreSQL]], [[Clickhouse]], [[OLAP]], [[Development/AI/Applications/MCP/MCP|MCP]], [[Multi-tenant]], [[Image]], [[Claude Code]], [[LangGraph]]

---

## Description

_Agno_ provides **software for building**, **running**, and **managing agent platforms**.

- Build `agents` **using any agent framework**.
- Run them as production services with session management, tracing, scheduling, and `RBAC`.
- Manage your platform using a single control plane.

_Agno_ has a 3-layer architecture. Everything except the control plane is free and open-source.

| Layer             | Use it to | What it does                                                       |
| ----------------- | --------- | ------------------------------------------------------------------ |
| **SDK**           | Build     | Build `agents`, multi-`agent` teams, and agentic workflows.        |
| **Runtime**       | Run       | Run your `agents`, teams, and workflows as a service.              |
| **Control Plane** | Manage    | Manage your platform using the [AgentOS UI](https://os.agno.com/). |

---

## Key concepts

- [**Production API.**](https://docs.agno.com/runtime/serve-as-api) 50+ endpoints with SSE and `websockets` make it easy to build a product on top of your agent platform.
- [**Storage.**](https://docs.agno.com/runtime/storage) Store sessions, memory, knowledge, and traces in your own `database`. Use `postgres` for quick read/write data like sessions and memory. Use `clickhouse` for `OLAP` data like traces.
- [**100+ integrations.**](https://docs.agno.com/tools/toolkits/overview) Integrate with 100+ tools using pre-built toolkits.
- [**Context Providers.**](https://docs.agno.com/runtime/context) Use strategies like context providers to access live data stored in Slack, Drive, wikis, `MCP`, and custom sources.
- [**Human approval.**](https://docs.agno.com/runtime/human-approval) Built in mechanisms for pausing runs for user confirmation up to blocking tools that require admin approval.
- [**Observability.**](https://docs.agno.com/runtime/observability) Get monitoring via OpenTelemetry tracing, run history, and audit logs out of the box.
- [**Security.**](https://docs.agno.com/runtime/security-and-auth) Get `JWT`-based `RBAC` and multi-user, `multi-tenant` isolation out of the box.
- [**Interfaces.**](https://docs.agno.com/runtime/interfaces) Expose your agents via Slack, Telegram, WhatsApp, Discord, AG-UI, A2A.
- [**Scheduling.**](https://docs.agno.com/runtime/scheduling) Cron-based scheduling and background jobs with no external infrastructure.
- [**Deploy anywhere.**](https://docs.agno.com/runtime/deploy) Run on any cloud platform that can run a containerized `image.

---

## Example

```python title:main.py
from agno.os import AgentOS
from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.tools.workspace import Workspace

agent = Agent(
    name="Agno Agent",
    model="openai:gpt-5.4",
    tools=[Workspace(".",
        allowed=["read", "list", "search"],
        confirm=["write", "edit", "delete", "shell"],
    )],
)

agent_os = AgentOS(
    agents=[agent],
    tracing=True,
    db=SqliteDb(db_file="tmp/agentos.db"),
)

app = agent_os.get_app()

```

20 lines of code and you have a `FastAPI` backend with 50+ endpoints, persisted sessions, tracing, scheduling and `RBAC`. Interact using the AgentOS UI, Slack, Discord or build a product on top. Run native Agno agents next to `Claude Code`, `LangGraph` and `DSPy` agents in the same AgentOS.

---

## Details

### Pick a Template

Start with the closest template to what you’re building.

_Agno_ gives you four templates: one foundational, three production.

| If you’re…                                                       | Start with                                                                |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Building your own `agent` platform from a blank canvas           | [Agent Platform](https://docs.agno.com/tutorials/agent-platform/overview) |
| Building an `agent` that can query Slack, GDrive, internal wikis | [Scout](https://docs.agno.com/tutorials/scout/overview)                   |
| Building an `agent` for data analytics, BI, metrics              | [Dash](https://docs.agno.com/tutorials/dash/overview)                     |
| Building a code companion to help maintain your codebase         | [Coda](https://docs.agno.com/tutorials/coda/overview)                     |
| Looking for a kitchen-sink with every _Agno_ feature             | [Demo OS](https://docs.agno.com/demo-os/overview)                         |

---

## Utils

### Use case

Write here...

```python title:main.py
print("Hello world!")

```

---

## Claude Sessions
