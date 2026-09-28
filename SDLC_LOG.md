# SDLC Log — sdlc-ip-api

A narrated log of how this project was actually run: not just what was decided, but why, and which alternatives were rejected. Written so someone else learning SDLC can follow the reasoning.

**Format:** each entry names the phase or sprint, which role was speaking, and ends with the decision reached. Disagreements and revisions are kept, not cleaned up.

---

## Pre-Sprint 0: Discovery

*Ordinary conversation, before any roles.* **Date:** 2026-09-26

**The starting idea was different.** The first proposal was to use an existing API Gateway project to demonstrate MPLS (a label-switching technology used in carrier networks) for visual learners.

**The idea was pushed back on, and why.** API Gateway works at the HTTP layer (layer 7). MPLS works between layers 2 and 3, and AWS doesn't expose it at all. Using API Gateway routing as an *analogy* for MPLS risked teaching a wrong mental model: MPLS labels are swapped hop by hop, while a gateway matches a path once.

**The real goal surfaced.** Clarifying "why" changed the project. The actual goal was a concrete GitHub portfolio piece showing Python and API Gateway skills plus networking understanding. With that goal, API Gateway earns its place as the front door to a Python service, and the networking knowledge lives in the code.

**Three candidate projects:**
1. MPLS label-switching simulator API — most distinctive, most effort
2. **IP toolkit API** — smallest, fastest to finish
3. BGP looking-glass API — real internet data, depends on a third-party provider

**Decision:** all three are wanted; start with the lowest-effort one (2) and run it through the same Agile SDLC roleplay used on an earlier project (Daily Driver).

> **Teaching note:** The most valuable Discovery question was "what are you actually trying to achieve?" The first idea (MPLS for visual learners) and the real goal (a portfolio piece) would have produced very different projects. Many projects fail not from bad code but from solving the stated request instead of the underlying need.

**Smaller Discovery decisions:**
- **Repo name: `sdlc-ip-api`.** Tradeoff considered: a public geolocation service called ip-api.com already exists. A name *starting* with `ip-api` could look related; `sdlc-ip-api` reduces that but still matches "ip-api" in searches. `ipkit-sdlc` avoided it fully. Chosen anyway for plainness — a conscious, low-stakes tradeoff.
- **Separate log from Daily Driver**, so each project's log tells one story.
- **End User persona: Alice, a junior network engineer**, rather than reusing Bob — the persona should match who would really use the product.

---

## Sprint 0: Chartering

**Role: Product Owner** drafted `CHARTER.md`: a vision, three MVP capabilities (subnet info, longest-prefix-match lookup, route summarisation), an explicit out-of-scope list, a stakeholder table, five epics, four known risks, and four open questions for Alice.

**Status:** DRAFT. Alice has not yet reviewed it. This entry will be updated with her feedback and any revisions.

> **Teaching note:** Two things to notice in the charter. First, risk R3 (`192.168.1.5/24` — reject or normalise?) *looks* technical but is really a product decision about what the user expects; charters should route those questions to the user, not let the developer decide silently. Second, the out-of-scope list says *why* each item is deferred, so nobody later mistakes "not now" for "forgotten".

### Charter review, round 1 — IPv6

**Role: End User (Alice)** pushed back on deferring IPv6: *"IPv6 doesn't play nice with API gateways, so if we need to scale up, better to find those issues now."*

**The premise was partly outdated.** Checking the facts found that AWS API Gateway has accepted calls from IPv6 clients since March 2025, via a "dualstack" setting (off by default). So the gateway itself wasn't the problem.

**But the instinct was right, and it found a real design flaw.** Separating "IPv6 *clients*" from "IPv6 *as data*" showed that the risk was in our own API design. CIDR notation contains `/`, which the gateway reads as a path separator, so `GET /subnet/10.0.0.0/24` would fail with a 404. That flaw affected IPv4 as well. A second design issue: IPv6 has no broadcast address, so one response shape can't assume one.

**Options considered:** (A) full IPv6 in the MVP; (B) design for both now, build IPv4 first, IPv6 in Sprint 2; (C) IPv4 only, IPv6 deferred indefinitely.

**Decision: B.** Charter updated: addresses never go in the URL path, response shape must fit both versions, dualstack enabled at deploy. Security Reviewer added risk R5 (source-IP rules must cover IPv6 clients).

