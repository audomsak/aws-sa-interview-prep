# API versioning strategies

## The one-line hook

> **Versioning exists because you can't force every consumer to upgrade the moment you change something — the whole discipline is about giving yourself room to evolve without breaking someone else's already-working integration.**

## The four common strategies, compared

| Strategy | How it looks | Tradeoff |
|---|---|---|
| **URI versioning** | `/v1/orders`, `/v2/orders` | Simple, visible, trivially cacheable and testable in a browser — but "pollutes" the URI and technically means the same resource has multiple different identifiers across versions, which REST purists dislike |
| **Header versioning** | Custom header, e.g. `X-API-Version: 2` | Keeps URIs clean and stable — but less discoverable (you can't see the version just by looking at a URL) and harder to casually test |
| **Content negotiation / media type versioning** | `Accept: application/vnd.company.v2+json` | The most technically "correct" REST approach — versioning is genuinely a property of representation, not identity — but meaningfully more complex for client developers to implement correctly |
| **Query parameter versioning** | `/orders?version=2` | Simple to add, but easy to forget (unlike a header, it's easy for a client to accidentally omit) and can complicate caching if not handled carefully |

**Memorable hook:** *"URI versioning is the one your least sophisticated partner integration team will get right on the first try. Content negotiation is the one a REST purist will insist is 'correct.' Neither is wrong — they're solving for different audiences."*

## What actually requires a new version, versus what doesn't

The real engineering discipline isn't picking a versioning *style* — it's knowing which changes are safe within an existing version and which genuinely require a new one:

| Safe within the same version (additive) | Requires a new version (breaking) |
|---|---|
| Adding a new optional field | Removing a field |
| Adding a new endpoint | Renaming a field |
| Adding a new optional query parameter | Changing a field's data type |
| Relaxing a validation rule | Making a previously optional field required |
| | Changing existing response structure/nesting |

**Memorable hook:** *"If an existing, unmodified client would break without any code changes on their end, it's a breaking change. If they'd just never notice the addition, it isn't."*

## Deprecation — the part people skip, and shouldn't

Shipping a new version doesn't retire the old one — a responsible deprecation process needs:

- **Advance notice**, communicated well before removal, not discovered by consumers via a sudden failure
- **`Deprecation` and `Sunset` HTTP headers** — machine-readable signals a client (or its monitoring) can detect automatically, rather than relying purely on documentation or email announcements
- **Usage monitoring** — knowing *who* is still calling the old version before retiring it, so you're not flying blind about the real-world blast radius of a sunset date

## The gateway's role in all of this

An API Gateway is the natural enforcement point for versioning in practice: routing `/v1/*` and `/v2/*` to different backend services (or the same service with version-aware logic), injecting `Deprecation`/`Sunset` headers automatically for old-version traffic, and — genuinely useful operationally — giving you a single place to actually *measure* how much traffic is still hitting a version you're trying to retire, rather than each backend service tracking that independently.

## Real-world examples

1. **The nbn Australia iB2B Business Services exposed to external Access Seekers.** Day 2 covered the Collaboration Protocol Agreement (CPA) as the formal contract governing exactly how a specific partner integrates — a CPA is, functionally, a natural place to pin a partner to a specific API version, since external partners genuinely cannot be forced to upgrade on your internal schedule the way an internal team might be.
2. **Kong route-based versioning for a customer migrating consumers from a legacy to a new API version**, directly relevant to your current CSM role — a realistic scenario of both versions needing to coexist safely while usage naturally migrates, with the gateway giving visibility into exactly how much traffic remains on the old version before a sunset date is safe to commit to.
3. **Choosing URI versioning for external partner-facing APIs versus header-based versioning for internal, more sophisticated consuming teams** — a genuine, defensible "it depends on who's actually consuming this" architecture decision, rather than a single dogmatic versioning policy applied everywhere regardless of audience.
