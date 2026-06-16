# Organizational structure

## Contents

- [Purpose](#purpose)
- [The two-entity model](#the-two-entity-model)
- [What each entity does](#what-each-entity-does)
- [Foundation governance: the alignment with the math-enabled democracy framework](#foundation-governance-the-alignment-with-the-math-enabled-democracy-framework)
- [Bootstrap: the path from solo developer to functioning structure](#bootstrap-the-path-from-solo-developer-to-functioning-structure)
  - [Phase 0 (now through pilot deployment)](#phase-0-now-through-pilot-deployment)
  - [Phase 1 (single-community MVP shipped)](#phase-1-single-community-mvp-shipped)
  - [Phase 2 (multi-community on a single instance)](#phase-2-multi-community-on-a-single-instance)
  - [The constitutional convention](#the-constitutional-convention)
  - [Phase 3 (External MVP: money and physical features)](#phase-3-external-mvp-money-and-physical-features)
  - [Phase 4 (Federation: multi-operator)](#phase-4-federation-multi-operator)
  - [Phase 5 (mature platform)](#phase-5-mature-platform)
- [How this binds against the patterns the analysis identified](#how-this-binds-against-the-patterns-the-analysis-identified)
- [What this structure does not provide](#what-this-structure-does-not-provide)
- [Legal and compliance posture](#legal-and-compliance-posture)
- [Relationship to other documents](#relationship-to-other-documents)

-----

## Purpose

This document specifies the organizational architecture for Haven: the legal entities, their relationships, their governance, and the bootstrap path from solo developer to functioning multi-stakeholder structure. It addresses questions that the architectural and federation-readiness documents reference but do not fully specify — what the foundation is, what Strutco is, how they relate, how lieutenants get authority, and how the structure is supposed to bind future founders against the pattern of comparable projects whose commitments weakened over time.

The structure is designed against the historical pattern documented in the founder-binding analysis. Projects that intended to remain mission-aligned have failed when their organizational arrangements gave one party (typically the founder or the founder’s company) enough authority to override the original commitments. The structure here aims for layered protection: cryptographic, legal, economic, and social mechanisms that each provide partial protection and together raise the cost of capture substantially.

-----

## The two-entity model

Haven is organized through two distinct legal entities with separate boards, separate funding, separate operations, and complementary roles.

**Strutco** is a California Public Benefit Corporation. It is the first operator of the Haven platform. Its purpose, stated in its articles of incorporation, is operating federated civic infrastructure with end-to-end encryption and pluggable governance for institutional communities. As a PBC, Strutco’s directors have legal obligation to pursue this public benefit alongside (not subordinate to) shareholder interests. Strutco employs developers, operates infrastructure, produces physical credentials, and generates revenue.

**Haven Foundation** is a 501(c)(3) tax-exempt organization. Its purpose is charitable, educational, and scientific: advancing federated civic infrastructure, supporting community institutions, advancing privacy-preserving technology. The Foundation holds the Haven trademark, maintains the protocol specification, operates the operator and plugin certification processes, holds platform-level governance authority through the math-enabled democracy framework, and provides default trust framework for credentials. It is legally separate from Strutco and structurally independent.

The two entities are necessary because their purposes are different and the legal forms have different constraints. A 501(c)(3) cannot operate for the benefit of private interests, which precludes it from being a commercial operator. A for-profit (even a PBC) cannot serve as the trust-anchor for a platform that claims to be founder-resistant — its shareholders’ interests would eventually conflict with that role. Splitting the functions across two entities lets each entity be the right shape for its role.

-----

## What each entity does

**Strutco’s responsibilities:**

- Operating the initial Haven instance, including hosting, technical operations, and customer support for early adopting communities
- Producing physical credentials (member cards, ID artifacts) for communities that order them
- Employing the founder and (over time) additional developers and operations staff
- Generating revenue through community dues, credential production, and possibly future services
- Compliance with applicable law as the operating entity (consumer protection, payment processing rules, data protection)
- Contributing financially to Haven Foundation as a donor (not as a controller)
- Operating one instance among future many; Strutco is structurally one operator and is not the platform itself

**Foundation’s responsibilities:**

- Holding the Haven trademark, with licensing terms that any operator must accept
- Maintaining the canonical protocol specification
- Operating the operator certification process, including criteria, review, and ongoing monitoring
- Operating the plugin certification process, including security review and ongoing monitoring
- Operating the humanness-issuer certification process: instance alliances issue humanness credentials, and certification of the *issuer* (not the operator — the two are distinct roles) is the collateral behind the sybil-resistance model. Certification covers verification rigor and issuance-rate transparency; discovered inflation triggers decertification, after which federation peers can stop honoring that issuer’s humanness credentials and its communities’ Foundation-level weight is discounted. This is the lever that lets humanness issuance be single-signer rather than threshold (see the architecture document’s Issuers section): the defense against a corrupt issuer is detection plus decertification, not key-splitting. Innocent communities hosted under a decertified issuer retain the migration exit (re-present external proof, re-issue elsewhere)
- Holding platform-level governance authority through the math-enabled democracy framework (the Foundation Alliance described below)
- Providing default credential trust framework that communities and operators can adopt or modify
- Operating public verifiable logs as platform infrastructure — equivocation defense, replica integrity attestations, audit trails for moderation and governance decisions (analogous to Certificate Transparency in the web PKI)
- Possibly operating base-tier replica server infrastructure as public service for small instances handling viral public content (design direction in active development)
- Supporting public-facing documentation, education, and outreach about civic platforms
- Receiving donations from operators, plugin authors, individuals, and grants from aligned foundations

**What neither entity does:**

- Neither entity controls the other in any meaningful operational sense
- Neither entity makes decisions on behalf of communities about their internal governance
- Neither entity is the platform; the platform is the federated network of instances and communities

-----

## Foundation governance: the alignment with the math-enabled democracy framework

The Haven Foundation board is the same body as the Foundation Alliance representatives in the math-enabled democracy framework described in the architectural documents. This alignment is structural and intentional.

The reasoning: the math-enabled democracy framework specifies that platform-level governance authority is held by representatives of member communities, exercising authority through threshold multi-signature. This is the Foundation Alliance. The Foundation, as a legal entity, requires a board for governance under 501(c)(3) law. Having two parallel governance structures (one cryptographic-democratic, one legal-corporate) would create ambiguity about which actually controls the Foundation. Aligning them eliminates the ambiguity.

In practice: each Foundation Alliance member (a community or alliance meeting the 0.01% threshold criterion specified in the governance framework) designates a specific human as their representative. That human serves on the Foundation board. The member entity can replace their representative at any time through their own governance. Foundation board composition tracks Foundation Alliance composition; changes flow through normal MLS-based composition operations.

This creates accountability that’s unusual for foundations. Standard 501(c)(3) boards are self-perpetuating (existing board chooses new board members). The Foundation’s board is community-perpetuating — board members are accountable to the communities that designated them. A community unhappy with their representative can replace them without needing the rest of the board to agree.

It also creates a unified authority chain. Foundation board votes (legal authority) and Foundation Alliance multi-signature operations (cryptographic authority) reflect the same underlying decisions by the same people. The Foundation cannot do legally what its representatives do not also do cryptographically; the Foundation Alliance cannot exercise authority that the legal Foundation does not also recognize.

-----

## Bootstrap: the path from solo developer to functioning structure

The structure described above is the target state. The current state is one person — the founder — without any of the entities incorporated, without representative communities to populate the Foundation Alliance, without contributors to be lieutenants. The bootstrap path is how the structure comes into existence.

### Phase 0 (now through pilot deployment)

Phase 0 covers two periods of project work without an operational distinction at the organizational level: pre-incorporation design work, and pilot deployment (Phase 0 in the features doc — a cut-down subset of single-community functionality). The founder controls everything throughout. Phase 0 ends when the single-community MVP (features Phase 1) ships.

**Founder’s situation:** Solo developer doing design work, then operating the pilot instance through Strutco. Strutco exists as a California LLC (planned conversion to PBC pending board formation and mission articulation). Foundation not yet incorporated at the start; incorporated during Phase 0. Pilot community begins using a cut-down version of the platform partway through Phase 0.

**Goals for this phase:**

- Complete substantial design work that is publicly documented
- Articulate Strutco’s public benefit mission specifically enough to inform the LLC-to-PBC conversion
- Identify a small number of people who would serve on an initial Foundation board
- Identify a small number of people who would be early contributors to the codebase
- Establish working relationships with potential pilot communities
- Build initial public presence so the project is discoverable
- Ship the pilot to the first community and validate core architectural decisions through real use

**Specific organizational steps:**

- Convert Strutco from LLC to California Public Benefit Corporation once mission and initial governance are articulated
- Incorporate Haven Foundation as a 501(c)(3) once initial board is identified
- Constitute the initial Foundation board of three to five members, including the founder and at least two others committed to the project’s mission
- Begin transferring trademark and protocol specification to the Foundation
- Begin building publication of design documents and outreach
- Engage legal counsel to handle incorporation steps and review founding documents
- Pilot community becomes the first member of the Foundation Alliance (one representative; threshold of one is one) when the pilot ships
- Open source repositories established with appropriate licensing (AGPL for server, GPL or similar for client) before or at pilot launch

**Acknowledged limitations:**

- The founder controls both Strutco and (effectively) the initial Foundation board
- There are no lieutenants because there are no contributors yet
- The math-enabled democracy framework is documented but not operational
- The structural protections exist on paper but the actual authority chain still terminates at the founder
- Foundation Alliance has zero or one member; threshold dynamics are meaningless
- This is the period that comparable failed projects (Couchsurfing, Bitcoin Foundation, Mastodon early years) most resemble

The bootstrap risk is real and is acknowledged. The structural protections are designed to take effect at the constitutional convention, which is triggered when the threshold metric is met (anywhere from Phase 2 onward); before then, the project depends on the founder’s continued commitment to the design and on the 501(c)(3) charitable-asset dedication as an external constraint.

### Phase 1 (single-community MVP shipped)

**Founder’s situation:** The full single-community MVP has shipped (features Phase 1 complete). The pilot community is using the complete single-community platform. Foundation exists with initial board.

**Goals for this phase:**

- Validate core architectural decisions through real use at full single-community scope
- Begin attracting open source contributors based on public documentation
- Build relationships with additional potential communities (for the multi-community work coming in Phase 2)
- Refine the design based on pilot experience at full feature scope

**Specific organizational steps:**

- Pilot community continues as the first member of the Foundation Alliance
- Contributor onboarding documented; first non-founder contributors begin participating
- Operator certification criteria drafted (will not be exercised until other operators emerge)
- Plugin certification criteria drafted (will not be exercised until plugins emerge)

**Acknowledged limitations:**

- Foundation Alliance has one or few members; threshold dynamics are meaningless
- Lieutenants do not yet exist in any meaningful sense
- The founder still effectively controls all technical decisions
- Strutco still controls the only operating instance

### Phase 2 (multi-community on a single instance)

**Founder’s situation:** Multiple communities are using the platform on Strutco’s instance (features Phase 2: multi-community on single instance). Some additional contributors exist. The founder is shifting from sole implementer to coordinator of a small contributor community.

**Goals for this phase:**

- Foundation Alliance composition becomes meaningful (multiple communities)
- First non-founder contributors with sustained engagement
- First subsystems have identified maintainers who could become lieutenants
- Initial operator certification criteria validated through Strutco itself
- Convention threshold metric is settled by end of Phase 2 with input from people other than the founder

**Specific organizational steps:**

- Foundation Alliance accepts additional representative communities through normal MLS-based composition changes
- Initial lieutenants identified for specific subsystems (probably the founder’s role decomposes into specific subsystem ownership for purposes of contribution attestation, even though lieutenant authority is not yet operational)
- Convention threshold metric (the objective trigger for convening) is established by Foundation board and emerging Foundation Alliance representatives — not by the founder unilaterally
- Foundation Alliance lifecycle items (grandfathering, schism thresholds for early phases, off-cycle convention threshold, convention interval) are surfaced for resolution at the founding convention

**Acknowledged limitations:**

- Strutco is still the dominant operator (probably the only one)
- The contributor community is small and likely overlaps significantly with Strutco employment
- Foundation Alliance is still small; threshold dynamics are still relatively brittle
- The framework’s transition has not yet occurred unless the convention threshold is met during this phase
- The structural protections begin to take shape but do not yet function (until the convention)

### The constitutional convention

The convention is triggered by the objective threshold metric (settled by end of Phase 2). When the metric is met, the convention follows. This can happen during Phase 2, or any phase after — the convention is scheduled by threshold, not by phase. At the convention:

- Current Foundation Alliance representatives ratify the founding documents and protocols
- The Foundation board and Foundation Alliance MLS become jointly operational (the board’s composition equals Foundation Alliance representatives from this point forward)
- Lieutenants are formally confirmed by Foundation Alliance representative vote with real cryptographic authority over their subsystems
- The founder’s special structural authority ends; the founder leaves the Foundation board cleanly
- The entrenchment classes are settled (which provisions are ordinary, which are entrenched; what the off-cycle convention threshold is; what interval recurring conventions take — 10–20 years)
- The Strutco capital-structure transfer restriction (described below in “Economic and capital-structure constraint as structural protection”) becomes entrenched

The convening is not a unilateral founder decision; the metric is met or not, and the convention follows. Once held, the framework operates as designed. The convention is the discrete event at which founder-binding becomes operational. The phases described below assume the convention has been held; if Phase 3 or 4 begins before the convention threshold is met, the bootstrap acknowledgments continue to apply through whichever phase the convention happens in.

### Phase 3 (External MVP: money and physical features)

**Founder’s situation:** The platform adds money-touching and physical features (features Phase 3: payment adapters, ticketing with sybil resistance, physical credential production, payment-related reputation signals). Strutco is still the dominant or sole operator; multi-operator hasn’t yet begun.

**Goals for this phase:**

- Compliance work completed for payment processing and credential production
- Physical credential production partnerships established
- Payment adapter ecosystem begins to function
- External integration adapters (good standing tracking, etc.) shipped

**Specific organizational steps:**

- Strutco’s revenue model fully operational (community dues, credential production, payment workflow coordination)
- Foundation revenue from certification fees begins (if other operators or plugins emerge)
- Legal compliance posture for money-touching and youth-serving features finalized

**Acknowledged limitations:**

- Strutco is still the dominant operator; multi-operator dynamics haven’t yet emerged
- The contributor community is still relatively small
- Foundation Alliance composition is meaningful but not yet at scale
- Whether the constitutional convention has been held at this point depends on whether the threshold metric has been met — Phase 3 may straddle the convention

### Phase 4 (Federation: multi-operator)

**Founder’s situation:** Multiple instances exist, operated by different parties (features Phase 4: federation protocol, cross-instance alliances, community migration between instances). Strutco is one operator among several. Foundation has substantial Foundation Alliance composition. Lieutenants exist for major subsystems. The constitutional convention has been held by this point (it should not be reasonable to ship federation without having met the convention threshold); founder-binding is operational.

**Goals for this phase:**

- Operator certification process exercised on real applications
- Plugin ecosystem begins to emerge
- Foundation funding diversifies beyond Strutco contributions
- Founder’s role becomes one among many, structurally and practically

**Specific organizational steps:**

- Foundation board composition equals Foundation Alliance representatives (settled at the convention)
- Lieutenant structure formalized with explicit subsystem boundaries and authority
- Operator certification revenue and donation base sustain Foundation independent of Strutco contributions
- Plugin certification revenue grows

**At this point:**

- The founder no longer has unilateral authority over technical direction (lieutenants hold subsystem authority)
- The founder no longer has unilateral authority over Foundation governance (board is Foundation Alliance representatives)
- The founder is no longer on the Foundation board (exit was clean at the convention; operators are ineligible for Foundation Alliance membership, so the founder does not return via a “Strutco representative” path)
- The founder retains operational authority within Strutco as CEO, but sale and controlling-share transfer are restricted under Strutco’s PBC charter (see “Economic and capital-structure constraint” below). Strutco is one operator among many.
- The structural protections are substantively functional

### Phase 5 (mature platform)

The structure operates as designed. The founder may continue to be involved as Strutco CEO, as an active contributor, as a public advocate. The founder’s continued involvement is by choice and contribution rather than by structural necessity. The platform continues without the founder if the founder steps back.

-----

## How this binds against the patterns the analysis identified

The founder-binding analysis identified several patterns of failure in comparable projects. The structure here addresses them as follows.

**Personal commitment is not durable.** The structure does not depend on personal commitment from the founder beyond the bootstrap period. The constitutional convention is the discrete event at which the founder’s continued mission-alignment stops being a load-bearing assumption; after the convention, the math-enabled democracy framework operates through cryptographic authority chains rather than through the founder’s good intentions. Before the convention, the 501(c)(3) charitable-asset dedication is an additional external constraint that does not depend on the founder, but the cryptographic framework itself operates only in degenerate form.

**Tax-exempt status as additional external constraint.** The structural protections do not depend solely on 501(c)(3) status — the cryptographic math-enabled democracy framework is the primary binding, operating through cryptographic authority chains that exist regardless of legal classification. But the 501(c)(3) charitable-asset dedication is an additional, external, attorney-general-enforceable constraint that does not terminate at the founder. This is both/and, not either/or. Charitable-asset dedication is especially valuable during the bootstrap period, when the cryptographic framework operates in degenerate form and the soft protections carry more weight. If the Foundation loses 501(c)(3) status, the platform’s cryptographic governance continues to function, but an external check on diversion of charitable assets is lost.

The Foundation’s mission-critical assets — the Haven trademark, the canonical protocol specification, the platform signing keys, the certification authority — are irrevocably dedicated to the charitable purpose. They may not be transferred for private benefit, and on dissolution they pass only to a like-purpose charity. Exact mechanism (asset-lock language in the Articles, board fiduciary duties, dissolution clause) needs counsel; the principle is that these assets are dedicated to the charitable purpose, not held at the discretion of any future board.

**Revenue dependency creates capture risk.** Strutco’s revenue from community dues at community-affordable prices does not scale to levels that would let it dominate the platform. The Foundation’s revenue from certification fees, diversified donations, and grants is intentionally distributed. Neither entity becomes dependent on a single revenue source that could be used as leverage.

**Foundation-owns-Subsidiary structure is the strongest formal arrangement.** Haven inverts this somewhat — the Foundation does not own Strutco — but the math-enabled democracy framework plus the Strutco transfer restriction provides a stronger constraint. The Foundation cannot make platform-level changes without Foundation Alliance representative consent, regardless of what Strutco wants. Strutco cannot be sold or have controlling interest transferred without Foundation consent (a restriction entrenched at the constitutional convention). The two constraints — Foundation cannot act unilaterally on protocol; Strutco cannot be transferred unilaterally — together protect both sides of the entity separation.

**Technical architecture matters more than legal architecture.** The structure leans heavily on this. End-to-end encryption, math-enabled democracy threshold signatures, lieutenant subsystem authority, federation, schism as first-class operation — these are technical commitments that legal arrangements alone could not enforce. The legal arrangements (PBC structure, Foundation, separation of entities) reinforce the technical commitments rather than substituting for them.

**Distributed governance creates pressure on formal authority.** The Foundation Alliance structure creates this directly. Foundation board members are accountable to the communities that designated them; communities can replace their representatives unilaterally. This is more responsive than the typical foundation board structure.

**Founder transition mechanisms must be designed in advance.** The constitutional convention is the transition mechanism, designed in advance. Phases 0–2 are growth stages leading up to the convention; the convention is the discrete event at which founder-binding becomes operational. Convening is triggered by an objective threshold set with input from people other than the founder. The founder’s exit from special structural authority is planned through this event, not reactive.

**Open licensing of contributions creates exit options.** AGPL for server code and GPL (or similar copyleft) for client code prevents enclosure. Communities can fork the platform if Foundation governance becomes captured. The plugin interface terms similarly require license compatibility to prevent enclosure of derivative work.

**Endowment building reduces dependency on future fundraising decisions.** The structure does not yet specify endowment-building, but it should over time. Foundation revenue from certification fees, if it grows to sustain operations, reduces ongoing fundraising pressure. Larger donations and grants could establish an endowment.

**Profitable business at the core can be an alternative to endowment.** Strutco’s profitability, if achieved at the modest scale the economics support, provides ongoing funding for Foundation contributions and broader platform sustainability without making the Foundation depend on any particular operator’s success.

**Narrow technical missions are easier to defend.** Haven’s mission is somewhat broader than Mullvad’s or Let’s Encrypt’s but narrower than Mozilla’s. The civic-institutional focus provides reasonable boundaries that reduce drift risk.

**Brand protection should be separate from operational protection.** The Haven trademark is held by the Foundation, not by Strutco. This separates brand authority from operational authority. The founder does not personally hold the trademark in the Linux model because the founder’s role is intended to be transient; the Foundation is the durable holder.

**Founder-as-CEO-of-competing-commercial-entity is a structural failure mode.** This is the WordPress/Mullenweg pattern explicitly addressed. The founder is CEO of Strutco (commercial entity) during the bootstrap phase, but the constitutional convention is the explicit transition: at the convention, the founder leaves the Foundation board cleanly, and operators (including Strutco) are ineligible for Foundation Alliance membership, so there is no back-door path back onto the Foundation board through “Strutco’s representative.” The founder may remain at Strutco as the company’s CEO indefinitely; the protection is that Strutco is one operator among many, the founder has no special platform-level authority through Strutco, and Strutco’s PBC articles restrict its sale or controlling-share transfer (so Strutco cannot itself become an acquisition vector).

**Lieutenants and distributed authority are how plugin ecosystems sustain mission.** The lieutenant structure is explicit in the math-enabled democracy framework. Lieutenants hold cryptographic authority over their subsystems, confirmed by Foundation Alliance representative vote. The founder cannot unilaterally appoint lieutenants; they must be confirmed by representatives accountable to communities.

**Economic and capital-structure constraint as structural protection.** Strutco’s pricing structure for civic communities limits its revenue ceiling in ways that prevent capital-based capture of the platform. Community-affordable pricing makes the platform accessible to small institutions and also caps Strutco’s ability to dominate the contributor community through hiring.

But pricing alone is not a lock — pricing is a policy choice that an acquirer of Strutco could change. The lock is a transfer restriction in Strutco’s PBC articles: Strutco may not be sold, merged, acquired, or dissolved, may not issue controlling or appreciating equity, and the founder may not transfer a controlling interest in Strutco, without Foundation consent. After the founding convention, changes to this transfer restriction are entrenched — amendable only at a constitutional convention, not by ordinary Foundation Alliance vote.

The transfer restriction is the compensating lock for the deliberate choice that the Foundation does not own Strutco. Without it, Strutco — the entity an acquirer would buy — is otherwise unbound, and capital-affordable pricing is just a current policy. The restriction lives in Strutco’s articles (corporate law), not in any alliance layer. The instance alliance does not have power over Strutco’s capital structure; that would conflate operator infrastructure with platform governance. The Foundation’s consent gate is what restricts capital actions, in keeping with the Foundation’s role as the durable platform-mission entity.

Honest acknowledgment: before the founding convention, the consent gate is weak. The founder controls Strutco and effectively controls the Foundation board, so “Foundation consent” is a documented intention more than an operational constraint. The protection hardens at entrenchment — once the founding convention has settled the transfer restriction as entrenched, changing it requires another convention. The pre-convention weakness is the same bootstrap acknowledgment that runs throughout this document. The mechanism (exact transfer-restriction language, Foundation consent procedure, integration with PBC fiduciary duties) needs counsel.

-----

## What this structure does not provide

The structure is layered protection, not absolute guarantee. Several limitations should be stated honestly.

**Pre-convention vulnerability.** Before the constitutional convention, the structural protections exist on paper but not in practice. The founder during the bootstrap period has effective control over the project. Comparable failed projects (Couchsurfing, Bitcoin Foundation) failed at this stage. Haven’s protection in this period is the founder’s personal commitment, the public documentation of the design, and the 501(c)(3) charitable-asset dedication — the first two soft, the third an external constraint that does not depend on the founder but cannot itself replace the cryptographic framework.

**Threshold compromise.** Foundation Alliance decisions require threshold of representatives. If more than the threshold can be compromised, coerced, or convinced to vote against mission, the protection fails. The structure does not protect against this kind of attack; it depends on the representative communities maintaining mission-aligned judgment.

**Capture through hiring.** Strutco’s economic constraints make this harder but not impossible. A well-funded actor could potentially employ enough contributors to influence technical direction, particularly during the period when the contributor community is small. Mitigation comes from contributor diversity, lieutenant accountability to Foundation Alliance rather than to employers, and the general visibility of who’s doing what work.

**Legal jurisdiction shifts.** California PBC law and US 501(c)(3) law could change in ways that affect the structure. The cryptographic and licensing protections continue to function regardless, but the formal entity structure depends on existing legal forms remaining viable.

**Founder unavailability during bootstrap.** If the founder becomes unavailable before the constitutional convention, the project’s continuation depends on whoever takes over. The Foundation’s initial board would have authority to continue, but the project’s substantive direction depends on the people involved. This is a real risk and the mitigation is partial (documentation, public design work, multiple initial board members).

**Adversarial fork.** Open licensing allows any party to fork the platform. A well-resourced fork with the wrong intentions could potentially attract users away from the Foundation-sanctioned version. The protection here is brand (Foundation holds trademark), federation (forks are easily distinguished from the main platform), and community judgment (institutions know whom they trust).

-----

## Legal and compliance posture

The architecture and organizational structure aim to take legally defensible positions across the major compliance regimes that apply to communication platforms. This section captures the posture; specific implementation requires legal counsel familiar with each area.

**Platform liability for user-generated content (Section 230).** The platform’s structural commitment to end-to-end encryption means operators cannot read community content. Strutco is positioned as a passive conduit for user-generated content, not as a publisher or active moderator. This is the strongest Section 230 position available. Strutco’s active services (credential production, payment workflow coordination) are distinct activities where it acts as a service provider, not as a content platform; the legal analysis for those is separate.

**CSAM compliance.** Federal law (18 USC 2258A) requires reporting apparent CSAM to NCMEC when the platform has actual knowledge. End-to-end encryption means Strutco generally cannot have actual knowledge of encrypted content. Knowledge arises from member reports with cryptographic evidence; the platform’s response is to verify, take action (community-level enforcement), and report to NCMEC where applicable. Strutco registers with NCMEC at incorporation, maintains internal CSAM response policies, and designates a contact person for legal compliance. The EU and UK regulatory trends toward requiring detection in encrypted systems are concerning but not yet enforced; the platform’s position is that structural encryption is legally permissible under current US law.

**Money transmission.** The payment adapter architecture is specifically designed to keep Strutco out of money transmission. Communities are merchants of record for their own transactions; payment processors handle funds; Strutco provides workflow coordination and credential issuance without ever holding user funds. This is structurally important and must be maintained — any future addition of escrow, intermediate holding, or platform-level routing would trigger state money transmitter licensing requirements that are expensive and operationally substantial. The architecture’s commitment to this position is structural, not merely policy.

**Privacy and data protection.** California CCPA/CPRA, GDPR (for EU users), and other state privacy laws all apply. The architecture’s commitments (minimal data collection, end-to-end encryption, user-controlled identity, federation, export and deletion as first-class features) substantially reduce compliance burden because much of what these regimes require is already structural. Compliance documentation (privacy policies, data processing agreements, user-facing mechanisms) is still required and developed before launch.

**Children and youth.** COPPA applies to users under 13 and requires verifiable parental consent. The platform supports youth participation through a guardian role primitive — a community role with specific oversight authority but without content read access. Guardians can vote on community decisions, receive safety alerts, and act on aggregate community information without surveilling specific interactions. For under-13 users, guardian authority maps to COPPA-required parental oversight. For 13-17 users, guardian authority is configurable per community (schools may want more oversight; friend groups may want less). Identity verification for guardian relationships uses the platform’s locality-grounded sybil resistance (institutional verification for schools and religious communities; vouching for informal groups). Specific guardian-role configurations are template- or plugin-based; the core guardian primitive and COPPA compliance mechanisms are platform-level.

**Government information requests.** The architecture limits what Strutco can produce in response to valid legal process. Strutco can produce account registration data (what was collected), login records, federation traffic logs, public content, and payment information for transactions Strutco was involved in. Strutco cannot produce encrypted content (not technically possible), information about communities hosted on other instances, or cross-community correlations beyond what’s stored. The platform publishes transparency reports on requests received and responded to, notifies users about requests where legally permitted, and pushes back on improper requests through legal counsel. The transparency about limits is itself protective — it positions the platform as inappropriate for surveillance use cases.

**Intellectual property and Foundation asset-lock.** The Foundation holds the Haven trademark, with licensing to operators on standard terms. Code is licensed AGPL (server) and GPL or equivalent copyleft (client) to prevent enclosure. Contributor License Agreement is likely required for substantial contributions; specific form to be developed. Trademark search and registration happens at appropriate phases.

The Foundation’s mission-critical assets — the Haven trademark, the canonical protocol specification, the platform signing keys, and the certification authority — are irrevocably dedicated to the charitable purpose under the Foundation’s Articles of Incorporation. These assets may not be transferred for private benefit and, on dissolution, pass only to another 501(c)(3) or like-purpose charity. This is the standard 501(c)(3) asset-lock structure, written explicitly into the Articles to make the dedication unambiguous. Exact language (dedication clause, dissolution clause, board fiduciary duties around mission-critical assets) needs counsel; the principle is that these assets belong to the charitable purpose, not to any future board’s discretion.

The asset lock is the corporate-law complement to the cryptographic governance framework. The framework binds what the platform does; the asset lock binds what happens to the platform’s institutional assets if governance is captured or the Foundation dissolves. Attorney-general enforcement of charitable-asset dedication is an external check that does not depend on Foundation Alliance representatives being available or aligned.

**Jurisdiction.** Strutco is California-based and subject to California law. Foundation jurisdiction is to be determined (likely California or Delaware). International users create compliance obligations under their jurisdictions’ laws (GDPR for EU, UK Online Safety Act provisions, etc.); for early US-only operation (Phases 0–2, single-instance), these are mostly deferrable but become relevant once federation enables cross-jurisdictional instance hosting (Phase 4).

**Insurance.** Strutco and the Foundation each maintain appropriate insurance from incorporation: general liability, professional/E&O, cyber liability, directors and officers coverage. Costs scale with operations; basic coverage during pre-revenue phases is modest.

**Bootstrap-specific exposure.** Early in Phase 0 (pre-incorporation work), the founder is personally exposed for project activities. Public statements should be conservative until corporate entities provide liability separation. Once incorporation completes (still within Phase 0), maintaining corporate formalities is required to preserve the corporate veil. Ongoing legal counsel becomes necessary as operations scale.

**What this section is not.** This is the platform’s compliance posture for orientation, not legal advice. Specific compliance requires counsel with platform/civic-tech experience and knowledge of the relevant jurisdictions. Several areas where the legal landscape is contested or evolving (E2E encryption requirements, money transmitter scope for adapter patterns, COPPA in institutional contexts) need specific legal review before public launch.

-----

## Relationship to other documents

This document is a peer to the architectural commitments document. It specifies the organizational structure that implements the founder-binding claims made in that document.

The math-enabled democracy framework referenced here is documented in the architectural commitments document and elaborated in the founder-binding analysis.

The federation-readiness document specifies the technical commitments that the organizational structure depends on (instance alliance keypair durability, operator transition mechanism, federation protocol).

The founder-binding analysis specifies the historical context against which this organizational structure is designed.

Future iterations may add more detail about specific bylaws, board composition requirements, certification criteria, and bootstrap milestones as those become clearer. This document is intentionally less specific than the architectural documents because the organizational details depend on legal advice, board composition, and operational experience that hasn’t yet been gathered.