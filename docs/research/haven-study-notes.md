# Study output: design directions needing further investigation
 
## Purpose
 
This document captures design directions surfaced during cryptographic study that need further investigation before they can become commitments in the architecture or feature documentation. Each item represents a primitive, library, or design pattern that appears relevant to Haven but has not yet been validated against the platform’s specific requirements.
 
These are not settled decisions. They are working hypotheses about directions worth pursuing, with explicit notes about what remains uncertain and what would be needed to validate or reject each direction.
 
The document is organized roughly by topic. As items are investigated and either committed to or rejected, they move out of this document — committed items into the architecture and feature documents, rejected items into a record of considered-and-rejected approaches.
 
-----
 
## Message franking for verifiable abuse reporting
 
### Status
 
Design direction substantially refined through extended study. Specific composition mapped well enough to bring focused questions to franking authors (Cornell Tech: Tyagi, Namavari, Ristenpart, Grubbs). Still needs cryptographer review before commitment, but the design space is now clear.
 
### Summary
 
Message franking resolves the apparent conflict between two requirements for E2EE moderation:
 
- Reports must be verifiable (moderator can confirm content was actually sent by accused, preventing fabricated reports)
- The system must not become a surveillance mechanism (past messages cannot be repurposed as permanent attributable evidence by future hostile actors)
The construction has the sender produce a signed commitment to message content during encryption; the recipient receives an opening that proves the commitment binds to specific plaintext; reports include the opening, allowing verifier confirmation; without the opening, the commitment proves nothing, preserving deniability outside the reporting flow.
 
Must use committing AEAD (not standard AES-GCM) to prevent invisible-salamanders-class attacks.
 
### Where franking applies in Haven
 
Franking applies to **every encrypted context**, not just casual chat. What varies is the *default disclosure policy*, not whether franking exists.
 
- Encrypted official channels (alliance reps, governance, institutional records): full franking, non-repudiable by default
- Encrypted community chat, working groups, DMs: full franking, deniable default with recipient-triggered reportability
- Alliance broadcasts: full franking, attributed at community level
- Truly public posts: no franking needed (already public)
DMs use the same franking as community chat, not lighter franking — harassment most often occurs in DMs.
 
### Variant composition by context
 
- **Community chat, working groups, DMs**: asymmetric + sealed-sender + transcript franking. Recipient pseudonymity preserved; reports survive forwarding to higher governance layers; pattern reporting works.
- **Alliance broadcasts**: asymmetric + transcript. Sender attribution at community level is intentional (communities are unit of alliance accountability).
- **Official encrypted channels**: asymmetric only. Senders attributed by default; transcript binding less essential because channels are formal.
### Verifier authority separated from enforcement
 
Verification is cryptographic and deterministic — performed by stable infrastructure at each governance layer (community, alliance, instance, federation/foundation). Verification keys are held by infrastructure, not by decision-makers.
 
Verification infrastructure produces verified-evidence packages. Enforcement is governance — jury systems, moderators, councils, alliance bodies receive the packages and decide.
 
This separation matters because:
 
- Juries don’t need to hold long-term keys; they’re selected for cases, receive verified evidence, deliberate, decide
- Verification can be re-performed by higher governance for appeals or oversight
- Audit trails are accountable; every report and outcome are recorded
### Broken franking and key-reveal escalation
 
If a malicious sender attaches a bogus commitment, the recipient’s verification fails. The construction handles this:
 
- Broken franking is itself reportable (the receipt is an anomaly worth reporting)
- Honest infrastructure rejects unsigned commitments, so every retained message has attributable commitment material even if bogus
- Recipients can escalate by revealing the decryption key for the specific message; verifier decrypts and verifies the sender’s signature inside the encrypted payload, bypassing the broken outer franking
Key-reveal cost: deniability is broken for that message. Trade-off: malicious senders cannot escape accountability through bogus franking.
 
Granularity matters — per-message key reveal is preferred over epoch-key reveal; exporter-derived moderation keys may be cleaner. This needs cryptographer input.
 
### The all-malicious-group attack
 
Key-reveal works when at least one recipient is honest. The case where every group member is malicious is not solved cryptographically. Addressed at governance, infrastructure, and external-cooperation layers:
 
- Group-level moderation pathways (operators, alliances, federation can revoke participation based on external signals)
- Identity revocation at AS layer (malicious identities can have credentials revoked)
- Pattern detection at infrastructure layer (traffic patterns, malformed-franking rates, suspicious credential clusters)
- External cooperation (law enforcement tips, NCMEC referrals, victim reports)
**Honest framing**: Haven makes abuse expensive, detectable, and actionable. It does not claim to prevent all abuse. This matches every other E2EE platform’s actual posture.
 
### What’s settled
 
- Direction is right; literature has matured to multiple viable variants
- Franking applies universally to encrypted contexts; only disclosure default varies
- Asymmetric + sealed-sender + transcript composition for most cases
- Verifier authority separated from enforcement; infrastructure holds keys, governance decides
- Broken franking handled through reportability and key-reveal escalation
- All-malicious-group case is handled at governance/infrastructure layers, not cryptographically
### What needs cryptographic review
 
Focused questions for franking authors:
 
- Composition with MLS PrivateMessages; commitment placement in AEAD payload; specific committing AEAD choice
- Composition with sender-key alliance broadcast; cleaner approaches than per-broadcast commitments?
- Multi-layer verifier keys; clean delegation structure for community/alliance/instance/federation
- Jury workflow; key structure for ephemeral re-verification without long-term key holding
- Key-reveal granularity (per-message vs epoch vs exporter-derived); composition with MLS forward secrecy
- Transcript franking value-add over Haven’s existing mailbox ordering
- Mailbox-as-opaque-address composition with sealed-sender franking
- All-malicious-group attack class; additional approaches developed in the field?
### What needs investigation beyond cryptographic review
 
- Library availability — no production library exists; implementation work is real
- Committing AEAD primitive availability in Rust
- Storage and retention specification (commitments retained alongside ciphertext; openings live with recipients only)
- UI specification (context indicators, retention indicators, reportability affordances, sender attribution visibility)
### Action items
 
