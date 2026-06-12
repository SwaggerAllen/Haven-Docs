# Haven

*Your life doesn’t fit in your pocket.*

Haven is a federated, end-to-end-encrypted platform built around communities rather than individual accounts — digital scaffolding for the institutions that hold civic life together. Neighborhood associations, schools, co-ops, congregations, and other real-world institutions get digital tools that fit how they actually work: in-person joining, community-defined governance, conversations no outside party can read, and no advertising or engagement optimization — structurally, not as policy. Haven is being designed in full before it is built. This repository holds the public design documentation, the research behind it, and the published writing introducing the project.

## The documents

### Design — `docs/design/`

Settled design direction. These documents describe what Haven *is*.

- [Architecture](docs/design/civic-platform-architecture.md) — the master document: principles, the entity model, the cryptographic and governance choices. Start here for the full picture.
- [Features](docs/design/civic-platform-features.md) — the complete feature set, organized into the six-phase plan from pilot to maturity.
- [Federation readiness](docs/design/civic-platform-federation-readiness.md) — the commitments the pilot must honor so that federation is an expansion, not a rewrite.
- [Whole-system governance](docs/design/haven-whole-system-governance.md) — how the platform itself is governed: math-enabled democracy, the people-vote principle, layered authority.
- [Organizational structure](docs/design/haven-organizational-structure.md) — the two-entity model (a public benefit corporation and an independent foundation) and the path for getting there.

**Working notes** — active design directions, explicitly *not* settled:

- [Study notes](docs/design/haven-study-notes.md) — cryptographic design directions. Each item flags what is settled versus what awaits external cryptographer review; nothing here should be read as a verified construction.
- [Civic-tech design notes](docs/design/haven-civic-tech-design-notes.md) — design decisions surfaced by the civic-tech study, principally around proposal/voting UX and configuration.
- [Architecture critique notes](docs/design/haven-architecture-critique-notes.md) — findings from a critique pass over the architecture (June 2026), proposed responses, and where each should eventually land in the settled documents.
- [Independent observation layer](docs/design/haven-independent-observation-layer.md) — a proposed mechanism for sanctioned, bounded observation of willing communities, housed in an independent research partner. A proposal with its hazards stated, not settled architecture.

### Research — `docs/research/`

The evidence base. These analyses inform the design but are not design.

- [Founder-binding analysis](docs/research/haven-founder-binding-analysis.md) — eleven case studies of how mission-driven projects get captured or hold, and what Haven takes from them.
- [Civic-tech landscape](docs/research/haven-civic-tech-landscape.md) — failure modes across two decades of civic technology, and where Haven sits among the survivors.

### Open questions — `docs/questions/`

The living record of what the design cannot yet answer on its own.

- [Open questions](docs/questions/haven-open-questions.md) — questions awaiting expert review, practitioner experience, or research that doesn’t exist yet, maintained as they’re asked, answered, and closed. Review briefs land alongside as they’re written. If one of these sits in your field, we’d like to hear from you — the essays explain the project; this document is where help is most useful.

### Writing — `docs/outreach/`

Published essays introducing the project, in plain language.

**Introducing Haven:**

1. [What if the problem isn’t the algorithm?](docs/outreach/intro-article-1-algorithm.md)
1. [We built the internet as if every man were an island](docs/outreach/intro-article-2-island.md)
1. [The five open problems](docs/outreach/intro-article-3-stuck.md)

**The Slow Play:**

1. [The competitive ground](docs/outreach/intro-article-4-timescale.md)
1. [Design-first methodology](docs/outreach/intro-article-5-methodology.md)
1. [Founder succession](docs/outreach/intro-article-6-survival.md)

## About Haven

The diagnosis Haven starts from is that the deepest problem with today’s platforms isn’t the algorithm — it’s the unit. The individual account is the first-class thing, and the communities where people’s actual lives happen are an afterthought the software can barely see. Haven moves the unit: communities are the primary entities, with real structure — membership, roles, rules, history, governance, and even the ability to split — modeled directly rather than approximated by a page with posts on it. The app lives in your pocket like any other; the life it serves — the school, the block, the congregation, the room — doesn’t, and isn’t supposed to.

A few commitments follow from that and are non-negotiable in the design. Everything inside a community is end-to-end encrypted, so no operator, advertiser, or platform company can read it — privacy is a structural property, not a setting. Joining happens in person, vouched for by an existing member, which grounds trust in real-world relationships and makes infiltration expensive in the one currency attackers can’t manufacture. Communities govern themselves under their own rules, and allied communities can recognize one another without any central authority over all of them. The business model excludes advertising, data sales, engagement optimization, and transaction fees on community commerce — and the organizational structure (a public benefit corporation building the platform, with an independent foundation as trust anchor) is designed to bind the operator, including the founder, to those commitments.

Haven is currently in its design phase, working toward a small pilot with a single founding community. The documentation here is the design work itself, published as it matures: the design documents reflect settled direction, the working notes hold open questions honestly, and the research records the evidence the decisions rest on. If you’re new to the project, the essays in `docs/outreach/` are the intended entry point; the [architecture document](docs/design/civic-platform-architecture.md) is the canonical technical reference.

## What publishes here

The default is that everything publishes: design documents, working notes, open questions, review briefs, and the answers we receive — including unsettled directions and unflattering findings. Designing in the open is part of the methodology, and part of making the founder replaceable. Two narrow categories are withheld: planning notes that discuss specific people, communities, and organizations who haven’t agreed to appear in published documents, and production notes for public-facing media. One time-bound caveat: once real communities are running on Haven, a newly discovered question that bears on the security of deployed systems will get a disclosure-timing review before it lands in the public record — published after mitigation or a reasonable window, in line with standard responsible-disclosure practice. Everything published before deployment carries no such risk.

A note on process: this documentation was drafted with substantial AI assistance. The architecture, the decisions, and the mistakes are the author’s.

— Allen Strut, Strutco