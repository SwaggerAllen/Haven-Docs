# Design notes from the civic-tech deep dive

*Captured 2026-06-06. These are the design decisions and directions that surfaced during the civic-tech study — distinct from the [civic-tech landscape](haven-civic-tech-landscape.md), which is a map rather than a decision log. They belong with the study notes as active design directions, recorded so they aren’t lost; several need real design work and at least one (friction-as-ratchet) explicitly needs input beyond the founder. Where a note has since been adopted into the architecture or feature documents, a status line says so.*

-----

## 1. Proposal types are content types

Proposal types, feed types, and content types all reduce to “typed schema plus declared interfaces.” This collapses the plugin surface from a dozen bespoke APIs to **one** surface. The unification is the precondition for plugin DX existing at all — hold onto it. A proposal is a content type that participates in a governance flow; it is not a separate primitive.

## 2. Configuration UX: sliders + presets + customize

Progressive disclosure — a handful of sliders with presets for the beginner, a customize button for granular power-user control. Config inputs/effects are machine-readable, which makes this the *easy* part of governance UX.

**The seam (this is where it gets clunky if undesigned):** in a governed community, dragging a slider is *proposing*, not *setting*. Direct manipulation feels instant; governed change isn’t. Once a community leaves founding state, the slider must quietly become a proposal-drafter: dragging it opens a pre-filled proposal showing current-vs-proposed with a clear diff and an unmissable “takes effect when ratified” affordance. The founding-state exemption buys the cold start (founder just sets things); the hard part is the handoff to governed config.

## 3. Proposal/voting flow: problem/solution framing

The hardest UX problem, and the place clunkiness lurks. Direction:

- **Topic/problem is a first-class object.** Solutions are *siblings under a problem*, not rebuttals threaded off an original. This dissolves the counter-proposal UI problem — no proposal owns the topic; they’re peers answering the same stated question. Legitimacy comes from alternatives being co-equal on the page.
- **A required problem statement is a content-neutral garbage filter.** You don’t just say what you want; you say why there’s a problem. Friction aimed precisely at noise: gentle on good-faith proposers, expensive for spam, and the *structure* gates rather than a moderator judging merit. This handles the vTaiwan/Madrid “drown the institution in unactionable volume” failure at the input layer, and dodges the soft-language problem — the platform enforces that slots are filled, never parses or rules on the prose.
- **Default flow: signature-gathering → batched ballot.** Proposals gather signatures; those that qualify get batched into a “ballot” voted on at once (the familiar petition-qualifies-then-batched model). Default, with other approaches allowed.
- **The qualifying threshold is a tunable UX/spam knob** (slider with presets). Too high and good proposals die ungathered; too low and the ballot bloats.
- **A solution can qualify onto an existing problem’s ballot,** not only start its own — otherwise people refile the problem to attach a variant and you get duplicate-topic sprawl.
- **“None of these” is a first-class ballot option** — otherwise you’ve quietly forced a choice and taught people the vote is rigged toward action. Also gives the problem somewhere to rest without a bad solution winning by default.
- **Problems recurse.** A problem gathers signatures to go live the same way solutions do; a problem nobody seconds never opens for solutions. This gates *tendentious premises* (“Problem: the treasurer is corrupt”) with one mechanic instead of two.
- **Decouple problem lifecycle from solution lifecycle.** A problem can persist across multiple ballot rounds (round fails quorum, or “none of these” wins) without the solutions under it being immortal. Separate state machines, linked — maps cleanly onto event-sourced state.

## 4. Proposal types are a configurable set, bound to class not mood

The menu of proposal types is configurable; presets pick a sensible subset; power users add or restrict. But bind proposal *type* to proposal *class*, not to the proposer’s mood:

- A book club gets simple-motion as the default for everything and never sees the heavy machinery.
- Changes to governance itself, treasury decisions over a threshold, or anything schism-eligible can be *configured to require* the full problem/solution structure regardless of who files.
- This extends the existing per-type temporal-mechanism idea: the *form* scales with stakes, not just the clock. Friction lands where it earns its weight and vanishes where it doesn’t.

