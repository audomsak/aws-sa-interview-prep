# Testing Camel routes

Testing is its own recurring interview category, separate from routing logic — and it's a genuinely good place to demonstrate production maturity rather than just tutorial-level knowledge.

## The one-line hook

> **Camel's testing story is built around one idea: swap real endpoints for `mock:` endpoints without touching your actual route logic, so you test the routing behavior in isolation from the systems it talks to.**

## The `mock:` component and setting expectations

Camel's `camel-test` module (built on JUnit) lets you replace a real endpoint's URI with a `mock:` equivalent purely for testing, then set **expectations** on that mock before running the route:

```java
mockEndpoint.expectedMessageCount(3);
mockEndpoint.expectedBodiesReceived("orderA", "orderB", "orderC");
mockEndpoint.assertIsSatisfied();
```

This lets a test assert not just "did it run without throwing," but precisely what was sent, how many times, and in some cases in what order — genuinely useful for verifying EIP behavior like a Content-Based Router sending the right message down the right branch.

## `adviceWith` — modifying a route for a test without touching production code

**`adviceWith`** is the mechanism that makes mock-based testing practical: it lets a test intercept and modify an *existing* production route definition at test time — replacing one real endpoint (say, a live database call) with a `mock:` endpoint — while leaving the rest of the route's actual logic completely untouched. This is the honest answer to "how do you test a route without changing the route" — you don't edit production code for testing; you advise the route at test time instead.

## `NotifyBuilder` — testing asynchronous routes without flaky sleeps

For routes involving async components (`seda`, `vm`, or any InOnly flow), a test can't just assert immediately after calling the route — the work may not have finished yet. The naive (and fragile) fix is `Thread.sleep()` and hope it was long enough. **`NotifyBuilder`** is Camel's real answer: it lets a test wait for a specific condition — "route X has completed processing 5 exchanges" — up to a timeout, and proceed the instant that condition is actually met, rather than sleeping a fixed, guessed duration that's either wastefully long or occasionally too short.

**Memorable hook:** *"`Thread.sleep()` in a test is a guess dressed up as a wait. `NotifyBuilder` waits for the actual thing you care about to actually happen."*

## Unit testing vs integration testing — a real strategy, not just a phrase

| | Unit test | Integration test |
|---|---|---|
| External endpoints | All mocked (`mock:` everywhere) | Real, or realistically simulated |
| Speed | Fast, runs on every build | Slower, often a separate CI stage |
| What it proves | The routing/EIP logic itself is correct | The route actually works against real (or containerized) dependencies |
| Typical tooling | `camel-test`, `mock:`, `adviceWith` | **Testcontainers** — spinning up real Kafka, Postgres, or an MQ broker in Docker for the duration of the test |

**The direct connection back to Day 1:** Testcontainers works by launching real, disposable containers for the test's lifetime — the exact same namespace/cgroup-isolated processes covered on Day 1's "container from scratch" page, just orchestrated by a test library instead of `kubectl` or `docker run` directly. Being able to name that connection explicitly is a strong signal of genuinely connected understanding rather than memorized, siloed facts.

## Real-world examples

1. **Testing iB2B routes against mocked WMQ/AMQ endpoints** instead of requiring a live legacy MQ connection during CI — a direct, realistic answer for how a Marlo-era integration test suite would actually have needed to work, given the WMQ/AMQ components in that project's real tech stack.
2. **Using Testcontainers to spin up a real Kafka broker for integration-testing the TnD Microservices platform's Kafka-based flows**, giving much higher confidence than mocking Kafka's actual partition/consumer-group behavior, which is genuinely hard to fake convincingly with a mock.
3. **Replacing a flaky `Thread.sleep()`-based test for an async `seda` route with `NotifyBuilder`** — a concrete, specific answer to "how do you avoid flaky tests in an async integration codebase," a very realistic follow-up question after any discussion of Camel's async components.