- Read foundational papers (Grubbs/Lu/Ristenpart 2017, Dodis et al. 2018, Tyagi et al. 2019, Issa et al. 2022, Namavari/Ristenpart 2024-2025)
- Investigate committing AEAD library availability in Rust
- Bring focused questions to Cornell Tech franking group when prepared
- Specify storage/retention semantics in coordination with replica design
- Specify UI affordances in coordination with product/UX work
-----
 
## Certificate Transparency-style verifiable logs
 
### Status
 
Substantially advanced through federation deep dive. The architectural commitment is clear: adopt C2SP tlog-witness protocol, participate in existing witness network, adopt temporal sharding pattern. Remaining work is Rust implementation path and operational specifics.
 
### Summary
 
CT-style append-only Merkle trees with inclusion proofs, consistency proofs, and witness coordination solve several problems across the platform:
 
- **Equivocation defense**: each instance publishes verifiable log entries of its state; peers verify against logs; an instance cannot present different state to different peers without the divergence being detectable.
- **Replica integrity attestations**: replicas publish signed Merkle roots over their held content; clients verify what’s claimed to be held vs. what’s served. Universal attestation across all replica content (not selective by content type).
- **Audit trails** for moderation actions, governance decisions, and other consequential platform events.
- **Potentially franking infrastructure** for variants that use log-based verification of openings.
These are different uses of the same underlying primitive. Building verifiable log infrastructure once serves multiple needs.
 
### What’s settled
 
- C2SP tlog-witness protocol adopted for witness coordination — interoperable with broader transparency log ecosystem (CT, Sigsum, others)
- Haven participates in existing witness network rather than building parallel infrastructure
- Temporal sharding (six-month log shards) bounds log growth — established pattern from Sunlight and TesseraCT
- Log operation is open; Foundation certification is trust signaling, not gating — anyone can operate verifiable logs; certified logs get default discoverability and trust
- Universal attestation across all replica content (not selective) — selective attestation would leak metadata to operators about which content is high-stakes
- Reference implementations to study: Sunlight (Filippo Valsorda / Let’s Encrypt) and TesseraCT (Google), both implementing Static CT API
### What needs investigation
 
- **Rust implementation path.** Port Sunlight/TesseraCT to Rust, build from scratch using their patterns, or use existing Rust crates? Rust ecosystem for tlog is less mature than Go ecosystem (Trillian, Sunlight, TesseraCT all Go).
- **Log type taxonomy.** Different log types (replica integrity, equivocation defense, audit trails, franking support) may have different operators and different discovery mechanisms. Need to map out the specific types and their relationships.
- **Certification criteria for log operators.** What standards must logs meet to be Foundation-certified? Availability requirements, audit cadence, governance, witness participation specifics.
- **Operational cost model.** Reference points exist (Geomys operates Sunlight log for ~$10K/year at WebPKI scale), but Haven’s expected scale will differ. Specific projections await infrastructure planning.
### Action items
 
- Investigate Rust implementation path for tlog infrastructure (existing crates survey, porting effort assessment)
- Read C2SP tlog-witness specification
- Study Sunlight and TesseraCT architectures
- Subscribe to relevant ecosystem mailing lists (witness network coordination, C2SP)
- Map out log type taxonomy
- Develop certification criteria proposal
-----
 
## Plain-English proof sketches (preparation for expert review)
 
### Status
 
Planned deliverable; blocked on foundational study (need to understand reductions well enough to attempt them) and on the protocol composition settling enough to prove things about.
 
### Summary
 
Before talking to cryptographers, attempt plain-English reduction-style proof sketches for the platform’s composition claims. Stated explicitly as informal attempts by someone studying cryptography (not a trained cryptographer), with honest disclaimers about their limits.
 
The value is threefold:
 
- Forces precise statement of what’s claimed and under what assumptions (most of the value is here)
- Surfaces the gaps where the composition might be unsound or where expert help is needed
- Gives cryptographers something concrete to react to (“here’s what we think holds and why; where does this break?”) rather than asking them to do all the work
This is expected to substantially improve credibility in outreach. A cryptographer can tell quickly whether someone understands what a security argument requires. Honest proof sketches signal that understanding better than either hand-waving or false formality.
 
### Composition claims that would need sketches
 
The individual primitives (MLS, Megolm, BBS, committing AEAD, CT-style logs) have their own proofs from their literatures. What needs proving is the composition:
 
- Two-layer alliance design provides claimed properties (governance-layer confidentiality holds despite broad messaging-layer key distribution; layers don’t leak into each other)
- Franking composition with MLS and sender-keys provides verifiability without compromising deniability outside reporting, and doesn’t reintroduce an invisible-salamander-class vulnerability
- Threshold multi-signature provides claimed authority properties (no sub-threshold subset can act; threshold requires distinct representatives)
- Operator transition preserves security across transitions (hostile operator can’t forge; legitimate transition doesn’t expose history)
- Replica integrity attestations detect the attacks claimed (selective withholding is detectable)
- Member-count verification doesn’t leak identity while preventing inflation
### What’s settled
 
- This is worth doing before cryptographer outreach
- Plain-English with honest disclaimers is the right register (not false formality, not hand-waving)
- The property-statement work is the first concrete step and most of the value
### What needs investigation / caution
 
- **Hold the results loosely.** The plain-English attempt will likely produce more confidence than warranted. The gap between a convincing sentence and an actual reduction is where subtle problems live. The invisible salamander bug would have survived a plain-English argument. The sketch that feels airtight is the one to be most suspicious of — formal verification is where airtightness gets tested.
- Needs enough provable-security understanding to attempt reductions (foundational study prerequisite)
- Needs the composition settled (franking variant chosen, threshold scheme chosen, etc.)
- The formal versions likely need expert hands or expert review; the sketches are preparation, not substitute
### Action items
 
- Complete enough foundational study to understand reductions and security games
- Once composition is settled, state each security property precisely (adversary model, capabilities, what’s being broken)
- Attempt plain-English reduction for each composition claim
- Note gaps honestly as they’re found
- Use the sketches (with disclaimers) to structure cryptographer outreach
- Eventually: formal proofs, likely with expert involvement, as their own document
-----
 
## Replica servers for public content scaling
 
### Status
 
Design direction articulated through conversation; not yet fully specified. The pattern composes existing primitives but raises governance questions that need more thought.
 
### The problem
 
A community on a small instance posts something publicly that gets attention. The instance is sized for the community’s normal traffic. Suddenly thousands of people want to read this one post. The small instance can’t handle the load. The community is effectively offline at the moment of highest interest.
 
