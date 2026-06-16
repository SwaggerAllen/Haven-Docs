# Technical architecture, in brief

What Haven’s cryptographic and protocol architecture consists of, for a reader fluent in MLS, anonymous credentials, and transparency logs. Each section links into the matching part of the [architecture document](docs/design/civic-platform-architecture.md) for full detail.

Haven is a federated platform whose first-class entity is the community. The architecture composes MLS, a sender-key broadcast layer, BBS anonymous credentials, single-writer append-only mailboxes, a blind delivery hub, and CT-style transparency logs.

## Identity and credentials

[Durable identity](docs/design/civic-platform-architecture.md#identity) is a keypair-with-lineage: a lineage rooted at a genesis key, the current authentication key as the latest link, rotation by signed succession. Two credential layers attach to that anchor — MLS leaf credentials for device→persona binding, and BBS verifiable credentials carrying selectively-disclosable claims (membership, role, attendance, age).

There are [two issuers](docs/design/civic-platform-architecture.md#issuers). The instance alliance issues a universal **humanness** credential, blind over a committed secret `pid` it never learns. Communities issue local **decency** credentials (membership, role, standing, vouching). Humanness roots per-group pseudonyms and anchors the governance nullifier.

A single [BBS credential](docs/design/civic-platform-architecture.md#anonymous-credentials) over `pid`, using `draft-irtf-cfrg-bbs-per-verifier-linkability`, yields a pseudonym per `(pid, scope)` — constant within a scope, unlinkable across scopes. The same primitive, varied by scope and anchor, serves three consumers: per-community identity, voting nullifiers (scope = decision id, anchor = humanness `pid`), and ticketing nullifiers (scope = event id, anchor configurable). Cardinality caps — one-per-person votes, N-per-person tickets — are enforced by application-layer counting at a single counting authority per scope.

## Sybil resistance

Community membership requires an [in-person QR or Bluetooth handshake](docs/design/civic-platform-architecture.md#in-person-trust-as-default) with an existing member. The humanness credential — issued by the instance alliance over external proof (ID, government attestation), and carried across instances by the alliance’s durable identity — provides cross-community sybil resistance and roots the per-group pseudonyms above.

## Messaging

A community is a single MLS group (RFC 9420); multiple feeds ride on it through distinct partition-key derivations. Confidential roles are [branched MLS sub-groups](docs/design/civic-platform-architecture.md#confidential-sub-groups-and-the-role-write-enforcement-model) bound by the parent’s resumption PSK; [schism](docs/design/civic-platform-architecture.md#schism) uses the same branch primitive at the split epoch.

An alliance runs [two protocols at once](docs/design/civic-platform-architecture.md#the-two-layer-alliance-design):

- **Governance layer** — a representative MLS group (one representative per constituent community) with full guarantees, used for threshold multi-signature on alliance decisions and confidential coordination.
- **Messaging layer** — a sender-key protocol (Megolm-style, via vodozemac) for content visible to all members of all constituent communities, providing boundary confidentiality, sender authentication, and per-sender-ratchet forward secrecy.

## Key derivation and write gating

Partition keys, sender keys, franking material, sub-group PSKs, and write-capabilities all [derive from the per-epoch MLS exporter secret](docs/design/civic-platform-architecture.md#key-derivation-and-capture-forward-discipline) under versioned category labels (`haven.partition.v1`, `haven.senderkey.v1`, `haven.franking.v1`, …). The exporter is epoch-ephemeral; anything that must outlive an epoch is captured forward at production time rather than re-derived.

Write-role access is gated by a writer sub-group whose per-epoch write-capability the server verifies: an outer write-cap signature (server-visible, anonymous) gates landing, and an inner persona signature (inside the ciphertext, reader-visible) is attributable. Feed shapes follow from read access (community or sub-group) crossed with write access (community or writer sub-group).

## State and delivery

The single state primitive is the [mailbox](docs/design/civic-platform-architecture.md#mailbox-addressing): a composite address `(cryptographic_entity_id, mailbox_role)` with exactly one authoritative instance. State changes are [signed messages appended to a log](docs/design/civic-platform-architecture.md#signed-state-and-event-sourcing), and current state is the replay of that log. Federation is log exchange between instances, migration is replay in a new location, replication is local copies of remote logs; no consensus protocol is used.

The instance runs a [blind delivery hub](docs/design/civic-platform-architecture.md#the-blind-hub) that sequences (consistent order per partition key) and fans out (pull-based). [Partition keys](docs/design/civic-platform-architecture.md#partition-keys-for-routing) — derived from MLS exporter or sender-key state, epoch-rotated — are opaque routing identifiers; the hub does not read content or track membership. Cross-mailbox references are `(partition_key, commitment)` pairs that remain valid across rotation.

## Franking

Verifiable abuse reporting under end-to-end encryption uses [message franking](docs/design/civic-platform-architecture.md#message-franking-for-verifiable-reporting): the sender produces a signed commitment to plaintext during encryption (over committing AEAD); the recipient holds an opening that binds the commitment to the plaintext; a report carries the opening. Franking applies to every encrypted context, with the default disclosure policy varying by context. Verification is deterministic and performed by infrastructure at each layer, producing verified-evidence packages; enforcement is a separate governance function (juries, councils). Broken franking escalates by per-message key reveal.

## Transparency and replicas

[CT-style append-only Merkle logs](docs/design/civic-platform-architecture.md#verifiable-logs) — with inclusion proofs, consistency proofs, and witness cosigning — back equivocation defense, replica integrity attestations, and audit trails. Haven adopts the C2SP tlog-witness protocol, participates in the existing witness network, and uses temporal sharding (six-month shards, the Sunlight/TesseraCT pattern). The log runs as a Go Static-CT service alongside the Rust core.

[Replication is three-tier](docs/design/civic-platform-architecture.md#three-tier-replication): one authoritative instance per mailbox, durable allied full replicas held by instance alliances under agreement, and decaying cache replicas. Each replica publishes signed Merkle roots over its holdings to the logs, uniformly across all content.

-----

*This documentation was drafted with substantial AI assistance. The architecture, the decisions, and the mistakes are the author’s.*

— Allen Strut, Strutco