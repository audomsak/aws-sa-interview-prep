# Observability stack architecture

Research flagged a specific, real trap here: "how would you design an observability stack for 40 services" is a common senior question, and **"I'd set up Prometheus for metrics, Grafana for..."** is explicitly the answer that loses the role — a tool list, not an architecture. This page is built to give you the answer that actually demonstrates real incident-ownership experience instead.

## The one-line hook

> **Naming Prometheus, Grafana, and a tracing tool isn't the answer — knowing exactly how they correlate with each other during a real incident, via a shared identifier, is.**

## The three pillars, and the different question each one answers

| Pillar | Answers | Character |
|---|---|---|
| **Metrics** (Prometheus) | "Is the system healthy, in aggregate, right now?" | Numerical, time-series, cheap to store at scale, low per-request resolution |
| **Logs** | "What exactly happened, in this specific instance?" | High detail, expensive to store/query at scale, need structure to be genuinely queryable |
| **Traces** (Day 5's OpenTelemetry material) | "Where did this specific request's time actually go, across every service it touched?" | Per-request causal chains, directly recalling Day 5's entire distributed tracing page |

**The sophisticated part of the answer**: knowing the actual **sequencing** during a real incident — start with **metrics/dashboards** to identify *which* service is unhealthy, move to **traces** to find *where* in that request's journey the problem actually lives, then drop into **logs** for the specific *why* at that exact point. This sequence, not just the three tool names, is what separates a real operational answer from a memorized list.

## Prometheus's pull-based architecture — a direct Day 5 callback

Prometheus **pulls** metrics — it scrapes configured targets on a regular interval, rather than services pushing metrics to it. This is a deliberate architectural choice, and it directly extends **Day 5's pull-vs-push backpressure material**: a pull-based collector is naturally more resilient — if Prometheus is briefly unavailable, it simply misses a scrape or two, rather than services queuing up a flood of pushed metrics that then overwhelms it the moment it comes back online.

Prometheus itself handles collection, time-series storage, and alerting rule evaluation. **Grafana** is a genuinely separate concern — visualization and dashboarding — commonly paired with Prometheus but architecturally distinct from it. **Alertmanager** is Prometheus's companion specifically for routing, deduplicating, and grouping alerts, rather than firing every raw threshold breach as its own separate, un-triaged alert.

## Alert fatigue — a concrete, practical mitigation checklist

- **Prioritize and reduce volume** — alert only on genuinely high-impact metrics, and aggregate repeated occurrences of the same issue rather than firing a fresh alert for each one.
- **Integrate context** — correlate alerts across systems, tying them to SLOs (full treatment on the next page) so an alert carries "why this matters," not just a raw number crossing a threshold.
- **Automate triage** — auto-remediate known low-severity issues automatically, and suppress alerts during planned maintenance windows so they don't compete for attention with genuine incidents.
- **Review and tune regularly** — post-incident reviews should specifically refine alert rules, with an explicit goal of keeping the false-positive rate low over time.

**Memorable hook:** *"An alert that fires on every minor blip and gets ignored is worse than no alert at all — it trains the team to stop trusting the alerting system exactly when a real incident needs them to trust it most."*

## Designing for 40 services — the complete answer

- **Centralized log aggregation** — not 40 separate sets of per-service log files nobody can search together.
- **Consistent metric naming and labeling conventions across every service** — without this, cross-service dashboards and correlation become genuinely impossible, since nothing lines up.
- **Distributed tracing with consistent context propagation** — directly recalling Day 5's specific warning about broken context propagation across asynchronous, message-queue boundaries, which becomes a much bigger risk at 40-services scale.
- **Correlating all three pillars via a shared identifier — the trace ID.** This is very likely the actual differentiator research is gesturing at: the ability to jump from a metric anomaly, to the specific traces occurring in that same time window, to the specific logs for that exact request — using one shared ID as the thread connecting all three, rather than three disconnected tools that each require a separate manual investigation.

## Real-world examples

1. **The exact research-flagged scenario, answered properly**: designing an observability stack for 40 services by leading with correlation — shared trace IDs, consistent labeling conventions across every service — rather than just naming Prometheus and Grafana. This directly addresses the specific trap research identified as costing candidates the role.
2. **Prometheus's pull-based architecture, directly connecting to Day 5's push-vs-pull and backpressure material** — a strong, specific, same-course cross-day technical connection rather than an isolated fact about one tool.
3. **Designing alert rules tied to actual SLOs** (a direct forward-reference to the next page) rather than arbitrary static thresholds, combined with the alert-fatigue-prevention checklist above — a practical, mature operational recommendation for a customer's production platform.