Existing federated systems handle this poorly. Without a solution, small instances become unviable for communities that occasionally get public attention, creating pressure toward consolidation on larger instances that can handle traffic spikes.
 
### The design direction
 
Replica servers host full histories of public content for their client communities through allied replica agreements. The replica server is the public-facing URL for communities; the home server is the member-facing URL.
 
- External users (non-members) interact with the replica server when reading public content
- Members interact with their home server for everything (including reading their own community’s public content)
- Becoming a member is the transition from “interacting with replica” to “interacting with home server”
- Curation happens community by community through existing mechanisms (registries, locality, interest); replica server doesn’t algorithmically curate
- Home server handles all member traffic and authoritative writes; replica server handles external reader traffic
### Architectural composition
 
The replica server is not a new primitive — it’s an instance with a specific operational pattern:
 
- The instance’s hosted “communities” are registries (specialized alliances for discovery)
- The registries hold allied replica relationships with their member communities for public content
- The replica server’s instance alliance has registries as its representative entities (alliance-of-alliances composition)
- Member communities are not directly members of the replica server’s instance alliance — they’re members of registries; registries are members of the replica server’s instance alliance
This is composition through the existing primitives (instances, alliances, registries, allied replicas) with a specialization for the public-content-scaling use case.
 
### Moderation considerations
 
The replica server hosts publicly-accessible content, which creates legal moderation obligations (CSAM, takedown requests, content liability). The replica server cannot be a dumb cache.
 
The replica operator gets moderation authority through governance: the replica server’s instance alliance (composed of registry representatives) governs the replica service’s policies. This prevents the replica operator from having unilateral content authority while ensuring legal obligations can be met.
 
When replica server moderation policies conflict with home server moderation policies, the existing multi-level moderation framework applies — different levels can act according to their own rules. Persistent disagreement leads to the schism pattern (registries can leave the replica server and operate elsewhere).
 
### What’s settled
 
- The architectural pattern (replica server as specialized instance with registries as hosted entities)
- The URL split between public-facing (replica) and member-facing (home)
- Curation through existing mechanisms rather than algorithmic promotion
- Moderation authority through governance rather than operator fiat
### What needs investigation
 
- Whether one-share-per-registry is the right voting model in replica server instance alliances (small specialty registries vs large locality registries have same weight)
- How registries decide which replica servers to participate in
- Implications of registry membership having governance weight beyond discoverability (changes what joining a registry means for a community)
- How replica servers announce which communities they host (federation protocol details)
- How clients discover and use replica servers (DNS-like? Configuration? Federation protocol extension?)
- Freshness mechanism (push from origin? pull on schedule? hybrid?)
- Economic model (Foundation public service? Strutco commercial service? Mixed?)
- Interaction with the join transition (replica server doesn’t handle joining but surfaces the path to it)
- How discovery services would compose with replica servers and registries
### Action items
 
- Consider this pattern when designing federation protocol details
- Think about whether the Foundation should commit to operating a base-tier replica service as public infrastructure
- Investigate how Bluesky/AT Protocol handle their PDS/relay distinction (their separation of authoritative hosting from content distribution is analogous)
- Revisit when working on registry design (registries become more important than the current docs suggest)
- Consider for architecture rewrite
-----
 
## Typed content schemas — settled as JSON Schema with Haven conventions
 
### Status
 
Settled through federation deep dive. Decision: JSON Schema (draft 2020-12) with Haven-specific conventions on top, plus interface-based composition for client rendering. The lexicon question is resolved in favor of JSON Schema; lexicons were considered and rejected because they provide API surface description AT Protocol needs but Haven doesn’t.
 
### What’s settled
 
- JSON Schema (draft 2020-12) as the schema substrate
- Haven-specific conventions layered on top: content addressing reference types, schema identifier scheme (reverse-domain notation like `app.haven.events.standard.v1`), schema registry/distribution model, version handling, interface satisfaction declarations
- Schema registry follows the same model as verifiable logs: Foundation operates baseline registry, certification is trust signaling, anyone can operate registries
- Interface-based composition for client rendering: schemas declare which interfaces they satisfy; clients implement against interfaces; old clients handle new schemas gracefully through interface compatibility plus fallback type field
- Foundation governs canonical interface library; communities can define custom interfaces (but clients aren’t expected to implement them)
### What needs work
 
These items remain open for development but the architectural commitment is set:
 
- Canonical interface library: specific initial set of interfaces (Renderable, Discussable, MediaContainer, Geographic, Timed, Reactable plus others), their specific field requirements, versioning approach
- Canonical schema set for core content types (events, offers/requests, announcements, discussions, chat, media types)
- Schema registry operation specifics (Foundation-operated baseline, distributed certification model, operational details)
- Interface governance process (how new canonical interfaces are added, deprecation handling, community-defined interface acceptance criteria)
- Implementation of schema validation in the platform stack
### Documentation status
 
The settled direction has been incorporated into the architecture doc and features doc. The content of this item now serves as a record of the decision and a guide to remaining implementation work.
 
### Action items
 
- Define initial canonical interface library
- Define initial canonical schemas for core content types
- Plan schema registry implementation
- Plan validation integration in platform code
-----
 
## Foundation Alliance lifecycle and legitimacy
 
### Status
 
Design directions surfaced through conversation; need development before commitment. Not blocking near-term work but should be settled before pilot deployment.
 
### The problems
 
Three related issues with how the Foundation Alliance transitions from bootstrap to mature operation:
 
**The founder-binding activation problem.** The current design hand-waves about when founder-binding “comes into effect” — there’s a gradual transition from founder-controlled bootstrap to representative-controlled mature operation, but no specific moment when the transition is recognized as complete. This is unsatisfying both legally and legitimacy-wise. The architecture’s whole pitch is structural protections that actually hold; an unclear moment of activation undermines that pitch.
 
**The early-adopter disenfranchisement problem.** The 0.01% threshold rule for Foundation Alliance composition becomes meaningfully constraining as the platform grows. Small early-adopter communities that joined during bootstrap will eventually fall below the threshold at recomputation. Without protection, they suddenly lose voting rights despite having taken on the risk of joining early. This creates incentives to schism early (form alliances of small communities to maintain threshold relevance) rather than to stick around and build organically.
 
