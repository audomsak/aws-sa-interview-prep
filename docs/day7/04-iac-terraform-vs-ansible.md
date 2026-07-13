# Infrastructure as Code — Terraform vs Ansible

## The one-line hook

> **"Terraform for Day 0, Ansible for Day 1 and Day 2" — provisioning the infrastructure is a fundamentally different job from configuring what runs on it, and confusing which tool does which is a specifically-flagged interview red flag.**

## What IaC solves

**Infrastructure as Code** defines infrastructure in version-controlled, reviewable, reproducible code files rather than manual console clicks — directly solving configuration drift, undocumented changes, environment inconsistency, and the inability to reliably reproduce an environment from scratch. A candidate who never mentions version control for infrastructure changes is signaling reliance on manual processes — informally, "ClickOps" — a specific, named red flag worth actively avoiding in how you describe your own experience.

## Declarative vs. imperative — and why declarative wins for infrastructure

**Imperative** means specifying the exact steps, in order — a bash script that installs packages. **Declarative** means specifying the desired end state and letting the tool figure out how to get there — a Terraform file or a Kubernetes manifest. Declarative wins for infrastructure specifically because it's **idempotent and self-correcting**: the tool converges reality toward your specification regardless of the actual starting point, which is exactly the same property Day 1's Kubernetes reconciliation material and this same day's GitOps page both already built from different angles.

## Terraform vs. Ansible — the precise distinction

| | Terraform | Ansible |
|---|---|---|
| **Job** | **Provisioning** — creates and manages cloud resources (VMs, networks, load balancers, databases) via provider APIs | **Configuration management** — runs procedural tasks against already-existing infrastructure |
| **Model** | Declarative desired-state | Declarative desired-state, expressed through more procedural, task-based playbooks |
| **Tracks** | What it created, in a **state file** | Current configuration state on each managed host, checked and corrected on each run |
| **Typical use** | Stand up a VPC, EC2 instances, an EKS cluster, an RDS database | Install packages, copy configuration files, restart services on hosts that already exist |

**Memorable hook, the exact framing worth quoting**: *"Terraform for Day 0, Ansible for Day 1 and Day 2."* Terraform builds the house. Ansible moves in the furniture and keeps the utilities running afterward.

**The specifically-named interview red flag**: confusing configuration management with provisioning — describing Ansible as something that "creates the VPC" or Terraform as something that "installs packages on running servers" — is a real, catchable mistake that signals the two tools' actual roles were never clearly understood.

## Terraform state — the mechanism that makes it actually work

Terraform records everything it has created in a **state file**, and uses that file — not a live query against the cloud provider on every run — to know what to create, change, or destroy on the next `apply`. Keeping that state file **on a single laptop is a real, concrete disaster for a team**: two people running `apply` simultaneously against locally-held state corrupt each other's view of reality, with no coordination between them at all.

**Remote state** — stored centrally, commonly in an S3 bucket with **DynamoDB providing locking** (a direct, concrete tie to Day 6's S3/DynamoDB material, now showing up as Terraform's own backend infrastructure), or in Terraform Cloud — fixes this by ensuring only **one `apply` runs at a time**, with everyone sharing one authoritative source of truth for state.

**Memorable hook:** *"Locking here is the exact same problem Day 5's consensus material solved in the abstract — ensuring only one writer can act on shared state at once — just applied to Terraform's own state file specifically."*

## Drift detection — the same discipline as GitOps, one layer down

If someone manually changes a resource directly in the AWS console — outside of Terraform entirely — reality has drifted from what the state file believes is true. Running `terraform plan` regularly (not just at deploy time) surfaces exactly this kind of drift, by comparing the state file against the cloud provider's actual current state. This is precisely the same discipline the previous page covered for GitOps and Kubernetes drift — just applied one layer down, to raw cloud infrastructure instead of application deployments.

## How they combine in a real stack

Most real production environments use **both**, in sequence: **Terraform provisions the underlying infrastructure** (the VPC, EC2 instances or an EKS cluster, an RDS database — directly connecting to Day 6's entire AWS architecture material), and **Ansible then configures what actually runs on top of it** — installing software, managing application-level configuration, restarting services as needed. Neither tool replaces the other; they operate at genuinely different layers of the same stack.

## Real-world examples

1. **A complete IaC pipeline for a customer's AWS environment**: Terraform provisioning the VPC, EC2 fleet, and EKS cluster (directly connecting to Day 6's AWS material), followed by Ansible configuring the application-level software running on top — a coherent, cross-day answer synthesizing two full days of this course's material.
2. **Terraform remote state backed by S3 with DynamoDB locking** as the concrete implementation detail, directly tying Day 6's S3/DynamoDB AWS knowledge to this day's IaC material — a strong, specific, same-course cross-day technical connection.
3. **Diagnosing infrastructure drift** — a security group rule manually changed in the AWS console, surfaced by a routine `terraform plan` — directly paralleling the GitOps drift-detection concept from the previous page, demonstrating that the same operational discipline applies at both the raw infrastructure layer and the application deployment layer.
