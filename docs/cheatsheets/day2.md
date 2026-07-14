# Day 2 hooks — Apache Camel & Enterprise Integration Patterns

Every hook from [Day 2](../day2/index.md), in page order — 54 in total. Each one is a compressed page: if a hook doesn't unfold back into its full answer in your head, that's the page to re-read. Built for the final day-before-interview pass.

## [Integration architecture fundamentals](../day2/01-integration-fundamentals.md)

- **Every integration architecture decision is really one question asked repeatedly: does the sender need to know the outcome right now, or can it move on and find out later?**
- *"Sync is a phone call — you're both on the line, together, right now. Async is a letter — you send it and get back to your day, trusting the queue to deliver it."*
- *"Orchestration is a conductor with a score, telling every musician when to play. Choreography is a group of dancers who've each learned their own steps and react to each other — nobody's holding the master script."*
- *"Point-to-point integration cost grows with the square of your system count. That's not a style preference — it's the actual reason integration platforms got invented."*

## [Camel core architecture](../day2/02-camel-core-architecture.md)

- **CamelContext is the runtime. A Route is a recipe. An Exchange is one order going through the kitchen. A Processor is one cook doing one step. A Component is a type of supplier Camel knows how to talk to.**
- *"The component is the type of shop. The endpoint is a specific shop's exact address, with directions."*
- *"Different DSLs, same underlying route model — Camel doesn't care which syntax you wrote the recipe in, only that the recipe compiles to the same sequence of steps."*

## [Camel vs MuleSoft vs Spring Integration vs traditional ESB](../day2/03-camel-vs-alternatives.md)

- **Traditional ESB is a heavyweight product you deploy integration *into*. Camel is a lightweight library you embed integration logic *inside of* — anywhere from a JBoss Fuse container to a single Kubernetes pod.**
- *"Asking 'Camel or MuleSoft' is a bit like asking 'a hammer or a fully-equipped workshop.' Camel is the tool. MuleSoft bundles the tool with a workshop, a catalog, and a foreman — at a price and with lock-in that comes with that."*

## [The EIP catalog, deeply — which pattern solves which problem](../day2/04-eip-catalog.md)

- **Every EIP answers one specific question about a message's journey: which way does it go, does it get copied, does it get split apart, or does it get put back together?**
- *"A Content-Based Router has many doors. A Message Filter has exactly one door and a trash can."*
- *"Recipient List picks everyone at once from a phone book. Dynamic Router asks 'where next?' after every single stop. Routing Slip already has the itinerary printed and stapled to the message before it leaves."*
- *"Wire Tap is exactly what it sounds like — a phone tap. The call proceeds normally; someone else just gets a copy, silently."*
- *"Claim Check is a coat check counter. You don't carry your coat through the whole party — you carry a numbered ticket, and collect the coat when you actually need it."*

## [Error handling & transactions](../day2/05-error-handling-transactions.md)

- **Camel gives you three separate levers for failure: retry it (redelivery), route it somewhere else on failure (Dead Letter Channel), or wrap it in a real transaction so it either fully happens or fully doesn't (transacted routes / XA).**
- *"Fixed-delay retries are like knocking on a stuck door at the exact same rhythm forever. Exponential backoff is knowing to wait longer each time — because if the door didn't open at 1 second, hammering it every 1 second isn't going to help, and might make things worse."*
- *"Two-phase commit is a coordinator asking 'is everyone ready?' before saying 'go' — nobody commits until everybody has already promised they can."*
- *"XA guarantees both happen together, at the cost of speed and tight coupling between resources. The outbox pattern accepts a tiny delay in exchange for much better throughput and no distributed transaction at all."*

## [Reliability & exactly-once delivery](../day2/06-reliability-exactly-once.md)

- **True exactly-once delivery across independent systems doesn't really exist. What you can actually build is at-least-once delivery plus idempotent processing — which behaves like exactly-once from the outside.**
- *"You can't make the network promise exactly-once. You can make your processing not care if it received the same thing twice — and that's what actually solves the problem."*

## [Performance & scalability](../day2/07-performance-scalability.md)

- **Camel is single-threaded per exchange by default. Every performance question is really asking: where, deliberately, did you introduce concurrency or streaming — and why there specifically?**
- *"`direct` is handing something to someone standing right next to you, and waiting. `seda` is dropping it in an internal mailbox and walking away. `vm` is the same mailbox, but shared between two separate departments in the same building."*
- *"Backpressure is the consumer saying 'slow down' instead of the producer finding out the hard way that the consumer fell over."*

