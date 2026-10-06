# Strategic Architecture

> **ruby-veterinary Documentation**
>
> **Document:** Strategic Architecture
>
> **Version:** 1.0.0
>
> **Status:** Living Document
>
> **Owner:** ruby-veterinary
>
> **Classification:** Architecture Standard

---

# Purpose

This directory holds the strategic architecture for ruby-veterinary: how bounded contexts are classified and mapped, how events flow across them, who owns what, how contexts communicate under failure, and the non-negotiable rules that keep the modular monolith honest.

Strategic architecture sits above the technical architecture in `architectural-decision-record.md` and `deployment-architecture.md` — it is about business boundaries, integration patterns, and ownership, not servers and containers.

---

# Reading Order

| # | Document | Question it answers |
|---|----------|---------------------|
| 1 | [01-domain-context-map.md](01-domain-context-map.md) | What are the contexts, how are they classified, and how do they relate? |
| 2 | [02-domain-event-flows.md](02-domain-event-flows.md) | What happens in the system, and which events move it forward? |
| 3 | [03-context-relationships.md](03-context-relationships.md) | Which integration pattern does each context pair use? |
| 4 | [04-ownership-matrix.md](04-ownership-matrix.md) | Who owns each data, event, API, and schema? |
| 5 | [05-communication-matrix.md](05-communication-matrix.md) | How do contexts talk, and what happens when a peer fails? |
| 6 | [06-architecture-rules.md](06-architecture-rules.md) | What are the MUST/SHOULD/MAY rules for every change? |

Companion diagram: [domain-context-map.puml](domain-context-map.puml) (render with PlantUML).

---

# Scope Note

ruby-veterinary is a modular NestJS monolith on one VPS with ten bounded contexts — not a fleet of microservices. These documents govern boundaries **inside** the monolith. Extraction of a context into its own service requires a new ADR and an update here (`../architectural-decision-record.md`).

---

# Related Documents

| Document | Relationship |
|----------|-------------|
| `../bounded-context.md` | Source context map this directory elaborates |
| `../domain-model.md` | Domains behind the contexts |
| `../domain-events.md` | Event catalog referenced by 02 and 05 |
| `../architectural-decision-record.md` | Why the monolith, why PostgreSQL |
| `../deployment-architecture.md` | Where the single deployable runs |

---

# Acceptance Criteria

- Every context in `../bounded-context.md` appears in all six documents
- Every event in `../domain-events.md` is covered by at least one flow in 02
- Every sanctioned integration pattern in `../bounded-context.md` maps to a MUST rule in 06

---

# Guiding Principle

> **Draw the boundary where the vocabulary diverges. Everything else is discipline: own your data, speak through events, and never reach into a neighbour's schema.**
