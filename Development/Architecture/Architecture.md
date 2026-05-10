---
aliases:
  - Architecture
tags:
  - learning
  - dev/architecture
date: 2026-05-09
---
**Sources**: Claude

**Related:** [[Development]], [[Design Pattern]], [[DDD]], [[MVC]], [[Microservices]], [[Hexagonal]], [[Event-Driven]], [[CQRS]], [[Repository]], [[API Gateway]], [[ReactJS]], [[API]], [[API Rest]], [[GraphQL]], [[SOLID]], [[DRY]], [[KISS]]

---

> Building a structure:
> 
> - _Architecture_ is the overall plan: where the rooms go, the foundation, and the supporting structure.
> 
> - `Design patterns` are decisions about the details: what kind of hinges the doors use, how the plumbing is installed.

___

## Description

It is the fundamental structure of the entire system: **how components are organized**, **how they communicate**, and **what principles govern the entire project**.

- **Scope:** The entire system or major components of it
- **Purpose:** To define the system’s structure, scalability, maintainability, and limits
- **Examples:** `DDD`, `MVC`, `Microservices`, `Hexagonal` (Ports & Adapters), `Event-Driven`, `CQRS`

---

## Details

### Architecture vs `Design Pattern`

| **Dimension**        | **Architecture**                       | **Design Pattern**       |
| -------------------- | -------------------------------------- | ------------------------ |
| **Level**            | Macro (system)                         | Micro (code)             |
| **Impact of change** | Global, high impact                    | Local, low risk          |
| **Who decides**      | The architect / team                   | The developer            |
| **When applied**     | Before writing code                    | During implementation    |
| **Reversibility**    | Difficult to change                    | Relatively easy          |
| **Example**          | `Event-Driven` for the entire platform | `Observer` for UI events |

| What defines it?       | Who breaks it?                         | **Risks**                            |
| ---------------------- | -------------------------------------- | ------------------------------------ |
| **Architecture**       | Overall structure and dependency rules | Violating it = severe technical debt |
| **Per-module pattern** | Local solution to the module's problem | Changing it = limited refactoring    |


### In Practice

An _architecture_ **determines and dictates** which ``design patterns`` make sense to use:

- In ``hexagonal`` _architecture_: the `Repository` and _Adapter_ _patterns_ are natural choices
- In ``microservices``: _Saga_, _Circuit Breaker_, and `API Gateway` take center stage
- In ``MVC``: _Observer_ and _Strategy_ fit very well into the controller layer

A common mistake is to confuse the two, for example, calling what is actually just **a pattern applied to a module** “MVC architecture,” or thinking that implementing several design patterns is equivalent to having a defined architecture.

**In real-world projects, multiple** ``patterns`` **are always used simultaneously**. _Architecture_ does not dictate a single ``pattern``, but rather establishes the **rules of the game** within which each module solves its own problems.


### Project Type

The fundamental principles are universal, but **each domain has its own predominant architectures and patterns** because each faces different challenges.

#### Front-end

The main challenge is **managing the UI, state, and user events**:

| **Architecture** | **When to use it?**                           |     |
| ---------------- | --------------------------------------------- | --- |
| Component-Based  | Almost always (``ReactJS``, _Vue_, _Angular_) |     |
| Flux / MVI       | Apps with complex state                       |     |
| Micro-frontends  | Large teams, separate domains                 |     |


#### Back-end

The main issues are **business logic, scalability, and maintainability**:

| **Architecture**      | **When to use it?**                                        |
| --------------------- | ---------------------------------------------------------- |
| Layered (N-layers)    | Medium-sized projects, small teams                         |
| ``Hexagonal`` / Clean | Complex domain, high testability                           |
| ``Microservices``     | High scale, independent teams                              |
| ``Event-Driven``      | Asynchronous processes, decoupling                         |
| ``CQRS``              | Read and write operations with very different requirements |


#### API (``REST`` / ``GraphQL`` / _gRPC_)

The main issue is **contracts, versioning, and communication between systems**:

| **Architecture**      | **When to use it?**                     |
| --------------------- | --------------------------------------- |
| ``REST`` by resources | Public or general-purpose ``APIs``      |
| ``GraphQL``           | Clients with variable data needs        |
| gRPC                  | High-performance internal communication |
| ``API Gateway``       | Single entry point to ``microservices`` |

#### Database

The main issues are **consistency, performance, and data model**.

| **Architecture**         | **When to use it?**                    |
| ------------------------ | -------------------------------------- |
| Normalized relational    | Structured data, strong integrity      |
| Event Sourcing           | Comprehensive auditing, change history |
| ``CQRS`` at the DB level | Separate read and write models         |
| Sharding / Partitioning  | Massive horizontal scaling             |

#### What IS universal

Regardless of the type of project, these principles always apply:

```
SOLID                  → class and module design
DRY / KISS             → overall code quality
Separation of Concerns → each component with a clear responsibility
Dependency Inversion   → depend on abstractions, not implementations
```

```
What is the dominant problem in this project?

├─ Managing complex UI        → front-end architectures/patterns
├─ Scaling business logic    → back-end, DDD, hexagonal
├─ Exposing/consuming services   → API and communication patterns
└─ Handling large volumes    → DB and persistence patterns
```


> In _full-stack_ projects, all these elements coexist at the same time. Which is why the overall _architecture_ must clearly define the **boundaries between layers** so that each domain can implement its own solutions without interfering with the others.

---

## Claude Sessions
