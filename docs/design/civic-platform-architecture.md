# Haven: Architecture

*Civic infrastructure for the digital age*

This document describes the architecture of Haven — a federated, end-to-end encrypted platform for civic communities. It captures the design’s principles, the entities and relationships that compose the system, the cryptographic and governance choices that make it distinctive, and the current state of design work.

This is design documentation, not a formal specification. It reflects current settled direction. Items still in active design are clearly marked. A note on process: this documentation was drafted with substantial AI assistance. The architecture, the decisions, and the mistakes are the author’s. Companion documents cover the [whole-system governance framework](haven-whole-system-governance.md), [organizational structure](haven-organizational-structure.md), [federation readiness commitments](civic-platform-federation-readiness.md), [founder-binding analysis](haven-founder-binding-analysis.md), [implementation-level feature specifications](civic-platform-features.md), and the [civic-tech landscape](haven-civic-tech-landscape.md). Active design directions are captured in [study notes](haven-study-notes.md) and [civic-tech design notes](haven-civic-tech-design-notes.md).

-----

## Purpose and principles

Haven is scaffolding for civic culture — infrastructure for the institutions that hold civic life together. Its purpose is to strengthen real-world communities and institutions by providing digital tools that fit how communities actually work, rather than tools optimized for engagement extraction.

The design follows from a few non-negotiable commitments:

**The platform models its trust mechanisms on existing human ones rather than inventing them from scratch.** Communities already have trust — they establish it through in-person introduction, vouching, institutional credentialing, accumulated reputation, governance, and the willingness to split when fundamental disagreements emerge. Haven inherits these mechanisms rather than replacing them. The humans in the system are both the trust substrate and the attack surface, and the architecture treats them as both: structures that work with how communities actually establish trust resist attack better than structures that treat trust as a problem to be solved entirely in software. Sybil resistance and trust were never purely technical problems, and treating them as if they were produces the partial or high-friction solutions the field is full of. The same observation explains the platform’s composability — human institutions already nest and recombine, so a design modeled on them composes for free.

**In-person interaction is the goal.** Digital activity should support and extend real-world community, not replace it.

**Communities are the primary unit.** Individual broadcast is not a first-class operation. People speak through their institutional contexts.

**The platform mirrors how communities organize themselves in the real world.** Governance modules support voting traditions, consensus practices, representative structures, and other patterns communities already use. The platform doesn’t impose a governance model; it provides primitives that real communities configure to match their existing structures.

**The platform does not make policy.** Communities configure their own governance, moderation, credential requirements, and schism rules. The platform provides mechanisms.

**No surveillance, no engagement optimization, no advertising.** Revenue must align with mission.

**Censorship-resistant by architecture.** End-to-end encryption and decentralization make platform-level censorship structurally impossible, not just disfavored.

**Decentralized so no single entity has too much power.** No central operator, no central registry, no single point of capture.

**Plural, not consensus.** The platform supports many incompatible communities coexisting. Disagreements resolve through schism and federation, not through platform arbitration.

**The architecture raises the cost of future operator pivots toward extraction.** Public commitments are backed by structural protections — foundation governance, open source code, end-to-end encryption that operators cannot circumvent without breaking the protocol, payment routing with no platform-controlled intermediate account, math-enabled democracy that distributes decision authority across representative communities. These mechanisms do not absolutely prevent extraction but make it substantially harder than in typical software projects. Comparable projects have made similar commitments and seen them weaken; the founder-binding analysis document examines why and what’s different here.

These principles are architectural constraints, not aspirations. The architecture is designed so that violating them would require breaking the protocol, not just changing a policy.

-----

## Core entities

### Users

Cryptographic identity (keypair). Identity is portable across instances. Users participate in communities by being members. Each user has a designated home community that determines their home instance and provides their primary social affiliation; the home community is the source of community-issued decency credentials. Humanness — the universal sybil-resistance credential that roots per-group pseudonyms — is issued separately by the instance alliance (see Issuers below).

### Communities

The social and governance unit. A group of people with shared rules, governance, and (typically) a real-world basis for membership. Communities have cryptographic identity (their own keypair, separate from any individual user’s). Communities are pseudonymous to instance operators — operators host opaque encrypted communities and cannot inspect their content or category.

### Instances

The operational/hosting unit. A server (or server cluster) that hosts communities and routes federated traffic. Instances are commodity infrastructure. An instance can host any number of communities. Operators have minimal visibility into what they host.

An instance is composed of two related entities: an operator entity (legal and technical authority) and an instance alliance (governance layer composed of representative communities). This split separates infrastructure decisions from governance decisions.

### Alliances

Explicit mutual-aid pacts between communities. Alliances are group entities with their own governance, cryptographic identity, and content. Alliances can share moderation decisions, shunning lists, credential trust, and event calendars. Alliances are distinct from federation — federation is between instances, alliance is between communities. Alliances can themselves be members of other alliances, supporting hierarchical structures.

The instance alliance is a specific kind of alliance — the alliance of representative communities for an instance. The Foundation Alliance is the platform-level alliance whose members are entities (communities or sub-alliances) meeting the 0.01% population threshold.

### Federation

The protocol-level relationship between instances that allows them to interoperate. Federation is established when users on one instance have memberships in communities on another instance, or through explicit instance-level configuration. Federation is direct rather than transitive — instances route only to peers they directly federate with.

### Registries

Locality-keyed or interest-keyed directories of communities. Registries are specialized alliances where listing equals membership. Multiple registries can exist per locality with different curatorial perspectives. Registries are pluralistic; no single registry is authoritative.

### Issuers

The platform separates two kinds of credential issuance, with different issuers for different scopes:

**Decency issuers** are communities. Every decency issuer is a community; not every community is an issuer. Communities attest to facts about their members (membership, residency, role, attendance, good standing, vouching) through cryptographically verifiable credentials. Issuance is a function of community life — a church verifying its own members is just part of being a church.

**The humanness issuer is the instance alliance.** Proof of humanness is universal (its job is cross-community sybil resistance) and is built from external attestations (ID card, government or external-issuer credentials) that have nothing to do with any specific community. Routing humanness through a community would force every community to trust one community’s verification rigor. Instead, the instance alliance issues a humanness credential by attesting to external proof. The instance alliance’s durable cryptographic identity (which survives operator changes and URL changes) means the humanness credential survives those changes too.

This split reflects a broader principle: humanness is universal, decency is local. The humanness credential roots BBS per-verifier pseudonyms (which derive from the secret it attests over) and anchors the people-vote nullifier (one humanness credential per human → one governance footprint regardless of how many personas the human operates).

Humanness is issued by a single signer per instance alliance, not a threshold of representatives. The property that no single party can unilaterally inflate the human population is carried not by splitting the signing key but by three other layers: issuance-rate transparency (anomalous issuance is publicly visible in the verifiable logs), attendance-anchoring of governance counting (minted credentials carry no Foundation-level weight without attested in-person attendance), and acceptance policy (every relying entity chooses which issuers to honor and can revoke acceptance or decertify an issuer). A single logged signing key is in fact *more* accountable for the decertification lever than a diffuse multi-party group — one responsible party, one key to revoke. Splitting the signing key across a threshold of representatives would put a signing ceremony on the credential-freshness hot path at user-population frequency, a cost not worth paying when detection-and-consequences already carries the no-unilateral-inflation property.

**Humanness is a propagated assertion, not a cryptographic guarantee.** At the bottom of every humanness credential is an offline human act — someone physically confirmed a real person. The credential chain transmits that assertion faithfully but cannot manufacture it, and across federation the trust degrades: accepting another community’s or alliance’s humanness credentials means trusting *their* verification practice, sight unseen. At the limit, federated acceptance proves only that the issuing party vouches for its members, not that each credentialed member is a distinct human. There is no cryptographic fix for this — the thing being trusted is an unwitnessed real-world act. The mitigations are all soft or economic: acceptance policy, issuer reputation, and consumer-side attendance-anchoring wherever humanness drives anything security-critical. This is the architecture’s honest floor for sybil resistance, stated once here and relied on elsewhere.

### Haven Foundation

The 501(c)(3) nonprofit entity that holds platform-level authority. Foundation board composition equals Foundation Alliance representatives — the foundation’s legal authority and the Foundation Alliance’s cryptographic authority reflect the same decisions by the same people. The foundation operates public infrastructure (transparency logs, default credential trust framework, certification programs) and certifies operators and plugins.

### Strutco

The Public Benefit Corporation building Haven. Strutco’s commercial activities (instance operation, professional services, technical development) fund the project; the foundation governs the project. Strutco does not have unilateral authority over platform direction — that’s the Foundation Alliance’s role. The relationship is described in the organizational structure document.

-----

## Cryptographic architecture

### Identity

Users and communities have keypairs. These are durable identities; instance addresses are mutable pointers. Identity is portable across instances and survives operator changes.

Cryptographic identities are unique by construction — no reuse, no retirement mechanism needed. Abandoned communities’ identities remain abandoned; new communities get new cryptographic identities. The addressing scheme `(cryptographic_entity_id, mailbox_role)` makes reuse structurally impossible because each entity has its own cryptographic identifier. This avoids confusion about whether content under an identifier is from the original or a reused entity, and avoids the equivocation surface that a name-to-key mapping authority would create.

### The two-layer alliance design

Alliances use two distinct cryptographic protocols for different purposes:

**Governance layer (representative MLS group).** A standard MLS group whose members are designated human representatives of the alliance’s constituent communities. Restricted to representatives, provides full MLS guarantees (forward secrecy, post-compromise security, deniability), and handles confidential governance work — proposals, threshold multi-signature on alliance decisions, sensitive coordination among representatives.

**Messaging layer (sender-key protocol).** A sender-key based protocol (Megolm-style, likely via vodozemac) for broader alliance content visible to all members of constituent communities. All members of all constituent communities can receive the alliance group key; the goal of this layer is broad participation, not confidentiality from members.

This composition uses two well-understood protocols rather than novel cryptography. Earlier design explored a single MLS group with communities as leaves; that approach required protocol extensions we cannot validate without significant cryptographic research. The two-layer approach uses validated protocols throughout.

The sender-key layer provides:

- Confidentiality at the alliance boundary (non-members cannot decrypt)
- Authentication of sender (signed by sender’s identity key)
- Per-message forward secrecy through per-sender ratchets
- Bounded key compromise impact through scheduled key rotation

It does not provide confidentiality from alliance members (any member can decrypt and leak). This privacy degradation matches the social reality — when many people in an alliance can read content, secrecy from members is not a guarantee any cryptographic protocol can deliver in practice. The protocol provides what’s actually meaningful (boundary confidentiality, authentication) without claiming what it can’t deliver.

### Threshold multi-signature for alliance authority

Alliance governance decisions require threshold multi-signature (default 2/3) from representatives. Each level of the platform that has governance authority (community, alliance, instance alliance, Foundation Alliance) operates through this pattern. Authority is exercised through cryptographic operations rather than through trusted intermediaries.

Specific threshold signature scheme (multi-sig vs FROST-style aggregation) is an implementation decision pending cryptographer input.

### Anonymous credentials

The platform uses BBS — a standardized family of pairing-based credential schemes (CFRG drafts including `bbs-signatures`, `bbs-blind-signatures`, `bbs-per-verifier-linkability`, plus W3C `vc-di-bbs` for VC integration). What the docs previously called “BBS+” is, in current standardization, BBS — a proof-tightened simplification.

BBS provides three properties the platform relies on:

- **Selective disclosure**: a credential signs multiple attributes; the holder can disclose any subset while keeping the rest hidden, with constant-size signatures.
- **Unlinkable presentations**: each presentation is a zero-knowledge proof of credential possession whose proof values are indistinguishable from random. Two presentations of the same credential are mutually unlinkable.
- **Per-verifier pseudonyms**: a credential includes a secret holder identifier (`pid`). The holder can derive a deterministic pseudonym scoped to any verifier/scope identifier such that pseudonyms are constant within a scope (recognizable across visits) but unlinkable across scopes (no observer can correlate the same holder’s activity across different scopes).

BBS was chosen over alternatives (CL/Idemix is older and heavier; Pointcheval-Sanders is cryptographically comparable) on ecosystem grounds — the extension drafts the platform requires (blind issuance, per-verifier-linkability) are concentrated in the BBS family, and W3C VC integration is settled. Compose validated primitives; don’t invent crypto.

#### Two-credential model: humanness root plus humanness-freshness

The platform separates long-lived humanness attestation from short-lived freshness proof:

- **Humanness root credential.** Long-lived, instance-alliance-issued, tied to the holder’s external proof check at issuance. Never presented directly. The `pid` is committed blindly at issuance — the instance alliance attests over a committed value without learning it (blind BBS issuance). The root is the foundation from which the holder’s per-verifier pseudonyms are derived.
- **Humanness-freshness credential.** Short-lived, instance-alliance-issued, reissued from the root over the same `pid`. The cheap reissue path — proof-of-possession of the root, no external proof re-check. Presented as proof that the holder is currently a humanness-attested user of this instance.

The combination handles revocation without an accumulator. Revoking a user means refusing to reissue their humanness-freshness credential; once the current short-lived credential expires, no presentation can succeed. Revocation latency equals the freshness credential’s lifetime, which is tunable. No accumulator, no heartbeat, no renewal round-trip — the freshness credential itself carries the freshness.

W3C Bitstring Status List, the current standard revocation mechanism for verifiable credentials, is unusable for Haven because it breaks unlinkability through per-credential indexes and issuer-contact-at-check. Unlinkable accumulator alternatives exist but are early-stage; the expiry-as-revocation approach avoids the dependency.

#### Decency credentials and credential linking

Decency credentials (membership, role, attendance, good standing, vouching) are issued by communities to their members. Each decency credential commits the same holder `pid` that’s bound in the holder’s humanness root.

At presentation, the holder produces one combined BBS proof: humanness-freshness + relevant decency credential(s) + a zero-knowledge equality proof showing both credentials commit the same `pid`. The verifier learns the holder is freshly humanness-attested *and* a member-in-good-standing of the relevant community, all bound to the same person, without learning the `pid` itself.

Credential-linking through shared-`pid` ZK equality is a well-understood BBS pattern (prove a common attribute equal across two credentials without revealing it). It serves both the revocation mechanism (above) and the eligible-secret-voting use case (below).

#### The three consumers: identity, voting, ticketing

Per-verifier pseudonyms (`pseudonym = f(pid, scope)`) serve three platform functions through varying the scope and the anchoring `pid`:

- **Identity (per-group pseudonyms).** Scope = community. Anchor = shared `pid` from humanness root. Within-scope linkability is the feature (members recognize each other in the community); cross-scope unlinkability prevents tracking across communities.
- **Voting nullifiers.** Scope = decision identifier. Anchor = fixed humanness `pid`. The fixed anchor is non-negotiable: a persona-anchored nullifier would let a human with multiple personas double-vote. One person, one vote, regardless of how many personas the person operates.
- **Ticketing nullifiers.** Scope = event identifier. Anchor = configurable along the sybil-tolerance gradient (account `pid` for permissive “one per identity” limits; phone-verified `pid` for stronger limits; decency `pid` for “one per community member”; humanness `pid` for strict “one per person”). Each community configures its own sybil tolerance per event.

Cardinality caps (one-per-person voting, N-per-person ticketing) are enforced by application-layer counting per scope rather than by indexed nullifiers with range proofs. Each scope has a single counting authority (the tally for a vote; the ticketing system for an event) that sees all presentations for that scope; the authority keeps a counter keyed by pseudonym and rejects past the cap. Voting (cap 1) and ticketing (cap N) become one construction. This works because each scope routes through exactly one authority — the same single-authority-per-mailbox principle the federation work relies on.

For secret ballots specifically, eligibility-linking is the complex case: the ballot must prove “eligible member who hasn’t voted” while revealing neither the identity pseudonym nor the `pid`. The shared-`pid` ZK equality between decency-eligibility credential and humanness-nullifier handles this — the same credential-linking primitive that supports revocation. Voting baseline is attributable; ballot-content-secrecy is a Phase 6 late addition for cases requiring coercion resistance.

#### Issuance privacy and observability

What each party sees, accounted for layer by layer:

- **Operator** sees neither issuance events (encrypted through MLS channels) nor presentations (encrypted through MLS channels).
- **Issuer at issuance** does not see `pid` (blind BBS), but the **humanness issuer necessarily sees a real-world anchor** for sybil-resistance dedup — typically an external credential check (ID, government attestation). This is the price of universal sybil resistance. Mitigations: the platform stores only one-way ID-hashes (never plaintext); pushing dedup to the external issuer via a nullifier (if/when external credentials support it — EUDI wallet, eIDAS) is the cleanest direction; the dedup registry is held by the single humanness signer, whose issuance rate is publicly logged so anomalous issuance is detectable and the issuer is decertifiable (see Issuers — the registry’s integrity rests on detection-and-consequences rather than on splitting the signing key).
- **Verifier at presentation** sees the proof, any selectively-disclosed attributes, and the per-scope pseudonym. Verification is offline — the issuer is not contacted at presentation time (BBS property, supporting the federation commitment that credentials are valid at presentation time, not at current-state).

The irreducible residual: humanness issuance touches a real-world anchor; cannot be hidden cryptographically because dedup against the human-anchor space is what makes sybil resistance work. The platform minimizes the surface (one-way hash, presentation-time invisibility, cleanest external-credential nullifier path when available) but cannot eliminate it.

#### Migration and revocation

Revocation against a credential-bearer who has left the instance is handled through the ID-hash:

- The instance maintains a one-way ID-hash for each humanness root it has issued. The hash serves as both the dedup key (one humanness per person) and the revocation key.
- When a user is revoked, the instance publishes the user’s ID-hash to a forward-only revocation log (carried on the platform’s existing verifiable-log infrastructure).
- At migration to another instance, humanness re-issuance is non-anonymous by design (the user re-presents external proof so the new instance can re-attest over the same blinded `pid`). The new instance checks the presented ID against the revocation log. Present → refuse. Absent → re-issue.

Per-presentation unlinkability is untouched. The revocation log is consulted only at the migration/re-issuance boundary, never during ordinary presentations.

#### Post-quantum horizon

BBS is pairing-based (BLS12-381) and not post-quantum. No standardized post-quantum anonymous credential scheme currently exists with the required properties (selective disclosure plus pseudonyms plus blind issuance in a PQ-secure construction is research-stage; the standardized PQ work has not yet reached anonymous credentials).

Implication: the credential layer is built behind a clean replaceable interface from the start. Replaceability is a design *requirement*, not an afterthought. When PQ anonymous credentials mature, the platform replaces BBS at the credential layer without disturbing the rest of the architecture. The MLS layer has its own PQ migration path (PQ/hybrid ciphersuites plus reinit); the credential layer does not, and the architecture acknowledges this asymmetry rather than hiding it.

This commitment is captured as Phase 6 (late protocol additions) and is an active watch item.

### Message franking for verifiable reporting

Abuse reporting uses message franking — a cryptographic technique that resolves the tension between verifiability and deniability. Senders produce signed commitments to message content during encryption; recipients receive openings that let them prove content authentically came from claimed senders; reports include openings allowing verification; without openings, commitments prove nothing, preserving deniability outside the reporting flow.

Composition by context:

- **Community chat, working groups, DMs:** asymmetric + sealed-sender + transcript franking. Recipient pseudonymity preserved; reports survive forwarding to higher governance layers; pattern reporting works.
- **Alliance broadcasts:** asymmetric + transcript franking. Sender attribution at community level (communities are the unit of alliance accountability).
- **Official encrypted channels:** asymmetric only. Senders attributed by default; transcript binding less essential for formal channels.

Verification is separated from enforcement. Verification keys are held by stable infrastructure at each governance layer. Decision-making bodies (juries, moderators, councils) receive verified-evidence packages and act on them through appropriate governance. Juries don’t hold long-term keys; verification can be re-performed at higher governance layers for appeals.

Broken franking is itself reportable. When a malicious sender produces bogus commitments, recipients can escalate by revealing the per-message decryption key, allowing infrastructure to bypass broken outer franking. Implementation details (committing AEAD primitive choice, key-reveal granularity, composition with MLS forward secrecy) are pending cryptographer review.

