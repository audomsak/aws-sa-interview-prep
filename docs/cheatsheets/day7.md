# Day 7 hooks — DevOps/DevSecOps & Full Review

Every hook from [Day 7](../day7/index.md), in page order — 23 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [Repository & branching workflow strategies](../day7/01-repo-branching-workflow-strategies.md)

- **Every branching strategy is really a different answer to one question: how long is code allowed to exist without being integrated into what everyone else is working from?** Git Flow says weeks. GitHub Flow says days. Trunk-Based Development says hours, or never at all.
- *"Git Flow manages risk by keeping unfinished work isolated in branches. Trunk-Based Development manages the exact same risk by keeping unfinished work isolated behind a flag instead — same goal, opposite mechanism."*

## [CI/CD pipeline architecture & artifact promotion](../day7/02-cicd-pipeline-architecture.md)

- **Build once, promote the same artifact through every environment. Rebuilding for staging and rebuilding again for production means you've lost reproducibility — "the rule nobody breaks," and one of the most common real sources of "works in staging, fails in prod" incidents when it is broken.**
- *"If you rebuild for production, you're not promoting what you tested — you're testing one thing and shipping something merely similar to it."*

## [GitOps deep dive — push vs pull](../day7/03-gitops-push-vs-pull.md)

- **Push-based CI/CD means your pipeline holds cluster credentials. Pull-based GitOps means it never has to — and that single difference is a genuine security posture improvement, not just a different workflow.**
- *"In push-based CI/CD, your build pipeline is one compromise away from being your cluster's biggest attack surface. In pull-based GitOps, it isn't — because it was never holding the keys to begin with."*
- *"GitOps isn't a new idea layered on top of Kubernetes — it's Kubernetes' own reconciliation philosophy, recognizing that the same trick works for deploying the applications, not just running them."*

## [Infrastructure as Code — Terraform vs Ansible](../day7/04-iac-terraform-vs-ansible.md)

- **"Terraform for Day 0, Ansible for Day 1 and Day 2" — provisioning the infrastructure is a fundamentally different job from configuring what runs on it, and confusing which tool does which is a specifically-flagged interview red flag.**
- *"Locking here is the exact same problem Day 5's consensus material solved in the abstract — ensuring only one writer can act on shared state at once — just applied to Terraform's own state file specifically."*

## [Ansible idempotency & dynamic inventory](../day7/05-ansible-idempotency-inventory.md)

- **Running the same playbook twice should never double-apply anything — it should only touch what's actually drifted. A script that appends a line to a file every time it runs is the exact opposite of this, and it's the cautionary example worth having in mind.**
- *"A proper Ansible module asks 'is this already true?' before doing anything. `shell` and `command` don't ask — they just do, every time, whether it needed doing or not."*

## [Helm — when it helps, when it hurts](../day7/06-helm-when-it-helps-hurts.md)

- **The question isn't "what is Helm" — it's "when does Helm make things worse." "I always use Helm" is explicitly not a senior answer.**
- *"Helm is genuinely excellent at packaging software for other people to configure and install. It's genuinely mediocre at making your own simple internal app easier to understand — and conflating those two use cases is exactly what 'I always use Helm' gets wrong."*

## [Observability stack architecture](../day7/07-observability-stack-architecture.md)

- **Naming Prometheus, Grafana, and a tracing tool isn't the answer — knowing exactly how they correlate with each other during a real incident, via a shared identifier, is.**
- *"An alert that fires on every minor blip and gets ignored is worse than no alert at all — it trains the team to stop trusting the alerting system exactly when a real incident needs them to trust it most."*

## [SLI/SLO & SRE fundamentals](../day7/08-sli-slo-sre-fundamentals.md)

- **An error budget is the mechanic that turns "ship faster vs. be more careful" from an endless political argument into a data-driven, automatic decision — and it's the single idea that ties this entire day's velocity themes (CI/CD, GitOps) and Day 5's reliability themes together into one coherent system.**
- *"The SLO is the bar you actually try to clear. The SLA is the bar you promised a customer, set deliberately lower, so a stumble against the SLO doesn't automatically mean breaking a contract."*
- *"An error budget turns 'should we ship this risky change' from a subjective argument into an objective, pre-agreed answer — you have budget, or you don't, and everyone already agreed on the rule before the argument ever needed to happen."*

## [DevSecOps — shift-left security pipeline](../day7/09-devsecops-shift-left-security.md)

- **Shift-left means security becomes everyone's job, embedded in the pipeline itself — not a gate at the end run by a separate team, and the specific stage placement (what runs early and fast vs. late and thorough) is itself the actual architecture decision being tested.**

## [Safe deployment patterns](../day7/10-safe-deployment-patterns.md)

- **Blue-green, canary, and rolling deployment are really the same question asked three different ways: how much of the blast radius do I expose at once, and how fast can I undo it if something's wrong — the exact same spectrum Day 5 covered for resilience patterns generally, now applied specifically to the moment of deployment itself.**
- *"Blue-green trades infrastructure cost for instant rollback. Canary trades rollout speed for a smaller blast radius. Rolling trades rollback speed for needing no extra infrastructure at all — there's no free option, just three different bills to pay."*

## [System-design scenario practice — four full walkthroughs](../day7/11-system-design-scenarios.md)

- **Every scenario prompt is secretly one of a handful of shapes. Recognize the shape, and you're not designing from scratch under pressure — you're adapting a design you've already rehearsed, out loud, four times.**

## [Full 7-day cross-topic mock interview & review](../day7/12-full-mock-interview-review.md)

- *"An interviewer scoring a scenario question is watching your process at least as much as your final diagram — clarifying requirements first is itself a signal, not a delay."*
