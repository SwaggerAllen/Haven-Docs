# Civic Platform Federation Readiness

## Contents

- [Architectural commitments for future federation](#architectural-commitments-for-future-federation)
- [1. The principle](#1-the-principle)
- [2. Core commitments](#2-core-commitments)
  - [2.1 Cryptographic identity is the basis of all identity](#21-cryptographic-identity-is-the-basis-of-all-identity)
  - [2.2 Instance identity is cryptographic and collectively held](#22-instance-identity-is-cryptographic-and-collectively-held)
  - [2.3 Mailboxes have composite addresses](#23-mailboxes-have-composite-addresses)
  - [2.4 Communities and users have home instances](#24-communities-and-users-have-home-instances)
  - [2.5 Federation is direct, not transitive](#25-federation-is-direct-not-transitive)
  - [2.6 Federation establishes through social structure](#26-federation-establishes-through-social-structure)
  - [2.7 Public content does not require federation](#27-public-content-does-not-require-federation)
  - [2.8 State derives from message logs](#28-state-derives-from-message-logs)
  - [2.9 Replicas come in tiers with different durability properties](#29-replicas-come-in-tiers-with-different-durability-properties)
  - [2.10 Writes go to the authoritative instance](#210-writes-go-to-the-authoritative-instance)
  - [2.11 Defederation is unilateral](#211-defederation-is-unilateral)
  - [2.12 Migration moves the authoritative state](#212-migration-moves-the-authoritative-state)
- [3. Data model implications](#3-data-model-implications)
  - [3.1 The Instance entity](#31-the-instance-entity)
  - [3.2 Mailbox addressing](#32-mailbox-addressing)
  - [3.3 Replica tracking](#33-replica-tracking)
  - [3.4 Federation Relationship](#34-federation-relationship)
  - [3.5 Event sourcing](#35-event-sourcing)
  - [3.6 Identifier scope](#36-identifier-scope)
- [4. Routing semantics](#4-routing-semantics)
  - [4.1 Local routing path](#41-local-routing-path)
  - [4.2 Read path](#42-read-path)
  - [4.3 The point](#43-the-point)
- [5. Bootstrap and discovery](#5-bootstrap-and-discovery)
  - [5.1 Initial address discovery](#51-initial-address-discovery)
  - [5.2 URL updates from peers](#52-url-updates-from-peers)
  - [5.3 No public addressing system needed](#53-no-public-addressing-system-needed)
- [6. Migration semantics](#6-migration-semantics)
  - [6.1 Community migration](#61-community-migration)
  - [6.2 User home instance migration](#62-user-home-instance-migration)
  - [6.3 Migration atomicity](#63-migration-atomicity)
- [7. Replication semantics](#7-replication-semantics)
  - [7.1 Replica tiers](#71-replica-tiers)
  - [7.2 What replicas hold](#72-what-replicas-hold)
  - [7.3 How to keep replicas updated](#73-how-to-keep-replicas-updated)
  - [7.4 Cache replica lifecycle](#74-cache-replica-lifecycle)
  - [7.5 Allied replica lifecycle](#75-allied-replica-lifecycle)
  - [7.6 Replica integrity](#76-replica-integrity)
  - [7.7 Failure modes and recovery](#77-failure-modes-and-recovery)
  - [7.8 Public verifiable logs](#78-public-verifiable-logs)
- [8. Operator transitions](#8-operator-transitions)
  - [8.1 The instance alliance keypair](#81-the-instance-alliance-keypair)
  - [8.2 Operator authorization](#82-operator-authorization)
  - [8.3 Cooperative operator transition](#83-cooperative-operator-transition)
  - [8.4 Hostile operator transition](#84-hostile-operator-transition)
  - [8.5 Abandoned operator transition](#85-abandoned-operator-transition)
  - [8.6 Composition changes](#86-composition-changes)
  - [8.7 Hardware migration](#87-hardware-migration)
  - [8.8 What is collectively held vs what is operator-held](#88-what-is-collectively-held-vs-what-is-operator-held)
  - [8.9 Limits](#89-limits)
- [9. Federation establishment](#9-federation-establishment)
  - [9.1 Triggers](#91-triggers)
  - [9.2 Mechanism](#92-mechanism)
  - [9.3 First contact](#93-first-contact)
  - [9.4 Federation establishment is not negotiation](#94-federation-establishment-is-not-negotiation)
- [10. What is explicitly deferred](#10-what-is-explicitly-deferred)
- [11. Tests for federation readiness](#11-tests-for-federation-readiness)
- [12. Relationship to other documents](#12-relationship-to-other-documents)

-----

## Architectural commitments for future federation

This document captures the architectural commitments that must be honored in the pilot implementation so that future federation work is not a rewrite. It is not a federation protocol specification. The wire protocol, handshake details, error semantics, and similar implementation specifics are deferred to actual federation implementation, which happens post-pilot.

The audience for this document is anyone implementing the pilot or working on data model decisions that may affect federation. The commitments here constrain how the pilot must be built, even though federation itself is not in the pilot’s scope.

-----

## 1. The principle

Federation is the technical layer by which instances cooperate to host a single logical platform. Federation cannot be added cleanly to a system whose data model assumes single-instance operation. The pilot must be built with federation in mind even though it deploys to a single instance.

The cost is small if commitments are honored from the start: a few extra columns in tables, a routing abstraction that’s trivial in single-instance mode, an instance entity that always points to “us.” The cost of retrofitting these is large: schema migrations, identifier rewriting, protocol refactoring, semantic ambiguity in legacy data.

This document lists the commitments. Each one is small individually. Together they preserve the option of federation without committing to its implementation.

-----

## 2. Core commitments

### 2.1 Cryptographic identity is the basis of all identity

- Users are identified by keypairs, not by `user@server` style identifiers
- Communities are identified by keypairs
- Alliances are identified by keypairs
- Instances are identified by keypairs
- The keypair is the durable identity; any human-readable name or URL is a routing hint that can change

### 2.2 Instance identity is cryptographic and collectively held

- An instance is composed of an operator entity and an instance alliance (see Instance operation in the feature list)
- The instance alliance has a cryptographic identity that serves as the instance’s federation identity
- The instance alliance is itself an MLS group whose members are the communities hosted by the instance
- Alliance authority is exercised through threshold multi-signature: 2/3 of representative communities (default; configurable within bounds) must sign for alliance actions to be valid
- The operator entity has its own keypair for internal operator authority (software signing, deployment authorization)
- The operator entity is authorized by the instance alliance through a signed attestation; this authorization can be revoked or replaced by alliance action
- Other instances federate with the instance alliance, verifying multi-signature against current composition
- The alliance identity is durable across operator changes — operator transitions are alliance actions that sign over operator authority from one entity to another
- Instance URL is a routing hint that can change; the alliance identity is permanent
- An instance changing URLs but keeping its alliance composition is the same instance
- An instance changing operators but keeping its alliance composition is the same instance from federation’s perspective
- Operations claiming to come from an instance are signed by threshold of the instance alliance’s current representative composition

### 2.3 Mailboxes have composite addresses

- Mailboxes are identified by (instance ID, UUID) pairs
- The instance ID identifies the authoritative instance for that mailbox
- The UUID identifies the specific mailbox on that instance
- Local routing is single-instance; the instance ID happens to always be “us”
- Cross-instance routing uses the instance ID to forward to the authoritative instance
- Code that processes mailbox addresses works uniformly whether the instance is local or remote

### 2.4 Communities and users have home instances

- Each community has an instance that hosts its authoritative state
- Each user has a home instance, determined by their home community
- A user moves home instances when their home community moves
- A community’s home instance is implicit in where its mailboxes live (their instance ID)

### 2.5 Federation is direct, not transitive

- An instance routes messages only to other instances it has direct federation with
- No routing through intermediate instances
- If A federates with B and B federates with C, A does not automatically reach C through B
- Communication paths are explicit federation relationships

### 2.6 Federation establishes through social structure

Federation between two instances is established when shared social structure first crosses their boundaries. The triggers are:

- A user on one instance joins a community on another instance
- A community on one instance joins an alliance that has members on another instance
- A community on one instance is targeted by a migration from another instance

These are the only ways federation forms. Federation does not form through:

- Instance operator decision in isolation
- Discovery directories or registries (though these can advertise instances)
- Speculative pre-establishment

The principle: federation is a technical consequence of real social or organizational connection. It is never abstract.

### 2.7 Public content does not require federation

- Public content (public feeds, public community pages, registry listings) is served as standard web content
- Any user agent can fetch public content from any instance without federation
- Discovery of communities through registries is browsing public content
- Federation only matters when private operations happen (joining, posting authenticated content, participating in governance)

### 2.8 State derives from message logs

- All meaningful state changes happen via signed messages persisted to a log
- Current state is derivable by replaying the log from the beginning
- The database is a cache of derived state; the log is the source of truth
- Migration is replaying logs in a new location
- Federation message exchange is log exchange

### 2.9 Replicas come in tiers with different durability properties

The platform supports three replica tiers plus user-side backup for portability. The architectural commitment is that the data model and protocol distinguish these tiers; the detailed policies are in the feature specification.

- **Authoritative**: exactly one instance per mailbox holds the source of truth
- **Allied full replicas**: durable, complete replicas held by instance alliances under explicit agreement; primary protection against operator hostility
- **Cache replicas**: automatic, partial, decaying replicas held wherever there is local interest; convenience and partition tolerance
- **User-side backup**: portability fallback; periodic snapshots users can hold

Allied replica agreements are made between instance alliances (the governance bodies that speak for operators on infrastructure commitments), not between communities directly. Communities express preferences; their instance alliances negotiate.

Replicas publish signed attestations of their holdings to make selective withholding detectable.

### 2.10 Writes go to the authoritative instance

- Reads can happen from any tier with a replica holding the content
- Writes always go to the authoritative instance for the mailbox
- The authoritative instance accepts writes in some order; that order is canonical
- Replicas eventually reflect the authoritative order
- Within-mailbox conflict resolution is unnecessary because there is a single authoritative source (cross-mailbox causality requires more thought; see open questions)

### 2.11 Defederation is unilateral

- An instance can defederate from a peer at any time without negotiation
- Defederation stops sending to and accepting from the defederated peer
- Users on the defederating instance lose access to communities on the defederated peer
- Communities on the defederated peer continue to operate; they’re just unreachable from this instance
- Defederation is a local choice with local consequences

### 2.12 Migration moves the authoritative state

- A community migration changes the authoritative instance for that community’s mailboxes
- Migration is atomic from the perspective of the moving community
- Peers update their instance ID → URL mappings based on signed announcements
- Replicas continue to function during migration (potentially with brief staleness)
- The community’s cryptographic identity does not change during migration

-----

## 3. Data model implications

The following data model decisions follow from the commitments above. The pilot implementation must include these even though federation is not exercised.

### 3.1 The Instance entity

An instance is composed of two related entities:

```
OperatorEntity:
  - id (primary key)
  - public_key (operator keypair)
  - operator_name
  - jurisdiction
  - contact_info
  - alliance_authorization (signed attestation from alliance authorizing this operator)
  - ...

InstanceAlliance:
  - id (primary key)
  - alliance_id (the alliance keypair / cryptographic identity)
  - operator_entity_id (which operator currently runs this instance)
  - current_url (routing hint; mutable)
  - mls_group_id (the alliance's MLS group; member communities are leaves)
  - threshold (default 2/3 of composition; configurable within governance bounds)
  - governance_configuration (template selection)
  - established_at
  - last_seen_at
  - federation_status (active, defederated, etc.)
```

The instance alliance uses a two-layer design: a governance MLS group whose members are designated representatives of hosted communities, exercising authority through threshold multi-signature; and a sender-key messaging layer for broader alliance content visible to all members of constituent communities. Alliance actions require threshold multi-signature from the governance layer. The alliance’s authority is held collectively by its representative communities, not by the operator.

In the pilot (single-instance, one community), the alliance has a single representative (that community); threshold of one is one. As more communities join the instance, they become alliance members; threshold becomes meaningful as composition grows.

The pilot has one InstanceAlliance row representing the local instance, with one OperatorEntity row for the local operator. Other instances would be added as federation establishes.

### 3.2 Mailbox addressing

```
Mailbox:
  - id (UUID, locally unique)
  - authoritative_instance_id (foreign key to Instance)
  - mailbox_type (relationship, feed, etc.)
  - created_at
  - ...other fields
```

In pilot, `authoritative_instance_id` always points to the local instance row. The code that uses mailbox addresses must respect both parts of the address even when remote routing is stubbed.

### 3.3 Replica tracking

```
Replica:
  - mailbox_address (instance ID + UUID)
  - local_state (derived state for this replica)
  - last_sync_at
  - established_via (what created this replica)
```

Not used in pilot but the schema accommodates it.

### 3.4 Federation Relationship

```
FederationRelationship:
  - local_instance_id
  - remote_instance_id
  - established_at
  - status (active, suspended, defederated)
  - established_via (which user/alliance/migration triggered this)
```

Not used in pilot but documented.

### 3.5 Event sourcing

All state changes flow through signed messages. The message log is canonical. Current state is derived. This is a significant implementation pattern that affects almost every table — the schema for current state is the cache; the schema for the message log is the source of truth.

The pilot must implement event sourcing throughout, not just for things that obviously need it. Retrofitting event sourcing later is painful.

### 3.6 Identifier scope

All identifiers used in cryptographic operations or external references are globally unique by virtue of being cryptographic (keypairs) or random (UUIDs). No auto-incrementing integers as primary identifiers in domain entities. Auto-incrementing IDs can exist as internal database optimizations but the canonical reference is always the cryptographic or UUID identifier.

-----

## 4. Routing semantics

The routing layer in the pilot is trivial because everything is local. But the abstraction must exist so that adding federation later is adding routing decisions, not restructuring.

### 4.1 Local routing path

- Receive write request for mailbox at (instance_id, uuid)
- Check authoritative instance ID
- If local: process directly
- If remote: stub (in pilot, error or no-op; later, forward)

### 4.2 Read path

- Receive read request for mailbox at (instance_id, uuid)
- Check local replicas
- If local replica exists: serve from replica
- If authoritative is local: serve from primary store
- If authoritative is remote and no replica: stub (in pilot, error; later, fetch or proxy)

### 4.3 The point

The routing layer is a single point that’s trivial in pilot and the natural place to add federation. Without it, federation is a structural change. With it, federation is filling in stubs.

-----

## 5. Bootstrap and discovery

### 5.1 Initial address discovery

When a user invites another user via QR code (or equivalent in-person mechanism), the QR encodes:

- The inviting user’s identity public key
- The inviting user’s instance ID (public key)
- The inviting user’s instance current URL
- The community being invited to
- Signed invitation token

The recipient’s client uses this to:

- Verify the invitation (cryptographic verification)
- Resolve the instance ID to a URL (from the QR)
- Initiate federation establishment with the inviter’s instance (if not already federated)

### 5.2 URL updates from peers

If an instance’s URL changes:

- While operational: instance signs a message announcing the new URL and sends to all known peers via existing federation channels
- After unexpected downtime: when the instance comes up at a new URL, it signs a message announcing the new URL and contacts each known peer

In both cases the message is signed by the instance alliance keypair (the federation identity). Peers verify the signature and update their instance ID → URL mapping. The cryptographic identity is what matters; the URL is just the current route.

### 5.3 No public addressing system needed

Because federation is direct, instances only need to know addresses for their immediate peers. There is no need for a global directory of instance ID → URL mappings. Each instance maintains its own mapping for the peers it federates with.

-----

## 6. Migration semantics

### 6.1 Community migration

When a community migrates from instance A to instance B:

- A’s authoritative state for the community’s mailboxes is transferred to B
- The community’s mailbox addresses update: `authoritative_instance_id` changes from A to B
- The community’s cryptographic identity (keypair) does not change
- B becomes the new authoritative instance for these mailboxes
- A may retain a replica during a grace period
- Peers learn of the migration through signed messages from the community
- Peers update their routing to send writes for these mailboxes to B

### 6.2 User home instance migration

When a user’s home community migrates (and therefore the user’s home instance migrates):

- The user’s identity keypair does not change
- The user’s home_instance_id updates
- Other instances the user has memberships on update their understanding of the user’s home
- Federation relationships with those instances may need to be re-established from the new home

### 6.3 Migration atomicity

Migration appears atomic from the perspective of users:

- Before: state is at A
- After: state is at B
- During: dual-write or quick cutover (implementation choice)

The signed migration announcement is the canonical moment of transition. Before it, A is authoritative. After it, B is authoritative.

-----

## 7. Replication semantics

### 7.1 Replica tiers

Three tiers of replication exist, with different establishment, content, and lifecycle properties.

**Allied full replicas** are established by explicit instance-alliance agreement. Communities express preferences for allied replication; their instance alliances negotiate the actual commitments (because instance alliances, not communities, control the infrastructure on which replicas live). Allied replicas hold the full encrypted message log indefinitely (or until the agreement ends).

**Cache replicas** form automatically wherever there is local interest. An instance creates a cache replica when a local user joins a community whose authoritative instance is remote, or when a local community joins an alliance with members on remote instances. Cache replicas hold a subset of content per configurable policy.

**User-side backup** is not a server-side replica but a user-side portability mechanism. Users can export their accessible content periodically. Provides lossy point-in-time recovery for communities without allied replicas.

### 7.2 What replicas hold

For cache replicas, content is held according to per-content-type policy:

- Membership state and governance state: held fully (small and critical)
- Recent message logs: held fully during active participation
- Older message logs: subject to TTL-based decay per configuration
- Media: lazy-fetched, possibly content-addressed within encryption boundaries

For allied full replicas, content is byte-for-byte equivalent to what the authoritative instance stores (modulo encryption details). Decryption depends on member keys; the replica contains encrypted blobs only.

### 7.3 How to keep replicas updated

The authoritative instance pushes new messages to peers maintaining replicas as they occur. Periodic reconciliation handles missed pushes (the peer asks “what have I missed since message X”). Both cache replicas and allied full replicas use the same update mechanism; they differ in retention, not in update.

### 7.4 Cache replica lifecycle

Cache replicas are created on first interest, maintained while interest exists, and garbage collected when interest ends. Configurable parameters:

- Storage budget per instance (operator policy)
- TTL for inactive content per content type
- Grace period for cleanup after local interest ends
- LRU eviction when storage budget is approached

If interest resumes after cleanup, the cache replica is reconstructed by fetching from the authoritative instance again.

### 7.5 Allied replica lifecycle

Allied replicas are established by negotiated agreement between instance alliances. The agreement specifies:

- Which communities/mailboxes are covered
- Duration or renewal terms
- Reciprocity (mutual replication) or compensation
- Termination conditions

Allied replicas survive defederation if the agreement predates it (the agreement is the commitment, not the federation status). They are garbage collected when the agreement ends, not when interest ends.

### 7.6 Replica integrity

Replicas publish signed attestations of their holdings periodically. Attestations include Merkle roots over current content. The attestations are submitted to public verifiable logs operated by the Foundation (see Section 7.8), and peers verify against the public log state rather than trusting operator-provided state. Mismatches between attested holdings and served content are detectable by clients. Witnesses sign log heads to prevent log operators from equivocating. This makes selective withholding (the “replica as censorship tool” concern) detectable through cross-checking against log attestations; governance action (community, alliance, or instance) acts on detected misbehavior.

### 7.7 Failure modes and recovery

- **Authoritative operator goes hostile**: community migrates using allied replicas as new authoritative source
- **Allied instance fails**: other allied replicas continue; community establishes new agreements
- **All instances fail**: user-side backups provide lossy point-in-time recovery
- **Community has no allies**: user-side backups are the available protection; this is a known limitation
- **Cache replica withholds content**: detectable through attestation mismatches against public log state

### 7.8 Public verifiable logs

The Foundation operates public verifiable logs as platform infrastructure, analogous to Certificate Transparency in the web PKI. Multiple log uses:

- Replica integrity attestations (preventing selective withholding by replicas)
- Equivocation defense (preventing instances from presenting different state to different federation peers)
- Audit trails for moderation actions and governance decisions
- Possibly franking verification support

Operators submit attestations; peers verify against public log state rather than trusting operator-provided state directly. Witnesses sign log heads to prevent log operators from equivocating. Logs are publicly readable; what’s logged is signed and (where applicable) encrypted, so what’s public is structural integrity rather than content.

Specific library choice (Rust alternatives to Trillian, which is Go-only) is pending investigation. The Foundation operating these logs is analogous to its operation of the default credential trust framework and operator/plugin certification — public infrastructure that operators rely on rather than each operator running their own equivocation defense.

This is foundational infrastructure that all federation participants depend on. Operators commit to submitting required attestations as part of operator certification.

-----

## 8. Operator transitions

The instance alliance authority is held collectively by representative communities through MLS group membership. Operator transitions are alliance actions: the alliance signs an attestation transferring operator authority from one entity to another. The protocol handles cooperative transitions (outgoing operator participates), hostile transitions (outgoing operator refuses cooperation), and abandoned transitions (outgoing operator has disappeared).

### 8.1 The instance alliance keypair

The instance alliance does not have a single shared private key. Instead, the alliance’s MLS group state is the source of truth for current composition. Alliance actions are valid when signed by threshold (default 2/3) of current representative communities, verifiable by anyone with the current MLS group state.

This means the “alliance keypair” is properly understood as the cryptographic identity established by the MLS group, with authority exercised through threshold multi-signature from current members. There is no single key that could be stolen, lost, or held hostage.

### 8.2 Operator authorization

An operator is authorized by a signed attestation from the alliance. The attestation includes:

- The operator entity’s public key
- The alliance’s identity
- The terms of authorization (typically broad: “this entity operates infrastructure for this alliance”)
- Effective timestamp
- Signed by threshold of alliance representatives

Federated peers verify this attestation when accepting messages from an operator. An operator without current alliance authorization is not recognized as legitimate.

### 8.3 Cooperative operator transition

Normal case where the outgoing operator participates:

1. Outgoing operator and new operator coordinate the change (operationally: hardware setup, data migration, scheduling)
1. New operator entity is established with its own keypair
1. Alliance representatives gather to sign the transition attestation
1. The attestation states: “alliance authorizes operator change from [outgoing] to [incoming], effective [timestamp]”
1. Threshold of representatives sign on their own devices
1. New operator coordinates signature collection (representatives don’t need to coordinate with each other)
1. When threshold is reached, the combined attestation is published to federated peers
1. Peers verify signatures against current alliance composition
1. Peers update their record of which operator is authorized for this alliance
1. New operator begins operating with verified authority

### 8.4 Hostile operator transition

Case where the outgoing operator refuses to cooperate:

1. Alliance representatives recognize the operator has gone hostile (suppressing updates, refusing migration requests, etc.)
1. New operator (volunteer or chosen by alliance governance) is identified
1. Alliance representatives gather to sign the transition attestation
1. Critical: representatives sign on their own devices, not through the operator’s infrastructure
1. Representatives’ local MLS state copies provide the alliance composition reference
1. If representatives have divergent state (because operator suppressed updates), they reconcile to the most recent state that 2/3+ agree on
1. New operator coordinates signature collection out-of-band (email, encrypted channels, in-person meetings — not through the hostile operator’s infrastructure)
1. When threshold is reached, the combined attestation is published to federated peers
1. Peers verify signatures and update operator authorization
1. New operator begins serving from new infrastructure (allied replica, backup, or fresh setup with data from user-side backups)
1. Federated peers now route to the new operator’s URL; old operator is no longer recognized

Communities that did not participate in the migration (those representing the dissenting minority, if any) remain on the original infrastructure under the original operator. They effectively form a new instance — they no longer have alliance authority over their former instance. This is consistent with the schism pattern: majority preserves continuity, dissenters lose continuity.

### 8.5 Abandoned operator transition

Case where the outgoing operator has disappeared (death, bankruptcy, hardware loss, etc.) without active hostility:

The protocol is essentially the same as the hostile case. Representatives’ local MLS state provides the alliance composition; threshold of representatives sign the transition; new operator takes over. The difference is operational: data recovery may depend on allied replicas (Tier 2) or user-side backups (Tier 4) since the original instance may be entirely unrecoverable.

This is why allied replicas matter — they are the primary protection against sudden operator loss. Communities without allied replicas are dependent on user-side backups, which are lossy and incomplete.

### 8.6 Composition changes

Adding or removing representative communities is itself an alliance action through normal MLS group operations:

- Add: MLS Add proposal, committed by threshold of current members
- Remove: MLS Remove proposal, committed by threshold of current members
- Update: MLS Update proposal for key rotation, no governance significance

Composition changes propagate to federated peers through signed announcements. Peers verify the announcement against the previous epoch’s composition. The chain of composition updates is verifiable from any known starting point.

### 8.7 Hardware migration

Hardware migration (new servers, possibly new URL) is separate from operator transition but often coincident. The protocol elements:

- If operator is unchanged: data migration is operationally complex but doesn’t require alliance signatures (operator is still authorized)
- URL change: signed announcement from current operator, verifiable by alliance authorization
- If hardware change requires data migration: standard data migration patterns apply (dual-write or quick cutover)
- If hardware change coincides with operator change: the operator transition attestation can also include the new URL

### 8.8 What is collectively held vs what is operator-held

To be explicit:

**Collectively held by the alliance:**

- Alliance identity (cryptographic, via MLS group)
- Authorization of who is the operator
- Authority to change the operator
- Authority to change the URL
- Authority to change alliance composition
- Authority over federation policy decisions

**Held by the operator:**

- Operator entity keypair (for infrastructure-level signatures)
- Infrastructure access (servers, networks, storage)
- Day-to-day operational decisions within the scope of authorization

The operator cannot unilaterally change the alliance identity, change its own authorization (only the alliance can do that), or claim authority for actions reserved to the alliance. The alliance cannot unilaterally take over infrastructure (the operator has the physical access), but it can revoke the operator’s authority and authorize a different entity to provide infrastructure.

### 8.9 Limits

The architecture provides structural protection against hostile operators, but it has limits that should be stated honestly:

- **Threshold compromise**: if more than 1/3 of representatives are compromised or coerced, the alliance can be captured. The protocol cannot defend against this; the defense is in how representative composition is chosen.
- **Sufficient representatives offline**: an alliance whose representatives are insufficiently engaged can deadlock. The recourse is the schism path; representatives who remain engaged can form a new alliance, but with continuity loss for any history before the schism.
- **Bootstrap period**: very early instances with few representatives have weaker protection. A single-representative instance alliance is effectively founder-controlled until composition grows.
- **Total infrastructure failure**: if all instances holding alliance data fail simultaneously without recovery paths, the alliance is unrecoverable. This is the case where user-side backups provide degraded recovery.

These limits are inherent to the design. The architecture raises the cost of compromise substantially compared to single-key or single-operator-controlled systems, but does not make compromise impossible.

-----

## 9. Federation establishment

### 9.1 Triggers

Federation between two instances establishes when:

- A user from instance A joins a community on instance B
- An alliance forms whose member communities span A and B
- A community migrates from A to B (or vice versa)

### 9.2 Mechanism

In each case, an authenticated request crosses from one instance to the other:

- For user join: the user’s identity key + invitation token reaches B via A’s federation request
- For alliance formation: the alliance’s bootstrap creates federation requests between member communities’ instances
- For migration: the migration announcement establishes the new federation relationship

Each request is cryptographically authenticated. The receiving instance verifies and either accepts or rejects.

### 9.3 First contact

For first contact, the requesting instance needs the receiving instance’s URL. This comes from:

- QR code (for user-driven join)
- Alliance bootstrap (alliance’s instance URL is shared at alliance formation)
- Migration announcement (announcing instance shares its current URL)

After first contact, the instances know each other’s keypairs and current URLs. Subsequent communication uses these without needing external discovery.

### 9.4 Federation establishment is not negotiation

There’s no separate “federation handshake” beyond the initial authenticated request. Federation is established by virtue of the first successful cross-instance operation. Both instances record the federation relationship and proceed.

-----

## 10. What is explicitly deferred

The following are not specified in this document and are designed during federation implementation:

- Specific wire protocol (HTTP/2 vs HTTP/3, message envelope formats, content types)
- Specific endpoint structure for federation APIs
- Specific equivocation defense protocol details (witness coordination, log scope, attestation frequency) — the architectural commitment to public verifiable logs is established in section 7.8; specific protocol design is pending Foundation infrastructure implementation
- Specific message franking composition with federation traffic — the architectural commitment to franking-based verification is established; specific cryptographic composition details are pending cryptographer review
- Rate limiting between instances
- Backpressure and flow control
- Spam handling for federated traffic
- Specific retry and timeout semantics
- Versioning and backward compatibility protocols
- Conflict resolution for edge cases (which shouldn’t exist given single-authority model)
- Performance optimizations (caching, batching, etc.)
- Specific failure detection and recovery
- Monitoring and observability for federation

These are real and important. They’re also implementation concerns, not architectural commitments. Designing them prematurely risks getting them wrong; designing them with implementation experience risks getting them right.

-----

## 11. Tests for federation readiness

These are questions to ask of any pilot implementation decision:

1. Does this data structure work the same way if the entity were on a different instance?
1. Is this identifier globally unique or only unique within this instance?
1. Does this state change go through a signed, persistable message?
1. Could this code path be made to forward to a remote instance with minimal change?
1. Does this assume a single source of truth that wouldn’t exist with multiple instances?
1. Does this rely on synchronous availability of all parties?

If the answer to any of these reveals federation-incompatibility, the decision needs reconsideration before implementation proceeds.

-----

## 12. Relationship to other documents

- **Architecture document:** captures the broader reasoning. Federation readiness commitments are referenced there but documented in detail here.
- **Pilot scope document:** describes what’s built first. Federation is not in pilot scope, but the commitments here constrain how pilot is built.
- **Feature list:** enumerates the full feature set. Federation features appear there as long-term capabilities. The bullet points capturing federation readiness commitments are mirrored in the feature list.

When these documents diverge, this document is authoritative for federation-related architectural commitments. Other documents should be updated to maintain consistency.

-----

*This document defines the architectural commitments that the pilot implementation must honor so that federation can be added without restructuring. It is not a federation protocol specification.*