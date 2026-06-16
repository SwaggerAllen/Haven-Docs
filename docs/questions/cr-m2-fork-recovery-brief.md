# Fork detection and recovery in a single-writer, blind-hub MLS deployment

*In brief: Haven sequences each MLS group through a single-writer, blind delivery hub, which forbids forks by construction — so a fork is always an attack signature (operator equivocation, or a botched replica failover), not a routine network condition. This lets Haven use DMLS for recovery rather than for ongoing decentralized operation: a member stranded on a losing branch self-heals onto the survivor from retained state. The cost, and the open question (§4): because a fork isn’t detectable until after the fact, every member must speculatively retain each epoch’s recovery material until convergence — and the ratchet-tree keys in that material are held in the clear, a standing forward-secrecy and post-compromise cost. Can that be brought down short of the theoretical HIBE construction?*

## 1. Setting

An MLS group’s messages are sequenced and fanned out by a single-writer **blind hub**: it orders ciphertext and delivers it, but cannot read content, see handshake structure, or learn group membership. Groups post to mailboxes addressed by per-epoch **partition keys** derived from the group’s MLS exporter secret; the hub sees only opaque, rotating keys. Select peers run **replicas** that mirror content. Because there is a single writer, a hostile hub’s powers reduce to availability (drop, delay, withhold) and ordering-integrity (fork, rollback): it is, at worst, a vandal rather than a spy.

Single-writer ordering means that under an honest operator there is exactly one chain. A fork is therefore never routine; it signals an attack: the operator **equivocating** (serving divergent commits for one epoch to different members), or a failover **manufacturing** a fork by promoting a replica while the original writer is still live, leaving two authoritative writers. This is why DMLS [DMLS], built on Fork-Resilient CGKA [FRCGKA], is used here as incident-scoped recovery rather than for routine out-of-order operation.

## 2. Recovery by self-heal

A member is *stranded* when they transition off the last consistent epoch — the **resumption epoch** — onto a branch that loses. Because they were at the resumption epoch, they hold its state, and DMLS lets them derive the surviving branch’s epochs directly from the retained punctured init secret, with no re-add, Welcome, or KeyPackage. What they need *delivered* is the surviving branch’s commit sequence, which is opaque ciphertext — served by a replica or any member already on the survivor.

A member who has already deleted the resumption epoch’s state — good forward-secrecy hygiene, faster than the fork was proven — cannot self-heal and falls back to re-onboarding from scratch. That fallback is what bounds the stakes of the question below: self-heal need only be the cheap common case, not a guarantee for every member.

## 3. Detection

Detection cannot ride the delivery channel. When the hub equivocates, the two branches are distinct epochs with distinct exporter-derived partition keys, so the victims are routed into separate partitions and each side sees what looks like a clean chain — the fork is invisible on the wire. Instead the **epoch authenticator** (RFC 9750) is the comparison object: two different authenticators for the same group and epoch are proof of a fork. These are recorded in a CT-style, witness-cosigned Merkle log under an index both branches can derive from their shared ancestor.

There is a writer-side limitation that matters for §4. The hub writes the log, so an equivocating hub can decline to write, or delay writing, the losing branch’s authenticator. It cannot forge or hide an entry it has written, but it controls *when* the divergence becomes provable — which bounds detection latency below and leaves the retention window stretchable.

Retention has to be speculative, which is the crux of the cost. A member cannot know in advance which epoch will be forked, and deleted private keys cannot be recovered, so to self-heal at all it must hold each epoch’s recovery material before any fork is detectable, and keep holding it until the group provably converges past that epoch. That material is two parts: the punctured init secret, and the ratchet-tree private keys the member holds at that epoch. The init half is cheap, since puncturing forward-secures the branch actually taken; the tree keys are retained in the clear, softening forward secrecy and — because others may still encrypt to them — weakening post-compromise security. Because the holding is continuous and per-epoch, this is a standing cost, not an incident-scoped one: a rolling window whose length is set by detection latency, which an equivocating hub can stretch.

## 4. The question: the cost of retained state

The standing cost above is the ratchet-tree private keys held in the clear. The init secret is already forward-secured by puncturing; the only construction that would do the same for the tree keys is the HIBE-based variant in [FRCGKA], which is theoretical-efficiency-only and not in the deployable draft. So the question is whether that cost can be brought down some other way.

There seem to be three ways the cost could come down, and I can’t tell how far any of them goes. **Shorten the window** — detection latency is partly Haven’s to drive down, though a hub that controls log writes can stretch it. **Forward-secure the retained keys** the way puncturing already forward-secures the init secret — short of the HIBE variant in [FRCGKA], is there a cheaper analog for the ratchet-tree decryption keys? **Retain fewer keys** — but retention is blind (a member can’t see the surviving branch until recovery, so it can’t know which of its keys the survivor will rotate away), so it currently has to hold its whole path-key set for each epoch until convergence; is there any way to narrow that?

The middle one is where I’d most want your eye, and the last is the one I’m least sure is even possible.

## 5. Secondary questions

Listed in case one sits in your field; §4 is the actual ask.

- **Composition with exporter-derived values.** Haven derives partition keys, write-capability keypairs, and other secrets from the MLS exporter; DMLS derives multiple init secrets via the Secret-Tree-as-PPRF. These should be independent — each branch is a distinct epoch with a distinct exporter secret — but does the modified key schedule disturb the exporter outputs in any way, or is there a seam I’m missing?
- **Composition with sub-group branching.** Confidential sub-groups have their own exporter and a resumption-PSK binding to the parent. Does the retained-tree-key PCS softening compound across nested groups during an incident?

## 6. Scope notes

- DMLS is an individual Informational Internet-Draft; the load-bearing citation is the peer-reviewed FRCGKA paper, with DMLS as the deployable specification.
- Branch selection — which fork survives — is treated as a consensus/governance problem and is out of scope for this note.

## References

- [FRCGKA] Alwen, Mularczyk, Tselekounis. *Fork-Resilient Continuous Group Key Agreement.* CRYPTO 2023. ePrint 2023/394.
- [DMLS] Kohbrok. *Decentralized Messaging Layer Security.* `draft-kohbrok-mls-dmls` (individual Internet-Draft).
- RFC 9420, *The Messaging Layer Security (MLS) Protocol.*
- RFC 9750, *The Messaging Layer Security (MLS) Architecture.*

-----

*This document was drafted with substantial AI assistance. The architecture, the decisions, and the mistakes are the author’s.*