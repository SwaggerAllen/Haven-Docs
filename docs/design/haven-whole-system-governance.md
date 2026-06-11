# Whole-system governance
 
## Purpose
 
This document specifies the governance framework for the Haven platform — how decisions are made about the platform itself (not about communities, which govern themselves through their own configured modules). It covers the math-enabled democracy framework, the people-vote principle, the layered authority structure, and the lieutenant pattern.
 
These mechanisms are referenced across other documents (organizational structure, federation readiness, founder-binding analysis) but are specified here in one place. The detailed implementation of specific cryptographic protocols and governance interfaces is deferred to later design work; this document captures the architectural commitments.
 
The framework is designed against the historical pattern documented in the founder-binding analysis. Comparable projects have failed when their governance arrangements concentrated authority enough that one party could override original commitments. The framework here distributes authority cryptographically, legally, and economically so that no single party can capture the platform after bootstrap.
 
-----
 
## The fundamental approach: math-enabled democracy
 
Platform-level governance operates through cryptographic mechanisms that enforce procedural rules, with human judgment applied through those mechanisms. Math handles attestation, threshold checking, and authority signature; humans handle the substantive decisions the procedures enable.
 
This is a hybrid between two extremes that fail in different ways:
 
**Pure social governance** is what most projects use — bylaws, board votes, contracts, organizational norms. It can be overridden unilaterally by whoever holds operational control. The Couchsurfing pattern, the Bitcoin Foundation pattern, the early Mastodon pattern — these are cases where the formal governance existed but didn’t actually constrain the founder or the operator. The math doesn’t enforce the procedure; the procedure relies on people choosing to follow it.
 
**Pure mathematical governance** is what some blockchain governance attempts — token voting, on-chain proposals, DAO mechanisms. It can’t handle the inexpressible cases: representative goes inactive due to illness; sybil attack through coordinated work; situation the rules didn’t anticipate; question that needs judgment rather than counting. The math enforces something, but the something doesn’t include enough judgment to govern well.
 
**Math-enabled democracy** uses cryptographic enforcement for the procedural layer (verifying who’s authorized, counting valid signatures, recording decisions) while human judgment operates through the procedures (deciding what to vote on, who to confirm as authoritative, when to escalate, when to refuse). The math prevents unilateral override; the humans handle judgment the math can’t capture.
 
The motivating principle is “if it isn’t guaranteed by math then it isn’t guaranteed” — but math cannot capture everything, so there has to be a human in there somewhere. Math handles the bookkeeping; humans handle the judgment that the bookkeeping enables.
 
Apache’s committer model is the closest existing analog: meritocratic at the contribution level (you have to actually contribute to be considered for committer); social judgment at the confirmation level (existing committers vote on new committers); foundation board authority for systemic decisions. Haven’s version adds cryptographic enforcement of the procedural layer that Apache handles through organizational norms.
 
-----
 
## The people-vote principle
 
Governance weight derives from individual humans, not from accounts, tokens, stake, contribution volume, capital, or any other proxy. Each real human contributes a total vote-mass of 1.0 to the platform governance system, distributed equally across their community memberships. A person in N communities contributes 1/N vote-mass through each community’s representative for alliance-level decisions.
 
This commits the platform to several specific positions:
 
**A credentialing threshold distinguishes real humans from accounts.** Accounts are cheap (a person can create many); humans are not (each person is one). Governance weight needs to track humans, not accounts. The credentialing threshold is enforced through the locality-grounded sybil resistance chain — in-person QR joining, vouching with social-distance-weighted accountability, locality grounding through real-world institutional or geographic basis.
 
**Institutional credentialing is the verification mechanism.** Institutions vouch for the humanity and membership of their participants through the credential system. The platform does not maintain its own humanity-verification mechanism; it relies on existing institutions (churches, schools, neighborhood associations, professional organizations, etc.) that already verify their members’ identities for their own purposes. These institutions are the trust anchors for governance participation.
 
**Cryptographic mechanisms enable anonymous voting with double-vote prevention.** Voters present anonymous credentials (BBS with per-verifier-linkability for nullifier-based double-vote prevention) that prove their authorization to vote without revealing their identity. Nullifiers prevent double-voting within a single decision; identity remains pseudonymous across votes. This is closely related to the ticketing primitives Haven needs to design — the cryptography is shared.
 
**The two-tier system is explicit.** Communities with institutional credentialing can participate in platform governance. Communities without institutional grounding can exist on the platform but cannot govern it. The platform’s focus on local civic engagement makes this position coherent — online-only communities without institutional grounding are not the platform’s primary audience, and excluding them from higher-level governance (while still allowing them on the platform) reflects the project’s actual scope.
 
