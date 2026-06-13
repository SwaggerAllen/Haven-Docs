# The graveyard, and where Haven stands in it

*This document merges two rounds of the civic-tech study: the failure-mode survey (the graveyard) and the comparative positioning that followed it. Part I maps how projects in this space die; Part II places Haven among the survivors and the incumbents. Confidence is flagged throughout — well-documented outcomes are distinguished from single cases and anecdote. Companions: the [founder-binding analysis](haven-founder-binding-analysis.md) maps the adjacent graveyard — how civic-tech-adjacent projects go *bad* (governance capture, extractive pivots) rather than fail to *matter* (irrelevance, unsustained need, dependency). The two are complementary; the cases barely overlap. The [design notes](haven-civic-tech-design-notes.md) hold the decisions this study produced.*

-----

# Part I: How civic tech dies

## Cases surveyed

- **Civic Tech Field Guide “Graveyard”** (Sifry & Stempeck) — curated catalogue of ~70 documented civic-tech failures with recurring patterns. The spine of the survey.
- **Diaspora** — decentralized social, record crowdfunding, never dented Facebook.
- **Decide Madrid / Consul** — municipal digital-participation platform; large success then atrophy.
- **vTaiwan / Pol.is** — the most-cited digital-democracy “success,” which stalled.
- **Platform co-ops** — Stocksy (survivor), Ampled (closed 2023), Resonate (perpetual bootstrap), plus movement-level academic diagnosis.
- **Secure Scuttlebutt (SSB)** — the architecturally closest cousin to Haven (keypair identity, web-of-trust, append-only logs, in-person QR onboarding in the Tremola variant).

-----

## The recurring death patterns

Sorted roughly from commercial/adoption deaths to governance/structural deaths. Confidence tags: **[strong]** = well-documented across multiple sources or curated research; **[moderate]** = documented but fewer sources or older; **[single case]** = one sharp instance.

1. **Early funding ≠ sustained engagement.** “If you build it, they don’t come.” Nine voter-info networks raised $20M+ between them and all died; Brigade burned $9.3M and limped. **[strong]**
1. **Unfelt or infrequent need.** Some civic needs aren’t felt intensely or often enough to sustain a dedicated product (generic voter information, deliberation-with-strangers). **[strong]**
1. **Death by portfolio deprioritization.** The owner refocuses on more promising projects and the tool withers (Pledgebank, OpenCongress). **[strong]**
1. **Collective-action friction.** Coordinating through the specialized tool is more work than a hashtag, petition, or GoFundMe; the general-purpose tool wins. **[strong]**
1. **Hyperlocal scale economics.** Pre-mobile, too expensive to scale (EveryBlock, Outside.in); or swamped by Facebook/Twitter. Survivors are the well-moderated, geographically bounded ones (Front Porch Forum at ~60% of Vermont households; SeeClickFix). **[strong]**
1. **The human element.** Builders skip user research, neglect relationships with institutional gatekeepers, and assume buy-in is easier than it is (Ostling study). **[moderate]**
1. **Self-hosting / setup friction.** Purist decentralization’s onboarding tax; “mom and pop can’t run a server” (Diaspora). **[strong]**
1. **Network-effect competition.** A better-resourced entrant with the same pitch drains the would-be base (Diaspora vs Google+). **[strong]**
1. **Founder dependency / tragedy / “hand it to the community.”** The handoff to a volunteer community is usually a slow fade (Diaspora; Mastodon’s nine-year single-founder span before a crisis handoff). **[strong]**
1. **Patron dependency.** Tied to one administration or funder; when the patron exits, the platform atrophies (Decide Madrid losing its mayor in 2019; vTaiwan losing direct government support). **[strong]**
1. **Non-binding consequence.** Output is advisory to a sovereign that can ignore it; once that’s clear, participation atrophies (vTaiwan never mandated; Madrid stopped responding to proposals). **[strong, two-case]**
1. **Two-sided usability failure.** Easy for the citizen, unusable for the institution; the institution drowns in unactionable volume and disengages, killing the loop from its end (Noveck’s diagnosis; Madrid; vTaiwan). **[strong]**
1. **UX complexity.** Even motivated civic users abandon clunky multi-tool flows (vTaiwan was four tools cobbled together). **[strong]**
1. **The financing bind.** Co-ops can’t take VC (no equity upside, can’t cede control) and banks won’t fund risky startups; anything that *needs* scale capital starves (movement-level academic finding). **[strong]**
1. **Replication is not enough.** Copying the incumbent’s product under nicer ownership isn’t a reason to exist (academic finding; Stocksy contrast). **[strong]**
1. **Ideology over value.** “Co-op,” “privacy,” “decentralized” are not themselves value propositions; thin member value can’t sustain (Ampled at $6.78/fan). **[strong]**
1. **Governance-on-paper breaks under stress.** Democratic bylaws, top-down death anyway — Ampled closed without the member vote its own bylaws required, founder unresponsive. **[single case, but sharp]**
1. **Volunteer / burnout dependency.** No sustainable paid labor → limp or shutdown (Resonate’s permanent bootstrap; Ampled’s stated cause). **[strong]**
1. **Purist-decentralization tax.** Append-only single-key logs make identity recovery, multi-device, deletion, bounded storage, and protocol evolution hard; “no servers” smuggles servers back as informal Pubs; immutability fights evolvability (so the forward path became a successor protocol). All SSB. **[strong]**
1. **Niche lock-in.** The failure isn’t incoherence — it’s *coherence around a too-specific user*. SSB was internally consistent and perfect for off-grid sailboat hackers, and bounced off everyone else. **[strong]**

### The survivors, and why they lived