> **Teaching note:** The End User's stated reason was inaccurate, yet the concern behind it was valuable. A good review doesn't dismiss a stakeholder because their explanation is wrong; it separates the *claim* (gateways can't do IPv6 — false) from the *concern* (hidden scaling problems — real) and investigates the concern. Also notice the cheapest time to fix the URL design flaw was before any endpoint existed.

### Charter review, round 2 — host bits (R3)

**Question:** `192.168.1.5/24` — reject it, or quietly treat it as `192.168.1.0/24`?

**Role: End User (Alice)** chose **reject**, with a clear error message and a link to docs explaining it in depth. Alice also checked her understanding: the input is rejected because it isn't a *network* address.

**Clarified:** correct. It's a valid *host (interface)* address — the kind typed into router configs — but a network address must have every host bit set to zero. With `/24`, the last 8 bits are host bits; `.5` is `00000101`, not zero.

**Decision:** reject, returning a stable error code (`HOST_BITS_SET`), a plain-language message, the suggested network address, and a link to a new `docs/errors.md`. Added to Epic D.

> **Teaching note:** Two things. First, the End User restating the requirement in her own words ("so it's not a network address?") is a cheap and effective way to catch misunderstandings before they become code. Second, separating a fixed error *code* (for scripts) from the human *message* (for people) lets you improve wording later without breaking anyone's integration.

### Charter review, round 3 — routing table size limit (R1)

**Role: Product Owner** proposed a cap of 1,000 routes per lookup request, with context: small office routers hold ~5–20 routes, campus routers hundreds, the full internet table over a million.

**Role: End User (Alice)** pushed back: 1,000 is far more than teaching needs; **cap at 512**.

**Decision:** 512, accepted. Lower cap = less work per request and less abuse potential. Tradeoff accepted: someone pasting a large real table gets an error, so the error must state their count and the limit. The limit lives in one named setting in code so changing it later is a one-line edit.

**Role: QA Lead** added boundary tests: exactly 512 routes accepted, 513 rejected.

> **Teaching note:** Limits should be sized to the product's real purpose, not to what's technically possible — here the End User knew the use case (teaching) better than the Product Owner's generic estimate. And every limit you add creates an edge worth testing: bugs cluster at boundaries (off-by-one errors like `>` vs `>=`), which is why QA tests 512 and 513 specifically.

### Charter sign-off — Sprint 0 closed

**Role: End User (Alice)** accepted the out-of-scope list unchanged and signed off the charter. Status changed from DRAFT to APPROVED.

**What charter review changed** (compared with the Product Owner's first draft):
1. IPv6 moved from "out of scope" to "designed now, built in Sprint 2"
2. New design rule: no addresses in the URL path (found via the IPv6 discussion)
3. New risk R5: IP-based rules must cover IPv6 clients
4. Host-bits input rejected with code, explanation, suggestion and docs link; `docs/errors.md` added to the backlog
5. Routing table cap set at 512, lowered from a proposed 1,000

> **Teaching note:** Compare the draft charter with the approved one. Five real changes came out of one review, and two of them (the URL rule and R5) were problems nobody had spotted when the draft was written. That's the case for review before building: each of these would have cost more to fix after code existed.

---

## Sprint 1: Planning

**Which epic first?** The Product Owner offered three starts: (A) subnet-info maths only, no API; (B) a thin end-to-end "walking skeleton" of subnet info plus the API layer, recommended; (C) longest-prefix-match lookup first.

**Role: End User (Alice)** chose **C, "because it does the most."**

**Role: Product Owner** pushed back on the reasoning, not the choice: C doesn't do more, it *needs* more, since a lookup depends on the same parsing, validation and error handling B would have built. Resolution: make the lookup itself the walking skeleton. Subnet info moves to Sprint 2.

> **Teaching note:** A walking skeleton is the thinnest version of the system that works end to end. It doesn't have to be the *easiest* feature, just one complete path through every layer. Choosing the most interesting feature is fine as long as the sprint stays thin.

### Route format

**Role: Product Owner** proposed the minimum a route needs: `prefix` and `next_hop`.

**Role: End User (Alice)** wanted more: an `interface`, and the `protocol` the route was learned from, so each route reads like a line of `show ip route`.

**Accepted**, with validation rules from the QA Lead. Notably: `connected` routes need an interface and must *not* have a next hop, because a directly connected network is reached straight out of the interface. Interface names are free text, not validated against vendor styles, to avoid a rabbit hole.

### Duplicate prefixes and administrative distance

Adding `protocol` changed an earlier question. On a real router, the same prefix learned from two sources is not an error: the router prefers the source with the lower **administrative distance** (static 1, OSPF 110, RIP 120). An API that displays `protocol` but ignores it would teach a wrong mental model.

**Options:** (A) protocol display-only, duplicates rejected; (B) choose by AD now; (C) A for Sprint 1, B in a later sprint.

**Decision: C.** Alice: *"We definitely want to handle it, but no need to drown now."* Added to the charter as Epic F.

> **Teaching note:** The charter was already approved, and it still changed. That's normal in Agile. What matters is that the change is visible: the charter carries a note pointing here, rather than being silently edited so it looks like it always said this. Also notice that one new field (`protocol`) reopened a question that already seemed answered. New data often brings new behaviour with it.

**Sprint 1 plan written:** see `SPRINT_1.md` (four stories, each tested before the next starts).

---

*Next entries: Sprint 1 build, story by story.*
