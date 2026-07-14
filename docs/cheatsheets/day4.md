# Day 4 hooks — Messaging & Event-Driven Architecture

Every hook from [Day 4](../day4/index.md), in page order — 63 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [Kafka internals deep dive](../day4/01-kafka-internals.md)

- **Kafka is a distributed, partitioned, replicated commit log — every other Kafka concept is a consequence of that one sentence, not a separate fact to memorize.**
- *"Kafka doesn't promise your topic is ordered. It promises your partition is ordered — and your key is what decides which partition a given entity's events all end up on."*
- *"`acks=0` is shouting into a room and walking away. `acks=1` is waiting for one person to say they heard you. `acks=all` is waiting for everyone in the room to confirm — slower, but you actually know it landed."*
- *"Retention is a rolling log file — old pages fall off the back. Compaction is a key-value store — it forgets history but never forgets the current answer for a given key."*

## [Kafka stream processing fundamentals](../day4/02-kafka-stream-processing.md)

- **Kafka Streams is a library, not a cluster — it embeds stream processing directly into your own application, using Kafka itself (not a separate system) for both input and fault-tolerant state.**
- *"KStream is Kafka's log-retention mode as an API. KTable is Kafka's log-compaction mode as an API — same underlying storage philosophy from the previous page, now with query semantics on top."*
- *"Stateful Kafka Streams operations aren't actually stateless-Kafka-plus-memory — the state itself is durably backed by a Kafka changelog topic, so crash recovery is just replaying that topic, not hoping the process comes back with its memory intact."*
- *"Tumbling windows are non-overlapping clock ticks. Hopping windows are the same clock ticks, but overlapping. Session windows don't care about the clock at all — they close based on silence, not time."*

## [Kafka geo-replication & multi-datacenter DR](../day4/03-kafka-geo-replication-dr.md)

- **MirrorMaker doesn't give you a magic distributed Kafka cluster spanning regions — it's just another Kafka consumer-and-producer pipeline, which means it inherits all of Kafka's normal replication lag characteristics, including the fact that your recovery point is never truly zero.**
- *"MirrorMaker isn't a special replication protocol — it's an ordinary Kafka consumer reading from one cluster and an ordinary Kafka producer writing to another, wearing a trench coat."*
- *"Active-passive defers the hard problem to failover day. Active-active pays for the hard problem — conflict resolution — every single day, in exchange for never having a failover day at all."*
- *"An offset is an address in one specific cluster's own history. Failing over consumers without offset translation is like handing someone a house number from the wrong street and expecting them to find the right house."*

## [ActiveMQ/IBM MQ vs Kafka: queue vs topic semantics](../day4/04-activemq-ibmmq-vs-kafka.md)

- **A Kafka topic can behave like a queue, or like a pub-sub topic, or like both simultaneously — depending entirely on how many consumer groups you point at it. Traditional JMS brokers make you choose the destination type upfront; Kafka makes the choice at consumption time.**
- *"JMS brokers ask you to pick queue or topic when you create the destination. Kafka asks you nothing upfront — the same topic is a queue to one consumer group and a topic to another, simultaneously, depending purely on how many groups are consuming it."*

## [Dead Letter Queue & poison-pill patterns](../day4/05-dlq-poison-pill-patterns.md)

- **A "poison pill" is a message that will never succeed no matter how many times you retry it — and in an ordered, partition-based system like Kafka, a single poison pill can silently block every message behind it, not just itself.**
- *"A transient failure is worth retrying because the world might be different next time. A poison pill fails because of what the message *is* — the world isn't going to change, so retrying it is just delaying the inevitable."*
- *"In Kafka, a poison pill doesn't just poison itself — it's a roadblock on the one road every message behind it has to travel, because the partition's ordering guarantee doesn't let you drive around it."*

## [Change Data Capture (CDC) deep dive](../day4/06-change-data-capture.md)

- **CDC replaces "periodically ask the database what changed" with "have the database tell you the instant something changes" — by reading its transaction log directly, not by querying application tables at all.**
- *"Log-based CDC reads the database's own diary, after the fact, at no cost to the writer. Trigger-based CDC makes every write also do extra homework. Polling just asks 'anything new?' over and over and hopes it doesn't miss a delete."*
- *"Debezium Server is CDC as infrastructure — bolt it on beside your existing database and app. Debezium Engine is CDC as a library — build it directly into the application, when running a separate Connect cluster is more operational overhead than you want."*

## [The Outbox pattern, fully realized](../day4/07-outbox-pattern-realized.md)

- **The dual-write problem is trying to atomically do two things that have no shared transaction. The outbox pattern's actual trick is turning that into ONE thing — a single local database write — and using CDC to make "publish the event" someone else's problem entirely.**
- *"A hand-built poller is a relay you have to write, test, and operate yourself. CDC on the outbox table is a relay that already exists — you're just pointing an existing, battle-tested tool at one specific table."*

## [The Saga pattern, full treatment](../day4/08-saga-pattern-full-treatment.md)

