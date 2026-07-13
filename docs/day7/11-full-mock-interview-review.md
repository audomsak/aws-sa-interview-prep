# Full 7-day cross-topic mock interview & review

This page is different from every Q&A drill so far. Every other day's drill stayed within that day's topic. A real interview won't — a single scenario question can easily need Day 1's containers, Day 4's messaging, and Day 6's AWS architecture all at once. This page is built specifically to force that integration before the real interview does it for you, live.

## How to use this page

Work through it in order: the framework first, then the two full scenario questions (treat these exactly like the real interview's "2 scenario architecture questions" — talk out loud, sketch if it helps, don't peek at the model answer outline until you've genuinely tried), then the rapid-fire cross-topic drill, then the week-in-review table as a final memory check before the actual interview.

---

## A framework for scenario architecture questions

When you get an open-ended "design X" prompt, work through these five steps out loud, in order — this structure is itself part of what's being evaluated, not just the final answer:

1. **Clarify requirements** — scale, latency/availability targets, consistency needs, budget constraints, team constraints. Don't design in a vacuum.
2. **Identify the real constraints and trade-offs** — what are you optimizing for, and what are you explicitly willing to sacrifice? (Direct callback to Day 6's Well-Architected pillars-in-tension material.)
3. **Propose a high-level architecture** — the major components and how they connect, before diving into any one of them.
4. **Deep-dive on 1-2 genuinely critical components** — the interviewer will usually steer you here; go deep on whichever piece actually carries the risk.
5. **Discuss failure modes and scaling** — what breaks first, and how does the design degrade gracefully rather than catastrophically? (Direct callback to Day 5's entire resilience material.)

**Memorable hook:** *"An interviewer scoring a scenario question is watching your process at least as much as your final diagram — clarifying requirements first is itself a signal, not a delay."*

---

## Scenario question 1

> **"Design a system that ingests real-time transaction events from a Thai bank's core banking system, processes them for fraud detection, and makes the result available to a customer-facing mobile app within 2 seconds — while ensuring no transaction is ever lost or double-processed, even during partial failures."**

Work through the five-step framework yourself before reading the outline below.

<details markdown>
<summary>Model answer outline — expand only after attempting your own</summary>

- **Requirements clarified**: sub-2-second end-to-end latency, zero data loss, no double-processing, real-time — this immediately rules out batch processing and points toward a streaming architecture.
- **Ingestion**: Kafka (Day 4) as the event backbone — partitioned by account or transaction ID for per-entity ordering, `acks=all` given the zero-loss requirement, directly justified by Day 4's precise breakdown of the `acks` durability tradeoff.
- **Exactly-once outcome**: not true exactly-once delivery (Day 4's honest framing) — at-least-once delivery plus an idempotent consumer keyed on transaction ID, using a distributed idempotent repository (not in-memory, since this runs on multiple replicas — a direct Day 1/Day 2 cross-reference).
- **Fraud processing**: Kafka Streams (Day 4) for real-time stream processing, likely windowed aggregation for pattern detection (velocity checks, unusual amount patterns).
- **Resilience**: circuit breaker and bulkhead (Day 5) around any external fraud-scoring service call, with a conservative fallback (flag for manual review rather than silently approving) if that dependency degrades.
- **Delivery to mobile**: API Gateway (Day 3, Kong) fronting the result, with the actual result pushed via a fast path — possibly a WebSocket or a poll-optimized endpoint, kept genuinely separate from the ingestion pipeline's own latency budget.
- **Infrastructure**: EKS (Day 6, and a direct nod to your OpenShift background) for the processing services, Multi-AZ at minimum, with the explicit judgment call (Day 6) that Multi-Region likely isn't justified here purely for availability — but might be, for Thai data residency reasons specifically.
- **Observability**: distributed tracing (Day 5) with a shared trace ID across the whole pipeline, SLOs (Day 7) defined directly against the 2-second latency requirement, with burn-rate alerting rather than static thresholds.

</details>

---

## Scenario question 2

> **"A customer wants to migrate a monolithic, on-premises JBoss Fuse integration platform — similar to your own nbn iB2B background — to AWS, with zero downtime during the migration and the ability to roll back at any point. Walk me through your approach."**

Work through the five-step framework yourself before reading the outline below.

<details markdown>
<summary>Model answer outline — expand only after attempting your own</summary>

- **Requirements clarified**: zero downtime, rollback capability at any point, an existing monolithic integration platform — this is explicitly a migration strategy question, not a greenfield design question.
- **Migration strategy (Day 6)**: given AWS's own guidance against refactoring during a large migration, the honest recommendation is **rehost or replatform first** — likely replatforming the JBoss Fuse workload onto EKS or ECS (Day 6's containers page) rather than a risky big-bang refactor to microservices during the move itself.
- **The Strangler Fig pattern**: incrementally route traffic away from the legacy monolith to new services standing in front of it (an API Gateway, Day 3, is the natural place to do this routing), rather than a single cutover — this directly delivers the "rollback at any point" requirement, since traffic can be routed back to the legacy system for any not-yet-migrated piece.
- **Zero downtime mechanics**: blue-green or canary deployment (this same day's earlier page) for each incrementally strangled piece, not a single all-at-once switch.
- **Data**: if the legacy platform owns its own database, CDC (Day 4) can stream changes to a new data store during the transition, keeping both old and new systems consistent while the migration is in flight.
- **Refactor later, not during**: once fully migrated and stable on AWS, *then* consider refactoring toward the kind of microservices decomposition your own TnD Microservices platform represents — directly citing Day 6's explicit sequencing guidance (rehost/replatform first, refactor after).
- **The direct personal callback worth making explicitly**: "This is genuinely close to my own nbn Australia experience — Marlo's JBoss Fuse/Camel platform, and the later TnD Microservices decomposition, were effectively this exact strategy applied in sequence."

</details>

---

## Rapid-fire cross-topic drill

Each question deliberately spans two or more days. Answer out loud, then check.

**1.** *(Day 1 + Day 4)* A Kafka consumer service is scaled to 5 pod replicas on Kubernetes. Why can't it safely use an in-memory idempotent repository, and what's the fix?
> Each pod has its own memory — a duplicate message routed to a different replica than the one that first processed it wouldn't be caught. Use a shared, durable repository (JDBC or a distributed cache) instead.

**2.** *(Day 2 + Day 4)* How does the outbox pattern from Day 2's preview become fully real using Day 4's material?
> CDC (via Debezium) reads the outbox table's own transaction log and streams new rows to Kafka automatically — replacing a hand-built polling relay entirely.

**3.** *(Day 3 + Day 5)* How does Envoy's outlier detection differ from application-level circuit breaking, and where would you configure it?
> Outlier detection ejects a specific unhealthy replica from a load-balancing pool, rather than opening a binary circuit for an entire dependency — configured at the mesh/Envoy layer (Day 3), not in application code.

**4.** *(Day 1 + Day 6)* Your OpenShift background applies directly to which AWS service, and why?
> EKS — OpenShift is itself a Kubernetes distribution, so the CRI/CNI/CSI/Operator model from Day 1 applies directly to EKS's control plane, with the main new detail being the AWS VPC CNI's real-IP-per-pod model instead of an overlay network.

**5.** *(Day 4 + Day 5)* Why does a saga's lack of isolation matter specifically in a Kafka-based choreographed saga?
> Kafka's partition-level ordering only guarantees order within a partition — a concurrent saga reading intermediate state (the semantic lock problem) can observe a partially-completed saga's effects before compensation, if it happens to read from an already-updated resource mid-saga.

**6.** *(Day 3 + Day 6)* Why might AWS API Gateway beat Kong for a customer fully committed to AWS serverless, despite Kong's portability advantage?
> First-party integration with Lambda and Cognito removes real friction a third-party gateway would need extra configuration to bridge — the AWS-native option wins on integration depth specifically when there's no multi-cloud requirement.

**7.** *(Day 5 + Day 7)* How does an error budget change how you'd respond to a circuit breaker tripping frequently in production?
> Frequent trips are burning the error budget — if it's being exhausted faster than the SLO period allows, that's a signal to halt feature work and prioritize fixing the underlying dependency, not just tuning the circuit breaker's threshold.

**8.** *(Day 2 + Day 7)* How does Camel K's Operator model connect to this day's GitOps material?
> Both are the same reconciliation loop pattern — Camel K's controller reconciles an `Integration` custom resource toward its declared state, exactly like ArgoCD/Flux reconcile a cluster toward what's declared in Git; the pattern from Day 1's `kube-controller-manager` resurfaces a third time.

**9.** *(Day 1 + Day 7)* Why does the "non-root container" discipline from Day 1 matter for this day's DevSecOps pipeline specifically?
> It's the same Linux capabilities-dropping principle from building a container by hand, now enforced as an automated, pipeline-checked security policy rather than a one-off manual configuration choice.

**10.** *(Day 4 + Day 6)* Why would you choose Aurora Global Database over standard cross-region replication for a multi-region requirement?
> Sub-second replication lag and fast regional promotion (minutes, not a full snapshot restore) — directly relevant if the Day 6 judgment call concludes Multi-Region is actually justified, not just Multi-AZ.

**11.** *(Day 3 + Day 4)* How would an AI Gateway's token-based rate limiting reuse Day 4's Kafka acks tradeoff thinking?
> Both are the same underlying "how much am I willing to wait/pay for certainty" tradeoff — `acks=all` trades latency for durability guarantees; token-based rate limiting trades throughput for cost control, using the same token bucket algorithm from Day 3's rate limiting page, just metering a different unit.

**12.** *(Day 5 + Day 6)* Why is chaos engineering directly relevant to validating a Multi-Region DR design, not just circuit breakers?
> An untested DR failover plan is exactly the same risk as an untested circuit breaker — AWS Fault Injection Simulator can deliberately simulate a region outage to prove the documented RTO/RPO is actually achievable, not just theoretically designed.

**13.** *(Day 2 + Day 6)* How does Day 2's B2B/Canonical Data Model math reappear on Day 6?
> The exact same N²-vs-N/hub-and-spoke math justifies VPC Transit Gateway over VPC Peering — same underlying principle, applied to network topology instead of data format transformation.

**14.** *(Day 1 + Day 5)* How does Day 1's Raft-based etcd consensus connect to Day 5's CAP theorem material?
> Consensus algorithms are the actual mechanism that makes strong consistency achievable during normal operation — CAP/PACELC describes the tradeoff at a theoretical level, Raft is one concrete, provably-correct way multiple real systems (etcd, Kafka's KRaft) actually implement the "consistency" side of that tradeoff.

**15.** *(Day 3 + Day 7)* Why would you recommend canary deployment specifically (not blue-green) for a Kong-fronted API change?
> Kong's traffic-splitting plugins make gradual percentage-based rollout natural to implement at the gateway layer, and canary's smaller blast radius fits a customer-facing API better than blue-green's all-at-once cutover, especially when the API gateway already has the mechanism built in.

**16.** *(Day 4 + Day 7)* How does the expand-and-contract migration pattern relate to Day 4's Saga compensating transactions?
> Both are built around the same core idea — every intermediate step must remain safe to reverse or coexist with the "before" state — expand-and-contract keeps a schema change backward-compatible at every step; a saga's compensating transaction keeps a business transaction reversible at every step, until the pivot transaction is reached.

---

## Week-in-review — every day, one line each

| Day | Core theme | The one thing worth having rock-solid |
|---|---|---|
| 1 | Linux, Containers, Kubernetes & OpenShift | Namespaces control what a process sees; cgroups control what it can use |
| 2 | Apache Camel & Enterprise Integration Patterns | Which EIP solves which specific problem — especially the Recipient List / Dynamic Router / Routing Slip trio |
| 3 | API Management & Gateway Architecture | API Gateway vs Service Mesh is about purpose (product vs infrastructure), not traffic direction |
| 4 | Messaging & Event-Driven Architecture | True exactly-once delivery doesn't exist — at-least-once plus idempotency is the real answer |
| 5 | Microservices & Distributed Systems Resilience | Sagas are ACD, not ACID — the missing Isolation has real, concrete consequences |
| 6 | AWS Core Architecture | State a real trade-off, not a definition — that's the entire senior/junior differentiator |
| 7 | DevOps/DevSecOps & Full Review | Pull-based GitOps is a security posture improvement, not just a workflow preference |

## Before the interview

Go back through each day's Q&A drill once more, cold, in the 24-48 hours before the actual interview — not to relearn anything, but to confirm the gap between what you can say instantly and what takes a beat to recall has closed. The material is genuinely comprehensive at this point; the remaining work is entirely repetition, not new learning.