**Fractional weight rewards focused engagement.** A person in three communities contributes 1/3 vote-mass through each. A person in one community contributes their full vote-mass through that one community. This rewards focused civic engagement — the active member of one parish has more representation in that parish’s alliance than the dilettante who’s nominally a member of five organizations. Communities can choose to require exclusive membership for governance purposes if they want maximum representative weight; that’s a community-level decision.
 
**The instance-alliance level breaks the people-vote pattern.** Instance alliances use one-share-per-community with two-thirds threshold rather than people-vote weighting. The reason: federation verifiers need to be able to check signatures against composition, and tracking member counts per community across federation is brittle. The unit being represented at the instance-alliance level is the community-as-tenant (a unit of infrastructure dependency), not the individual citizen. This exception is documented honestly rather than glossed over.
 
-----
 
## Layered authority
 
Authority is distributed across multiple layers, each with its own scope and accountability structure. No single layer can act unilaterally on matters reserved to other layers.
 
### Platform identity layer
 
The Haven Foundation holds platform-level identity through the Foundation Alliance. This is the MLS group of representative entities at the platform level, exercising authority through threshold multi-signature (default two-thirds of composition). Platform-level decisions — protocol changes affecting all instances, certification criteria for operators and plugins, baseline rule set for moderation, federation policy at the platform level — require Foundation Alliance authority.
 
Foundation Alliance representatives are accountable to the communities they represent. The Foundation board is composed of these representatives. Foundation legal authority (board votes) and Foundation Alliance cryptographic authority (multi-signature operations) reflect the same underlying decisions by the same people; there is no parallel structure that could disagree with itself.
 
### Foundation Alliance composition criteria
 
A Foundation Alliance member must have at least 0.01% of the platform’s institutionally credentialed population as members. This threshold serves several purposes simultaneously.
 
**It scales with platform size.** At 1M institutionally credentialed users, the threshold is 100 users — small communities can qualify. At 100M users, the threshold is 10000 users — Foundation Alliance membership requires meaningful constituency. The rule scales the eligibility bar automatically with the platform’s actual reach.
 
**It caps composition at 10000 members.** By construction, no more than 10000 entities can each have 0.01% of the population. This stays comfortably within vanilla MLS scaling limits (no dependency on “trees of MLS trees” or similar speculative cryptographic developments). The Foundation Alliance can operate as a standard MLS group throughout the platform’s growth.
 
**It naturally drives aggregation.** Small communities cannot meet the threshold individually at scale. To gain Foundation Alliance representation, they must either grow substantially or join alliances that meet the threshold collectively. This is consistent with how civic representation works in the real world — small congregations are part of dioceses, small associations are part of larger networks. The platform’s alliance hierarchy supports this aggregation.
 
**It preserves the people-vote principle.** The “members” being counted are institutionally credentialed humans, deduplicated across the entity’s constituent communities. Vote-mass aggregates up through the entity’s internal structure; the Foundation Alliance receives representation proportional to actual human constituents, not account counts or community counts.
 
**It excludes online-only entities.** Communities without institutional grounding cannot accumulate institutionally credentialed members. They can exist on the platform but cannot gain Foundation Alliance membership. This is consistent with the platform’s civic-institutional focus.
 
**Operators are ineligible.** Operators (e.g., Strutco) are infrastructure providers, not human constituencies. Their participation in Foundation Alliance governance would conflate infrastructure with constituency. Operators are accountable to the platform through other mechanisms (certification, federation, the math-enabled democracy framework), not through Foundation Alliance representation.
 
**Instance alliances are ineligible.** Instance alliances aggregate hosted communities as infrastructure-tenants on a one-share-per-community basis. This unit is not commensurable with the deduplicated people-vote the Foundation Alliance counts — admitting an instance alliance would either double-count (its constituent communities are already represented) or mis-weight the people-vote. The same constituent communities reach Foundation Alliance representation through their own membership or through alliances-of-communities organized around shared mission rather than shared hosting.
 
Membership is limited to communities and alliances-of-communities with institutionally-credentialed human constituents.
 
### Foundation Alliance composition mechanics
 
Composition changes annually (or at whatever interval the Foundation Alliance configures). Between recomputation events, composition is stable. This avoids constant churn from entities hovering near the threshold.
 
At each recomputation:
 
