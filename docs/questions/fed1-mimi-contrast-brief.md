# Haven and the MIMI mainline: three divergences

*In brief: Haven and the MIMI working-group design both build federated MLS with hub-mediated delivery and per-room pseudonymity, and make opposite cuts on three coupled axes: a structurally blind hub vs. a pseudonymous state-tracking hub, BBS credentials vs. symmetric identity-link keys, and an in-person join gate vs. last-resort KeyPackages. This note states where Haven diverges and why, and asks which of those cuts is mispriced. Both designs concede traffic-correlation; Haven claims no advantage there.*

## Setting

Haven sequences each MLS group through a single-writer hub that is **structurally blind**: it orders and fans out ciphertext addressed by per-epoch partition keys derived from the group's MLS exporter, and never sees content, membership, or handshake structure. The MIMI mainline (`draft-ietf-mimi-protocol`) instead has the hub **track group state** (AppSync proposals carried in PublicMessages) and serve GroupInfo to joiners, with metadata exposure reduced optionally, per room, via Minimal Metadata Rooms (MMR): the participant list and credentials become pseudonyms, with real identifiers encrypted to room members but not the hub.

The threat models differ in emphasis, and that drives everything below. Haven's operator is a potential **adversary over governance-bearing state** — votes, membership, and roles are committed to the encrypted MLS state, so the hub seeing group structure is itself the harm. MIMI treats the hub's metadata visibility primarily as a privacy cost to be minimized, and its metadata story is still settling — pseudonymity is opt-in per room and the identity-link key management is an open TODO — so the contrast below is a difference of default and degree, not a clean opposite.

## Axis 1 — the hub: structural blindness vs. pseudonymous tracking

MMR keeps the hub tracking group state and serving GroupInfo, and makes pseudonymity an opt-in per-room mode; even in an MMR the hub still observes per-pseudonym membership churn and activity. Haven blinds the hub structurally, for every group, and pays for it: no hub-served GroupInfo, so joins cannot be hub-assisted (see Axis 3).

Our reasoning is that a pseudonymous-but-tracking hub still sees the *shape* of every community — who acts, when, in what structure — and for governance that shape is the sensitive object even when names are pseudonyms. The question (carried to "what we're asking" below) is whether the governance setting actually warrants the stronger cut, or whether blindness costs what pseudonymity would have bought more cheaply.

## Axis 2 — pseudonyms: a credential bundle vs. symmetric link-keys

These look like the same mechanism and are not. MMR pseudonyms hide identity *from the hub* while remaining linkable *to room members* (all participants decrypt the identity-link ciphertext); the live draft flags its key-management scheme as an explicit TODO ("efficient; FS/PCS; all participants can learn identities"). Haven's pseudonyms come from a single BBS credential blind-issued over a committed secret `pid`, and carry four properties at once: selective disclosure; per-verifier pseudonyms unlinkable *across* scopes (not just to the hub); nullifier soundness, one pseudonym per (`pid`, scope); and blind issuance, the issuer never learning `pid`.

Three of those four are not goals MMR's scheme attempts. Cross-scope unlinkability to *other members* is the opposite of MMR's all-members-can-link requirement. Scoped-nullifier soundness and blind issuance have no MMR analogue at all — because Haven's pseudonyms carry governance (one-person-one-vote, N-ticket caps, sybil resistance), not only metadata hiding. So "could Haven adopt MMR pseudonyms instead" resolves cleanly: no, they do not provide the cross-scope unlinkability or the nullifier/issuance properties governance needs.

The question runs the other way, and the draft already gestures at its own intended answer: the identity-link scheme is meant to be replaced by TreeWrap, a symmetric tree-based key-management construction. So the honest question isn't "should you use BBS instead" — it's whether a BBS-family credential is even a candidate for that slot, or whether it *over*-serves, since its cross-scope unlinkability actively fights MMR's all-members-can-link requirement. Our expectation is that it over-serves and TreeWrap is right for MMR's narrower goal; we'd value confirmation that we're reading the divergence correctly rather than missing a reason the bundle would help. (Whether Haven's four-way bundle is itself sound is a separate question, posed to the BBS authors as CR-B1; this note assumes it and asks only about the contrast.)

## Axis 3 — the join, where the first two couple

Pseudonymity forces join restrictions in both designs — the live MMR text concedes that its key management "leads to restrictions in how the MMR can be joined and which users each participant can add." MMR resolves this with last-resort KeyPackage sets, and its own authors note that reusing such a set lets the hub learn one user is active in two groups. Haven forbids last-resort KeyPackages entirely; without hub-served GroupInfo (Axis 1), every join is in-person-rooted or member-assisted, which closes that reuse leak at the cost of never admitting an absent, unconnected, non-interactive joiner.

Both designs independently land on member-mediated joins for pseudonymous membership. The question: is that convergence fundamental — does unlinkable pseudonymous membership *require* a member to mediate the identity link — or is it an artifact of these two designs, with a hub-assisted pseudonymous join possible that neither of us found?

## What we're asking

The one that matters most is **Axis 1**: against an adversarial operator over governance-bearing state, does structural blindness buy anything pseudonymous tracking doesn't already give? Structural blindness is the load-bearing commitment the rest of Haven is built around, so a strong argument that it's unnecessary is the most consequential thing you could tell us.

Axes 2 and 3 are secondary — included so the comparison is legible, and worth a line only if you have the appetite: whether the BBS bundle is the wrong tool where TreeWrap suffices (Axis 2), and whether member-mediated joins are actually avoidable (Axis 3). Each is a place to tell us we're wrong; neither is the headline.

## Scope notes

- The soundness of Haven's four-property BBS conjunction is not asked here; it is CR-B1, addressed to the BBS authors. This note treats the bundle as given and compares goals.
- MMR's identity-link key management is, by the draft's own marking, an open TODO; comparisons are against the current text, not a finished scheme.

## References

- `draft-ietf-mimi-protocol` (Minimal Metadata Rooms; AppSync state synchronization).
- `draft-kohbrok-mimi-metadata-minimalization` (the MMR pseudonymization scheme's origin sketch).
- `draft-mahy-mimi-pseudonyms` (pseudonymous join flows; selective-disclosure credentials as one option).
- RFC 9420, *The Messaging Layer Security (MLS) Protocol*.

---

*This document was drafted with substantial AI assistance. The architecture, the decisions, and the mistakes are the author's.*
