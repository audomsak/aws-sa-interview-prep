# Day 3 interview Q&A drill

**How to use this page:** cover the answers, read only the question, answer out loud as if the interviewer is sitting across from you, *then* check. The goal isn't to reread — it's to catch the gap between what you can say cold and what you can only recognize when you see it.

---

**1. Why does an API Gateway exist at all — what problem does it actually solve?**

> **Hook: "Same N² math as Day 2's integration hub, applied to client-facing traffic."**
> Without a gateway, every external client (web, mobile, partner) needs to know about, integrate with, and separately secure every backend service directly — with N clients and M services, that's up to N×M direct relationships, each absorbing every internal refactor as a breaking change. A gateway centralizes routing, authentication, rate limiting, protocol translation, and observability behind one stable front door.

---

**2. What's the difference between contract-first and code-first API design, and why does it matter for parallel team development?**

> **Hook: "Code-first documents what you built. Contract-first designs what consumers need, and holds implementation accountable to it."**
> Contract-first means the OpenAPI specification is written before implementation, letting other teams (mobile, frontend, partners) build against a generated mock server immediately while the backend is still being built. Code-first generates the spec from the implementation afterward, meaning consumers only learn the real contract once code — and often the API itself — already exists, blocking or complicating parallel development.

---

**3. Explain Kong's control plane / data plane separation, and why data planes are deliberately stateless.**

> **Hook: "The control plane is never in the request path — same principle as a service mesh's control plane, or Kubernetes' control plane not sitting in pod-to-pod traffic."**
> Data plane nodes proxy real client traffic and are stateless, so they scale horizontally behind a load balancer with no coordination needed between them. The control plane manages configuration (Services, Routes, Plugins) and pushes it to data planes — but if the control plane goes down, existing data plane nodes keep serving traffic using their already-cached configuration; only *new* config changes stall.

---

**4. What's the difference between Kong's traditional (DB) mode, DB-less mode, and hybrid mode?**

> **Hook: "Hybrid mode is DB-less applied specifically to data planes, with a separate control plane still holding the database."**
> Traditional mode has every node connect directly to a shared database, coupling every node's health to database availability. DB-less mode loads configuration from a declarative file into memory with no database at all. Hybrid mode separates the two: control plane nodes hold the database and manage configuration; data plane nodes run DB-less, receiving pushed configuration from the control plane — the recommended production pattern, since it isolates database concerns from the actual traffic-serving path.

---

**5. How does Kong Konnect's deployment model solve the "noisy neighbor" problem?**

> **Hook: "Multi-tenant control plane, single-tenant data planes — shared governance, isolated traffic."**
> Konnect provides a centralized, multi-tenant control plane (hosted by Kong) paired with dedicated, single-tenant data planes deployed per environment or tenant. This avoids sharing actual traffic-serving data plane capacity across tenants in high-traffic environments, where one tenant's load spike could otherwise degrade performance for another sharing the same data plane nodes — while still centralizing configuration governance.

---

**6. Explain how Kong plugin scoping precedence can cause unexpected production behavior.**

> **Hook: "A more specific scope silently overriding a broader one is one of the most common real sources of confusion."**
> Plugins can be scoped globally, per-service, per-route, or per-consumer, and a more specific scope generally overrides or adds to a broader one for the same plugin type. A route-level rate-limiting override, for instance, can silently supersede an expected global policy — leading to a route behaving differently than the rest of the platform in a way that isn't obvious just from looking at the global configuration.

---

**7. What's the fixed window rate limiting boundary flaw, precisely?**

> **Hook: "A client can legally burst double the limit right at the window edge."**
> If a client sends no requests for the first 50 seconds of a 60-second, 100-request window, then fires 100 requests in the last 10 seconds, the window resets and they can immediately fire another 100 requests in the first 10 seconds of the new window — 200 requests within a 20-second span, double the intended rate, entirely within the algorithm's own rules.

---

**8. Why is token bucket the most commonly used rate limiting algorithm in production systems?**

> **Hook: "It's the only algorithm here that treats bursting as a designed-for behavior, not an edge case."**
> Token bucket only needs to store two numbers per client — current token count and the last request's timestamp — calculated lazily rather than needing a background refill process, making it memory-efficient at scale. It also naturally supports realistic traffic patterns: a client can legitimately burst up to the bucket's capacity after being idle, then is throttled to the steady refill rate once drained.

---