- **Front Porch Forum / SeeClickFix** — geographically bounded, well-moderated, mundane-utility-done-well. Won on exactly the axis the dead lost on. (FPF gets a full treatment in Part II.)
- **Stocksy** — founder-capitalized (sidestepped the financing bind rather than solving it), *deliberately capped growth*, real member economics (50–75% royalties + dividends), differentiated on curation rather than replicating or out-scaling the incumbent, self-sustaining from operations. (Full treatment in Part II.)
- **Pol.is / Consul as tools** — survived by decoupling from any single deployment and becoming reusable infrastructure stewarded by an independent body (Computational Democracy Project; Consul Democracy Foundation). The tool outlived the flagship by escaping the patron.
- **PolicyKit as a research-stewarded primitive** — the same escape pattern in the academic register: a framework for programmable governance in online communities (Zhang et al.), stewarded by the Metagovernance Project rather than tied to one community’s deployment. It matters to Haven beyond the survival lesson: PolicyKit is the conceptual root of the governance-over-encryption lineage Haven sits in — PolicyKit (plaintext governance engine) → MlsGov (PolicyKit-style governance demonstrated over MLS, IEEE S&P 2024) → Haven (that approach plus federation, communities-as-entities, schism, and a blind-operator model). Haven should cite this lineage rather than present its governance-over-MLS as sui generis; a knowledgeable reader places it immediately.

-----

## Where Haven sits

Three buckets: deaths the design structurally **dodges**, deaths it **bets against** (leans right but no guarantee), and deaths that remain **live risks**. The bucketing itself is a hypothesis to pressure-test, not a verdict.

### A. Deaths Haven structurally dodges

The design makes these failure modes hard to *occur*, not merely unlikely.

- **Non-binding consequence (11) and unfelt need / build-it-they-won’t-come (1, 2).** Haven’s central wager: attach to institutions that *already* meet, already have felt needs and real internal consequence (parish, HOA, co-op), rather than manufacturing a new civic behavior. The vote *is* the decision, not a recommendation to an external sovereign — consequence is endogenous. Most of the graveyard tried to manufacture civic behavior; Haven rides existing intensity. *Caveat: holds only where the platform’s governance maps to where the community’s authority actually sits, not as a parallel toy a real pastor/board ignores.*
- **Patron dependency (10).** Not government-owned; no single political patron whose exit kills it. *(Trades for founder/operator dependency — bucket C.)*
- **Financing bind / scale-or-die (14).** PBC on community-affordable dues + credential revenue + foundation; the capped revenue ceiling is treated as a feature, and federation means per-instance sustainability without global scale. Structurally Stocksy (bounded, self-sustaining), not Ampled (scale-chasing on grants). The strongest, most under-discussed positioning asset.
- **Self-hosting friction (7).** Individuals don’t self-host; instances are operated entities. *(Trades for the in-person QR gate — different, deliberate friction; bucket C.)*
- **Identity loss / no recovery (part of 19).** SSB loses you permanently on key loss; Haven built the layer SSB refused — social recovery, device handoff, signed key succession, portability across instances. SSB is the cautionary tale that vindicates this work.
- **Smuggled-back servers (part of 19).** SSB pretended it needed no servers, then needed Pubs informally; Haven names the instance/blind hub as first-class infrastructure from the start — “Pubs done deliberately,” privacy designed in rather than bolted on.

### B. Deaths Haven bets against

Design leans the right way, but it’s a live wager, not a structural guarantee.

- **UX complexity (13).** Haven is far more complex than vTaiwan’s four tools. The make-or-break per the founder. Mitigation: templates / sliders / presets / progressive disclosure, with three surfaces that must be nailed — configuration, proposal/voting, events/calendar. Complexity killing motivated civic users is *demonstrated*, not hypothetical.
- **Two-sided usability (12).** Haven’s “institution side” is the community’s own governance roles, not an external bureaucracy — better odds than Madrid/vTaiwan. But if proposing is easy and the governance side is unusable for the people who actually run the community, the same atrophy applies. Mitigations in the design notes (problem/solution framing, signature-gather→ballot, filtered views, structured input gating noise).
- **Replication-not-enough (15) and ideology-over-value (16).** Haven can’t win as “Discord but member-owned.” UX earns the look; the architecture (the can’t-follow moat) plus serving institutional shapes incumbents flatten is the switching justification. Felt value must be real institutional utility (events/governance/credentials working well), not just “nicer / more private.”
- **Niche lock-in (20).** The solo-coherent-vision risk: the coherence may secretly be coherence-around-one-imagined-congregation, fitting the pilot parish and bouncing off the HOA, the co-op, the collective. Hedged by a varied pilot surface and getting diverse real communities in front of the design early. *This is the strategic reason the study produced “varied pilot surface + early pilot” as commitments.*
- **The plugin safety-net fallacy.** “If we get the basics wrong, plugins let others fix them” is true in Phase 5 and false in Phase 1–4 — the entire survival window. For that window the basics *are* the product. Bet: ruthlessly good first-party basics per institution type, defined as near-term product spec rather than backfilled detail.

### C. Live risks that remain

Failure modes the design does not neutralize.

