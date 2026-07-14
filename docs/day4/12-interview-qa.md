# Day 4 interview Q&A drill

**How to use this page:** cover the answers, read only the question, answer out loud as if the interviewer is sitting across from you, *then* check. The goal isn't to reread — it's to catch the gap between what you can say cold and what you can only recognize when you see it.

---

**1. Why does Kafka guarantee ordering only within a partition, and how do you get ordering for a specific entity's events?**

> **Hook: "Kafka doesn't promise your topic is ordered — it promises your partition is ordered, and your key decides which partition an entity lands on."**
> Partitions are Kafka's unit of parallelism, each an independent, ordered log — there's no ordering guarantee across different partitions of the same topic. Messages sharing the same key are deterministically hashed to the same partition, which is how you get per-entity ordering: all events for a given order or customer land on one partition and are processed in the order they were produced, even as other entities' events are interleaved across other partitions.

---

**2. Explain In-Sync Replicas (ISR) and the tradeoff involved in unclean leader election.**

> **Hook: "Kafka is always available, sometimes consistent — and unclean leader election is the actual knob controlling which one wins."**
> The ISR is the set of replicas fully caught up with a partition's leader within a defined time window; only an ISR member is normally allowed to become the new leader on failover, to avoid promoting a replica that's missing recent writes. If every ISR member is unavailable and unclean leader election is disabled (the safer default), Kafka waits rather than risk data loss. Enabling unclean leader election trades that consistency guarantee for availability, promoting a non-ISR replica that might be missing data just to keep the partition serving traffic.

---

**3. What problem did KRaft solve that ZooKeeper-based coordination had?**

> **Hook: "Two separate distributed systems coordinating with each other is a split-brain risk waiting to happen."**
> ZooKeeper handled cluster metadata and coordination as a genuinely separate system from Kafka itself. KRaft folds that metadata management directly into the Kafka brokers via a Raft-based controller quorum, removing the operational and correctness risk of keeping two independent distributed systems consistent with each other, and allowing clusters to scale to far more partitions than ZooKeeper-based coordination comfortably supported.

---

**4. When would you choose `acks=1` over `acks=all`, given the durability difference?**

> **Hook: "acks=1 is waiting for one person to confirm they heard you; acks=all is waiting for the whole room."**
> `acks=all` waits for the full ISR to acknowledge, giving the strongest durability at the cost of higher latency — the right choice for anything where message loss is genuinely unacceptable, like a financial transaction event. `acks=1` only waits for the leader, accepting a small risk of loss if the leader fails before followers replicate, in exchange for lower latency — a reasonable tradeoff for high-volume, more loss-tolerant data like telemetry or analytics events.

---

**5. What's the difference between Kafka's retention-based deletion and log compaction, and when would you use compaction?**

> **Hook: "Retention is a rolling log file. Compaction is a key-value store that's forgotten its own history."**
> Time or size-based retention deletes entire old segments once they age out or exceed a size limit, regardless of key. Log compaction instead keeps only the latest value per key, using tombstone records (null values) to mark deletions during the next compaction pass. Compaction fits topics where you only care about current state per entity — CDC topics and changelog-style use cases in particular, exactly like a Kafka Streams KTable.

---

**6. Explain KStream vs KTable, and how KTable connects back to log compaction.**

> **Hook: "KStream is retention-mode-as-an-API. KTable is compaction-mode-as-an-API."**
> A KStream represents a stream of independent events — every record is a distinct fact. A KTable represents the current state per key — a changelog, conceptually backed by a compacted topic, where new records for a key overwrite rather than accumulate. This is the same underlying storage philosophy as log compaction, just exposed with query-like semantics on top.

---

**7. Why might a team choose Kafka Streams over standing up a separate Flink or Spark cluster?**

> **Hook: "Kafka Streams is a library your app depends on, not a separate cluster to operate."**
> Kafka Streams embeds directly into your own application as a Java library, using Kafka's existing consumer group mechanism for coordination — there's no separate processing cluster to stand up and operate. For straightforward stream transformations, this avoids meaningful operational overhead compared to running an entirely separate stream processing platform, at the cost of being less feature-rich for very advanced processing needs.

