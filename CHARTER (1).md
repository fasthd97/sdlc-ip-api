# Project Charter — sdlc-ip-api

**Status:** APPROVED (signed off by End User, Alice, 2026-09-26)
**Author:** Product Owner (persona)

---

## 1. Vision

A small, well-tested public API that answers the everyday IP questions a junior network engineer looks up by hand — "what's in this subnet?", "which route would this packet take?", "can these routes be summarised?" — returning clear JSON a script or a person can use.

## 2. Target user

**Alice, junior network engineer.** Knows IP addressing and subnetting from study, is still building confidence. Uses online subnet calculators today and double-checks her own maths. Would call this API from a browser, `curl`, or a Python script.

## 3. In scope (MVP)

| # | Capability | Example question it answers |
|---|---|---|
| F1 | **Subnet info** — network, broadcast, netmask, wildcard mask, prefix length, usable host range and count | "What's the usable range of 192.168.10.0/26?" |
| F2 | **Route lookup (longest-prefix match)** — given a small routing table and a destination IP, return the route a router would choose and why | "With these 4 routes, where does 10.1.2.3 go?" |
| F3 | **Route summarisation** — collapse a list of prefixes into the smallest equivalent set | "Can 10.0.0.0/24 and 10.0.1.0/24 be one route?" |

**IP versions (decided in charter review, option B):** the API is *designed* for IPv4 and IPv6 from the start (request format, response shape), but Sprint 1 *builds* IPv4 only. IPv6 is implemented in Sprint 2.

**Design constraints from charter review:**
- **Addresses never go in the URL path.** CIDR notation contains `/`, which the gateway treats as a path separator, and IPv6 colons are handled inconsistently by tools. Single values go in a query string (`GET /subnet?cidr=...`); lists go in a JSON body (`POST`).
- **Response shape must work for both versions.** IPv6 has no broadcast address, so subnet info can't assume one.
- **Dualstack enabled at deploy**, so IPv6 *clients* can call the API too (a configuration setting, separate from IPv6 *data*).

**Technical constraints (carried over from the Watch API project):**
- Python, standard library `ipaddress` for the maths (no third-party IP library needed)
- AWS Lambda (container image) behind API Gateway, raw hand-written router — no framework
- pytest test suite; every endpoint tested with valid and invalid input
- Deployed live so the README can link a working demo, within AWS free-tier usage

## 4. Out of scope (deliberately deferred, not forgotten)

- **IPv6 implementation in Sprint 1** — designed for now, built in Sprint 2 (see §3)
- **Listing every host in a subnet** — see risk R1
- **A web front end / visualisation** — the API comes first; a page can be a later epic
- **Authentication (JWT)** — already practised in the Watch API; the MVP is a public read-only calculator. *Security Reviewer may challenge this.*
- **Saving anything** — no database; every request is self-contained
- **The MPLS simulator and BGP looking glass** — separate future projects

## 5. Stakeholders

| Role | Played by | Cares about |
|---|---|---|
| End User — Alice | pyschoben | Correct answers, clear error messages, easy to call |
| Product Owner | Claude (persona) | Scope discipline — shipping a small complete thing |
| Security/Privacy Reviewer | Claude (persona) | Abuse of a public unauthenticated endpoint; cost exposure |
| QA Lead | Claude (persona) | Acceptance criteria written *before* code; edge cases |
| Future Maintainer | pyschoben + Claude | Simple code, documented decisions, easy to redeploy |

## 6. Initial backlog (unordered, unestimated — Sprint 1 planning will prioritise)

- [ ] Epic A: Subnet info (F1)
- [ ] Epic B: Longest-prefix-match lookup (F2)
- [ ] Epic C: Route summarisation (F3)
- [ ] Epic D: API layer — router, validation, error format (stable error codes + `docs/errors.md`)
- [ ] Epic E: Deploy + live demo + README polish
- [ ] Epic F: Administrative distance — when the same prefix is learned from different protocols, choose by standard AD and explain the choice (added in Sprint 1 planning; planned for a later sprint)

*Change after approval (Sprint 1 planning): routes gained `interface` and `protocol` fields, and Epic F was added. See SDLC_LOG.md.*

## 7. Known risks

- **R1 — Unbounded input.** A request that makes the server do work proportional to the input can be abused: a 50,000-line routing table in F2, or (if ever added) listing hosts of `10.0.0.0/8` ≈ 16.7M addresses. Needs explicit size limits on every input. **Decided: F2 routing tables capped at 512 routes** (sized for teaching use, kept as one named setting in code), with an error stating the count and the limit. QA will boundary-test 512 (accepted) and 513 (rejected).
- **R2 — Public endpoint abuse / surprise bill.** Unauthenticated means anyone can call it. Mitigations to evaluate: API Gateway throttling, an AWS budget alarm.
- **R3 — Ambiguous input.** `192.168.1.5/24` has host bits set: it's a valid *host (interface)* address but not a *network* address. **Decided: reject** with a clear error — a stable error code, a plain-language explanation, the suggested network address (`192.168.1.0/24`), and a link to `docs/errors.md` explaining it in depth.
- **R4 — "Correct" is harder than it looks.** Edge cases like `/31` and `/32` (usable host counts), and `0.0.0.0/0` as a default route in F2. Tests must cover these explicitly. IPv6 adds `/127`, `/128` and `::/0` in Sprint 2.
- **R5 — IPv6 clients bypassing IP-based rules.** Raised by Security Reviewer: if we ever restrict or throttle by source IP, those rules must include IPv6 once dualstack is on.

## 8. Open questions for Alice

1. ~~IPv4 only for the MVP, or IPv6 too?~~ **Decided: option B** (see §3)
2. ~~R3: reject or normalise `192.168.1.5/24`?~~ **Decided: reject with explanation, suggestion and docs link** (see §7)
3. ~~For F2, how big a routing table do you realistically paste in?~~ **Decided: cap at 512 routes** (see §7)
4. ~~Anything in "Out of scope" you'd move in, or vice versa?~~ **Decided: out-of-scope list accepted as is**

## 9. Definition of done (whole project)

1. F1–F3 work through the live API, proven by a green pytest suite covering valid, invalid, and edge-case input (R4)
2. Every input has a documented size limit (R1), and throttling + a budget alarm are in place (R2)
3. README explains what it does, how to call it, and links the live demo
4. `SDLC_LOG.md` covers every sprint, including a retrospective
