# High availability & multi-region/DR design

Research converged on this exact question as the real judgment test of the whole day: **when is Multi-AZ enough, and when do you actually need Multi-Region?** Getting this precisely right — including knowing when to say "you don't need Multi-Region" — is worth more than reflexively recommending maximum redundancy everywhere.

## The one-line hook

> **The judgment isn't "more redundancy is always better" — it's knowing that Multi-AZ already covers the vast majority of real availability requirements, and Multi-Region is usually driven by something other than pure availability entirely.**

## Multi-AZ — sufficient for most real requirements

Availability Zones within a region are physically separate data centers, connected by low-latency links, engineered so a failure in one doesn't take down the others. Most AWS managed services make Multi-AZ protection relatively easy and inexpensive to achieve: RDS Multi-AZ, an ALB spreading traffic across AZs, an Auto Scaling Group spanning multiple AZs. A **single AZ failure** is a real, planned-for event. A **full region failure** is dramatically rarer and larger in blast radius — which is exactly why Multi-AZ, not Multi-Region, is the correct default for most workloads.

## Multi-Region — usually driven by something other than availability alone

The honest reasons to go Multi-Region, in rough order of how often they actually apply:

- **Data residency/sovereignty regulations** — a direct callback to Day 4's geo-replication material: certain data may be legally required to stay within a specific country's borders, a genuinely live constraint in the Thai financial and government context.
- **Global latency** — serving users on different continents with acceptably low latency, when a single region's physical distance to some users is itself the problem.
- **True disaster recovery against a full region outage** — the rarest, largest-blast-radius scenario, and the one most often over-weighted relative to its actual likelihood.

**Memorable hook:** *"Multi-Region is very often not really a disaster-recovery decision at all — it's a data-residency or latency decision wearing DR clothing. Naming the real driver correctly changes the whole architecture conversation."*

## The real cost of Multi-Region — why it's not a free upgrade

Multi-Region is dramatically more expensive and operationally complex than Multi-AZ: cross-region data replication has real latency, forcing exactly the eventual-consistency trade-off from Day 5's CAP theorem and PACELC material — you cannot get instant, strongly-consistent cross-region replication without paying a real latency cost. DNS-based failover (Route 53) adds real operational complexity, and — a genuine, often-skipped discipline — **testing region failover regularly** is something few teams actually do well, meaning an untested Multi-Region DR plan may not actually work when needed, echoing Day 5's Chaos Engineering point about untested resilience code.

## The building blocks, when Multi-Region genuinely is justified

- **Route 53** health checks and failover routing policies (failover, latency-based, geolocation) — the DNS layer that actually redirects traffic during a regional event.
- **Aurora Global Database** (from the previous page) — sub-second cross-region replication with fast regional promotion.
- **DynamoDB Global Tables** — multi-region, multi-master replication with automatic (last-writer-wins) conflict resolution.
- **S3 Cross-Region Replication** (also from the previous page).
- **CloudFront** — global edge distribution for content, reducing the latency problem at the delivery layer without needing full application-tier multi-region replication.

## RTO/RPO — the business-driven inputs that should actually decide this

**Recovery Time Objective (RTO)**: how long can the system acceptably be down? **Recovery Point Objective (RPO)**: how much data can acceptably be lost? These are **business requirements**, not technology choices — the DR architecture should be selected to meet a stated RTO/RPO, not chosen first and then whatever RTO/RPO happens to fall out of it. This directly echoes Day 4's honest framing of MirrorMaker's RPO being bounded by real replication lag, not an assumed zero — the same discipline applies here at the whole-region level.

## The four DR strategy tiers — a spectrum, not a binary choice

| Tier | RTO | Cost | Description |
|---|---|---|---|
| **Backup & Restore** | Hours | Cheapest | Data backed up, infrastructure stood up from scratch only when needed |
| **Pilot Light** | Tens of minutes | Low | A minimal, always-on core (often just the database) in the DR region, scaled up on failover |
| **Warm Standby** | Minutes | Moderate | A scaled-down but fully functional copy of the whole stack running in the DR region continuously |
| **Multi-Site Active-Active** | Near-zero | Highest | Both regions fully active and serving traffic simultaneously |

**Memorable hook:** *"These four tiers are a dial, not an on/off switch — the right answer is whichever tier actually matches the customer's stated RTO/RPO, not automatically the most expensive one available."*

## Real-world examples

1. **The exact judgment question from research, answered directly**: recommending Multi-AZ as sufficient for the large majority of a Thai enterprise customer's workloads, reserving true Multi-Region specifically for data-residency-driven requirements or a small number of genuinely mission-critical systems — a sophisticated answer that deliberately avoids the trap of reflexively recommending maximum redundancy everywhere.
2. **Thai financial or government data residency requirements as a non-availability-driven reason for multi-region architecture**, directly recalling Day 4's geo-replication material — reinforcing that Multi-Region decisions often aren't really about DR at all.
3. **Choosing between Pilot Light and Warm Standby based on a customer's actual stated RTO/RPO**, rather than defaulting to the most expensive tier — a concrete, business-requirement-driven recommendation process that demonstrates real architectural judgment.
