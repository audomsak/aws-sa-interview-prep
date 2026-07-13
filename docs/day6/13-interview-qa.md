# Day 6 interview Q&A drill

**How to use this page:** cover the answers, read only the question, answer out loud as if the interviewer is sitting across from you, *then* check. The goal isn't to reread — it's to catch the gap between what you can say cold and what you can only recognize when you see it. Given this is the actual employer's own stack, hold yourself to the senior-answer bar throughout: name specific services and state a real trade-off, not a generic definition.

---

**1. What determines whether a subnet is "public" or "private" in a VPC?**

> **Hook: "It's entirely determined by the route table, not an inherent property of the subnet."**
> A subnet is public if its route table sends internet-bound traffic (0.0.0.0/0) to an Internet Gateway, and private if it doesn't. There's no separate "make this subnet public" flag — it's purely a consequence of routing configuration.

---

**2. Explain the difference between Security Groups and NACLs precisely.**

> **Hook: "A Security Group only needs to hear the door open once and remembers. A NACL has amnesia after every packet."**
> Security Groups are stateful and operate at the instance level, with allow-only rules evaluated together — return traffic is automatically permitted regardless of outbound rules. NACLs are stateless and operate at the subnet level, supporting both allow and deny rules evaluated in rule-number order, and require an explicit rule to permit return traffic since nothing is remembered between packets.

---

**3. Why does VPC Peering not scale well for connecting many VPCs, and what solves it?**

> **Hook: "The same N² math from Day 2's integration fundamentals, now applied to network topology."**
> VPC Peering creates a direct 1:1 connection with no transitive routing — connecting N VPCs this way requires a direct peering connection between every pair, an N² problem. Transit Gateway solves it with a hub-and-spoke model supporting genuine transitive routing, the same underlying fix as a hub-and-spoke integration platform, just applied to network connections instead of B2B data formats.

---

**4. Why would you use a VPC Gateway Endpoint for S3 traffic instead of routing it through a NAT Gateway?**

> **Hook: "It's free, and it avoids NAT Gateway data processing charges entirely for that traffic."**
> A Gateway Endpoint (available for S3 and DynamoDB specifically) is implemented as a route table entry, costs nothing, and keeps that traffic off the public internet entirely — avoiding NAT Gateway's per-GB data processing charges for exactly the traffic that would otherwise flow through it.

---

**5. Can an IAM policy override a Service Control Policy that denies an action?**

> **Hook: "SCPs can only restrict, never grant."**
> No. An SCP applied at the account or OU level sets a maximum permission ceiling for everything inside it — no IAM policy inside that account, however permissive, can override an SCP-level deny. Effective permission is always the intersection of every applicable layer: the SCP, any permission boundary, and the IAM policy itself.

---

**6. Why is it recommended to never use IAM users for applications?**

> **Hook: "Roles issue temporary credentials that expire automatically — users carry long-lived credentials that can be leaked and stay valid indefinitely."**
> IAM roles provide temporary, session-scoped credentials assumed by a service or person, reducing the blast radius if credentials are exposed. IAM users carry long-lived access keys that, if leaked, remain valid until manually rotated or revoked — a meaningfully larger and longer-lived risk for anything automated.

---

**7. At what point would you recommend a customer adopt AWS Control Tower?**

> **Hook: "Once you have more than 3-4 AWS accounts."**
> Control Tower automates landing-zone setup — multi-account structure, baseline guardrails, centralized logging, identity federation, and account vending. Below roughly 3-4 accounts, manually maintaining consistent guardrails is manageable; beyond that, doing it by hand becomes its own operational risk, which is exactly the threshold Control Tower is built to address.

---

**8. What's the actual difference between a Reserved Instance and a Savings Plan?**

> **Hook: "RI commits to a specific instance type in a specific region. Savings Plans commits to a dollar amount, usable anywhere."**
> A Reserved Instance locks in a specific instance type and region for 1 or 3 years, with the option of a zonal capacity reservation guarantee, and can be resold on the RI Marketplace. A Savings Plan instead commits to a dollar amount of compute usage per hour, flexible across instance families, regions, and even services (EC2, Lambda, Fargate) — comparable discount depth, meaningfully more flexibility.

---

**9. What's the difference between stopping and terminating an EC2 instance?**

> **Hook: "Stopping preserves the EBS root volume and lets you restart. Terminating deletes it, by default, permanently."**
> Stopping shuts the instance down but preserves its EBS root volume (still billed for storage, not compute), and it can be restarted later. Terminating permanently deletes the instance and, by default, its EBS root volume, with no ability to restart it afterward.

---

**10. Why is horizontal scaling generally preferred over vertical scaling in AWS architectures?**

