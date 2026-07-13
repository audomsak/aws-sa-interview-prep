# EC2 & compute purchasing options

## The one-line hook

> **The purchasing option decision is never "which is cheapest" in the abstract — it's "how predictable is this workload's shape," and the four options map directly onto four different answers to that one question.**

## The four purchasing options, precisely

| Option | Commitment | Discount | Fits |
|---|---|---|---|
| **On-Demand** | None — pay per hour/second | Baseline (most expensive) | Unpredictable, short-term, or testing workloads |
| **Reserved Instances (RI)** | Specific instance type + region, 1 or 3 years | Up to ~72% | Steady-state workloads with a known, fixed instance type — can be zonal (guaranteed capacity reservation) or regional, and resold on the RI Marketplace if needs change |
| **Savings Plans** | A **dollar amount per hour** of compute usage — not a specific instance type | Comparable to RI discounts | Predictable *spend*, but flexibility across instance families, regions (Compute Savings Plans), and even across **services** (EC2, Lambda, Fargate) |
| **Spot Instances** | None — bid on spare capacity | Up to ~90% | Fault-tolerant, interruption-tolerant workloads — batch processing, CI/CD runners, stateless tiers behind Auto Scaling |

**The precise distinction between RI and Savings Plans worth having exactly right**: an RI commits to a *specific instance type in a specific region* — rigid, but with the option of a zonal capacity reservation guarantee. A Savings Plan commits to a *dollar amount* regardless of which instance type, region (for Compute Savings Plans), or even which compute service actually consumes it — meaningfully more flexible, at roughly comparable discount depth.

**The real cost of Spot**: instances can be reclaimed with only a **2-minute interruption warning** — genuinely fine for stateless, retryable, horizontally-scaled work, genuinely dangerous for anything stateful or synchronous that can't tolerate sudden termination.

**Memorable hook:** *"On-Demand is renting by the night. RI is a 1-3 year apartment lease for one specific unit. Savings Plans is committing to a monthly budget you can spend on any unit, any building. Spot is squatting in whatever's empty, knowing you might get a 2-minute notice to leave."*

## Stopping vs. terminating — a specific, frequently-asked distinction

| | Stopping | Terminating |
|---|---|---|
| **Instance state** | Shut down, can be restarted | Permanently deleted, cannot be restarted |
| **EBS root volume** | Persists (still billed for storage, not compute) | Deleted by default (unless explicitly configured to persist) |
| **Use case** | Pausing a workload temporarily without losing its disk state | Genuinely done with the instance |

## Horizontal vs. vertical scaling — the AWS-native default, and why

In most AWS architectures, **horizontal scaling is preferred** — for two concrete reasons: **fault tolerance** (losing one instance out of many doesn't cause an outage, directly echoing Day 5's bulkhead/redundancy thinking) and **cost flexibility** (capacity can be added or removed in small, granular increments rather than resizing one large instance). This is the same underlying instinct as Day 1's Kubernetes horizontal pod scaling, just expressed at the EC2 instance level instead of the container level.

## EBS vs. Instance Store — a quick, important distinction

- **EBS (Elastic Block Store)**: persistent, network-attached block storage that survives an instance stop (and can even be detached and reattached elsewhere) — the default choice for anything needing durability.
- **Instance Store**: ephemeral, physically attached storage, offering higher raw performance, but **data is lost the moment the instance stops or terminates** — appropriate only for genuinely ephemeral, cache-like, or reconstructable data.

## EC2 vs. Lambda vs. Fargate/ECS — the compute decision framework

| | Best fit |
|---|---|
| **EC2** | Full OS control, long-running or stateful workloads, legacy applications with complex runtime dependencies |
| **Lambda** | Event-driven, short-duration (under 15 minutes) tasks — microservices, lightweight APIs, automation |
| **Fargate** (serverless containers) | Containerized applications needing a consistent runtime without managing the underlying servers |

A specific, current, quotable piece of guidance worth having ready: **default to Fargate for new containerized workloads, and only reach for self-managed EC2-backed ECS/EKS when a specific cost or feature reason genuinely warrants it** — the operational burden of managing the underlying compute is real, and shouldn't be the default without a stated justification.

## Real-world examples

1. **A mixed purchasing strategy for the TnD Microservices platform**: Reserved Instances or Savings Plans for the steady-state Kafka broker fleet (predictable, always-on), paired with Spot Instances for stateless, horizontally-scaled batch processing workers — a defensible, concrete cost architecture grounded in your one actual AWS project.
2. **Kong Gateway's own deployment pattern**, directly relevant to your current role — a steady baseline of gateway data-plane traffic with room to flex is a textbook Savings Plans fit, rather than a rigid per-instance-type RI commitment.
3. **Correctly and precisely answering "what's the difference between stopping and terminating an instance"** — a specific, frequently-flagged question worth having exactly right rather than approximately right.
