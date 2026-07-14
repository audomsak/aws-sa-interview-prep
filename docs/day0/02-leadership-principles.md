# Amazon Leadership Principles & your story bank

Even in a "functional" AWS interview, every interviewer is collecting Leadership Principle (LP) data points alongside the technical signal — behavioral questions routinely take a third to half of an Amazon loop, and a technically perfect performance with empty behavioral answers fails it. This page maps the LPs to *your* actual project history and gives you a story bank to personalize. The scaffolds below are anchored to real projects from your resume, but **the specifics — numbers, dates, names, outcomes — must come from your memory**. Fill them in days before the interview; a scaffold recited without specifics reads as exactly what it is.

## The one-line hook

> **Amazon interviewers don't score adjectives, they score evidence: one specific situation, what *you* (not "we") did, and a measurable result. Prepare 6 real stories well and you can cover all 16 principles — don't prepare 16 stories badly.**

## STAR, tuned for this interview

**S**ituation (2 sentences of context) → **T**ask (what was *your* responsibility) → **A**ction (what *you* did — the longest part, first person singular) → **R**esult (quantified, plus what you learned). Two minutes total, then stop — the layered-answer rule from the [previous page](01-how-to-answer.md) applies to behavioral answers too, and follow-ups here drill your story's details, which is why inflating anything is fatal: the interviewer's follow-up ("what exactly did the latency drop to?") lands precisely where an embellishment lives.

**Memorable hook:** *"'We' is invisible to an Amazon interviewer — they're hiring you, not your old team. Every Action sentence should survive the question: 'and what did YOU specifically do?'"*

## The 16 principles, one line each

| Principle | The question behind it |
|---|---|
| Customer Obsession | Did you start from the customer's problem and work backwards? |
| Ownership | Did you act beyond your job description, long-term over short-term? |
| Invent and Simplify | Did you find the simpler path, not just the clever one? |
| Are Right, A Lot | Do you have good judgment — and seek disconfirming views? |
| Learn and Be Curious | Do you learn things nobody required you to learn? |
| Hire and Develop the Best | Have you raised the bar in others? |
| Insist on the Highest Standards | Did you refuse to ship the not-good-enough thing? |
| Think Big | Did you propose something beyond the asked-for scope? |
| Bias for Action | Did you make a reversible decision quickly rather than study it? |
| Frugality | Did you accomplish more with less? |
| Earn Trust | Did you admit a mistake, deliver hard truths, listen? |
| Dive Deep | Did you go to the actual data/logs/root cause yourself? |
| Have Backbone; Disagree and Commit | Did you push back with conviction — then commit fully once decided? |
| Deliver Results | Did the thing actually ship and work, despite obstacles? |
| Strive to Be Earth's Best Employer | Did you make the team safer, more inclusive, more effective? |
| Success and Scale Bring Broad Responsibility | Did you weigh the wider impact of a decision? |

For an SA role, the most heavily probed are **Customer Obsession, Dive Deep, Earn Trust, Have Backbone, Ownership, and Learn and Be Curious** — an SA's whole job is customer trust plus technical depth. Weight your preparation accordingly.

## Your story bank — six scaffolds to personalize

Each scaffold names the LPs it can serve. In the interview, one well-told story flexes across several principles depending on which part you emphasize.

**Story 1 — The production incident you owned end-to-end (nbn iB2B, Marlo).** *Ownership, Dive Deep, Deliver Results.* Six-plus years on a national-scale B2B integration platform guarantees you lived through at least one serious production incident — a stuck queue, a poison message storm, a partner integration failing at the worst time. Fill in: the incident, how you traced it to root cause (the deeper and more hands-on, the better — logs, WMQ queue depths, Camel route traces), what you changed so it couldn't recur, and the measurable outcome. The follow-up will be "what was the actual root cause?" — this story only works at full technical specificity, which is exactly why it doubles as Dive Deep.

