# RDS/Aurora & database architecture

## The one-line hook

> **Multi-AZ and Read Replicas solve two completely different problems — availability and read scaling — and combining them into one answer, or confusing which one does which, is one of the most common real mistakes in an RDS conversation.**

## Multi-AZ vs. Read Replicas — precisely distinguished

| | Multi-AZ | Read Replica |
|---|---|---|
| **Replication** | Synchronous | Asynchronous |
| **Purpose** | High availability — automatic failover | Read scaling — offloading read traffic from the primary |
| **Readable?** | Traditionally no (though newer RDS Multi-AZ configurations and Aurora support readable standbys) | Yes — that's the entire point |
| **Failover** | Automatic, triggered by RDS itself on primary failure | Manual promotion to standalone primary if needed — a deliberate DR action, not an automatic failover |

**Memorable hook:** *"Multi-AZ is a hot spare waiting to take over the instant the primary fails. A Read Replica is a worker helping carry read load — it can eventually become the primary if you promote it, but that's a decision you make, not something that happens automatically for you."*

They're **not mutually exclusive** — a well-architected production database commonly runs Multi-AZ for availability **and** one or more Read Replicas for read scaling, simultaneously, solving both problems at once with the right tool for each.

## What actually happens during an RDS Multi-AZ failover — a specifically-flagged real question

1. RDS detects the primary instance has failed, via its own health checks.
2. The synchronous standby in the other AZ is **promoted** to become the new primary.
3. RDS updates the database's **DNS CNAME endpoint** to point at the newly-promoted instance.
4. The application, which was already connecting via that same stable endpoint, reconnects automatically — **no application-side configuration change is needed**, since the endpoint itself never changed, only what it resolves to.
5. This typically completes in under a couple of minutes, depending on the database engine and instance size.

**Memorable hook:** *"The application never needs to know a failover happened — it was always talking to a name, not an IP, and RDS just quietly repointed that name to the new primary."*

## Aurora — AWS's own re-architected engine

**Aurora** is AWS's MySQL- and PostgreSQL-compatible engine, built on a genuinely re-architected **distributed, self-healing storage layer** — data is automatically replicated **six ways across three Availability Zones**, with compute and storage decoupled from each other. This architecture gives Aurora meaningfully **faster failover** than standard RDS Multi-AZ, and support for **up to 15 read replicas** (well beyond typical standard RDS limits), since replicas can share the same underlying distributed storage rather than each needing a full independent copy.

**Aurora Serverless v2** scales database capacity automatically — including scaling down close to zero for genuinely intermittent workloads — directly relevant to the cost-optimization theme running through this whole day: paying for a fixed, always-on instance size for a workload that's actually bursty or intermittent is a real, avoidable cost.

## Aurora Global Database — multi-region, forward-referencing later today

**Aurora Global Database** extends a single Aurora cluster across **multiple AWS regions**: one primary region handling writes, with up to five secondary regions receiving data with **typically sub-second replication lag**, each capable of serving low-latency local reads. In a genuine regional outage, a secondary region can be promoted with a **recovery time measured in minutes**, dramatically faster than the alternative of restoring from a cross-region snapshot backup. This is the direct, concrete mechanism behind the multi-region availability judgment covered in full on the next page.

## A brief relational vs. NoSQL decision framework

| | RDS/Aurora (relational) | DynamoDB (NoSQL) |
|---|---|---|
| **Fits** | Complex queries, joins, multi-table transactions, a well-understood, relatively stable schema | Extremely high scale, single-digit-millisecond latency requirements, flexible or evolving schema, key-value/document access patterns |
| **Capacity model** | Instance-sized, provisioned | Provisioned or fully on-demand, scaling automatically with traffic |

This isn't a full DynamoDB deep dive — just enough of a decision framework to correctly redirect a workload that doesn't actually fit the relational model this page is centered on.

## Real-world examples

1. **Walking through RDS Multi-AZ failover mechanics precisely**, step by step as above — a specific, directly research-flagged question, and a strong opportunity to demonstrate real operational understanding rather than a surface-level definition.
2. **Recommending Aurora over standard RDS for a customer needing faster failover and greater read-scaling headroom**, with Aurora Serverless v2 specifically suggested for a workload with genuinely intermittent traffic — directly tying database architecture back to the cost-optimization theme.
3. **Aurora Global Database for a customer needing both disaster recovery and low-latency reads across two geographic regions** — a realistic scenario for an account with both a primary Thailand presence and a secondary region requirement, and a direct, concrete forward-reference to the HA/multi-region judgment material on the next page.
