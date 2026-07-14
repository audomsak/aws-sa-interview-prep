# System-design scenario practice — four full walkthroughs

The mock interview page closing this course contains two scenario questions built around your own project history. This page adds four more, chosen because they're the four most statistically likely *shapes* of scenario prompt in an AWS SA interview: the classic 3-tier web application, the multi-region DR judgment call, the event-driven distributed-transaction design, and the cost-optimization review. Same rules as the mock: use the five-step framework from the [mock interview page](12-full-mock-interview-review.md) (clarify → constraints → high-level → deep-dive → failure modes), talk out loud, sketch, and don't open a model answer until you've genuinely attempted the scenario.

## The one-line hook

> **Every scenario prompt is secretly one of a handful of shapes. Recognize the shape, and you're not designing from scratch under pressure — you're adapting a design you've already rehearsed, out loud, four times.**

---

## Scenario 1 — the classic: a highly available 3-tier web application

> **"A retail company wants to move its customer-facing web application to AWS. It must survive the loss of a data center, handle a 10x traffic spike during promotions, and keep the database from being reachable from the internet. Walk me through your design."**

This is the single most common SA screening scenario — it's asked precisely because every layer has a wrong-but-plausible option, and your Day 6 material covers every right one.

<details markdown>
<summary>Model answer outline — expand only after attempting your own</summary>

