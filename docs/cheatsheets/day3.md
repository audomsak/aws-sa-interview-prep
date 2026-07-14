# Day 3 hooks — API Management & Gateway Architecture

Every hook from [Day 3](../day3/index.md), in page order — 54 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [The API Gateway pattern & why it exists](../day3/01-api-gateway-pattern-fundamentals.md)

- **An API Gateway exists so that external clients integrate with one stable front door, instead of every client needing to know about, trust, and separately secure every backend service behind it.**
- *"A gateway isn't just a router with extra steps — it's the one place where cross-cutting concerns (auth, rate limiting, observability) get enforced consistently, instead of trusting every individual service to implement them correctly and identically."*
- *"One gateway trying to please every client type eventually pleases none of them well. BFF says: give the mobile app its own front door, shaped exactly for what the mobile app needs, and give the partner integration a different one."*

## [OpenAPI-first / contract-first API design](../day3/02-openapi-contract-first-design.md)

- **Contract-first means the API specification is the first thing written, and everything else — implementation, documentation, gateway configuration, even mock servers for other teams — is generated from or verified against it. Code-first means the spec is an afterthought, generated from whatever the code happened to end up looking like.**
- *"Code-first documents what you built. Contract-first designs what your consumers actually need, and holds the implementation accountable to it."*
- *"A contract without contract testing is just a promise nobody's checking. Contract testing is the mechanism that catches the moment a provider quietly stops keeping that promise."*

## [Kong architecture deep dive](../day3/03-kong-architecture-deep-dive.md)

- **Kong's entire architectural story is the same separation Day 1 and Day 2 kept surfacing all week: a stateless thing that handles traffic, and a separate thing that manages configuration and never sits in the request path.**
- *"If the control plane goes down, existing traffic keeps flowing — data plane nodes already have their configuration cached locally and don't need to ask the control plane per request. Only *new* config changes stall."*

## [Kong plugins & declarative config](../day3/04-kong-plugins-declarative-config.md)

- **A Kong plugin is Lua code that hooks into a specific phase of the request/response lifecycle — and every plugin attached to a route runs as a chain, in a defined order, not as independent, isolated add-ons.**
- *"Plugins aren't a pile of independent features — they're a pipeline. The order they run in is itself part of the design, not an implementation detail you can ignore."*
- *"`deck` does for Kong config exactly what `kubectl apply` does for Kubernetes manifests — describe the desired state, let the tool reconcile reality toward it."*

## [Rate limiting algorithms](../day3/05-rate-limiting-algorithms.md)

- **Every rate limiting algorithm is a different tradeoff between memory cost, precision, and whether it allows bursts — there's no universally "best" one, only the right fit for a specific traffic pattern.**
- *"Fixed window doesn't actually limit the rate — it limits the count per arbitrary clock interval, and clever clients can straddle the boundary to burst right through it."*
- *"Token bucket is the only algorithm here that treats bursting as a legitimate, designed-for behavior instead of an edge case to guard against — which is exactly why real production APIs (Stripe, AWS) use it."*
- *"Token bucket lets a burst through and then slows down. Leaky bucket never lets a burst through at all — it smooths everything to one steady drip, no matter how it arrived."*

## [OAuth2 & OIDC deep dive](../day3/06-oauth2-oidc-deep-dive.md)

- **OAuth2 answers "can this app do this on my behalf?" — authorization. OIDC, built on top of OAuth2, answers "who is this person, actually?" — authentication. Confusing the two is one of the most common real mistakes in API security design.**
- *"Never trust what the token claims about itself — the `alg` header, in particular, is data from the attacker's own forged message, not a security decision to honor."*

## [mTLS & service-to-service security](../day3/07-mtls-service-to-service-security.md)

- **Regular TLS proves the server is who it claims to be. Mutual TLS (mTLS) proves both sides are who they claim to be — and you've already built the exact mechanism behind it once this week, on Day 1, just applied to Kubernetes control plane components instead of application services.**
- *"mTLS answers 'which machine is this,' not 'which user is this.' Confusing the two is exactly the same mistake as confusing OAuth2 authorization with OIDC authentication from the previous page."*
- *"One token can't honestly answer two different questions. Service identity and user identity are separate facts, and the token propagation pattern keeps them as two separate tokens instead of awkwardly conflating them into one."*

## [API Gateway vs Service Mesh](../day3/08-api-gateway-vs-service-mesh.md)