Note: many groups worry more about ease-of-change than about spam. That’s fine — the amount of friction a group wants is itself a config axis, chosen once through governance and visible/ratified like any other decision. More honest than the platform deciding for them.

## 5. Friction-as-ratchet (needs design input beyond the founder)

The one real trap in (4): the decision to *lower* friction must itself be *higher-friction* than a routine motion. Otherwise the attack/decay path is trivial — pass an easy simple-motion that makes everything simple-motion, and you’ve dismantled the guardrails through the gap they were guarding. Lowering the friction floor is exactly the first move of a hostile faction or an impatient majority.

So “change the proposal-type defaults” should sit near governance-change thresholds (grace period, higher quorum, possibly schism-eligible) — **even when the change is toward ease.** Ratchet logic: easy to operate within your chosen friction, deliberately harder to remove friction than to add it. This keeps simple-motion an honest deliberate setting rather than a corrosive loophole.

**Flag:** this is subtle governance-design territory — the kind of parameter the whole-system-governance doc lists as needing real design with input beyond the founder. “Configurable proposal types” carries a hidden constraint (asymmetric change thresholds) and must not be implemented as a flat configurable list that’s trivially flattened.

*(Adopted: the asymmetric change thresholds are now specified in the feature list, Sections 7.5 and 7.7. The deeper parameter design still wants outside input.)*

## 6. AI as a plugin, made genuinely removable by structure

AI help for the soft, language-y problems (clustering near-duplicate solutions, flagging a problem statement that’s actually a solution in disguise, drafting a neutral problem restatement for the proposer to accept or reject) should be an *optional plugin*, for privacy reasons. The structured problem/solution objects are what make this clean: the AI gets well-shaped jobs on typed objects, never sees more than it’s handed, and is genuinely removable rather than secretly load-bearing. The structure is the precondition for the AI being optional.

## 7. Basics-as-near-term-product

For Phase 1–4 — the entire survival window — the basics *are* the product. Plugins are a Phase-5 multiplier bolted onto a base already won, not a safety net under the wire (the net is temporally unavailable in exactly the period where wrong basics kill you). So “the basics” is promoted from poorly-defined someday-detail to near-term product spec: **the minimum viable content / proposal / event / governance set per institution type.** The varied pilot surface tells you what that set is; the early pilot keeps it honest.

## 8. Build the first-party basics on the public extension API (dogfooding)

Don’t build content types and later build a plugin system for them — build the first-party basics *against the public extension API from day one*. Three payoffs: it proves the API is sufficient (if you can’t build events on it, neither can anyone), it produces the exemplars every ecosystem bootstraps from (devs build by copying a working plugin), and it means there’s no separate “now stand up the plugin system” project. Phase 5 becomes mostly *exposing the API + building more of the feature set via plugins*, which is the existing plan.

## 9. Plugin DX: elegance must be invisible to the plugin author

Elegant internals recruit hobbyists, not ecosystems (SSB is the cautionary case — elegant, beloved by a niche, never broadened). What produces an ecosystem: a user base worth a dev’s weekend, near-zero authoring friction, and the hard parts hidden behind a boundary the plugin author never crosses. The interface-composition abstraction should let someone ship a content type touching zero crypto, zero federation, zero partition keys — “a schema declaring Renderable + Geographic + Timed.” Same rule as crypto-invisible-to-users: the elegance has to be *invisible* to the plugin dev to do its job.

## 10. Plugin cold-start: the embedded dev with an itch

The first plugin authors aren’t a generic OSS movement — they’re already inside the user base: the person who is *both* a member of a civic community *and* a developer, who needs their HOA to track something the basics don’t, and builds it because they need it. Scratch-your-own-itch, vouched-in, embedded, skin in that specific game — the same locality-grounded structure as everything else in Haven. The incentive stack already fits: contribution attestations are status/credit; certification is the money-and-trust path.

## 11. The additive/interactive plugin split

A difficulty gradient runs through the plugin surface:

- **Additive plugins** introduce a new typed thing that renders through existing interfaces and touches nothing else (new content type, feed shape, homily record, commission tracker). Nearly free — the “schema plus declared interfaces” case. Phase 5 really is mostly exposing the API for these.
- **Interactive plugins** plug into a flow the platform already owns and enforces (ticketing touches payment, inventory, sybil-resistant nullifiers; a voting flow touches eligibility, the tally authority, the people-vote nullifier, schism eligibility). The steep end — they *participate* in something the core guarantees, they don’t just render.

## 12. The governing rule: plugins compose primitives, never author guarantees

A plugin can be allowed to define a *flow* but never an *invariant*. The platform’s guarantees (one-human-one-vote, one-ticket-per-person at the configured tolerance, eligibility actually meaning eligibility) are cryptographic and cannot live in plugin code — or any author, including a malicious one, could forge them. So for any proposed extension point: **if exposing it would let a plugin forge an invariant, expose the *primitive* and let the plugin orchestrate, not the mechanism.** Split each interactive surface into a protected core the platform owns and a configurable flow the plugin expresses; the design work is finding the seam.

- **Voting seam.** Invariant (NOT pluggable): per-decision nullifier anchored on the humanness `pid`, counted by the single tally authority, eligibility via shared-`pid` ZK equality. Pluggable: ballot structure, tally function (Condorcet, IRV, score, approval), nomination, quorum/window logic, presentation. The governance plugin API is essentially *“given a verified-eligible set of unforgeable ballots, return a tally and presentation.”* The plugin computes; it never counts identity. (This is where the problem/solution/ballot object from note 3 reconnects — it’s the plugin-facing flow object.)
- **Ticketing seam.** Invariant (NOT pluggable): per-event nullifier + application-layer cardinality count at the single ticketing authority (the cap-1/cap-N unification). Pluggable: the payment *adapter* interface (already the established pattern) plus the surrounding flow (transfer/resale rules, tiered/dynamic pricing, waitlist, check-in UX, bundles). The clarity move is **naming the adapter interface and the sybil-resistance core apart** — they’re collapsed under the one word “ticketing API” today.

The asymmetry — *hard for the core author, easy for the plugin author* — is the whole game, and it’s the same principle as crypto-invisible-to-users and elegance-invisible-to-devs. Three faces of one rule: the difficulty lives in the core so it doesn’t live at the edges. What makes the interactive surfaces exposable at all is the credential layer’s “compose validated primitives, don’t invent crypto” discipline — the plugin author orchestrating a voting flow is composing primitives without touching BBS, calling `verify-eligible-ballot` without knowing what a nullifier is.

## 13. E2C as a study topic (ownership-transition)

“User-owned after X years” is not a bespoke dream — it’s a named, currently-active pattern: **Exit to Community** (Nathan Schneider, 2019), implemented via **steward ownership** / **perpetual purpose trusts**, with the **Purpose Trust Ownership Network** as a current resource. Most relevant alongside the [founder-binding analysis](haven-founder-binding-analysis.md). Two specifics worth holding:

- Schneider & Mannan’s canonical E2C paper models a hypothetical social network and names **federation** (open-source the tech, let separate legal entities operate it) as one of three transition strategies. That is Haven’s architecture described as an ownership-transition mechanism — the exit may already be designed into the platform.
- If investment is ever taken: the term that protects the project isn’t the X-years promise, it’s **retaining board control and majority until the transition triggers fire** (the Stocksy/Mullvad lesson — founder-capitalized, control never ceded). A mandatory-transition / foundation-early term sheet isn’t a concession; it would *accelerate* the founder-binding the governance docs admit is weakest during bootstrap.

-----

## 14. Configurable cadence and post-length (post-type interface) — a basic, not a nice-to-have

Cadence (e.g., once-daily digest delivery) and length bounds (min/max words) are post-type config: field validation plus a delivery setting, touching no invariants. The additive-plugin / typed-schema case (note 11), genuinely free.