- Current members verify their institutionally credentialed member count
- Entities below the threshold lose Foundation Alliance membership
- Applicants now meeting the threshold gain membership
- Composition changes commit atomically through Foundation Alliance MLS proposals
- New composition is announced to federated peers
- Next year’s threshold is computed from new platform population
Verification of membership counts uses cryptographic deduplication where possible (anonymous credentials with linkable presentation can count distinct humans without revealing identities — the same primitive used for voting and ticketing). Where cryptographic verification is not yet available, institutional attestation with audit-on-demand is acceptable; entities making false claims face reputation consequences and can be challenged through Foundation Alliance process.
 
The bootstrap period has effectively no threshold (0.01% of a small early-platform user base is fractional). Foundation Alliance composition in early phases is determined by whoever applies and meets the qualitative criteria (institutional credentialing, demonstrated commitment to platform governance). The threshold becomes meaningfully constraining only at platform scale.
 
### Subsystem layer (lieutenants)
 
Specific subsystems of the platform have designated maintainers — lieutenants — who hold cryptographic authority over their subsystem. Subsystem updates require lieutenant signature. The Foundation Alliance confirms lieutenants through threshold multi-signature, based on contribution attestation eligibility.
 
This is the Apache-hybrid model with cryptographic enforcement. Contribution attestations gate eligibility — someone must have actually contributed to a subsystem to be considered for lieutenant of it. The Foundation Alliance representatives apply judgment about whether to confirm an eligible candidate. The confirmation produces cryptographic authority; signatures from confirmed lieutenants are required for subsystem changes.
 
Subsystem boundaries are themselves a governance decision. Initial subsystems likely include:
 
- Cryptographic protocol (MLS group operations, key management)
- Federation layer (cross-instance routing, replica protocol)
- Credential system (issuance, verification, revocation)
- Plugin API (extension points, security model)
- Governance system (voting, proposals, role management)
- Moderation system (reporting, escalation, federation effects)
These are not final; the Foundation Alliance can adjust subsystem boundaries as the platform develops. New subsystems can be created; existing ones can be split or merged.
 
Lieutenants can be employed by any operator (including Strutco) or independent. Their lieutenant role is foundation-granted, not employer-granted. An operator cannot dictate technical direction by hiring lieutenants because lieutenant authority comes from Foundation Alliance confirmation, not from employment. This is the structural protection against operator capture of technical direction.
 
Lieutenant terms are bounded; renewal requires reconfirmation by Foundation Alliance. Removal requires Foundation Alliance threshold vote. A lieutenant who acts against the platform’s interests can be removed; an effective lieutenant can be renewed indefinitely.
 
### Instance layer
 
Each instance has its own instance alliance — the MLS group of communities hosted by that instance — exercising authority through threshold multi-signature for instance-level decisions. Instance alliances authorize operators, set instance-level moderation rules, manage federation policy with peers, and govern infrastructure decisions.
 
Instance alliance authority is bounded by Foundation Alliance authority on platform-level matters (baseline rules, certification status). Within those bounds, instance alliances are autonomous.
 
The operator entity is authorized by the instance alliance through a signed attestation. Operator transitions are instance alliance actions. The operator has irreducible infrastructure authority (running servers, deploying code, handling support) but not protocol authority (which is foundation-level) or instance-policy authority (which is instance-alliance-level).
 
### Community layer
 
Communities govern themselves through their own configured governance modules. The platform provides primitives; communities choose templates and configurations that match their existing structures. Community-level decisions — who joins, what content is permitted, how disputes are handled, what fees are charged — are made by community members through their own processes.
 
Communities have authority over their own affairs. Other layers cannot override community-internal decisions except where higher-level rules are violated (the multi-level rule system in moderation).
 
### Alliance layer
 
Alliances are governance structures composed of multiple communities (or, recursively, alliances of alliances). Alliances have their own keypairs, governance, and authority over alliance-level decisions. Member communities retain their own internal governance; alliance authority is over things that affect the alliance as a collective.
 
Foundation Alliance is a specialized alliance — the platform-level governance alliance. Instance alliances are specialized alliances — the per-instance governance alliances. Other alliances exist for whatever purposes their member communities choose (denominational alliances, regional networks, professional associations, etc.).
 
### Authority interactions
 
The layers don’t have a strict hierarchy in the sense that higher always overrides lower. They have different scopes:
 
- **Foundation Alliance scope**: platform-level matters (protocol, certification, baseline rules)
- **Instance Alliance scope**: instance-level matters (hosting, operator selection, instance policies)
- **Alliance scope**: alliance-specific matters (alliance membership, alliance rules, alliance activities)
- **Community scope**: community-internal matters (community governance, content, membership)
- **Lieutenant scope**: subsystem matters (technical decisions within the subsystem’s boundary)
Conflicts between layers are resolved by recognizing scope — a community decision about its internal content cannot be overridden by foundation-level governance unless the content violates foundation-level rules. A foundation-level rule change cannot be made by an instance alliance acting alone. Each layer acts within its own scope.
 