**9. Why does distributed rate limiting require centralized state like Redis, instead of each node counting independently?**

> **Hook: "With N nodes each counting independently, a client can get N times the intended limit."**
> If each horizontally-scaled gateway or service instance tracks its own local counters, a client's requests being load-balanced across different nodes means no single node ever sees the client's true total request count — effectively multiplying their real allowed rate by the number of nodes. Centralizing counters in a shared, fast store like Redis ensures every node checks and increments against the same state, regardless of which node actually receives a given request.

---

**10. What's the actual difference between OAuth2 and OIDC — and why is confusing them a real security design mistake?**

> **Hook: "OAuth2 answers 'can this app do this on my behalf?' OIDC answers 'who is this person, actually?'"**
> OAuth2 is strictly an authorization framework — granting a client limited, scoped access to a resource on a user's behalf. It was never designed to assert the human's actual identity to a resource server. OIDC layers authentication on top, via an ID Token (a JWT with identity claims) issued alongside the access token. Treating a plain OAuth2 access token as proof of user identity — without an actual OIDC ID token — is a genuine, common security design mistake.

---

**11. Why is the Authorization Code flow with PKCE considered more secure than the Implicit flow?**

> **Hook: "The access token never touches the browser in Authorization Code — it does, directly, in Implicit."**
> In Authorization Code flow, only a short-lived authorization code is returned to the browser; the actual token exchange happens server-to-server, keeping any client secret off the client side. PKCE extends this to clients that can't hold a secret (mobile apps, SPAs) by requiring a code_verifier at exchange time that only the original client possesses, making a stolen authorization code alone useless to an attacker. Implicit flow returned the access token directly in the browser redirect with no exchange step, making it inherently more exposed — which is why it was formally removed in OAuth 2.1.

---

**12. Explain the JWT `alg: none` attack and how to defend against it.**

> **Hook: "Never trust what the token claims about itself."**
> An attacker takes a valid JWT, modifies the payload, and changes the header's algorithm field to `none`. A verifier that blindly trusts the token's own declared algorithm may accept it without checking any signature at all. The defense is that server-side validation must never read the algorithm to use from the token itself — it must validate against a pre-configured allowlist (such as only accepting `RS256`) and reject anything outside it.

---

**13. What does refresh token rotation actually protect against?**

> **Hook: "It turns silent, indefinite token theft into a detectable event."**
> Every time a refresh token is used to obtain a new access token, the authorization server issues a brand-new refresh token and invalidates the old one. If an attacker has stolen a refresh token and uses it, the legitimate client's next attempt to use its now-superseded token will fail — giving a clear, detectable signal that theft occurred, rather than allowing an attacker to silently ride along on a stolen long-lived token indefinitely.

---

**14. What does mTLS actually prove, and what does it not prove?**

> **Hook: "mTLS proves which machine — not which user."**
> Mutual TLS has both the client and server present certificates, each verified against a trusted CA, giving authentication, encryption, and integrity for the connection. It proves machine/service identity — that this specific service, holding this specific certificate, is on the other end of the connection. It says nothing about which end user, if any, the request is ultimately on behalf of — that requires a separate user-identity token layered on top.

---

**15. Describe the token propagation pattern for combining service identity and user identity across a microservices call chain.**

> **Hook: "Two tokens, because they answer two different questions."**
> A service token (an mTLS certificate, or a service-issued JWT) proves which service is calling — "am I willing to accept requests from this service at all?" A user token (typically OIDC-issued) is passed through the entire call chain unmodified, proving which end user the request is for — "is this specific user authorized for this specific operation?" Each service in the chain validates both independently, rather than re-authenticating the user at every hop or trusting service identity alone.

---

**16. Why is "API Gateway is north-south, Service Mesh is east-west" considered an oversimplification, if not outright wrong?**

> **Hook: "Both can technically handle either direction — the real distinction is purpose, not traffic direction."**
> An API gateway can manage internal service-to-service traffic through internal gateways with governance and versioning; a service mesh's ingress gateway can handle external client-facing entry into the mesh. Traffic direction doesn't cleanly separate the two. The real distinction is purpose: a gateway treats services as products, with user governance and business context; a service mesh is business-agnostic infrastructure, concerned with whether a network call between two services succeeds reliably and securely.

---

**17. Why shouldn't you route all internal service-to-service traffic through a central API gateway?**