This governance-and-moderation-over-MLS approach builds on published work rather than inventing the category. PolicyKit (Zhang et al.) established programmable governance for online communities in the plaintext setting; MlsGov (“Private Hierarchical Governance for Encrypted Messaging,” Namavari, Wang, Ristenpart et al., IEEE S&P 2024) demonstrated private hierarchical governance layered on MLS, with a working prototype. Haven sits in the MlsGov quadrant and extends it toward federation, communities as first-class entities, schism, and a blind (rather than semi-honest) operator model — the lineage runs PolicyKit → MlsGov → Haven. Transcript franking, which Haven uses for ordering-and-contiguity assurance in reports, follows Namavari & Ristenpart’s 2025 transcript-franking work; whether Haven’s single-sequencer transport simplifies that construction is an open review question.

The franking design is captured in detail in the study notes.

### Verifiable logs

Multiple platform functions require verifiable append-only logs — equivocation defense (preventing instances from presenting different state to different peers), replica integrity attestations (preventing selective withholding), audit trails for moderation actions and governance decisions, possibly franking verification.

Haven adopts the C2SP tlog-witness protocol for witness coordination, making Haven’s logs interoperable with the broader transparency log ecosystem (Certificate Transparency, Sigsum, others). Haven participates in the existing witness network rather than building parallel infrastructure. Temporal sharding (six-month log shards) bounds log growth, following the pattern established by Sunlight and TesseraCT in the WebPKI context.

Log operation is open; Foundation certification is trust signaling, not gating. Anyone can operate verifiable logs. The Foundation Alliance certifies logs that meet specific standards (availability, audit, governance, witness participation). Certified logs get default discoverability and trust through Foundation infrastructure; uncertified logs can be used through explicit reference. This distributes operational responsibility while preserving meaningful trust signals.

Reference implementations (Sunlight from Filippo Valsorda / Let’s Encrypt, TesseraCT from Google) implement the Static CT API and serve as design references. Specific Rust implementation path is pending investigation.

### Key derivation and capture-forward discipline

Multiple platform functions derive material from MLS exporter secrets — partition keys for routing, sender-keys for alliance messaging, sub-group PSKs for confidential roles, franking key material, and more. These derivations share a discipline that follows from a single load-bearing fact: anything derived from a group’s exporter secret is computable by every member of that group, and the exporter secret itself dies with each epoch’s commit.

**One derivation tree, versioned-category labels, instance discriminator in context.** Derivations use registered versioned category labels (`haven.partition.v1`, `haven.senderkey.v1`, `haven.franking.v1`, `haven.subgroup-psk.v1`) with stable internal identifiers as the context parameter. Version tags in labels provide the migration hook for protocol versioning. Two-step derivations (community exporter → community-alliance key → alliance sender-keys and partition keys) are expected.

**The exporter is overloaded, so the exporter secret is never retained.** All consumers (sender keys, partition keys, franking, sub-group PSKs) derive from the same per-epoch exporter secret. Domain separation prevents cross-derivation but gives zero isolation under root compromise. The exporter secret is strictly epoch-ephemeral and never retained.

**Capture-forward-never-re-derive.** This is the general principle that follows: the exporter yields per-epoch material that dies with the epoch; anything that must outlive the epoch is captured forward at production time, never re-derived. The archive captures ciphertext forward when retention is configured. Franking captures the opening at send time, so verification is opening-based and FS-consistent. Partition references retain the derived routing label, never the secret. Nothing outliving an epoch is recovered by retaining the exporter.

The discipline unifies retention, franking, and partition reference mechanisms under one principle and prevents the footgun of treating exporter secrets as long-lived material.

### Confidential sub-groups and the role-write enforcement model

Some feeds need confidentiality from non-role-holders (treasury, council, pastoral care) or write-role enforcement that the server can actually verify (announcements, creator-audience feeds). Both needs are addressed by the same mechanism: branched sub-groups.

**Sub-groups via branching with resumption PSK.** RFC 9420 supports branching — creating a new MLS group with a subset of an existing group’s participants. The new sub-group injects the parent’s resumption PSK at creation, which cryptographically proves members were in the parent at the branch epoch. This is the confidential-role mechanism: a publisher sub-group, a council sub-group, a pastoral-care sub-group each exists as its own MLS group with its own exporter, its own partition keys, and its own write-cap. The sub-group’s existence is not visible to non-members; only members can derive its routing identifiers.

(Reinitialization — closing a group and recreating it with the same members under different parameters — is a separate primitive used for whole-group migration, including the eventual ciphersuite/post-quantum migration. It is not the sub-group mechanism.)

**Parent Remove must fan out to sub-groups.** Branching has no continuing effect on parent or sub-group: the resumption PSK proves members were in the parent at the branch epoch, not that they still are. Keeping sub-group membership consistent with parent membership is an application-layer obligation — a parent Remove must trigger a Remove fan-out into every sub-group the persona belonged to. Forgetting this invariant leaves ex-members with council or treasury access.

**Role-write enforcement: anonymous outer signature plus inner persona signature.** The server cannot enforce role-based write access by inspecting content (E2E encryption hides what would need to be inspected); enforcing nothing about content authorization is correct for an untrusted-server model. But the server can enforce that *writes to a partition come from authorized writers* by verifying a signature against a write-capability public key derived from a writer sub-group’s exporter. Two signatures, two jobs:

- **Outer write-cap signature** (server-visible): verified by the server against the partition’s write-cap public key (derived from the writer sub-group’s exporter with `haven.write-cap.v1`, rotating per epoch). Anonymous — server learns an authorized writer acted, not which one.
- **Inner persona signature** (reader-visible, inside ciphertext): proves which specific writer for accountability and franking. Server never sees this signature.

The server now does a per-write signature verify, which is an accepted cost in exchange for the write-landing gate.

**Three feed shapes follow from the read/write test.** For each feed, ask: must non-members be cryptographically unable to read? Must non-role-holders be cryptographically unable to write?

- **Open-read / open-write** (general chat): partition over the community group; write-cap = community-membership gate. Any member writes; server blocks non-member writes.
- **Open-read / role-write** (announcements, creator/audience feeds): writers are a publisher sub-group. They encrypt to the community read key (all members read) and sign the partition write with the publisher write-cap (server admits only publisher writes).
- **Confidential-read / role-write** (treasury, council, pastoral care): one sub-group gates both. The role *is* sub-group membership.

**Caveats.** The write-cap is a shared secret among the writer subset; a malicious writer can leak it to an outsider. This is inherent to any shared-key gate; mitigated by per-epoch rotation and by the leaker being an attributable writer through the inner signature. No scheme fully prevents a member proxying writes. Cost: more sub-groups to maintain and rotate, plus per-write server verification.

### Signed state and event sourcing

State changes happen through cryptographically signed messages persisted to a log. Current state is derived by replaying the log. Federation is log exchange between instances. Migration is replaying logs in a new location. Replication is local copies of remote logs. All state-changing operations have a signed authoritative source; no consensus protocol is needed because every meaningful resource has a single authoritative location.

-----

## Content model

The platform uses typed content schemas rather than ActivityPub’s loosely-typed activities. Each content type has a declared schema with typed fields, validation rules, and versioning.

Why typed schemas: different community types need very different content structures. Parishes need homily texts and sacramental records; HOAs need meeting minutes and contractor proposals; artist collectives need exhibition records and commission contracts. Generic platforms force everyone into one shape that fits none well. Typed schemas let communities have content matched to their actual work, with plugins introducing new types as needed.

Core content types include:

- **Events** — time, place, RSVP, attendance verification. The structural anchor for in-person interaction.
- **Offers and requests** — mutual aid coordination. Fulfilled offers and closed loops generate behavioral reputation.
- **Announcements** — read-only or near-read-only, from accounts with that permission in the community.
- **Discussions** — threaded, the closest thing to traditional posts, deliberately not the default.
- **Chat** — connective tissue between structured activities.

The default community feed surfaces events, offers, and requests. Discussions and chat are available but not foregrounded. There is no global feed; users always choose a community context.

### Schema format

Content schemas use JSON Schema (draft 2020-12) with Haven-specific conventions on top. JSON Schema has massive ecosystem support across Haven’s implementation stack (Elixir, React Native, Rust), composition features that handle Haven’s use cases (allOf, oneOf, anyOf, $ref, conditional validation), and well-established versioning patterns.

AT Protocol lexicons were considered and rejected; lexicons provide API surface description (queries, procedures, subscriptions) that AT Protocol needs because of its large schema-driven federation API, but Haven’s federation API is more bounded and operations can be described separately from data schemas.

Haven-specific conventions layered on JSON Schema:

- Content addressing reference types (commitments — franking, MLS epoch authenticators, log tree heads — verified at protocol layer)
- Schema identifier scheme (reverse-domain notation, e.g., `app.haven.events.standard.v1`)
- Schema registry/distribution (Foundation operates baseline registry for canonical schemas; communities can operate registries for custom schemas; schemas can be inline-embedded for one-off cases)
- Version handling (major versions in identifiers, minor versions through additive changes, explicit migration paths for breaking changes)
- Interface satisfaction declarations (see below)

### Feeds defined by allowed content types

Communities declare their feed structure as part of community state managed through governance. A feed declaration includes feed identification (stable internal identifier plus mutable display name), role-based permissions, accepted content types/schemas, and optional relationships to other feeds.

Feeds can be specialized by accepted content type: a short-form video feed accepting only video records under N minutes, a microblogging feed accepting text plus images but not video, a photo feed accepting images only. The feed’s identity is shaped by what it accepts. This lets communities compete with format-focused commercial platforms within civic infrastructure rather than forcing users to leave for specialized formats.

### Interface-based composition for client rendering

Clients cannot be expected to implement every possible schema; the schema ecosystem will grow over time. Old clients need to handle new schemas gracefully without being updated, while preserving rich rendering when the client does support a schema.

Schemas declare which interfaces they satisfy. Interfaces are contracts about field presence and semantics, defined separately from specific schemas. A schema can claim satisfaction of multiple interfaces. Clients implement against interfaces, not against specific schemas.

