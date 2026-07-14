# Day 5 hooks — Microservices & Distributed Systems Resilience

Every hook from [Day 5](../day5/index.md), in page order — 56 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [CAP theorem & data consistency patterns](../day5/01-cap-theorem-data-consistency.md)

- **CAP theorem isn't a permanent architectural choice — it's about what you do the instant a network partition actually happens. Outside of a partition, you can have both consistency and availability; CAP only forces a choice during the failure itself.**
- *"CAP isn't 'pick 2 of 3' forever — it's 'partition tolerance is mandatory, so pick your fallback between C and A for the moments things actually break.'"*
- *"W + R > N is just pigeonhole logic — if your write touched more than half the replicas and your read checks more than half, they're mathematically guaranteed to share at least one replica in common."*

## [Consensus algorithms — Raft/Paxos deep dive](../day5/02-consensus-algorithms-raft-paxos.md)

- **Consensus is the problem of getting a group of unreliable, distributed nodes to agree on a single value — and it's the actual mechanism underneath leader election, replicated logs, and distributed locks, whether or not you ever touch the algorithm directly.**
- *"Raft's whole design philosophy is 'a human should be able to draw this state machine from memory' — which is exactly the three-box diagram above."*
- *"KRaft uses Raft for agreeing on cluster metadata. ISR-based partition leader election is a separate, Kafka-specific mechanism for agreeing on which replica leads a given partition. Same underlying quorum philosophy, two genuinely different mechanisms — don't flatten them into one."*

## [Circuit breaker pattern deep dive](../day5/03-circuit-breaker-deep-dive.md)

- **A circuit breaker is a deliberate, explicit choice to trade consistency for availability — the direct, concrete application of the CAP/PACELC tradeoff from earlier today, expressed as a single, well-known pattern.**
- *"Closed is normal traffic. Open is 'don't even bother asking, I already know the answer is no.' Half-Open is cautiously knocking on the door again after a while to see if anyone's home yet."*

## [Bulkhead, timeout & retry patterns](../day5/04-bulkhead-timeout-retry.md)

- **Timeout bounds a call. Retry handles a transient failure within that bound. Bulkhead limits how much of the system one failing dependency can drag down. Circuit breaker is what stops the whole cycle once failures stop being transient.**
- *"Thread pool isolation gives each dependency its own lifeboat. Semaphore isolation just limits how many people can be near the railing at once — cheaper, but not the same guarantee."*
- *"Backoff without jitter is a synchronized crowd all trying the door again at exactly the same second. Jitter is what breaks that synchronization apart."*
- *"These aren't four competing patterns you choose between — they're four layers that wrap each other, each one handling a different failure mode the others don't cover."*

## [Load shedding & backpressure](../day5/05-load-shedding-backpressure.md)

- **Circuit breakers and bulkheads protect a service from a failing dependency it's calling OUT to. Load shedding protects a service from too much traffic coming IN — a genuinely different problem, at a different point in the request path.**
- *"Circuit breaker protects you from someone else's problem. Load shedding protects everyone else from yours, when you're the one about to fall over."*
- *"Kafka never needed a special backpressure feature bolted on — pulling instead of pushing was backpressure, built into the model from day one."*
- *"Everything from the previous page is about being a good, resilient client. Load shedding and backpressure are about being a good, resilient server — two different halves of the same overall resilience story."*

## [Fallback & graceful degradation](../day5/06-fallback-graceful-degradation.md)

- **A circuit breaker deciding to fail fast is only half the story — fallback is what you actually do with that failure, and graceful degradation is the broader philosophy of deciding, ahead of time, which parts of your system are allowed to disappear under stress.**
- *"A fallback isn't just 'return an error nicely' — it's an actual product decision about what the user gets instead, and the four shapes above cover almost every real case."*
- *"A fallback that everything depends on isn't really a fallback anymore — it's just your primary dependency wearing a disguise."*

## [Chaos Engineering](../day5/07-chaos-engineering.md)

- **An untested circuit breaker is just code that's never been exercised — chaos engineering is the discipline of proving your resilience patterns actually work, on purpose, before a real outage finds out for you.**
- *"Resilience code you've never actually triggered isn't proven resilience — it's just untested code that happens to be about failure handling."*

## [CQRS, full treatment](../day5/08-cqrs-full-treatment.md)

