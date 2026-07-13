# Helm — when it helps, when it hurts

## The one-line hook

> **The question isn't "what is Helm" — it's "when does Helm make things worse." "I always use Helm" is explicitly not a senior answer.**

## What Helm actually is, briefly

Helm packages Kubernetes manifests into reusable **charts** — templated YAML with configurable **values** — and manages **releases**, versioned deployments of a specific chart-plus-values combination, as one coherent unit you can install, upgrade, or roll back together.

## When Helm genuinely earns its complexity

- **Distributing reusable, parameterized software** across many environments or customers — an ISV shipping their application as a chart that different customers install with different configuration values is the textbook fit.
- **Complex, multi-resource applications** with many interdependent Kubernetes objects, where managing them as one versioned release is genuinely more coherent than tracking many separate manifest files by hand.
- **A large existing ecosystem of public charts** (Bitnami and others) for common infrastructure software — databases, message brokers — meaning you often don't need to hand-write manifests for well-known software at all.

## When Helm makes things worse — the actual senior insight

- **Debugging overhead.** A broken deployment now requires mentally "un-templating" YAML to understand what was actually applied — `helm template` or `helm get manifest` can show the rendered output, but that's a genuine extra debugging step plain manifests simply don't require at all.
- **Unnecessary complexity for simple apps.** A single-deployment, single-service application with little real configuration variance across environments doesn't need a templating engine — plain manifests, or a lighter-weight overlay approach (below), are more directly readable and have nothing extra to debug.
- **Values file sprawl.** As a chart grows, its `values.yaml` can become its own complex, hard-to-reason-about configuration surface — arguably relocating complexity rather than actually removing it.
- **A real, if manageable, tension with GitOps.** Day 7's earlier GitOps page centered on "what's declared in Git is exactly what's running" — a Helm release involves an extra compilation step from chart-plus-values into a rendered manifest, some friction with that philosophy's cleanest form. ArgoCD and Flux both support Helm chart sources directly, so this is a real nuance worth naming, not a hard blocker.

**Memorable hook:** *"Helm is genuinely excellent at packaging software for other people to configure and install. It's genuinely mediocre at making your own simple internal app easier to understand — and conflating those two use cases is exactly what 'I always use Helm' gets wrong."*

## Kustomize — the notable, lighter-weight alternative

**Kustomize** takes a different approach: **overlay-based patching** of a base manifest per environment, rather than a full templating language — no separate templating syntax, works natively with `kubectl`, and is often the better fit for internal applications with modest environment-to-environment variance, where Helm's full machinery is more than the problem actually needs.

## The decision framework

| Scenario | Best fit |
|---|---|
| Distributing packaged software to many consumers with real configuration variance | **Helm** |
| Consuming well-known third-party infrastructure software | **Helm** — leverage the existing public chart ecosystem |
| An internal app with simple, modest cross-environment differences | **Plain manifests or Kustomize** |

## Real-world examples

1. **Kong's own Helm chart for deploying Kong Gateway on Kubernetes**, directly relevant to your current employer — a genuine, product-grounded example of Helm being the right choice: complex, widely-used, configurable infrastructure software distributed to many different customer environments.
2. **A simple, single-service internal microservice on the TnD Microservices platform genuinely not needing Helm** — plain manifests or Kustomize overlays being the more honest, lower-overhead choice for something with minimal cross-environment variance, a real "not everything needs Helm" judgment call.
3. **Debugging a broken Helm release using `helm get manifest`** to inspect the actual rendered YAML that was applied — a concrete, specific operational technique demonstrating real hands-on troubleshooting rather than surface-level tool familiarity.