- **Founder dependency + bootstrap vulnerability (9).** The org/governance docs admit it: structural protections are on-paper until Foundation Alliance composition is meaningful. Ampled (17) is the sharp cautionary instance — democratic governance broke under financial duress, top-down closure despite bylaws. This is the direct cross-reference into the [founder-binding analysis](haven-founder-binding-analysis.md), and the precise stress case the math-enabled-democracy bet exists to survive.
- **Operator dependency.** Substitutes for the patron dependency Haven dodges. Hostile/disappeared operators are designed-for (migration, allied replicas), but operator quality and availability remain a real dependency.
- **Plugin double-adoption.** Ecosystem requires devs requires user base — adoption inside adoption. Mitigations (build basics as plugins / dogfood the API; embedded-dev-with-an-itch rather than a generic OSS movement; contribution attestations + certification as incentives) are promising but the cold-start is a genuine risk.
- **In-person QR gate as growth limiter.** Solves trust and sybil resistance elegantly — independently arrived at by SSB’s Tremola variant (two phones scan a QR in person) — but does not by itself solve growth, and rules out the viral fallback the dead projects at least had. Growth is intrinsically local/physical. Held from the start of the study as both cornerstone and constraint.
- **Portfolio deprioritization (3), founder-attention variant.** A solo founder spread across Strutco, Haven, Siege Engine, and the public-facing writing carries the same focus risk that killed Pledgebank and OpenCongress — not as a governance failure but as an attention one. A structural observation rather than a criticism; worth naming because it’s a documented pattern and the current situation matches its shape.

-----

## The through-line

Most of the graveyard died trying to *manufacture* civic behavior or chasing *scale* it couldn’t fund. Haven’s design is a sustained bet on the opposite of both: attach to pre-existing institutional intensity, and stay deliberately small-and-sustainable per instance. That single posture is what lets it dodge bucket A. What it cannot design away (bucket C) clusters around the bootstrap period and the in-person gate — the places where the project depends on the founder, the operator, and physical-local growth. And the whole thing still rests on bucket B: winning on UX while making the architecture the reason to switch. The survivors (Front Porch Forum, Stocksy) won on exactly that combination — bounded scope, real utility, a reason to exist beyond ideology — which is what Part II takes up.

-----

# Part II: Where Haven stands among the living

Positioning uses a deliberately narrow lens: Haven’s real category is civic/community governance, not decentralized social in general, so the famous decentralized-social names appear in Part I as cautionary cases rather than here as peers. Three kinds of comparator, each answering a different question:

1. **Survivors by posture** — *does the model work, and what does the architecture add?* → Front Porch Forum.
1. **Survivors by structure/economics** — *can capped, member-owned, capture-resistant economics survive, and does multi-stakeholder governance actually operate?* → Stocksy United. (Bridges into the founder-binding analysis.)
1. **Survivor by governance function** — *is configurable collaborative self-governance buildable and sustainable, and how do you finance it without ceding control?* → Loomio. (The closest functional cousin; also a financing precedent.)
1. **Incumbents by displacement** — *what is a pilot community actually choosing against, and why would it switch?* → the institutional SaaS (church management, HOA platforms) and the general-purpose tools.

-----

## Front Porch Forum (survivor, product/posture mirror)

**One line:** FPF proves the demand and the posture are real; Haven’s architecture is what it would take to make FPF’s magic *portable, operator-independent, and institution-shaped*.

**The facts.** ~235–250k members in a state of ~270k households (near half of Vermont adults); ~97% positive in academic survey (69.5% “very valuable,” 27.4% “somewhat”); founded 2000/incorporated 2006, ~250 town-specific forums, ~30 staff including ~12 moderators. A Vermont public benefit corporation, family-owned (the Wood-Lewises), deliberately refusing to expand beyond its region.

**What it does right — and Haven independently arrived at almost all of it:** enforced locality as the unit (real name + street address; eligibility is living/working in the area); heavy human moderation as the core product (half the staff are moderators; every post read pre-publication; a friendly steward note rather than silent deletion); anti-engagement by design (once-a-day digest, which makes a flame war arduous); mission-aligned economics (no VC; PBC not meant to earn beyond sustaining itself; non-surveillant local advertising — Vermont small businesses, nonprofits, agencies); and utility-first value (events, a business directory, and repeated disaster-response during Vermont floods).

**The core of the comparison — coupled equilibria.** FPF and Haven are two internally-coherent equilibria for the same problem, local trust, and their choices are *coupled, not independent*:

- **FPF spends in the room:** weak, self-asserted door (an address on the honor system) + heavy continuous human moderation + real names + operator-reads-everything. Trust is manufactured at the content layer, daily, by people.
- **Haven spends at the door:** strong cryptographic gate (in-person QR/NFC vouching) + light reactive moderation + pseudonymity + operator-blind. Trust is manufactured once, at entry.

The coupling is the insight: **weak gate requires strong moderation; strong gate enables weak moderation.** FPF can afford a soft door because it reads everything; Haven *must* have a hard door because it reads nothing. Each is consistent. FPF is the proof the soft-door/heavy-moderation equilibrium genuinely works — at the cost of the three things it can’t escape.

**FPF’s three ceilings = what Haven’s architecture is *for*:**

1. **FPF *is* the trusted operator, and that doesn’t transfer.** Its moderation and norms rest on the founders’ two-decade local reputation — beautiful and non-portable, which is exactly why it can’t be copy-pasted to Ohio and why it’s founder-dependent. Haven puts *structure* where FPF has a *person*, so many communities can have FPF-grade trust without a Wood-Lewis each.
1. **One operator reads everything.** FPF’s quality comes from humans reading every message — which means FPF (and any subpoena, acquirer, or future owner) can. Haven’s E2E + per-community pseudonymity is the structural answer to “what happens when the trusted operator isn’t.”
1. **It’s one civic commons, not many self-governing institutions.** FPF is neighbors-in-a-town; it has no concept of the parish that governs itself, the HOA with a binding budget vote, the co-op with members and credentials. That first-class community-as-institution is Haven’s actual category, and the thing FPF structurally isn’t.