Pedigree makes it more than free, though. FPF’s once-a-day digest is exactly this knob, and it does real work for them — updating only once a day is what makes a flame war arduous and optimizes for thoughtfulness over volume. So this turns the single best calming mechanism of the only working analog into a per-feed setting. Slower-and-longer as a *configurable cadence* (not a hardcoded global default) suits large deliberative alliances especially — the bigger and more deliberative the body, the more a slow lane beats a fast one. Goes in the post-type interface.

**Reclassification (the rule the FPF comparison forces):** a feature present in the only genuinely comparable working product is not a nice-to-have. Mechanically this is plugin-shaped; in priority it belongs in the near-term basics tier (note 7), built first-party against the extension API (note 8).

*(Adopted: delivery cadence and post-length bounds are now feed configuration in the feature list, Section 4.3.)*

## 15. Pre-publication review feed = write-gate sub-group + intake queue (the moderation-tension reconciliation)

The ask (a Matrix-style “policy server” that reviews everything en route to a more open community/alliance feed) does **not** port as stated: community-feed content is E2E-encrypted with no plaintext chokepoint in transit, by design — a server-side reviewer in the delivery path is exactly the centralized readability the blind hub removed.

What ports, and yields the same outcome, is already a primitive: **a review sub-group as a write-gate** (the role-write enforcement pattern). The reviewer sub-group holds the write-cap for the open feed; members don’t post to the open feed directly, they post to a review queue (a confidential-read feed the reviewer sub-group can read); a reviewer re-publishes approved items into the open feed. No new mechanism — compose the publisher-sub-group pattern with an intake queue.

**Why this matters beyond the feature: it reconciles the FPF moderation tension.** FPF’s magic is humans reading every post before publication (half its staff are paid moderators, every message read pre-publication). Haven’s E2E forbids that *at the operator* — which read in positioning as “the thing that makes FPF lovely is the thing Haven’s privacy forbids.” But Haven forbids operator-readability, not *community-role* readability. A community-appointed reviewer sub-group reading its own members’ queued posts — in plaintext it is entitled to — is categorically different from an operator reading everything: it’s the community moderating itself, with the operator still blind. So this is how a community that wants FPF-style pre-moderation gets it *without breaking operator-can’t-read*. That’s the answer to the tension, and the reason this isn’t a nice-to-have either.

Design constraints to carry:

- **Disclosure.** The review queue is a confidential-read sub-group; posters knowingly hand plaintext to the reviewer set. This must be a visible, governed feed property the way history-status is (“posts here are read by reviewers before publication”).
- **Federation boundary.** It cannot be “review everything en route to the alliance feed” at an alliance-level chokepoint — alliance broadcast is the sender-key layer, with no plaintext chokepoint there either. The correct shape is each community reviewing its *own outbound* contributions before they enter the alliance stream, which is the more federation-honest model anyway (communities are the unit of alliance accountability, per the franking design). The large-alliance use case works as “every member community gatekeeps its own contributions,” not “a central alliance board reviews the firehose.”
- **Bottleneck is the failure mode.** Pre-moderation bottlenecks; FPF pays for it with paid moderators at a ~1:2 steward-to-content ratio. A volunteer reviewer sub-group in a large community is exactly the vTaiwan two-sided-usability death — posters wait, reviewers drown, the queue becomes theater. So pair the slow-cadence setting (note 14) in by default: a review feed is inherently a slow feed, and a real-time reviewed feed is a promise you can’t staff. Pre-review and once-a-day belong together.

**Reclassification:** same as note 14 — present (in human-mediated form) in the only working analog, so a high-priority near-term capability, not a far-future plugin.

*(Adopted: pre-publication review is now a named moderation model in the feature list, Section 14.2.)*

## 16. Moderation redistributes, it doesn’t vanish — and the commons is the thin spot

This is the design rationale behind several decisions below; it’s also a positioning claim, developed in the [civic-tech landscape](haven-civic-tech-landscape.md).

FPF *centralizes* moderation: one ~12-person paid team reads everything for all 250k Vermonters, funded by local ad revenue. That’s exactly why it can’t expand past its region — Ohio would need its own hired team. Haven *distributes* it: each little org moderates its own feed, labor borne locally by community-appointed roles, in proportion to that community’s own content. That model scales precisely because no central team reads the whole firehose — the structural reason a Haven small-town mesh (a set of little orgs sharing a registry alliance, which would *look* like FPF) could grow past one state where FPF won’t. So “we can’t afford FPF’s moderation” is true at the center and beside the point: Haven doesn’t moderate at the center, by design.

