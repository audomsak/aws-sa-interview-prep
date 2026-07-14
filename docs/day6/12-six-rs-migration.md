# The 6 R's of migration

## The one-line hook

> **The 6 R's aren't a menu you pick one item from — they're a portfolio assessment framework, and most real migrations apply several of them simultaneously across different applications based on each one's actual business criticality and technical debt.**

## The six strategies

| Strategy | What it means | Effort/benefit |
|---|---|---|
| **Rehost** (lift and shift) | Move as-is, minimal or no changes | Fastest, lowest effort, captures no cloud-native benefit yet |
| **Replatform** (lift, tinker, and shift) | Move with some targeted optimization | Moderate effort, real managed-service benefits without full redesign |
| **Repurchase** (drop and shop) | Replace with a different product, often SaaS | A licensing/business decision, not really an architecture one |
| **Refactor / Re-architect** | Rebuild for cloud-native architecture | Highest effort and cost, highest potential benefit |
| **Retire** | Decommission — it's not needed anymore | Zero migration effort, pure cost/complexity reduction |
| **Retain** | Deliberately leave it where it is, for now | Zero migration effort, a legitimate choice, not a failure |

## Rehost — the default for scale and speed

**Lift and shift**: move servers and applications to AWS (typically EC2) with essentially no architectural changes, using tools like AWS Application Migration Service or VM Import/Export. This is the right call under time pressure (an expiring data center lease, an urgent need to exit on-prem), or for legacy applications with inaccessible source code or sparse documentation where modification isn't realistically an option. The honest limitation: it captures **none** of the cloud-native benefits yet, and can inherit whatever performance or architectural issues already existed.

## Replatform — the pragmatic middle ground, with a concrete example worth using directly

**Lift, tinker, and shift**: move to AWS while making **targeted** optimizations — the canonical, research-cited example worth having ready: migrating a **self-managed MySQL database running on EC2 to Amazon RDS for MySQL**. This picks up real managed-service benefits — automated backups, automatic patching, built-in high availability (a direct callback to this same day's RDS page) — without a full application redesign. A genuinely useful cited figure: replatforming **typically delivers 20-40% cost savings compared to a pure rehost**, for meaningfully less effort than a full refactor.

**Memorable hook:** *"Replatforming is the golden mean — more benefit than a straight lift-and-shift, less risk and cost than a ground-up rebuild."*

## Repurchase — a business decision more than a technical one

Moving to a different product entirely — commonly a SaaS replacement (moving a self-hosted CRM to Salesforce, for instance). The important distinction: this isn't fundamentally an architecture decision, it's a licensing and business one, even though it shows up on the same 6 R's list.

## Refactor/Re-architect — the highest-value, highest-risk option, with a specific sequencing insight

Rebuilding the core architecture to be genuinely cloud-native — a monolith becoming microservices, for instance. Driven by real business pressure: a system that genuinely can't scale further, can't ship features fast enough, or a mainframe application nobody left on the team actually knows how to maintain.

**A specific, nuanced, AWS-documented sequencing insight worth having exactly right**: for a **large-scale migration** (many applications), refactoring is explicitly **not recommended as the primary strategy** — it's too complex and expensive to manage across a large portfolio simultaneously. The recommended sequencing instead: **rehost or replatform first, get the application running in the cloud, and refactor afterward**, once the hard part (moving the application, its data, and its traffic) is already done, and the organization has had a chance to build real cloud skills in the meantime.

**Memorable hook:** *"Refactoring during the move is renovating a house while it's still on the truck. Get it to the new lot first — refactor once it's actually standing on solid ground."*

## Retire — the most overlooked, highest-ROI step

Decommissioning applications that are no longer actually needed. A genuinely useful, research-cited figure: as much as **10-20% of a typical enterprise IT portfolio** is no longer useful and can simply be turned off. This should be identified during the **initial assessment phase**, before any real migration effort is spent moving something nobody actually needs anymore.

## Retain — a legitimate choice, not a failure mode

Deliberately keeping something on-premises for now — because there's no clear business case yet, because compliance or latency requirements keep it local (directly echoing this day's earlier data-residency material), or because it was recently upgraded and isn't due for a refresh. Worth stating plainly: Retain is a **deliberate, defensible architectural decision**, not an admission that a migration effort failed to reach it.

## Decision criteria — a portfolio assessment, not a universal rule

Business criticality, current architecture and technical debt, timeline and budget, performance/scalability requirements, and compliance needs — each application in a real portfolio gets evaluated against these, and the resulting mix of strategies across a large migration is normal, expected, and correct, not a sign of an indecisive plan.

## Real-world examples

1. **The TnD Microservices decomposition, viewed through this exact lens, was effectively a Refactor/Re-architect strategy** — a monolithic Test & Diagnostics platform rebuilt as independently deployable microservices — a strong, concrete, personal example already covered extensively across the whole week, now correctly labeled with its formal migration-strategy name.
2. **The research's own replatforming example — self-managed MySQL on EC2 moving to Amazon RDS — connects directly to this same day's RDS page**, a clean, natural cross-page reinforcement within Day 6 itself.
3. **Recommending Retire as a genuine first step in a Thai enterprise customer's migration assessment**, citing the realistic 10-20% "dead weight" portfolio statistic — a cost-conscious, practically-grounded recommendation that shows real migration-program experience, not just familiarity with the six labels.