The structural property: no layer can unilaterally take authority outside its scope. The Foundation Alliance cannot dictate community-internal decisions. Communities cannot change foundation rules. Lieutenants cannot rewrite governance protocols (that’s foundation-level). Operators cannot override their instance alliances (that’s the structural protection against operator misbehavior).
 
-----
 
## The lieutenant pattern in detail
 
The lieutenant pattern is the mechanism through which technical authority is distributed without becoming captured. This section specifies it more carefully because it’s load-bearing for the founder-binding story.
 
### Contribution attestations
 
Contributions to the platform are recorded as cryptographically signed attestations. The attestations include:
 
- Who made the contribution (cryptographic identity)
- What the contribution was (commit reference, review reference, etc.)
- When it occurred (timestamp)
- Who attested to it (the relevant subsystem maintainer or reviewer)
- What category (code, documentation, review, design, governance work)
Attestations are public and auditable. They cannot be forged because they are signed by their attestors. They cannot be revoked except through visible action with audit trail.
 
This creates a public record of who has contributed what to the platform. The record is the basis for lieutenant eligibility — someone proposed as a lieutenant for subsystem X must have a record of substantial contribution to subsystem X.
 
The specific thresholds for eligibility are governance decisions (the Foundation Alliance sets and adjusts them). Reasonable defaults: minimum number of contributions over a minimum time period, with quality measured by reviewer attestation rather than just quantity.
 
### Confirmation process
 
When a lieutenant slot is open (either new subsystem or vacancy in existing one), eligible candidates can be proposed. Proposals can come from existing lieutenants, from Foundation Alliance representatives, or from candidates themselves.
 
The Foundation Alliance votes on confirmation. The default threshold is the same as for other Foundation Alliance actions (two-thirds of representatives). Confirmed lieutenants receive cryptographic authority over their subsystem.
 
The Foundation Alliance applies judgment beyond mechanical eligibility. A candidate might be technically eligible but lack the judgment, communication skills, or alignment that the subsystem needs. Foundation Alliance representatives can vote against confirmation even for eligible candidates. Confirmation is not automatic; it requires representative judgment.
 
### Lieutenant authority
 
A confirmed lieutenant has cryptographic authority over their subsystem. Subsystem changes (protocol updates, API changes, security patches) require lieutenant signature. The platform’s protocol enforces this through signature verification.
 
Multiple lieutenants can exist for a subsystem if the subsystem is large enough to need them. Default is one lieutenant per subsystem; growth can lead to additional lieutenants or to splitting the subsystem.
 
Lieutenants are expected to apply judgment within their subsystem. Contentious changes might require lieutenant discussion with other lieutenants or with Foundation Alliance representatives. Non-contentious changes can be made on lieutenant authority alone. The judgment of what’s contentious is part of the lieutenant’s role.
 
### Lieutenant removal
 
A lieutenant who fails to apply appropriate judgment, who acts against platform interests, or who is no longer engaged can be removed through Foundation Alliance threshold vote. Removal is itself a Foundation Alliance action subject to normal procedures.
 
A lieutenant who is removed loses cryptographic authority. Their previous signatures remain valid (subsystem updates they made remain in effect) but they cannot sign new updates. A replacement can be confirmed through the normal process.
 
This is the mechanism through which subsystem authority remains accountable to platform governance. A lieutenant who diverges from the platform’s interests doesn’t have indefinite authority; they can be removed by representatives accountable to communities.
 
### What lieutenants don’t do
 
Lieutenants have authority over their subsystem; they do not have authority over the platform as a whole. They cannot:
 
- Change platform-level governance procedures (that’s Foundation Alliance)
- Override Foundation Alliance decisions (that’s structural)
- Make decisions affecting other subsystems unilaterally (cross-subsystem changes require coordination)
- Act on platform identity (that’s Foundation Alliance through Foundation Alliance MLS group)
A lieutenant’s authority is bounded by their subsystem. The boundary is itself a governance decision; Foundation Alliance can adjust it.
 
-----
 
## Bootstrap: the path from founder control to functioning governance
 
The framework described above is the target state. The current state is one person — the founder — and the framework operates only in degenerate form (single-share Foundation Alliance, founder making technical decisions because there are no lieutenants to confirm). The bootstrap is how the framework comes into actual operation.
 
### Phased growth and the constitutional convention
 
