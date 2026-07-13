# Service mesh resilience patterns

Day 3 introduced the service mesh's sidecar and control-plane/data-plane architecture in the context of comparing it to an API gateway. This page connects that material directly to today's entire theme: where do circuit breaker, bulkhead, retry, and timeout actually *live*, and what changes when you move them out of application code and into infrastructure?

## The one-line hook

> **Every resilience pattern covered today can be implemented in application code, per service — or configured once, declaratively, at the mesh level, and enforced consistently across every service regardless of what language it's written in.**

## The core value proposition

Implementing circuit breakers, retries, timeouts, and bulkheads correctly in **every individual service's application code** (Resilience4j-style, from earlier today) means every team, in every language and framework across a polyglot platform, has to remember to configure it — and configure it *consistently*. A service mesh moves these concerns **out of application code and into the sidecar proxy** — configured once, declaratively, and enforced identically no matter what the underlying service is written in.

## Concrete Istio/Envoy resilience configuration

| Application-level pattern | Mesh-level equivalent |
|---|---|
| Bulkhead (thread pool / connection pool isolation) | `DestinationRule` connection pool settings — max connections, max pending requests, configured per destination |
| Circuit breaker | **Outlier detection** — ejecting an unhealthy endpoint from the load-balancing pool after a threshold of consecutive failures |
| Retry with backoff | Declarative retry configuration on a `VirtualService` — attempt count, per-try timeout, and which conditions actually trigger a retry |
| Timeout | Declarative timeout configuration, also at the `VirtualService` level |

## Outlier detection — a precise, worth-knowing distinction from application-level circuit breaking

**Application-level circuit breaking** is typically binary for a single dependency: the circuit is open or closed for calls to "the payment service" as a whole. **Mesh-level outlier detection** operates differently and more granularly: with multiple replicas of the same service behind a load balancer, outlier detection **ejects only the specific unhealthy replica** from the pool, while traffic continues flowing normally to the other healthy replicas of that same service.

**Memorable hook:** *"Application-level circuit breaking asks 'is this dependency healthy, yes or no?' Mesh-level outlier detection asks 'which specific replica of this dependency is unhealthy, and can I just stop sending it traffic while using the others normally?' — a more precise question, answerable only because the mesh sees individual endpoints, not just a service name."*

## The genuine tradeoff, stated honestly

Mesh-level resilience buys **polyglot consistency** and meaningfully **less application code to write and maintain** — at the cost of an additional infrastructure layer to operate, and **less fine-grained, business-aware control** than well-instrumented application code can offer. A mesh can retry a failed call or reroute around an unhealthy endpoint, but it fundamentally **cannot return a custom, business-specific fallback response** the way application-level fallback code from earlier today can — the mesh doesn't know your domain, only your network behavior.

**Memorable hook:** *"The mesh is excellent at 'try again' and 'stop sending traffic there' — it has no idea what a sensible fallback order confirmation actually looks like. That part still has to live in your application."*

## Where they genuinely combine — not either/or

Many real production systems use **both deliberately**: mesh-level circuit breaking, retries, and timeouts as a **consistent baseline safety net** enforced identically across every service regardless of language, **plus** application-level fallback logic (Resilience4j or equivalent) specifically for the business-specific, custom degradation behavior — a cached response, a queued retry, a generic default — that the mesh genuinely cannot provide on its own.

## Real-world examples

1. **A polyglot platform mixing Java and Node.js** (directly matching the real tech stack across your nbn TnD Microservices and Digital Products projects) benefiting from mesh-level resilience configuration that applies consistently across both language stacks — without needing a separate, correctly-configured Resilience4j-equivalent library maintained in each language.
2. **Kong Mesh**, directly recalling Day 3's material and your current employer's product line, providing exactly this Envoy-based resilience layer as part of Kong's own broader platform — a strong, concrete, current product tie-in.
3. **Combining Istio-level outlier detection as a baseline safety net with application-level Resilience4j fallback logic returning a specific, business-meaningful cached response** — the honest, sophisticated "use both, for different reasons" answer, rather than treating mesh-level and application-level resilience as competing alternatives.
