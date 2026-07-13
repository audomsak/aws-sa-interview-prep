# Kafka stream processing fundamentals

## The one-line hook

> **Kafka Streams is a library, not a cluster — it embeds stream processing directly into your own application, using Kafka itself (not a separate system) for both input and fault-tolerant state.**

## Why this is different from standing up Flink or Spark

Unlike Apache Flink or Spark Streaming, which run as their own separate processing clusters, the **Kafka Streams API** is just a Java library your application depends on — the "cluster" is really just multiple instances of your own application, each processing a subset of partitions, coordinating through Kafka's existing consumer group mechanism. This is a genuine sizing decision worth being able to articulate: for straightforward stream transformations, embedding Kafka Streams directly avoids the operational cost of standing up and running an entirely separate processing platform.

## KStream vs KTable — the two core abstractions

| | KStream | KTable |
|---|---|---|
| **Represents** | A stream of independent events/records | The **current state** of each key — a changelog, conceptually a table |
| **Directly connects to** | An ordinary Kafka topic | A **compacted** Kafka topic — the exact log compaction mechanism from the previous page |
| **Analogy** | Every row is a fact that happened | Every row is the *latest* fact — new records overwrite, they don't accumulate |

**Memorable hook:** *"KStream is Kafka's log-retention mode as an API. KTable is Kafka's log-compaction mode as an API — same underlying storage philosophy from the previous page, now with query semantics on top."*

## Stateless vs stateful operations

- **Stateless**: `map`, `filter`, `flatMap` — each record is transformed independently, with no memory of prior records needed.
- **Stateful**: aggregations (`count`, `reduce`), and **joins** (stream-stream, stream-table, table-table) — these require a **local state store** (backed by RocksDB by default) plus a **changelog topic** that continuously backs up that local state to Kafka, so if an instance crashes, another instance can rebuild the exact same state from the changelog rather than losing it.

**Memorable hook:** *"Stateful Kafka Streams operations aren't actually stateless-Kafka-plus-memory — the state itself is durably backed by a Kafka changelog topic, so crash recovery is just replaying that topic, not hoping the process comes back with its memory intact."*

## Windowing — a genuine interview differentiator

| Window type | Behavior |
|---|---|
| **Tumbling** | Fixed-size, **non-overlapping** — every record belongs to exactly one window |
| **Hopping** | Fixed-size, but windows can **overlap** — a record may belong to multiple windows |
| **Sliding** | Windows defined relative to record timestamps rather than clock boundaries — driven by event proximity, not fixed clock ticks |
| **Session** | Defined by a **gap of inactivity** — a window stays open as long as new records keep arriving within the gap threshold, and closes once that gap is exceeded |

**Memorable hook:** *"Tumbling windows are non-overlapping clock ticks. Hopping windows are the same clock ticks, but overlapping. Session windows don't care about the clock at all — they close based on silence, not time."*

## Event time vs. processing time — why this matters for correctness

**Event time** is when something actually happened (the timestamp embedded in the record); **processing time** is when your application happens to process it. Network delays, retries, and consumer lag mean these are routinely different, and out-of-order or late-arriving data is a normal, expected condition in any real stream — not an edge case. Correct windowing logic needs to reason in event time, with some tolerance (a "grace period") for how late a record can arrive and still be included in its correct window, rather than naively bucketing by whenever the record happened to be processed.

## Exactly-once processing, extended

Day 2 already covered Kafka's idempotent producer and transactional writes as the mechanism behind Kafka's own exactly-once semantics. Kafka Streams builds directly on that same mechanism, extending it across the full **consume → process → produce** loop for a stream processing application — so a crash mid-processing doesn't result in either a lost update or a duplicated one, using the same underlying idempotent-producer-plus-transactions foundation, not a separate mechanism.

## Real-world examples

1. **Real-time aggregation on the TnD Microservices platform** — counting events per service, or detecting anomalies over a sliding window of recent activity — a plausible, concrete extension of that platform's actual Kafka usage.
2. **Session windows for tracking customer usage sessions**, directly relevant given your telecom/nbn background — grouping a customer's activity events into sessions based on gaps of inactivity is a textbook session-windowing use case in exactly that kind of domain.
3. **Choosing Kafka Streams over standing up a separate Flink cluster** when the transformation logic is straightforward enough to embed directly into an existing Java service already talking to Kafka — a real, defensible "avoid unnecessary operational complexity" sizing decision.