The path from founder control to functioning governance has two parts. The phases describe growth — platform maturity, lieutenant emergence, alliance composition — and align with the technical phase plan in the features document. The constitutional convention is the discrete transition where founder-binding becomes operational. Growth happens incrementally; the transfer of authority does not.
 
**Phase 0 (pre-incorporation through pilot deployment).** Pre-incorporation design work and the cut-down pilot deployment (features Phase 0). Foundation incorporated during this phase with initial small board. Foundation Alliance has zero or one member community. No lieutenants. Founder makes technical decisions; founder is on Foundation board. Founder-controlled.
 
**Phase 1 (single-community MVP shipped).** Full single-community MVP is operational (features Phase 1). Foundation Alliance has one representative (the pilot community). Threshold of one is one. Founder-controlled.
 
**Phase 2 (multi-community on single instance).** Multi-community functionality shipped (features Phase 2). Foundation Alliance composition becomes substantive. First non-founder contributors emerge. First subsystems identified for eventual lieutenant assignment. Founder-controlled. The convention threshold metric is settled by end of this phase, with input from people other than the founder.
 
**The constitutional convention.** A single discrete event triggered by an objective threshold (Foundation Alliance composition, credentialed-population reach, contributor-community milestones — exact metric to be specified by end of Phase 2 with input from people other than the founder). The convention can be triggered during Phase 2 or any phase after, depending on when the threshold metric is met. At the convention, current Foundation Alliance representatives ratify the founding documents and protocols, the Foundation board and Foundation Alliance MLS become jointly operational, the founder’s special structural authority ends, and the entrenchment classes for amendments are settled (see “Constitutional entrenchment” below). The convention is the transition; lieutenant emergence and alliance growth during bootstrap are not themselves founder-binding.
 
The convention is triggered by the metric; convening it is not a unilateral founder decision. The threshold itself is settled by Foundation board members and Foundation Alliance representatives, not by the founder alone.
 
**Phase 3 (External MVP: money and physical features).** Money-touching features added (features Phase 3: payment adapters, ticketing with sybil resistance, physical credential production). Still single-operator or operator-dominant. Whether the convention has been held at this point depends on whether the threshold was met during Phase 2. If not yet, founder-controlled through Phase 3 with the bootstrap acknowledgments still applying.
 
**Phase 4 (Federation: multi-operator).** Cross-instance federation shipped (features Phase 4). Multiple instances exist. Foundation Alliance has substantial composition. The constitutional convention has been held by this point — shipping federation without having met the convention threshold is not reasonable, since federation makes the alliance composition the load-bearing legitimacy mechanism. Foundation board composition equals Foundation Alliance representatives. Lieutenants hold cryptographic authority over their subsystems. Founder may continue to contribute through Strutco or as a contributor but has no special structural authority.
 
**Phase 5 (mature platform).** Framework operates as designed (features Phase 5). Plugin ecosystem fully external. Platform continues regardless of founder’s involvement.
 
### Constitutional entrenchment
 
The Foundation Alliance is the platform’s amending body. Two classes of provisions exist, with different amendment thresholds.
 
**Ordinary provisions.** Amendable by the standard Foundation Alliance two-thirds threshold. Most operational details, parameter tunings, governance interfaces, certification criteria, subsystem boundaries.
 
**Entrenched provisions.** Amendable only at a constitutional convention. The entrenched class is the core substrate guarantees the platform must not silently lose:
 
- Operator-blindness (end-to-end encryption — operators cannot inspect what they host)
- No-skim and no-extraction-beyond-pricing (revenue is bounded by community-affordable pricing; no taking of value beyond what’s needed to operate)
- No attention-auction (no advertising, no engagement optimization, no data brokerage)
- The people-vote principle (one person, one vote; per-community behavioral reputation; universal identity verification through humanness; cross-community sybil resistance)
- The Strutco transfer restriction (the capital-structure lock described in the organizational structure document)
- The convention mechanism itself (the existence and operation of conventions, and the entrenchment classes themselves)
**Conventions recur on a fixed interval** — every 10–20 years, exact interval set at the founding convention — and may be convened off-cycle by a high threshold (TBD; significantly higher than the ordinary two-thirds threshold; settled at the founding convention).
 
**Haven does not use a golden share or external veto-share.** Constitutional entrenchment achieves the same protection democratically rather than through an external custodian. A golden share concentrates the protection in one party whose continued mission alignment must be preserved; entrenchment distributes the protection across the Foundation Alliance representative body, with the requirement that changing entrenched provisions requires convening a constitutional convention rather than a routine vote.
 
