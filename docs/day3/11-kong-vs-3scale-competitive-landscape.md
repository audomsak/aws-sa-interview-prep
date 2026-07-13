# Kong vs 3scale (and the wider landscape)

You've genuinely sold both sides of this comparison — 3scale as a Red Hat Solution Architect, Kong today as a CSM. That's rare, authentic material most candidates simply don't have, and worth using directly rather than reciting a generic feature comparison.

## The one-line hook

> **Kong and 3scale actually share the same technical lineage — both are Nginx-based gateways underneath — which means the real differentiators aren't "which one is technically better," they're extensibility model, ecosystem positioning, and which use case each platform was originally built to solve.**

## Kong vs Red Hat 3scale, honestly

| | Kong | Red Hat 3scale |
|---|---|---|
| **Gateway lineage** | Nginx + OpenResty + LuaJIT | APIcast, also Nginx-based |
| **Origin/heritage** | Built API-gateway-first, extensibility via a large Lua plugin ecosystem | Built with strong roots in **API-as-a-product** — developer portals, monetization, API lifecycle management were core from early on, not bolted on later |
| **Extensibility model** | Write custom Lua plugins, hooking into the request/response lifecycle directly | More built-in policy-based configuration; extending beyond built-in policies is less of an open, code-first developer experience than Kong's plugin model |
| **Kubernetes-native story** | Kong Ingress Controller, CRDs for routing/plugins, DB-less mode built for GitOps | Deep integration with Red Hat OpenShift specifically — a natural fit if a customer is already committed to the OpenShift ecosystem |
| **Control plane model** | Konnect — SaaS-hosted, multi-tenant control plane with dedicated data planes anywhere | Historically similar in spirit: a 3scale SaaS (or on-prem) management/control layer paired with on-prem APIcast gateway instances |
| **Developer portal / monetization maturity** | Present, but a comparatively newer focus area | Historically a genuine strength — 3scale's core original use case was literally API productization and monetization for API-as-a-business scenarios |

**Memorable hook:** *"Kong grew up as a gateway that became a platform. 3scale grew up as an API-as-a-product platform that happened to need a gateway underneath it. Same Nginx foundation, very different center of gravity."*

## Where each one genuinely wins

- **3scale wins** when a customer is already deep in the Red Hat/OpenShift ecosystem, or when the core business need is genuinely API productization — external developer portals, usage-based monetization, formal API-as-a-product lifecycle management.
- **Kong wins** when a customer wants maximum extensibility (custom Lua plugins for bespoke requirements), a lighter-weight and more Kubernetes-native operating model, or is building a modern, cloud-native, possibly multi-cloud platform without a pre-existing OpenShift commitment.

## The wider landscape, briefly

| Platform | Positioning |
|---|---|
| **Apigee** (Google) | Strong enterprise governance and analytics focus, often chosen by large enterprises wanting deep API lifecycle analytics |
| **AWS API Gateway** | Native to AWS, serverless-first — deeply integrated with Lambda and Cognito, usage-based pricing baked into the AWS billing model |
| **Azure API Management** | Azure's native equivalent, similarly deep first-party cloud integration |
| **Tyk** | Another open-source-first gateway, positioned similarly to Kong in the lighter-weight, developer-friendly space |

**The specific, relevant nuance for this exact interview**: for a customer fully committed to AWS serverless architecture (Lambda functions, Cognito for auth), **AWS API Gateway often wins on integration depth and simplicity** over a third-party gateway like Kong or 3scale — not because it's technically superior in the abstract, but because first-party integration removes real friction a third-party tool can't fully close. A credible, honest competitive answer acknowledges this rather than reflexively defending "the tool I currently sell" in every scenario.

## Real-world examples

1. **Your own career arc — 3scale at Red Hat, Kong at Kong.** This is the single strongest piece of material on this whole page: you're not reciting a feature comparison you read somewhere, you've genuinely positioned both products competitively, in real customer conversations. Say that directly.
2. **A Red Hat OpenShift customer already invested in 3scale, versus a cloud-native customer without that constraint choosing Kong** for its lighter-weight, more flexible Lua plugin model — a realistic, honest positioning scenario straight from your actual sales experience.
3. **An AWS-native customer where AWS API Gateway's tight Lambda/Cognito integration might beat a third-party gateway on simplicity**, despite Kong or 3scale's stronger multi-cloud portability — a nuanced, credible answer that's especially relevant given this is, itself, an AWS interview.