> **Hook: "Fault tolerance and cost flexibility — losing one instance out of many doesn't cause an outage, and capacity can be added in small increments."**
> Horizontal scaling (adding more instances) means the loss of any single instance doesn't take down the whole system, unlike vertical scaling's single larger instance being a bigger single point of failure. It also allows capacity changes in small, granular increments rather than resizing one large resource — the same underlying instinct as Day 1's Kubernetes horizontal pod scaling, applied at the EC2 level.

---

**11. What's the difference between ECS and EKS, and how does your OpenShift background apply to EKS specifically?**

> **Hook: "OpenShift is a Kubernetes distribution — EKS's control plane concepts are already familiar, not new material."**
> ECS is AWS's own, simpler, proprietary container orchestrator, tightly integrated with AWS services but not Kubernetes. EKS runs genuine upstream Kubernetes, with AWS managing the control plane (API server, etcd, scheduler, controller-manager). Since OpenShift is itself a Kubernetes distribution, the CRI/CNI/CSI/Operator model already covered this week applies directly to EKS — the main genuinely new detail is EKS's default AWS VPC CNI assigning pods real routable VPC IPs, rather than the overlay-network approaches (Calico, OVN-Kubernetes) covered on Day 1.

---

**12. Is "ECS vs EKS vs Fargate" really a single three-way choice?**

> **Hook: "No — orchestrator (ECS vs EKS) and launch type (EC2 vs Fargate) are two separate, orthogonal decisions."**
> ECS vs EKS decides which orchestrator manages your containers. EC2 vs Fargate decides who manages the underlying compute those containers run on — and this choice applies independently to either orchestrator. Treating it as one combined choice, rather than two separate ones, is a common oversimplification.

---

**13. What causes Lambda cold starts, and what are the concrete mitigations?**

> **Hook: "Roughly 50ms to 2 seconds, depending on runtime, memory, and VPC attachment."**
> The first invocation after a period of inactivity requires initializing a new execution environment. Mitigations include Provisioned Concurrency (pre-warmed environments), smaller deployment packages, faster runtimes (Go, Rust, Node over JVM-based languages), avoiding unnecessary VPC attachment, and — specifically for Java — Lambda SnapStart, which resumes from a snapshot of an already-initialized environment instead of fully re-initializing.

---

**14. How do you distinguish a genuine Lambda throttling problem from ordinary cold-start latency during an incident?**

> **Hook: "Throttling actively rejects requests. Cold starts just make them slower — the fix for one does nothing for the other."**
> Throttling occurs when account-level or reserved concurrency limits are exceeded, actively rejecting invocations. Cold-start latency means requests still succeed, just slower than expected. Diagnosing which one you're facing determines the fix: raising concurrency limits solves throttling; provisioned concurrency or faster initialization solves cold starts — applying the wrong fix does nothing for the actual problem.

---

**15. What's the difference between the three Lambda invocation models, and why does it matter for error handling?**

> **Hook: "Synchronous is a phone call, asynchronous is a voicemail Lambda promises to call back about, poll-based is Lambda checking a mailbox on its own schedule."**
> Synchronous invocations (API Gateway) have the caller wait for a direct response. Asynchronous invocations (S3 events, EventBridge) are internally queued by Lambda, which automatically retries on failure. Poll-based invocations (SQS, DynamoDB Streams) are pulled by Lambda's own internal polling service, with batching and retry behavior governed by the source's specific configuration rather than one uniform Lambda-wide policy.

---

**16. Design an S3 storage strategy for compliance documents needing 7-year retention, balancing cost against occasional retrieval needs.**

> **Hook: "S3 Standard for 30 days, then lifecycle to Glacier Deep Archive with Object Lock — accepting the 12-hour retrieval SLA."**
> Active documents stay in S3 Standard for their first 30 days of likely access, then a lifecycle policy automatically transitions them to Glacier Deep Archive — the cheapest tier — with S3 Object Lock enabled for genuine WORM compliance, explicitly accepting a 12+ hour retrieval SLA in exchange for the cost savings. This is the specific, complete, senior-grade answer combining storage classes, lifecycle policies, and compliance features into one coherent recommendation.

---

**17. What's the difference between S3 Versioning, MFA Delete, and Object Lock?**

> **Hook: "Versioning protects against a mistake. MFA Delete protects against a mistake with valid credentials. Object Lock protects against valid admin credentials being misused."**
> Versioning preserves every prior variant of an object against accidental overwrite or deletion. MFA Delete requires active multi-factor confirmation specifically to permanently remove a version. Object Lock provides true WORM compliance — an object under lock cannot be deleted or overwritten for its retention period, even by an account administrator, a materially higher bar needed for genuine regulated compliance retention.

---

**18. Explain the precise difference between RDS Multi-AZ and Read Replicas.**