Canonical interfaces are Foundation-governed and form the standard set. Many small interfaces, granular and composable — each declaring one capability rather than a complete content shape. Initial set likely includes Renderable (title, author, timestamp, body), Discussable (Renderable plus discussion thread), MediaContainer (Renderable plus media references), Geographic (structured location), Timed (start time and optional end time), Reactable (supports reactions/responses).

A schema like `app.haven.parish.homily.v1` might claim `Renderable + Discussable + MediaContainer`. A community event claims `Renderable + Geographic + Timed`. Combinations express what the content supports.

Schemas also declare a fallback type for clients that don’t implement any relevant interfaces. The fallback provides graceful degradation even when interface-based rendering fails. Universal baseline is `Renderable`.

Communities can define custom interfaces, but clients aren’t expected to implement them — communities defining custom interfaces take responsibility for limited client support.

### History and retention

MLS forward secrecy means epoch keys are deleted on schedule and old content is cryptographically inaccessible to anyone who wasn’t a member when it was sent. New members joining a group get nothing pre-join by default — this is forward secrecy working as designed, not a gap. Any history access for new members or new devices is necessarily an application-level mechanism layered on top, with the FS sacrifice deliberately scoped.

Member-facing history is supported through two independent per-feed artifacts. They answer different questions and have opposite default trust postures, so collapsing them into one artifact with two policies fails in both directions: institutional-grade controls smother routine onboarding, or the archive inherits onboarding’s looseness.

**Onboarding context** is the welcome mat — a new member isn’t dropped in with zero context. Short rolling window (last week, last N posts). Self-limiting; content ages out. Access bar is low, since this is what every new member is handed. Available on open-read feeds only; never on confidential sub-groups. Closer to a retention setting on the live feed (“keep the trailing N days decryptable to new joiners”) than a separate cryptographic store. The FS waiver is time-bounded and self-healing — a joiner sees the last week; a week later that content is gone from the onboarding store.

**Institutional archive** is the system of record — retention, recovery, audit. Long to indefinite retention. Access bar is high; every access is audited; reads are exceptional rather than routine. Available for anything retained, including confidential sub-groups. This is where the bounded, governed FS sacrifice from the recovery model lives: content is continuously re-encrypted under a long-lived archive key as it’s posted. The archive key’s decryption authority is threshold-shared among an archivist role; every archive access is logged to the verifiable log infrastructure. Both gating and audit are required — gating without auditing lets a colluding threshold quietly surveil.

The four combinations are all meaningful: neither (high-privacy chat with pure FS), onboarding only (announcements where newcomers get recent context but nothing retained long-term), institutional only (treasury sub-group where joining grants archive eligibility but not an onboarding dump of recent treasury talk), or both (parish main feed).

History status is a disclosed, governed per-feed property — posters need correct expectations about whether their content will outlive the epoch and be visible to people who weren’t present when it was written. The feed structure declaration carries the history configuration alongside its other properties.

### Recovery and the forward secrecy carve-out

Recovery from device or key loss works through a layered model that degrades along a dependency axis. Device handoff (E2E between user’s own devices) works when the user still has one device. User-side backup with social recovery is the self-service disaster path. Peer history provides gap-fill when those fail. Community archive is authoritative when peers aren’t enough.

Forward secrecy protects non-retained content. Retained content (in either history artifact) is protected by access controls and audit, not by forward secrecy. The carve-out reflects a hard limit: no construction gives both “recoverable after losing only device” and “cryptographically unrecoverable after epoch keys are deleted” — these are contradictory. The work is bounding the FS sacrifice and making it visible and governed, not eliminating it.

Recovered history is untrusted plaintext until verified against an independently-attested commitment. Universal replica attestation, franking commitments, and epoch authenticators mean an attested commitment exists for every retained message. Recovery is “obtain plaintext from some holder, verify against attested commitments” regardless of source.

-----

## Identity, reputation, and credentials

The platform’s identity and reputation systems operationalize trust mechanisms communities already use. People meet in person and introduce one another; institutions vouch for their members; reputation accumulates through participation; some credentials require formal issuance and others require only the vouching of an existing member. The cryptography in this section gives these mechanisms verifiable, portable, privacy-preserving form — it does not replace them.

### Sliding-scale pseudonymity

Users may participate with minimal information by default. Each verifiable credential provides a universal reputation bump scaled by the sybil-resistance value of the credential. More verification helps users join communities with higher thresholds; less may be required to participate in low-threshold communities.

Communities choose their own verification thresholds. The protocol makes low-verification participation viable through vouching, so people who cannot or will not verify are not excluded from civic life.

### Universal vs. contextual reputation

**Identity confidence** (universal): derived from verified credentials. Reflects humanness and uniqueness, not behavior.

**Behavioral reputation** (per-community): accumulates within a specific community based on participation, fulfilled offers, attendance, vouches received and given. Does not transfer between communities.

### Vouching

Inviting someone to a community is vouching for them. Vouching has transitive consequences with decay: if a vouchee misbehaves, the voucher takes a meaningful hit; deeper ancestors take diminishing hits. Vouches are part of community behavioral reputation systems.

### Credential ecosystem

The platform’s credentials split between humanness (universal, instance-alliance-issued) and decency (local, community-issued), as described in the core entities section.

**Decency credentials.** Every decency issuer is a community. Issuance is a function of community life — a church verifying its own members is just part of being a church. Dedicated issuers (notary-style) are possible but are also communities with the same governance structures.

Three layers of issuer trust, in increasing order of caution: direct trust (community explicitly accepts credentials from a specific issuer), alliance trust (communities accept credentials from issuers their alliance partners trust), transitive trust (opt-in, with decay; off by default). Communities can also negatively mark issuers — explicit distrust, parallel to shunning.

**Humanness credentials.** Issued by the instance alliance, attesting to external proof of humanness (ID card, government or external-issuer credentials). The instance alliance’s durable cryptographic identity means humanness credentials survive operator changes. Whether *other* instances honor instance-A’s humanness credentials is a cross-instance trust-policy decision using the same direct/alliance/transitive trust machinery — federation makes humanness portable but doesn’t make it automatically trusted everywhere. Universal must not be over-read as “automatically trusted everywhere.”

**Blind issuance.** The humanness issuer attests over a blinded secret intended to remain unknown to the issuer. If the blind-issuance property holds as intended in the chosen BBS construction, a compromised instance alliance could mint false or duplicate humanness credentials (sybils) but could not impersonate existing members’ pseudonyms — which would invert MLS’s worst failure case from impersonation to sybil-minting, with sybils being the bounded, detectable, governance-handled harm. This property depends on the exact behavior of the blind-issuance construction and is pending cryptographer confirmation.

External credentials are supported (Apple Wallet, state ID, university credentials) for community-irrelevant verifications. The humanness credential is itself one such use — external proof attested by the instance alliance. The internal issuer ecosystem handles community-specific credentials.

Physical credential production (NFC member cards, ID artifacts, event credentials, lanyards) is a primary revenue stream and platform service. The physical credential is cryptographically linked to the digital one. Aligned incentives — revenue scales with institutional adoption, not with attention captured.

### Verification levels are not visible within a community

Once you’re a member, you’re a member. Verification level affects what communities will accept you, not how your contributions appear inside them. No checkmarks, no tiers within communities.

-----

## Governance

### Multi-level structure

Authority distributes across multiple layers, each with its own MLS group, its own threshold multi-signature requirements, and its own scope:

- **Community level.** Members govern community-internal matters through configurable governance modules.
- **Alliance level.** Representative communities govern shared matters through alliance MLS group with threshold multi-signature.
- **Instance alliance level.** Representative communities for an instance govern instance-level matters (operator authorization, federation policy, instance-wide rules).
- **Foundation Alliance level.** Member entities (communities or sub-alliances meeting the 0.01% threshold) govern platform-level matters through Foundation Alliance MLS group with threshold multi-signature.

Each level has authority within its scope. Higher levels establish floors; communities can be more restrictive than higher levels require but cannot opt out of higher-level rules.

### Math-enabled democracy

The platform’s governance is structured around the principle that cryptographic mechanisms enforce procedural rules while human judgment applies through them. The math-enabled democracy framework provides:

- People-vote principle: governance weight derives from individual humans, with fractional weight (1/N) for members participating in N communities, except at instance-alliance level where one-share-per-community supports federation verifiability
- Layered authority: each level has scope-appropriate authority
- Lieutenant pattern: subsystem maintainers hold cryptographic authority over their subsystems, with Foundation Alliance confirmation based on contribution attestation

The whole-system-governance document develops this framework in detail.

### Foundation Alliance composition

A Foundation Alliance member must have at least 0.01% of the platform’s institutionally credentialed population as members. This threshold:

- Scales automatically with platform size
- Caps composition at 10000 entities (within standard MLS scaling limits)
- Drives aggregation rather than requiring categorical alliance/community distinctions
- Preserves the people-vote principle through institutionally credentialed member count
- Excludes online-only entities by requiring institutional credentialing

Composition recomputes annually (or at whatever interval the Foundation Alliance configures). Verification uses cryptographic deduplication where possible, with institutional attestation and audit-on-demand as the fallback approach.

Three structural properties protect Foundation Alliance composition against the circularity inherent in self-issued humanness:

- **Instance alliances cannot be Foundation Alliance members.** The humanness issuer must not directly hold governance weight denominated in the credential it issues — that would let an issuer mint weight for itself, the sharpest form of the circular-trust-root problem. Excluding instance alliances forces the path to Foundation weight through a real social alliance (affiliation, not server plumbing — the alliance/instance distinction is load-bearing here): an issuer can mint credentials but cannot convert them to its own governance weight without a socially-grounded community vouching it in.
- **Foundation Alliance membership uses the same requirements and join flow as any other alliance — there is no privileged path.** This guarantees the Foundation Alliance is covered by the same sybil-resistance machinery (the in-person gate, attendance-anchoring, locality-grounded resistance) as every ordinary alliance, with no bespoke admission mechanism to separately secure or attack.
- **The Foundation Alliance is therefore competable-with.** Because no mechanism is privileged, a rival alliance can form with the same structure, governance math, and sybil resistance and compete for legitimacy. The Foundation Alliance’s authority comes not from protocol privilege but from stewardship of the canonical codebase (lieutenants elected over it) and accumulated recognition; a competitor may form but must fork the entire code-and-maintenance apparatus and rebuild legitimacy from nothing — expensive, not impossible. This is the same anti-capture principle as first-class schism, lifted to the alliance layer: the platform makes communities forkable, the trust anchor forkable, and the founder bound by succession — exit is always possible, never free.