**The privacy mirror.** FPF buys accountability by *spending* privacy (real names, legible to all); Haven tries to buy accountability while *preserving* privacy (pseudonyms + vouching-with-decay + franking). FPF is evidence the cheap version works fine for a neighborhood; Haven’s harder bet is that *institutions* need the pseudonymity FPF can’t offer (support groups, recovery ministries, tenant unions, congregations in hostile jurisdictions).

**Scope discipline (important):** don’t pitch against FPF on neighborhood-forum turf, where FPF’s simpler model is genuinely *better*. Haven’s machinery only earns its cost where pseudonymity, institution-shape, operator-independence, or founder-survival are load-bearing.

**Proof-plus-risk (the tie to founder-binding):** FPF is simultaneously Haven’s best proof of demand (the model works, people love it, the economics sustain) *and* a live illustration of the risk Haven is built to address — it has **not** solved succession (family PBC, no member governance, no exit-to-community structure). 250k Vermonters’ real names, addresses, and readable history under a single operator is benign today and a honeypot the day ownership changes. FPF answers “can a calm, local, mission-driven civic platform work?” — yes. Haven asks the next question FPF hasn’t: “what makes it survive its founders, scale past one state, and not require trusting one operator with everything?”

**Two tensions already turned into design** (see design notes 15 and 16–20): the **review sub-group** lets a Haven community buy back FPF’s warm pre-publication moderation *within its own trusted role, operator still blind*; and the **registry-funded commons via local-business credentials** reproduces FPF’s local-advertising funding model at the registry tier without a platform cut.

-----

## Stocksy United (survivor, structure/economics mirror)

**One line:** Stocksy proves that capped, member-owned, capture-resistant economics survive 13+ years in production *and* that multi-stakeholder equal-vote governance operates — but it does both under a **purpose-homogeneity Haven deliberately forgoes**, which is exactly where Haven’s residual risk lives.

**The structure.** A multi-stakeholder platform co-op under British Columbia’s Cooperative Association Act. Three classes — A (advisors/board, up to five, founders + outside advisors), B (staff), C (contributing artists) — all with *equal votes*, 1 member = 1 vote. The ownership share costs **$1 and never appreciates**; no annual dues; patronage splits 5%/5%/90% (artist share by contribution to sales). Founder-capitalized with ~$1M from two co-founders, paid off within four years while revenue passed $10M; ~$24.7M paid to ~1,000 artists over 2013–2018.

**The financing move, stated precisely:** the co-op structure *itself* foreclosed VC — no equity (share value doesn’t rise), and co-op principles prevent outside financial control — so investors had nothing to chase. Not a choice made *against* investors; a property that made them *irrelevant*.

**The mapping to Haven:**

1. **Multi-class equal-vote governance is the people-vote principle, proven in production.** Stocksy has exactly Haven’s structural problem — multiple stakeholder types with genuinely different interests sharing authority without one class capturing it — and has run it for over a decade via elections, resolutions, and AGMs without deadlock or capture. When the whole-system-governance open questions feel untested, this is the case that says the *shape* works.
1. **The $1 non-appreciating share is the capture-resistance move Haven makes differently.** Stocksy makes capture impossible by making ownership *worthless to accumulate*; Haven makes it hard by *distributing authority cryptographically*. Both attack the same target — “is there anything here an acquirer could buy?” — from opposite sides. For founder-binding purposes, Stocksy is the cleanest demonstration the answer can be structurally *no*.
1. **Reserved outside-advisor board seats are a small, partly-stealable idea** — deliberately seating outsiders to keep the board from getting inward-looking. But it does *not* port cleanly: Haven’s board *is* the Foundation Alliance representatives, a body answering to an electorate that could reach six figures, so a handful of appointed seats are rounding error on authority. How outside voice reaches a board that *is* the representative electorate remains an open governance question.
1. **The divergence that carries the most weight — heterogeneity of purpose.** Stocksy’s members share a single commercial telos (sell stock, split proceeds); they disagree about curation and policy, not about *what the co-op is for*. That shared purpose is what makes equal-vote governance tractable. Haven’s communities have *radically heterogeneous* purposes, the platform explicitly supports *incompatible* communities coexisting, and schism is a first-class exit. So Stocksy proves multi-stakeholder governance works *when the stakeholders want the same thing*; Haven bets it works when they emphatically don’t. **The thing that makes Haven’s governance bet harder than Stocksy’s isn’t the cryptography — it’s the heterogeneity of purpose.** Which is precisely why schism and federation are load-bearing rather than ornamental: they absorb the divergence Stocksy never has to.

**Net:** de-risks Haven’s *economic* model substantially (capped, member-owned, capture-resistant, VC-foreclosed — this can last), de-risks the *governance mechanism* partially (multi-stakeholder equal-vote operates), and throws the real residual into relief: not “can multi-stakeholder governance work” (yes) but “can it work across *incompatible* purposes” (unproven; the reason schism/federation matter).

-----

## Loomio (survivor; the closest functional cousin + a financing precedent)

**One line:** Loomio is the closest functional cousin to Haven’s governance layer in the entire study — configurable collaborative decision-making, built by a worker co-op that dogfoods it — and its 2015 ethical-capital raise is the most concrete real-world precedent for taking capital without ceding control. It de-risks two Haven bets (governance-as-config works in production; the co-op financing bind has a known instrument-level answer) while making vivid exactly the slice Haven adds underneath.

**What it is.** Open-source collaborative decision-making, born 2011 out of Occupy Wellington plus the Enspiral network (founders Rich Bartlett, Jon Lemmon, Ben Knight), built to translate general-assembly direct democracy into an online tool — “decision-making without meetings.” Co-op formed 2012, international crowdfunding 2013, Loomio 1.0 in 2014, still operating ~14 years on. SaaS model: free (donation-supported) plus paid premium for business/government/nonprofit, and low one-time payments for community/volunteer groups. Has served thousands of groups.