**Entrenched does not mean inviolable.** Nothing in the framework is permanently un-amendable. Entrenched provisions are changeable only at a convention, and the founding convention itself decides what is entrenched. The framework’s commitments are durable because changing them requires a discrete formal event, not because they are technically impossible to change.
 
**Entrenchment provides no protection before the first convention.** During the bootstrap period, the entrenched provisions are documented intentions rather than operational constraints. The honest acknowledgment that the framework operates in degenerate form before the convention applies here too — entrenchment is what binds the framework after the convention, not before.
 
### The honest acknowledgment
 
During the bootstrap period (Phases 0 through 2, and possibly Phase 3 depending on when the convention threshold is met), the structural protections exist on paper but not in practice. The founder has effective control over the project. This is the period when comparable failed projects (Couchsurfing, Bitcoin Foundation, early Mastodon) most resemble Haven.
 
The protection in this period is:
 
- Public documentation of the design (so future contributors understand what was intended)
- The founder’s personal commitment to the framework
- The Public Benefit Corporation structure of Strutco (legal obligation to pursue stated public benefit)
- The 501(c)(3) charitable-asset dedication (charitable purpose constrains the Foundation regardless of governance state — an external, attorney-general-enforceable constraint that does not terminate at the founder)
- The trademark held by the Foundation (separation of brand from founder)
- The open source licensing (community can fork if framework is abandoned)
None of these are structural in the sense the math-enabled democracy is structural. They are softer protections that depend on the founder’s continued mission alignment plus external legal constraint. The convention is the transition from soft to structural protection.
 
This is acknowledged because hiding it would be dishonest. The framework is being designed; it is not yet operating.
 
### Initial Foundation Alliance composition
 
The pilot community is the first member of the Foundation Alliance. During bootstrap, the 0.01% threshold is effectively zero — any institutionally credentialed entity that applies and meets qualitative criteria can join. Additional members are accepted through normal Foundation Alliance composition changes (MLS proposals, threshold confirmation).
 
Threshold dynamics become meaningful when composition reaches, roughly, five to ten members. Before that, the multi-signature requirement is mechanically satisfiable but doesn’t represent meaningful diversity of judgment. The bootstrap period is acknowledged as functionally founder-controlled regardless of formal composition.
 
The bootstrap risk: if Foundation Alliance composition grows slowly, the founder’s effective control persists for longer than the design intends. Mitigation: deliberate effort to attract diverse representative entities. The publicity and outreach work explicitly serves this purpose — not just attracting users but attracting Foundation Alliance representatives that diversify governance.
 
As the platform grows, the 0.01% threshold becomes meaningfully constraining and Foundation Alliance composition naturally selects for significant constituencies. Small early-adopter communities that joined during bootstrap may find themselves below the threshold at later recomputation events; they retain platform participation but not Foundation Alliance membership unless they grow or merge into larger alliances.
 
### Initial lieutenants
 
Lieutenants don’t exist meaningfully until there are contributors beyond the founder. The bootstrap requires:
 
- Open source contributors with sustained engagement
- Specific subsystems with sufficient complexity to warrant dedicated maintenance
- Contribution attestation records demonstrating eligibility
- Foundation Alliance composition sufficient to apply meaningful judgment on confirmation
This is a multi-year process under realistic assumptions. The framework specifies the destination; the path is incremental.
 
### The single-share Foundation Alliance period
 
When Foundation Alliance has only one or few members, the “two-thirds threshold” is trivially satisfied. This is functionally founder control. The framework exists in form but not in substance.
 
The acknowledgment: this is unavoidable. Bootstrapping to multi-share authority requires multiple shareholders. There is no protocol-level fix for the period before they exist. The protection during this period is the founder’s commitment, the public documentation, the 501(c)(3) charitable-asset dedication, and the explicit plan to convene the constitutional convention.
 
-----
 
## How this binds against the founder-binding failure patterns
 
The founder-binding analysis identified specific patterns of failure in comparable projects. The framework here addresses them as follows.
 
**Personal commitment is not durable.** The framework does not depend on personal commitment from the founder beyond bootstrap. Once Foundation Alliance composition is meaningful, technical and governance authority is cryptographically distributed. The founder’s continued alignment is structurally enforced rather than relying on continued good intentions.
 
**Founder controls everything until they don’t.** This is the Couchsurfing pattern — founder authority held indefinitely until transferred under pressure or never transferred at all. The framework addresses it through a constitutional convention triggered by an objective threshold settled with input from people other than the founder. The convention is a discrete event at which the founder’s special structural authority ends and the Foundation board plus Foundation Alliance MLS become jointly operational. The convening is not a unilateral founder decision; the metric and threshold are set during Phase 2 by the Foundation board and emerging Foundation Alliance representatives.
 
