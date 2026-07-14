# Day 6 hooks — AWS Core Architecture for Solutions Architects

Every hook from [Day 6](../day6/index.md), in page order — 71 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [VPC networking deep dive](../day6/01-vpc-networking-deep-dive.md)

- **A subnet isn't inherently "public" or "private" — that's entirely determined by what its route table points to. Everything else in VPC networking builds on that one fact.**
- *"IGW is a two-way door. NAT Gateway is a one-way door that only opens outward — private instances can walk out, but nothing from outside can walk in through it."*
- *"A Security Group only needs to hear the door open once — it remembers, and lets the reply back in automatically. A NACL has amnesia after every single packet — it needs its own explicit rule for the reply, every time."*
- *"Peering is point-to-point integration for networks. Transit Gateway is the hub-and-spoke fix for the exact same N² problem — same math, completely different domain, and worth saying so explicitly."*

## [The edge & traffic layer — ELB, Route 53, CloudFront, Global Accelerator](../day6/02-edge-traffic-layer.md)

- **Three different jobs, three different layers: Route 53 decides *which endpoint* (DNS), CloudFront and Global Accelerator decide *how the packets travel* (the edge), and the load balancer decides *which target does the work*. Answer any traffic question by naming which of the three jobs is actually being asked about.**
- *"ALB reads the request; NLB just forwards the connection. The moment someone says 'route by URL path' you're at layer 7 — ALB. The moment they say 'static IP,' 'preserve source IP,' or 'not HTTP,' you're at layer 4 — NLB."*
- *"DNS is the slowest failover layer you own — the record changes instantly, but the world's resolvers only notice when their cached TTL expires."*
- *"CloudFront is a warehouse network — it stores copies close to customers. Global Accelerator is a private highway network — nothing is stored, the truck just skips the public roads. Cacheable HTTP → CloudFront; anything else that needs to be fast or fail over instantly → Global Accelerator."*

## [IAM & identity architecture](../day6/03-iam-identity-architecture.md)

- **Effective permission on AWS is never just "what does this one policy say" — it's the intersection of every guardrail layered on top of it: the policy itself, any permission boundary, and any Service Control Policy above that. Missing one layer of that stack is the single most common real IAM mistake.**
- *"IAM policies are the only layer that can say yes. Permission boundaries and SCPs can only ever say 'no, not even if something else says yes' — they're ceilings, not grants."*

## [EC2 & compute purchasing options](../day6/04-ec2-compute-purchasing-options.md)

- **The purchasing option decision is never "which is cheapest" in the abstract — it's "how predictable is this workload's shape," and the four options map directly onto four different answers to that one question.**
- *"On-Demand is renting by the night. RI is a 1-3 year apartment lease for one specific unit. Savings Plans is committing to a monthly budget you can spend on any unit, any building. Spot is squatting in whatever's empty, knowing you might get a 2-minute notice to leave."*

## [Containers on AWS — ECS vs EKS vs Fargate](../day6/05-containers-on-aws.md)

- **ECS vs EKS is "which orchestrator." EC2 vs Fargate is "who manages the compute underneath it." These are two separate, orthogonal decisions — and conflating them into one choice is a common, avoidable mistake.**

## [Lambda & serverless architecture](../day6/06-lambda-serverless-architecture.md)

- **Lambda isn't one invocation model — it's three genuinely different ones (synchronous, asynchronous, poll-based), and the retry/error-handling behavior is meaningfully different across all three. Treating Lambda as one uniform thing is where real production surprises come from.**
- *"Synchronous is a phone call. Asynchronous is a voicemail Lambda promises to call back about, retrying if the first callback fails. Poll-based is Lambda itself checking a mailbox on a schedule — the retry behavior belongs to the mailbox's own rules, not to Lambda uniformly."*

## [S3 deep dive](../day6/07-s3-deep-dive.md)