**The early-stage schism threshold problem.** Default schism thresholds tuned for mature platform operation may be wrong for early-life Foundation Alliance dynamics. Small composition means small numbers of dissenting representatives can trigger schism, possibly inappropriately. Thresholds may need to be different during early phases when the alliance is still establishing its character.
 
### The constitutional convention direction
 
Rather than gradual hand-waved transition, an explicit constitutional convention establishes legitimacy. The convention is a designated moment when the Foundation Alliance, having reached sufficient composition and maturity, formally constitutes itself as the authoritative governance body.
 
What this would involve:
 
- A defined set of triggering conditions (composition reaches a threshold, time elapsed since bootstrap, specific maturity indicators)
- A formal proceeding with all current representatives participating
- Ratification of the founding documents and protocols by the representatives themselves
- A specific moment of legal and cryptographic transition where founder-binding becomes fully operational
The constitutional convention serves several purposes:
 
- Creates clear before/after boundary for founder-binding activation
- Generates representative ownership of the foundation’s authority (they ratified it, not just inherited it)
- Provides a public, witnessed moment of legitimacy that’s harder to undermine later
- Establishes precedent for how future foundational changes happen
This is analogous to historical constitutional conventions (American, various others) where ratification by representative bodies created legitimacy that purely written documents couldn’t.
 
### The grandfathering direction
 
Communities that joined the Foundation Alliance before reaching the 0.01% threshold need protection against sudden disenfranchisement as the threshold scales up. Options:
 
**Permanent founding member status.** Communities that joined before some specific point retain Foundation Alliance membership regardless of threshold. The privilege is conferred for taking early risk; it doesn’t expire.
 
**Aggregation incentives.** Small early communities can form alliances with each other to meet threshold collectively. The platform’s existing alliance hierarchy supports this; the question is whether the incentives are strong enough that communities choose aggregation over disenfranchisement or premature schism.
 
**Decay schedules.** Founding members retain full voting rights for some period, then transition gradually to reduced influence as new members meeting current thresholds join. This balances honoring early risk against not freezing the early composition forever.
 
**Constitutional protection.** The grandfathering provisions themselves get written into the constitutional convention’s ratified documents, making them harder to change later than ordinary governance decisions.
 
Probably some combination. The specific design needs work but the direction is toward protecting early adopters while preventing the early composition from permanently controlling later platform.
 
### The early-stage schism threshold direction
 
Default schism thresholds need separate tuning for different phases:
 
**Bootstrap phase (composition < ~10 representatives):** schism is mechanically trivial (small numbers, easy to reach threshold) but might be inappropriate (single dissenter could trigger). Possibly higher thresholds during this phase to prevent premature fracture.
 
**Early-growth phase (composition ~10-50):** thresholds appropriate to current composition; possibly with explicit consideration of how mature the platform’s character is.
 
**Mature phase (composition >50):** default thresholds apply.
 
The tuning question is what specific numbers serve each phase. Probably needs simulation or stress-testing rather than pure reasoning.
 
### Possible computational framework for stress testing
 
Mulling a simulation framework for stress-testing how groups might behave under various threshold and incentive structures. Not blocking other work; would be valuable for grounding the threshold-tuning decisions in something more rigorous than intuition.
 
Possible shape: an agent-based simulation where modeled communities have configurable preferences, alliance memberships, and decision-making patterns. Test various threshold structures against scenarios (hostile actor trying to capture; legitimate disagreement leading to potential schism; founder corruption attempts; growth pressure on threshold rules) and observe outcomes.
 
This is its own substantial project but could produce evidence-based parameters rather than guessed ones. Worth considering if/when other work permits.
 
### What’s settled
 
- These three problems exist and need addressing
- Constitutional convention is the right direction for the activation problem
- Some form of grandfathering is necessary for the disenfranchisement problem
- Schism thresholds need phase-specific tuning
### What needs investigation
 
- Specific triggering conditions for the constitutional convention
- Exact mechanism of the ratification proceeding
- Choice of grandfathering approach (permanent, decay schedule, constitutional protection, combination)
- Specific threshold numbers for each phase
- Whether the computational framework is worth building
### Action items
 
- Settle these by end of phase 2, before Foundation Alliance becomes load-bearing through cross-community interactions and substantial disputes
- Specific trigger/timeline for activation to be determined with input from people other than the founder — the decision about when founder-binding becomes operational shouldn’t itself be a unilateral founder decision
- Consider computational framework if other work permits
- Document constitutional convention design once specific
- Integrate grandfathering into Foundation Alliance composition documentation when ready
-----
 
## Cross-mailbox causality semantics
 
### Status
 
Settled through federation deep dive. Six commitments captured in federation readiness doc section 10.
 
### Summary
 
Federation requires committing to specific semantics for how time and ordering work across mailboxes. The platform makes six commitments: credentials valid at presentation time, content reference by cryptographic commitment, membership at action time, alliance composition at tally time (with schism handling), federation peer authority at message time, schism preserves history including in-progress operations.
 
### What’s settled
 
The six commitments are settled and captured in federation readiness section 10. Limitations honestly noted: clock manipulation only partially constrained, coordinated multi-operator attacks not detectable without external observers, real-time consistency not available.
 
### What needs work
 
- Specific representation of these semantics in the protocol message formats
- Test cases demonstrating expected behavior under various edge cases (mid-vote schism, credential expiration during in-progress operation, etc.)
- Documentation aimed at implementers
### Action items
 
- Develop concrete test cases for each commitment
- Plan implementation-level representation in the protocol
-----
 
## Working group tracking
 
### Status
 
Active ongoing engagement with relevant standards bodies and ecosystems.
 
### Summary
 
Several IETF working groups and open ecosystems are doing work directly relevant to Haven. Tracking them prevents reinventing the wheel and keeps Haven aligned with evolving standards.
 
### Active tracking
 
**MIMI (Messaging Layer Security Identifiers/Internet Messaging Interoperability) working group.** Current adopted drafts: `draft-ietf-mimi-protocol` (-05), `draft-ietf-mimi-arch` (-02), `draft-ietf-mimi-content` (-07). Terminology: room = group; hub = server with delivery responsibility; followers = non-hub servers. Haven’s DS federation is MIMI-shaped but not MIMI-conformant: aligns on topology (hub-and-followers), leave mechanics, AppSync proposal mechanism, and partition key idea; diverges on the blind-hub commitment (Haven hub stays opaque to membership; adopted MIMI mainline moved toward state-tracking hub). Tracking for ecosystem developments and for the leave/AppSync mechanics; not expecting protocol convergence with the WG mainline.
 