Anti-inflation direction (working, not yet settled): because the threshold is denominated in member count, the counting population should be anchored to attested in-person event attendance rather than raw credential holding, with issuance-rate transparency, statistical sampling of claimed members, and tenure-vesting of governance weight as supporting mechanisms. This converts a key-minting attack into a hire-real-humans-for-real-events attack and matches the cost wall real institutions face. The denominator must use the same attested count as the numerator. This direction is developed in the whole-system-governance document and remains subject to validation.

### Configurable governance modules

Communities choose their own governance procedures through governance modules. The platform provides primitives (members, roles, votes, proposals, schism) that compose into many specific governance patterns. Sortition, simple majority, supermajority, consensus, representative structures, and other traditions are all supported through configuration.

Specific procedural systems (Robert’s Rules of Order, Quaker consensus, conclave) are plugin territory rather than platform features. The platform provides what plugins compose into specific traditions.

### Temporal governance mechanisms

Communities can configure governance temporality:

- **Grace periods.** Minimum time between proposal posting and earliest commit, preventing rushed decisions
- **Quorum requirements.** Minimum participation level for vote validity, configurable as absolute number, percentage of eligible voters, or percentage of recently-active members
- **Voting windows.** Predictable periods when voting happens, with proposals accumulating between windows (matches synods, annual meetings, quarterly board sessions)
- **Emergency provisions.** Configurable mechanisms with higher thresholds for time-sensitive matters
- **Recurring proposals.** Scheduled recurring decisions (annual budgets, periodic elections)
- **Per-type configuration.** Different proposal types can have different temporal mechanisms

These patterns can be combined; a community might use voting windows with grace periods within windows and quorum requirements for validity.

### Schism

Schism is a first-class operation. When a controversial decision splits a community, the minority can fork into a new community rather than being forced to accept the majority’s choice. The new community inherits content history, credentials, federation links, issuer trust settings, and per-member behavioral reputation. Majority keeps the name; the schismatic community starts with a placeholder and chooses its own through its governance.

Alliance memberships transfer to the schismatic community by default — both communities are members of the alliances the original belonged to. Alliances can eject the schismatic community through their own governance if they don’t want both, but the default favors continuity. This matters because alliances often reflect deep institutional relationships (denominational membership, sister-organization status) that a schism shouldn’t unilaterally end; alliances retain authority to decide whether to maintain both relationships through their normal governance processes.

Schism resolves minority faction protection without requiring platform arbitration. It handles controversial migrations: members who disagree with a migration form a community before migration happens rather than reverting after.

-----

## Moderation

Community self-governance of moderation is not a policy preference; the encryption forces it. An operator who cannot read content cannot moderate it, cannot be liable for it, and cannot decide what’s allowed — so moderation has nowhere to live except inside the community. The same property, run the other way, is why incumbent institutional software stays broadcast-only: a vendor that can read everything it hosts is liable for member-to-member speech and would have to moderate it at a scale no software company staffs, so it doesn’t host it. Incumbents are trapped on their side of the line by the same architecture that holds Haven on this one; crossing it would mean abandoning the readable data model their products and revenue rest on. (The incumbent analysis is in the civic-tech landscape.)

### Multi-level rule-based moderation

Rules exist at multiple levels (community, alliance, instance, Foundation), each with published rules and authority to act on violations. Rules are text — human-readable criteria moderators apply when reviewing reports, not platform-enforced code.

When a member reports, they see rules applicable at all relevant levels and select which they believe have been violated. The report routes to whichever levels had rules selected. Each level processes its own report according to its own rules. A community cannot suppress reports invoking higher-level rules; the instance and foundation each receive their own copies if those rules are invoked.

### Rules apply prospectively

Rules apply only to content posted while they were in effect. Content posted under an earlier rule set is evaluated against that earlier rule set, not against current rules. This is fundamental fairness — no retroactive enforcement of rules that didn’t exist when content was created — and is structurally enforced through event-sourced rule state. The reporting flow reconstructs the rule sets that applied at the content’s posting timestamp.

### CSAM as universal prohibition

Child sexual abuse material is included in foundation baseline rules and cannot be opted out by any community or instance. Verified reports trigger legally-required NCMEC reporting and federation-wide effect. Operators register with NCMEC as electronic service providers. The platform cooperates with valid legal process within its technical limits — encrypted content cannot be produced because the platform cannot decrypt it.

### Verification via franking

Reports include franking material — cryptographic proof linking them to actual platform content. This prevents fabricated reports while preserving the report itself as evidence. Pseudonymity is preserved across communities except for severe verified violations requiring cross-instance action.

### Guardian role for youth participation

Communities serving minors can configure guardian roles where adult community members have appropriate oversight without surveillance. COPPA compliance, configurable by age, with safety reports always escalating regardless of normal moderation paths.

### What the platform does not do

The platform supports the moderation communities already do — surfacing reports to the appropriate governance bodies, providing verifiable evidence, recording decisions. It does not impose platform-wide content policy or perform platform-level enforcement:

- Server-side content scanning (impossible due to encryption)
- Client-side scanning (rejected on privacy grounds)
- AI-based detection on encrypted content
- Cross-community surveillance of users
- Pre-emptive bans based on patterns alone (always requires verified report with human review)

### Structural friction against criminal use

The platform’s onboarding requirements (in-person QR exchange, locality-grounded vouching, institutional credentialing) come from how civic communities actually form, not from anti-abuse considerations. The friction is incidental to using mechanisms that work for real institutions, not a feature added to deter criminals. The consequence is that criminal enterprises typically choose paths of least resistance: platforms with low-friction account creation (Signal, WhatsApp, Telegram) offer adequate privacy without onboarding cost. Haven offers no additional privacy benefit while imposing significant friction.

The all-malicious-group attack (every member of a group running malicious clients to use Haven for prohibited content) is not cryptographically prevented but addressed through governance and infrastructure: group-level moderation pathways, identity revocation at credential authority layer, pattern detection at infrastructure layer, external cooperation with law enforcement. The platform’s claim, stated precisely: Haven makes abuse expensive, detectable, and actionable; it does not prevent all abuse.

-----

## Federation

### Direct federation model

Federation is direct, not transitive. Instances federate with each other through explicit relationships; messages between non-federated instances are not relayed. This keeps trust relationships explicit and bounded.

Federation is triggered by social structure crossing instance boundaries: users joining cross-instance communities, alliances forming with members on multiple instances, communities migrating between instances. Federation does not form abstractly between instance operators; it always traces back to real social or organizational connection.

Federation establishment happens as a side-effect of the in-person QR-driven invitation flow rather than a separate handshake protocol. The QR code carries identity, instance ID, current URL, community context, and signed invitation. The receiving instance verifies and initiates federation with the inviting instance if not already federated. The first authenticated community-join is the federation establishment.

### Hub instance pattern for alliances

Every alliance has a hub instance where its authoritative state lives. Joining an alliance means federating with that one hub instance. Member communities don’t federate directly with each other through alliance membership; they federate with the alliance hub.

What lives on the alliance hub: representative MLS group state (governance layer), alliance broadcast streams (sender-key messaging layer), governance records, alliance membership records, and other alliance-scoped state.

Alliance hub selection happens when the alliance is formed. The founding communities decide which instance hosts the alliance state — typically one of the member communities’ hosting instances, but could be a dedicated alliance-hosting instance. Alliance migration follows the same pattern as community migration: cryptographic identity is stable; hosting instance changes through alliance governance plus atomic handoff.

This generalizes the hierarchical-structures pattern (alliance of alliances) to all alliances. An alliance of N communities means N federation relationships with the hub, not N×(N-1) between communities. Federation cost scales with the number of alliances an instance participates in, not with the size of those alliances.

Cross-community interaction outside alliance scope doesn’t automatically federate through the alliance hub. Direct cross-community interactions (DMs, shared resources, separate alliance memberships) create separate federation relationships if needed.

### Instance identity

Each instance is identified by its instance alliance’s cryptographic identity — durable across operator changes. Operator transitions don’t change the instance’s federation identity. Federated peers verify operations against the instance alliance identity, not the operator entity directly. The operator is authorized by the alliance through a signed attestation; peers verify the chain.

### Mailbox addressing

Mailboxes have composite addresses: `(cryptographic_entity_id, mailbox_role)`. The cryptographic entity ID identifies whose mailbox it is (community, user, alliance); the mailbox role identifies which specific mailbox for that entity (feed name, “governance”, “dm-with-X”). The authoritative instance for a mailbox is metadata, not part of the address — when a community migrates instances, the address stays the same; only the authoritative-instance pointer changes.

Display names of feeds are mutable metadata; stable internal identifiers (used for cryptographic derivation) handle rename cases.

### Partition keys for routing

Per-mailbox partition keys are derived from cryptographic state — MLS exporter secrets for MLS-based mailboxes, sender-key state for alliance broadcast mailboxes. Partition keys serve as routing identifiers without revealing which mailbox or social entity they correspond to. They rotate with epoch changes, preventing long-term correlation.

A single MLS group per community supports multiple feeds; feed-specific routing happens through different partition key derivations from the same MLS state. The delivery service sees opaque partition keys without learning what they represent.

### The blind hub

The instance hosting a community runs a blind delivery hub. The hub does sequencing (consistent order per partition key) and fanout (pull-based serving to anyone presenting the key). It does not track group membership, does not authenticate joining members at the protocol level, and does not reconstruct group state for offline joiners.

Cryptographic membership enforcement comes from partition keys being derivable only by members plus the per-epoch write-capability (in the role-write enforcement model described above) gating writes. The math, not the operator, enforces what can be enforced.

A state-tracking hub would be needed for exactly one job — reconstructing group state for offline joiners with no existing group relationship. The in-person membership model means every join is in-person-rooted, parent-state-derived, or interactive, so a joiner always has group state through one of these channels. The blind-hub model and the in-person join model are two views of the same architectural decision.

### Cross-mailbox references

