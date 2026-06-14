# Architecture critique: findings, directions, and fold-in targets

**Status: working notebook. Nothing here is settled design.** This document captures findings from an external-style critique pass over the architecture doc (2026-06-12), the proposed responses worked out in discussion, and where each item should eventually land. Items follow the standard lifecycle: validated directions move into the architecture/feature/governance docs; rejected ones move to the considered-and-rejected record. Each item carries a status flag and fold-in target(s).

Items are ordered by severity, not by document order.

-----

## 1. Foundation-level sybil resistance (the humanness-count problem)

**Status: direction proposed, needs validation. Highest priority in this doc.**

**The problem.** Humanness is issued by the instance alliance, but the instance alliance’s own legitimacy is measured in humanness-credentialed members — a circular trust root. Worse, Foundation Alliance composition (the 0.01% threshold) is denominated in exactly the quantity a corrupt or lazy issuer can inflate, and people-vote — the platform’s capture defense — bottoms out in instance alliances honestly checking IDs. Community→alliance sybil-resistance inheritance is real but inherits *trust*, not *verification*: an alliance is only as sybil-resistant as its constituent issuers were honest, which is precisely the assumption under attack. The current “institutional attestation and audit-on-demand” fallback is the thinnest load-bearing element in the design.

**Reframe.** Don’t try to make the count *correct*; make inflation *expensive, visible, and punishable* — the same “expensive, detectable, actionable” standard the architecture already applies to abuse. Under that standard the problem appears tractable, collapsing to the long-game-cooperator residual already classified as bounded.

**Proposed mechanism set:**

1. **Issuer certification with skin in the game.** Certification targets the *instance alliance as humanness issuer*, not the operator (the operator never touches issuance; operator certification is the wrong collateral). Discovered inflation → decertification → federation peers stop honoring that alliance’s humanness credentials (machinery already exists: cross-instance honoring is a trust-policy decision) → its communities’ Foundation weight discounted. Punishment lands partly on innocent hosted communities, but the migration design already gives them an exit: re-present external proof, re-issue at a new instance, checked against the revocation log.
1. **Attendance-anchored Foundation counting.** Anchor the counting population to attested in-person event attendance — an entity’s count toward the 0.01% threshold is not “members holding humanness credentials” but something like “members with attested attendance at N in-person events over the trailing year.” Events with attendance verification are already a core content type, so honest communities pay nothing. This converts a key-minting attack into a hire-physical-bodies-for-a-year attack — the same cost wall real institutions face (church packing, party entryism), consistent with the match-real-institutions philosophy. *Open sub-question:* prove “attended ≥N attested events” without cross-community linkability. Looks expressible with existing machinery (shared-`pid` credential linking, scoped nullifiers) but this is a composition claim → cryptographer review list, not settled.
1. **Issuance-rate transparency.** Log every humanness issuance as an opaque event (rates only, no identities) to the existing verifiable-log infrastructure. Implausible growth becomes publicly anomalous before any audit. Converts audit-on-demand from hand-wave to triggered process. Near-zero cost.
1. **Statistical sampling instead of universal verification.** Random-sample a candidate entity’s claimed members; sampled pseudonyms must produce a fresh proof and appear at an attested verification event. Bounds the inflation rate an issuer can sustain undetected. Consent-to-sampling is a condition of claiming the threshold. *Note:* this is a natural second use case for the proposed independent observation layer (see `haven-independent-observation-layer.md`) — counting audit is a different remit than ethnographic observation and would need its own access discipline, but the independence and arm’s-length structure carry over. Flag for that doc’s lifecycle decision.
1. **Tenure vesting.** Foundation-counting weight vests with credential age and participation history. Mass fresh issuance carries zero immediate governance weight, forcibly converting every fast attack into the slow version — exposed to rate transparency and sampling the entire time it runs.
1. **In-person verification of representatives at Foundation level.** Cheap (composition caps at 10,000 entities; the constitutional convention is a natural venue). Closes the representative-impersonation hole. Does *not* close the inflated-count hole — both layers are needed: representatives verified in person, counts verified by attendance-anchoring plus sampling.