**IETF Authenticated Transfer working group.** Chartered early 2026 to formalize AT Protocol federation patterns. Relevant for understanding how the broader ecosystem handles federation. AT Protocol’s labels primitive is worth understanding even though Haven uses different primitives.
 
**C2SP tlog-witness ecosystem.** The C2SP (Community Cryptography Specification Project) maintains the tlog-witness protocol Haven adopts. Witness network coordination is ecosystem-wide; participating means tracking specification evolution.
 
### Action items
 
- Subscribe to MIMI mailing list (immediate priority)
- Subscribe to IETF AT Transfer working group lists when chartered
- Subscribe to C2SP discussion channels
- Establish process for tracking draft changes relevant to Haven design decisions
-----
 
## MLS identity, key derivation, and confidentiality mechanisms
 
### Status
 
Substantially advanced through MLS deep dive Sessions 1 and 2. Identity architecture, key derivation discipline, sub-group mechanisms, and role-write enforcement model are settled at the architectural level. Implementation details require cryptographer review.
 
### Summary
 
The platform’s MLS-based mechanisms compose several validated MLS primitives (RFC 9420) with BBS per-verifier-linkability (CFRG draft `draft-irtf-cfrg-bbs-per-verifier-linkability`) and a discipline of capture-forward-never-re-derive that governs how derived material outlives epochs.
 
### Identity architecture (settled)
 
**Two credential layers, thin coupling.** MLS leaf credentials authenticate device→persona binding (visible to group members and AS); BBS verifiable credentials carry selectively-disclosable claims (membership, role, attendance, age). Both attach to the same durable identity anchor; the leaf credential does not embed VCs.
 
**Durable identity is a keypair-with-lineage.** Not a UUID. Reputation and recovery attach to a lineage rooted at a genesis key; current authentication key is the latest link. Rotation without losing reputation comes from signed succession, not from a name-to-key mapping authority.
 
**Per-group identity via BBS per-verifier-linkability.** Every group gets its own cryptographically unlinkable pseudonym derived from `(credential secret, scope)`. Constant within a group, unlinkable across groups. Single underlying secret — separation is hard against external linkage but not compromise-resilient. A user needing compromise-resilient separation makes a separate account.
 
**Personas are application-layer.** Not a protocol concept. The protocol exposes multiple independent identities per human (free, because the platform never enforces one-identity-per-human at the identity layer) plus per-group pseudonyms under any identity. The client maps which identity backs which group’s pseudonym.
 
**Humanness universal, decency local.** Humanness credential issued by the instance alliance (universal cross-community sybil resistance; built from external attestations). Decency credentials issued by communities (membership, role, good standing, vouching — community is the attesting authority by construction). The humanness credential roots BBS pseudonyms and anchors the people-vote nullifier.
 
**The AS is functions, not a service.** No central authentication service exists. Reference identifiers come from the in-person QR introduction; issuance is split (self-generated identity key, instance-alliance-issued humanness credential, community-issued decency credentials); verification is the custom credential validation routine each member runs at join/commit; equivocation detection lives in the verifiable log layer. MLS’s “trusted AS” assumption is relocated to the in-person introduction, not weakened.
 
**Blind issuance for the humanness credential.** The instance alliance attests over a blinded secret — never learns the secret pseudonyms derive from. A compromised issuer can mint sybils (bounded, detectable, governance-handled) but cannot impersonate existing members’ pseudonyms. Same inversion pattern as the AS finding from Session 1.
 
### Key derivation discipline (settled)
 
**One derivation tree, versioned-category labels.** Exporter derivations use registered versioned category labels (`haven.partition.v1`, `haven.senderkey.v1`, `haven.franking.v1`, `haven.subgroup-psk.v1`, `haven.write-cap.v1`) with stable internal identifiers as context. Version tags provide the migration hook for protocol versioning.
 
**The exporter secret is never retained.** All consumers derive from the same per-epoch exporter secret; domain separation prevents cross-derivation but gives zero isolation under root compromise. Exporter secret is epoch-ephemeral.
 
**Capture-forward-never-re-derive.** The exporter yields per-epoch material that dies with the epoch. Anything that must outlive the epoch is captured forward at production time, not re-derived. Archive captures ciphertext forward; franking captures opening at send time; partition references retain the routing label, never the secret. This discipline unifies retention, franking, and partition reference mechanisms.
 
### Sub-groups and role-write enforcement (settled)
 
**Confidential roles use branched sub-groups bound by resumption PSK.** Branching creates a new MLS group with a subset of parent participants; injecting the parent’s resumption PSK at creation cryptographically proves sub-group members were members of the parent at the branch epoch. Reinitialization is for whole-group migration (ciphersuite/PQ transition), not sub-groups.
 
**Parent Remove must fan out to sub-groups.** Application-layer invariant. The resumption PSK proves members were in the parent at the branch epoch, not that they still are. Forgetting this leaves ex-members with role access.
 
**Schism uses the same primitive.** Branch at the split epoch; the resumption-PSK binding proves the schismatic faction were genuine members; new-identity and history-inheritance come from existing federation mechanism.
 
**Role-write enforcement model.** Server cannot enforce role-based write access by inspecting content. Genuine write-role enforcement comes from a writer sub-group whose write-capability (derived from the sub-group’s exporter, `haven.write-cap.v1`, per-epoch rotation) the server can verify against. Two signatures: outer write-cap signature (server-visible, anonymous — server learns an authorized writer acted, not which one) and inner persona signature (reader-visible, inside ciphertext, attributable). Server pays per-write verification cost in exchange for the actual landing-gate.
 
**Three feed shapes** follow from the combination of read access (community vs sub-group) and write access (community vs writer sub-group): open-read/open-write, open-read/role-write, confidential-read/role-write.
 
### Multi-device (settled)
 
