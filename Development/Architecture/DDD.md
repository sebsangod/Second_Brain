---
aliases:
  - DDD
  - Domain-Driven Design
tags:
  - learning
  - dev/architecture
date: 2026-07-09
---
**Sources**: Eric Evans — _Domain-Driven Design: Tackling Complexity in the Heart of Software_ (2003), [Martin Fowler - DomainDrivenDesign](https://martinfowler.com/bliki/DomainDrivenDesign.html), Claude

**Related:** [[Architecture]], [[Hexagonal]], [[CQRS]], [[Repository]], [[Microservices]], [[Event-Driven]], [[SOLID]], [[Layer]], [[FastAPI]]

---

> _DDD_ is not a folder structure. `domain/`, `application/`, `repositories/` are just the *tactical* output. The real work happens **before writing any code**: talking to domain experts until the team and the business share the exact same words for the exact same concepts.

---

## Description

`Domain-Driven Design` (_DDD_) is a software design approach introduced by Eric Evans in 2003 to deal with **complex business domains** — systems where the hardest problems aren't technical (scaling, latency, infra) but come from the business logic itself: contradictory rules, the same word meaning different things to different teams, and behavior that keeps changing as the business evolves.

- **Scope:** How the business domain is modeled and how that model maps to code
- **Purpose:** Keep the software model aligned with how the business actually works, so complexity lives in one well-understood place instead of leaking into every layer
- **Not for:** Simple `CRUD` systems where the domain is trivial — _DDD_ adds ceremony that only pays off when the business logic is genuinely complex

---

## Key concepts

### Strategic design — the "why" and "where"

- **Ubiquitous Language**: a shared vocabulary between developers and domain experts, used literally in conversations, documentation, and code. If the business calls it a "Subscription", the code must never call it a "Contract" or an "Order".
- **Bounded Context**: an explicit boundary within which a model and its Ubiquitous Language are valid. The same word can mean different things in different contexts — that's a good sign, not a conflict to resolve.
- **Subdomains**: `Core` (what makes the business competitive — invest here), `Supporting` (necessary but not differentiating — build it simple), `Generic` (solved problems — buy or reuse, e.g. auth, billing).
- **Context Mapping**: how bounded contexts relate to each other (`Shared Kernel`, `Customer-Supplier`, `Conformist`, `Anti-Corruption Layer`, `Open Host Service`).

### Tactical design — the "how", inside one bounded context

- **Entity**: has identity that persists over time even as its attributes change (a `User`, an `Order`).
- **Value Object**: has no identity, defined only by its attributes, immutable (`Money`, `Address`, `DateRange`).
- **Aggregate**: a cluster of entities/value objects treated as one consistency unit, with a single **Aggregate Root** as its only entry point (an `Order` aggregate may contain `OrderLines`, but nothing outside touches an `OrderLine` directly).
- **Domain Service**: business logic that doesn't naturally belong to a single entity (e.g. "transfer funds between two accounts").
- **`Repository`**: an abstraction for retrieving/persisting aggregates, so the domain never talks to a database directly.
- **Domain Event**: something meaningful that happened in the domain (`OrderPlaced`, `SubscriptionExpired`), used to decouple side effects.
- **Factory**: encapsulates complex creation logic for entities/aggregates when a plain constructor isn't enough.

---

## Details

### The problem _DDD_ actually solves

Before _DDD_, most complex backends drift toward one of two failure modes:

1. **Big ball of mud**: business rules scattered across controllers, scripts, stored procedures, and API handlers — wherever it was fastest to add the next feature.
2. **Anemic domain model**: data classes with getters/setters and zero behavior, while all the actual logic lives in "manager" or "service" classes that operate on that data from the outside — technically object-oriented, but no real domain modeling.

Both patterns work fine for small systems. They collapse under **domain complexity**: once the business has many interacting rules (discounts depending on tier, region, and campaign; `KYC` rules that vary by country; invoicing rules that differ per tax regime...), nobody can hold the whole rule set in their head, and the same rule gets reimplemented — inconsistently — in three different files.

_DDD_'s answer: put the business rules in one place (the `domain` layer), express them in the same words the business uses (`Ubiquitous Language`), and draw explicit boundaries (`Bounded Contexts`) so no single model has to represent the *entire* business at once.

### Why "domain-driven" and not "database-driven" or "API-driven"

Most teams design software from the outside in: start from the database schema, or from the API contract, and let the domain model fall out of whatever those force it to look like. _DDD_ inverts that — **model the domain first**, in code that has zero knowledge of `HTTP`, `SQL`, or any framework — then adapt persistence and delivery mechanisms to fit that model, never the other way around. This is why a proper _DDD_ implementation keeps a framework-agnostic `domain/` folder, and why a persistence model (a `Beanie Document`, a `SQLAlchemy` model) is always a *translation* of the domain entity, never the entity itself.

### Strategic design in practice: turning a problem into Bounded Contexts

This is the actual translation step from "a problem/project" into _DDD_ — it happens **before** any code:

1. **Talk to domain experts, not just stakeholders.** Whoever actually does the work (support agents, accountants, warehouse staff) knows the real rules and exceptions better than any spec.
2. **Run an Event Storming session** (even an informal one): write every significant business event on a sticky note (`OrderPlaced`, `PaymentFailed`, `InvoiceIssued`), in chronological order, no diagrams yet. Patterns and natural groupings emerge from the event sequence.
3. **Cluster events into candidate Bounded Contexts.** Groups of events that share vocabulary and are owned by the same part of the business become one context (e.g. all invoicing/tax events → a `Billing` context).
4. **Classify each context as Core / Supporting / Generic.** This tells you where to invest engineering effort and where to buy/reuse instead of building.
5. **Draw the Context Map.** For each pair of contexts that need to talk to each other, decide the relationship explicitly: does `Billing` dictate the model to `Orders` (`Customer-Supplier`), or does `Orders` protect itself from a legacy system's model with an `Anti-Corruption Layer`?
6. **Only then, inside each context, do tactical modeling** — entities, value objects, aggregates — using the vocabulary agreed on in step 1.

A common mistake is skipping straight to step 6 — creating folders named `domain/`, `application/`, `infrastructure/` — without ever doing steps 1–5. That produces _DDD_-flavored *file structure* with none of the actual benefit, because the bounded contexts were never truly identified.

### When _DDD_ is worth it (and when it isn't)

| Signal | Verdict |
|---|---|
| The business has complex, changing rules that differ by segment/region/tier | Use _DDD_ — this is exactly its target problem |
| Multiple teams need to work on different parts of the business without stepping on each other | Use _DDD_ — `Bounded Contexts` give each team an isolated model |
| The system is mostly `CRUD` with thin validation | Skip _DDD_ — a simple layered/`MVC` structure is cheaper and just as correct |
| Domain experts can't explain the rules consistently themselves | Do the `Ubiquitous Language` work *first* — that's the actual value, before any code |
| It's a small internal tool or a short-lived MVP | Skip _DDD_ — the strategic-design investment won't pay back in time |

### Relationship to other architectures

_DDD_ is a modeling discipline, not a layering scheme by itself — it pairs naturally with:

- **[[Hexagonal]]** (Ports & Adapters): the domain sits at the center, dictating interfaces (`ports`) that infrastructure (`adapters`) must implement. This is where the `Repository` interface pattern comes from.
- **[[CQRS]]**: separates the read model from the write model — useful when a Bounded Context's queries and commands have very different shapes or performance needs.
- **[[Event-Driven]]** architecture: `Domain Events` are a natural fit for decoupling side effects across (or within) Bounded Contexts.

_DDD_ tells you *what* the domain model should look like and *where its boundaries are*; `Hexagonal`/`CQRS`/`Event-Driven` tell you *how to wire it* to the outside world.

---

## Examples

### E-commerce

- `Catalog` context: a `Product` is mostly descriptive data (name, images, price) — low complexity.
- `Inventory` context: the *same* product is a `StockItem` with reservation rules, warehouse location, and concurrency concerns — a completely different model of the same physical thing.
- `Ordering` context: an `Order` aggregate enforces invariants like "cannot ship if any line is out of stock" or "total must match line items".
- These three contexts never share a single `Product` class — each translates the concept into what it actually needs.

### Fintech / banking

- `Accounts` context: balances, ownership, account types.
- `Ledger` context: an append-only, immutable record of transactions — a fundamentally different consistency model than `Accounts` (often where `Event Sourcing` shows up naturally).
- `Fraud Detection` context: doesn't care about balances at all, only transaction *patterns* — a completely different Ubiquitous Language from the other two.

### Healthcare

- `Clinical` context: a `Patient` is a medical record — diagnoses, allergies, prescriptions.
- `Billing` context: the same person is mostly an `Insured Party` — coverage, co-pays, claims.
- `Scheduling` context: the same person is an appointment-slot holder — almost no clinical data needed at all.
- Forcing one universal `Patient` model across all three contexts is exactly the mistake the `Bounded Context` concept exists to prevent.

### A recognizable real-world case

A backend that handles user accounts, company records, invoicing, and role-based access is, in _DDD_ terms, likely 3–4 separate Bounded Contexts (`Users`, `Companies`, `Invoicing`, `Security`) — each with a different owner, a different rate of change, and its own vocabulary. It's common for these to end up implemented as one flat context instead, with every model dumped together regardless of which business concern it belongs to. Redrawing that boundary — even before touching any folder structure — is the actual strategic-design step most "_DDD_" backends skip.

---

## Utils

### Decision checklist: translating a problem into _DDD_

Before writing a single `domain/` folder, answer these in order:

1. **What are the significant events in this business process?** (List them in plain business language, not technical terms.)
2. **Which events naturally cluster together** because they're owned by the same team or share the same vocabulary?
3. **For each cluster, is it Core (competitive advantage), Supporting (necessary), or Generic (solved elsewhere)?**
4. **Where do two clusters need to exchange data?** For each connection, name the relationship (`Shared Kernel`, `Customer-Supplier`, `Anti-Corruption Layer`, ...) — don't leave it implicit.
5. **Only now: inside each cluster, what are the Entities, Value Objects, and the one Aggregate Root per consistency boundary?**

If you can't answer #1 without reading the codebase first, the domain experts haven't been consulted enough yet — go back and talk to them before modeling anything.

---

## Claude Sessions