**Structural note — the denominator.** The 0.01% threshold divides by total platform population, so one inflating instance shifts *everyone’s* threshold. The denominator must use the same attested count as the numerator or the scheme leaks.

**Honest residual to state explicitly.** A patient, funded adversary willing to operate real humans through real events for years can still buy governance weight. Nothing fixes that — including in real-world institutions. Tenure vesting plus 2/3 thresholds force them to sustain it at scale, in public view of the rate logs, for years. State this in the architecture doc’s register: mechanism, cost analysis, named residual.

**User story and deployment sequencing (added after discussion, 2026-06-12).**

*Scoping the member burden.* Attendance anchoring affects Foundation *counting* only — participation is untouched; non-attending members simply don’t count toward their community’s 0.01% weight. Keep N low (one or two attested events per trailing year — annual-meeting rhythm, not continuous attestation). Attestation is a byproduct of events communities already hold (the GTM selects for gathering-first communities); the new work lands on organizers running attendance verification, which is already a core feature with independent value (behavioral reputation), not on members doing anything new.

*Fairness residual (name now, solve later).* Homebound, disabled, and dispersed members get discounted from governance weight. Real institutions share this problem (hence proxy/absentee mechanisms). Candidate relief valve: vouched non-attendance attestation at reduced weight. Not designed; flagged.

*Early-phase vulnerability and what actually ships when.* Pre-convention there is no Foundation Alliance to capture; the founder backstop is the authority, which is what the phased founder-binding design is for. The window that matters is convention composition (end of Phase 2). Ship-day requirements are therefore the cheap subset:

1. **Rate transparency from day one** — logging only; also creates the historical audit trail for later review of the early period.
1. **Tenure vesting declared early** (declaration is policy, not machinery).
1. **Declared-in-advance counting rules:** announce from the start that convention counting will use the attested standard with **no grandfathering of unattested counts**. This is the mechanism that prevents the lax early period from locking in.

The heavy machinery (unlinkable attendance proofs, sampling program) must exist *by the convention*, aligning with the existing “Foundation Alliance lifecycle — settle by end of Phase 2” active-design item.

*Named wrinkle: tenure vesting rewards early entry,* interacting with the already-listed early-adopter-cartel edge case — communities planted in Phase 1 vest by convention. Mitigation is structural: the early network is small, gatekeeper-led, and in-person-gated, so fake communities are most expensive exactly when the network is most socially legible, and day-one rate logs preserve auditability. Accepted early vulnerability is the variant the GTM design defends best.

**Fold-in targets:** architecture doc (new “Foundation-level sybil resistance” section; revise “Sybil resistance edge cases” open question); whole-system-governance doc (people-vote anchoring, threshold denomination); organizational-structure doc (certification program scope: issuer certification, not just operator; convention counting rules declared in advance); features doc (attendance-credential counting with phase tags: rate transparency Phase 0, attestation/sampling machinery by end of Phase 2); adoption-gtm doc (organizer-side attestation burden; early-cartel legibility argument); study notes (the unlinkable attendance-proof composition question).

-----

## 2. Threshold issuance vs. short-lived freshness credentials

**Status: open tension, no direction chosen. Must resolve before crypto review.**

The architecture claims threshold issuance prevents any single representative from holding the dedup registry, *and* that revocation latency equals freshness-credential lifetime (“tunable”). These conflict operationally: if freshness reissue is a genuine threshold ceremony, human coordination sits on the hot path at user-population frequency; if reissue is delegated to an automated signer, the threshold property is decorative for the high-frequency path and the registry effectively lives with whoever runs the signer.