- **A saga gives you Atomicity, Consistency, and Durability — but not Isolation. It's ACD, not ACID, and the missing "I" is the single most important thing to understand about how sagas actually behave in production.**
- *"Sagas don't just relax isolation as an implementation detail — they remove it as a guarantee entirely. If your design assumes no one can see a saga's in-between state, that assumption is simply false, and semantic locking is the deliberate patch for it."*

## [Choreography vs Orchestration — the decision framework](../day4/09-choreography-vs-orchestration.md)

- **Choreography is a saga with no one in charge. Orchestration is a saga with a single component that knows the whole plan. The real decision isn't which is "better" — it's how many steps, how much branching logic, and how much visibility you actually need.**
- *"Choreography is a dance where everyone already knows their own steps. Orchestration is a conductor with a score, telling every musician exactly when to play — the same analogy from Day 2, now specifically about who's responsible for a saga's actual progress and recovery."*

## [Event Sourcing vs CDC](../day4/10-event-sourcing-vs-cdc.md)

- **In Event Sourcing, the events ARE the database — they're the primary source of truth. In CDC, a conventional database is still the source of truth, and the event stream is just a derived, secondary side effect of it.**
- *"Ask yourself: if you deleted the event stream, would you have lost your data, or just lost a convenient copy of it? In Event Sourcing, you'd have lost everything. In CDC, you'd still have the actual database sitting right there."*
- *"Event Sourcing asks you to redesign how your application thinks about data, from the ground up. CDC plus Outbox asks you to add one table and point a tool at it — a much smaller architectural bet for most teams."*

## [AWS-native messaging — SQS, SNS, EventBridge, Kinesis, MSK](../day4/11-aws-native-messaging.md)

- **Don't pick the service first — name the semantics first: queue, fan-out, router, or replayable log. Once you've said the semantics out loud, the AWS service names itself.**
- *"SQS and SNS are the JMS queue and topic reborn as serverless. Kinesis and MSK are the Kafka log. EventBridge is Day 2's content-based router sold as a managed service."*
- *"SQS's visibility timeout is a lease, not an ack — 'this message is mine for 30 seconds; if I go silent, put it back.'"*
- *"SNS + one SQS queue per consumer = Kafka's consumer groups, assembled from Lego. Each queue is a consumer group's cursor."*
- *"EventBridge is a router, not a firehose — pick it for 'send the right event to the right place,' never for 'move a million events a minute.'"*
- *"MSK when the Kafka ecosystem is the point; Kinesis when not running Kafka is the point."*

## [Day 4 interview Q&A drill](../day4/12-interview-qa.md)

- *"Kafka doesn't promise your topic is ordered — it promises your partition is ordered, and your key decides which partition an entity lands on."*
- *"Kafka is always available, sometimes consistent — and unclean leader election is the actual knob controlling which one wins."*
- *"Two separate distributed systems coordinating with each other is a split-brain risk waiting to happen."*
- *"acks=1 is waiting for one person to confirm they heard you; acks=all is waiting for the whole room."*
- *"Retention is a rolling log file. Compaction is a key-value store that's forgotten its own history."*
- *"KStream is retention-mode-as-an-API. KTable is compaction-mode-as-an-API."*
- *"Kafka Streams is a library your app depends on, not a separate cluster to operate."*
- *"Tumbling windows don't overlap. Hopping windows do. Session windows don't care about the clock at all — they close based on silence."*
- *"MirrorMaker inherits all of Kafka's normal replication lag characteristics — your RPO is never actually zero."*
- *"An offset is an address in one specific cluster's own history — the same number can point to a different message in a different cluster."*
- *"The same topic is a queue to one consumer group and a topic to another, simultaneously."*
- *"Guaranteed, ordered, individually-tracked delivery of high-value transactions is exactly what MQ was built for."*
- *"Kafka's simplicity trades away automatic dead-lettering — you build it at the application level."*
- *"A poison pill doesn't just poison itself — it's a roadblock on the one road every message behind it has to travel."*
- *"Log-based CDC reads the database's own diary at no cost to the writer."*
- *"Tombstones aren't just a delete notification — they're required for log compaction to actually forget the key."*
- *"An idle or broken CDC connector can quietly fill a production database's disk."*
- *"The outbox pattern doesn't make two writes atomic — it turns two writes into one."*
- *"CDC on raw entity tables leaks your internal database schema to every consumer."*
- *"Atomicity of the local write, not full ACID across the whole flow."*
- *"Other processes can observe a saga's half-finished state — that's not a bug, it's a removed guarantee."*
- *"Everything before the pivot can be compensated. Everything after can only be retried."*
- *"Start with orchestration unless you have a strong specific reason not to — it's easier to debug and explain."*
- *"If you deleted the event stream, would you lose your data, or just a convenient copy of it?"*
- *"Name the semantics first — queue, fan-out, router, or log — and the AWS service names itself."*
- *"SQS's visibility timeout is a lease, not an ack."*
- *"SNS plus one SQS queue per consumer = consumer groups assembled from Lego."*
- *"MSK when the Kafka ecosystem is the point; Kinesis when not running Kafka is the point."*