**The product — the functional mirror.** Translates direct-democracy hand-signals into a pie chart of positions plus a comment thread. Proposal/decision types span the governance-model spectrum: consensus, consent, majority, advice process, “safe to try / check for objections,” score and ranked-choice voting, time polls — all time-bound with reminders, with subgroups under privacy controls. Designed for *synthesis and solution-finding rather than majority rule* (divergent then convergent thinking). Crucially it is **governance-model-agnostic via editable templates** that groups customize and encode so they are repeatable for new members — and Loomio explicitly tells groups to *reference it in their constitution or bylaws to make decisions binding*. That is Haven’s configurable-proposal-types / governance-as-config design note, plus the binding-consequence answer to the vTaiwan failure, both shipping since 2012. The “advice process” and “safe-to-try/objections” patterns are governance primitives Haven’s configurable types could learn from or support.

**The structure — a Stocksy-class mirror, plus dogfooding.** A worker cooperative owned by the people who build it; worker-members hold *non-financial governance shares* (control decoupled from financial return, like Stocksy’s $1 share). It is part of Enspiral, a ~12-company non-hierarchical network of sibling social enterprises, and it open-sources its own processes, structures, and policies in a public Co-Op Handbook. It uses Loomio to govern itself — the same dogfooding pattern Haven adopted for its extension API. Stated aim: become a *multi-stakeholder co-op with user representation on the board* — the same direction as Stocksy’s multi-class model and Haven’s community representation.

**The financing precedent (directly relevant to founder-binding and funding).** In 2015 Loomio raised US$450,000 of *ethical capital* from impact investors (including Sopoong Ventures, a Seoul social-venture fund) using **Redeemable Preference Shares (RPS)**:

- *Redeemable* = the shares can only be bought back by the cooperative, never traded externally; the company redeems them with an agreed return after an agreed period, *provided it produces sufficient surplus*.
- A separate class from worker-member shares, carrying **no decision-making control** — governance stays with the workers, and the constitution keeps the social mission first.

So Loomio threaded the exact needle Part I flagged as the co-op killer (the financing bind — can’t take VC, banks won’t lend) with an instrument that compensates investors *without* ceding control or mission. This is the concrete, decade-old answer to the founder’s early musing (“I might take investment under exactly the right terms… nobody goes in for that on a financial basis”): there *are* values-aligned impact investors who do capped-return, non-controlling, mission-first deals — not VC, and not a lone wealthy patron, but a documented, repeatable instrument. Nathan Schneider — who later coined Exit to Community — wrote up the Loomio raise as part of this movement, so this single case links the financing-bind failure mode → the funding question → E2C/steward-ownership → a working mechanism.

**Where Loomio stops — the slice Haven adds.** It is a governance-only module on conventional infrastructure: hosted and operator-readable (no E2E or operator-blindness), conventional accounts (no membership-integrity layer — no sybil resistance, in-person vouching, or credentials), no federation, and none of the events/payments/directory institutional substrate. Its consensus/synthesis orientation reflects small-group activist roots; co-founder Bartlett’s later work turned toward small-group organizing (“Microsolidarity”), a reminder that scaling online deliberation to large bodies is genuinely hard. And like Stocksy it is a *bounded survivor*, not a breakout — it deliberately rejected the raise-big / achieve-ubiquity / sell path and stayed a durable niche tool.

**Net:** Loomio is the single best proof that Haven’s governance-as-configurable-deliberation is buildable and that people will govern real organizations with it, and the best real-world template for financing a mission-locked platform without ceding control. It validates the governance design and the financing path at once, while making vivid that Haven’s differentiating bet is everything *underneath* the decision layer — membership integrity, operator-blind privacy, federation. In one framing: **Haven is Loomio’s governance ambition on a membership-integrity-and-privacy substrate Loomio never built, financeable by a structure Loomio already proved.**

-----

## The institutional incumbents

**The axis these illuminate is the most practical one:** the survivors answer “can a civic platform work”; the incumbents answer “what is a pilot community *actually choosing between*?” The realistic alternative to Haven is not Facebook or a dead co-op — it’s the institutional SaaS the community already pays for, or the general-purpose tool that’s *good enough*. This is the graveyard’s harshest question made concrete: **“why use yours when what we already have works?”** UX parity on the basics is therefore table stakes, not a differentiator; the architecture is the switching justification.

### Church management software (the pilot vertical)

**The finding:** ChMS is a back-office for staff, not a body politic for members. It is **one-way by architectural necessity**, and Haven is two-way-and-self-governing for the *same* structural reason ChMS can’t be — operator readability. The incumbents can’t follow without abandoning the data model their product and revenue rest on. This is the strongest “go where they can’t follow” instance the comparative study produced.

**No governance primitives exist in the category.** Every product describes itself in administrator terms — manage members, track attendance, process donations, communicate *to* the congregation. Planning Center says it outright: the core purpose of its apps is to serve church staff and volunteers, with the member app a later add-on for congregants to connect and consume. What a member *does* is give, register, join (leader-led) groups, update their own profile, watch sermons, pre-check kids, browse a directory. No proposals, no votes, no deliberation, no rule-making, no schism. The member is a managed record and a consumer, not a citizen who governs. Haven’s premise — community as a first-class self-governing body — is a different *category*, not a better instance of this one.

**One-way communication is architectural, not a missing feature.** ChMS is broadcast-only (staff → members) and doesn’t host member-to-member speech. The reason is forced by the data-custody model: because the vendor and staff can read everything, any member speech they host is speech *they are liable for and must moderate* — which would require FPF-scale human moderation they have no staff for and no business model to fund. So they don’t offer it. The architecture that makes them liable (readable, centrally-custodied) is the architecture that makes them mute. They can’t add two-way without taking on moderation liability they can’t staff, or dumping it on the church office (the FPF labor problem the office also doesn’t want). The reason they’re one-way is the reason they’ll *stay* one-way.

