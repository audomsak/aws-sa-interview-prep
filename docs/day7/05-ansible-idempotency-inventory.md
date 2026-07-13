# Ansible idempotency & dynamic inventory

Research called idempotency "the first filter, not optional" for Ansible-focused interview questions — the bar isn't just defining it, it's showing you've handled the edge cases where it actually breaks.

## The one-line hook

> **Running the same playbook twice should never double-apply anything — it should only touch what's actually drifted. A script that appends a line to a file every time it runs is the exact opposite of this, and it's the cautionary example worth having in mind.**

## Idempotency, precisely

**Idempotency** means running an operation multiple times produces the same result as running it once. In Ansible terms: executing a playbook a second time against a host that's already in the desired state should report **no changes**, not silently reapply everything again. This is precisely what makes it safe to run a playbook on a schedule — a nightly cron job, a periodic compliance check — without fear of compounding, unintended changes each time it fires.

This is a direct extension of Day 2's idempotent consumer material — the same underlying principle, just applied to configuration management instead of message processing.

## How Ansible modules achieve this — and where it actually breaks

Most **Ansible modules** are idempotent by design: the `package` module checks whether a package is already installed before attempting to install it; `template`/`copy` compare file content or checksums before overwriting; the `service` module checks current running state before starting or stopping anything. Each module does its own state-checking internally.

**The genuine, specific trap worth knowing precisely**: using the `shell` or `command` module for a task that already has a proper, purpose-built idempotent module available — running `shell: apt install nginx` instead of the `apt`/`package` module, for instance — silently **breaks idempotency**, because raw shell/command execution has **no built-in state-checking at all**. It just runs the command, every single time, unconditionally.

**Memorable hook:** *"A proper Ansible module asks 'is this already true?' before doing anything. `shell` and `command` don't ask — they just do, every time, whether it needed doing or not."*

### Handling operations that genuinely must use `shell`/`command`

Sometimes a raw shell command is unavoidable — no purpose-built module exists for the specific task. Ansible provides real tools to make even these idempotent:

- **`creates`** — only run the task if a specified target file **doesn't already exist** (a strong signal the work hasn't been done yet).
- **`removes`** — the inverse, only run if a specified file **does** exist.
- **`changed_when`** — an explicit conditional overriding Ansible's default "this task always reports changed" assumption for shell/command tasks, letting you define precisely what actually counts as a real change.

This is exactly the "edge case handling, not just happy-path playbooks" bar research flagged directly — a shell task wrapped correctly with `creates` or `changed_when` is a concrete, specific answer, not a vague "I try to keep things idempotent."

## Handlers — idempotency's close cousin

A **handler** (triggered via `notify`) only runs when an actual **change** occurred earlier in the playbook — restarting a service only if its configuration file actually changed, for instance, rather than restarting it unconditionally on every single playbook execution. This reinforces the same changed-state-driven philosophy, applied specifically to side-effect actions like service restarts.

## Static vs. dynamic inventory

A **static inventory** is a plain text file listing managed hosts — simple, but it **"rots the moment someone adds a server"** (the exact, memorable framing worth using), since nothing keeps it automatically in sync with actual infrastructure. A **dynamic inventory** instead pulls the current host list from a live source of truth — AWS EC2 tags, a CMDB, NetBox, vCenter — so the inventory stays accurate as infrastructure genuinely changes, without anyone needing to remember to update a file by hand.

**The directly AWS-relevant, current detail**: an **AWS EC2-tag-based dynamic inventory plugin** querying instances by tag is exactly the mechanism that would keep an Ansible inventory automatically correct as an Auto Scaling Group (Day 6's material) adds and removes instances — a static inventory file simply cannot keep up with that kind of genuinely elastic infrastructure.

## Ansible Vault — a direct forward-reference

**Ansible Vault** encrypts sensitive data (secrets, credentials) directly within playbooks or variable files, so they can live safely in version control rather than sitting in plaintext — a direct connection forward to this same day's DevSecOps secrets management page.

## Real-world examples

1. **Your own resume's listed use of Ansible for the nbn iB2B platform** — the strongest possible anchor for this entire page, since it's a genuine, first-hand example rather than a hypothetical. Being able to describe real idempotent playbook design choices from that actual work is directly credible in a way abstract knowledge isn't.
2. **An AWS EC2-tag-based dynamic inventory**, directly recalling Day 6's Auto Scaling Group material, automatically including or excluding hosts as instances come and go — a concrete, current, AWS-specific application of dynamic inventory.
3. **A concrete story about fixing a shell task that broke idempotency** — using the `creates` parameter to prevent a raw shell command from destructively re-running on every playbook execution — a specific example demonstrating exactly the edge-case-handling bar research identified as the actual test.