Per-device leaves are the chosen approach. Device-count-to-co-members leak is accepted (in-person community model already gives co-members substantial knowledge of each other). Per-device recovery (compromise → Remove leaf → PCS heals) is the load-bearing benefit. Device authorization is scoped to the per-group pseudonym (not the global identity key) to avoid re-linking pseudonyms across groups.
 
### History recovery and retention (settled)
 
Recovery model: layered ladder (device handoff → user-side backup → peer history → community archive) plus commitment-verification (recovered plaintext is untrusted until checked against an attested commitment) plus configurable retention with self-service backstop.
 
**Member-facing history is two independent per-feed artifacts.** Onboarding context (short rolling window, low access bar, open-read feeds only, time-bounded self-healing FS waiver — does not need the hard archive primitive) and institutional archive (long retention, high access bar, every access audited, scope includes confidential sub-groups, hard FS sacrifice with threshold gating). The two cannot be collapsed into one artifact with two policies because they have opposite default trust postures — collapsing fails in both directions.
 
The four combinations (neither, onboarding only, institutional only, both) are all meaningful per feed. The institutional-only case is what proves the separation: joining a treasury committee grants archive eligibility but not an onboarding dump of recent treasury talk.
 
**New-member history reduces to archive access; no protocol-level history sharing.** MLS forward secrecy means epoch keys are deleted on schedule and new members get nothing pre-join by default. The two options are (a) retain old epoch secrets and hand them to the joiner (breaks FS for the whole group; violates capture-forward discipline; rejected) or (b) re-encrypt old content under something the new member can access (the write-forward archive; adopted). MLS protocol-level FS stays fully intact; the FS sacrifice is isolated to the governed archive layer.
 
**History access is scoped per-feed by the same read-role that gates live access.** A returning device of an existing member may see everything that member could; a brand-new community member sees only what a new member is entitled to. Confidential sub-group history is gated separately by sub-group membership.
 
**History status is disclosed and governed.** Whether a feed has each artifact is a visible per-feed property — posters need correct expectations about whether their content will outlive the epoch.
 
Forward secrecy claim has a carve-out: FS protects non-retained content; retained content is protected by access controls and audit, not by forward secrecy. This honest acknowledgment matters — no construction gives both “recoverable after losing only device” and “cryptographically unrecoverable after epoch keys are deleted.”
 
The institutional archive is the most dangerous artifact in the system (the one place FS is deliberately broken). Threshold-shared decryption authority bounds who can open it; audit logging makes opening undeniable. Both required; gating without auditing lets a colluding threshold quietly surveil.
 
### KeyPackage handling (settled)
 
No last-resort KeyPackages anywhere in Haven. Every join is in-person-rooted (QR handshake supplies fresh single-use KeyPackage), parent-state-derived (existing member added to sub-group/feed; no DS stash fetch), or interactive (joiner actively participating supplies fresh KeyPackage). The trichotomy is exhaustive given the in-person membership model — the offline-no-relationship add that last-resort exists for never occurs.
 
Bootstrap KeyPackages are single-use and short-lived (bounded to invite validity). Multi-use invite links are rejected — they would force reusable bootstrap KeyPackages, reintroducing both the reuse-FS problem and a bypass of the in-person-trust gate.
 
Stash exhaustion handling: proactive replenishment plus delayed-add semantics. Civic-use-case rhythms make this acceptable.
 
Implementer note: an off-the-shelf MLS stack offers last-resort as the default patch for stash exhaustion; do not adopt reflexively. The reason last-resort is unnecessary in Haven (the trichotomy) is not visible from the library.
 
Reopening condition: if a future flow ever needs to add a party who is not present, not already a member, and not interactively participating, last-resort would need reconsideration. Recorded as the reopening condition, not a current gap.
 
### Capability negotiation (settled)
 
Single universal BBS credential path; no fallback identity path. `required_capabilities` enforces the floor. Capability mechanism retained for ciphersuite/version agility (the eventual post-quantum transition), not for credential heterogeneity. One-way-door property of `required_capabilities` means universal client support must precede group requirement.
 
### What needs cryptographer review
 
Carried forward from sessions 1, 2, 3, and 4:
 
- BBS device-authorization composition (per-group pseudonym authorizing MLS leaf signature key without re-linking pseudonyms across groups; what’s the construction for a device-authorizing key bound to a pseudonym that is itself a group element)
- Institutional archive-key construction (threshold encryption vs proxy re-encryption; rotating archivist set without re-encrypting whole archive). Narrowed in Session 3: the onboarding-context artifact does not need this primitive; only the institutional archive does.
- Blind-issuance properties for the humanness credential (instance alliance issuer) and AS-finding analog (home community issuer for decency credentials) — confirm participating issuer cannot derive or forge existing member’s pseudonym
- Per-epoch write-capability keypair construction and rotation
- McMillion partition derivation epoch-in-Context rationale (understand whether redundant with exporter epoch-scoping or deliberate explicit-binding; relevant to the capture-forward discipline)
- Fork-recovery state retention: minimum state and window needed to detect and reconcile a fork while preserving as much forward secrecy as possible
### DS federation: blind hub (settled)
 
Haven runs a blind delivery hub — handshakes are MLS PrivateMessages; the hub does not track group membership; cryptographic membership enforcement comes from partition-key derivability (only members can derive) plus the anonymous write-cap (only authorized writers can produce).
 
The blind hub does two things: sequencing (consistent order per partition key) and fanout (pull-based on partition keys). It cannot do the one job that requires a state-tracking hub (reconstructing GroupInfo for offline joiners), but the in-person join model means no joiner is ever offline with no existing group relationship — every join is in-person-rooted, parent-state-derived, or interactive. The blind-hub model and the in-person join model are two views of the same architectural decision.
 
This is a principled divergence from the adopted MIMI protocol mainline (which moved toward state-tracking hub). Haven is MIMI-shaped but not MIMI-conformant — aligns on hub-and-followers topology, leave mechanics (three-step Remove proposals plus AppSync removing user from participant list), AppSync mechanism for participant-list sync, and the partition-key idea (`draft-mcmillion`); diverges on hub state-tracking. Real maintenance and interop cost, taken with eyes open.
 
Cross-instance state synchronization falls out of the model: sequence-per-partition plus pull-by-partition-key fanout, with the in-person join trichotomy removing the need for any separate sync mechanism. No separate state-sync protocol designed.
 
### Operator-hostile state reconciliation: fork recovery (settled)
 
