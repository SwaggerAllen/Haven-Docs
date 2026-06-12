# Haven open questions

**Status: living ledger.** This document tracks the specific questions Haven’s design currently cannot answer on its own — questions that need expert eyes, practitioner experience, or research that doesn’t exist yet. It is published as part of the design-in-the-open methodology: the five open problems in the introductory series are the strategic statement of what’s unsolved; this ledger is the operational decomposition, maintained as questions are asked, answered, and closed.

How to read an entry: each question has an ID, a status (**open** / **asked**, with a link to the public thread where applicable / **answered**, with a link or summary / **closed**, with the resolution), and where one exists, a link to a review brief — a short standalone writeup of the construction, the claimed property, and what’s already been considered. Questions marked **(research)** are ones we believe could support standalone publishable work; if one of these is in your field and looks interesting, the design documents are the case study and we’d welcome the collaboration.

If you can answer any of these, or can point to prior art we’ve missed, we want to hear from you — publicly or privately. Answers given privately are summarized here only with permission, attributed or not as the answerer prefers.

-----

## Franking and abusive-content reporting

Haven adopts asymmetric message franking (the Grubbs/Lu/Ristenpart 2017 through Namavari/Ristenpart 2024–25 line) for verifiable abuse reporting in E2EE contexts. The composition questions:

- **CR-F1 (research).** Composition of asymmetric franking with MLS PrivateMessages: commitment placement within the AEAD payload, choice of committing AEAD, and whether the composition reintroduces any invisible-salamanders-class behavior. *Status: open; brief in preparation.*
- **CR-F2.** Franking over a sender-key broadcast layer riding on MLS-distributed keys — is there a cleaner construction than per-broadcast commitments? *Status: open; brief in preparation.*
- **CR-F3.** Key-reveal escalation granularity for broken-franking recovery: per-message reveal vs. epoch-key reveal vs. exporter-derived moderation keys, and each option’s interaction with MLS forward secrecy. *Status: open.*
- **CR-F4.** Multi-layer verifier-key delegation (the same report verified at community, alliance, instance, and federation layers) and ephemeral jury re-verification without long-term key holding — are there known patterns? *Status: open.*
- **CR-F5.** Does transcript franking add value when the transport already provides authenticated per-mailbox total ordering, or are we over-building? *Status: open.*
- **CR-F6.** State of the art on the all-malicious-group reporting gap — anything beyond governance/infrastructure-layer handling since the 2019–2025 papers? *Status: open.*

## Anonymous credentials (BBS)

Haven’s identity layer is a single BBS credential blind-issued over a committed secret `pid`, consumed by per-verifier pseudonyms, voting/ticketing nullifiers, and credential linking.

- **CR-B1 (research).** Does a single BBS credential, blind-issued over a committed secret `pid` plus disclosable attributes, securely support *simultaneously*: (a) selective disclosure, (b) per-verifier pseudonyms unlinkable across scopes, (c) nullifier soundness within scopes — with the issuer never learning `pid`? Plus device authorization as a bound value inside the proof. Fraunhofer ePrint 2025/824 is the closest reference composition we’ve found; part of the question is whether it’s the right reference and where our variant differs dangerously. *Status: open; brief in preparation.*
- **CR-B2.** Credential-linking integration surface: does the CFRG BBS proof API as drafted expose cross-credential shared-attribute equality directly, or does it require a hand-composed Sigma conjunction over a shared blinded scalar? (Technique confidence is high; this is a spec-surface question.) *Status: open.*
- **CR-B3 (research).** Unlinkable attendance-count proof: prove “attended at least N attested in-person events within a trailing window” across communities, without cross-community linkability, composed from shared-`pid` linking and scoped nullifiers. This anchors Foundation-level governance counting; it is a new construction question, not just a composition check. *Status: open; brief in preparation.*
- **CR-B4.** Recount-publication linkability: if a per-scope counting authority publishes the full pseudonym→count multiset so any member can recount a tally, does publication create correlation surface across scopes? *Status: open.*
- **CR-B5.** Blind-issuance soundness in Haven’s issuer topology: confirm a participating issuer (instance alliance for humanness; home community for membership credentials) cannot derive or forge an existing member’s pseudonym. *Status: open.*
- **CR-B6.** Issuer-hiding via the BBS-native randomizable-key line (ePrint 2026/369, /555, /870): public verifiability and composition with per-verifier pseudonyms and nullifiers. *Status: open; deliberately deferred (Phase 6) — tracked, not urgent.*
- **CR-M7.** Threshold issuance vs. short-lived freshness credentials: if freshness reissue is a genuine threshold operation, human coordination sits on the hot path; if it’s delegated to an automated signer, the threshold property is decorative for the high-frequency path. Candidate resolutions (threshold-at-root-only; FROST-style machine-held shares; longer lifetimes) need a worked trust comparison. *Status: open; internal comparison in progress, then external review.*

## MLS composition and group-messaging internals

- **CR-M1.** In `draft-mcmillion` partition derivation, is epoch-in-Context redundant with exporter epoch-scoping, or deliberate explicit binding? (Relevant to our capture-forward key-derivation discipline.) *Status: open.*
- **CR-M2.** Fork detection and recovery in a single-sequencer deployment: epoch-authenticator divergence attested to witness-cosigned transparency logs as fork proof, and the minimum incident-scoped state retention that preserves the most forward secrecy during reconciliation. *Status: open; brief in preparation.*
- **CR-M3.** Per-epoch anonymous write-capability keypair construction and rotation (exporter-derived), composed with a blind delivery hub. *Status: open; brief in preparation.*
- **CR-M4 (research).** Institutional archive-key construction: threshold decryption vs. proxy re-encryption for a deliberately-FS-broken archive, and rotating the archivist set without re-encrypting the whole archive. *Status: open; brief in preparation.*
- **CR-M5.** BBS device authorization composing with MLS leaf signature keys: what is the construction for a device-authorizing key bound to a pseudonym that is itself a group element rather than a signing key? *Status: open.*
- **CR-M6.** MLS at ~10,000 members (the Foundation Alliance group): real-world commit and tree-operation performance at that scale, and the fitness of current library options for external-credential validation hooks. *Status: open.*