**Haven is two-way-self-governing by the same property, inverted.** ChMS: operator can read → liable → must moderate → can’t staff it → no member speech. Haven: operator *can’t* read → can’t be the moderator → moderation has to live in the community, because there’s nowhere else it can live. Same property (who can read content), opposite outcome. “We let communities govern themselves” isn’t a generous policy; it’s the only place moderation *can* sit when the operator is blind. And it isn’t copyable: an incumbent can’t match it without abandoning the readable data model its product and revenue depend on. This is stronger than the FPF “can’t follow” instance — FPF *chose* its model and could in principle change; ChMS is *trapped* by liability.

**“Self-moderation” is the whole substrate — which is why it’s a different application.** It’s not “members can post” plus a chat box. It’s per-community roles and write-gates (members post; community-appointed roles moderate), the optional review sub-group for FPF-style pre-publication warmth with the operator still blind, multi-level rules so a community’s own norms govern its own feed, franking so reports carry proof without surveillance, and distributed-moderation economics (each community bears its own small proportional load; no central team). A ChMS bolting on “members can post” has one chat box and an unsolved moderation problem; Haven has the governance-and-moderation substrate the posting feature plugs into. *That* is the whole other application.

**Keep two-way optional, or it reads as a handed-over headache.** Two-way civic communication is genuinely harder and riskier than one-way, and many church offices actively *don’t want* the moderation burden — that caution isn’t only architectural. So two-way is configured, not imposed: a community that wants the quiet one-way bulletin gets exactly that (announcements feed, role-write, no member posting); a community that wants to be a living two-way commons gets that, with the tools to survive it. Haven isn’t forcing two-way on the parish that wants a newsletter; it’s making the parish that wants to be a *community* possible.

**The basics Haven must match** (the table-stakes surface): member directory, events/calendar, giving with end-of-year tax statements, volunteer scheduling, broadcast email/SMS, attendance — and **secure child check-in** (QR labels, check-in/out), a marquee, safety-critical flow that maps onto Haven’s guardian-role + events + attendance but must be built as a *polished flow*, not left as raw primitives.

**Data custody — Haven’s real edge.** ChMS is centralized custody by definition: Planning Center’s own security guidance lists what the admin (and vendor) hold — personal info, financial data, child location, attendance, prayer requests, medical notes — all on the vendor’s servers, readable by admin and vendor, with breach/scam history. Export is universal CSV, so “leave with your data” is *not* a Haven differentiator. Haven’s edge is the thing none of them can offer: the operator can’t read it in the first place, and the data isn’t sitting in a vendor database to be breached, subpoenaed, or sold.

**Giving fees — a double-edged contrast.** The category monetizes giving via transaction fees (~3% + $0.30–0.45 per gift; ~$15k/year on $500k of donations, often with an option to push the fee onto the donor). Haven’s no-cut / no-intermediate-account model structurally refuses this — churches keep more (clean differentiator) — but it is also exactly why Haven can’t fund itself the way the entire incumbent category does (loops to design note 20: opex discipline, not ideology, is the real risk).

**Planning Center is an in-vertical founder-binding proof point.** Bootstrapped, never raised VC, sustainable growth since the 1990s, and in 2024 the founder created the Ministry Centered Foundation to make the company “not for sale” — converting to a nonprofit under the foundation when he steps away or dies. Two consequences: (1) economic validation — the no-VC, mission-locked, sustainable model demonstrably works *in churches specifically*, alongside Stocksy; (2) a positioning subtlety — **the dominant incumbent has already claimed the “we’ll never sell out / no VC / mission-first” high ground, so against Planning Center that story is parity, not edge.** Haven’s edge over PC is purely architectural (operator-blind data-custody + self-governance), not mission or price. PC’s binding is the softer benevolent-founder-plus-future-nonprofit shape (closer to Proton or Mastodon’s eventual handoff) versus Haven’s distributed-cryptographic governance — its own comparison in the [founder-binding analysis](haven-founder-binding-analysis.md)’s terms.

**The open-source / self-hosted niche.** Rock RMS (free, technical) and ChurchCMS (free, open-source, self-hosted, “no vendor lock-in”) are the closest to “you own it” — but they carry the self-hosting-friction ceiling that capped Diaspora and SSB. Haven sidesteps it by *not* asking churches to run servers (instances are operated entities), occupying a middle: more sovereign than the SaaS incumbents (operator-blind, self-governing) without the self-host burden of the open-source ones.

**The field, as of early 2026:** Planning Center (the industry “gold standard”: modular People/Services/Groups/Giving/Check-Ins/Registrations/Publishing; pay per module; ~$0 to ~$1,466/mo; robust for larger churches, can get expensive). Tithely Church Management (formerly Breeze, acquired and folded into Tithely’s all-in-one platform; flat-rate; the simplicity/value option for small-to-mid churches). Pushpay (large churches). Rock RMS (free, open-source, requires technical setup). Church Community Builder (deep customization). Newer engagement/app builders (Pews, Buildify — branded apps, “code ownership” pitch).

### HOA / community association software

**The finding is a misfit.** The manager-vs-member axis is confirmed *hardest of any vertical* here, communication is **surveilled rather than silent**, and governance is administered compliance — but **HOAs are a structural misfit for Haven’s model and should not be an early target**, despite the heavy surface overlap. The misfit is the headline, and it’s more useful than another clean win because it says where *not* to aim.

