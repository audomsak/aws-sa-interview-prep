# Performance & scalability

This page directly targets the "100,000 messages a minute from Kafka" and "multi-GB file without running out of memory" scenarios research flagged as common at the senior level — both are fundamentally about the same discipline: never holding more in memory, or doing more synchronously, than the workload actually requires.

## The one-line hook

> **Camel is single-threaded per exchange by default. Every performance question is really asking: where, deliberately, did you introduce concurrency or streaming — and why there specifically?**

## The threading model, from the default upward

By default, a Camel route processes one Exchange synchronously, step by step, on the thread that received it. That's simple and predictable — and a real bottleneck the moment the workload demands otherwise. Camel gives you several deliberate ways to change that:

| Mechanism | What it changes | Fits |
|---|---|---|
| **`threads()` DSL** | Explicitly hands off processing to a configured thread pool at that point in the route | A specific slow step (a downstream call) that shouldn't block the rest of the route |
| **`parallelProcessing()` on Splitter/Multicast** | Processes each split/multicast branch concurrently instead of sequentially | Fan-out scenarios — calling 3 independent downstream services, or processing split batch items |
| **Consumer concurrency** (e.g. Kafka `consumersCount`, JMS `concurrentConsumers`) | Runs multiple consumer threads pulling from the same source in parallel | Raw ingestion throughput — directly the lever for the 100K messages/minute scenario |

## `direct`, `seda`, and `vm` — the internal routing components, compared

A frequently confused trio, and a good trap question:

| Component | Execution | Scope |
|---|---|---|
| **`direct`** | Fully **synchronous** — the calling route blocks until the target route completes | Same CamelContext only |
| **`seda`** | **Asynchronous**, via an in-memory queue — the calling route hands off and continues immediately | Same CamelContext only (same JVM) |
| **`vm`** | Asynchronous, same in-memory queue model as `seda` | Across **multiple CamelContexts in the same JVM** — `seda` can't cross that boundary, `vm` can |

**Memorable hook:** *"`direct` is handing something to someone standing right next to you, and waiting. `seda` is dropping it in an internal mailbox and walking away. `vm` is the same mailbox, but shared between two separate departments in the same building."*

**The critical caution with `seda`/`vm`:** their internal queues are, by default, unbounded — under sustained load faster than the downstream can drain, that queue grows without limit and becomes exactly the kind of uncontrolled memory growth that causes an out-of-memory crash. Setting an explicit bounded queue size, with a defined rejection/blocking policy once full, is the real, production-grade configuration — not the default.

## Backpressure — slowing the producer down on purpose

**Backpressure** is the mechanism by which a slow consumer signals an upstream producer to *stop sending so fast*, rather than the consumer silently falling further and further behind (or crashing). Bounded queues (as above) are one crude form of it — the queue filling up naturally applies backward pressure once producers start blocking on a full queue. Camel's reactive streams support (bridging to Reactor/RxJava-style backpressure-aware streams) is the more sophisticated version, letting a slow downstream service explicitly throttle how fast an upstream Kafka consumer pulls new records, rather than pulling at full speed and burying the system in an internal queue regardless.

**Memorable hook:** *"Backpressure is the consumer saying 'slow down' instead of the producer finding out the hard way that the consumer fell over."*

## Streaming — the direct answer to "multi-GB file without OOM"

Two changes, used together, are the complete answer:

1. **Streaming parsers instead of full in-memory parsing.** A traditional DOM-style XML parser loads the *entire* document into memory as a tree before you can process any of it — fatal for a multi-GB file. A streaming parser (StAX for XML, a streaming JSON parser) processes the document incrementally, element by element, never holding the whole thing in memory at once.
2. **Streaming mode on the Splitter (`streaming()`).** By default, Camel's Splitter can materialize the *entire* list of split parts before processing any of them — which defeats the purpose if the source itself was streamed. Enabling streaming mode processes and releases each split part as it's produced, so memory usage stays roughly constant regardless of the total file size.

Combined with the **Claim Check** pattern from the EIP page — never carrying the full payload through the route body at all, only a reference to it — this is the complete, credible answer to the "multi-GB file" scenario, not just "increase the JVM heap."

## Aggregator performance: in-memory vs persistent

The **Aggregator** EIP (covered on the EIP catalog page) needs somewhere to hold partially-aggregated state while waiting for its completion condition. For high-volume or long-running aggregations, an **in-memory `AggregationRepository`** is both a memory-growth risk and a durability risk — a crash loses all in-flight aggregation state. A **persistent, JDBC-backed `AggregationRepository`** survives restarts and bounds memory growth, at the cost of a database round-trip per aggregation step — a real, worth-naming tradeoff rather than a default nobody questions.

## Real-world examples

1. **The 100,000 messages/minute Kafka scenario from research.** The complete, credible answer: match Kafka consumer concurrency to the topic's partition count (more consumer threads than partitions buys nothing), use `parallelProcessing` for any independent fan-out to downstream services, and put a bounded `seda` queue between fast ingestion and any slower downstream call so a temporary slowdown doesn't cascade into an OOM crash.
2. **The multi-GB file scenario**, solved concretely with streaming parsers, streaming Splitter mode, and Claim Check together — a specific, technically complete answer rather than "just process it in chunks."
3. **Scaling the TnD Microservices platform on Kubernetes**, connecting directly back to Day 1: horizontal pod autoscaling only helps throughput if Kafka consumer group sizing (partition count vs consumer count) is actually configured to take advantage of the extra replicas — more pods with a partition count that can't support more parallel consumers buys nothing, a genuinely strong cross-topic answer.
