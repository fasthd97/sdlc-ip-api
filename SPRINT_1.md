# Sprint 1 Plan — Longest-prefix-match lookup (walking skeleton)

**Goal:** `POST /lookup` works end to end, run and tested locally (no AWS yet), together with the router, validation and error format every later endpoint will reuse.

**Out of this sprint:** subnet info (Sprint 2), IPv6 (Sprint 2), administrative distance (Epic F, later), deployment.

---

## Request format

```json
{
  "destination": "10.1.2.3",
  "routes": [
    {"prefix": "10.1.0.0/16", "next_hop": "192.168.1.1", "interface": "Gi0/1", "protocol": "ospf"},
    {"prefix": "192.168.1.0/24", "interface": "Gi0/1", "protocol": "connected"},
    {"prefix": "0.0.0.0/0", "next_hop": "192.168.1.254", "interface": "Gi0/1", "protocol": "static"}
  ]
}
```

## Route validation rules

| Field | Rule |
|---|---|
| `prefix` | Required. Must be a network address; host bits set → `HOST_BITS_SET` |
| `protocol` | Required. One of `connected`, `static`, `rip`, `ospf`, `eigrp`, `isis`, `bgp` |
| `interface` | Free text, length-limited. Not validated against vendor naming styles. Required for `connected` |
| `next_hop` | IP address. Required for every protocol **except** `connected`, which must not have one |
| Whole table | At most 512 routes. Duplicate prefixes rejected (until Epic F adds AD) |

---

## Stories (each tested before the next starts)

**S1 — Router and error format**
- Unknown path → 404 with a JSON error body
- Invalid JSON body → clear error, not a crash
- Every error has a stable `code`, a readable `message`, and a `docs` link

**S2 — Request validation**
- Every rule in the table above has a passing test for the valid case and each invalid case
- Boundary: 512 routes accepted, 513 rejected, and the error states the count and the limit

**S3 — Longest-prefix match**
- The most specific matching route wins (e.g. a `/24` beats a `/16` beats `0.0.0.0/0`)
- `0.0.0.0/0` catches anything with no more specific match
- No match at all returns a clear "no route" answer, not an error crash
- The response names the chosen route and why it won (its prefix length)

**S4 — Error docs**
- `docs/errors.md` has one section per error code created in this sprint, each linked from the error response

---

## Definition of done (Sprint 1)

1. S1–S4 each proven by pytest; the whole suite green
2. `docs/errors.md` complete for every error code in use
3. SDLC_LOG.md updated, including a Sprint 1 retrospective
