# Fallback & graceful degradation

## The one-line hook

> **A circuit breaker deciding to fail fast is only half the story — fallback is what you actually do with that failure, and graceful degradation is the broader philosophy of deciding, ahead of time, which parts of your system are allowed to disappear under stress.**

## Fallback — the four common shapes

| Fallback type | What it does | Example |
|---|---|---|
| **Cached/stale data** | Serve the last known good response instead of a live one | Spotify serving cached or default recommendations when the personalization service is unavailable |
| **Default/generic response** | A reasonable, non-personalized default | Netflix's Hystrix falling back to a generic "trending" list when the recommendation service is down |
| **Degraded functionality** | Omit a non-critical piece rather than failing the whole request | An e-commerce page loading normally but simply omitting the reviews section if that specific service is down |
| **Queue for later processing** | Accept the request, defer the actual work | Queuing a payment request asynchronously with a "processing" status instead of failing the user-facing call outright |

**Memorable hook:** *"A fallback isn't just 'return an error nicely' — it's an actual product decision about what the user gets instead, and the four shapes above cover almost every real case."*

## Graceful degradation — deciding what's allowed to break

The broader philosophy fallback serves: **partial functionality is almost always better than total failure**, but that only works if you've explicitly decided, ahead of time, which features are genuinely **core** (must work, no matter what) versus **enhancement** (acceptable to degrade or disappear entirely under stress). This is a real product and architecture conversation, not something to improvise mid-incident — deciding during a design review that account balance is core but personalized offers are enhancement is a very different exercise than discovering it for the first time during an actual outage.

## Fallback chains — layered, not single-shot

A fallback can itself have a fallback: try the live data source first, fall back to a cache if that fails, and fall back to a static default if even the cache is unavailable. This layered approach is more resilient than a single fallback path, since it doesn't create one new single point of failure to replace the original one.

**The genuine caution worth naming**: if every fallback across a whole system routes to the **same shared cache or default service**, that shared fallback resource quietly becomes a new, unexamined single point of failure — exactly the kind of subtle risk that shows real production maturity to flag unprompted.

**Memorable hook:** *"A fallback that everything depends on isn't really a fallback anymore — it's just your primary dependency wearing a disguise."*

## How this connects to the circuit breaker page

In practice (Resilience4j's `@CircuitBreaker(fallbackMethod = "...")` annotation is the concrete, idiomatic example), the fallback method is defined **alongside** the circuit breaker itself — the breaker decides *when* to stop calling the real dependency, and the fallback method defines *what happens instead* the moment it does. They're designed and configured together, not as separate, independently-decided concerns.

## Real-world examples

1. **Spotify's cached/default content fallback**, a clean, well-known, specific case study worth being able to name directly rather than describing fallback only in the abstract.
2. **A Kong-fronted API serving a cached response via a caching plugin when the true backend is unavailable**, rather than a hard error — directly relevant to your current role and Day 3's Kong plugin material, and a concrete example of where fallback logic can live at the gateway layer rather than in every individual service.
3. **Designing a Thai banking customer portal so account balance (core) always works even if transaction history or personalized offers (enhancement) degrade under load** — a defensible, business-informed degradation design directly grounded in the regulated financial-services context of your background.