---

**8. What's the actual difference between tumbling, hopping, and session windows?**

> **Hook: "Tumbling windows don't overlap. Hopping windows do. Session windows don't care about the clock at all — they close based on silence."**
> Tumbling windows are fixed-size and non-overlapping — every record belongs to exactly one window. Hopping windows are also fixed-size but can overlap, so a single record may fall into multiple windows. Session windows are defined by a gap of inactivity rather than fixed clock boundaries — the window stays open as long as records keep arriving within the gap threshold, closing once that gap is exceeded.

---

**9. Why is MirrorMaker described as "just another Kafka consumer and producer," and why does that matter for your recovery point objective?**

> **Hook: "MirrorMaker inherits all of Kafka's normal replication lag characteristics — your RPO is never actually zero."**
> MirrorMaker 2 is built on Kafka Connect and functions as an ordinary consumer reading from a source cluster and an ordinary producer writing to a target cluster — it's not a special zero-lag replication protocol. Because it's asynchronous like any other Kafka pipeline, there's always some replication lag, meaning a genuinely honest DR design states an RPO bounded by that lag, not an assumption of zero data loss on failover.

---

**10. What's the consumer offset translation problem in cross-cluster failover, and why does it matter?**

> **Hook: "An offset is an address in one specific cluster's own history — the same number can point to a different message in a different cluster."**
> Kafka consumer offsets are specific to the cluster they were committed against. If consumers fail over from cluster A to cluster B, offset 4,521 in cluster A isn't guaranteed to be the same message as offset 4,521 in cluster B's replicated copy, since replication timing can shift positions. MirrorMaker's offset translation feature maintains a mapping so failed-over consumers resume from the semantically correct position, not just a matching offset number.

---

**11. How does a single Kafka topic behave like both a queue and a pub-sub topic, depending on consumption pattern?**

> **Hook: "The same topic is a queue to one consumer group and a topic to another, simultaneously."**
> Point one consumer group (with multiple instances) at a topic, and each message goes to exactly one consumer within that group — competing-consumers, queue-like behavior. Point multiple independent consumer groups at the same topic, and each group receives its own full copy of every message — pub-sub behavior. Unlike JMS, where you choose queue or topic upfront at the destination level, Kafka makes this choice implicitly, based purely on how many groups are consuming.

---

**12. When would you recommend IBM MQ over Kafka for a customer's integration platform?**

> **Hook: "Guaranteed, ordered, individually-tracked delivery of high-value transactions is exactly what MQ was built for."**
> For mission-critical, transactional workloads — banking transaction processing being the canonical case — IBM MQ's mature per-message features (priority, TTL, redelivery policies), enterprise-grade compliance posture, and destructive-read delivery model fit the requirement better than Kafka's simpler, replay-oriented pull model. Kafka is the better fit when the priority is high-throughput event streaming feeding multiple independent downstream consumers with replay capability.

---

**13. Why doesn't Kafka provide automatic dead-letter routing the way JMS brokers or Camel do?**

> **Hook: "Kafka's simplicity trades away automatic dead-lettering — you build it at the application level."**
> Traditional JMS brokers and frameworks like Camel provide built-in dead-letter routing as a broker or framework feature. Kafka doesn't — a failing consumer has to explicitly catch the processing exception, produce the failed message to a separate DLQ topic itself, and only then commit the offset to move past it. The one exception is Kafka Connect, which does provide native dead letter queue support via configuration, relevant directly to CDC pipelines.

---

**14. Explain head-of-line blocking in Kafka and why it's a bigger risk than in a traditional queue.**

> **Hook: "A poison pill doesn't just poison itself — it's a roadblock on the one road every message behind it has to travel."**
> Because Kafka guarantees ordering within a partition, a consumer generally can't skip past a failing message without breaking that guarantee for everything behind it — so a single poison pill can block every subsequent message in that partition indefinitely. A traditional queue can more often set aside an individual problem message without blocking unrelated messages behind it. The fix is the same application-level DLQ pattern: detect the deterministic failure, route it to a DLQ, and deliberately advance the offset past it.