References across mailboxes use `(partition_key, commitment)` pairs. The partition key identifies the routing context (active or historical); the commitment identifies the specific content (franking commitment, MLS epoch authenticator, log tree head). After partition key rotation, historical partition keys continue to route to their archived content; cross-references remain valid.

### Three-tier replication

Replicas hold encrypted message logs:

- **Authoritative.** Exactly one instance per mailbox holds the source of truth.
- **Allied full replicas.** Durable, complete replicas held by instance alliances under explicit agreement. Permanent, agreed protection against operator hostility.
- **Cache replicas.** Automatic, partial, decaying replicas held wherever there’s local interest. Performance and resilience.

Each replica publishes signed attestations of holdings (Merkle roots over content), submitted to public verifiable logs (see below). Attestation is uniform across all replica content — no selective application by content type or perceived importance, because selective attestation would leak metadata about which content is considered high-stakes. Universal attestation makes all content look the same to operators, preserving metadata privacy.

### Operator transitions and fork recovery

**Threat model.** End-to-end encryption plus the blind hub mean a hostile operator never threatens confidentiality. The hub cannot read content, cannot learn membership, cannot target specific members or split along social fault lines. The threat collapses to availability (withhold/drop/delay; detected by replica attestations against verifiable logs) and ordering-integrity (fork or rollback; the hard case). Hostile attacks are coarse, not surgical — blindness demotes the hostile operator from surgeon to vandal.

**Operator transition.** Hostile operator recovery depends on representatives having local MLS state copies and reconciling to consistent state when the operator suppresses updates. If representatives have divergent state, they reconcile to the most recent state 2/3+ agree on; threshold of representatives signs transition attestation; new operator takes over with verified authority.

**Fork recovery.** A hostile sequencer can fork a group: deliver different commits for the same epoch to different members, producing two cryptographically-valid divergent epoch chains that cannot be merged. This is inherent to MLS and broader CGKA constructions. The platform’s response is choose-and-re-establish rather than tolerate-concurrent-forks. Fork-tolerant CGKA variants retain key material permanently, which is a standing FS reduction the architecture rejects.

Detection uses epoch authenticators: two different authenticators for the same `(group, epoch)` attested to verifiable logs is cryptographic proof of a fork, attributable to the operator. Recovery: the most recent witness-cosigned uncontradicted authenticator is the resumption point; threshold of representatives signs which branch survives; members stranded on the abandoned branch are re-added to the survivor via fresh commits; content history survives through allied replicas; new hub sequences onward. State retention for fork detection and reconciliation is incident-scoped, not standing — bounded FS softening during the incident window, then deletion.

Operator-induced forks are distinct from schism. Schism is deliberate governance fork (minority branches intentionally, new identity, both communities continue as legitimate groups). Operator-induced fork is involuntary (one community split with same identity, one branch must be abandoned and its members re-merged). The machinery overlaps (branch, re-add) but schism creates two legitimate groups while fork recovery collapses back to one.

Specific protocols for state reconciliation, rollback authentication, and incident-scoped retention timing are pending cryptographer review.

### Replica servers for public content scaling

Replica servers host full histories of public content for client communities through allied replica agreements. The replica server is the public-facing URL; the home server is the member-facing URL. External users interact with the replica server when reading public content; members interact with their home server for everything. The replica server is composed through existing primitives (instances with allied replica relationships, registries as the unit of governance) rather than being a new primitive.

This design direction is in active development — operational details, economic model, and governance specifics are being worked through.

### Public verifiable logs

Foundation operates public verifiable logs as platform infrastructure. Operators submit replica integrity attestations, governance decision audit trails, and federation-relevant state attestations. Witnesses sign log heads to prevent equivocation. Peers verify against public log state rather than trusting operator-provided state.

### Defederation

Defederation is unilateral with local consequences. An instance can defederate from a peer at any time, losing access to communities on the defederated peer for its users.

The federation readiness document captures architectural commitments in detail.

-----

## Home community and migration

### Home community

Each user has exactly one home community. The home community determines the user’s home instance, is the user’s primary social affiliation for cross-community trust signaling, and is the most consequential community for the user.

The home community is distinct from the humanness root. The humanness credential is issued by the instance alliance and roots BBS per-verifier pseudonyms; it is not affected by changes in home community membership *within an instance*. The home community provides social affiliation and is the source of community-issued decency credentials (membership, role, good standing). Changing home community within the same instance is a local credential swap — dropping one membership credential, acquiring another — and does not touch pseudonym continuity or the people-vote anchor.

Users can be members of any number of non-home communities. Joining a community on a different instance creates a federation relationship between the user’s home instance and the community’s instance.

### Community migration

Communities can migrate between instances atomically with both instances participating in handoff. The community’s keypair stays the same; the address changes. Signed migration announcement propagates through gossip. Federation peers and registries update via the announcement.

If members object to a migration, they schism before it happens rather than reverting after. There is no fork-with-same-identity problem because schism creates a new identity for the dissenting community.

When a user’s home community migrates to a new instance, the user’s home instance also changes. Their humanness credential, issued by the previous instance alliance, becomes a cross-instance trust matter. The new instance alliance decides whether to honor the previous instance’s humanness credential through the standard direct/alliance/transitive trust framework that applies to any cross-instance credential.

What this means in practice depends on the relationship between the instances:

- **Migration to an aligned instance.** New instance alliance has direct or alliance trust with the previous one; humanness credentials are honored; users carry humanness through migration with no friction.
- **Migration to an unrelated instance.** New instance alliance has no established trust with the previous one; the alliance decides whether to honor the credentials or require re-attestation. Re-attestation is the standard humanness flow (external proof attested by the new instance alliance over a blinded secret); the user’s pseudonyms persist because they derive from the user-held secret, not the issuer.
- **Migration away from a hostile operator.** The new instance alliance may have specific reason not to honor the previous one’s humanness credentials — for instance, if the previous instance’s operator was compromised or its alliance was issuing sybils. In the worst case this is effectively a humanness wipe: users on the migrating community must re-attest humanness through the new instance alliance. This is a real cost of hostile-operator escape.

The trust framework makes humanness portable in the common case while preserving the new instance alliance’s authority to refuse credentials from sources it can’t verify. Universal must not be over-read as “automatically trusted everywhere.”

Users whose primary social affiliation is more important to them than continuing with the migrating community can shift their home community designation to another community on their current instance before the migration completes. The migration announcement includes a window for this; after the window closes, members who took no action move with the community to the new instance. This matches how real institutional migration works — when a congregation changes denomination, members get to decide whether to stay with the congregation or seek a new one in the original denomination.

-----

## Locality and discovery

### In-person trust as default

Community membership requires in-person QR code or Bluetooth pairing with an existing member. The in-person requirement is non-negotiable for sybil resistance; for members who cannot travel, an existing community member goes to them rather than the membership happening remotely.

The gate is one of exactly two coherent answers to the same problem. Front Porch Forum — the longest-running working analog — spends in the room: a weak, self-asserted door (an address on the honor system) compensated by heavy continuous human moderation, real names, and an operator who reads everything. Haven spends at the door: a strong cryptographic gate compensating for light, reactive moderation, pseudonymity, and an operator who reads nothing. The choices are coupled, not independent — a weak gate requires strong moderation, and a strong gate is what makes light moderation safe. FPF proves the first equilibrium works, at the cost of a trusted operator who reads everything and whose standing doesn’t transfer past one region. Haven’s in-person requirement is the price of the second. (The comparison is developed in the civic-tech landscape.)

Alliance relationships between communities follow the same principle. Two members from two different communities exchange community-handshake QR codes in person to initiate alliance formation; each community then ratifies through its own governance, and the alliance proceeds through an unincorporated phase before incorporating under a chosen template. Joining an existing incorporated alliance also requires in-person community-handshake QR exchange between a member of the joining community and a member of an existing alliance member community — the same in-person grounding that applies to forming the alliance applies to extending it.

### Locality registries

Pluralistic, no central operator. Registries are specialized alliances whose member communities are listed in the registry. Curators (libraries, regional councils, denominational bodies) operate registry alliances and govern membership through standard alliance governance.

Multiple registries per locality are expected. Different curators serve different communities of communities. Communities can be members of multiple registries (diaspora communities, multi-site institutions, mobile communities). Registries can themselves be members of larger registries.

### Privacy in discovery

Registry contents available as public content via standard web requests; no federation required to browse. Communities can be listed without revealing internal membership. Some communities can be invisible — joinable only with specific knowledge, not listed in any registry.

-----

## Revenue model

Platform revenue is bounded by architectural commitments. The architecture makes transaction fees, advertising, and data sales structurally harder than in typical platforms — not absolutely impossible, but requiring visible deviation from established patterns.

### Minimal community dues

Free up to 10 members, modest flat fee scaling thereafter. No progressive pricing that punishes growth. Designed accessible to small community groups while modestly supporting infrastructure costs. The instance operator receives these dues, not the foundation directly.

### Physical credential production (primary revenue)

Production and distribution of physical credentials (NFC member cards, ID artifacts, event credentials, lanyards) linked to digital ones. Aligned incentives — revenue scales with institutional adoption, not with attention captured. Communities order credentials; the foundation produces and ships them.

### Instance operator revenue share

Instance operators may receive a small share of credential production revenue from communities they host, creating a viable business model for instance operation that aligns with platform health.

### Payment processing without platform cut

Payment uses pluggable adapters (Stripe, PayPal, others) for community-owned resources like dues, tickets, and credentials. Money flows from payer to recipient directly through the processor; the platform does not intercept funds. The adapter interface is structured such that there is no platform-controlled intermediate account where fees could be collected.

This is a structural property of the adapter pattern, not absolute cryptographic prevention — a future operator could deploy a modified adapter, but doing so requires breaking the established pattern visibly. Open source code and federation verification make such deviations detectable.

### What the model excludes

- Advertising
- Engagement optimization
- Data sales
- Transaction fees on community commerce
- Premium tiers with platform-level feature differentiation
- Payment processing for community-level commerce outside platform-tracked resources

### The structural floor: bulletin-board vs. attention-auction

The advertising exclusion has a precise boundary worth naming, because it is the exact line a future operator under revenue pressure would be tempted to blur, and naming it is what makes the blur visible.