Candidate resolutions to evaluate (none chosen): threshold applies only to root issuance and revocation-log writes while freshness reissue is single-signer automated with its own rate transparency; threshold-key automated signing (FROST-style, signers are machines holding shares under different representatives’ control); longer freshness lifetimes accepting slower revocation. Each has different trust and ops profiles — needs a worked comparison.

**Fold-in targets:** architecture doc (two-credential model section must say which); study notes / cryptographer review list.

-----

## 3. Operator metadata visibility (traffic analysis)

**Status: accepted residual — decision made, needs writing down. Cryptographically unsolvable at this layer.**

The blind hub still sees partition-key activity volumes, timing, message sizes, fanout shape. Epoch-rotating partition keys limit long-term correlation but not within-epoch analysis. Direct federation makes the inter-instance social graph explicit by design; QR-driven federation establishment timestamps join events. A blind operator with a traffic graph is less blind than the current framing implies.

**Decision:** chalked up as unsolvable at the cryptography layer. Tor (or similar transport-layer mitigations) can reduce client-to-hub correlation but is not an absolute defense and brings its own ops and UX costs — possible future mitigation, not a commitment.

**Action:** add a “metadata residuals” passage to the architecture doc in the same honest register as the issuance-anchor residual: what the operator sees, what rotation does and doesn’t buy, what federation-graph visibility means, why this is accepted, what partial mitigations exist. The doc currently accounts for issuance residuals well and traffic residuals not at all; symmetry is the fix.

**The argument the passage must make (added 2026-06-12, from the FED1/MIMI thread).** The sharpest attack on accepting this residual is “inconvenient, not impossible is no protection at all” — i.e., partial privacy is *theater* because users misjudge their exposure. The passage must answer it directly, and the answer is a layered-defense thesis, not a concession: Haven does not claim correlation-resistance; it claims correlation, even when successful, buys an attacker little and costs them a lot. The correlation attack is only the first barrier: (1) **technical friction** (partition keys + anonymous BBS) converts impulsive operator action into deliberate sustained effort; (2) **social friction** — the operator is overriding a *visible democratic decision*, often against people known personally (non-cryptographic, so the critique can’t reach it); (3) **exit cost** — schism and portability mean a targeted community leaves intact, so the attack fails its goal and costs the operator standing and members; (4) **scale economics** — continuously re-correlating every community on a large server across rotating partition keys is expensive per-community and continuous, so it doesn’t scale to mass surveillance. This is the *opposite* of theater: theater is appearance substituting for protection, whereas here the protection is real but located in the *consequences and economics* of the attack rather than its cryptographic impossibility — and honest disclosure of the leakage is exactly what keeps users from being misled. **Haven-specific amplifier:** because governance decisions are committed to MLS state, the encrypted state *is* the governance record, so an operator defeating it doesn’t merely read messages — it visibly attacks the integrity of the democratic substrate, exposed by transparency logs and representative reconciliation. The stakes of breaking governance-bearing encrypted state sit far above the generic-messaging case, making Haven’s acceptance of the residual more defensible in context than the same residual in a general messenger (a weight MIMI’s DS, not carrying governance, wouldn’t have to consider). This thesis is also the spine of any eventual FED1/MIMI conversation about whether the partition-key approach is “privacy theater.”

**Fold-in targets:** architecture doc (cryptographic architecture / blind hub sections — the residuals passage carries the layered-defense thesis); possibly federation-readiness doc (federation-graph visibility is a commitment-relevant property).

-----

## 4. Liveness under single-authoritative-writer

**Status: open. Partially acknowledged in existing open questions (“operator disappearance recovery”).**

Single-authoritative-writer per mailbox is the right call against split-brain, but every community’s liveness depends on one sequencer. The threat model demotes the hostile operator to “vandal,” yet availability vandalism against the only authoritative hub is a full denial of service, and recovery (2/3+ representative reconciliation, threshold transition attestation) is a human-latency process. The most common real-world case is not hostility but disappearance — operator stops responding, infra decays. Allied replicas protect durability, not liveness.

