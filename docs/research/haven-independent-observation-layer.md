# Independent observation layer (the auditor program)

*Captured 2026-06-06. A proposed mechanism, not settled architecture — it emerged from the positioning round of the civic-tech deep dive (the Stocksy comparison, by way of the question of outside board voices). It is recorded as its own doc because it grew past a design note into a distinct mechanism with its own rationale, hazards, and open questions. It sits next to, but is separate from, the unresolved board-composition question.*

Related: the niche-lock-in risk and the early-pilot / varied-surface commitments (see [civic-tech landscape](haven-civic-tech-landscape.md) and [design notes](haven-civic-tech-design-notes.md)); the access-discipline patterns it inherits from (institutional archive, the review sub-group, franking verification-key custody).

-----

## The problem it solves: opacity, not insularity

Two different governance deficits surfaced from the Stocksy comparison, and they must not be collapsed:

- **Insularity** — the Foundation Alliance becoming an echo chamber of mutually-aligned representatives. A legitimacy/diversity problem. Stocksy addresses its analog by reserving board seats for outside advisors. This is a *board-composition* question (see “the open question it sits beside”).
- **Opacity** — nobody with platform-level authority can see what is actually happening inside the communities they govern. An *information* problem, and the more dangerous of the two, because governance acting on bad information about its own communities produces well-meaning rules that are catastrophic on the ground.

The opacity deficit is structural and self-inflicted by design. Haven is operator-blind and its member communities are pseudonymous; the same blindness that protects members from the operator also blinds the governance layer from the members. **The people with authority cannot observe the thing they have authority over.** Stocksy never needs auditors because Stocksy is transparent to itself — its members *are* the product and are legible to one another. Haven needs them precisely because it isn’t. No board composition can fix this; insiders or outsiders, a board still can’t see in. The fix has to be a mechanism that creates sanctioned, bounded sight where the architecture otherwise allows none.

This also makes the mechanism the **standing institutional answer to niche-lock-in.** The survival-critical risk identified across the failure-mode round was coherence around one imagined community (the SSB hazard). “Get diverse real communities in front of the design early” is the pilot-stage answer; the observation layer is its *permanent* form — the thing that keeps the vision honest against real communities forever, against the specific blindness the privacy model imposes. Framed this way it isn’t a governance nicety; it’s the observation layer operator-blindness makes mandatory.

## The mechanism

An **independent observation layer**: trained observers who go into willing communities, watch how the platform is actually used, and report product-and-civic learning back to governance — housed in an external research partner (a university or civic-tech research lab), *not* in Strutco or the Foundation.

### Independence is load-bearing, not a nice-to-have

The observer’s value is tied to their independence the way franking verification keys are held by stable infrastructure that is not the enforcer. The independence does double structural work:

- **It makes the observation credible.** An auditor employed by Strutco or the Foundation is a company observer with a friendly title; their findings are suspect in exactly the direction that matters (flattering the platform).
- **It makes the access safer.** This is the subtle part. An observation program is, by construction, a deliberate *hole in the pseudonymity guarantee* — a sanctioned window into communities the architecture otherwise spent everything to keep closed. If Strutco employs the observers, that window is owned by the operator, and you have built the surveillance backdoor with extra steps. If an independent lab employs them, the access is mediated by an institution whose interests are publication and rigor rather than platform health, and the consent relationship is community-to-researcher, governed by *their* ethics apparatus, not Haven’s.

So the third party is not merely more credible — its independence is the property that makes the window worth opening at all.

### IRB as borrowed accountability

A university partner brings an **Institutional Review Board**: human-subjects review, informed-consent protocols, data-handling standards, conflict-of-interest disclosure. This is exactly the governance scaffolding an observation-inside-pseudonymous-communities program needs — and it already exists, is already credible, and is pointedly *not Haven’s to bend*. The partnership borrows an accountability structure, not just a research budget.