- **"S3 is cheap and durable" is the junior answer. "S3 Standard for the first 30 days, then lifecycle to Glacier Deep Archive, accepting a 12-hour retrieval SLA for the cost savings" is the senior one — the entire difference is a stated, specific trade-off.**
- *"Versioning protects against a mistake. MFA Delete protects against a mistake that also had valid credentials. Object Lock protects against someone with valid credentials and full admin rights, deliberately or under duress — a materially higher bar, and the one regulated compliance retention actually needs."*

## [RDS/Aurora & database architecture](../day6/08-rds-aurora-database-architecture.md)

- **Multi-AZ and Read Replicas solve two completely different problems — availability and read scaling — and combining them into one answer, or confusing which one does which, is one of the most common real mistakes in an RDS conversation.**
- *"Multi-AZ is a hot spare waiting to take over the instant the primary fails. A Read Replica is a worker helping carry read load — it can eventually become the primary if you promote it, but that's a decision you make, not something that happens automatically for you."*
- *"The application never needs to know a failover happened — it was always talking to a name, not an IP, and RDS just quietly repointed that name to the new primary."*

## [DynamoDB deep dive & the SQL vs NoSQL decision](../day6/09-dynamodb-deep-dive.md)

- **Relational databases let you design around your data and figure out the queries later. DynamoDB makes you design around your queries — know every access pattern first, then build the table to serve exactly those. Get that backwards and no amount of provisioned capacity will save you.**
- *"DynamoDB scales like a supermarket adding checkout lanes — infinitely, as long as customers spread out. A hot partition is everyone insisting on lane 3. Adaptive capacity sends staff to help lane 3; it still can't make one lane serve the whole store."*
- *"A GSI is a second table DynamoDB maintains for you with a different key — pay for it, get eventual consistency, query anything. An LSI is just a second sort order within the same partition — cheaper, stronger consistency, but you're locked in at creation."*
- *"Ask one question of the requirements: 'Do we know every query today?' Yes at scale → DynamoDB. No, or joins and ad-hoc reporting → relational. And never run analytics on DynamoDB — export to S3 and point Athena at it."*

## [The Well-Architected Framework, all 6 pillars](../day6/10-well-architected-framework.md)

- **The Well-Architected Framework isn't a checklist to maximize on every pillar simultaneously — the pillars actively conflict with each other, and the actual skill being tested is making a conscious, defensible trade-off between them, not scoring well on all six at once.**
- *"Reciting all six pillars is table stakes. Saying 'for this specific workload, I'm consciously trading some Cost Optimization for Reliability, and here's exactly why the business requirement justifies it' is the senior answer."*

## [High availability & multi-region/DR design](../day6/11-ha-multi-region-dr.md)

- **The judgment isn't "more redundancy is always better" — it's knowing that Multi-AZ already covers the vast majority of real availability requirements, and Multi-Region is usually driven by something other than pure availability entirely.**
- *"Multi-Region is very often not really a disaster-recovery decision at all — it's a data-residency or latency decision wearing DR clothing. Naming the real driver correctly changes the whole architecture conversation."*
- *"These four tiers are a dial, not an on/off switch — the right answer is whichever tier actually matches the customer's stated RTO/RPO, not automatically the most expensive one available."*

## [The 6 R's of migration](../day6/12-six-rs-migration.md)

- **The 6 R's aren't a menu you pick one item from — they're a portfolio assessment framework, and most real migrations apply several of them simultaneously across different applications based on each one's actual business criticality and technical debt.**
- *"Replatforming is the golden mean — more benefit than a straight lift-and-shift, less risk and cost than a ground-up rebuild."*
- *"Refactoring during the move is renovating a house while it's still on the truck. Get it to the new lot first — refactor once it's actually standing on solid ground."*

## [Cost optimization strategies](../day6/13-cost-optimization.md)

- **Cost optimization isn't a one-time architecture review — it's a continuous discipline, because right-sizing, purchasing models, and waste all drift the moment a workload's actual usage pattern changes.**
- *"Budgets tell you when you've crossed a line you already drew. Cost Anomaly Detection tells you something's behaving strangely even if you never thought to draw a line there in the first place."*

## [Shared Responsibility Model & security architecture](../day6/14-shared-responsibility-security.md)

