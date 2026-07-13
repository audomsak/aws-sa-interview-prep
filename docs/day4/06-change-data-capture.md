# Change Data Capture (CDC) deep dive

## The one-line hook

> **CDC replaces "periodically ask the database what changed" with "have the database tell you the instant something changes" — by reading its transaction log directly, not by querying application tables at all.**

## Why CDC beats traditional batch ETL

Traditional batch ETL periodically bulk-exports whole tables — leading to stale data between runs, wasted work reprocessing unchanged rows, and real load on the source system every time it runs. CDC instead captures **row-level changes (inserts, updates, deletes)** as they actually happen, streaming only what changed, continuously — lower latency, lower source impact, and no wasted reprocessing of data that didn't change.

## The three capture methods, compared

| Method | How it works | Tradeoff |
|---|---|---|
| **Log-based** (preferred) | Reads the database's own transaction log directly — the Write-Ahead Log (WAL) in Postgres, binlog in MySQL, redo logs in Oracle | Minimal impact on the source system, correct ordering guaranteed by the log itself, captures deletes correctly — the most robust option where available |
| **Trigger-based** | Database triggers write changes into a side table on every insert/update/delete | Simpler conceptually, but adds real overhead and complexity directly onto the source database's write path |
| **Query-based / polling** | Periodically compares snapshots, or polls using a high-water-mark column | Simplest to set up, but higher latency, and can straightforwardly **miss deletes** entirely if there's no "deleted" flag to poll for |

**Memorable hook:** *"Log-based CDC reads the database's own diary, after the fact, at no cost to the writer. Trigger-based CDC makes every write also do extra homework. Polling just asks 'anything new?' over and over and hopes it doesn't miss a delete."*

## Debezium — the industry-standard implementation

**Debezium** is the de facto standard open-source CDC platform, built on top of **Kafka Connect**. It reads a source database's transaction log directly and produces change events to Kafka topics, with connectors for all the major relational databases (PostgreSQL, MySQL, SQL Server, Oracle, DB2). Each captured change is emitted as a structured **envelope** containing the operation type (insert/update/delete), the **before** and **after** state of the row, source metadata, and transaction information — a self-describing record, not just a raw diff.

Debezium also handles the **initial snapshot** problem automatically: on first startup, it takes a full snapshot of existing data before switching over to continuous log streaming, so downstream consumers get both the current state and every subsequent change, without a separate manual bootstrap step.

### Two deployment models

- **Debezium Server** — a standalone process (a Kafka Connect worker) that connects to the database and streams to Kafka, requiring **zero application code changes**.
- **Debezium Engine** — an embeddable library, letting an application run Debezium's capture logic in-process, without needing a separate Kafka Connect deployment.

**Memorable hook:** *"Debezium Server is CDC as infrastructure — bolt it on beside your existing database and app. Debezium Engine is CDC as a library — build it directly into the application, when running a separate Connect cluster is more operational overhead than you want."*

## Operational gotchas worth naming precisely

- **Tombstone records for deletes.** Debezium emits a **tombstone** (a record with a null value) after a delete event, which is specifically required for Kafka's log compaction (covered on the internals page) to correctly remove the key from a compacted topic entirely — not just mark it deleted.
- **Replication slot growth.** On Postgres specifically, if a Debezium connector falls behind or stops entirely, the database retains WAL segments for that connector's replication slot **indefinitely** by default — a real, concrete way an idle or broken CDC connector can quietly fill a production database's disk. Monitoring slot lag and setting a safety limit on retained WAL size is a genuine, non-optional operational practice.
- **Schema evolution.** When a source table's schema changes, downstream consumers need to handle the shape change gracefully — a **schema registry** with enforced compatibility rules (backward, forward, or full compatibility) is the standard defense against a source schema change silently breaking every downstream consumer at once.
- **Ordering guarantees.** Debezium preserves commit order **within a single table's partition** — strict ordering *across* multiple tables, or across partitions of the same table, isn't automatically guaranteed and needs to be handled deliberately in consumer logic if genuinely required.

## Real-world examples

1. **A modern rebuild of the Marlo/nbn Amdocs billing migration**, using CDC/Debezium instead of the batch data-conversion and cleansing approach actually used for that project historically — a strong, credible "here's how I'd architect this differently with today's tooling" retrospective answer, directly grounded in a real project on your resume.
2. **Syncing data changes from one TnD Microservices service's own database to other interested services**, without a separate batch ETL job — a natural, realistic CDC use case for exactly the kind of decomposed, database-per-service architecture that platform represents.
3. **Explaining the replication slot growth gotcha for a Postgres-backed CDC pipeline** — a specific, credible, non-obvious operational detail that demonstrates real hands-on familiarity rather than surface-level product knowledge.