> **Hook: "Same N² problem from Day 2, reused — now applied to internal traffic instead of external."**
> With 50 services each potentially calling 10 others, routing all 500 of those internal paths through one central gateway makes it both a performance bottleneck and a single point of failure, and adds an unnecessary network hop's worth of latency to calls that don't need centralized, business-facing governance at all — internal service-to-service concerns are better handled by a service mesh's distributed sidecar model.

---

**18. How does the sidecar pattern in a service mesh relate to what you learned about Kubernetes pod networking on Day 1?**

> **Hook: "The sidecar is just another container sharing the same pod network namespace the pause container already set up."**
> Every container in a Kubernetes pod shares one network namespace by default (via the invisible "pause" container covered on Day 1). A service mesh's Envoy sidecar takes direct advantage of that sharing — running as a second container in the same pod, it transparently intercepts every request in and out of the application container without any application code changes, handling mTLS, retries, and circuit breaking on its behalf.

---

**19. Why does an AI Gateway need token-based rate limiting instead of request-based rate limiting?**

> **Hook: "LLM cost is measured in tokens, not requests — a request-count limiter doesn't actually protect your budget."**
> A single LLM request can consume wildly different amounts of tokens depending on prompt and response length, and cost scales with tokens consumed, not request count. A rate limiter counting only requests could let a client send a small number of extremely token-expensive requests that blow through a cost budget a request-count limit was never designed to catch — token bucket's underlying algorithm generalizes cleanly to this, just metering tokens instead of requests.

---

**20. What's the connection between an AI Gateway and MCP (Model Context Protocol) security?**

> **Hook: "MCP alone has real security gaps; an AI Gateway can add the governance layer it doesn't guarantee on its own."**
> MCP defines how models interact with tools and external context, but doesn't inherently provide strong authentication, authorization, or visibility over that traffic. An AI Gateway sitting in front of MCP servers and tool calls can add that missing governance layer — and critically, needs to be context-aware enough to distinguish a plain model invocation from an autonomous tool call, since those carry meaningfully different security implications.

---

**21. What actually counts as a breaking API change requiring a new version, versus a safe additive change?**

> **Hook: "If an existing, unmodified client would break without any changes on their end, it's breaking. If they'd simply never notice, it isn't."**
> Adding a new optional field, a new endpoint, or relaxing a validation rule are safe within an existing version. Removing a field, renaming a field, changing a field's data type, or making a previously optional field required are all breaking changes that require a new version — because an existing client's code, written against the old contract, would fail without any changes on their part.

---

**22. Why is URI versioning still common for external, partner-facing APIs despite being less "RESTfully pure" than content negotiation?**

> **Hook: "URI versioning is the one your least sophisticated partner integration team will get right on the first try."**
> URI versioning is simple, visible, and trivially testable — a partner engineer can see and understand `/v2/orders` without any special client tooling. Content negotiation is more technically correct from a REST purist standpoint, but requires meaningfully more sophisticated client-side implementation to set the right `Accept` header correctly — a real cost when the actual audience is external partners with varying technical sophistication, not an internal team you can mandate tooling standards for.

---

**23. What's the real historical/architectural difference between Kong and Red Hat 3scale, beyond "they're competitors"?**

> **Hook: "Kong grew up as a gateway that became a platform. 3scale grew up as an API-as-a-product platform that happened to need a gateway underneath it."**
> Both are actually Nginx-based underneath (Kong via OpenResty/Lua, 3scale via APIcast) — the real differentiator isn't the gateway core, it's what each platform was originally built around. Kong's center of gravity is gateway extensibility via a large Lua plugin ecosystem; 3scale's original strength was API productization — developer portals, monetization, and formal API lifecycle management as core, early capabilities, not features bolted on later.

---

**24. For a customer fully committed to AWS serverless architecture, would you recommend Kong or AWS API Gateway, and why?**

> **Hook: "First-party integration removes real friction a third-party tool can't fully close — acknowledging that honestly is stronger than defending your own product reflexively."**
> For a customer deeply invested in Lambda and Cognito, AWS API Gateway often wins on integration depth and operational simplicity — not because it's technically superior in the abstract, but because native, first-party integration avoids friction a third-party gateway would need extra configuration to bridge. Kong's advantage — multi-cloud portability, plugin extensibility, a consistent gateway experience across environments — matters most when a customer isn't fully committed to one cloud provider's native stack. A credible answer weighs the customer's actual constraints rather than defaulting to one tool in every scenario.