- **"Managed" does not mean "secure by default" — and teams have genuinely shipped RDS instances sitting in public subnets with no encryption because someone assumed AWS managing the database engine meant AWS was also handling the security configuration around it.**
- *"Managed means AWS handles the parts of the stack it controls. It has never meant AWS makes your configuration choices secure for you — those choices are still entirely yours, on every managed service, every time."*
- *"Preventive controls are the locks on the doors. Detective controls are the security cameras — you need both, because locks alone don't tell you if one ever actually got picked."*

## [Day 6 interview Q&A drill](../day6/15-interview-qa.md)

- *"It's entirely determined by the route table, not an inherent property of the subnet."*
- *"A Security Group only needs to hear the door open once and remembers. A NACL has amnesia after every packet."*
- *"The same N² math from Day 2's integration fundamentals, now applied to network topology."*
- *"It's free, and it avoids NAT Gateway data processing charges entirely for that traffic."*
- *"SCPs can only restrict, never grant."*
- *"Roles issue temporary credentials that expire automatically — users carry long-lived credentials that can be leaked and stay valid indefinitely."*
- *"Once you have more than 3-4 AWS accounts."*
- *"RI commits to a specific instance type in a specific region. Savings Plans commits to a dollar amount, usable anywhere."*
- *"Stopping preserves the EBS root volume and lets you restart. Terminating deletes it, by default, permanently."*
- *"Fault tolerance and cost flexibility — losing one instance out of many doesn't cause an outage, and capacity can be added in small increments."*
- *"OpenShift is a Kubernetes distribution — EKS's control plane concepts are already familiar, not new material."*
- *"No — orchestrator (ECS vs EKS) and launch type (EC2 vs Fargate) are two separate, orthogonal decisions."*
- *"Roughly 50ms to 2 seconds, depending on runtime, memory, and VPC attachment."*
- *"Throttling actively rejects requests. Cold starts just make them slower — the fix for one does nothing for the other."*
- *"Synchronous is a phone call, asynchronous is a voicemail Lambda promises to call back about, poll-based is Lambda checking a mailbox on its own schedule."*
- *"S3 Standard for 30 days, then lifecycle to Glacier Deep Archive with Object Lock — accepting the 12-hour retrieval SLA."*
- *"Versioning protects against a mistake. MFA Delete protects against a mistake with valid credentials. Object Lock protects against valid admin credentials being misused."*
- *"Multi-AZ is a hot spare for availability. A Read Replica is a worker helping carry read load."*
- *"The application never needs to know a failover happened — it was always talking to a name, not an IP."*
- *"The pillars actively conflict with each other — the senior skill is stating which one you're consciously prioritizing, and why."*
- *"Multi-Region is usually driven by something other than pure availability — often data residency or latency, not disaster recovery."*
- *"RTO and RPO are business requirements that should determine the DR tier — not the other way around."*
- *"Refactor during the move is renovating a house while it's still on the truck — get it to the new lot first."*
- *"As much as 10-20% of a typical enterprise IT portfolio is no longer useful and can simply be turned off."*
- *"Managed means AWS handles the parts of the stack it controls — it has never meant your configuration choices are secure by default."*
- *"CloudTrail for the log, GuardDuty for the alert, Access Analyzer for the review, then rotate and investigate at scale with Athena."*
- *"Do we know every query today? Yes at scale → DynamoDB. No, or joins and reporting → relational."*
- *"Adaptive capacity sends staff to help lane 3 — it still can't make one checkout lane serve the whole store."*
- *"A GSI is a second table with a different key that DynamoDB maintains for you; an LSI is just a second sort order inside the same partition."*
- *"Global Tables let every region write and resolve conflicts last-writer-wins; Aurora Global Database has one writer region and fast promotion of the rest."*
- *"ALB reads the request; NLB just forwards the connection."*
- *"Weighted for canaries, latency for active-active, failover for DR, geolocation for compliance — and DNS is the slowest failover layer you own."*
- *"CloudFront is a warehouse network; Global Accelerator is a private highway — one stores copies, the other just skips the public roads."*
- *"The ALB balances and routes; the gateway governs."*