### Access discipline (inherited from existing patterns)

The window must inherit the same discipline as the institutional archive and the review sub-group, or it quietly becomes the backdoor:

- **Consented and invited, not imposed.** An observer enters a community because that community admits them — a temporary, vouched/credentialed role like any member — not because the Foundation can insert observers. Communities opt into being studied. Self-selection is acceptable: the goal is ethnographic depth, not a representative census.
- **Pattern reported, not content or identity.** Output to governance is “here is how proposals actually get used / where onboarding breaks / what the events flow feels like” — qualitative product-design ground truth — never “here is what community X said.” Learning flows up; surveillance does not. Same line as franking: the report carries the pattern, not the identities.
- **Bounded and logged.** Time-boxed engagements, visible to the community, on the record. The community knows the observer is present and what they are for.

## Hazards (the agenda-leak worry, which cuts both ways)

Independence protects against platform self-flattery but removes Haven’s control, so new hazards appear:

- **The researcher’s agenda becomes the lens.** A lab studying polarization will *see* polarization, because that is the instrument they brought; you get rigorous observation refracted through their research question, which may not be your product question. Mitigation: plurality (more than one relationship over time) and an explicit remit (“we need product-design ground truth — where onboarding breaks, how proposals get used — not a thesis”).
- **Publication incentives can diverge from community interest.** A juicy finding about a vulnerable community is both publishable and harmful. The IRB covers much of this, but the platform holds a line regardless: the communities are the subjects, their protection outranks the paper.
- **Independence is real or it isn’t.** If the Foundation funds the lab directly and substantially, this recreates the Mozilla–Google problem in miniature — nominal independence, actual dependence. The funding should be arm’s length (the lab’s own grants, a pooled fund, an aligned outside foundation) so the research budget is not a Strutco line item the researchers know not to bite. “Independent auditor” must not mean “auditor we pay.”

## The open question it sits beside (board composition)

Separate from this mechanism, and left open: how outside voice relates to the Foundation board when the board *is* the Foundation Alliance representatives.

Stocksy’s outside-board-seats trick does not port. In Stocksy, board and electorate are commensurable (~1,000 members, five seats; the board is a meaningful organ of that body). In Haven, the Foundation Alliance is a cryptographic electorate of potentially 100k+ people via representatives, and the board is those representatives — so a handful of appointed outside seats is rounding error on authority. Against a six-figure represented population, appointed outside voices “may as well be purely advisory.” That may be the honest answer (influence through voice and report, not vote), and outside *board* voices, if advisory, can be unpaid or honorarium-level. But the design of board-level outside voice is unresolved and should stay open.

The observation layer **stands on its own regardless of how the board question resolves**, because it answers the opacity deficit that no board composition can touch. Capture it as its own mechanism, not as a board variant.

## Summary of the find

- The deficit is *opacity* (governance can’t see into the communities it governs), distinct from *insularity* (board echo chamber); it is structural and imposed by operator-blindness.
- The mechanism is an **independent observation layer** housed in an external research partner, where independence is simultaneously the credibility guarantee and the privacy safeguard.
- A university partner supplies a borrowed accountability structure (IRB) the program needs and that is not Haven’s to bend.
- Access is consented, invited, pattern-not-content, bounded, and logged — inheriting institutional-archive / review-sub-group discipline.
- Hazards: agenda-as-lens, publication-vs-community-interest, and nominal-vs-real independence; mitigated by plurality, an explicit product remit, a community-protection line, and arm’s-length funding.
- It is the standing institutional answer to niche-lock-in — the permanent form of “diverse real communities in front of the design,” made mandatory by the privacy model’s blindness.

Open: the exact relationship between board-level outside voice and the represented electorate; whether the observation layer supplements or is wholly separate from any standing board outside-voice mechanism (current lean: wholly separate, since it solves a different problem); and the funding path, constrained by the arm’s-length requirement above.