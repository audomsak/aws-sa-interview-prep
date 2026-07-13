# Choreography vs Orchestration — the decision framework

Day 2 introduced choreography and orchestration as general integration concepts. This page makes it concrete and specific to Saga coordination — including a real decision framework and the actual tools used in production.

## The one-line hook

> **Choreography is a saga with no one in charge. Orchestration is a saga with a single component that knows the whole plan. The real decision isn't which is "better" — it's how many steps, how much branching logic, and how much visibility you actually need.**

## Choreography-based sagas — decentralized

Each service publishes an event when its local transaction completes; other services listen for events they care about and react independently — there is no central coordinator at all.

| Strength | Weakness |
|---|---|
| No single point of failure — nothing centrally coordinating means nothing centrally breaking | As participant count grows, the full flow becomes genuinely hard to trace — "where is this saga right now?" has no single place to look |
| Simple and natural for small, stable flows | Cyclic event dependencies can emerge as services react to each other in increasingly tangled ways |
| Loose coupling — services only know about events, not about each other directly | Timeouts, retries, and other resiliency patterns must be implemented **individually in every participating service**, not centrally |

## Orchestration-based sagas — centralized

A **saga orchestrator** (sometimes called a Saga Execution Coordinator) explicitly tells each participant what to do, one step at a time, using **command-based, request/reply-style** communication rather than fire-and-forget events — and persists the saga's state (often called a saga log) so it can recover correctly after a crash, resuming exactly where it left off rather than losing track of an in-flight saga.

| Strength | Weakness |
|---|---|
| Clear, centralized visibility into exactly where any given saga is | The orchestrator can become a bottleneck or single point of failure — mitigated by persisting state and making the orchestrator itself replayable/recoverable |
| Handles complex, branching, conditional logic naturally | Tighter coupling — participants now depend on the orchestrator's command protocol |
| Resiliency patterns (timeouts, retries) implemented once, centrally | More infrastructure to build and operate than "just publish events" |

**Memorable hook:** *"Choreography is a dance where everyone already knows their own steps. Orchestration is a conductor with a score, telling every musician exactly when to play — the same analogy from Day 2, now specifically about who's responsible for a saga's actual progress and recovery."*

## The decision framework

| Factor | Favors choreography | Favors orchestration |
|---|---|---|
| **Step count** | 2-4 steps | 5+ steps |
| **Logic complexity** | Simple, linear flow | Conditional branches, parallel steps |
| **Change frequency** | Stable, rarely-changing workflow | Frequently evolving business process |
| **Project context** | Greenfield, services already naturally publish domain events | Brownfield — retrofitting saga behavior onto an existing set of services |
| **Team preference** | Full service autonomy, no shared coordination dependency | Clear, centralized visibility into business process state |

One credible expert opinion worth having ready, since it's a genuinely defensible default: **"start with orchestration unless you have a strong specific reason not to"** — it's easier to debug, easier to maintain, and easier to explain to a team than choreography's distributed, harder-to-trace flow, even though choreography is the theoretically more decoupled option.

**A nuance worth stating too**: many real production systems use **both** — choreography for simple, stable domain events, and orchestration specifically for the complex, auditable business-process-level sagas that genuinely need centralized visibility.

## Real frameworks worth naming

| Framework | Style | Fit |
|---|---|---|
| **Temporal** | Orchestration, treating sagas as long-running durable workflows | Increasingly the modern default for complex, long-running business processes needing strong durability guarantees |
| **Camunda** | Orchestration, BPMN (Business Process Model and Notation) standard | Strong fit where business stakeholders need to see and understand the actual process flow, not just engineers |
| **Eventuate Tram Saga** | Orchestration | Spring Boot/Micronaut-focused, a natural fit for a Java shop already in that ecosystem |
| **Axon Saga** | Choreography-oriented | Also Spring Boot-focused, lighter-weight for event-driven-first teams |
| **Eclipse MicroProfile LRA** | Choreography | REST/HTTP-transport-based distributed transaction implementation |

## Real-world examples

1. **A TnD Microservices-style order-processing flow starting as simple 3-service choreography, then needing to evolve toward orchestration as conditional logic grew** — a realistic, credible "how architecture actually evolves under real requirements" narrative, rather than a static, one-time design decision.
2. **Red Hat's own BPMN-adjacent tooling heritage** (the jBPM/Camunda-adjacent ecosystem) as a natural connection point to your Red Hat background — a concrete, personal link to orchestration-style tooling positioning conversations you'd plausibly have had.
3. **Recommending Temporal (or a similar durable execution platform) for a complex, long-running, auditable business process** — a loan origination workflow in a Thai banking context is a strong, realistic example — where centralized visibility and durability guarantees genuinely matter more than choreography's looser coupling.
