# AWS Solutions Architect — Interview Crash Course

A 7-day, 10-hours-a-day, level-8-depth technical drilling course for the AWS Solutions Architect functional interview. Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and deployed automatically to GitHub Pages via GitHub Actions.

## Status — COURSE COMPLETE

- ✅ **Day 1 — Linux, Containers, Kubernetes & OpenShift** (complete: 13 pages, diagrams, 24-question interview drill — covers Linux networking, building a container from scratch, cgroups/namespaces, Docker vs Podman, container networking, Kubernetes the hard way, Kubernetes architecture/CRI, CNI, CSI, OpenShift architecture, OpenShift on-prem deployment, and OpenShift SCC/Operators/Routes)
- ✅ **Day 2 — Apache Camel & Enterprise Integration Patterns** (complete: 11 pages, diagrams, 22-question interview drill — covers integration architecture fundamentals, Camel core architecture, Camel vs MuleSoft/Spring Integration/ESB, the full EIP catalog, error handling & transactions, exactly-once delivery, performance & scalability, testing, Camel K/Quarkus, and B2B/legacy standards)
- ✅ **Day 3 — API Management & Gateway Architecture** (complete: 12 pages, diagrams, 24-question interview drill — covers the API Gateway pattern, contract-first/OpenAPI design, Kong architecture, Kong plugins, rate limiting algorithms, OAuth2/OIDC, mTLS, API Gateway vs Service Mesh, AI Gateways, API versioning, and Kong vs 3scale)
- ✅ **Day 4 — Messaging & Event-Driven Architecture** (complete: 11 pages, diagrams, 24-question interview drill — covers Kafka internals, stream processing, geo-replication/DR, ActiveMQ/IBM MQ vs Kafka, DLQ/poison-pill patterns, CDC, the Outbox pattern realized with CDC, the full Saga pattern, choreography vs orchestration, and Event Sourcing vs CDC)
- ✅ **Day 5 — Microservices & Distributed Systems Resilience** (complete: 12 pages, diagrams, 24-question interview drill — covers CAP theorem/PACELC, Raft/Paxos consensus, circuit breaker, bulkhead/timeout/retry, load shedding/backpressure, fallback/graceful degradation, chaos engineering, full CQRS treatment, full Event Sourcing treatment, distributed tracing/OpenTelemetry, and service mesh resilience patterns)
- ✅ **Day 6 — AWS Core Architecture for Solutions Architects** (complete: 13 pages, diagrams, 26-question interview drill — covers VPC networking, IAM & identity, EC2 purchasing options, containers on AWS (ECS/EKS/Fargate), Lambda/serverless, S3, RDS/Aurora, the Well-Architected Framework's 6 pillars, HA/multi-region/DR judgment, the 6 R's of migration, cost optimization, and the Shared Responsibility Model)
- ✅ **Day 7 — DevOps/DevSecOps & Full Review** (complete: 11 pages, diagrams — covers repo/branching strategies (Git Flow, GitHub Flow, Trunk-Based, mono/polyrepo), CI/CD pipeline architecture, GitOps push-vs-pull, Terraform vs Ansible, Ansible idempotency, Helm, observability architecture, SLI/SLO/SRE, DevSecOps shift-left security, safe deployment patterns, and a full cross-topic mock interview with 2 scenario questions + 16 cross-day drill questions + a week-in-review)

**All 7 days complete.** 83 content pages across the whole course, 7 interview Q&A drills (~160 questions), plus a closing cross-topic mock interview tying every day together.

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
    ├── day1/                    # Linux, Containers, Kubernetes & OpenShift
    │   ├── index.md
    │   ├── 01-linux-networking.md
    │   ├── 02-container-from-scratch.md
    │   ├── 03-cgroups-namespaces.md
    │   ├── 04-docker-vs-podman.md
    │   ├── 05-container-networking.md
    │   ├── 06-kubernetes-the-hard-way.md
    │   ├── 07-kubernetes-cri.md
    │   ├── 08-cni.md
    │   ├── 09-csi.md
    │   ├── 10-openshift-architecture.md
    │   ├── 11-openshift-onprem-deployment.md
    │   ├── 12-openshift-scc-operators-routes.md
    │   └── 13-interview-qa.md
    ├── day2/  ... day7/          # placeholders, to be filled in day by day (also being reshaped for 10-hour days)
```