---

**15. What's the difference between log-based, trigger-based, and query-based (polling) CDC, and which is generally preferred?**

> **Hook: "Log-based CDC reads the database's own diary at no cost to the writer."**
> Log-based CDC reads the database's transaction log directly (WAL, binlog, redo logs), with minimal impact on the source and correct ordering guaranteed by the log itself — the generally preferred approach. Trigger-based CDC uses database triggers to write changes to a side table, adding overhead directly to the source's write path. Query-based/polling periodically compares snapshots or polls a high-water-mark column, and can straightforwardly miss deletes entirely if there's no explicit deleted flag.

---

**16. Why does Debezium emit a tombstone record after a delete event?**

> **Hook: "Tombstones aren't just a delete notification — they're required for log compaction to actually forget the key."**
> A tombstone is a record with a null value for a given key. It's specifically required so that Kafka's log compaction can correctly remove that key entirely from a compacted topic during the next compaction pass, rather than just marking it deleted while the key's history still lingers.

---

**17. What's the replication slot growth problem in a Postgres-based CDC pipeline, and why is it a real operational risk?**

> **Hook: "An idle or broken CDC connector can quietly fill a production database's disk."**
> If a Debezium connector falls behind or stops entirely, Postgres retains WAL segments for that connector's replication slot indefinitely by default, since the database doesn't know it's safe to discard that history yet. An unmonitored, stalled connector can silently grow WAL retention until it fills the source database's disk — a genuine, non-obvious operational risk that needs active slot-lag monitoring and a configured safety limit.

---

**18. What's the dual-write problem, and how does the outbox pattern actually solve it?**

> **Hook: "The outbox pattern doesn't make two writes atomic — it turns two writes into one."**
> The dual-write problem is the risk of a process crashing between two independent writes — updating a database and separately publishing an event — leaving them inconsistent. The outbox pattern writes the business data change and an outbox row in the same local database transaction, so an ordinary single-resource transaction already guarantees both happen together, with no XA needed. A separate relay (ideally CDC) then reliably propagates the outbox row onward.

---

**19. Why use CDC on the outbox table specifically, rather than CDC'ing your actual business tables directly?**

> **Hook: "CDC on raw entity tables leaks your internal database schema to every consumer."**
> The outbox table is a deliberately designed, stable event schema — a contract. CDC'ing raw business tables directly would couple every downstream consumer to your internal database structure, so any internal refactor (renaming a column, restructuring a table) becomes a breaking change for external consumers. CDC'ing the outbox table keeps that internal/external boundary clean, the same instinct behind API contract stability.

---

**20. What guarantee does the outbox pattern actually provide, and what does it NOT provide?**

> **Hook: "Atomicity of the local write, not full ACID across the whole flow."**
> It guarantees the business change and the intent-to-publish record are atomically consistent with each other locally — never one without the other. It does not make the overall flow synchronously consistent: propagation to Kafka is still asynchronous, so the system is eventually consistent, with a real (if usually short) window where the database has changed but downstream consumers haven't yet been notified.

---

**21. Explain why sagas are described as "ACD, not ACID," and what the missing Isolation actually means in practice.**

> **Hook: "Other processes can observe a saga's half-finished state — that's not a bug, it's a removed guarantee."**
> Each saga step commits independently and immediately, with no distributed lock holding the whole transaction open. This gives Atomicity (via compensation), Consistency (eventual), and Durability (each local commit is durable) — but not Isolation, since a concurrent process can genuinely observe a saga's intermediate, partially-completed state while it's still in flight. A concrete example: one saga's in-flight inventory reservation can be read by a concurrent saga as if it were final, before the first saga's later steps even complete.

---

**22. What is a pivot transaction in a saga, and why does its placement matter?**