**Revenue dependency creates capture.** The framework addresses this by separating governance authority from revenue. Strutco’s revenue is bounded by community-affordable pricing; the Foundation’s revenue is diversified across certification fees, donations, and grants. Neither entity becomes dependent on a single revenue source that could be used as leverage.
 
**Technical architecture matters more than legal architecture.** The framework leans heavily on this. The math-enabled democracy is cryptographic; subsystem authority is cryptographic; lieutenant signatures are required for protocol changes. These are technical commitments that legal arrangements alone could not enforce.
 
**Lieutenants and distributed authority are how plugin ecosystems sustain mission.** The lieutenant pattern is explicit in the framework. Subsystems have designated maintainers with real cryptographic authority. The founder cannot remain the single decision-maker for all subsystems; subsystem authority transfers to confirmed lieutenants as the platform grows.
 
**Foundation needs to be real, not nominal.** The Foundation Alliance representative structure makes the Foundation substantive. Board members are accountable to the communities that designated them; communities can replace representatives unilaterally. This is more responsive than typical foundation boards.
 
**Founder transition mechanisms must be designed in advance.** The constitutional convention is the mechanism. Phases 0–2 are descriptive growth stages leading up to the convention; the convention itself is the transition. The founder’s exit from special structural authority is planned through this discrete event, not reactive.
 
**Open licensing creates exit options.** AGPL for server, GPL or equivalent for client. The community can fork if the framework is abandoned. The framework is durable but not absolute; if it fails, the work doesn’t disappear.
 
**Founder-as-CEO-of-competing-commercial-entity is a failure mode.** This is the WordPress/Mullenweg pattern. The framework addresses it by separating Strutco from the Foundation, with the founder leaving the Foundation board at the constitutional convention. The founder may remain at Strutco indefinitely; Strutco is one operator among many, and the Strutco capital-structure transfer restriction (see organizational structure document) prevents Strutco itself from being sold or used as an acquisition vector.
 
**Plugin governance is platform governance.** The plugin certification process and plugin directory governance are Foundation Alliance functions. Plugin authors have stake in platform governance through their contribution attestations. The plugin system is not separable from platform governance.
 
-----
 
## What this framework does not provide
 
The framework is layered protection, not absolute guarantee. Honest acknowledgment of limits:
 
**Pre-convention vulnerability.** During the bootstrap period (Phases 0–2, possibly into Phase 3 depending on when the convention threshold is met), the framework operates in degenerate form. The founder has effective control. Comparable failed projects failed during this stage. The mitigation is the founder’s personal commitment plus the 501(c)(3) charitable-asset dedication plus the explicit plan to convene; the framework itself cannot enforce its design while in single-share form. Constitutional entrenchment, similarly, provides no protection before the first convention — entrenched provisions are documented intentions during bootstrap rather than operational constraints.
 
**Threshold compromise.** If more than one-third of Foundation Alliance representatives are compromised, coerced, or convinced to vote against mission, the framework fails. The framework does not protect against this; it depends on representative communities maintaining mission-aligned judgment.
 
**Capture through hiring.** A well-funded actor could employ enough contributors to influence technical direction, particularly during the period when the contributor community is small. The economic constraint on Strutco (community-affordable pricing caps revenue) provides partial protection; the broader contributor community provides additional protection over time. During bootstrap, this risk is real.
 
**Sufficient representatives offline.** Foundation Alliance whose representatives are insufficiently engaged can deadlock. The recourse is the schism path (active representatives can leave for a new alliance, with continuity loss for any history before the schism). The default two-thirds threshold balances security against operational resilience.
 
**Coordination failures.** Even with engaged representatives, coordinating signature collection for Foundation Alliance actions requires functional communication channels. If the platform itself is being used to suppress Foundation Alliance communications (hostile founder during bootstrap), out-of-band coordination is necessary. This is awkward but possible; the framework relies on it working when needed.
 
**Adversarial fork.** Open licensing allows any party to fork. A well-resourced fork with the wrong intentions could attract users. The protection is brand (Foundation holds trademark), federation (forks are easily distinguished), and community judgment (institutions know whom they trust).
 
**Founder unavailability during bootstrap.** If the founder becomes unavailable before the constitutional convention, the project’s continuation depends on whoever takes over. The Foundation’s initial board has authority to continue, but the project’s substantive direction depends on the people involved.
 
-----
 
## Relationship to other documents
 
This document specifies the governance framework that other documents reference.
 