The extractive bottom of the advertising slope is not “advertising” — it is *attention extraction*: algorithmic amplification, surveillance targeting, engagement auctions. Haven’s primitives do not contain that machinery. Feeds are typed and chronological; there is no amplification engine to sell and no surveillance signal to target on. A registry or community can sell a local business *a place on the bulletin board* — a credentialed role with write access to an offers feed, dues flowing processor-direct to the registry’s legal entity with no platform cut. It structurally **cannot** sell *amplification of attention* — the mechanism does not exist to be sold.

The line between bulletin-board advertising (a local business posting to a feed locals chose to read) and attention-auction advertising (an engagement engine selling targeted reach) is therefore a capability that was never built, not a policy that has to be held. Bulletin-board advertising is supportable and funds genuinely civic surfaces; attention-auction advertising would require adding machinery — amplification, targeting, engagement optimization — whose construction would be visible to everyone. That is the good kind of guardrail: structural, not willpower.

This floor is what the “no advertising” exclusion above actually means. The exclusion is not a vow that no money ever touches a feed; it is the absence of the attention-extraction machinery that makes advertising extractive.

-----

## Tickets, good standing, and credential-gated access

### Sybil-resistant ticketing

Tickets are platform-native credentials. The platform owns inventory, enforces sybil-resistant per-person limits through anonymous credentials with linkable presentation, and is authoritative for who has a ticket. Payment uses standard payment adapters.

Most ticketing platforms allow any payer with money to buy any number of tickets, enabling scalping and bot purchases. Because the platform has cryptographic identities with configurable sybil tolerance, communities can require ticket buyers meet specific identity confidence thresholds and enforce per-identity limits that actually mean one person per ticket rather than one account.

### Good standing through credential issuance

Communities track member good standing through credential issuance triggered by external events. For payment-based good standing (dues, contributions), the community’s instance receives signals from external payment systems via webhook, polling, or manual confirmation. Each event triggers credential issuance: “Alice is current on giving” becomes an “in good standing” credential.

Communities use good standing credentials to gate other actions — voting eligibility, leadership positions, event access — per their own governance. The same pattern serves attendance credentials, donor recognition tiers, and other credentials triggered by external events.

-----

## Creator and audience roles within communities

News organizations, artist collectives, religious publications, educational content, and political accountability feeds are supported through role differentiation within communities rather than separate platform primitives. Communities define internal roles with asymmetric permissions: creators have editorial authority and governance weight; audience members have read access to designated feeds but no editorial voice.

This pattern reuses existing primitives (roles, role-based feed access, role-based voting weight, role-based credentials). Communities choose admission criteria for each role, supported by templates.

-----

## Plugin system

Communities extend the platform with tools specific to their needs through plugins. The platform provides primitives (identity, governance, credentials, communications); plugins compose those primitives into specific shapes communities require.

Plugins can introduce new content types (with their schemas), new governance procedures (Robert’s Rules implementations, consensus circles, conclaves), new credentials, new integration adapters, new moderation tools, new feed types. Plugin authors are identified through contribution attestations; plugins are certified through Foundation processes.

The plugin system serves community diversity. Professional associations needing verified credentials and ethical oversight, religious communities needing ritual integration, artist collectives needing commission tracking, schools needing parent involvement without student surveillance — each gets tools matched to their actual work.

-----

## Substrate and implementation

The platform composes existing cryptographic primitives rather than building on an existing federated substrate. Matrix, AT Protocol, and ActivityPub were considered and rejected as foundations:

- **Matrix:** identity not portable (long-promised, undelivered), room metadata visible to homeservers, power-level model conflicts with pluggable governance, missing concepts (credentials, schism, alliance, locality, vouching).
- **AT Protocol:** built around public content with corporate ownership; not designed for E2E.
- **ActivityPub:** loose typing, design assumptions far from Haven’s requirements.

Composition uses MLS (RFC 9420), Megolm/vodozemac for sender-key alliance messaging, BBS for anonymous credentials, libsodium for primitives, CT-style verifiable logs for transparency infrastructure. Build a parallel protocol designed for specific requirements rather than fight an existing substrate’s assumptions.

### Implementation stack

- **Backend:** Elixir / Phoenix with Phoenix Channels for real-time messaging and OTP supervision trees for fault tolerance
- **Web client:** React with TypeScript, using live_react for server-side rendering of components from Phoenix
- **Mobile client:** React Native with TypeScript, sharing business logic with the web client
- **Cryptography:** Rust (OpenMLS for MLS, vodozemac for sender-key protocol) compiled to native libraries for iOS/Android/server and WASM for web
- **Database:** PostgreSQL storing encrypted blobs and platform metadata
- **Object storage:** S3-compatible for encrypted media attachments
- **Design system:** Tailwind CSS shared across Phoenix templates, React web, and React Native (via NativeWind)

### Cryptographic protocol design status

The two-layer alliance composition (MLS governance with sender-key messaging) uses established protocols but the composition needs external cryptographic review before implementation begins. The message franking design has been mapped well enough to bring focused questions to franking authors — and a June 2026 audit pass against current literature found the franking line (MlsGov, transcript franking) and several other areas (DMLS for fork recovery, the 2025 BBS pseudonym/device-binding work, proactive secret sharing for archive rotation) had moved closer to Haven’s needs than earlier notes recorded, resolving roughly half the open cryptographic questions as already-answered or already-committed and sharpening the rest. The genuinely open questions now cluster tightly around the franking compositions and the BBS four-way credential conjunction. The supporting study notes’ cryptographic references predate this and need a freshness pass. Verifiable log infrastructure adopts C2SP tlog-witness for witness coordination and the Sunlight/TesseraCT temporal-sharding pattern; specific Rust implementation path is pending investigation. Content schema format is JSON Schema with Haven conventions on top.

Pilot deployment timing depends on completing design work and review process, not on calendar commitment.

-----

## What’s currently in active design

Items where direction is settled but specifics need work. Captured in detail in the study notes document.

### Cryptographic implementation decisions pending review

*Updated after the June 2026 audit pass. Items are marked open (needs external review), resolved (closed by reading or analysis), or committed (settled design). The open-questions ledger carries the full resolutions.*

Genuinely open — the focused review asks:

- **Franking composition with MLS PrivateMessages** (open): committing-AEAD choice and placement; confirmed to frank *underneath* MLS’s own AEAD; builds on MlsGov (S&P 2024), so the ask is how Haven’s federation/schism/blind-operator divergences compose, including whether MlsGov’s threat model is semi-honest where Haven’s is blind.
- **Franking over the sender-key broadcast layer** (open, most novel): no published construction; per-message-commitment requirement pulls against sender-key amortization, compounded by the MLS/sender-key integration.
- **Key-reveal escalation vs. forward secrecy** (open, sharpened): does Hecate-style preprocessed-token / key-evolving franking compose with Haven’s exporter-derived moderation keys to escalate without a per-message-reveal FS hit?
- **Transcript franking value over authenticated transport** (open): follows Namavari & Ristenpart 2025; does Haven’s single-sequencer ordering simplify the construction, given transcript franking’s distinct contribution is contiguity/non-omission, not order?
- **BBS four-way conjunction** (open, flagship): single blind-issued credential over committed `pid` supporting selective disclosure + per-verifier pseudonyms + scoped nullifier soundness + device authorization, simultaneously, issuer never learning `pid`. Newer 2025 literature assembles much of this in pieces (Fraunhofer 2025/824 near-match minus the pseudonym/nullifier axis; Mayrhofer pseudonyms-with-rate-limiting note; device-binding 2025/1995; SoK 2026/330 claiming Schnorr-composability) — the ask is whether that composability covers Haven’s exact conjunction and where the one uncovered seam is. Subsumes the prover-committed-`pid` assumption (issuer-known-`pid` mode explicitly not used) and the stable-pseudonym-bound-to-rotating-leaf-key residual.
- **Archive access architecture** (open, reshaped): committee rotation without re-encryption is solved (proactive secret sharing — CHURP/COBRA; no-erasure variant 2026/1072); the open question is threshold-decrypt vs. proxy-re-encrypt for the FS-broken archive, and whether DPSS resharing counts as a loggable access under the mandatory-audit requirement.

Resolved by the audit (no external review needed; see ledger):

- Credential-linking integration surface — the per-verifier-linkability draft’s pseudonym-equality pattern links credentials across issuers within a verifier scope through the specified API; out-of-scope linking needs hand-composition.
- Blind-issuance issuer non-derivability — holds under prover-committed `pid` (two-part `nym_secret`); a design assertion, folded into the BBS conjunction review.
- McMillion partition derivation — RFC 9420 delegates exporter Label/Context to the deploying protocol and the key schedule already binds full group state; Haven drops epoch-from-Context and uses a distinct hardcoded label.
- Multi-layer franking verifier delegation — basic franking property (commitment outside encryption; no keys move); jury-ephemerality is the existing expiry-as-revocation discipline (`texp` / time-based signatures; cf. multi-moderator construction 2026/010).

Resolved as already-committed architecture:

- Per-epoch write-capability keypair (role-write enforcement) — specified in the role-write model; derivation/rotation detail tracked below.

Threshold signature scheme: **resolved — humanness moves to single-signer** (rate-transparency + attendance-anchoring + acceptance policy carry the no-unilateral-minting property; FROST considered, not adopted). The threshold-vs-freshness hot-path tension dissolves with it.

- Fork detection and recovery (open, warm): detection via epoch-authenticator comparison is canonical (RFC 9750); the FS-vs-retention tension is addressed by AMT key-puncturing / DMLS (draft-kohbrok-mls-dmls, prototyped on OpenMLS). Haven’s contribution is incident-scoped use under a single sequencer plus witness-log equivocation proofs; the DMLS authors actively solicit applications.
- State reconciliation protocol for operator-hostile recovery (open)
- Equivocation defense specifics (log type taxonomy, certification criteria for log operators)
- Alliance sender-key derivation timing (snapshot-at-rotation direction set; rotation triggers/cadence interplay with community epoch unspecified)
- Standing check: audit every security-critical consumer of humanness for a second detective/economic gate (single-signer issuance is sound only where no consumer trusts raw humanness ungated)

### Architectural directions in development