> **Hook: "Everything before the pivot can be compensated. Everything after can only be retried."**
> The pivot transaction is the dividing line between a saga's reversible and irreversible steps — either the last undoable step, or the first retryable one. Steps after the pivot can't be cleanly compensated (you can't take back a sent email), so they're handled by retrying until they succeed instead. Good saga design deliberately moves the pivot as late as possible, placing genuinely non-compensatable actions only after it.

---

**23. When would you choose choreography over orchestration for a saga, and what's a reasonable default?**

> **Hook: "Start with orchestration unless you have a strong specific reason not to — it's easier to debug and explain."**
> Choreography fits simple, stable flows with 2-4 steps and services that already naturally publish domain events, prioritizing full autonomy and no single point of failure. Orchestration fits complex flows with 5+ steps, conditional branching, or parallel steps, where centralized visibility into saga progress matters more than avoiding a coordinating component. Many real systems use both — choreography for domain events, orchestration for the business-process-level saga that needs clear, auditable visibility.

---

**24. What's the actual difference between Event Sourcing and CDC, since they both involve streams of change events?**

> **Hook: "If you deleted the event stream, would you lose your data, or just a convenient copy of it?"**
> In Event Sourcing, the sequence of events IS the primary source of truth — current state is derived by replaying or folding events, and deleting the event stream would mean losing everything. In CDC, a conventional database remains the source of truth, and the event stream is a derived, secondary side effect of ordinary writes — deleting it would lose a convenient copy, not the actual data. CDC combined with the outbox pattern is usually the more pragmatic choice for teams with existing conventional CRUD services, since it doesn't require the upfront event-first redesign full Event Sourcing demands.

---

**25. You clearly know Kafka. On AWS, when would you pick SQS over Kinesis or MSK?**

> **Hook: "Name the semantics first — queue, fan-out, router, or log — and the AWS service names itself."**
> If the requirement is a durable work queue with competing consumers and no need to re-read history, that's queue semantics — SQS, and reaching for a retained log there is over-engineering. If the requirement is a high-throughput ordered stream that multiple independent readers consume and can replay, that's log semantics — Kinesis or MSK. Fan-out to several independent consumers is SNS (usually SNS→SQS), and content-based routing across many targets or SaaS/cross-account integration is EventBridge. Stating the semantics before the service name is exactly what separates a considered answer from a brochure answer.

---

**26. How does SQS handle a consumer that crashes mid-processing, and how does that compare to Kafka's model?**

> **Hook: "SQS's visibility timeout is a lease, not an ack."**
> On receive, the message becomes invisible to other consumers for the visibility timeout; if the consumer deletes it in time, it's gone — a destructive read, like a JMS queue — and if the consumer goes silent, the message reappears for redelivery, with a redrive policy moving it to a DLQ after `maxReceiveCount` attempts. Kafka has no per-message lease at all: a consumer just advances its group's offset, and a crash means the group rebalances and the next consumer resumes from the last committed offset. Both deliver at-least-once, so idempotent consumers are required either way — but SQS tracks each message individually while Kafka tracks only a cursor.

---

**27. How would you get Kafka's "one topic, many consumer groups" behavior from serverless AWS services?**

> **Hook: "SNS plus one SQS queue per consumer = consumer groups assembled from Lego."**
> Publish the event once to an SNS topic and subscribe one SQS queue per downstream consumer — each queue is that consumer's own durable buffer, drained independently at its own pace, exactly the role a consumer group's offset plays in Kafka. Subscription filter policies even give per-consumer content filtering. The honest gap to volunteer: there's no replay — SQS is a destructive read, so a new consumer added later starts from now, whereas a new Kafka consumer group can rewind the whole retained log.

---

**28. Kinesis vs MSK — same log semantics, so how do you actually choose?**

> **Hook: "MSK when the Kafka ecosystem is the point; Kinesis when not running Kafka is the point."**
> MSK is literally managed Kafka, so existing producer/consumer code, Kafka Streams applications, and above all Kafka Connect pipelines — including Debezium, which means the whole CDC/outbox architecture from today — carry over unchanged. Kinesis wins on the operations story: serverless scaling, IAM-native auth, no broker or partition management, with a near one-to-one concept mapping (shard≈partition, sequence number≈offset, KCL application≈consumer group). So the decision is mostly about what you're carrying: an existing Kafka estate or ecosystem dependency says MSK; a greenfield AWS-only build with no Kafka operational experience says Kinesis.
