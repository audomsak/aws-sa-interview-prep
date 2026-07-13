# AI Gateways

This is the newest, most current material in the whole course — genuinely useful given the AWS job description explicitly calls out AI Agents as a preferred qualification, and your own employer is actively building in exactly this space.

## The one-line hook

> **An AI Gateway is what happens when you realize LLM and agent traffic doesn't behave like normal API traffic at all — it's metered in tokens, not requests; it streams; it can be redirected across multiple providers; and a "request" might actually be an autonomous tool call, not a human-initiated action.**

## Why traditional API gateway thinking doesn't fully cover AI traffic

| Traditional API traffic assumption | Why it breaks for LLM/agent traffic |
|---|---|
| Cost/usage is measured in request count | LLM cost is measured in **tokens** consumed — a rate limiter counting requests, not tokens, doesn't actually protect your budget |
| One backend serves a given route | Teams commonly need to route across **multiple LLM providers** (OpenAI, Anthropic, AWS Bedrock) for cost, latency, or failover reasons — needing a unified interface, not N separate integrations |
| Identical requests are wasteful duplicates worth caching exactly | LLM prompts that are *semantically* similar but not byte-identical waste just as much cost re-computing an answer — exact-match caching misses this entirely |
| A response is either fully ready or an error | LLM responses commonly **stream** token-by-token, a fundamentally different traffic shape than a typical request/response cycle |
| The caller is a known, deterministic client | With AI **agents**, the caller may be an autonomous system making its own decisions about which tool to call next — the gateway needs to distinguish "this is a model invocation" from "this is a tool call the model decided to make," since those carry very different security implications |

## What an AI Gateway adds on top of a traditional gateway

- **Token-based rate limiting** — quota enforcement based on actual token consumption, not request count, directly extending this morning's rate limiting algorithms (token bucket, in particular, generalizes naturally from "API call tokens" to "LLM tokens" — same underlying algorithm, different unit being metered).
- **Multi-provider routing and failover** — a unified interface in front of multiple LLM providers, so an application can fail over from one provider to another (or route by cost/latency) without the application itself needing provider-specific integration code.
- **Semantic caching** — caching based on the *meaning* of a prompt rather than its exact text, catching cost-saving cache hits that exact-match caching would completely miss.
- **Prompt-level security controls** — guarding against prompt injection and similar AI-specific attack patterns, a category of concern that simply didn't exist for traditional deterministic APIs.
- **AI-specific observability** — tracking token usage, cost per request, and model-level latency/performance, rather than just generic HTTP request metrics.

**Memorable hook:** *"A traditional gateway asks 'is this request allowed?' An AI gateway has to ask that too, plus 'how many tokens is this going to cost,' 'which provider should actually handle this,' and — for agentic traffic — 'is this a model just talking, or a model about to take an action?'"*

## The Model Context Protocol (MCP) angle — a genuinely current, relevant detail

Research into this space surfaced a specific, current concern worth being able to name: **MCP (Model Context Protocol) itself has real security gaps** when deployed without additional governance — an AI Gateway sitting in front of MCP servers and tool calls can add the authentication, authorization, and visibility that the protocol alone doesn't guarantee. Being able to distinguish a plain model invocation from an autonomous tool call — and apply different security policy to each — is exactly the "context-aware, AI-native gateway" capability the newest generation of tools (including Kong's own direction) is being built around.

## Real-world examples

1. **Kong's own AI Gateway capability, directly relevant to your current role.** Kong is actively positioning itself in this exact space — being fluent here isn't abstract industry awareness, it's genuinely current product knowledge for your day-to-day work, and a strong, specific answer if asked about where API management is heading.
2. **A direct, substantive bridge to the AWS job description's AI Agents callout.** Rather than an empty "I've read about AI agents," this page gives you real, defensible technical material — token-based rate limiting, multi-provider routing, MCP governance — that connects your actual current-role expertise to a stated gap area in the role you're interviewing for.
3. **Token bucket rate limiting, reused from this morning's material, applied to LLM cost control** — a clean, same-day cross-reference showing the underlying algorithm generalizes cleanly from "requests per second" to "tokens per minute," just with a different unit being metered.
