# AWS Solutions Architect — Interview Crash Course

A 7-day, 10-hours-a-day, level-8-depth technical drilling course for the AWS Solutions Architect functional interview. Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and deployed automatically to GitHub Pages via GitHub Actions.

## Status — COURSE COMPLETE

- ✅ **Day 0 — Interview craft** (complete: 2 pages — answer-delivery technique for follow-up-chain interviews, and the 16 Amazon Leadership Principles with a 6-story STAR bank scaffolded on the candidate's real projects)
- ✅ **Day 1 — Linux, Containers, Kubernetes & OpenShift** (complete: 13 pages, diagrams, 24-question interview drill — covers Linux networking, building a container from scratch, cgroups/namespaces, Docker vs Podman, container networking, Kubernetes the hard way, Kubernetes architecture/CRI, CNI, CSI, OpenShift architecture, OpenShift on-prem deployment, and OpenShift SCC/Operators/Routes)
- ✅ **Day 2 — Apache Camel & Enterprise Integration Patterns** (complete: 11 pages, diagrams, 22-question interview drill — covers integration architecture fundamentals, Camel core architecture, Camel vs MuleSoft/Spring Integration/ESB, the full EIP catalog, error handling & transactions, exactly-once delivery, performance & scalability, testing, Camel K/Quarkus, and B2B/legacy standards)
- ✅ **Day 3 — API Management & Gateway Architecture** (complete: 12 pages, diagrams, 24-question interview drill — covers the API Gateway pattern, contract-first/OpenAPI design, Kong architecture, Kong plugins, rate limiting algorithms, OAuth2/OIDC, mTLS, API Gateway vs Service Mesh, AI Gateways, API versioning, and Kong vs 3scale)
- ✅ **Day 4 — Messaging & Event-Driven Architecture** (complete: 12 pages, diagrams, 28-question interview drill — covers Kafka internals, stream processing, geo-replication/DR, ActiveMQ/IBM MQ vs Kafka, DLQ/poison-pill patterns, CDC, the Outbox pattern realized with CDC, the full Saga pattern, choreography vs orchestration, Event Sourcing vs CDC, and AWS-native messaging: SQS, SNS, EventBridge, Kinesis, MSK)
- ✅ **Day 5 — Microservices & Distributed Systems Resilience** (complete: 12 pages, diagrams, 24-question interview drill — covers CAP theorem/PACELC, Raft/Paxos consensus, circuit breaker, bulkhead/timeout/retry, load shedding/backpressure, fallback/graceful degradation, chaos engineering, full CQRS treatment, full Event Sourcing treatment, distributed tracing/OpenTelemetry, and service mesh resilience patterns)
- ✅ **Day 6 — AWS Core Architecture for Solutions Architects** (complete: 15 pages, diagrams, 34-question interview drill — covers VPC networking, the edge/traffic layer (ELB, Route 53, CloudFront, Global Accelerator), IAM & identity, EC2 purchasing options, containers on AWS (ECS/EKS/Fargate), Lambda/serverless, S3, RDS/Aurora, DynamoDB & SQL vs NoSQL, the Well-Architected Framework's 6 pillars, HA/multi-region/DR judgment, the 6 R's of migration, cost optimization, and the Shared Responsibility Model)
- ✅ **Day 7 — DevOps/DevSecOps & Full Review** (complete: 12 pages, diagrams — covers repo/branching strategies (Git Flow, GitHub Flow, Trunk-Based, mono/polyrepo), CI/CD pipeline architecture, GitOps push-vs-pull, Terraform vs Ansible, Ansible idempotency, Helm, observability architecture, SLI/SLO/SRE, DevSecOps shift-left security, safe deployment patterns, four full system-design scenario walkthroughs, and a full cross-topic mock interview with 2 scenario questions + 16 cross-day drill questions + a week-in-review)
- ✅ **Hook cheat sheets** (complete: 7 generated sheets collecting ~370 hooks for the final day-before-interview pass)

**All 7 days complete**, plus a Day 0 interview-craft primer and hook cheat sheets. 89 content pages across the whole course, 7 interview Q&A drills (~175 questions), 6 full scenario walkthroughs, plus a closing cross-topic mock interview tying every day together.

## Local development

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open `http://127.0.0.1:8000`.

## Publishing to GitHub Pages

1. Push this repository to GitHub (e.g. `aws-sa-interview-prep`).
2. In the repo's **Settings → Pages**, set **Source** to **GitHub Actions**.
3. Push to `main` — the included workflow at `.github/workflows/deploy.yml` builds the site with `mkdocs build --strict` and deploys it automatically.
4. Your course will be live at `https://<your-username>.github.io/aws-sa-interview-prep/`.

No manual `gh-pages` branch management needed — the workflow handles build and deploy on every push to `main`.

## Project structure

```
.
├── mkdocs.yml                  # site config, nav, Material theme, Mermaid support
├── requirements.txt            # mkdocs-material
├── .github/workflows/deploy.yml # CI/CD to GitHub Pages
└── docs/
    ├── index.md                 # course home page / roadmap
    ├── day0/                    # Interview craft — answer technique & Leadership Principles (2 pages + index)
    ├── day1/                    # Linux, Containers, Kubernetes & OpenShift (13 pages + index)
    ├── day2/                    # Apache Camel & Enterprise Integration Patterns (11 pages + index)
    ├── day3/                    # API Management & Gateway Architecture (12 pages + index)
    ├── day4/                    # Messaging & Event-Driven Architecture (12 pages + index)
    ├── day5/                    # Microservices & Distributed Systems Resilience (12 pages + index)
    ├── day6/                    # AWS Core Architecture for Solutions Architects (15 pages + index)
    ├── day7/                    # DevOps/DevSecOps & Full Review (12 pages + index)
    └── cheatsheets/             # generated per-day hook sheets for final review (7 sheets + index)
```