**Manager-vs-member, hardest in the study.** Two buyers, neither the resident: HOA software is built for volunteer boards, property-management software for professionals managing portfolios, and the big platforms (Condo Control, TownSq, ManageCasa) serve both. Either way the resident is the *administered subject* — pays dues, submits requests, receives notices, votes when balloted, reserves amenities — and in the professionally-managed case a paid third party stands between the community and itself. The most-administered member of any vertical.

**Communication is surveilled, not silent.** HOA platforms carry more two-way surface than ChMS (discussion forums, classifieds), but it’s board-controlled and board-logged: announcement boards are explicitly for posts where responses aren’t required, and all message-board activity is automatically logged and saved so boards can reference it when disputes arise. Same data-custody → liability → control logic as ChMS, but resolved by *surveillance* (member speech permitted but monitored for the board’s dispute and enforcement use) rather than by *silence*. The administrator still reads everything.

**Governance = administered compliance.** E-voting is annual board elections and statute-mandated ballot measures, run top-down by the board or manager and frequently outsourced to election specialists (ElectionBuddy, BigPulse, Vote HOA Now); compliance-first (state statute, governing documents, quorum, audit trails, secret ballot, proxy/vote-transfer and weighted voting). The member receives a ballot link. No member-initiated proposals, no continuous self-governance — voting as an episodic compliance event the board administers, not a standing capability the community owns.

**Three structural mismatches make HOAs a misfit:**

1. **Involuntary, property-based membership** vs Haven’s voluntary, in-person-vouched joining. The roster *is* the property records, administered top-down; Haven’s sybil-resistance and locality model doesn’t map onto “membership = deed.”
1. **Property-weighted voting and proxies** vs Haven’s people-vote / one-person-one-vote anchor. Serving HOAs would require primitives that *contradict* the people-vote foundation, not just extend it.
1. **Adversarial enforcement, and boards that *want* to surveil residents.** Violations, liens, architectural denials, and dispute documentation under binding state statute — with logged resident communications sold as a feature. Haven’s operator-blind model structurally *can’t* give a board “log everything residents say for dispute use,” so Haven’s core privacy property is, for an HOA board, a *missing feature*, not a selling point.

Net: the very things Haven offers (residents self-govern; the operator can’t surveil) are partly what an HOA *board* doesn’t want. HOAs are where Haven’s model fits *worst* despite looking like an obvious target. Haven’s natural verticals are **voluntary, mutual-trust bodies** — congregations, mutual-aid networks, co-ops, voluntary associations — where membership is chosen, voting is people-based, and the relationship is collaborative rather than adversarial. Self-managed small HOAs are the least-bad fit, but the property-membership and weighted-voting mismatches persist even there. Same shape as the FPF scope-discipline finding: don’t pitch where the model fits worse.

**Two credits to the incumbents.** HOAs prove strong *paid* demand for secure, auditable community voting and for unifying a famously messy fragmented stack (email threads, paper forms, spreadsheets, PDFs, enforcement records) into one system — and Haven’s native unification (governance + comms + events + payments + documents + credentials) genuinely beats the fragmented HOA stack *in the abstract*. And the HOA secret-ballot need (neighbor pressure, board retaliation) is a real-world instance of Haven’s Phase-6 vote-content-secrecy use case.

**The field:** ManageCasa, FRONTSTEPS, Condo Control, AppFolio, RealPage OneSite, TownSq, HOA Sites, PayHOA. Several already include e-voting, resident portals, communication, amenity booking, violation/architectural-request workflows — direct overlap with Haven’s governance/events/communication surfaces, serving the property manager rather than the community as a self-governing body.

### General-purpose tools (where the contrast flips)

Against these the contrast genuinely flips — they *are* two-way and mostly *don’t administer* the community — so the differentiator moves from “self-governance exists at all” to **self-governance that’s structural rather than granted, without surveillance, without operator ownership, and without attention extraction.** The lens is the “own your community” marketing overlap, and it surfaces a positioning *risk*, not just a contrast.

**The ownership spectrum (what “own” buys, and where it stops):**

- **Own nothing — rented land: Facebook Groups, Nextdoor.** The platform owns everything; the algorithm controls reach; they can delete your group; the model is attention/surveillance advertising (Nextdoor adds real-name local-deals data; it’s a public company under quarterly pressure). You are the product. This is the pole the others define themselves against.
- **Own admin rights, as a guest: Discord, Slack, Band.** You control your server/workspace — settings, membership, moderation — but on the operator’s infrastructure, under its content policies, readable by it, deplatformable by it. Discord is explicit that creators manage servers while *Discord provides the platform and enforces platform-wide content policies* (and it bought Sentropy to AI-scan content — the operator reads); Slack is blunter still — “the company owns the Slack, basically,” Salesforce owns the platform. Discord’s citizen-moderator culture is the Reddit pattern: real in culture, unpaid, granted, not structural — and ads are now entering Discord. “Own” = revocable administrative control inside someone else’s house.
- **Own brand + data + monetization — the marketed claim: Mighty Networks, Circle.** These actually sell the phrase (“Own Your Audience”: control your brand and member data, no algorithm deciding reach, keep your monetization), pitched directly against Facebook. Real and meaningful *versus Facebook* — but their own reviews concede the platform “retains control over user data to some extent,” they’re VC-backed creator-economy SaaS ($40–400/mo; Circle raised ~$24.7M), still hosted, readable, deplatformable, acquisition-exposed. “Own” = branding + export + monetization, and it **stops at the platform’s door.**
- **Groups.io** sits to the side: email-list utility, subscription not ads, genuinely portable (it’s email), but minimal governance and operator-readable — the most *neutral* of the set, the least *sovereign*.
- **Own structurally: Haven.** Operator can’t read (E2E), can’t deplatform (federation + migration with identity and members intact), governance enforced by cryptographic structure rather than platform permission, no attention engine to sell, no payment cut. “Own” = properties that hold *even against the operator*.