Needs: a specified failover protocol with target latencies; what’s automatable vs. what requires representative ceremony; whether allied replicas can serve degraded read-only service during an authority gap; detection thresholds (how long unresponsive before transition machinery triggers).

**Fold-in targets:** architecture doc (operator transitions section); federation-readiness doc (peers need to know how to behave during an authority gap); features doc (phase tagging for failover tooling).

-----

## 5. Cryptographic composition surface vs. review proportionality

**Status: open — process problem, not design problem.**

The aggregate composition (MLS + branched sub-groups + resumption PSKs + sender-key layer + BBS blind issuance / per-verifier pseudonyms / credential-linking ZK equality / nullifiers + committing-AEAD franking inside MLS + threshold signatures + exporter-derived everything) is novel even though each primitive is validated — the same reason the single-MLS-with-community-leaves design was killed applies in weaker form to the residual composition. The pending-review list is multiple papers’ worth of analysis; “external cryptographer review” singular is not proportionate.

Also: the architecture’s interdependence (“evaluated as a whole”) is brittleness against review findings. If review kills one property — e.g., blind issuance doesn’t deliver the impersonation-to-sybil inversion — the failure cascades through migration, revocation, and the people-vote anchor. Worth pre-mapping the dependency graph of review findings: for each pending-review item, what fails downstream if the answer is no, and what the fallback is. The plain-English reduction-style proof sketches already planned are the right vehicle; this adds a “blast radius” column.

**Fold-in targets:** study notes (review-list restructure with dependency/blast-radius mapping); architecture doc (design-status section: scope the review as a program, not an event); possibly intro-article-5 methodology framing eventually (it’s honest about this kind of cost already).

-----

## 6. Security invariants living in application code

**Status: open. Two named instances; probably more to inventory.**

(a) **Parent Remove → sub-group Remove fan-out** is an application-layer obligation whose failure mode is ex-members keeping treasury/council access. Needs to be specified as a named invariant with a verification story (e.g., periodic reconciliation between parent roster and sub-group rosters; audit-log entry on every fan-out; alarm on divergence), not a paragraph caveat.

(b) **The per-scope counting authority for nullifier cardinality caps** reintroduces a trusted party at the vote-counting step of math-enabled democracy. Verifiable logs make the tally *attributable*, not *correct*. Minimum fix: counting authority publishes the full pseudonym→count multiset (pseudonyms are already scope-local and unlinkable elsewhere) so any member can recount; commitment to the multiset goes to the verifiable log. Evaluate whether per-scope pseudonym publication has unconsidered linkability costs → cryptographer review list.

**Action:** inventory pass over the architecture doc for other invariants of this class (cross-layer obligations enforced only by application code) so they can be named, given verification stories, and tested as a set.

**Fold-in targets:** architecture doc (sub-groups section, math-enabled democracy section); study notes (recount-publication linkability question); features doc (reconciliation/alarm tooling, phase-tagged).

-----

## 7. Schism inheritance vs. archive governance; alliance-transfer default

**Status: open conflict between two settled mechanisms — needs a design decision, not just wording.**

(a) **History inheritance.** Schism promises the minority inherits content history, but retained history lives in the institutional archive whose decryption authority is threshold-shared among the *original* community’s archivists — controlled, post-schism, by the majority. Either schism gets a cryptographic carve-out (weakening the archive-governance story) or “inherits content history” is aspirational beyond the minority’s own local state. Candidate directions: archive-key escrow that splits at schism (heavy); inheritance scoped to open-read feeds plus members’ own local/peer state, with confidential-archive inheritance explicitly excluded and documented (honest, cheap); schism-triggered governed export with both factions’ threshold participation (middle). Lean toward honesty-about-scope unless a real constituency needs confidential-archive inheritance.