- **Clarify first**: traffic profile and baseline, session state (sticky or stateless?), read/write ratio, RTO/RPO expectations, budget posture. The 10x spike and "survive a data center loss" are the two anchors — Multi-AZ, and elasticity.
- **Network layout (Day 6 VPC)**: one VPC, subnets across **at least two AZs**; public subnets holding only the load balancer and NAT Gateways (one per AZ); application tier and database in private subnets. "A subnet is only public because of its route table" is worth saying out loud.
- **Traffic layer (Day 6 edge page)**: Route 53 → CloudFront for static assets and edge TLS → **ALB** across AZs (layer-7, path routing, WAF attached). Name the arrows: HTTPS end-to-end.
- **Application tier**: Auto Scaling Group of EC2 (or ECS/Fargate — say why: team's container maturity) spanning AZs, scaling policy on a real signal (ALB request count per target beats CPU for web tiers). The 10x spike is why the tier must be **stateless** — sessions to ElastiCache or DynamoDB (with TTL), never instance memory.
- **Database tier**: RDS/Aurora **Multi-AZ** (availability — synchronous standby, automatic failover) plus **read replicas** if the read ratio justifies them (scaling) — Day 6's "two different problems, two different tools" distinction, stated explicitly. Private subnets, security group allowing only the app tier's security group — reference, don't list IPs.
- **Failure modes**: lose an AZ → ALB health checks stop routing there, ASG rebalances, RDS fails over (~1-2 min, DNS CNAME repoint — the app reconnects to the same endpoint). What breaks first under the spike? Likely database connections — mention RDS Proxy or connection pooling before the interviewer does.
- **Trade-off to volunteer**: this design is deliberately single-region — Multi-AZ covers the stated requirement, and multi-region would double cost and complexity for a requirement nobody stated. Saying *what you chose not to build* is senior signal.

</details>

---

## Scenario 2 — the judgment call: multi-region DR for a payments platform

> **"A payments company running in ap-southeast-1 asks for 'zero downtime, even if the whole region fails.' Regulators require their data to stay recoverable to within seconds of the failure. Design their DR posture — and challenge the requirement if you think you should."**

The prompt is a trap by design: it invites you to build the most expensive thing. The senior move is exactly what it offers — challenge the requirement, with numbers.

<details markdown>
<summary>Model answer outline — expand only after attempting your own</summary>

- **Clarify by translating slogans into numbers**: "zero downtime" → what RTO is actually tolerable, and for which flows? Payment *authorization* might genuinely need minutes; back-office reporting does not. "Within seconds of the failure" → RPO in seconds, which constrains the data layer far more than the compute layer.
- **Name the four DR postures** (Day 6): backup & restore → pilot light → warm standby → multi-site active-active, each a cost/RTO trade. Split the platform: **active-active only for the authorization path**, warm standby for everything else — per-workload DR posture, not one blanket answer, is the core of the model answer.
- **Data layer, the hard part**: RPO-in-seconds cross-region means **Aurora Global Database** (sub-second replication lag, single writer, minutes to promote) — and an honest sentence on what asynchronous replication means: an RPO of seconds is not an RPO of zero, and a synchronous cross-region write would buy zero RPO at the price of adding regional round-trip latency to every payment. If parts of the model are key-value with tolerable last-writer-wins, DynamoDB Global Tables give true active-active writes (Day 6's comparison).
- **Traffic and failover (edge page)**: Route 53 failover or latency policies with health checks — plus the TTL honesty ("DNS failover takes minutes to fully propagate") — or **Global Accelerator** for near-instant failover without the DNS wait, which the payments latency profile probably justifies.
- **Failure modes**: the failover *itself* is the risky machine — is region-B capacity actually running (warm) or theoretical (pilot light)? How is failover *tested*? (Day 5's chaos engineering: regular game days, or the DR plan is fiction.) Split-brain risk on failback.
- **Trade-off to volunteer**: state the rough cost shape out loud — active-active roughly doubles run cost; that's why it's reserved for the authorization path only. Challenging "zero downtime, everywhere" into "seconds of RPO and minutes of RTO where it counts, warm elsewhere" *is* the demonstration of judgment the prompt is fishing for.

</details>

---

## Scenario 3 — home turf: an event-driven order system with no lost orders

> **"Design an e-commerce order-processing system on AWS: an order placement must atomically update the order database and notify inventory, payment, and shipping services — and a failure in any downstream service must never lose or duplicate an order. The team is small and has no Kafka operational experience."**

This is Days 2, 4, and 5 wearing an AWS costume — and the final constraint is doing real work: it steers the messaging choice.

<details markdown>
<summary>Model answer outline — expand only after attempting your own</summary>

- **Clarify**: order volume (does this even need a streaming backbone?), ordering requirements (per-order suffices — global ordering is a myth nobody needs), what "notify" means (fire-and-forget events vs a coordinated workflow with rollback).
- **The atomic write is the outbox pattern (Day 4)**: order row and event row in one local transaction — never dual-write to a database and a message bus. On AWS without Kafka: DynamoDB as the order store makes **DynamoDB Streams** the outbox relay for free (the change event is emitted from the same write); on RDS, a transactional outbox table with a poller, honestly labeled as the less elegant version.
- **Fan-out (Day 4's AWS messaging page)**: **SNS → one SQS queue per consumer** (inventory, payment, shipping) — Kafka's consumer-group pattern from serverless parts, each service with its own durable buffer, subscription filters if consumers care about different event types. The "no Kafka experience" constraint is the explicitly-stated reason MSK/Kinesis lose here — say the semantics ("fan-out to independent durable consumers"), then the service.
- **No duplicates**: at-least-once is a fact of life end-to-end, so **idempotent consumers** keyed on order ID — a conditional write against a DynamoDB idempotency table (with TTL) per consumer. "Exactly-once is an end-to-end property you build, not a checkbox you enable" is the Day 4 sentence to say verbatim.
- **The coordinated-workflow half (Day 4 saga)**: if payment failure must release reserved inventory, that's a saga — and with a small team, **orchestration** (Step Functions as the saga orchestrator) over choreography, for debuggability and visible state; compensating actions defined per step; the pivot transaction placed as late as possible.
- **Failure modes**: poison messages → per-queue DLQs with redrive policies and an alarm on DLQ depth (Day 4's DLQ page, as SQS checkboxes); a slow consumer backs up only its own queue (bulkhead by construction — Day 5); the outbox relay lagging → orders accepted but events delayed: eventual consistency, monitored via queue age metrics.

</details>

---

## Scenario 4 — the review: cut this bill in half without breaking it

> **"A customer's monthly AWS bill has tripled in a year. Their setup: a fleet of always-on m5.4xlarge EC2 instances at ~15% average CPU, everything on gp2 EBS and S3 Standard forever, a NAT Gateway funneling heavy S3 traffic, all inter-service traffic crossing AZs, and RDS instances for dev/test running 24/7. Walk me through your cost review — what do you cut first, and what do you refuse to cut?"**

Cost scenarios reward a *method*, not a list of discounts. The prompt hides six specific findings — a strong answer finds them and sequences them by effort-to-savings ratio.

<details markdown>
<summary>Model answer outline — expand only after attempting your own</summary>

- **Method first (Day 6 cost page)**: visibility before action — Cost Explorer, tagging, and the question "what changed a year ago?" Then sequence: waste first (free to cut), then right-sizing, then commitment discounts *last* — never buy a Savings Plan sized to today's waste.
- **The six findings, in attack order**: (1) **dev/test RDS running 24/7** — schedule off-hours stop, zero risk; (2) **15% CPU on m5.4xlarge** — right-size down and/or Graviton, guided by Compute Optimizer data, not vibes; (3) **S3-through-NAT-Gateway** — a **Gateway VPC Endpoint** is free and eliminates NAT data-processing charges for that traffic entirely (the Day 6 VPC page's exact example); (4) **gp2 → gp3** — same performance floor, lower price, trivial migration; (5) **S3 Standard forever** — lifecycle policies to Infrequent Access/Glacier tiers by access pattern (or Intelligent-Tiering when the pattern is unknown), with the retrieval-SLA trade-off stated; (6) **cross-AZ chatter** — AZ-aware routing/topology where the architecture allows, with the honest caveat that this one is engineering work, not a checkbox.
- **Then, and only then, commitments**: with the fleet right-sized and stable, cover the steady-state floor with Savings Plans/RIs, leave the spiky remainder on-demand or Spot (stateless, interruption-tolerant tiers only — Day 6's purchasing-options judgment).
- **What you refuse to cut**: Multi-AZ on production databases, backups, and observability — cost optimization that erodes availability posture is a false economy, and saying so unprompted is the Well-Architected trade-off language (cost pillar vs reliability pillar, in tension, resolved explicitly) the whole day rewards.
- **Close with governance, not heroics**: budgets and alerts, tagging enforcement, a recurring right-sizing review — otherwise the bill regrows; the one-off cleanup is the *least* valuable half of the answer.

</details>

---

## After the four scenarios

Two habits to carry from this page into the real interview: first, every model answer above spends its first breath on **clarifying or challenging the requirements** — that's not preamble, it's the scored behavior. Second, notice how little *new* material this page contains: every component in every answer is a hook from an earlier day. That's the point — scenario fluency is retrieval plus sequencing, and the [cheat sheets](../cheatsheets/index.md) exist to keep the retrieval half warm.