> **Hook: "Multi-AZ is a hot spare for availability. A Read Replica is a worker helping carry read load."**
> Multi-AZ uses synchronous replication to a standby in another AZ, with automatic failover triggered by RDS itself on primary failure — traditionally not readable. Read Replicas use asynchronous replication specifically to scale read traffic, and can be manually promoted to a standalone primary if needed, which is a deliberate action, not an automatic failover. They solve different problems and are commonly combined, not chosen between.

---

**19. Walk through exactly what happens during an RDS Multi-AZ failover.**

> **Hook: "The application never needs to know a failover happened — it was always talking to a name, not an IP."**
> RDS detects the primary's failure via health checks, promotes the synchronous standby to primary, and updates the database's DNS CNAME endpoint to point at the newly-promoted instance. The application, already connecting via that same stable endpoint, reconnects automatically with no configuration change needed, typically completing in under a couple of minutes.

---

**20. Name all six Well-Architected Framework pillars, and explain why simply listing them isn't the senior answer.**

> **Hook: "The pillars actively conflict with each other — the senior skill is stating which one you're consciously prioritizing, and why."**
> Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, and Sustainability. Listing all six and connecting each to a service is table stakes. The senior answer explicitly acknowledges tension between pillars — for example, maximum Reliability costs real money, conflicting with Cost Optimization — and states a specific, business-justified trade-off for the actual workload in question, rather than claiming to maximize every pillar simultaneously.

---

**21. When is Multi-AZ sufficient, and when do you actually need Multi-Region?**

> **Hook: "Multi-Region is usually driven by something other than pure availability — often data residency or latency, not disaster recovery."**
> Multi-AZ, protecting against a single data center failure, is sufficient for the large majority of real availability requirements, since a full region failure is dramatically rarer and larger in blast radius. Multi-Region is usually justified by data residency/sovereignty regulations, global latency requirements, or — less often than assumed — true disaster recovery against a full regional outage. Recommending Multi-Region by default, without one of these specific drivers, is over-engineering relative to the actual requirement.

---

**22. Name the four DR strategy tiers and what determines which one a customer actually needs.**

> **Hook: "RTO and RPO are business requirements that should determine the DR tier — not the other way around."**
> Backup & Restore (cheapest, RTO in hours), Pilot Light (a minimal always-on core, RTO in tens of minutes), Warm Standby (a scaled-down but fully functional copy, RTO in minutes), and Multi-Site Active-Active (both regions fully active, near-zero RTO, highest cost). The correct tier is whichever one actually meets the customer's stated Recovery Time Objective and Recovery Point Objective — not automatically the most expensive option available.

---

**23. Explain the 6 R's of migration, and the specific sequencing guidance AWS gives for large-scale migrations.**

> **Hook: "Refactor during the move is renovating a house while it's still on the truck — get it to the new lot first."**
> Rehost (lift and shift), Replatform (lift, tinker, and shift), Repurchase (drop and shop for SaaS), Refactor/Re-architect (rebuild cloud-native), Retire (decommission), and Retain (deliberately keep in place). For large-scale migrations specifically, AWS recommends rehosting or replatforming first to get applications running in the cloud, then refactoring afterward — refactoring during the migration itself is explicitly discouraged as too complex to manage across a large portfolio simultaneously.

---

**24. What's a realistic, often-overlooked first step in a migration portfolio assessment?**

> **Hook: "As much as 10-20% of a typical enterprise IT portfolio is no longer useful and can simply be turned off."**
> Identifying candidates for Retire during the initial assessment phase, before any real migration effort is spent moving applications nobody actually needs anymore — a genuinely high-ROI, frequently-skipped step relative to jumping straight into rehost/replatform/refactor planning.

---

**25. A customer assumes their RDS instance is secure because RDS is a "managed service." What's wrong with that assumption?**

> **Hook: "Managed means AWS handles the parts of the stack it controls — it has never meant your configuration choices are secure by default."**
> AWS manages RDS's database engine patching and underlying hardware, but security group rules, IAM permissions, subnet placement (public vs. private), and whether encryption at rest is enabled all remain entirely the customer's responsibility. This exact assumption has led real teams to ship RDS instances in public subnets with no encryption enabled.

---

**26. Walk through a concrete incident response workflow for a suspected compromised IAM credential.**

> **Hook: "CloudTrail for the log, GuardDuty for the alert, Access Analyzer for the review, then rotate and investigate at scale with Athena."**
> CloudTrail provides the API activity log to investigate; a CloudWatch alarm, often fed by a GuardDuty finding, surfaces the suspicious activity; IAM Access Analyzer reviews external or cross-account access for anything unexpectedly broad. On confirming a real incident, rotate the affected credentials, revoke active sessions, and use Athena to run SQL queries directly against CloudTrail logs at scale for a full investigation — a specific, operational response, not a vague "we'd investigate and respond."