- **CQRS isn't "use two databases" — it's the recognition that the requirements for changing data and the requirements for reading it are often genuinely different, and forcing one model to serve both means compromising on both.**
- *"A read model isn't cheating by pre-computing the answer — that's the entire point. It's a cache you designed on purpose, not one you were forced into later because a query got slow."*
- *"Knowing when NOT to reach for CQRS is just as strong a signal as knowing how to build it — reflexively applying it everywhere is itself a red flag, not a strength."*

## [Event Sourcing, full treatment](../day5/09-event-sourcing-full-treatment.md)

- **In Event Sourcing, current state isn't stored anywhere — it's computed, on demand, by replaying every event that ever happened to that entity, from the beginning (or from a snapshot forward).**
- *"A snapshot is a checkpoint save — you don't replay a video game from the very first frame every time you load it, you load the last save and play forward from there."*
- *"Versioning tells you an event is stale after the fact. Reordering makes you wait to be sure of the order before acting. Commutativity is the only option where the order never mattered in the first place — but you don't always get to choose that."*

## [Distributed tracing & OpenTelemetry](../day5/10-distributed-tracing-opentelemetry.md)

- **A trace is a tree of spans, held together entirely by context propagation — and the moment that propagation breaks across an async boundary, you don't get an error, you just silently get two disconnected traces instead of one.**
- *"Context propagation doesn't fail loudly — it fails silently. You just end up with two traces that used to be one, and nothing tells you that happened except a trace that inexplicably stops where you expected it to continue."*

## [Service mesh resilience patterns](../day5/11-service-mesh-resilience.md)

- **Every resilience pattern covered today can be implemented in application code, per service — or configured once, declaratively, at the mesh level, and enforced consistently across every service regardless of what language it's written in.**
- *"Application-level circuit breaking asks 'is this dependency healthy, yes or no?' Mesh-level outlier detection asks 'which specific replica of this dependency is unhealthy, and can I just stop sending it traffic while using the others normally?' — a more precise question, answerable only because the mesh sees individual endpoints, not just a service name."*
- *"The mesh is excellent at 'try again' and 'stop sending traffic there' — it has no idea what a sensible fallback order confirmation actually looks like. That part still has to live in your application."*

## [Day 5 interview Q&A drill](../day5/12-interview-qa.md)

- *"CAP only forces a choice during an actual network partition — outside of one, you can have both."*
- *"Even with no partition happening at all, there's still a latency-vs-consistency tradeoff."*
- *"It's pigeonhole logic — if your write and your read both touch more than half the replicas, they're guaranteed to overlap on at least one."*
- *"Getting a group of unreliable, distributed nodes to agree on one single value, even when some fail or messages are delayed."*
- *"A term is Raft's logical clock — any message carrying an outdated term gets automatically rejected."*
- *"KRaft uses Raft for cluster metadata. ISR-based partition leader election is a separate, Kafka-specific mechanism."*
- *"Closed counts failures. Open refuses to even try. Half-Open cautiously tests the water again."*
- *"Failing fast with a fallback is choosing to respond immediately, rather than waiting for what might eventually be a correct answer."*
- *"Thread pool isolation gives each dependency its own lifeboat. Semaphore isolation just caps how many people can be near the railing."*
- *"Backoff without jitter is a synchronized crowd all trying the door again at exactly the same second."*
- *"A system-wide circuit breaker for the retry mechanism itself."*
- *"Circuit breaker protects you from someone else's problem. Load shedding protects everyone else from yours."*
- *"The pull request rate IS the backpressure signal — no separate mechanism needed."*
- *"A fallback is a product decision about what the user gets instead, not just 'return an error nicely.'"*
- *"A fallback that everything depends on isn't really a fallback anymore — it's your primary dependency wearing a disguise."*
- *"An untested circuit breaker is just code that's never been exercised."*
- *"Responsible chaos engineering starts small and expands only with earned confidence."*
- *"It's not cheating by pre-computing the answer — that's the entire point."*
- *"Knowing when NOT to use it is just as strong a signal as knowing how to build it."*
- *"The event stream is an excellent source for read-model projections — but it's not a requirement."*
- *"A snapshot is a checkpoint save — you don't replay from the very first frame every time."*
- *"Versioning tells you an event is stale after the fact. Reordering makes you wait to be sure. Commutativity is the only option where order never mattered."*
- *"Context propagation doesn't fail loudly — you just silently end up with two traces instead of one."*
- *"Application-level asks 'is this dependency healthy, yes or no?' Mesh-level outlier detection asks 'which specific replica is unhealthy?'"*