## Federation and distributed systems

- **FED1.** Haven runs a blind delivery hub — hub-and-followers topology without hub state-tracking, a knowing divergence from the MIMI mainline, with the in-person join model removing the need for hub GroupInfo reconstruction. What are we underestimating about the maintenance and interop cost? *Status: open.*
- **FED2.** Single-authoritative-writer liveness: failover patterns for operator *disappearance* (not hostility) — detection thresholds, degraded read-only service from allied replicas during an authority gap, and what’s automatable vs. what requires representative ceremony. *Status: open.*
- **FED3.** The replica-server pattern (public-facing replica URL, member-facing home URL) is analogous to AT Protocol’s PDS/relay split. What has separating authoritative hosting from content distribution cost in practice, and how should thundering-herd events against small origins be handled? *Status: open.*
- **FED4.** Cross-mailbox causality commitments (credentials valid at presentation time, alliance composition at tally time, schism preserving in-progress operations, etc.): what breaks these commitments in deployed federated systems? *Status: open.*
- **FED5.** Hub-visible traffic analysis in E2EE group messaging: known transport-layer mitigations short of full onion routing, and the current state of the art for bounding what a blind-but-traffic-observing hub learns. *Status: open.*

## Transparency logs

- **LOG1.** Rust implementation path for Static-CT-API-style logs: existing crates vs. porting the Sunlight/TesseraCT patterns — what prior art exists? *Status: open.*
- **LOG2.** Witness-network participation for a non-WebPKI use case (instance equivocation defense, replica integrity attestation): policy or capacity concerns from witness operators; precedents beyond CT and Sigsum. *Status: open.*
- **LOG3.** Operational cost reality at Haven’s shape: many small logs vs. few large ones, and temporal sharding at sub-WebPKI scale. *Status: open.*

## Governance and civic practice

- **CIV1.** Has anyone validated governance-rule parameters (vote thresholds, schism thresholds, phase-dependent tuning) via agent-based simulation rather than intuition? We are considering building such a framework and would rather extend existing work. *Status: open.*
- **CIV2.** What kills deliberation platforms *after* adoption — practitioner post-mortems beyond the published record, and specifically whether governance-feature complexity contributed. *Status: open.*
- **CIV3.** Membership-weighted vs. entity-weighted federated governance in practice: lived experience and observed gaming patterns. *Status: open.*
- **CIV4.** Organizations that tie governance weight to verified attendance (unions, co-ops, congregations): what proxy and absentee mechanisms proved necessary, and what participation discount do disabled, homebound, and dispersed members actually experience? *Status: open.*
- **NEW5 (research).** Cost-of-attack analysis of attendance-anchored governance counting: the claim is that anchoring counting to attested in-person attendance converts sybil attacks from key-minting into operating-real-humans-through-real-events, with tenure vesting forcing the slow version. This is a mechanism-design claim that deserves security-economics scrutiny, not just cryptographic scrutiny. *Status: open.*

## Usable security and trust & safety

- **NEW1 (research).** Known failure modes of in-person QR key ceremonies at community scale, and of asking non-expert users to make per-message disclosure and escalation decisions (key-reveal, report-or-not). The design leans heavily on ceremonies; the usable-security literature studies exactly where ceremonies fail. *Status: open.*
- **NEW2.** Case-load realism for volunteer jury moderation: the franking pipeline produces verified-evidence packages for juries and councils — what evidence-package design and case-flow shape has and hasn’t worked under real abuse volume? *Status: open.*
- **NEW3.** In-person-rooted joining and accessibility: the in-person gate is a deliberate sybil-resistance choice with a real cost for disabled, rural, and immunocompromised people. What accommodation patterns preserve the gate’s properties while reducing the exclusion? *Status: open.*

## Operations and economics

- **NEW4.** What does running a federated-network instance actually cost, in money and hours, at various scales — and what made operators quit? Years of fediverse operational experience exist; Haven’s operator-plurality assumption depends on getting this right. *Status: open.*

## Legal and organizational

These will largely be answered through professional engagement rather than open correspondence, but the questions themselves are part of the design record:

- **LEG1.** Founder-binding enforceability: does the two-entity structure (public-benefit corporation plus charitable foundation) legally hold against the capture and failure modes documented in the founder-binding analysis? *Status: open.*
- **LEG2.** Legal exposure of instance operators in jurisdictions hostile to E2EE, and what the platform must provide operators to make operation viable. *Status: open.*
- **LEG3.** Mandatory-reporting obligations for a blind operator: how verified-report workflows (franking openings supplied by recipients) map onto reporting duties when the operator cannot decrypt content. *Status: open.*
- **LEG4.** What a pseudonymous steward of a public-benefit corporation can and cannot sign, represent, and publish under. *Status: open.*
- **LEG5.** Community treasuries and money-transmission exposure across the phase plan. *Status: open.*

-----

## Lifecycle

Answered questions get their resolution captured here (with thread links for public answers, permissioned summaries for private ones) and folded into the study notes and, when validated, the architecture and feature documents. New questions are added as design work and answers surface them. Questions that turn out to be wrong questions get closed with a note saying why — that’s part of the record too.