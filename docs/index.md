# AWS Solutions Architect — Interview Crash Course

A 7-day, 10-hours-a-day technical drilling course built for one purpose: surviving deep, follow-up-after-follow-up technical questioning in an AWS Solutions Architect functional interview — the kind where a casual "yes I know Kubernetes" gets chased all the way down to raw Linux namespaces, a hand-built control plane, and how OpenShift deploys it all on-premises.

## How this course is built

Every day follows the same shape, aimed at making dense material stick in one week:

1. **Foundational explanation** — plain language first, jargon second, every acronym spelled out on first use.
2. **A mental model or analogy** — something you can picture under interview pressure, not just recite.
3. **2-3 real-world examples** — tied to your own project history (nbn Australia, Red Hat OpenShift PoCs, Kong customer accounts) wherever they genuinely fit, plus general industry examples elsewhere.
4. **Diagrams** — every major mechanism gets a picture, not just a paragraph.
5. **An interview Q&A drill** — 10+ hard questions per day, each answer built around a short memorable hook so you can reconstruct the full answer from one phrase under pressure.

## The 7-day roadmap

| Day | Focus | Why it's here |
|---|---|---|
| **[Day 0](day0/index.md)** | Interview craft — read first (~1 hour) | How to deliver answers in a follow-up-chain interview (layered answers, trade-off-first language, the honest-unknown protocol), plus Amazon Leadership Principles with a STAR story bank scaffolded on your real projects — the behavioral half of an Amazon loop that pure technical prep leaves uncovered |
| **[Day 1](day1/index.md)** | Linux, Containers, Kubernetes & OpenShift | You list containers/Kubernetes/OpenShift on your resume — expect drilling from raw Linux primitives up through a hand-built cluster and on-prem OpenShift deployment |
| **[Day 2](day2/index.md)** | Apache Camel & Enterprise Integration Patterns | Your 6+ years of Camel/JBoss Fuse work at Marlo (nbn Australia iB2B) is a deep-dive target — includes EIP catalog, transactions, exactly-once delivery, performance tuning, testing, cloud-native Camel, and B2B/legacy standards (SOAP, eBXML, EDI, DataPower) |
| **[Day 3](day3/index.md)** | API Management & Gateway Architecture | Kong is your current employer — includes API Gateway pattern, contract-first design, Kong architecture (CP/DP, hybrid mode, Konnect), plugins, rate limiting, OAuth2/OIDC, mTLS, API Gateway vs Service Mesh, AI Gateways, versioning, and Kong vs 3scale |
| **[Day 4](day4/index.md)** | Messaging & Event-Driven Architecture | Kafka and JMS run through your project history — includes Kafka internals, stream processing, geo-replication/DR, MQ vs Kafka, DLQ patterns, CDC, the Outbox pattern realized with CDC, the full Saga pattern, choreography vs orchestration, Event Sourcing vs CDC, and mapping it all onto AWS-native messaging (SQS, SNS, EventBridge, Kinesis, MSK) |
| **[Day 5](day5/index.md)** | Microservices & Distributed Systems Resilience | The "so how does this actually survive failure" day — CAP theorem, Raft/Paxos consensus, circuit breaker/bulkhead/retry/load shedding/fallback, chaos engineering, full CQRS and Event Sourcing treatment, distributed tracing/OpenTelemetry, and service mesh resilience patterns |
| **[Day 6](day6/index.md)** | AWS Core Architecture for Solutions Architects | The actual employer's own stack — VPC, the edge/traffic layer (ELB, Route 53, CloudFront, Global Accelerator), IAM, EC2 purchasing, containers (ECS/EKS/Fargate), Lambda, S3, RDS/Aurora, DynamoDB & SQL vs NoSQL, the Well-Architected Framework's 6 pillars, HA/multi-region/DR judgment, the 6 R's of migration, cost optimization, and the Shared Responsibility Model |
| **[Day 7](day7/index.md)** | DevOps/DevSecOps & Full Review | Repo/branching strategies, CI/CD pipeline design, GitOps push-vs-pull, Terraform vs Ansible, Ansible idempotency, Helm, observability, SLI/SLO/SRE, DevSecOps shift-left security, safe deployment patterns, four full system-design scenario walkthroughs, and a full 7-day cross-topic mock interview |

After Day 7, the **[hook cheat sheets](cheatsheets/index.md)** collect every memorable hook in the course (~370, one per drilled idea) into seven pages — the fastest possible full-course review for the day before the interview.

## Study rhythm for each day (10 hours)

- **Hours 1-7:** work through the topic pages in order — read, sketch the diagrams yourself from memory once you've seen them, don't skip the real-world examples. On days with a "build it by hand" page (Day 1's from-scratch container and hard-way Kubernetes build are the prime examples), actually slow down and re-derive each step rather than reading past it — that's where the depth comes from.
- **Hours 8-9:** close the laptop lid on the material and do that day's Q&A drill cold. Write your answer, *then* check it against the page. The gap between what you wrote and the model answer is exactly what to re-study.
- **Hour 10:** buffer — catch up on anything the Q&A drill exposed as shaky before moving to the next day. Skimming that day's [hook cheat sheet](cheatsheets/index.md) is the fastest way to find what's shaky: any hook that doesn't unfold back into its full answer marks the page to revisit.

## Status — COURSE COMPLETE

- ✅ Day 0 — Interview craft (complete: 2 pages — answer-delivery technique, and the 16 Leadership Principles with a 6-story STAR bank to personalize)
- ✅ Day 1 — Linux, Containers, Kubernetes & OpenShift (complete: 13 pages, 24-question interview drill)
- ✅ Day 2 — Apache Camel & Enterprise Integration Patterns (complete: 11 pages, 22-question interview drill)
- ✅ Day 3 — API Management & Gateway Architecture (complete: 12 pages, 24-question interview drill)
- ✅ Day 4 — Messaging & Event-Driven Architecture (complete: 12 pages, 28-question interview drill)
- ✅ Day 5 — Microservices & Distributed Systems Resilience (complete: 12 pages, 24-question interview drill)
- ✅ Day 6 — AWS Core Architecture for Solutions Architects (complete: 15 pages, 34-question interview drill)
- ✅ Day 7 — DevOps/DevSecOps & Full Review (complete: 12 pages, 4 full system-design scenario walkthroughs, 16-question cross-topic drill + 2 full scenario questions + week-in-review)
- ✅ Hook cheat sheets (complete: 7 generated sheets, ~370 hooks, for the final day-before-interview pass)

All 7 days complete, plus the Day 0 interview-craft primer and the hook cheat sheets. 89 content pages, 7 interview Q&A drills (~175 questions total), 6 full scenario walkthroughs, and a final cross-topic mock interview closing the course.