**Threat floor: a hostile operator is a vandal, never a spy.** E2E plus blind hub mean a hostile hub cannot read content, learn membership, or target specific members or feeds. The threat collapses to availability (detected by replica attestations) and ordering-integrity (the hard case).
 
**Forking is inherent to MLS, but detectable.** A hostile sequencer can deliver different commits for the same epoch to different members, producing two cryptographically-valid divergent chains that cannot be merged. This is inherent to CGKA. Detection uses the epoch authenticator: two different authenticators for the same (group, epoch) attested to verifiable logs and witness-cosigned is cryptographic proof of a fork.
 
**Choose-and-re-establish, never merge.** Haven keeps the single-sequencer model and recovers from forks rather than tolerating them. Decentralized MLS and similar fork-tolerant CGKA variants were rejected — they retain key material permanently (a standing FS reduction), and fork consolidation is unsolved in general.
 
Recovery procedure: detect via epoch-authenticator divergence; fix the resumption point at the last attested-consistent epoch; governance threshold of representatives selects the surviving branch; stranded members are re-added to the survivor via fresh commits (using the branch/re-add machinery); content history survives via allied replicas; new hub sequences onward. State retention for fork detection and reconciliation is incident-scoped (retain just enough, just long enough), not standing — bounded FS softening during the incident window, then deletion.
 
**Operator-induced fork is distinct from schism.** Schism is deliberate governance fork (minority intentionally branches, new identity, both communities legitimate). Operator-induced fork is involuntary (same identity, one branch abandoned, members re-merged). Machinery overlaps but schism creates two legitimate groups while fork recovery collapses back to one. Conflating them would legitimize the attacker’s branch.
 
### Humanness credential instance migration (settled)
 
The Session 2 issuer reversal moved root/humanness-credential issuance to the instance alliance. The migration handling falls out of existing federation machinery:
 
- **Operator change / hardware migration**: instance alliance keypair is durable across these (federation design), so humanness credentials are untouched. Already solved by operator-transition mechanism.
- **Cross-instance-alliance move**: new alliance re-attests humanness, re-issues over the same blinded secret so pseudonyms persist. Same shape as community migration, one level up. The new alliance can either accept a signed cross-instance attestation of prior verification or re-verify the external proof from scratch — per-alliance policy.
- **Cross-instance honoring**: standard direct/alliance/transitive trust framework decision. “Universal” means portable, not automatically trusted everywhere.
The worst case (hostile-operator migration where the new alliance refuses to honor the previous alliance’s humanness credentials) is effectively a humanness wipe — users must re-attest. This is a real cost of hostile-operator escape, honestly acknowledged rather than papered over.
 
### Implementation choice direction
 
mls-rs offers user-defined credentials with custom validation routines that bridge to external credential schemes — the validation hook is the AS, in-library. This is the best fit for Haven’s credential validation shape. OpenMLS treats credentials as opaque (passes them along, validates in app code); as of 2025 it accepts unknown credential types at the protocol layer but offers no in-library validation hook. Neither blocks the design; one weight toward mls-rs to be balanced against other factors.
 
### Action items
 
- Bring cryptographer questions to review (BBS device authorization, archive key construction, blind issuance, write-cap construction, McMillion epoch-in-Context, fork-recovery state retention)
- Track parked items: protocol versioning (reinit identified as ciphersuite/PQ migration tool); per-alliance policy for cross-instance humanness re-issuance handshake
- Plan implementation choice between mls-rs and OpenMLS
- Borrow MIMI leave mechanics and AppSync details when implementing
-----
 
## BBS credential track
 
### Status
 
Substantially advanced through BBS deep dive Session 1. The credential layer’s architectural commitments are settled: BBS as the standardized family, two-credential humanness model, credential-linking through shared-`pid` ZK equality, one construction for the three consumers (identity/voting/ticketing), application-layer counting for cardinality caps, deliberate deferral of issuer-hiding and vote-content-secrecy to Phase 6. The Phase 6 late-protocol-additions bucket is the structural concept for these deliberately-deferred separable features.
 
### What’s settled
 
**BBS as the family.** The scheme is BBS in current standardization (CFRG drafts: `bbs-signatures`, `bbs-blind-signatures`, `bbs-per-verifier-linkability`, plus W3C `vc-di-bbs` for VC integration). What earlier docs called “BBS+” is the older variant; modern BBS is proof-tightened. Chosen over PS/CL on ecosystem grounds — the extension drafts Haven requires are concentrated in the BBS family.
 
**One blind-issued credential over committed `pid`.** A single BBS credential blindly committing the holder’s secret `pid` plus disclosable attributes supports selective disclosure, per-verifier pseudonyms unlinkable across scopes, and nullifier soundness within scopes — all from the same `pid`, issuer never learning `pid`.
 
**Two-credential humanness model.** Long-lived humanness root (instance-alliance-issued, tied to external proof at issuance, never presented directly) plus short-lived humanness-freshness credential (reissued from root over same `pid`, presented at each interaction). Revocation = refuse to reissue freshness; the current short-lived credential expires; no accumulator, no heartbeat, no renewal round-trip.
 
**Combined presentation.** Holders present humanness-freshness + decency credential + shared-`pid` ZK equality + selective disclosure in one combined BBS proof. The verifier learns the holder is freshly humanness-attested AND a member-in-good-standing AND any disclosed attributes, all bound to the same person, without learning `pid`.
 
**Migration vs revocation distinction.** Cross-instance revocation matters only at migration, which is already non-anonymous (the user re-presents external proof). The new instance checks the presented external proof against a forward-only ID-hash revocation log. Per-presentation unlinkability untouched. The instance stores only one-way ID-hashes, never plaintext external proof.
 
**Triple-duty consumers.** Identity (per-group pseudonyms, scope = community, anchor = shared `pid`), voting nullifiers (scope = decision id, anchor = humanness `pid` for one-person-one-vote), ticketing nullifiers (scope = event id, anchor configurable along sybil-tolerance gradient). All one per-verifier-pseudonym construction varied by scope and anchor.
 
**Cardinality via app-layer counting.** Voting (cap 1) and ticketing (cap N) become one construction: same nullifier per (`pid`, scope), single counting authority per scope keeps a counter, rejects past the cap. Validity rests on the single-authority-per-scope property — each scope routes through exactly one authority (one tally per decision, one ticketing system per event). This matches the federation single-authority-per-mailbox principle.
 
