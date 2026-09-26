# sdlc-ip-api

A small IP toolkit API (subnet info, longest-prefix-match route lookup, route summarisation) built with Python, AWS Lambda and API Gateway — and deliberately run through a full Agile SDLC as a learning and portfolio project.

> **Status:** Sprint 0 complete: charter approved. Sprint 1 planning next. No code yet.

## Why this repo exists

1. **Show real networking knowledge in code**, not just as a theme: the core logic is the same decision a router makes (longest-prefix match), not a CRUD app.
2. **Practice Python and API Gateway** with patterns carried over from an earlier project (raw Lambda router, pytest, container images).
3. **Practice SDLC properly** — sprints, stakeholder roles, acceptance criteria, security review, retros — with the whole process written up for anyone else learning SDLC.

## Documents

| File | What it is |
|---|---|
| [`CHARTER.md`](CHARTER.md) | Sprint 0 project charter: vision, scope, stakeholders, backlog, risks |
| [`SPRINT_1.md`](SPRINT_1.md) | Sprint 1 plan: longest-prefix-match lookup, stories and definition of done |
| [`SDLC_LOG.md`](SDLC_LOG.md) | Narrated log of how decisions were made and why, written for a future learner |

## How this project is run

Agile, with small sprints. Stakeholder roles are role-played:

- **End User — Alice**, a junior network engineer (played by the author)
- **Product Owner, Security/Privacy Reviewer, QA Lead, Future Maintainer** (played by Claude, an AI assistant)

The roleplay is a learning device: it forces priorities to be argued and written down rather than assumed.