(b) **Alliance-transfer-by-default as infiltration primitive.** A three-person hostile splinter automatically lands inside every alliance the parent belonged to; ejection requires affirmative governance action from each alliance. The continuity rationale is sound for good-faith schisms. Candidate mitigations preserving the default: a minimum-fraction threshold below which transfer is opt-in by the alliance rather than automatic; or a probationary status (member-without-confidential-access) until the alliance affirms. Note the interaction with the two-layer alliance design: at minimum, schismatic communities should not automatically receive a seat in the representative MLS governance group — broadcast-layer continuity and governance-layer admission can sensibly differ.

**Fold-in targets:** architecture doc (schism section, history/retention section); whole-system-governance doc (alliance admission); features doc (phase tagging for schism mechanics).

-----

## 8. Instance economics and the “commodity infrastructure” claim

**Status: open — strategy-level, not protocol-level.**

The ops load asked of instances (blind hub sequencing, per-write signature verification, replica attestation, witness participation, federation, webhook integrations, NCMEC registration, undefined posture in E2E-hostile jurisdictions) plus thin revenue (small credential-production share) sits uneasily with “instances are commodity infrastructure.” Operator plurality is itself a security assumption (censorship resistance, no single point of capture), so instance economics is a security topic, not just a business one. Related: the 10,000-member Foundation Alliance MLS group is at the documented edge of MLS practicality — performance benchmarking is already on the investigations list; keep it tied to this concern.

Needs: a worked instance-operator cost model (what it actually takes to run one, at pilot scale and at maturity) and an honest statement of which operator profiles are viable in which phases. Phase 0–1 reality is probably “Strutco plus a small number of committed operators,” which is fine if said out loud and bounded by the federation-readiness commitments.

**Fold-in targets:** organizational-structure doc (operator economics); adoption/GTM doc (operator recruitment realism); architecture doc (soften or qualify “commodity infrastructure”); features doc (phase tags on operator tooling).

-----

## Additions to the cryptographer review list

Consolidated from the items above, for merge into the study-notes review list:

1. Unlinkable attendance-count proof: “attended ≥N attested events in window” across communities without cross-community linkability, composed from shared-`pid` linking and scoped nullifiers (item 1).
1. Threshold-vs-automated freshness reissue: trust analysis of candidate constructions, including FROST-style machine-held shares (item 2).
1. Recount-publication linkability: does publishing the per-scope pseudonym→count multiset for member recounting create correlation surface (item 6b).
1. Blast-radius mapping as a review deliverable format: for each reviewed property, the downstream dependency set if it fails (item 5 — process, but the review should be scoped this way).

-----

## Addendum (2026-06-12): resolution of item 6b’s recount question, and two new fold-ins

The recount-publication question spun out of item 6b (tracked as CR-B4 in the open-questions ledger) closed by analysis; full resolution recorded there. Design consequences for the fold-in queue:

1. **Per-vote disclosure scope is governance configuration, not a security parameter.** Real institutions deliberately span the full spectrum (roll-call votes to secret ballots); the platform makes each community’s choice enforceable, not the choice itself. Requirements: scope fixed and visible before ballots are cast; loosening future-vote disclosure is itself a ratcheted governance action; no retroactive widening, as a cryptographic property rather than a policy flag. Recount worksheets default to members-only. → civic-tech design notes (config sliders), features doc.
1. **Correlation residuals should be priced as a cumulative budget, not item-by-item.** The in-person gate, attendance-anchored counting, and optional public votes each spend cross-scope unlinkability in opt-in increments; a community’s disclosure choices compound. The metadata-residuals writeup (item 3) should account for the budget, and the configuration UX should eventually surface a community’s cumulative public surface. Anchor for the docs: members-only scope-local pseudonym worksheets are strictly more private than existing democratic practice (public real-name voter participation files; public election bulletin boards).
1. **Contested-vote auditing imports election practice** (accredited observers, risk-limiting-audit escalation): auditor access pre-committed in the vote’s configuration and dormant until invoked — never negotiated after a dispute. This is the third independent arrival at bounded, consented, independent human access as the escalation tier above cryptographic verification (ethnographic observation; sybil sampling, item 1 mechanism 4; now contested votes) — evidence the observation layer is load-bearing architecture rather than a bolt-on. → observer doc lifecycle note, whole-system-governance doc.
1. **The study notes’ cryptographic references need a freshness pass (added 2026-06-12 during the audit campaign).** Auditing the crypto questions surfaced, three separate times, that the field had moved closer to Haven’s design than the study notes recorded: DMLS / AMT puncturing for fork recovery (CR-M2), MlsGov for governance-over-MLS (CR-F1), and a richer 2025 credential-composition literature (Mayrhofer pseudonyms note, device-binding 2025/1995, SoK 2026/330) for CR-B1. The notes appear to be written against a ~2024 snapshot; the 2025 literature has been busy in exactly Haven’s areas. Implication: the un-audited crypto items (CR-B5, CR-M3, CR-M4, CR-M5, CR-F4, CR-F5, CR-M7) likely also have stale references and should be audited before their briefs, not just compiled from the notes. Two public-doc-urgent positioning fold-ins fall out: the architecture doc should cite and position against **MlsGov** (private hierarchical governance over MLS, S&P 2024) and name **PolicyKit** as the plaintext-governance root of the lineage (PolicyKit → MlsGov → Haven); the civic-tech landscape should add PolicyKit as a reference/survivor. A knowledgeable reader reaches for MlsGov within seconds of “private governance over MLS,” so the gap is conspicuous while the docs are public.

-----

## Addendum (2026-06-12, cont.): crypto-audit campaign closeout — design decisions and new residuals

The audit pass over the cryptographer review list (closing roughly half the items as homework or committed design, sharpening the rest) surfaced several things that are *design decisions and residuals*, not open questions, and so belong here rather than in the open-questions ledger.

**A. Single-signer humanness issuance (CR-M7 resolved).** Threshold issuance for humanness is dropped, not optimized. The no-unilateral-minting property it protected is now carried by three other layers: rate-transparency logging, attendance-anchoring (item 1), and per-relying-entity acceptance policy (each entity chooses which issuers to honor and can revoke acceptance / decertify). A single logged signing key is *more* accountable for the decertification lever than a diffuse k-of-n group. FROST / non-interactive threshold signing: considered, not adopted, for humanness. → architecture doc (humanness issuance), supersedes critique item 2’s “two-credential section” framing for the humanness path; updates the fold-in table row 2.

**B. The humanness transitive-trust boundary (new named residual).** Humanness is a *propagated human-verification assertion*, not a cryptographic guarantee: someone physically confirmed a real person at the bottom, and the credential chain transmits that assertion faithfully but can never manufacture it. Across federation the trust degrades — accepting a community’s humanness credentials proves only that the issuing community vouches for its members (at the limit, “at least one real person vouched for this group”), not that each credentialed member is a distinct human. Mitigations are all soft/economic: acceptance policy, issuer reputation, consumer-side attendance-anchoring. There is no cryptographic fix because the thing trusted is an unwitnessed offline human act. This is the same floor first hit from the issuance side in the item-1 sybil discussion, now stated from the consumer side. It should be written as the architecture’s honest floor, once, and referenced — not rediscovered. → architecture doc (residuals register, alongside the issuance-anchor and metadata residuals); interacts with item 1 (a lax-verification federated community is an inflation vector attendance-anchoring only partly closes).

**C. Humanness-consumer audit (new standing check).** Single-signer humanness is sound only where every security-critical consumer of humanness sits behind a second detective or economic gate (rate log, attendance, nullifier scope). Action: inventory humanness consumers; flag any that trust raw humanness ungated, because the dropped threshold was protecting exactly those. → architecture doc (a verification pass, like the app-layer-invariant inventory in item 6).

**D. Three structural commitments on the Foundation Alliance (fold-in; never previously written down).** These soften the item-1 circularity and are settled design, not open questions:

1. **Instance alliances cannot be Foundation Alliance members.** The humanness issuer must not directly hold governance weight denominated in the credential it issues — that is the worst form of the item-1 circularity (issuer minting weight for itself). Excluding instance alliances forces the path to Foundation weight through a *real social alliance* (affiliation sense, not server-plumbing sense — the alliance/instance distinction is load-bearing here): the issuer can mint credentials but cannot convert them to its own weight without a socially-grounded community vouching it in. A structural firewall on the circularity.
1. **Foundation Alliance membership uses the same requirements and flow as any other alliance — no privileged path.** This guarantees the Foundation Alliance is covered by the same sybil-resistance machinery (in-person gate, attendance-anchoring, locality-grounded resistance) as every other alliance, with no bespoke admission flow to maintain or attack. Forbids ever giving the Foundation Alliance a special membership mechanism.
1. **Corollary — the Foundation Alliance is competable-with.** Because there is no privileged mechanism, a rival alliance can form with the same structure, math, and sybil resistance, and compete for legitimacy. The Foundation Alliance’s authority comes not from protocol privilege but from stewardship of the canonical codebase (lieutenants elected over it) and accumulated legitimacy; a competitor *may* form but must fork the entire code-and-maintenance apparatus and rebuild legitimacy from zero — “expensive, not impossible,” at the governance layer. This resolves the apparent tension between “competable-with” and “authoritative”: the codebase and recognition are the contested prize, not the membership flow.
   *Thesis worth stating explicitly in the governance doc:* the platform makes communities forkable (schism), the trust anchor forkable (competable alliance), and the founder bound (succession) — three layers, one anti-capture principle: exit is always possible, never free. → whole-system-governance doc (Foundation Alliance admission, the forkability thesis), architecture doc (entity model: instance-alliance exclusion).

-----

## Fold-in tracking

|Item                                             |Architecture                                  |Governance                         |Org structure       |Federation readiness         |Features                   |Study notes                  |Other                                               |
|-------------------------------------------------|----------------------------------------------|-----------------------------------|--------------------|-----------------------------|---------------------------|-----------------------------|----------------------------------------------------|
|1. Foundation sybil resistance                   |new section + open-q revision                 |people-vote anchoring              |issuer certification|—                            |attendance counting, phases|attendance-proof question    |independent-observation-layer (counting-audit remit)|
|2. Threshold vs freshness                        |RESOLVED: single-signer humanness (addendum A)|—                                  |—                   |—                            |—                          |considered-not-adopted: FROST|—                                                   |
|8. Instance economics                            |qualify “commodity”                           |—                                  |operator economics  |—                            |operator tooling phases    |—                            |adoption-gtm                                        |
|9. Foundation Alliance structure (addendum D)    |entity model: instance-alliance exclusion     |admission flow + forkability thesis|—                   |—                            |—                          |—                            |—                                                   |
|10. Humanness transitive-trust floor (addendum B)|residuals register                            |—                                  |—                   |federation: trust degradation|—                          |—                            |—                                                   |
|3. Metadata residuals                            |residuals passage                             |—                                  |—                   |graph visibility             |—                          |—                            |—                                                   |
|4. Liveness/disappearance                        |operator transitions                          |—                                  |—                   |authority-gap behavior       |failover tooling           |—                            |—                                                   |
|5. Review proportionality                        |design-status scoping                         |—                                  |—                   |—                            |—                          |review-list restructure      |intro-article-5 (eventual)                          |
|6. App-layer invariants                          |sub-groups + MED sections                     |—                                  |—                   |—                            |reconciliation tooling     |recount question             |—                                                   |
|7. Schism vs archive                             |schism + retention                            |alliance admission                 |—                   |—                            |schism mechanics           |—                            |—                                                   |
|8. Instance economics                            |qualify “commodity”                           |—                                  |operator economics  |—                            |operator tooling phases    |—                            |adoption-gtm                                        |