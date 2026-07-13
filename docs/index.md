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
| **[Day 1](day1/index.md)** | Linux, Containers, Kubernetes & OpenShift | You list containers/Kubernetes/OpenShift on your resume — expect drilling from raw Linux primitives up through a hand-built cluster and on-prem OpenShift deployment |
| **[Day 2](day2/index.md)** | Apache Camel & Enterprise Integration Patterns | Your 6+ years of Camel/JBoss Fuse work at Marlo (nbn Australia iB2B) is a deep-dive target — includes EIP catalog, transactions, exactly-once delivery, performance tuning, testing, cloud-native Camel, and B2B/legacy standards (SOAP, eBXML, EDI, DataPower) |
| **[Day 3](day3/index.md)** | API Management & Gateway Architecture | Kong is your current employer — includes API Gateway pattern, contract-first design, Kong architecture (CP/DP, hybrid mode, Konnect), plugins, rate limiting, OAuth2/OIDC, mTLS, API Gateway vs Service Mesh, AI Gateways, versioning, and Kong vs 3scale |
| **[Day 4](day4/index.md)** | Messaging & Event-Driven Architecture | Kafka and JMS run through your project history — includes Kafka internals, stream processing, geo-replication/DR, MQ vs Kafka, DLQ patterns, CDC, the Outbox pattern realized with CDC, the full Saga pattern, choreography vs orchestration, and Event Sourcing vs CDC |
| **[Day 5](day5/index.md)** | Microservices & Distributed Systems Resilience | The "so how does this actually survive failure" day — CAP theorem, Raft/Paxos consensus, circuit breaker/bulkhead/retry/load shedding/fallback, chaos engineering, full CQRS and Event Sourcing treatment, distributed tracing/OpenTelemetry, and service mesh resilience patterns |
| **[Day 6](day6/index.md)** | AWS Core Architecture for Solutions Architects | The actual employer's own stack — VPC, IAM, EC2 purchasing, containers (ECS/EKS/Fargate), Lambda, S3, RDS/Aurora, the Well-Architected Framework's 6 pillars, HA/multi-region/DR judgment, the 6 R's of migration, cost optimization, and the Shared Responsibility Model |
| **[Day 7](day7/index.md)** | DevOps/DevSecOps & Full Review | Repo/branching strategies, CI/CD pipeline design, GitOps push-vs-pull, Terraform vs Ansible, Ansible idempotency, Helm, observability, SLI/SLO/SRE, DevSecOps shift-left security, safe deployment patterns, and a full 7-day cross-topic mock interview |

## Study rhythm for each day (10 hours)

- **Hours 1-7:** work through the topic pages in order — read, sketch the diagrams yourself from memory once you've seen them, don't skip the real-world examples. On days with a "build it by hand" page (Day 1's from-scratch container and hard-way Kubernetes build are the prime examples), actually slow down and re-derive each step rather than reading past it — that's where the depth comes from.
- **Hours 8-9:** close the laptop lid on the material and do that day's Q&A drill cold. Write your answer, *then* check it against the page. The gap between what you wrote and the model answer is exactly what to re-study.
- **Hour 10:** buffer — catch up on anything the Q&A drill exposed as shaky before moving to the next day.

## Status — COURSE COMPLETE

- ✅ Day 1 — Linux, Containers, Kubernetes & OpenShift (complete: 13 pages, 24-question interview drill)
- ✅ Day 2 — Apache Camel & Enterprise Integration Patterns (complete: 11 pages, 22-question interview drill)
- ✅ Day 3 — API Management & Gateway Architecture (complete: 12 pages, 24-question interview drill)
- ✅ Day 4 — Messaging & Event-Driven Architecture (complete: 11 pages, 24-question interview drill)
- ✅ Day 5 — Microservices & Distributed Systems Resilience (complete: 12 pages, 24-question interview drill)
- ✅ Day 6 — AWS Core Architecture for Solutions Architects (complete: 13 pages, 26-question interview drill)
- ✅ Day 7 — DevOps/DevSecOps & Full Review (complete: 11 pages, 16-question cross-topic drill + 2 full scenario questions + week-in-review)

All 7 days complete. 83 content pages, 7 interview Q&A drills (~160 questions total), plus a final cross-topic mock interview closing the course.
