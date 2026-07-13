# Event Sourcing vs CDC

Full CQRS + Event Sourcing coverage is Day 5's job. This page exists purely to draw the boundary between the two clearly, since "aren't these basically the same thing?" is a genuine, common point of confusion worth resolving directly.

## The one-line hook

> **In Event Sourcing, the events ARE the database — they're the primary source of truth. In CDC, a conventional database is still the source of truth, and the event stream is just a derived, secondary side effect of it.**

## The core distinction

| | Event Sourcing | CDC |
|---|---|---|
| **Source of truth** | The sequence of events itself | The conventional (relational) database |
| **How current state is obtained** | Replaying/folding events from the beginning, or from a snapshot forward | Simply querying the database directly, as normal |
| **What the event stream represents** | The authoritative history — nothing else | A derived, secondary artifact — a byproduct of ordinary CRUD writes |
| **Application design impact** | Requires genuinely event-first modeling from the start | Application code stays conventional CRUD — CDC bolts on externally |

**Memorable hook:** *"Ask yourself: if you deleted the event stream, would you have lost your data, or just lost a convenient copy of it? In Event Sourcing, you'd have lost everything. In CDC, you'd still have the actual database sitting right there."*

## Why CDC + Outbox is usually the pragmatic default, not full Event Sourcing

Full Event Sourcing is a genuinely significant architectural commitment: it requires event-first domain modeling from day one, infrastructure for replaying and projecting events into queryable read models, and a snapshotting strategy so replaying isn't prohibitively slow as history grows. **CDC combined with the Outbox pattern, using Debezium, is usually the more pragmatic choice** for teams that already have conventional, working CRUD-based services — it adds reliable event propagation on top of an existing data model, without requiring a from-scratch rewrite of how the application thinks about its own data.

**Memorable hook:** *"Event Sourcing asks you to redesign how your application thinks about data, from the ground up. CDC plus Outbox asks you to add one table and point a tool at it — a much smaller architectural bet for most teams."*

## When full Event Sourcing genuinely earns its cost

- When the **event history itself is the actual business value** — audit-heavy domains where "reconstruct exactly how we arrived at this state" is a first-class requirement, not an afterthought.
- When the domain's logic genuinely benefits from **temporal, event-based modeling** rather than current-state modeling — complex state machines where "what happened, in what order" carries real business meaning.
- When a system is built heavily around **CQRS**, wanting to derive and rebuild multiple independent read models from one authoritative raw event history (full treatment on Day 5).

## They're not mutually exclusive

Worth stating if the conversation goes deeper: CDC can also be used to propagate events **out of** an already event-sourced system's own event store to other consumers — the two patterns aren't strictly competing, they're just optimized for different starting points (conventional CRUD vs. event-first design).

## Real-world examples

1. **The TnD Microservices platform's likely-conventional, CRUD-based service data models** are a strong argument for CDC + Outbox as the pragmatic default — no evidence in your project history suggests those services were built event-first, so bolting on reliable event propagation via CDC is the realistic path, not a full Event Sourcing rewrite.
2. **A highly-audited financial ledger system genuinely needing complete historical replay capability** — a plausible scenario given your banking-adjacent account context — is a case where full Event Sourcing's upfront cost might actually be justified, since the event history itself carries direct business and regulatory value.
3. **Correctly answering "aren't Event Sourcing and CDC basically the same thing?"** — a real conceptual trap question, and this page's central distinction (source of truth vs. derived artifact) is the precise, correct way to resolve it rather than conflating the two.