**The thin spot is the commons.** The registry-alliance feed — the town-wide conversation, the part that actually resembles FPF — belongs to no single community, so its moderation is nobody’s local proportional labor. It’s the highest-value, highest-conflict surface and the one with the least obvious funding. Haven’s distributed model funds the communities but not the commons. That gap, not moderation generally, is the real problem to solve.

## 17. Registry staff as a role — plus the legal-entity wrinkle

A registry that wants paid staff (commons-feed moderators, curators, support) expresses *the authority* as a role like any other — “registry staff,” with permissions/duties (e.g., moderation authority over the commons feed), filled through standard alliance governance. On the authority layer, “just another role” is correct.

The wrinkle: **employing people requires a backing legal entity.** Alliances (and registries, which are specialized alliances) are *cryptographic* entities, not legal ones; payroll, employment, and merchant-of-record are legal-world functions. This is the same split the platform already uses for instances — a cryptographic instance alliance paired with a concrete legal *operator entity*. A registry that employs staff needs the equivalent pairing: the cryptographic registry-alliance for governance/authority, and a legal entity (a “registry operator,” possibly the hub instance’s operator, possibly dedicated) as employer and merchant-of-record. The role grants authority on-platform; the legal entity handles employment off-platform. So the capturable correction to “just another role”: yes for authority, but a staffed registry implies a backing legal entity, consistent with the existing operator/alliance pattern — not free, but already-patterned.

## 18. Funding the commons via local-business credentials

The fill for the commons gap, built entirely from existing primitives:

- **“Local business” is a registry-issued credential.** Registries are already locality-keyed curated alliances — the right body to decide who counts as local, and federated per-locality so that judgment never has to scale centrally (the local-vs-non-local advertising call that wouldn’t scale at the platform level *does* scale when made at the registry level, by locals).
- **A business pays registry dues for a role** with write access to a business/offers feed. Money flows business → registry (legal entity, per note 17), processor-direct, **no platform cut, no intermediate account** (consistent with CCF-9 / the adapter pattern).
- **This funds the registry’s commons and its staff** (note 17). It is FPF’s funding model reproduced at the registry tier — local businesses paying to reach the local audience, funding local moderators — except the revenue accrues to the registry rather than to a central operator, and there can be many self-funding registries rather than one.

Design stance: the business-advertising-feed plugin can’t be *stopped* (the primitives don’t forbid it, and business↔user interaction is a genuinely important civic surface). Don’t stop it — *aim* it. Aimed at the registry, it pays for the one thing the distributed model otherwise can’t fund. Communities should be able to define roles with different dues so locally-determined local businesses can have an advertising/offers feed.

## 19. The structural floor: bulletin-board vs. attention-auction

The advertising slope has a floor, which is why it’s less slippery than it feels. The extractive bottom isn’t “advertising” — it’s *attention extraction*: algorithmic amplification, surveillance targeting, engagement auctions. Haven’s primitives don’t contain that machinery. Feeds are typed and chronological; there is no amplification engine to sell and no surveillance signal to target on. So a registry can sell a business *a place on the bulletin board*, but it structurally **cannot** sell *amplification of attention* — the mechanism doesn’t exist to be sold.

The line between FPF-style local advertising and Meta-style extraction is therefore **a capability that wasn’t built, not a policy that has to be held.** Advertising-as-bulletin-board is supportable; advertising-as-attention-auction would require adding machinery visible to everyone. That’s the good kind of guardrail — structural, not willpower. This distinction is now written into the architecture document’s revenue model, because it’s the exact line a future operator under revenue pressure would be tempted to blur, and naming it is what makes the blur visible.

## 20. Revenue: “no cut” is not a vow of poverty

“No cut of on-platform revenue” conflates two things; separating them dissolves part of the survival-vs-ideology worry:

- **No transaction skim / no attention auction** — structural, mission-critical, keep. No platform-controlled intermediate account; no amplification engine. Architecture-enforced, not policy.
- **Charge robustly for services** — dues tiers, Foundation certification, credential manufacture, Strutco professional services and support. None of these is a *cut* of anyone’s commerce; they’re fees for things actually provided.

You can charge strongly for services while never building the skim. So the real risk is not the ideology — it’s **opex outrunning thin service revenue before scale.** The Stocksy lesson: capped-revenue-as-discipline survives *only when opex is capped to match*. The thing to watch is therefore not ideological purity but whether opex stays small enough that slow civic-modest service revenue clears it — which throws the weight onto the bets already named (AI-cheap maintenance, slow growth, don’t hire ahead of revenue, reinvest AI savings into the cryptographer review). The danger is letting opex creep to startup-normal while revenue stays civic-modest.

The survival-vs-ideology pressure lives specifically in the bootstrap window, where the founder-binding hasn’t engaged yet and the founder is the weak point. The architecture is the commitment device against building the skim under duress; note 19’s bulletin-board/attention-auction line is the specific distinction that device has to protect.

## 21. Vote disclosure scope is governance config, not a security parameter (added 2026-06-12)

A vote’s *disclosure scope* — who can see how individuals voted, and whether the per-vote recount worksheet (the nullifier/pseudonym→count multiset that lets members verify a tally) is members-only or public — is a governance choice the community makes, not a setting the platform imposes. Real institutions deliberately span the whole range for good reasons: roll-call votes exist *because* accountability sometimes requires attributability (a party chapter endorsing a candidate may want “we voted 60–40, and here is who stood up” as public speech); secret ballots exist *because* protection sometimes requires unlinkability. The platform’s job is to make each community’s choice enforceable and visible, not to choose.

This slots into the voting seam (note 12’s invariant/pluggable split) as a new config axis alongside quorum and “none of these,” with hard requirements that are *not* pluggable:

- **Default is members-only.** Recount worksheets publish to members by default; public attribution is an explicit opt-in per vote (or per proposal class, bound to class not mood, per note 4).
- **Fixed before ballots are cast, never widened retroactively.** A vote’s disclosure scope is set and visible before voting opens. No-retroactive-widening is a *cryptographic* property, not a policy flag — what was published members-only must be out of reach to widen later, or the configurability is a surveillance trap with extra steps.
- **Loosening future-vote disclosure is itself ratcheted** (note 5): it should sit near governance-change thresholds, deliberately harder than tightening, for the same reason friction-removal is.

The honest cost, for the residuals accounting rather than this note: each public vote is a high-quality correlation anchor (a timestamped membership-participation snapshot), and a community that gathers publicly, attests attendance, and votes publicly has spent most of its cross-scope unlinkability — which may be exactly what a political chapter wants and exactly wrong for a recovery group. The configuration UX should eventually surface a community’s *cumulative* public surface, not just per-vote toggles in isolation. Anchor worth stating plainly: members-only scope-local pseudonym worksheets are already strictly more private than existing democratic practice, where voter participation is public by legal name and verifiable-election bulletin boards are fully public.

*(Sourced from the architecture-critique recount analysis; the cryptographic half — that publishing worksheets adds nothing the unlinkability threat model didn’t already grant — is recorded in the open-questions ledger under CR-B4.)*

## The three strategic commitments

Falling out of the whole study, in service of one thing — that for Phase 1–4 the basics *are* the product:

1. **A varied pilot surface** — diverse institution types in front of the design, so coherence ends up being coherence-around-civic-life, not coherence-around-one-imagined-parish (the SSB niche-lock-in inoculation).
1. **An early pilot** — start the conversation before the thing feels ready; its job at this stage is to keep the single vision honest against a real institution’s actual mess, not to deploy.
1. **Design for pluggability early** — precisely: *build the first-party basics against the public extension API from day one* (dogfooding), not *ship the plugin ecosystem early*. The ecosystem stays Phase 5; what moves early is dogfooding the API so the basics double as the seed corn.