**Story 2 — The monolith decomposition (TnD Microservices).** *Invent and Simplify, Think Big, Deliver Results.* Decomposing a Test & Diagnostics monolith into Kafka/Kubernetes microservices on AWS. Fill in: what was wrong with the monolith (deployment coupling? scaling?), how the decomposition was scoped (which service first, and why — emphasize simplification choices over clever ones), and a before/after result (deployment frequency, incident isolation, scaling cost). Emphasize the *sequencing judgment* — that's the Think Big + pragmatism combination Amazon scores.

**Story 3 — The time you recommended against the bigger sale (Red Hat or Kong presales).** *Earn Trust, Customer Obsession, Frugality.* Any presales SA has a moment of telling a customer they don't need everything being sold — the honest "an ALB alone is genuinely fine until your APIs become products" conversation from Day 6, or right-sizing an OpenShift footprint against the account team's instincts. Fill in: the customer situation, what you recommended *against* your own short-term interest, and what it did for the relationship long-term. This is the single most SA-shaped story in the bank — have it polished.

**Story 4 — The disagreement you lost (or won) properly (any project).** *Have Backbone; Disagree and Commit, Are Right A Lot.* Fill in a real architectural disagreement: choreography vs orchestration on an integration flow, Kafka vs a simpler queue, refactor-during-migration vs after. What matters to the interviewer: you argued from data with conviction, *and* — if overruled — you committed genuinely, without sabotage-by-lukewarmness. If your version ends with "and I was right after all," pick a different ending: the disagree-and-commit half is the rarer, more valuable signal.

**Story 5 — The thing you learned because nobody stopped you (career-spanning).** *Learn and Be Curious, Hire and Develop the Best.* The arc from Java integration developer → Red Hat platform SA → Kong API-management SA *is* this story: each jump required self-taught depth (containers/Kubernetes, then the whole API management domain), plus the AWS SAA certification pursued on your own initiative. Fill in: one concrete learning push, what triggered it, how you went beyond the minimum, and — for the Develop-the-Best flavor — an instance of pulling colleagues or customers up the same curve (workshops, PoC enablement, internal training).

**Story 6 — Delivering under a hard constraint (nbn iB2B or a customer PoC).** *Bias for Action, Insist on the Highest Standards, Deliver Results.* A regulatory deadline, a partner cutover date, a PoC with an immovable demo date. Fill in: the constraint, the corner you *refused* to cut (standards), the corner you *chose* to cut with a plan to repay it (bias for action — reversible decisions made fast), and the shipped result. The deliberate contrast between those two corners is the story's spine.

## Mapping table — walk in with this in your head

| If asked about... | Lead with |
|---|---|
| Customer Obsession / Earn Trust | Story 3, backup Story 6 |
| Ownership / Dive Deep | Story 1 |
| Invent & Simplify / Think Big | Story 2 |
| Backbone / Disagree & Commit | Story 4 |
| Learn & Be Curious | Story 5 |
| Deliver Results / Bias for Action | Story 6, backup Story 1 or 2 |
| "Tell me about a failure" | Story 4 told as the mistake + what changed in your judgment — never claim you've had no failures |

## Real-world examples

1. **"Tell me about a time you disagreed with a decision" is near-certain in an Amazon loop** — and the trap is telling a story where you were right and everyone eventually agreed. The stronger, rarer answer is the one where you were overruled and committed anyway — prepare Story 4 in that direction deliberately.
2. **The behavioral follow-up chain works exactly like the technical one** — "what did you do?" → "what did the customer say?" → "what was the number?" — which is why each scaffold above insists on filled-in specifics. A story you can't survive three follow-ups on isn't ready.
3. **LP answers can double as technical evidence**: Story 1 told properly demonstrates Day 4's DLQ/poison-pill material in passing; Story 2 demonstrates Day 6's migration sequencing. When a behavioral story naturally lands on course material, one grounding sentence of technical specificity raises both scores at once.