**Eligibility-linking for secret ballots.** A secret ballot proves “eligible member AND unique human who hasn’t voted” by linking the decency-eligibility proof to the humanness nullifier via shared-`pid` ZK equality — same credential-linking primitive that powers revocation. Credential-linking does double duty: revocation and eligible-secret-voting.
 
**Vote-content-secrecy is Phase 6.** Anonymity (nullifier) is baseline; ballot-content-secrecy (hiding which choice was selected) is a separate orthogonal property only needed for coercion resistance, which Haven’s attributable-by-default governance rarely requires. Addable as a per-vote configuration without disturbing identity/pseudonym/credential machinery.
 
**Issuer-hiding is Phase 6.** Target Bobolz-style verifier-policy issuer-hiding, BBS-native via randomizable-key constructions (publicly-verifiable line being developed in 2026). Interim alliance-credential hack ships early with zero new crypto, covers the common case. Trigger to pull forward: hostile-jurisdiction communities becoming a priority. Separable from per-verifier primitive; rolls out at short-lived-credential reissue.
 
**Post-quantum horizon.** BBS is pairing-based (BLS12-381) and not post-quantum. No standardized PQ anonymous credential scheme currently exists with the required properties. The MLS layer has a PQ migration path (reinit + PQ/hybrid ciphersuites); the credential layer does not. Implication: isolate the credential layer behind a clean replaceable interface from the start. Replaceability is a design requirement.
 
### What needs cryptographer review
 
The BBS-track cryptographer review consolidates around two questions:
 
1. **The composition question.** Does a single BBS credential, blind-issued over a committed secret `pid` plus disclosable attributes, securely support simultaneously (a) selective disclosure, (b) per-verifier pseudonyms provably unlinkable across scopes, and (c) nullifier soundness (one pseudonym per (`pid`, scope)) — all from the same `pid`, issuer never learning `pid`? Plus device-authorization as a bound value inside the BBS proof (rather than a separate pseudonym keypair, since the pseudonym is a group element not a signing key).
1. **Credential-linking integration surface.** The technique (shared-`pid` ZK equality across two BBS credentials, prove a common attribute equal without revealing it) is well-understood — built on textbook Sigma proofs of commitment-equality over the Pedersen/Fiat-Shamir foundations BBS already uses. Different issuer keys don’t break it (same scalar, conjoined verification relations). HIGH confidence on the technique. The open question is the integration surface (MEDIUM confidence): does the adopted CFRG BBS proof API expose cross-credential equality directly, or is it a small hand-composed Sigma conjunction over a shared blinded scalar?
For Phase 6 work (not blocking baseline):
3. Issuer-hiding construction: confirm the chosen BBS-native randomizable-key scheme is publicly verifiable and composes with per-verifier pseudonym and nullifier mechanisms.
 
### What’s been considered and rejected
 
- **Bitstring Status List** for revocation — breaks unlinkability through per-credential indexes and issuer-contact-at-check.
- **Unlinkable accumulators** (ALLOSAUR, SD-BLS, CRSet) — all 2024-25 and unsettled, and unnecessary since expiry-as-revocation eliminates the need.
- **Heartbeat** (a third short-lived token shuttling freshness from instance to community) — a third standing credential lifecycle existing only to carry a fact the presented humanness-freshness already carries. Same family as rejecting last-resort KeyPackages and standing exporter retention.
- **Indexed nullifiers + range proofs** for N-per-person ticketing — eliminated by application-layer counting per scope.
- **PS or CL signatures** instead of BBS — comparable cryptographically but the ecosystem (extension drafts, W3C VC integration) is concentrated in BBS.
### Watch items
 
- **PQ anonymous credentials.** No current migration path; isolate credential layer behind replaceable interface. Track research progress.
- **External-credential nullifier support** (EUDI wallet, eIDAS, mDL). Would enable cleanest humanness dedup by pushing dedup to the external issuer — Haven learns “used once” without the real-world identity. Worth watching as digital identity wallets evolve.
- **BBS-native publicly-verifiable issuer-hiding constructions** (randomizable-key line, ePrint 2026/369, 2026/555, 2026/870). The Phase 6 issuer-hiding work waits for this line to settle.
### Reference designs
 
- Fraunhofer AISEC ePrint 2025/824 — anonymous credential system using BBS with privacy-preserving revocation, non-duplication, and device binding (BBS + status credentials + DAA-style binding). Requirement set similar to Haven’s; study as a reference composition (not necessarily to adopt).
- SmartphoneDemocracy (arXiv 2507.09453, Jul 2025) — BBS + EUDI wallet, two-layer nullifier (registration-uniqueness off-chain + per-election on-chain), ballot secrecy via additively-homomorphic encryption. Validates the humanness-anchored dedup and external-credential-nullifier path.
### Action items
 
- Bring composition and credential-linking questions to cryptographer review
- Continue tracking BBS-native publicly-verifiable issuer-hiding constructions (2026 line of work)
- Track EUDI/eIDAS/mDL for nullifier support that would enable cleaner humanness dedup
- Study Fraunhofer 2025/824 and SmartphoneDemocracy as reference designs
- Plan replaceable-interface design for the credential layer (PQ contingency)
-----
 
## Items to add as study progresses
 
This section will grow as foundational study surfaces additional directions. Each item gets the same treatment: brief summary, what’s settled, what needs investigation, action items.
 
Likely future additions (rough guesses based on the topics flagged in the federation deep dive and MLS/BBS sessions):
 
- Threshold signature schemes (FROST vs. simple multi-signature)
- Specific committing AEAD primitives
- Canonical interface library specifics
- Canonical schema set for core content types
- Backpressure signaling protocol specifics
-----
 
## Process notes
 
This document is a working notebook, not finished documentation. Standards for what goes in:
 
- Direction must be specific enough to investigate further (not just “we should think about X”)
- Status should be honest about what’s settled vs. what’s uncertain
- Action items should be concrete enough that they can be checked off
Items that have been validated and committed to move into the architecture and feature documents. Items that have been investigated and rejected get noted in a separate considered-and-rejected list.
 
When in doubt about whether something belongs here vs. in a primary document, default to here. The primary documents should reflect committed direction; this document captures the work in progress.