- Replica servers for public content scaling (composition with registries, governance, economic model)
- Foundation Alliance lifecycle (constitutional convention, grandfathering, schism threshold tuning) — settle by end of phase 2, with input from people other than the founder
- Canonical interface library for content rendering (Renderable, Discussable, MediaContainer, Geographic, Timed, Reactable — initial set, versioning, governance process)
- Canonical schema set for core content types (events, offers/requests, announcements, discussions, chat, media types)
- Cross-mailbox causality semantics (six commitments captured; implementation details emerging)

### Library and infrastructure investigations

- vodozemac API suitability for use outside Matrix ecosystem (extraction effort)
- Rust implementation path for tlog infrastructure (port Sunlight/TesseraCT vs build from scratch vs use existing)
- Committing AEAD libraries in Rust
- Performance benchmarking for sender-key operations at Foundation Alliance scale

### Working group tracking

- MIMI working group tracking (subscribe to mailing list; Haven is MIMI-shaped not MIMI-conformant — diverges on blind-hub model, aligns on topology and leave/AppSync mechanics)
- IETF Authenticated Transfer working group (AT Protocol federation patterns, chartered early 2026)
- C2SP tlog-witness ecosystem (specification evolution, witness network participation)

### Documentation and presentation

- Plain-English reduction-style proof sketches as preparation for cryptographer review
- Public-facing investigations document (this is closer to ready than study notes suggest)
- Study-notes cryptographic-reference freshness pass (the June 2026 audit found the field repeatedly ahead of the notes; references predate ~2024 and need updating before briefs are written from them)
- Architecture/study-notes reconciliation pass (the audit found at least one review-list item — the write-cap keypair — was already committed design; check for other already-settled items mislabeled open)

-----

## Phase 6: late protocol additions

A standing bucket for protocol features that are deliberately deferred to the end of the design horizon. Each member of this bucket shares two properties: it is separable from the baseline (the platform works without it), and it is either waiting on standardization to mature or addable without disturbing the long-lived credential root.

This is not a backlog of unfinished baseline work. These are intentional late additions whose deferral is itself a design decision. The bucket is expected to grow as later design work surfaces more features in this category.

**Issuer-hiding.** A credential-presentation property that lets the holder prove “issued by some member of this alliance” without revealing which community within the alliance. Target construction: Bobolz-style verifier-policy issuer-hiding, BBS-native via randomizable-key constructions (the publicly-verifiable line being developed in 2026). Interim mechanism: the alliance issues an alliance-membership credential derived from the community credential, optionally bound to the same `pid` through credential-linking — ships early with zero new crypto, covers the common case (prove alliance membership to ordinary verifiers), fails only the hostile-jurisdiction case where the alliance itself must not learn the community-to-user link. Trigger to pull forward: hostile-jurisdiction communities becoming a priority population. Separable from the per-verifier primitive; rolls out at short-lived-credential reissue with no migration.

**Vote-content-secrecy.** Anonymous voting (the baseline) means a vote isn’t tied to identity; the platform’s nullifier mechanism handles that. Ballot-content-secrecy is a separate orthogonal property — also hiding *which* choice was selected, not just who selected it. Required only for coercion-resistance use cases (vote-buying, intimidation defense), which Haven’s attributable-by-default governance rarely needs. The cryptography is moderate and well-trodden (homomorphic tally with verifiable decryption, Helios-style); full receipt-freeness is harder and may not be needed for any specific deployment. Addable as a vote configuration option without disturbing identity, pseudonym, or credential machinery — changes only the ballot payload. New-votes-only at deployment.

**Post-quantum hardening.** The credential layer (BBS, pairing-based on BLS12-381) is not post-quantum, and no standardized post-quantum anonymous credential scheme currently exists with the required properties (selective disclosure plus pseudonyms plus blind issuance in a PQ construction is research-stage). The MLS layer has its own PQ migration path (PQ/hybrid ciphersuites plus reinit); the credential layer does not.

Unlike issuer-hiding and vote-content-secrecy, post-quantum hardening of the credential root is *not* a free roll-forward at short-lived-credential reissue. Replacing the root signature scheme is a genuine migration with no current mechanism. The implication for the present-day architecture: isolate the credential layer behind a clean replaceable interface from the start. Replaceability is a design requirement from day one. When PQ anonymous credentials mature, BBS gets replaced at the credential layer without disturbing the rest of the architecture.

This is an active watch item. External-credential nullifier support (EUDI wallet, eIDAS, mDL) for cleaner humanness dedup is also worth tracking as it would reduce one of the residual privacy tensions in the credential layer.

-----

## Open questions

Beyond active design work, areas needing detailed work before implementation:

**Governance specifics.** Specification format for pluggable governance modules. Standard module library. Conflict resolution when multiple proposals are pending. Compromised admin scenarios with E2E constraints.

**Federation protocol details.** Specific wire protocol. Authentication of federation traffic at transport layer. Migration atomicity mechanism. Backpressure signaling semantics, priority indication on messages, best-effort vs must-deliver message classification, federation policy expression. (Federation graph scaling is largely addressed by the hub instance pattern; remaining concerns are operational/implementation-level.)

**Safety mechanisms.** Coordinated harassment using platform structure. Alliance-public feed moderation at scale. State-actor compulsion scenarios beyond simple operator subpoenas. Legal architecture across multiple jurisdictions. Compliance strategy for jurisdictions hostile to E2E.

**Sybil resistance edge cases.** The platform’s main sybil resistance comes from inherited community practice — in-person introduction, locality-grounded vouching, institutional credentialing all handle the common cases by construction. The remaining edge cases are real but bounded: long-game cooperator attacks (genuine humans cooperating to defeat the system over time), reputation laundering through community-hopping, early-adopter cartel dynamics in vouching networks. At the Foundation level there is an additional structural concern, since humanness is self-issued and governance weight is denominated in it: addressed by the three structural commitments above (instance alliances excluded from the Foundation Alliance, no privileged admission path, competability) plus the working anti-inflation direction (attendance-anchored counting, issuance-rate transparency, sampling, tenure vesting). The honest residual is that a patient, funded adversary operating real humans through real events over years can still buy governance weight — the same residual every real-world institution carries — bounded by tenure vesting, the supermajority thresholds, and public rate logs. The humanness transitive-trust floor (see Issuers) is the related limit on the credential side.

**Operational unknowns.** What the foundation is legally in specific jurisdictions. Long-term funding for protocol development. Operator disappearance recovery beyond hostile-operator design.

**Implementation specifics.** API design. Testing strategy for distributed cryptographic systems. MLS integration choices (NIFs vs Ports for Elixir-Rust bridge). React Native bridge approach for Rust crypto. Database schema for event-sourced state with derived caches.

-----

## Distinctive design principles (summary)

These are the choices that distinguish Haven from existing alternatives. They are intentionally interdependent because they follow from one observation: civic life is already structured, and a platform modeling it inherits that structure rather than imposing one.

1. **Community-pseudonymous infrastructure.** Operators cannot inspect what they host.
1. **In-person gating for membership.** Soft locality enforcement through physical pairing.
1. **No central feed.** Users always choose a community context.
1. **Two-layer alliance design.** MLS governance for representatives, sender-key messaging for broader content; matches social reality of confidentiality.
1. **Math-enabled democracy.** Cryptographic mechanisms enforce procedural rules; human judgment applies through them.
1. **Multi-level rule-based moderation.** Rules at community, alliance, instance, and Foundation levels; rules apply prospectively from when they were in effect.
1. **Humanness universal, decency local.** Humanness credentials (cross-community sybil resistance) issued by the instance alliance from external attestation; decency credentials (membership, role, good standing) issued by communities. Universal identity confidence; per-community behavioral reputation.
1. **Sliding-scale pseudonymity.** Communities choose verification thresholds; vouching enables low-verification participation.
1. **Pluggable governance that mirrors real-world organization.** Platform provides primitives; communities configure governance.
1. **Configurable temporal mechanisms.** Grace periods, quorum, voting windows match institutional governance patterns.
1. **Schism as first-class operation.** Minority factions fork cleanly with full inheritance.
1. **Pluralistic locality registries.** Decentralized discovery through curated views.
1. **Federation distinct from alliance.** Infrastructure vs. social relationships.
1. **Instances as operator-plus-alliance.** Legal/technical authority separated from governance authority.
1. **Verifiable abuse reporting through message franking.** Reportability without surveillance; verification separated from enforcement.
1. **Public verifiable logs through C2SP tlog-witness.** Equivocation defense, replica integrity, audit trails through shared infrastructure interoperable with the broader transparency log ecosystem.
1. **Creator and audience roles within communities.** One-to-many publication supported through role asymmetry.
1. **Typed content schemas with interface-based composition.** JSON Schema with Haven conventions; schemas declare which interfaces they satisfy; clients render against interfaces for graceful degradation.
1. **Sybil-resistant ticketing.** Platform-native credentials with configurable per-person limits.
1. **Revenue from credential production, not attention.** Aligned incentives.
1. **Payment routing without platform-controlled intermediate account.** Money flows directly through processors.
1. **0.01% threshold rule for Foundation Alliance.** Scales with platform size, caps at 10000 entities, drives aggregation.
1. **Plugin system for community-specific tools.** Communities extend the platform with tools matched to their actual work.
1. **The architecture raises the cost of future operator pivots.** Structural protections combine to make extractive pivots substantially harder than in typical software projects.

Cashed out from a community’s side, the principles amount to four claims no incumbent can truthfully make: the operator cannot read your members’ messages; you can move the whole community to another host with members and identities intact; no one can delete or sell the community out from under you; and no one is mining your members’ attention. “Ownership” on other platforms is permission — branding, data export, admin rights, all revocable by the operator who grants them. These four hold even against the operator.

The architecture is meant to be evaluated as a whole — individual choices may seem arbitrary in isolation but support each other in combination because they all rest on the same observation about how trust and institutions actually work.

-----

*This document reflects current settled design direction. Substantial design work has been completed (governance framework, organizational structure, two-layer alliance design, moderation, operator transitions, replica architecture, message franking direction); substantial work remains (cryptographic protocol specification with external review, performance validation, library investigations, pilot deployment, recruitment, funding). The documents will continue to be updated as work progresses.*