- **"API Gateway is north-south, Service Mesh is east-west" is the answer most people give, and it's a myth. Both can technically handle either traffic direction. The real distinction is purpose: a gateway treats services as products; a mesh treats them as infrastructure.**
- *"A gateway asks 'is this client allowed to call this API, and how do we bill/govern that?' A mesh asks 'is this network call between two services going to succeed, securely and reliably?' — genuinely different questions, regardless of which direction the traffic happens to be flowing."*
- *"Trying to make one tool do both jobs doesn't just add complexity — it actively breaks specific things the other tool was purpose-built to solve."*

## [AI Gateways](../day3/09-ai-gateways.md)

- **An AI Gateway is what happens when you realize LLM and agent traffic doesn't behave like normal API traffic at all — it's metered in tokens, not requests; it streams; it can be redirected across multiple providers; and a "request" might actually be an autonomous tool call, not a human-initiated action.**
- *"A traditional gateway asks 'is this request allowed?' An AI gateway has to ask that too, plus 'how many tokens is this going to cost,' 'which provider should actually handle this,' and — for agentic traffic — 'is this a model just talking, or a model about to take an action?'"*

## [API versioning strategies](../day3/10-api-versioning-strategies.md)

- **Versioning exists because you can't force every consumer to upgrade the moment you change something — the whole discipline is about giving yourself room to evolve without breaking someone else's already-working integration.**
- *"URI versioning is the one your least sophisticated partner integration team will get right on the first try. Content negotiation is the one a REST purist will insist is 'correct.' Neither is wrong — they're solving for different audiences."*
- *"If an existing, unmodified client would break without any code changes on their end, it's a breaking change. If they'd just never notice the addition, it isn't."*

## [Kong vs 3scale (and the wider landscape)](../day3/11-kong-vs-3scale-competitive-landscape.md)

- **Kong and 3scale actually share the same technical lineage — both are Nginx-based gateways underneath — which means the real differentiators aren't "which one is technically better," they're extensibility model, ecosystem positioning, and which use case each platform was originally built to solve.**
- *"Kong grew up as a gateway that became a platform. 3scale grew up as an API-as-a-product platform that happened to need a gateway underneath it. Same Nginx foundation, very different center of gravity."*

## [Day 3 interview Q&A drill](../day3/12-interview-qa.md)

- *"Same N² math as Day 2's integration hub, applied to client-facing traffic."*
- *"Code-first documents what you built. Contract-first designs what consumers need, and holds implementation accountable to it."*
- *"The control plane is never in the request path — same principle as a service mesh's control plane, or Kubernetes' control plane not sitting in pod-to-pod traffic."*
- *"Hybrid mode is DB-less applied specifically to data planes, with a separate control plane still holding the database."*
- *"Multi-tenant control plane, single-tenant data planes — shared governance, isolated traffic."*
- *"A more specific scope silently overriding a broader one is one of the most common real sources of confusion."*
- *"A client can legally burst double the limit right at the window edge."*
- *"It's the only algorithm here that treats bursting as a designed-for behavior, not an edge case."*
- *"With N nodes each counting independently, a client can get N times the intended limit."*
- *"OAuth2 answers 'can this app do this on my behalf?' OIDC answers 'who is this person, actually?'"*
- *"The access token never touches the browser in Authorization Code — it does, directly, in Implicit."*
- *"Never trust what the token claims about itself."*
- *"It turns silent, indefinite token theft into a detectable event."*
- *"mTLS proves which machine — not which user."*
- *"Two tokens, because they answer two different questions."*
- *"Both can technically handle either direction — the real distinction is purpose, not traffic direction."*
- *"Same N² problem from Day 2, reused — now applied to internal traffic instead of external."*
- *"The sidecar is just another container sharing the same pod network namespace the pause container already set up."*
- *"LLM cost is measured in tokens, not requests — a request-count limiter doesn't actually protect your budget."*
- *"MCP alone has real security gaps; an AI Gateway can add the governance layer it doesn't guarantee on its own."*
- *"If an existing, unmodified client would break without any changes on their end, it's breaking. If they'd simply never notice, it isn't."*
- *"URI versioning is the one your least sophisticated partner integration team will get right on the first try."*
- *"Kong grew up as a gateway that became a platform. 3scale grew up as an API-as-a-product platform that happened to need a gateway underneath it."*
- *"First-party integration removes real friction a third-party tool can't fully close — acknowledging that honestly is stronger than defending your own product reflexively."*
