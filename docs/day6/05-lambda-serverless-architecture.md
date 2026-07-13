# Lambda & serverless architecture

## The one-line hook

> **Lambda isn't one invocation model — it's three genuinely different ones (synchronous, asynchronous, poll-based), and the retry/error-handling behavior is meaningfully different across all three. Treating Lambda as one uniform thing is where real production surprises come from.**

## The three invocation models — worth being precise about

| Model | Trigger examples | Behavior |
|---|---|---|
| **Synchronous** | API Gateway | The caller waits for the function to complete and return a response directly |
| **Asynchronous** | S3 events, EventBridge | Lambda internally queues the event and invokes the function separately; **failures are automatically retried** by Lambda itself |
| **Poll-based** | SQS, DynamoDB Streams, Kinesis | Lambda's own internal polling service reads from the source and invokes the function — batching, retry, and error handling behavior are governed by the source-specific polling configuration, not a single uniform Lambda retry policy |

**Memorable hook:** *"Synchronous is a phone call. Asynchronous is a voicemail Lambda promises to call back about, retrying if the first callback fails. Poll-based is Lambda itself checking a mailbox on a schedule — the retry behavior belongs to the mailbox's own rules, not to Lambda uniformly."*

## The cold start problem, precisely

The **first invocation after a period of inactivity** carries meaningfully higher latency — a concrete, quantified range worth having ready: **roughly 50ms to 2 seconds**, depending on the runtime, allocated memory, and whether the function is attached to a VPC.

### Mitigations, specific and current

- **Provisioned Concurrency** — keep a specified number of execution environments pre-initialized and ready, eliminating cold start latency for that reserved capacity at a continuous cost.
- **Smaller deployment packages** — less code to load means less to initialize.
- **Faster runtimes** — Go, Rust, and Node generally have faster cold-start init times than JVM-based runtimes like Java, which have historically been the worst case.
- **Avoid VPC attachment unless genuinely needed** — Lambda functions attached to a VPC have historically carried extra cold-start overhead (from needing to attach an ENI), though this has improved significantly in recent years — still worth naming as a real, if lessened, consideration.
- **Lambda SnapStart** (Java specifically) — a named, current AWS feature that snapshots an already-initialized execution environment and resumes directly from that snapshot rather than fully re-initializing from scratch, directly targeting the historically worst-case JVM cold start problem.

## Concurrency — the mechanism behind Lambda throttling

Every concurrent invocation of a function gets its **own execution environment**. AWS enforces an **account-level concurrency limit**, and individual functions can be given **reserved concurrency** (guaranteeing capacity, but also capping it) or **provisioned concurrency** (pre-warmed, as above). **Diagnosing a Lambda throttling problem** — a specifically flagged, realistic senior-level question — means distinguishing between a genuine concurrency limit being hit (requests being actively rejected/throttled) versus ordinary cold-start latency (requests succeeding, just slower than expected); the fix for one (raising reserved/account concurrency limits) does nothing for the other (which needs provisioned concurrency or faster initialization instead).

## Distributed tracing for serverless — a direct Day 5 callback

**AWS X-Ray** is AWS's own native distributed tracing service — directly the same trace/span/context-propagation model built from first principles on Day 5's OpenTelemetry page, just AWS's specific implementation of it. A serverless architecture (API Gateway → Lambda → DynamoDB) instrumented with X-Ray gives exactly the same end-to-end request visibility Day 5 covered generically, now grounded in a concrete, AWS-native tool worth naming by name in this interview specifically.

## When Lambda is (and isn't) the right choice — recap and reinforce

Lambda fits **event-driven, short-duration (under 15 minutes) tasks** — lightweight APIs, automation, microservices reacting to events. It is **not** the right fit for long-running or genuinely stateful workloads (EC2's territory) or workloads needing full custom runtime control (the container territory from the previous page).

## Real-world examples

1. **AWS X-Ray as the concrete, AWS-native extension of Day 5's distributed tracing material** — a strong, specific answer if asked how you'd add observability to a serverless architecture, grounding an already-covered concept in the actual tool this interview cares about.
2. **Choosing SQS-triggered (poll-based) Lambda for processing a backlog of events** versus API-Gateway-triggered (synchronous) Lambda for a real-time endpoint — a precise, invocation-model-aware architecture decision, not a generic "Lambda handles both fine" answer.
3. **Diagnosing a production Lambda throttling incident** — walking through the distinction between a genuine concurrency limit being exceeded versus ordinary cold-start latency, and applying the correct fix (reserved/account concurrency vs. provisioned concurrency) for whichever it actually is — directly matching a question research flagged as realistic and senior-level.
