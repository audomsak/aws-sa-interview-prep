# Cost optimization strategies

This page is deliberately the one that pulls together threads from almost every earlier page today — cost optimization on AWS isn't one technique, it's the same underlying discipline applied at every layer already covered.

## The one-line hook

> **Cost optimization isn't a one-time architecture review — it's a continuous discipline, because right-sizing, purchasing models, and waste all drift the moment a workload's actual usage pattern changes.**

## Right-sizing — continuous, not one-time

**AWS Compute Optimizer** analyzes real utilization metrics and recommends downsizing over-provisioned EC2 instances or RDS databases. The key discipline worth stating explicitly: this needs to run **continuously**, not as a one-off exercise at launch — a workload's actual resource needs drift over time, and yesterday's right-sized instance is tomorrow's over-provisioned one.

## Pulling together what's already been covered today

| Layer | Cost lever | Where it was covered |
|---|---|---|
| **Compute purchasing** | Match Reserved Instances, Savings Plans, or Spot to actual workload predictability | Page 3 |
| **Storage** | Lifecycle policies and Intelligent-Tiering, moving data to cheaper tiers automatically as access patterns change | Page 6 |
| **Network** | VPC Gateway Endpoints avoiding NAT Gateway fees for S3/DynamoDB traffic; Transit Gateway consolidating VPC-to-VPC connections into a single charge rather than many separate peering connections | Page 1 |
| **Database** | Aurora Serverless v2 scaling near-zero for intermittent workloads; DynamoDB on-demand mode for unpredictable traffic | Page 7 |
| **Compute model** | Serverless (Lambda/Fargate) paying only for actual usage, versus an always-on EC2 fleet sized for peak load | Pages 3-5 |

**This table is itself the point worth making in an interview**: cost optimization isn't a separate topic bolted onto architecture — it's a lens applied to every decision already made elsewhere in the design.

## CloudFront's specific, concrete cost mechanics

Worth having exactly right, since it's a specific, quantifiable detail rather than a vague "CDNs are cheaper" gesture: **CloudFront-to-origin data transfer is free**, and **CloudFront-to-internet transfer is cheaper than serving the same data directly from the origin to the internet**. This makes CloudFront a genuine cost optimization, not just a performance one — reducing both origin load and the actual data transfer bill simultaneously.

## Monitoring and governance — catching drift before it becomes a surprise bill

- **AWS Budgets** — proactive spend alerts, set against expected thresholds.
- **Cost Anomaly Detection** — machine-learning-based detection of unusual spending patterns, catching a runaway cost problem in near-real-time rather than discovering it for the first time at month-end billing.
- **Cost Explorer** — analysis and forecasting, for understanding trends rather than just point-in-time spend.

**Memorable hook:** *"Budgets tell you when you've crossed a line you already drew. Cost Anomaly Detection tells you something's behaving strangely even if you never thought to draw a line there in the first place."*

## A complete architectural pattern for bursty/elastic workloads

A genuinely useful, complete answer to have ready for "design a cost-optimized architecture for a bursty workload":

- **Compute**: serverless (Lambda/Fargate) where the workload fits, or aggressive Auto Scaling scale-in combined with Spot Instances for fault-tolerant components.
- **Database**: DynamoDB on-demand mode, or Aurora Serverless v2 scaling near-zero during quiet periods.
- **Storage**: S3 Intelligent-Tiering, automatically adapting to access pattern changes without manual intervention.
- **Caching**: CloudFront and ElastiCache absorbing read load at a cheaper layer, directly reducing how much expensive backend compute a traffic spike actually reaches — conceptually the same instinct as Day 5's load shedding, just applied to cost rather than availability.

## Real-world examples

1. **A comprehensive, multi-layered cost optimization proposal for a Thai enterprise customer**, combining Compute Optimizer right-sizing, Savings Plans, S3 Intelligent-Tiering, and VPC Gateway Endpoints into one coherent recommendation — a complete, credible answer that pulls together this entire day's material rather than naming one isolated technique.
2. **Recommending Cost Anomaly Detection and Budgets as an ongoing operational discipline**, not a one-time setup, for a customer specifically worried about unexpected bills — a practical, governance-minded recommendation.
3. **Designing a cost-optimized architecture for a bursty seasonal workload** — Thai retail holiday traffic or month-end banking batch processing are both realistic examples — using serverless compute, on-demand database scaling, and CloudFront caching together, a complete scenario answer synthesizing the whole day rather than a single service recommendation.
