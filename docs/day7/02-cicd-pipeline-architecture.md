# CI/CD pipeline architecture & artifact promotion

## The one-line hook

> **Build once, promote the same artifact through every environment. Rebuilding for staging and rebuilding again for production means you've lost reproducibility — "the rule nobody breaks," and one of the most common real sources of "works in staging, fails in prod" incidents when it is broken.**

## The core pipeline stages

Build → Test (unit, integration, sometimes end-to-end) → Package into an artifact → Deploy, environment by environment — with security scanning stages running in parallel alongside test, not bolted on as an afterthought at the end (full treatment on the DevSecOps page later today).

## Build once, promote everywhere — the rule, precisely

An **artifact** — a Docker image, a JAR, a zipped Lambda bundle — is built exactly **once**, in the CI stage, and identified by an **immutable reference** (a content digest or a specific version tag, never a mutable tag like `latest`). That same artifact is then **promoted** through dev, staging, and production — moved or re-tagged in an artifact registry, never rebuilt.

**Why rebuilding per environment is genuinely dangerous, not just wasteful**: a second build, even from identical source code, can silently pick up a different dependency version (if a floating version range resolves differently), a different build timestamp, or subtly different toolchain behavior — meaning what actually got tested in staging is **not** provably the same thing running in production. Promoting the literal, already-tested artifact closes that gap entirely.

**Memorable hook:** *"If you rebuild for production, you're not promoting what you tested — you're testing one thing and shipping something merely similar to it."*

**The specific, important detail about immutable references**: deploying by pulling `latest` is a real anti-pattern, since `latest` is a mutable pointer — two different pulls of `latest`, minutes apart, can resolve to genuinely different images. Production deployments should always reference an immutable digest or a specific, fixed version tag.

## Pipeline speed — a concrete, practical checklist

- **Profile first.** Find the actual slow stage before optimizing blindly — a common, wasted effort is speeding up a stage that was never the bottleneck.
- **Cache dependencies and build layers** — avoid re-downloading the same dependencies from scratch on every single run.
- **Parallelize independent stages** — lint, unit tests, and security scans can usually run concurrently rather than sequentially, since none of them depend on each other's output.
- **Shrink images and build context** — multi-stage Docker builds keep the final image lean, speeding up both the build itself and every subsequent pull.

## Jenkins vs. modern SaaS CI (GitHub Actions, GitLab CI) — a real judgment, not just tool trivia

| | Jenkins | GitHub Actions / GitLab CI |
|---|---|---|
| **Hosting** | Self-hosted — you run and maintain the Jenkins infrastructure itself | SaaS-hosted, no separate infrastructure to operate |
| **Extensibility** | Highly extensible via a large plugin ecosystem | Extensible, but more opinionated and tightly scoped to the platform |
| **Integration** | Works with any source control platform | Tightly integrated with its own source control platform specifically |
| **Operational cost** | Real, ongoing — patching, scaling, and securing Jenkins itself is your job | Effectively none — the CI platform's own uptime and scaling is the vendor's responsibility |

The honest, current judgment: **Jenkins remains common in mixed cloud/on-prem environments or shops with deep existing plugin investment**, but for a team starting fresh, the reduced operational overhead of SaaS-hosted CI is a real, legitimate reason to default there instead — not just a fashion trend.

## Artifact promotion mechanics and multi-service coordination

An artifact registry (a container registry, or a tool like Nexus/Artifactory) is the central store artifacts move through. Promotion means moving or re-tagging the same artifact reference between environment-scoped tags — never re-triggering a build. For coordinating releases across many services, teams often use **release trains** (a fixed cadence, such as bi-weekly, where multiple services release together in lockstep) versus fully independent per-service deploys — a real coordination-vs-autonomy tradeoff. Rollback, done correctly, is simply **pinning back to the previous known-good artifact reference** — which is exactly why immutable references matter so much: you can only reliably roll back to something you can precisely and unambiguously identify.

## Real-world examples

1. **The build-once-promote-everywhere principle applies regardless of which branching strategy or repo structure a team uses**, directly connecting back to the previous page — Git Flow, GitHub Flow, Trunk-Based, mono-repo, or polyrepo all still need this same underlying discipline; it's a unifying principle across every variation already covered.
2. **A Kong Gateway Docker image built once, tagged with an immutable version, and promoted through dev, staging, and production**, rather than rebuilt at each stage — a concrete, product-grounded example directly relevant to your current role.
3. **Diagnosing a "works in staging, fails in production" incident that traces back to a violated build-once-promote-everywhere rule** — someone rebuilt for production instead of promoting the already-tested staging artifact, and a subtle dependency resolution difference between the two builds caused the divergence — a realistic, valuable incident story that demonstrates why this rule matters in practice, not just as a slogan.
