# How to answer — technique for follow-up-chain interviews

The interview this course prepares for has a specific shape: an experienced interviewer picks something off your resume, asks an easy opener, and then follows up, and follows up again, until they find the floor of your knowledge. The whole course is built to lower that floor. This page is about the delivery layer — the habits that make the same knowledge score higher.

## The one-line hook

> **Answer in layers, not essays: one sentence that shows you know the shape of the answer, then the mechanism, then the trade-off — and stop. The follow-up is not a threat; it's the interview working as designed. Let them choose the descent.**

## The layered answer — the single most important habit

Under pressure, the instinct is to prove depth by saying everything at once. Resist it. A strong answer to almost any technical question has three layers, delivered in order, in 30-60 seconds:

1. **The hook** — one sentence that frames the whole answer. This is exactly what every drill answer in this course opens with, and it's why: *"NetworkPolicy is just an API object — Kubernetes never enforces it itself, the CNI plugin does."*
2. **The mechanism** — two or three sentences of how it actually works.
3. **The trade-off or limit** — one sentence on where this breaks, what it costs, or when you'd choose differently.

Then **stop and let the interviewer steer**. A layered answer with a clean stop signals confidence and gives them a menu of follow-up directions; a five-minute monologue signals nervousness and usually wanders somewhere unprepared.

**Memorable hook:** *"Give them the trailer, not the film. If they want the film, they'll ask — and every follow-up they choose is one you invited."*

## Trade-off-first language — the senior/junior separator

Day 6's research finding applies to the whole week: interviewers aren't scoring whether you know what a service *is*, they're scoring whether you can state what it *costs*. Practice the verbal pattern: **"I'd use X, accepting Y, because Z matters more here."**

- Junior: "I'd use S3 because it's cheap and durable."
- Senior: "S3 Standard for the first 30 days, then lifecycle to Glacier Deep Archive — accepting a 12-hour retrieval SLA for the cost savings."

If an answer you're about to give has no "accepting..." clause, pause — you're probably about to give the brochure version.

## Scenario questions — clarify before you design

The full five-step framework (clarify → constraints → high-level → deep-dive → failure modes) lives on the [mock interview page](../day7/12-full-mock-interview-review.md) and is drilled on the [system-design scenarios page](../day7/11-system-design-scenarios.md). The one part worth internalizing as *style*: the first thing out of your mouth after a "design X" prompt should be **questions, not architecture**. Scale, latency and availability targets, consistency needs, budget, team skills. Clarifying first is itself a scored signal — it's what distinguishes an architect from someone reciting a reference architecture.

While sketching: lay traffic out left-to-right in layers, **name every arrow** (protocol, sync/async), and for each component say one sentence about **how it fails** — that habit alone quietly demonstrates half of Day 5.

## The honest-unknown protocol

Every interviewer chases bluffs, and a caught bluff costs more than ten honest gaps. When you genuinely don't know:

1. Say so, in one clause, without drama: "I haven't run that in production —"
2. Bridge to the nearest thing you *do* know: "— but it's solving the same problem as X, which I know well, so I'd expect..."
3. Reason out loud from first principles, and label it as reasoning: "...my reasoning, not experience."

This turns a dead end into a demonstration of exactly the skill an SA job actually is: reasoning about unfamiliar systems using adjacent knowledge. Done once, credibly, it also buys trust for every other answer you give.

**Memorable hook:** *"'I don't know, but here's how I'd reason about it' is a strong answer. 'I don't know' delivered as a full stop is a weak one. Pretending to know is the only disqualifying one."*

## Using your project stories without being asked

The course ties every topic to your real history (nbn iB2B, TnD Microservices, Red Hat, Kong) for a reason: an answer that ends with one grounding sentence — "...and that's exactly what our WMQ setup at nbn was doing" — is remembered as *experience*, while the same answer without it is remembered as *study*. One sentence, genuinely earned, never forced: if the connection isn't real, skip it. A fabricated-sounding anchor costs more than none.

## Pace, energy, and the close

- **It's a conversation, not an exam.** Thinking silently for ten seconds feels fine to you and awkward to them; think out loud instead — "there are two ways to read this question..." buys the same time and scores points.
- **Check direction on long answers**: "I can go deeper on the replication mechanics or move on to failure handling — which is more useful?" hands the wheel back gracefully.
- **Have two questions ready to ask them** that show architectural curiosity about *their* world — how the SA role splits time between breadth (many customers) and depth (few), and what a great first-90-days looks like, are both safe and genuine.

## Real-world examples

1. **The hook-first habit is already built into this course** — every drill answer across all seven days opens with a one-phrase hook precisely so that under pressure you can reconstruct the full answer from the phrase. The [cheat sheets](../cheatsheets/index.md) collect them all; the delivery skill is saying the hook, then *unpacking it slowly* rather than reciting the rest from memory.
2. **The "why do I need Kong if I have an ALB?" question from Day 6** is a model of the layered answer: hook ("the ALB balances and routes; the gateway governs"), mechanism (the Day 3 catalog), trade-off ("if none of those needs exist yet, the ALB alone is genuinely fine") — and it's strongest delivered with the calm of someone who fields it from real customers weekly, which you do.
3. **Day 4's exactly-once material is the model for the honest-unknown protocol in reverse** — the course deliberately teaches the *honest* framing ("not true exactly-once delivery — at-least-once plus idempotent consumers") because interviewers set traps around exactly this kind of over-claim. Delivering a limitation unprompted reads as seniority.