The **organizational structure** document describes the legal entities (Strutco PBC, Haven Foundation 501(c)(3)) and their relationships. The math-enabled democracy framework described here is what the Foundation board and Foundation Alliance representatives operate through. Foundation board composition tracks Foundation Alliance composition; the two are the same governance body operating in different legal and cryptographic contexts.
 
The **founder-binding analysis** describes the historical patterns of failure in comparable projects. This framework is designed against those patterns; the analysis informs which structural protections are necessary.
 
The **federation readiness** document describes the technical commitments for instance alliances and operator transitions. The instance alliance pattern in that document is a specialized application of the math-enabled democracy framework — instance alliances are MLS groups with threshold multi-signature, just like Foundation Alliance. The structural pattern is consistent across the platform.
 
The **architectural commitments** document references the foundation governance and math-enabled democracy at the highest level. This document fills in the specifics.
 
The **feature list** references the moderation system, which depends on the rule-making authority at each level (community, alliance, instance, foundation). The Foundation Alliance baseline rule set is one of the things the math-enabled democracy framework governs.
 
-----
 
## Open questions
 
Several aspects of the framework need detailed design before implementation:
 
**Specific cryptographic primitive for anonymous voting.** The framework specifies BBS with per-verifier-linkability for nullifier-based double-vote prevention. The exact protocol (signature scheme, nullifier construction, verification mechanism) needs specification. This is shared work with the ticketing redesign.
 
**Specific governance interface for Foundation Alliance.** How do representatives actually coordinate to sign attestations? UI for “here is an attestation that needs your signature; here is the context; vote yes/no/abstain”? Coordination protocol for distributing drafts and aggregating signatures?
 
**Contribution attestation format.** What specifically gets attested? How are attestations stored and queried? What’s the eligibility computation from raw attestations?
 
**Lieutenant subsystem boundaries.** Initial subsystems suggested above are not final. The Foundation Alliance will need to specify boundaries when lieutenants are first confirmed.
 
**Foundation Alliance composition mechanics.** The 0.01% threshold rule is specified, but operational details need work: exact cryptographic protocol for membership count verification, audit-on-demand process, handling of borderline cases, what happens between recomputation events if an entity’s membership changes dramatically, the relationship between “platform institutionally credentialed population” and federated instances where exact counts may not be available in real time.
 
**Convention trigger metric.** The convention is triggered by an objective threshold, but the specific metric is not yet set. Candidates include Foundation Alliance composition count, credentialed-population reach, contributor community size, specific milestone events, or some combination. The metric must be settled by end of Phase 2 with input from people other than the founder.
 
**Parameter governance.** Thresholds (two-thirds default), eligibility criteria, subsystem boundaries — these are themselves governance decisions. How can they be changed? Foundation Alliance vote? Higher threshold for changing parameters than for normal decisions?
 
**Foundation Alliance lifecycle items in active design:**
 
- *Grandfathering for early adopters.* Communities that joined the Foundation Alliance during bootstrap need protection against sudden disenfranchisement as the 0.01% threshold becomes meaningfully constraining. Options include permanent founding member status, aggregation incentives, decay schedules, or constitutional protection. Probably some combination. Resolution at the founding convention.
- *Schism threshold tuning for early phases.* Default schism thresholds tuned for mature operation may be wrong for early-life Foundation Alliance dynamics. Small composition means small numbers of dissenting representatives can trigger schism, possibly inappropriately. Thresholds likely need phase-specific tuning.
- *Off-cycle convention threshold.* Conventions recur on a fixed interval (10–20 years, set at the founding convention), but the threshold for convening off-cycle has not been set. It must be significantly higher than the ordinary two-thirds threshold; specifics settled at the founding convention.
- *Convention interval.* The recurring-convention interval (somewhere in 10–20 years) is set at the founding convention itself.
These items must be settled by end of Phase 2 (when cross-community interactions and alliance functionality make Foundation Alliance load-bearing, and when convention-readiness is approaching). The specific trigger and timeline for the founding convention should be determined with input from people other than the founder. A computational framework for stress-testing how groups behave under various threshold structures has been considered as a way to ground threshold-tuning decisions in evidence rather than intuition.
 
**Schism handling for Foundation Alliance.** If Foundation Alliance representatives disagree fundamentally, can the Foundation itself schism? What does that mean legally and cryptographically?**Cross-credential interaction.** Different institutions may issue overlapping credentials. How does the framework handle cases where someone has credentials from multiple sources that grant different voting weight?
 
These questions are real and need answers before the framework is fully operational. They do not need to be answered now — the framework specifies the architectural commitments; the implementation details can be developed as the platform grows toward needing them.