## [Testing Camel routes](../day2/08-testing-camel-routes.md)

- **Camel's testing story is built around one idea: swap real endpoints for `mock:` endpoints without touching your actual route logic, so you test the routing behavior in isolation from the systems it talks to.**
- *"`Thread.sleep()` in a test is a guess dressed up as a wait. `NotifyBuilder` waits for the actual thing you care about to actually happen."*

## [Camel K / Camel Quarkus / cloud-native Camel](../day2/09-camel-k-quarkus-cloud-native.md)

- **JBoss Fuse deployed Camel as one long-running JVM hosting many routes together. Camel Quarkus and Camel K each go further in the same direction: smaller, faster-starting, more disposable units — right down to a single route as a single Kubernetes-native deployment.**
- *"A traditional JVM warms up like a car engine on a cold morning. A GraalVM native image starts like flipping a light switch — because most of the 'startup work' already happened at build time, not runtime."**
- *"Camel K is Day 1's Operator pattern, applied specifically to integration routes — you declare 'this route should exist,' and a controller reconciles the cluster toward that, exactly like the `KafkaCluster` custom resource example from Day 1's OpenShift material."*

## [B2B & legacy integration standards](../day2/10-b2b-legacy-integration.md)

- **SOAP, eBXML, and EDI all exist for the same underlying reason: formal, strictly-contracted B2B exchange between organizations that don't trust each other's systems enough to integrate loosely — and that formality is still very much alive wherever real regulatory or financial stakes are involved.**
- *"REST hands you flexibility and trusts you to be disciplined. SOAP hands you a formal contract and enforces the discipline for you — which is exactly why regulated, cross-organization B2B integration still reaches for it."*
- *"EDI isn't old because nobody modernized it. It's old because it's a decades-deep network of trading-partner agreements, and untangling that network is a much bigger project than any one company's integration platform."*
- *"DataPower is a bouncer and a translator standing at the door, checking IDs (schema validation, WS-Security) and converting the conversation into a language the house understands (XSLT), before anyone even gets to the actual application logic inside."*

## [Day 2 interview Q&A drill](../day2/11-interview-qa.md)

- *"Does the sender need the answer right now, or can it move on and find out later?"*
- *"Point-to-point grows with the square of your system count — that's the whole justification."*
- *"CamelContext is the runtime, Route is the recipe, Exchange is one order going through the kitchen."*
- *"It's a hammer vs a fully-equipped workshop."*
- *"Recipient List picks everyone at once from a phone book. Dynamic Router asks 'where next?' after every stop. Routing Slip already has the itinerary printed before it leaves."*
- *"Streaming parser plus streaming Splitter plus Claim Check — never hold the whole thing at once."*
- *"Redelivery is the retry attempts; Dead Letter Channel is where a message goes once retries run out."*
- *"XA guarantees both happen together at the cost of speed and coupling. The outbox pattern accepts a small delay for much better throughput."*
- *"You can't make the network promise exactly-once — you can make your processing not care if it got the same thing twice."*
- *"Each pod has its own memory — a duplicate hitting a different replica wouldn't be caught at all."*
- *"Kafka's EOS is scoped to Kafka-to-Kafka — it doesn't automatically extend to a database or JMS broker touched in the same logic."*
- *"`direct` is handing something to someone next to you and waiting. `seda` is an internal mailbox. `vm` is the same mailbox shared across departments in the same building."*
- *"`adviceWith` advises the route at test time — you don't edit production code to make it testable."*
- *"`Thread.sleep()` is a guess dressed up as a wait."*
- *"Fuse is one long-running JVM hosting many routes. Quarkus is a fast, lightweight single-purpose container. Camel K is an Operator that handles the whole deployment for you."*
- *"Camel K is the Operator pattern, applied specifically to integration routes."*
- *"SOAP is a message format. eBXML is a whole framework for how two organizations formally agree to exchange messages at all."*
- *"EDI isn't old because nobody modernized it — it's a decades-deep network of trading-partner agreements."*
- *"DataPower is a bouncer and a translator at the door, before anyone reaches the application logic inside."*
- *"Same N²-vs-2N math as hub-and-spoke topology, applied to data formats."*
- *"XSLT is a declarative, standard, tooling-supported way to express 'this shape becomes that shape' — custom code reinvents that, usually worse."*
- *"If you need 3 simple integrations and you're already deep in Spring, Spring Integration avoids pulling in a much larger framework."*
