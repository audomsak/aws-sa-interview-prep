# Containers on AWS — ECS vs EKS vs Fargate

This page is a direct continuation of Day 1's entire Kubernetes deep dive — not new material, but the same material with AWS's own managed control plane underneath it. If Day 1 felt solid, this page should feel like recognition, not new learning.

## The one-line hook

> **ECS vs EKS is "which orchestrator." EC2 vs Fargate is "who manages the compute underneath it." These are two separate, orthogonal decisions — and conflating them into one choice is a common, avoidable mistake.**

## ECS — AWS's own, simpler, proprietary orchestrator

**Elastic Container Service (ECS)** is AWS's own container orchestrator — not Kubernetes, a separate, simpler model built around **Task Definitions** (what to run) and **Services** (how many, and how they're kept running), with tight native integration into IAM, Application Load Balancer, CloudWatch, and VPC networking. There's no separate Kubernetes-style control plane to reason about — AWS handles orchestration entirely as an integrated service.

## EKS — AWS's managed Kubernetes, and where Day 1 pays off directly

**Elastic Kubernetes Service (EKS)** runs genuine, upstream Kubernetes — AWS manages the **control plane** (the exact kube-apiserver, etcd, scheduler, and controller-manager components built from scratch on Day 1's "Kubernetes the hard way" page), while you manage worker nodes (or hand that off to Fargate too, covered below).

**This is the single strongest cross-day connection of the whole week to make explicitly**: your **Red Hat OpenShift background is EKS's control plane, essentially unchanged** — OpenShift *is* Kubernetes, Day 1 already built the CRI/CNI/CSI/Operator mental model from first principles, and EKS is simply AWS's own managed distribution of that same core. There is close to zero new conceptual ground here.

**One genuinely different, worth-knowing detail**: EKS's default networking is the **AWS VPC CNI** — unlike the overlay-network approaches (Calico in VXLAN mode, OVN-Kubernetes) covered on Day 1, the AWS VPC CNI assigns each pod a **real, routable VPC IP address directly**, rather than encapsulating pod traffic inside an overlay. This is a genuinely different architectural choice, not just a different vendor's implementation of the same idea — worth naming precisely if asked to compare.

## Launch types — the orthogonal second decision

| | EC2 launch type | Fargate launch type |
|---|---|---|
| **Who manages the underlying compute** | You — patch, scale, and secure the EC2 worker nodes yourself | AWS — fully serverless, no nodes to manage at all |
| **Control** | Full node-level access (SSH, DaemonSets, custom AMIs) | Much less low-level control — no node access, limited DaemonSet-style workloads |
| **Cost** | Generally lower per-vCPU/memory at the underlying EC2 rate | Higher per-vCPU/memory, but no operational overhead of managing nodes |

**Critically, this choice applies to both ECS and EKS** — you can run ECS-on-EC2, ECS-on-Fargate, EKS-on-EC2, or EKS-on-Fargate (via EKS Fargate profiles, which schedule specific pods onto Fargate instead of self-managed worker nodes). Treating "ECS vs EKS vs Fargate" as one three-way choice, rather than two separate two-way choices, is a real, catchable oversimplification.

## Other EKS-specific tools worth naming

- **EKS Fargate profiles** — a declarative way to say "these specific pods run on Fargate," rather than an all-or-nothing cluster-wide choice.
- **Karpenter** — AWS's modern, fast node autoscaler for EKS, increasingly replacing the traditional Kubernetes Cluster Autoscaler, provisioning right-sized nodes in response to unschedulable pods more quickly and flexibly.

## The decision framework

| Choose | When |
|---|---|
| **ECS** | The team wants AWS-native simplicity, doesn't need Kubernetes-specific portability or the broader CNCF ecosystem, and prioritizes lower operational overhead over flexibility |
| **EKS** | The team already has Kubernetes expertise (directly your situation, given OpenShift), needs multi-cloud or hybrid portability, wants the CNCF ecosystem (Helm, Operators — a direct Day 1 callback), or is migrating an existing on-prem Kubernetes/OpenShift workload |
| **Fargate** (with either) | Default choice for new workloads, per the previous page's guidance — reach for self-managed EC2 worker nodes only when a specific cost or feature reason genuinely justifies the added operational burden |

## Real-world examples

1. **Your Red Hat OpenShift background making EKS immediately familiar.** This is worth stating explicitly and confidently in an interview: "OpenShift is a Kubernetes distribution — everything I covered about CRI, CNI, CSI, and Operators applies directly to EKS, just with AWS's own managed control plane and typically a different default CNI." A genuinely strong, personal, cross-day answer.
2. **The TnD Microservices platform's actual Kubernetes deployment**, given AWS was already explicitly part of that project's tech stack — if that platform needed to run on AWS specifically, EKS would be the natural choice, since the platform was already built on Kubernetes rather than a proprietary orchestration model.
3. **Recommending ECS with Fargate over EKS for a customer team without existing Kubernetes expertise**, prioritizing the fastest path to production with minimal operational overhead — a genuine, judgment-driven recommendation, deliberately contrasted with the EKS recommendation that fits your own OpenShift-experienced background specifically.