**The insight: every market “own your community” is ownership-as-permission; Haven’s is ownership-as-structure.** Branding, portability, admin control, monetization — all granted by the platform and revocable by it. The Reddit 2023 API blackout and Facebook’s power to delete a group are the proofs: when ownership is tested, it turns out to be the operator’s. Haven’s guarantees are enforced by architecture and survive the operator turning hostile, getting acquired, or shutting down. The competitors sell the *experience* of ownership on someone else’s substrate; Haven gives ownership of the substrate’s *guarantees*.

**The positioning risk — don’t lead with the slogan.** “Own your community” is **saturated and diluted**; Mighty Networks and Circle have spent its value, so the audience hears the thin meaning (branded space + export + monetize). If Haven leads with the phrase it sounds like Mighty-Networks-with-extra-steps (or crypto vaporware), and the structural difference is *invisible because the words are identical*. Haven must cash “own” out as the specific irrevocable claims the competitors *cannot* truthfully make: **the operator cannot read your members’ messages; you can move your whole community to another host and keep your members and identities; no one can delete or sell your community out from under you; no one is mining your members’ attention.** Lead with the four claims, not the word.

**Reddit (cautionary one-liner, not an entry):** the platform whose brand *is* citizen-moderation grants it by permission and revoked it the moment it mattered (the API blackout) — the cleanest illustration that culturally-granted self-governance is revocable, which is exactly what Haven makes irrevocable.

-----

## Cross-cutting findings

- **The headline result — no incumbent in any category has structural self-governance.** ChMS administers (member is a record); HOA administers and surveils (member is a subject); chat-first and community-platform tools offer two-way talk with role/permission hierarchies but no voting, proposals, or member-initiated binding decisions — and what self-governance culture exists (Discord/Reddit citizen-mods) is granted by the operator and revocable. Across the entire field, governance is either administration or unstructured talk with permissioned moderation. **Haven’s governance substrate is unique across the whole competitive landscape.** This is the single biggest result of the comparative study.
- **UX parity is table stakes; architecture is the switch.** “As smooth as Discord” is a reason to *stay* on Discord (design-notes UX/architecture split).
- **Payment model is a structural differentiator both ways.** Incumbents monetize giving/dues via transaction fees. Haven’s no-cut/no-skim is a clean contrast *and* the reason Haven can’t fund itself the way they do (design note 20).
- **The manager-vs-member axis — confirmed across both institutional verticals; flips on general-purpose.** ChMS: member is a managed record, no governance primitives (one-way by liability). HOA: hardest — most-administered subject, often a paid manager in between, member speech *surveilled* not silent. General-purpose: the axis flips (these are two-way and mostly don’t administer), so the differentiator becomes structural-not-granted ownership + no surveillance + no attention extraction.
- **“Own” is the contested word.** Every market “own your community” is ownership-as-permission (branding, export, monetization, admin control — revocable); Haven’s is ownership-as-structure (operator can’t read / can’t deplatform / can’t sell out from under you / isn’t mining attention). Lead with those four claims, not the saturated slogan.
- **Export ≠ Haven’s portability claim.** Incumbents already export CSV, so Haven’s edge is operator-can’t-read plus identity-intact migration, not mere data export. Don’t overclaim “you can leave.”

-----

## Synthesis — where Haven sits

The comparator sweep covered three survivors (FPF, Stocksy, Loomio) and three incumbent categories (ChMS, HOA, general-purpose). Where Haven sits:

- **Against the survivors:** FPF proves the posture works and is the closest living analog; Haven’s architecture is what would make FPF’s magic portable, operator-independent, and institution-shaped. Stocksy proves the capped, member-owned, capture-resistant economics last — but under purpose-homogeneity Haven forgoes, which is why schism and federation are load-bearing. Loomio proves the configurable self-governance layer is buildable and sustainable, and that it can be financed without ceding control (redeemable preference shares) — while showing Haven’s real bet is the membership-integrity, privacy, and federation substrate *underneath* the decision layer.
- **Against the institutional incumbents:** they serve the administrator, not the body politic — ChMS by silence, HOA by surveillance — and have no governance primitives. Haven’s edge is data-custody (operator can’t read) and structural self-governance, not mission or price (Planning Center already owns the mission high-ground in churches).
- **Against the general-purpose tools:** they’re two-way and don’t administer, so the contrast flips to ownership-as-structure vs ownership-as-permission, and no-surveillance/no-attention-extraction.

**In one line:** *no incumbent, in any category, offers structural self-governance — they offer administration, or talk with revocable moderation. Haven’s substrate is unique across the entire field, and the defensible edge everywhere is architectural (operator-blind, self-governing, non-extractive), cashed out as concrete irrevocable claims rather than the saturated “own your community” slogan.*

**Scope discipline carried forward:** don’t fight FPF on neighborhood-forum turf; don’t chase HOAs (structural misfit — involuntary property-membership, weighted/proxy voting, board-surveillance-as-a-feature); Haven’s natural verticals are voluntary mutual-trust bodies (congregations, mutual-aid, co-ops, voluntary associations). This carries forward into adoption planning.

**Complements the [founder-binding analysis](haven-founder-binding-analysis.md):** Planning Center (in-vertical, bootstrapped, no-VC, Ministry Centered Foundation), FPF (proof-of-demand with unsolved succession), Stocksy ($1-share capture-resistance), Loomio (redeemable preference shares — capital without control, the working instrument for the financing bind), and Exit to Community / steward-ownership as the named ownership-transition pattern.