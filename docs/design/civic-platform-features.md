# Haven Feature List
 
## Long-term vision, implementation-level detail
 
This document enumerates the full feature set of the platform as input for technical decomposition. Features are tagged with the phase in which they ship.
 
-----
 
## Phase plan
 
The platform ships in six phases of increasing scope. Each phase is self-sufficient — a community using a Phase N platform should be able to use it for real civic purposes, with later phases adding value rather than fixing brokenness.
 
### Phase 0: Pilot
 
A cut-down subset of Phase 1 functionality. Specific scope TBD with the pilot community. Goal: validate core architecture with the pilot institution before completing the full single-community MVP. Phase 0 ends when the platform reaches Phase 1 functional completeness.
 
### Phase 1: Single-community MVP
 
Single instance, one community, no federation. Includes basic identity, single community structure, feeds, basic governance (operator-decides), calendar, encrypted messaging, basic digital credentials, in-person QR joining. The platform is functionally complete for one community on one operator-controlled instance.
 
### Phase 2: Internal MVP
 
Multi-community on a single instance. Adds: full alliance support (single-instance), schism mechanism, registries (single-instance), full credential ecosystem (digital), full governance primitives, instance operation features, full moderation primitives, reputation system, templates. The platform becomes useful beyond the pilot for institutions willing to be on the central instance.
 
### Phase 3: External MVP
 
Adds money-touching and physical features: pluggable payment adapters, ticketing with sybil resistance, physical credential production, external integration adapters (good standing tracking, etc.), payment-related reputation signals. Requires compliance work, legal entity setup, and credential production partnerships.
 
### Phase 4: Federation
 
Cross-instance everything: federation protocol implementation, cross-instance alliances, community migration between instances, instance migration, replicas, federation-aware reputation, defederation mechanisms. The architectural commitments laid down in earlier phases become functional.
 
### Phase 5: Mature
 
Plugin system as full external extension framework, issuer-hiding credentials, language packs (i18n plumbing is cross-cutting from Phase 0; specific languages are Phase 5), and a few odds and ends. Many things that might have been Phase 5 features (custom feed types, governance modules, discovery algorithms, integration adapters, bridge implementations) become plugins built against extension points that exist from earlier phases. The platform graduates from internally-pluggable (governance modules, payment adapters, etc., extensible by platform code) to externally-extensible (third parties can extend without modifying platform code).
 
-----
 
## Cross-cutting architectural foundations
 
These architectural commitments apply across all phases. The pilot implementation (Phase 0) and the single-community MVP (Phase 1) must honor them so later phases can be built without restructuring. They are not phase-specific features but constraints on how every phase is built.
 
### CCF-1: Cryptographic identity is the basis of all identity
 
- Users, communities, alliances, and instances are identified by keypairs
- The keypair is the durable identity; any human-readable name or URL is a routing hint that can change
- No server-rooted identity schemes (no `user@server.tld`)
### CCF-2: Instance identity is the alliance, held collectively
 
- Each instance is composed of an operator entity and an instance alliance
- The instance alliance is an MLS group whose members are the communities hosted by the instance
- Alliance authority is exercised through threshold multi-signature (default 2/3 of representative communities)
- The alliance identity is durable across operator changes (operator transitions are alliance actions)
- The operator entity has its own keypair for internal authority (software signing, deployment)
- The operator entity is authorized by the alliance through a signed attestation
- Operations claiming to come from an instance are signed by threshold of alliance composition
### CCF-3: Mailboxes have composite addresses
 
- Mailboxes are identified by `(cryptographic_entity_id, mailbox_role)` pairs
- The cryptographic entity ID identifies whose mailbox it is (community, user, alliance) — durable across instance changes
- The mailbox role identifies which specific mailbox for that entity (feed name, “governance”, “dm-with-X”, stable internal identifier)
- The authoritative instance for a mailbox is a mutable metadata pointer, not part of the address — when a community migrates, the address stays the same and only the pointer changes
- Code processing mailbox addresses works uniformly whether the entity is hosted locally or remotely
- In Phases 0–3, the authoritative instance pointer always refers to the local instance; the abstraction is in place for Phase 4
### CCF-4: State derives from signed message logs
 
- All meaningful state changes flow through signed messages persisted to a log
- Current state is derivable by replaying the log
- The database stores derived state as a cache; the log is the source of truth
- This enables federation (log exchange), migration (log replay), and audit (log inspection) in later phases
### CCF-5: All cryptographic identifiers are globally unique
 
- Cryptographic identities (keypairs) and UUIDs serve as canonical references
- Auto-incrementing integers can exist as internal database optimizations only
- Domain entities are always referenced by their cryptographic or UUID identifier
### CCF-6: Federation will be direct, not transitive
 
- The data model must support cross-instance addressing even in single-instance mode
- An instance entity exists from Phase 0 (it just always points to “us”)
- Federation relationship entity exists in the schema from Phase 0 (empty until Phase 4)
- No transitive routing assumptions anywhere in the code
### CCF-7: Public content is web content
 
- Public feeds, public community pages, and registry listings are served as standard web content
- No federation required to read public content
- This applies from Phase 0; the architecture must not assume private-only content
### CCF-8: Replicas are local copies of remote logs
 
- The platform must accommodate replicas of mailboxes owned by other instances
- Replicas are populated from authoritative logs and held locally for fast read access
- In Phases 0–3, no replicas exist (single instance); the abstraction exists for Phase 4
### CCF-9: No transaction fees, ever
 
- The platform’s payment adapters route money payer-to-recipient directly through processors
- There is no platform-controlled intermediate account where fees could be collected
- This is an architectural property, not a policy commitment
- Any payment-related code must preserve this property
### CCF-10: Operators cannot read community content
 
- End-to-end encryption is structural, not configurable
- The architecture must make content inspection by operators impossible, not merely disallowed
- This applies from Phase 0 even though only basic encrypted messaging ships then
### CCF-11: Users can leave with their data
 
- Users can export their identity, credentials, and accessible content at any time
- Export format is standard and documented
- Import on a new device or different instance preserves identity continuity through cryptographic verification
- This is a structural commitment to user agency; the alternative is users being held hostage to operator decisions
- Applies from Phase 0; full federation-related migration capabilities ship in Phase 4
### CCF-12: Internationalization plumbing exists from day one
 
- All user-visible strings are externalized for translation from Phase 0
- Date, time, number, and currency formatting use locale-aware libraries from Phase 0
- Text direction (LTR/RTL) is handled in the UI layer from Phase 0
- Specific language translations are Phase 5 deliverables; the plumbing they require is not deferrable
- Building this in later means refactoring every UI surface, which is expensive and error-prone
-----
 
## 1. Identity and accounts
 
### 1.1 Cryptographic identity [Phase 1]
 
- Users have a long-term identity keypair generated client-side at account creation
- Identity private keys never leave the user’s devices
- Identity public keys are the canonical user identifier across the platform
- Users can rotate identity keys with continuity preserved via signed succession
- Identity keys can be backed up via user-controlled mechanisms (passphrase-encrypted backup, paper recovery codes, social recovery through trusted contacts)
- Lost identity keys can be replaced via social recovery (multiple trusted contacts attest to the new key being the same person)
### 1.2 Multi-device support [Phase 1]
 
- A user can use the same identity across multiple devices (phone, tablet, laptop)
- Each device has its own device-specific keypair derived from or authorized by the identity key
- Adding a new device requires authorization from an existing device (QR code or short numeric code)
- Removing a device revokes its access without affecting other devices
- Devices sync their state through end-to-end encrypted channels
- Users can see a list of their active devices and revoke any of them
### 1.3 Pseudonymity and verification [Phase 2]
 
- Users participate by default with no verifiable information beyond their identity key
- Users can attach verifiable credentials to their identity to increase trust signals
- Each credential type provides a universal reputation contribution scaled by sybil-resistance value
- Credentials are selectively disclosable (prove “I’m over 18” without revealing birthday)
- Users control which credentials are visible to which communities
- Verification level affects what communities will accept the user but not how they appear within communities they’ve joined
- Users can present different display names and profile information per community
### 1.4 Configurable sybil tolerance [Phase 2]
 
- Any entity that gates an action based on identity confidence configures its sybil tolerance from a gradient
- Very permissive: no identity restrictions; throwaway identities accepted
- Permissive: same cryptographic identity can only perform action once; new identities free to participate
- Moderate: identities must have minimum confidence score (filters brand-new throwaway accounts)
- Strict: identities must have specific verified credentials (phone, email, government ID, etc.)
- Very strict: identities must have institutional credentials (verified community membership, recognized issuer attestation)
- Sybil tolerance is set by the gating entity, not the platform
- Used for: trial periods, community joining, certain actions within communities, alliance membership, etc.
- The platform provides the gradient; entities choose their threshold per use case
- Sybil resistance is probabilistic; tolerance reflects cost-benefit assessment, not absolute prevention
### 1.5 Account lifecycle [Phase 1]
 
- Account creation does not require email, phone, or any external identifier
- Accounts can be deleted, which removes the user from active membership in all communities
- Content the user posted remains in communities per the community’s retention policy
- Accounts can be in a deactivated state (preserved but not active) and reactivated later
- Users can export all their data in a standard format
### 1.7 Profiles and presentation [Phase 1]
 
Each user has profile information used for self-presentation. Profiles are scoped per community — users can present differently in different communities, matching how people naturally adapt their self-presentation to context.
 
**Profile fields:**
 
- Display name (per community)
- Avatar / profile image (per community, see Section 5.6 for media handling)
- Bio or description (per community, optional)
- Pronouns (per community, optional)
- Contact information (per community, optional, visible per community visibility configuration)
**Profile mechanics:**
 
- Users set their initial profile when joining a community
- Profile can be updated at any time via the user’s settings
- Profile changes propagate through the community’s MLS channels
- Profiles are visible to community members per the community’s visibility configuration
- Users can use the same display name and avatar across multiple communities if they choose, or different presentations in each
- No global “user profile” — only per-community profiles
**Default presentation:**
 
- New users get a default avatar (algorithmically generated from their identity key, like an identicon) until they upload one
- Default display name comes from what they set at community join time
- Bio is empty by default
- Users are not required to fill in optional fields
**Public profile aspects:**
 
- When a user’s profile appears in public content (public feed posts, public community member lists if the community publishes them), the profile information shown is public by definition
- Users are notified before posting to public contexts that their profile information will be visible externally
- Communities can configure which profile fields appear in their public contexts (e.g., display name and avatar but not contact info)
**Cross-community considerations:**
 
- Users uploading the same avatar to multiple communities upload it separately to each (no deduplication)
- Each community’s avatar is encrypted with that community’s keys and stored independently
- See Section 19.6 for the operator-correlation properties of profile media
-----
 
## 2. Communities
 
### 2.1 Community structure [Phase 1]
 
- A community has a cryptographic identity (keypair) separate from any individual member’s
- Communities have a name and description visible to members
- Communities have a configurable membership cap (or no cap)
- Communities have a home instance but can migrate to other instances
- Communities have a creation date and member list (members can see other members)
- Communities can be marked as public (discoverable) or private (invitation-only at discovery level)
### 2.2 Community creation [Phase 1]
 
- Any user can create a new community
- Community creation prompts for: name, description, founding template selection
- Templates are composable — see Section 16 — drawn from governance templates, role templates, feed templates, and decision procedure templates
- The founder is automatically the first member with the highest administrative role
- The community begins in a “founding” state where the founder can configure structure without governance
- After the community has more than a configurable number of members, governance is required for structural changes
### 2.3 Community membership [Phase 1]
 
- Joining a community requires QR code or NFC exchange with an existing member, in-person
- The inviting member is recorded as the voucher
- Membership applications can require approval by community governance depending on configuration
- Members can leave a community at will
- Members can be removed by appropriate governance action
- Members can be temporarily suspended (no posting/reading rights) without full removal
- Membership lists are visible to members but not to instance operators
### 2.4 Community configuration [Phase 1]
 
- Communities have configurable settings for: name, description, default retention period, schism thresholds, governance procedures, default feed configuration
- Configuration changes go through community governance (after the founding state)
- Configuration is versioned with history available to members
- Some configuration is locked from creation (e.g., the existence of a governance feed)
### 2.5 Community archival and dissolution [Phase 2]
 
- Communities can be archived by governance decision (preserved but inactive)
- Communities can be fully dissolved by governance decision with extra care
- Dissolution preserves content per retention policy and notifies members
- Members of dissolved communities are notified and can export their participation history
-----
 
## 3. Roles and permissions
 
### 3.1 Role system [Phase 1]
 
- Roles are community-defined; no fixed set imposed by the platform
- Roles have names, descriptions, and visual indicators (color, icon, badge)
- A community can define any number of roles
- A member can hold multiple roles simultaneously
- Roles can be hierarchical (some roles imply other roles) or flat
- Roles can have term limits, expirations, or be permanent
### 3.2 Role assignment mechanisms [Phase 1]
 
- Manual assignment: members with the appropriate permission assign roles to other members
- Election-based assignment: a vote determines who holds the role
- Sortition-based assignment: random selection from an eligible pool
- Self-assignment: members can claim the role if they meet criteria
- Inheritance: holding role X automatically grants role Y
- Credential-based: holding a specific credential grants a role automatically
- Each role has a configured assignment mechanism
- Assignment mechanisms can change through governance
### 3.3 Permission system [Phase 1]
 
- Permissions are bindings between roles and actions
- Action categories: post, read, moderate, configure, invite, govern, manage members, manage roles, create feeds, retire feeds, issue credentials, manage moderation rules
- Each action has a specified scope: community-wide, per-feed, per-object
- Permissions combine additively when a member holds multiple roles
- Default permissions exist for new communities via template
- Permissions can be changed through governance
### 3.4 Special roles [Phase 1]
 
- “Founder” role exists at community creation and has full permissions; can be relinquished or transferred
- “Admin” or equivalent high-permission role exists by default
- “Member” role exists by default with basic posting and reading rights
- Communities can rename or restructure these defaults
### 3.5 Guardian role [Phase 2]
 
A platform-level primitive supporting youth participation and other oversight-requiring contexts. Guardians have specific authority over communities they oversee without surveilling the content.
 
- Guardian role linked to specific members through verified relationship (institutional credential for schools/religious communities; vouching for informal groups)
- Guardians have voting rights on community-level decisions configured by the community
- Guardians receive notifications about specific events (new member additions, financial transactions, policy changes, safety alerts)
- Guardians cannot read message content, posts, or specific interactions
- Guardians can see aggregate community health information (general activity level, member count)
- Guardian authority scales with member age by community configuration:
  - Under 13: Full guardian authority (COPPA compliance requirement)
  - 13-15: Reduced authority by default; community can configure
  - 16-17: Minimal authority by default; mostly notification-based
  - 18+: No guardian role unless community specifically retains it for disability accommodation or similar
- Safety reports (CSAM, immediate harm) reach guardians regardless of community configuration
- Specific guardian-role configurations available as templates (school, friend group, religious education, homeschool co-op)
- COPPA-compliance mechanisms (verifiable parental consent, parental data access, retention limits) are platform-level for under-13 users
### 3.6 Role visibility [Phase 2]
 
- Roles can be visible to all community members or restricted to specific contexts
- Members can see who holds which roles (with respect to visibility settings)
- Role history is preserved (who held what role when)
-----
 
## 4. Feeds
 
### 4.1 Feed structure [Phase 1]
 
- A feed is a named, configured communication context within a community
- A community has multiple feeds
- Each feed has its own configuration independent of other feeds
- Feeds have an associated mailbox UUID for routing
- Feeds have an associated public key set for signature verification
- Feeds can have associated calendars
### 4.2 Feed types [Phase 1]
 
- General discussion feed: text-based conversation
- Announcement feed: one-to-many communication, typically restricted write permissions
- Event feed: structured events with date, time, location, RSVP
- Decision feed: governance proposals and voting
- Media feed: optimized for image and video sharing
- Working group feed: persistent feed for a subgroup
- Pastoral care feed: extra-private, restricted membership within the community
- Custom feed types via configuration
### 4.3 Feed configuration [Phase 1]
 
- Name and description
- Content type (determines available content formats and UI treatment)
- Visibility: who in the community can see the feed, or whether public
- Write permissions: who can post to the feed
- Retention policy: how long content persists
- Moderation permissions and moderation model: who moderates and how (see Section 14)
- Notification defaults: how new content notifies members
- Delivery cadence: real-time, or batched digest on a schedule (e.g., once daily). Slow cadence is a deliberate calming mechanism — a flame war is arduous at one delivery per day — and suits large or deliberative feeds especially
- Post length bounds: optional minimum and maximum per accepted content type
- Pinning status: whether this feed is prominently displayed
- Indelibility: whether posts can be edited or deleted
- Verification: whether posts are signed for external verification
### 4.4 Feed lifecycle [Phase 1]
 
- Persistent feeds are created via governance proposal and vote
- Ephemeral feeds (event feeds, etc.) are auto-created by structured activities
- Feeds can be retired (no new posts, content preserved)
- Retired feeds can be reactivated by governance
- Feeds can be fully deleted via governance with content handled per retention policy
- Auto-created event feeds transition through active → post-event grace period → archived
### 4.5 Feed discovery within community [Phase 1]
 
- Members see pinned feeds prominently
- Members can browse all feeds they have access to
- Members can filter feeds by type, activity level, recent activity
- Members can mute or hide feeds without leaving them
- Members are auto-subscribed to default feeds when joining; can opt out of optional feeds
### 4.6 Cross-feed operations [Phase 2]
 
- Cross-posting (post to multiple feeds simultaneously with appropriate permissions)
- Cross-feed search within a community
- Cross-feed calendar aggregation
- Cross-feed mentions and references
### 4.7 Public feeds [Phase 2]
 
- Communities can configure feeds as publicly readable (no community membership required to read)
- Public feeds use MLS for publisher access control just like private feeds — “public” is a property of external publication, not a different cryptographic mechanism
- Publisher MLS group members are individuals with the publication role; standard MLS Add/Remove proposals manage publishers
- Public feed posts are signed by the publisher’s identity key with proof of MLS group membership at posting time
- Public feeds are discoverable through alliances/registries
- Public feeds are served as standard web pages with stable URLs
- Public feed content is indexable by search engine crawlers
- See Section 17 for public content indexability and link metadata
- See Section 19 for operator-correlation properties of public feeds
-----
 
## 5. Content
 
### 5.1 Content types [Phase 1]
 
The platform uses typed content schemas rather than loosely-typed content. Each content type has a declared schema with typed fields, validation rules, and versioning. Core content types include:
 
- Text messages (markdown or rich text subset)
- Images (with size limits and format support)
- Video (with size limits and format support — pre-recorded, not live)
- Voice messages (audio recordings)
- Files (with size limits and type restrictions)
- Events (structured with date, time, location, RSVP)
- Polls (within governance feeds)
- Offers (structured: what’s offered, when, where, how to claim)
- Requests (structured: what’s needed, when, where, how to respond)
- Announcements (structured: title, body, expiration)
The typed schema approach uses JSON Schema (draft 2020-12) with Haven-specific conventions on top. Rationale: JSON Schema has massive ecosystem support across Haven’s implementation stack (Elixir, React Native, Rust), well-established composition features (allOf, oneOf, anyOf, $ref, conditional validation), and mature versioning patterns. AT Protocol lexicons were considered and rejected — lexicons provide API surface description that AT Protocol needs because of its large schema-driven federation API, but Haven’s federation API is more bounded and operations can be described separately from data schemas.
 
This enables:
 
- Communities with very different content needs (parishes, HOAs, artist collectives, schools) to have content matched to their actual work rather than forced into one generic shape
- Plugins to introduce new content types through schema declarations
- Federation correctness through explicit type identifiers
- Validation at platform boundaries rather than best-effort interpretation
Haven-specific conventions layered on JSON Schema:
 
- Content addressing reference types (custom format/pattern indicating commitment references — franking commitments, MLS epoch authenticators, log tree heads — verified at protocol layer)
- Schema identifier scheme: reverse-domain notation (e.g., `app.haven.events.standard.v1`)
- Schema registry/distribution: Foundation operates baseline registry for canonical schemas; community-level distribution for custom schemas; inline embedding for one-off cases
- Version handling: major versions in identifiers; minor versions through additive changes; explicit migration paths for breaking changes
- Interface satisfaction declarations (see Section 5.7)
Schema authority distributes across levels: Foundation maintains baseline schemas; communities can introduce community-specific schemas; plugin authors declare schemas for plugin-introduced types. Schemas version explicitly so they can evolve without breaking existing content.
 
Schema registry follows the same model as verifiable logs: Foundation operates baseline registry for canonical schemas, certification is trust signaling, anyone can operate registries, schemas can be distributed through any mechanism. Open operation, distributed responsibility.
 
### 5.2 Content operations [Phase 1]
 
- Post content to a feed
- Edit own content (where feed permits)
- Delete own content (where feed permits)
- Moderate others’ content (where permission granted)
- React to content with emoji
- Reply to content (threaded conversation)
- Mention specific members in content
- Cite or reference other content (links to specific posts)
- Mark content as read/unread
- Save or bookmark content for later reference
### 5.3 Reactions [Phase 1]
 
- Members can react to content with a predefined set of emoji
- Reactions are encrypted; reaction counts may be visible at aggregate level
- Reactions are attributable to the reacting member within the encrypted layer
- Reactions can be removed
### 5.4 Content lifecycle [Phase 2]
 
- Content states: active, retained, deleted
- Active content appears in normal feed views
- Retained content is held in the institutional archive (see Section 5.9) for feeds configured with that artifact; access is audited and threshold-gated
- Deleted content is removed from server storage
- Transitions are governed by community retention policy
- Content can be explicitly marked for institutional archive or deleted by appropriate permission holders
- Members can save content client-side before server-side deletion
### 5.5 Search [Phase 2]
 
- Client-side search within a feed
- Client-side search across feeds within a community
- Search across archived content (slower)
- Search by content type, date range, author, reaction type
- Public feed content searchable by external search engines
### 5.6 Media handling [Phase 1]
 
Media (images, video, audio, files, etc.) is encrypted per community and stored independently, with cryptographic ownership verification.
 
**Encryption and storage:**
 
- Media uploaded to a community is encrypted with that community’s keys
- Encrypted media is stored in S3-compatible object storage
- Different communities encrypt with different keys; the same source content uploaded to multiple communities produces different ciphertext per community
- No deduplication across communities; each upload is independent
**Ownership verification:**
 
- The uploading user signs the hash of the ciphertext (not the plaintext)
- The signature covers: ciphertext hash, community ID, upload timestamp, attachment context
- Different communities produce different ciphertext and therefore different signatures, naturally
- Verifiers download the encrypted media, compute hash, verify the uploader’s signature, then decrypt with the community key
- The signature proves the user uploaded this specific encrypted blob to this specific community
**Media types supported:**
 
- Images (PNG, JPEG, WebP, etc.)
- Video (MP4, WebM)
- Audio (MP3, AAC, Opus)
- Documents (PDF, plain text)
- Generic files (with size limits configurable per community)
**Reference from posts:**
 
- Posts reference media by Media entity ID
- Multiple posts within a community can reference the same Media entity (no duplication within a community)
- Cross-community references are not possible (different encryption keys)
**Lifecycle:**
 
- Media is retained as long as any post references it
- When all references are removed and retention policy allows, media is garbage collected
- Media is deleted when the community is deleted
**Operator visibility:**
 
- Operator sees encrypted media blobs in storage
- Operator sees upload patterns (timing, size, frequency) but not content
- Same source content in multiple communities produces unrelated ciphertext from the operator’s perspective; access pattern correlation can still defeat this (see Section 19.6)
### 5.7 Interface-based composition for client rendering [Phase 1]
 
Clients cannot be expected to implement every possible schema; the schema ecosystem will grow over time. The interface system enables old clients to handle new schemas gracefully without being updated, while preserving rich rendering when the client does support a schema.
 
**Schemas declare which interfaces they satisfy.** Interfaces are contracts about field presence and semantics, defined separately from specific schemas. A schema can claim satisfaction of multiple interfaces.
 
**Clients implement against interfaces, not against specific schemas.** A client that implements `Renderable` can display anything claiming to satisfy `Renderable`. A client that implements `MediaContainer` can show media for any schema claiming that capability.
 
**Canonical interfaces are Foundation-governed** and form the standard set. Many small interfaces, granular and composable. Initial canonical interfaces:
 
- `Renderable`: title, author, timestamp, body
- `Discussable`: Renderable plus discussion thread
- `MediaContainer`: Renderable plus media references (audio, video, image)
- `Geographic`: location field with structured geographic data
- `Timed`: start time and optional end time
- `Reactable`: supports reactions/responses
A schema like `app.haven.parish.homily.v1` might claim `Renderable + Discussable + MediaContainer`. A community event claims `Renderable + Geographic + Timed`. The combinations express what the content supports.
 
**Interfaces are themselves JSON Schemas.** An interface specifies required fields and types. A schema satisfies an interface if it includes everything the interface requires.
 
**Schema declarations include interface satisfaction claims:**
 
```json
{
  "$id": "app.haven.parish.homily.v1",
  "title": "Parish Homily",
  "satisfies": ["Renderable", "Discussable", "MediaContainer"],
  "fallbackDisplayType": "app.haven.discussion.v1",
  "properties": { ... }
}
```
 
**Fallback type field for completely unknown cases:** schemas declare a fallback type for clients that don’t implement any relevant interfaces. The fallback provides graceful degradation even when interface-based rendering fails.
 
**Validation:** when a schema is registered or used, validators check the schema’s structure (standard JSON Schema validation) and verify the schema actually satisfies each interface it claims (each interface’s required fields are present). Schemas claiming interfaces they don’t actually satisfy are rejected.
 
**Client rendering flow:**
 
1. Client receives content with schema identifier
1. Client looks up schema (from registry or inline definition)
1. Client identifies which interfaces the schema claims
1. Client renders using the most capable interface combination it supports
1. If no interfaces match, fall back to fallback type rendering
1. If fallback type also unknown, render as basic Renderable (the universal baseline)
**Interface versioning:** interfaces have versions like schemas. Schemas claim satisfaction of specific versions. Additive changes only for minor versions; breaking changes require new major version. Foundation Alliance governance for interface library changes.
 
**Community-defined interfaces** are allowed but clients aren’t expected to implement them — communities defining custom interfaces take responsibility for limited client support.
 
### 5.8 Feed structure declaration [Phase 2]
 
Communities declare their feed structure as part of community state managed through governance. A feed structure declaration includes:
 
- Feed identification: name used in partition key derivation, stable internal identifier for rename handling
- Role-based permissions: which roles can read, which can write, what other permissions apply
- Content type configuration: which schemas this feed accepts (see Section 5.1)
- History configuration: whether the feed has an onboarding context window, an institutional archive, both, or neither (see Section 5.9)
- Optional relationships to other feeds (for cross-feed patterns; the protocol supports declaring relationships but doesn’t specify what they mean)
**Feeds defined by allowed content types:** a feed’s configuration includes which content schemas it accepts. A community can have specialized feeds tuned for specific content types — a short-form video feed accepting only video records under N minutes, a microblogging feed accepting text plus images but not video, a photo feed accepting images only. The feed’s identity is shaped by what it accepts.
 
**Rationale:** organic differentiation within communities lets them compete with format-focused commercial platforms (TikTok-style, Twitter-style, Pinterest-style) without users needing to leave civic infrastructure. Different content types can have different moderation policies appropriate to their format. Clean separation of concerns within communities — a parish can have announcements (text), service photos (images), event videos (short video), and chat (text plus light media) all separately configured.
 
**Schema references support both identifier and inline definition:**
 
- **String reference:** the feed declares acceptance of a schema by canonical identifier. Posters and verifiers look up the schema in a registry. Efficient for common schemas.
- **Inline definition:** the feed includes the full schema definition. No external lookup. Useful for community-specific custom schemas.
Both patterns are supported. Standard schemas use string references; custom schemas can be defined inline or registered for reuse.
 
### 5.9 History artifacts: onboarding context and institutional archive [Phase 2]
 
Member-facing history is supported through two independent per-feed artifacts with opposite default trust postures. They cannot be collapsed into one artifact with two policies because that would either force institutional-grade controls onto routine onboarding or make the institutional archive inherit onboarding’s looseness.
 
**Onboarding context** (optional, per feed):
 
- Purpose: a new member isn’t dropped in with zero context
- TTL: short rolling window (last week / last N posts, configurable per feed)
- Self-limiting — content ages out automatically
- Access bar: low; this is what every new member is handed; read on join is routine
- Scope: open-read feeds only; never on confidential sub-groups
- FS posture: time-bounded, self-healing waiver — a joiner sees the configured window; later, that content is gone from the onboarding store
- Implementation: closer to a retention setting on the live feed than a separate cryptographic store; does not need the institutional archive’s hard primitive
**Institutional archive** (optional, per feed):
 
- Purpose: the system of record — retention, recovery, audit
- TTL: long to indefinite, per community governance
- Access bar: high threshold; every access audited; read is exceptional rather than routine
- Scope: anything retained, including confidential sub-groups (sub-group archive is gated by sub-group membership, not by parent community membership)
- FS posture: bounded, governed FS sacrifice — content continuously re-encrypted under a long-lived archive key as it’s posted
- Implementation: archive key’s decryption authority is threshold-shared among an archivist role; every archive access logged to verifiable log infrastructure; gating without auditing is rejected (lets a colluding threshold quietly surveil)
**The four combinations:**
 
|Feed shape        |Onboarding context|Institutional archive|Example                                                                                 |
|------------------|:----------------:|:-------------------:|----------------------------------------------------------------------------------------|
|Neither           |—                 |—                    |High-privacy chat: pure FS, no history for anyone                                       |
|Onboarding only   |✓                 |—                    |Announcements: newcomers get recent context; nothing retained long-term                 |
|Institutional only|—                 |✓                    |Treasury sub-group: full audited record, but joining doesn’t grant a recent-history dump|
|Both              |✓                 |✓                    |Parish main feed: welcome mat plus system of record                                     |
 
The institutional-only case demonstrates why the artifacts must be separate: joining the treasury committee grants institutional-archive *eligibility* (you can request audited historical access) but not an onboarding dump of recent treasury talk. Different artifact, different trigger, different bar.
 
**History status is disclosed.** Whether a feed retains content (and under which artifact) is a governed property visible to anyone posting to the feed. Posters need correct expectations about whether their content will outlive the epoch and be shown to people who weren’t present when it was written.
 
**Confidential sub-group history.** Sub-group pre-join history is gated by sub-group membership, not by parent community membership. A new community member does not automatically gain access to sub-group history; sub-group archive is managed under the sub-group’s own history artifacts. See Section 13.8 for sub-group mechanics.
 
-----
 
## 6. Events and calendars
 
### 6.1 Event structure [Phase 1]
 
- Events have: title, description, start time, end time, location, organizer, attendee list, RSVP status per attendee
- Events can be all-day or time-bounded
- Events can have recurrence (weekly, monthly, custom patterns via RRULE)
- Events have an associated auto-created feed for that specific event instance
- Events can be created from a template
### 6.2 Event operations [Phase 1]
 
- Create event (with permission)
- Edit event (creator or appropriate role)
- Cancel event (notification sent to attendees)
- Postpone or reschedule event
- RSVP to event (yes, no, maybe)
- Mark attendance at event (self-mark or by event organizer)
- Comment on event in its associated feed
- Invite specific members to event (in addition to feed-level access)
### 6.3 Calendars [Phase 1]
 
- Each feed can have an associated calendar
- Calendars aggregate events from feeds (per-feed, per-community, per-member, cross-community)
- Members have a personal calendar view (events they’re invited to or RSVP’d to)
- Communities have a community calendar (all events members can see)
- Calendar conflict detection within member’s view
- Calendar exports as iCal (subscribable feed URL with authentication)
### 6.4 Recurring events [Phase 1]
 
- Recurrence rules: daily, weekly, monthly, yearly with patterns (every Tuesday, first Sunday of the month, etc.)
- Each instance of a recurring event is its own event with its own feed
- The recurring event series can be edited (affects all future instances)
- Individual instances can be edited (affects only that instance)
- Individual instances can be cancelled without cancelling the series
### 6.5 Attendance and credentialing [Phase 1]
 
- Attendance can be tracked per event
- Attendance generates credentials within the community (member attended event X)
- Attendance can feed reputation/standing
- Attendance reports available to organizers and appropriate roles
- Members can see their own attendance history
### 6.6 Ticketed events [Phase 3]
 
- Events can have ticket configuration (count, price, sybil tolerance, transfer policy, per-person cap)
- Tickets are platform-native (see Section 27.14)
- Payment for tickets uses standard payment adapters
- Tickets are anonymous credentials with per-event nullifiers — the same BBS per-verifier-pseudonym primitive used for voting, scoped to event identifier instead of decision identifier
- Sybil tolerance configures which `pid` anchors the per-event nullifier: account `pid` (permissive “one per identity”), phone-verified `pid` (stronger), decency `pid` (“one per member”), humanness `pid` (strict “one per person”)
- Per-person caps (one ticket per person, N tickets per person) are enforced by application-layer counting per event — the ticketing authority keeps a counter keyed by per-event pseudonym and rejects past the cap; voting (cap 1) and ticketing (cap N) are the same construction
- Free ticketed events (capped RSVP with sybil resistance) use the same mechanism without payment
- Check-in at the event verifies the ticket credential
-----
 
## 7. Governance
 
### 7.0 Basic governance [Phase 1]
 
Phase 1 governance is template-based with sensible defaults. Communities new to platform-based governance need starter configurations that approximate familiar patterns. They can adjust later (or in Phase 2 when full primitives ship), but they shouldn’t have to design governance from scratch on day one.
 
**Starter templates ship with Phase 1:**
 
- **Founder-led:** founder is admin with full authority; other members are participants. Approximates traditional ownership/founder-control structures. Default for new communities.
- **Council:** a designated council of N members (founder selects initially) makes decisions collectively; other members are participants. Approximates board/elder/council models common in religious and civic institutions.
- **Member-vote on major decisions:** founder/admin handles day-to-day; specific decision types (membership changes, configuration changes, dissolution) require member vote. Approximates how many small institutions actually operate.
- **Consensus:** all members must accept significant decisions; no formal vote, but objections block. Approximates Quaker meeting practice and similar consensus traditions.
**Phase 1 governance primitives:**
 
- Founder/admin role with full authority
- Designated decision-maker roles (council members, etc.) per template
- Simple binary decisions (yes/no on specific actions)
- Member assent or objection (for templates that require it)
- Audit log of significant decisions
- Template selection at community creation
- Template can be changed by governance action per the current template’s rules
**Not in Phase 1:**
 
- Formal proposal/vote/decision machinery with deliberation periods (Phase 2)
- Multiple ballot types (Phase 2)
- Custom tally methods (Phase 2)
- Configurable thresholds beyond template defaults (Phase 2)
- Term limits and elections (Phase 2)
The intent is to give pilot communities working governance from day one without requiring them to invent it. They get a sensible default and can adjust within the template’s bounds. Phase 2 adds the full primitive set that lets communities design custom governance structures.
 
### 7.1 Proposals [Phase 2]
 
- Members with appropriate permission can create proposals
- Proposals have: title, description, type, target object (if applicable), proposed change, voting period, threshold, ballot type, tally method
- Proposals are posted in a governance feed
- Proposals enter a deliberation period before voting
- Proposals can be amended during deliberation by the proposer or with appropriate permission
- Proposals can be withdrawn by the proposer before voting begins
### 7.2 Ballot types [Phase 2]
 
- Yes/no/abstain (binary decisions)
- Ranked choice (rank options by preference)
- Approval voting (select all acceptable options)
- Score voting (rate each option on a scale)
- Range voting (similar to score, with different aggregation)
- Consensus signaling (object, accept, stand aside)
- Custom ballot types can be added through governance
### 7.3 Tally methods [Phase 2]
 
- Simple majority (more than half of cast votes)
- Supermajority (configurable threshold, e.g., 2/3)
- Instant runoff (for ranked choice ballots)
- Condorcet methods (for ranked choice ballots)
- Approval threshold (proposal passes if approval percentage above X)
- Score average (for score/range ballots)
- Consensus (no formal objections, with timeout fallback)
- Unanimous (requires all eligible voters)
- Quorum requirements applied across all methods
### 7.4 Voting eligibility and vote weight [Phase 2]
 
- Eligible voters defined by role (any role can be specified as a voting role)
- Vote weight configurable (one-person-one-vote default, weighted vote possible)
- Voting periods configurable per proposal type
- Vote results visible after voting period ends
- Public votes are signed and attributable (the baseline; matches Haven’s attributable-by-default governance)
- Anonymous votes use per-decision pseudonyms derived from humanness credential — the platform can verify “eligible voter, one vote per person” without learning which voter; eligibility-linking proves “member in good standing AND unique human who hasn’t voted” via shared-`pid` ZK equality between the decency credential and the humanness nullifier
- One-person-one-vote anchors the per-decision nullifier on the humanness `pid` specifically; a persona-anchored nullifier would let a human with multiple personas double-vote
- Cardinality enforcement (one vote per person) is application-layer counting per decision rather than range proofs — the single tally authority per decision keeps a counter keyed by per-decision pseudonym
- Vote-content-secrecy (hiding which choice was selected, beyond just hiding who) is a Phase 6 late protocol addition for coercion-resistance use cases; not part of baseline because Haven’s attributable-by-default governance rarely needs it; addable as a per-vote configuration option without disturbing identity/pseudonym/credential machinery
### 7.5 Proposal types [Phase 2]
 
- Configuration changes (community settings, feed configuration)
- Feed lifecycle (create, modify, retire feeds)
- Membership decisions (admit, suspend, remove members)
- Role changes (create roles, assign roles, modify permissions)
- Governance changes (modify the governance procedures themselves)
- Alliance decisions (join alliance, leave alliance, propose changes to allied governance)
- Migration decisions (move community to different instance)
- Schism eligibility flagging
- Resource decisions (dues, treasury, expenses)
- Moderation rule decisions (adopt, modify, or repeal community moderation rules)
- Custom proposal types via configuration
- The configured proposal-type set is governance configuration subject to the asymmetric change thresholds in Section 7.7
### 7.6 Decision enactment [Phase 2]
 
- Passed decisions are automatically applied by the system where possible
- Some decisions require human follow-through (manual coordination)
- Decision history is preserved indefinitely as community record
- Failed proposals are also recorded with vote tallies
- Members can review historical decisions and how each member voted (for public votes)
### 7.7 Meta-governance [Phase 2]
 
- Rules for changing governance rules
- Governance changes can require higher thresholds than routine decisions
- Governance changes can require waiting periods before enactment
- Governance changes are themselves schism-eligible
- **Asymmetric thresholds for friction changes.** The configured set of proposal types and the forms they require (Section 7.5) is itself governance configuration, and it cannot be a flat configurable list. Changes that lower friction — moving decision classes from the full proposal structure to simple motions, lowering qualifying thresholds, shortening grace periods — require meta-governance thresholds (higher quorum, longer grace period, optionally schism eligibility) even when the change is toward ease. Without the asymmetry, one routine motion can reconfigure everything to routine motions, dismantling the guardrails through the gap they were guarding. Raising friction proceeds at ordinary thresholds; removing it cannot. Templates ship with this asymmetry preconfigured.
### 7.8 Temporal governance mechanisms [Phase 2]
 
Three distinct temporal patterns for voting that communities can configure. These address different problems and can be combined.
 
**Grace periods.** A configurable minimum time between proposal posting and earliest commit, regardless of when threshold is reached. Prevents rushed decisions where a few online members push through significant changes before others can see and respond.
 
- Grace period starts at proposal posting
- Votes can be cast and changed during the grace period
- Final tally is at the end of the grace period (or when configurable later cutoff is reached)
- Action does not execute until grace period elapses, even if threshold is reached earlier
- Default grace periods scale with proposal type: short for routine decisions (hours), longer for substantive changes (days), longest for constitutional changes (weeks)
- Communities configure their grace periods within governance bounds
**Quorum requirements.** A configurable minimum participation level for a vote to be valid. Prevents small subsets of engaged members from making decisions that affect the broader community without meaningful participation.
 
- Can be defined as absolute number, percentage of eligible voters, or percentage of recently-active members
- “Recently-active” is verifiable through the event log (when members last interacted with the community)
- Communities configure their quorum requirements based on their scale and engagement patterns
- Small communities may require near-universal participation; large communities require active subset
- Failed quorum results in proposal failure, with the proposal eligible for resubmission
**Voting windows.** Predictable periods when voting happens, with proposals accumulating between windows. Matches how many institutional communities actually operate — synods, annual meetings, quarterly board sessions, legislative cycles.
 
- Communities can configure proposal submission to be continuous or windowed
- Voting happens during specified windows (annual week, quarterly window, monthly day, etc.)
- Proposals submitted between voting windows queue until the next window
- Members know when to pay attention; doesn’t require constant engagement
- Particularly suited for larger communities and formal institutional governance
**Combinations.** These patterns can be combined. A community might use voting windows (proposals voted on quarterly) with grace periods within the window (proposals must be visible for X days before voting closes) and quorum requirements (Y% of active members must participate for valid result).
 
**Emergency provisions.** Communities can configure emergency mechanisms with their own constraints — higher thresholds, designated emergency roles, mandatory audit, time-limited authority. Emergency actions bypass normal temporal mechanisms but are recorded and reviewable. Use cases: safety threats, time-sensitive opportunities, technical incidents requiring rapid response.
 
**Recurring proposals.** Communities can configure proposals that recur on schedule (annual budgets, periodic elections, membership renewals) without requiring resubmission each cycle. The proposal exists as a recurring agenda item; voting happens on schedule.
 
**Per-type configuration.** Different proposal types can have different temporal mechanisms. A community might use simple grace period for routine moderation decisions, full voting windows for budget approval, and emergency procedures for safety actions. Communities define the categories and their rules.
 
### 7.9 Moderation rules [Phase 2]
 
- Communities can adopt explicit moderation rules through governance
- Moderation rules are human-readable statements (not programmatic enforcement)
- Rules are referenced by moderators when taking action
- Rules can be amended or repealed through the same governance process
- Rule history is preserved
- Members can read all current and historical rules
-----
 
## 8. Schism
 
### 8.1 Schism mechanism [Phase 2]
 
- Certain proposals can be flagged as schism-eligible
- During voting, dissenting voters can check “schism on loss” option
- If schism declarations meet community’s threshold and the proposal passes, schism window opens
- During the window, members can declare or withdraw schism intent
- At window close, new community is created with schismers as founders
### 8.2 Schism inheritance [Phase 2]
 
- New community inherits content history (cryptographically accessible to its members)
- New community inherits credentials previously issued
- New community inherits alliance memberships by default — both communities are members of the alliances the original belonged to; alliances can eject the schismatic community through their own governance if they don’t want both, but the default favors continuity
- This default matters because alliances often reflect deep institutional relationships (denominational membership, sister-organization status) that a schism shouldn’t unilaterally end; alliances retain authority to decide whether to maintain both relationships through their normal governance processes
- New community inherits federation links
- New community inherits credential issuer trust settings
- New community inherits per-member behavioral reputation
- New community starts with cryptographically distinct identity
### 8.3 Schism naming [Phase 2]
 
- Majority faction keeps the original community name
- Schismatic community starts with a placeholder name
- Schismatic community can rename via its own governance
- Alliances and discovery handle name disambiguation if needed
### 8.4 Schism governance [Phase 2]
 
- Schism thresholds configurable per community
- Schism eligibility configurable per community
- Schism is itself a governance operation with its own rules
- Hostile schism attempts (factions trying to take over) are mitigated by member-continuity heuristics
-----
 
## 9. Federation
 
### 9.1 Instance identity [Phase 4]
 
- Each instance has its own cryptographic keypair as its durable identity
- Instance keypair authenticates the instance in all federation contexts
- Instance URL is a routing hint that can change; the keypair is permanent
- An instance changing URLs but keeping its keypair is the same instance
- Operations claiming to come from an instance are signed by the instance keypair
### 9.2 Mailbox addressing [Phase 4]
 
- Mailboxes are addressed as composite `(cryptographic_entity_id, mailbox_role)` pairs
- The cryptographic entity ID identifies whose mailbox it is (community, user, alliance) — durable across instance changes
- The mailbox role identifies which specific mailbox for that entity (feed name, “governance”, “dm-with-X”, stable internal identifier)
- The authoritative instance is metadata (a mutable pointer), not part of the address — community migration keeps the address stable; only the pointer changes
- All code that processes mailbox addresses respects this model uniformly
- Local routing happens when the authoritative instance pointer is the current instance; remote routing forwards to the authoritative instance
### 9.3 Direct federation only [Phase 4]
 
- An instance routes messages only to instances it has direct federation with
- No transitive routing through intermediate instances
- If A federates with B and B federates with C, A does not automatically reach C through B
- Hierarchical structures (alliance of alliances) work because each level’s hub instance is directly federated with relevant peers
### 9.4 Federation establishment triggers [Phase 4]
 
- Federation between two instances establishes when shared social structure crosses them
- Triggers: user joining cross-instance community, alliance forming with members on multiple instances, community migrating between instances
- Federation does not form abstractly between instance operators
- Federation does not form through discovery directories alone
- Federation is always rooted in real social or organizational connection
### 9.5 Public content access [Phase 4]
 
- Public content (public feeds, public community pages, registry listings) is served as standard web content
- Any user agent can fetch public content from any instance without federation
- Discovery through registries is browsing public content
- Federation matters only for private operations (joining, posting, governance participation)
### 9.6 State derived from message logs [Phase 4]
 
- All meaningful state changes flow through signed messages persisted to a log
- Current state is derivable from the log
- Database stores derived state as a cache; the log is the source of truth
- Migration is replaying logs in a new location
- Federation message exchange is log exchange
### 9.7 Replication [Phase 4]
 
The platform supports three tiers of replication for federated content, plus user-side backup as a portability fallback. Different tiers exist for different trust relationships and storage tradeoffs.
 
**Tier 1: Authoritative.** Exactly one instance per mailbox holds the authoritative copy. This is the source of truth. Writes always go to the authoritative instance; reads can happen from any tier.
 
**Tier 2: Allied full replicas.** Specific instance alliances commit to holding full copies of federated communities’ history. Established through explicit agreement between instance alliances (not between communities directly — communities don’t own infrastructure, instance alliances do). These are the primary protection against operator hostility — a community whose authoritative instance goes bad can migrate to an allied instance with full history intact. Allied replicas have no automatic decay; they are deliberate, durable commitments.
 
**Tier 3: Cache replicas.** Instances automatically maintain partial replicas of remote mailboxes that local members or alliance memberships connect to. Subset of recent content according to configurable policy. Decay automatically based on TTL and storage limits. Cleaned up when local interest ends, with configurable grace periods. These provide read locality, partition tolerance, and convenience — not durable protection.
 
**User-side backup.** Users can export their accessible content periodically (daily, weekly, configurable). Provides point-in-time portability and lossy recovery option for communities that don’t have natural instance allies. See CCF-11.
 
### 9.7.1 Per-content-type replication policies [Phase 4]
 
- **Membership state** (small, critical): full replication wherever the community has federated participation; not subject to cache decay
- **Governance state** (small, critical): same as membership state
- **Recent message logs**: full replication in cache replicas during active participation period
- **Older message logs**: subject to cache decay per configuration; full retention in allied replicas
- **Media** (large, expensive): content-addressed within per-community encryption boundaries to enable deduplication where possible; lazy-fetched in cache replicas; full retention in allied replicas
### 9.7.2 Allied replica agreements [Phase 4]
 
- Established between instance alliances (the governance bodies of instances), not directly between communities
- Communities express preferences for allied replication; their instance alliances negotiate the actual agreements
- Agreement terms are explicit: which communities, what content, what duration, what reciprocity or compensation
- Instance alliances can refuse to host allied replicas if storage cost is prohibitive (this is an explicit decision, communicated)
- Foundation alliance may require communities participating in foundation-level governance to maintain at least one allied replica (constitutional baseline)
- Instance alliances offer allied replication as a service; they do not require it of their member communities
### 9.7.3 Cache replica behavior [Phase 4]
 
- Automatically created when local users or alliance memberships connect to remote mailboxes
- Storage budget per instance is configurable by operator
- TTL for inactive content configurable per content type
- Grace period for cleanup after local interest ends (default: 30 days, configurable)
- LRU-style eviction when storage budget approached
- Cleanup of cache replicas is local operator decision; no platform mandate
### 9.7.4 Replica integrity [Phase 4]
 
- Replicas periodically publish signed attestations of what they hold (Merkle root over current content)
- Allied replicas attest to complete history
- Cache replicas attest to current holdings and what they have cleaned
- Attestations are submitted to public verifiable logs operated by the Foundation (see Section 9.14); peers verify against the public log state rather than trusting operator-provided state
- Witnesses sign log heads to prevent log operators from equivocating
- Mismatches between attestations and served content are detectable by clients
- Selective withholding by a replica is detectable through cross-checking against log attestations
- Protocol-level enforcement of replica honesty combines detection through public logs with governance action (community/alliance/instance can act on detected misbehavior)
### 9.7.5 Write and read paths [Phase 4]
 
- Reads can happen from any tier with a replica holding the requested content
- Writes always go to the authoritative instance for the mailbox
- The authoritative instance accepts writes in canonical order
- Replicas eventually reflect the authoritative order
- No conflict resolution needed for in-mailbox writes (single source of truth per mailbox)
- Cache replicas may serve stale data when authoritative is unreachable; clients are informed when data is potentially stale
### 9.7.6 Failure modes and recovery [Phase 4]
 
- **Authoritative operator goes hostile**: community migrates using allied replicas as new source of truth
- **Allied instance fails**: other allied replicas continue; community establishes new allied relationships
- **All instances fail**: user-side backups provide lossy point-in-time recovery
- **Community has no natural allies**: user-side backups are the available protection; this is a known limitation
- **Cache replica withholds content**: detectable through integrity attestations; community/instance alliance can act on the information
- **Network partition between replicas**: replicas continue serving what they have; reconciliation when partition heals; writes blocked until authoritative reachable
### 9.7.7 Cold archive [Phase 4, optional]
 
- Communities can configure off-platform cold archive of their history (S3 Glacier, Backblaze B2 Cold, etc.)
- Cold archive provides long-term durability beyond active operational replication
- Cold archive uses standard export format (see CCF-11)
- Cold archive is community decision, not platform requirement
### 9.7.8 Cross-cutting interactions [Phase 4]
 
- Defederation affects cache replicas: replicas of defederated peers are cleaned up after grace period
- Allied replicas survive defederation if the allied agreement predates the defederation (the agreement is the commitment, not the federation status)
- Schism creates a complex case: replicas held by the pre-schism community become replicas of both forks; cleanup decisions are per-fork governance
### 9.8 Write and read paths [Phase 4]
 
- Reads can happen from any instance with a replica
- Writes always go to the authoritative instance for the mailbox
- The authoritative instance accepts writes in canonical order
- Replicas eventually reflect the authoritative order
- No conflict resolution needed (single source of truth per mailbox)
### 9.9 Cross-instance operations [Phase 4]
 
- Users can be members of communities on different instances than their home instance
- Communities can have members from many instances
- Migration of communities and users between instances preserves identity and relationships
- Federation traffic is encrypted; instances cannot read content they route
### 9.10 Home instance concept [Phase 4]
 
- Each user has a designated home instance based on their home community
- The home instance routes communication for the user
- Changing home community changes home instance
- Home instance changes are atomic with appropriate state transfer
### 9.11 Address discovery [Phase 4]
 
- Initial address discovery happens via QR code (which includes the inviter’s instance ID and current URL)
- URL changes propagated by signed messages from the instance to its known peers
- Sudden URL changes (after downtime) handled by signed announcement to known peers when instance comes back up
- No global addressing system needed; each instance maintains mappings only for its direct peers
### 9.12 Migration semantics [Phase 4]
 
- Communities can migrate between instances
- Migration changes the authoritative instance for the community’s mailboxes
- Community cryptographic identity does not change during migration
- Peers learn of migration through signed messages
- Migration is atomic from users’ perspective with grace-period dual availability
- Migration tooling ships with federation, not after it
### 9.13 Defederation [Phase 4]
 
- An instance can defederate from a peer at any time without negotiation
- Defederation stops sending to and accepting from the defederated peer
- Users on defederating instance lose access to communities on the defederated peer
- Communities on the defederated peer continue operating; they’re just unreachable from this instance
- Defederation is a local choice with local consequences
### 9.14 Public verifiable logs [Phase 4]
 
- The Foundation operates public verifiable logs as platform infrastructure (analogous to Certificate Transparency in the web PKI)
- Multiple log uses:
  - Replica integrity attestations (preventing selective withholding)
  - Equivocation defense (preventing instances from presenting different state to different peers)
  - Audit trails for moderation actions and governance decisions
  - Possibly franking verification support
- Operators submit attestations; peers verify against public log state
- Witnesses sign log heads to prevent log operators from equivocating
- Logs are publicly readable; the content logged is itself signed and (where applicable) encrypted; what’s public is the structural integrity, not the content
- Specific library choice (Rust alternatives to Trillian) is pending investigation
- Foundation operating logs as public service is analogous to its operation of default credential trust framework and operator/plugin certification — public infrastructure that operators rely on rather than each operator running their own
-----
 
-----
 
## 10. Alliances
 
Alliances are group entities composed of multiple communities. They have their own governance, structure, and operations. Bilateral relationships between two communities are a degenerate case of alliances with two members. Registries are a specific kind of alliance focused on discovery (see Section 11).
 
### 10.1 Alliance formation [Phase 2]
 
Alliance formation requires in-person contact, matching the platform’s broader pattern that meaningful relationships start in person rather than online. The mechanism uses an extension of the QR-driven invitation flow already used for community joining.
 
**The community-handshake QR exchange:**
 
- Two members from two different communities, each with appropriate permission within their respective communities, meet in person and exchange community-handshake QR codes
- A community-handshake QR encodes the proposing community’s identity, the proposer’s identity, the proposer’s role authorization, and a signed proposal-to-form
- Permission to propose alliance formation is configurable per community; default is community admin authority. Communities can extend this to other roles or to any member, depending on template
- Each proposer’s community must then ratify the proposal through community governance
- When both communities have ratified, the **unincorporated alliance** comes into being
**The unincorporated alliance is a temporary governance state:**
 
- The alliance exists with a cryptographic identity and hub instance, but operates under restricted governance until its charter is ratified
- The hub instance is initially the proposing community’s instance (the community of whichever proposer triggered the QR exchange); this can be changed by the charter
- Each member community has one representative who votes on alliance matters during this phase. A community can designate a permanent alliance representative role, or nominate someone for a temporary role specific to this alliance
- All members of all member communities can see alliance proceedings during the unincorporated phase. This is higher visibility than the eventual operating state, befitting the founding context where members are evaluating what their community is committing to
- At least one discussion feed is open during the unincorporated phase, governed by the hub instance’s rules (for moderation) plus the alliance’s 2/3 ejection rule (for managing alliance-level participation)
- The unincorporated alliance can take only three kinds of governance actions: admit new founding members, eject members (2/3 vote), and ratify the charter (unanimous vote of representatives)
**Joining the unincorporated alliance:**
 
- The two original proposing communities set minimal initial configuration at QR-scan time:
  - Sybil tolerance gating for prospective founding members (which membership signals or credential requirements apply to a community seeking to join — drawn from the standard sybil tolerance gradient)
  - Joining threshold for new founding members: either “any community meeting the sybil gating can apply” or a specific representative vote threshold for admission (default 2/3)
- A community applying to join during the unincorporated phase goes through its own governance to apply, and then through the alliance’s joining threshold to be admitted
- A 2/3 vote is required for new communities to join, not automatic acceptance — without this, a single QR-coded sybil community could disrupt formation by joining and then obstructing proceedings
- New founding members participate in charter ratification on equal footing with the original two
**The 2/3 ejection mechanism:**
 
- During the unincorporated phase, any member community can be ejected by 2/3 vote of representatives
- Ejection is available for any reason, not only deadlock-breaking — it is the alliance’s only ongoing governance lever during this phase
- An ejected community is removed from the unincorporated alliance and from the discussion feed; their representative loses voting authority
- The ejection threshold being 2/3 rather than higher acknowledges that founding alliances need a way to move forward when reconciliation fails
**Charter ratification — the exit from unincorporated state:**
 
- The charter contains the alliance’s initial template choice (default: equal-vote council, 2/3 by people weight, voting by representatives) plus any non-default initial configuration: alliance name and purpose, custom thresholds, custom roles, hub instance choice if different from default, ongoing membership criteria, dues structure if any
- Charter ratification requires unanimous vote of representatives
- After ratification, the alliance is incorporated and operates under its charter; the unincorporated-phase governance restrictions no longer apply
- Customization proposals beyond the template can be ratified as part of the charter or after, but the template itself must be unanimous at ratification
**Failure to reach charter:**
 
- If unanimity cannot be reached and the alliance does not want to use the 2/3 ejection mechanism, member communities can withdraw through their own governance
- An unincorporated alliance with fewer than two member communities dissolves quietly
- No platform-imposed expiration; the proto-alliance ends when its membership drops below two or when all parties give up
- Most founding cohorts should be able to converge through some combination of patience, ejection, and withdrawal
**Why the unincorporated phase exists:**
 
- Without it, the first two communities to scan QRs could lock in a template that favors their preferences before inviting others to join. With the unincorporated phase, communities joining during formation have equal voice in template selection
- The unanimity requirement for template ratification means later-joining founding members cannot be outvoted on the alliance’s founding character
- The 2/3 ejection provides a pressure valve so a single intransigent community cannot prevent the alliance from incorporating
- The visibility property lets community members evaluate the founding decision their representative is participating in
### 10.2 Alliance structure [Phase 2]
 
- An alliance is a group of communities with shared governance
- Alliances have their own cryptographic identity (keypair)
- Alliances have a name, description, and purpose
- Alliances have their own governance separate from any member community’s governance
- Alliances can have their own roles (positions held by individual members of member communities)
- Alliances can have their own feeds for alliance-wide communication
- Alliances can have any number of member communities (two or more)
- An alliance comes into being through the formation flow described in Section 10.1 (unincorporated phase, charter ratification, incorporation)
### 10.3 Alliance membership [Phase 2]
 
- After incorporation, a community joins an alliance through a community-handshake QR exchange between a member of the joining community (with appropriate permission, configurable per community, default community admin) and a member of an existing alliance member community
- Joining requires consent from both sides:
  - The joining community ratifies through its own governance (deciding whether the community wants to be part of this alliance under its charter terms)
  - The alliance consents through its own governance per its charter (deciding whether to admit this community)
- Neither side’s consent alone is sufficient; the join is complete only when both have ratified
- The in-person QR requirement applies whether the alliance is being formed or extended — alliances are relationships between communities, and like all such relationships on the platform, they begin in person
- Member communities can leave alliances through their own governance
- Alliances can remove member communities through alliance governance, per the alliance’s charter
- Membership terms (rights, obligations, dues) are configured per alliance in the charter
### 10.4 Alliance governance [Phase 2]
 
- Each alliance has its own governance configuration, settled at incorporation through the charter and amendable thereafter through alliance governance
- The default template is equal-vote council with 2/3 by people weight, voting by representatives. Other templates are available at charter ratification; the specific template library is to be designed
- Alliance voting can be by community (each member community gets a vote) or by individuals (members of member communities vote)
- Representative selection is per member community: either an existing alliance-representative role, or a nomination for this alliance specifically
- Alliances have their own proposal types, ballot types, tally methods
- Alliance decisions can affect member communities (within terms of alliance membership)
- Alliances can amend their own governance through meta-governance
### 10.5 Alliance functions [Phase 2]
 
- Shared moderation: alliance members can apply moderation across member communities
- Shared shunning: collective blocklists honored across member communities
- Shared credentials: credentials from one member community trusted in others (configurable)
- Shared event calendars: events visible across member communities
- Cross-community member recognition: members of one member community recognized in others
- Joint resource sharing: shared treasury, shared services
- Shared juror pools for cross-community moderation (see Section 14)
- Discovery functions (when alliance acts as registry, see Section 11)
### 10.6 Alliance roles [Phase 2]
 
- Alliance can define roles held by individuals from member communities
- Examples: alliance moderator, alliance representative, alliance treasurer, alliance juror
- Roles are filled through alliance governance (election, sortition, appointment)
- Roles confer permissions within the alliance scope
### 10.7 Alliance dissolution [Phase 2]
 
- Alliances can be dissolved through alliance governance
- Dissolution terms (asset division, member notification, transition period) configurable
- Member communities revert to non-allied state
- Dissolution preserves historical record
### 10.8 Alliance feeds [Phase 2]
 
Alliance feeds come in distinct types, each with a different cryptographic mechanism appropriate to its participation model. The feed type is determined at feed creation based on purpose.
 
**Coordination feeds:**
 
- Standard MLS group of designated representatives from member communities
- Each member community contributes representatives via its own governance (e.g., an “alliance representative” role)
- Used for alliance governance, planning, decision-making, moderation council
- Sized by representative count (not total member count); typically tens to low hundreds
- Full MLS properties: per-member forward secrecy, post-compromise security, direct membership
**Broadcast feeds:**
 
- Alliance publishes; content signed by alliance keypair
- Content distributed through hierarchical fan-out to member communities
- Each level (alliance → member communities → individual members) re-encrypts with its own keys
- Member communities receive alliance content and propagate through their own MLS groups
- Members see alliance broadcasts in their community feeds
- Write-restricted to alliance roles; read-by-all-members-of-member-communities
**Alliance-public feeds:**
 
- Sender-key protocol (Megolm-style, likely via vodozemac) for content visible to all members of constituent communities
- Alliance group key shared with all members; each sender maintains a per-sender ratchet for forward secrecy
- New member adds: existing member encrypts current group key under new member’s alliance public key; threshold of representatives signs acceptance
- Member removals trigger group key rotation; new key distributed to remaining members (O(N) operation, acceptable at expected governance frequencies)
- Scheduled rotation independent of membership changes limits damage from key compromise; default monthly, configurable by alliance governance
- Membership in the sender-key layer is driven by the governance MLS group state; composition changes there trigger key distribution events in the messaging layer
- Recipient model is community-configurable: communities choose whether all members or only designated alliance participants internally receive the group key
- Provides confidentiality at the alliance boundary (non-members cannot decrypt) and authentication of sender (signed by sender’s identity key)
- Does not provide confidentiality from alliance members; this privacy property matches the social reality of broad participation
- Persistent storage for governance-shaped events (membership changes, key rotations); TTL-based decay for message content
- Recovery from peer member as fallback when local replica has decayed
- Reuses MLS event format and signature scheme for protocol envelope where appropriate
**Restricted alliance feeds:**
 
- Standard MLS group for specific subsets of alliance members
- Examples: alliance moderation council, juror pool, treasury committee
- Membership is by individual (not by community); members are individuals with appropriate alliance roles
- Sized by subset; typically small enough for direct MLS
- Used for sensitive cross-community work requiring strict per-individual E2E
### 10.9 Alliance hierarchies [Phase 2]
 
- Alliances can themselves be members of other alliances
- An alliance of alliances (regional alliance composed of local alliances) is supported
- An alliance joining another alliance follows the same in-person community-handshake QR requirement that applies to communities joining alliances — a member from a community of the prospective member alliance and a member from a community of the receiving alliance meet in person
- Both-sides consent applies as for individual community joining: the prospective member alliance ratifies through its own governance, and the receiving alliance accepts per its charter; both required
- No depth limit in principle; practical limits per implementation
- Each level has its own governance
-----
 
## 11. Registries (specialized alliances)
 
Registries are alliances whose primary function is to curate discoverability of communities within a geographic or topical scope. Being a registry member means being listed; being listed means being a registry member.
 
### 11.1 Registry structure [Phase 2]
 
- A registry is an alliance with a geographic or topical scope
- The alliance’s member communities are exactly the communities the registry lists
- Registry governance decides which communities are admitted
- Multiple registries can exist per locality with different curatorial perspectives
- Registries themselves are discoverable through bootstrap directories and inter-registry alliances
### 11.2 Registry membership [Phase 2]
 
- Communities apply to be listed in a registry
- Listing is alliance membership; benefits include discoverability
- Communities can be in multiple registries (member of multiple registry alliances)
- Communities can withdraw from registries voluntarily
- Registries can remove member communities through alliance governance
### 11.3 Registry discovery [Phase 2]
 
- Users browse registries to find communities in their locality or topic area
- Users can filter member communities by type, size, governance style, language, etc.
- Discovery is locality-keyed; cross-locality discovery happens through registry alliances or explicit search
- Discovery does not require account creation
- Registry contents pulled client-side for local filtering where feasible
### 11.4 Cross-registry operations [Phase 2]
 
- Registries can ally with other registries (alliance of registries)
- Registry alliances enable cross-locality discovery
- Multiple registries can share curatorial signals
- Communities in one registry can be visible through allied registries
### 11.5 Registry governance [Phase 2]
 
- Standard alliance governance applies
- Curatorial decisions (admit, remove, modify listing) are governance actions
- Registry governance is transparent to listed communities
- Disputes within registries follow alliance governance procedures
### 11.6 Registry bootstrap [Phase 2]
 
- New registries can be created by any group of communities forming an alliance with registry purpose
- Bootstrap directories list known registries
- Reputation and continuity contribute to registry trust over time
-----
 
## 12. Credentials and verification
 
### 12.0 Basic credential management [Phase 1]
 
Phase 1 ships with a working credential system at minimal scope. Communities can issue credentials, members can hold them, and verification works within the community. The full credential ecosystem (external issuers, cross-community trust networks, sophisticated presentation flows) ships in Phase 2.
 
**Phase 1 credential capabilities:**
 
- Users have a credential wallet showing credentials issued to them
- Credentials display: issuer (community), claim type, claim value, issuance date, expiration date
- Users can present credentials within the issuing community for verification
- Issuance produces a notification to the recipient and a record in the wallet
- Expired credentials are visually distinguished in the wallet but preserved in history
- Revoked credentials are marked but preserved (history is auditable)
- Basic selective disclosure (present a single credential without revealing others in the wallet)
- Credentials persist across device additions (synced via E2E channel like other state)
**Phase 1 credential types** that ship out of the box:
 
- Membership credential (issued when a user joins a community, expires when membership ends)
- Role credential (issued when a user takes on a role, expires when role ends)
- Attendance credential (issued when a user attends an event)
- Vouching credential (issued when a member vouches for another person, persists with traceability — see Section 15.3)
**Not in Phase 1:**
 
- External issuers (government, Apple Wallet, etc.) — Phase 2
- Cross-community credential trust — Phase 3
- Sophisticated zero-knowledge proofs — Phase 2
- Credential trust networks via alliances — Phase 3
- Public credential verification (verifying as someone outside the issuing community) — Phase 2
- Credential renewal flows — Phase 2
The intent is that pilot members get working credentials for their community life (proof of membership, attendance, role) without requiring the full ecosystem of external trust and cross-community verification.
 
### 12.1 Credential structure [Phase 1]
 
- Credentials are W3C Verifiable Credentials using BBS signatures (the standardized family: `bbs-signatures`, `bbs-blind-signatures`, `bbs-per-verifier-linkability`, with W3C `vc-di-bbs` for VC integration)
- A credential has: issuer, subject (committed `pid`), claim type, claim values, issuance date, expiration date, signature
- Credentials support selective disclosure (prove some claims without revealing others) and unlinkable presentations (two presentations are mutually unlinkable proofs)
- Each holder has a secret `pid` (holder identifier) that is committed blindly into every credential they hold; the same `pid` enables per-verifier pseudonyms scoped to any verifier/scope
- Credentials commit `pid` blindly at issuance — the issuer signs over a committed value without learning it (blind BBS)
- Credential-linking through shared-`pid` ZK equality allows the holder to prove two credentials commit the same `pid` without revealing the `pid` itself — used for combined presentations and revocation
### 12.2 Two-credential humanness model [Phase 1]
 
- The platform splits humanness attestation into a long-lived root and short-lived freshness:
- **Humanness root credential.** Long-lived, instance-alliance-issued at signup tied to external proof check, blindly committing the user’s `pid`. Never presented directly to verifiers. The foundation from which per-verifier pseudonyms derive.
- **Humanness-freshness credential.** Short-lived, instance-alliance-issued, reissued from the root over the same `pid`. Cheap reissue path (proof-of-possession of the root, no external proof re-check). Presented as proof the holder is currently humanness-attested.
- Revocation works by refusing to reissue the freshness credential; once the current short-lived credential expires, no presentation can succeed. No accumulator, no heartbeat, no renewal round-trip.
- Revocation latency equals the freshness credential’s lifetime (configurable per community policy; short lifetime = faster revocation + more reissue churn).
- W3C Bitstring Status List (the standard VC revocation mechanism) is not used — it breaks unlinkability through per-credential indexes and issuer-contact-at-check.
### 12.3 Internal credential issuers [Phase 1]
 
- Communities can be configured as decency-credential issuers
- Issuer communities have credential types they can issue (membership, role, attendance, vouching, good standing)
- Issuance is a community governance action
- Credentials issued by a community are signed by the community’s keypair and commit the holder’s `pid` (shared with the holder’s humanness root)
- Issuance produces a credential record and notification to recipient
- Issuance happens through encrypted MLS channels; the operator does not observe issuance events directly
- This protects community-level pseudonymity for communities without public faces (operator cannot link MLS group members to a community identity via observable credential issuance)
### 12.4 External credential issuers [Phase 2]
 
- Platform supports external issuers (government ID, Apple Wallet, university, employer)
- External credentials are verified per their issuance protocol
- External credentials provide stronger sybil resistance for some claim types
- Users can attach external credentials to their platform identity
- External-credential nullifier support (EUDI wallet, eIDAS, mDL) is a watch item — would enable cleaner humanness dedup by pushing dedup to the external issuer
### 12.5 Combined presentations [Phase 1]
 
- At presentation, the holder produces one combined BBS proof: humanness-freshness credential (proves current humanness attestation) + relevant decency credential(s) (proves membership/role/good standing/etc.) + shared-`pid` ZK equality (binds both to the same person) + selective disclosure of any attributes the verifier needs
- The verifier learns the holder is freshly humanness-attested AND a member-in-good-standing AND any disclosed attributes, all bound to the same person, without learning the `pid` itself
- Per-verifier pseudonyms (when needed for identity, voting, or ticketing) derive deterministically from the `pid` scoped by the verifier/scope identifier
- Verification is offline — the issuer is not contacted at presentation time (BBS property, supporting the federation commitment that credentials are valid at presentation time, not at current state)
### 12.6 Credential trust [Phase 3]
 
- Communities specify which issuers they trust for which claim types
- Trust can be direct (community explicitly trusts issuer)
- Trust can be transitive through alliance (trust an allied community’s trusted issuers)
- Trust can be configured with weights (some credentials count more than others)
- Communities can negatively mark issuers (explicit distrust)
- Humanness trust is a cross-instance trust-policy decision (whether instance A’s humanness credentials are honored by instance B); the same direct/alliance/transitive framework applies
### 12.7 Credential management [Phase 2]
 
- Users have a credential wallet showing all their credentials
- Users control which credentials are visible to which communities
- Users can present credentials proactively or in response to requests
- Credential proofs are zero-knowledge where possible
- Expired credentials are clearly indicated
- Revoked credentials are removed from active use but historical record preserved
### 12.8 Credential lifecycle [Phase 2]
 
- Issuance: issuer creates credential and delivers to subject through encrypted MLS channel
- Storage: subject holds credential in their wallet
- Presentation: subject proves credential to verifier (offline, no issuer contact)
- Renewal: subject renews credential before expiration (for humanness-freshness, this is reissue from the root)
- Revocation: issuer refuses to reissue (for freshness-based credentials) or publishes revocation through the ID-hash log (for cross-instance revocation, see federation readiness doc Section 6.2.1)
- Expiration: credential automatically expires per its terms
### 12.9 Credential continuity across operator changes [Phase 4]
 
- Credentials are cryptographically signed by the issuing community or instance alliance’s keypair, not by the operator
- Credential verification depends on the issuer’s public key, not on operator infrastructure
- Verification can happen client-side without operator cooperation
- When a community migrates to a new operator, existing credentials remain valid
- A hostile operator cannot invalidate previously-issued credentials
- Members can hold local copies of their credentials and verify them independently of any operator
- This is the architectural protection against operator defection for credential infrastructure
### 12.10 Post-quantum horizon [watch item]
 
- BBS is pairing-based (BLS12-381) and not post-quantum
- No standardized post-quantum anonymous credential scheme currently exists with the required properties (selective disclosure plus pseudonyms plus blind issuance in a PQ construction is research-stage)
- The credential layer is built behind a clean replaceable interface from the start; replaceability is a design requirement
- When PQ anonymous credentials mature, BBS is replaced at the credential layer without disturbing the rest of the architecture
- See Phase 6 (architecture doc) — post-quantum credential hardening is a deliberate watch item with no current migration path
-----
 
## 13. Messaging protocol
 
### 13.1 MLS group management [Phase 1]
 
- Each community has at least one MLS group for cryptographic operations
- Restricted feeds within a community may have their own MLS subgroups
- MLS groups manage member key state and group key derivation
- Adding/removing members happens via MLS commits
- Members read group state and verify membership cryptographically
### 13.2 Mailbox-based delivery [Phase 1]
 
- Mailboxes are the routing primitive (UUID identifiers, no per-user inboxes)
- Each relationship and each feed has its own mailbox
- Messages are written to mailboxes and read by authorized parties
- Server cannot determine who wrote what; only that authorized writes occurred
- Mailbox access requires cryptographic authentication
### 13.3 Mailbox rotation [Phase 1]
 
- Mailboxes can be rotated by mutual agreement of parties
- Rotation protocol: propose new mailbox, acknowledge, confirm, terminate old mailbox
- Rotation is encrypted; server doesn’t learn relationship between old and new mailboxes
- Rotation cadence configurable per relationship/feed
- Lost rotations can be recovered through party communication
### 13.4 Message structure [Phase 1]
 
- Messages have: sender (within group, not visible to server), content, content type, timestamp, signature, references (replies, mentions)
- Messages are encrypted with group keys before reaching server
- Server stores opaque encrypted blobs
- Decryption happens client-side using member’s MLS state
### 13.5 Real-time delivery [Phase 1]
 
- Push notifications via platform-specific mechanisms (APNs, FCM)
- Notification content is delivered encrypted; decryption happens on device
- Real-time delivery via WebSocket connections to authenticated clients
- Offline message queueing on server until clients fetch
- Catch-up mechanism for clients returning after extended offline
### 13.6 Multi-device coordination [Phase 1]
 
- Each device is technically a separate MLS leaf
- Application layer presents devices as unified user
- New device addition requires authorization from existing device
- Device removal cleanly excludes that device
- Message history syncs across user’s devices via encrypted channel
### 13.7 Hierarchical alliance messaging [Phase 4]
 
- Alliances use a two-layer cryptographic design (see Section 10.8): MLS group for representative governance; sender-key protocol (Megolm-style via vodozemac or equivalent) for broader alliance content
- Governance layer (representative MLS group): standard MLS group whose members are designated human representatives of constituent communities; provides full MLS guarantees; handles confidential governance work (proposals, threshold multi-signature, sensitive coordination)
- Messaging layer (sender-key protocol): all members of all constituent communities receive the alliance group key; goal is broad participation, not confidentiality from members
- The sender-key layer provides confidentiality at the alliance boundary, authentication of sender (community-level), per-message forward secrecy through per-sender ratchets, and bounded key compromise through scheduled rotation
- Does not provide confidentiality from alliance members — when many people can read content, secrecy from members is not a guarantee any protocol can deliver in practice; this matches social reality
- Membership changes in the governance MLS group trigger key distribution and rotation events in the messaging layer
- Nested alliances (alliance whose members are alliances) compose the same pattern: representative MLS groups at each level, sender-key messaging at each level
- All members receive the broadcast directly through the sender-key layer; cost is O(N) on membership changes and scheduled rotations but acceptable at expected alliance frequencies
- The two-layer composition uses validated protocols throughout; specific implementation decisions (sender-key library choice, threshold scheme, composition with MLS state changes) are pending cryptographer review
### 13.8 Confidential sub-groups [Phase 2]
 
- Some feeds need confidentiality from non-role-holders (treasury, council, pastoral care, working groups with confidential material)
- Confidentiality from non-members of a sub-role requires a separate MLS group (a sub-group) with restricted membership — anything derived from the community exporter is computable by every community member
- Sub-groups are created by branching from the parent community group (RFC 9420 branching mechanism) with the parent’s resumption PSK injected at creation
- The resumption PSK cryptographically proves sub-group members were members of the parent at the branch epoch — provenance preventing an operator from slipping outside keys into the sub-group
- Branched sub-groups are full MLS groups with their own exporters, their own partition keys (unlinkable to parent feeds at the DS), their own franking, and their own write-cap if role-write is enforced
- **Parent Remove must fan out to sub-groups.** Branching has no continuing effect on parent or sub-group: the resumption PSK proves members were in the parent at the branch epoch, not that they still are. Keeping sub-group membership consistent with parent membership is an application-layer obligation. A parent Remove must trigger Remove fan-out into every sub-group the persona belonged to. Forgetting this invariant leaves ex-members with role access.
- Schism uses this primitive: branch a sub-group at the split epoch, then claim history inheritance through signed attestations (federation doc mechanism); the resumption-PSK membership-binding proves the schismatic faction were genuine members at the split
### 13.9 Role-write enforcement [Phase 2]
 
The server cannot enforce role-based write access by inspecting content (E2E encryption hides what would need to be inspected); this is correct for an untrusted-server model. The server *can* enforce that writes to a partition come from authorized writers by verifying a signature against a write-capability public key.
 
**Mechanism:**
 
- For feeds requiring genuine role-write enforcement, writers form a sub-group (e.g., publisher sub-group for an announcements feed; council sub-group for council deliberation)
- The writer sub-group’s exporter derives both a partition key (routing) and a write-capability keypair (`haven.write-cap.v1`), rotating per epoch with the partition
- Each write to the partition carries two signatures:
  - **Outer write-cap signature** (server-visible): verified by the server against the partition’s write-cap public key. Anonymous — server learns an authorized writer acted, not which one. This is the landing-gate; the server rejects writes that don’t verify.
  - **Inner persona signature** (reader-visible, inside ciphertext): proves which specific writer for accountability and franking. Server never sees this.
- The server pays a per-write signature verification cost in exchange for actually enforcing the role-write gate
**Three feed shapes:**
 
|Feed shape                    |Read access                                         |Write access                 |
|------------------------------|----------------------------------------------------|-----------------------------|
|Open-read / open-write        |Community-membership gate                           |Community-membership gate    |
|Open-read / role-write        |Community-membership gate (encrypt to community key)|Publisher sub-group write-cap|
|Confidential-read / role-write|Sub-group membership (encrypt to sub-group key)     |Same sub-group’s write-cap   |
 
**Caveats:**
 
- The write-cap is a shared secret among the writer subset; a malicious writer can leak it to an outsider. Inherent to any shared-key gate. Mitigated by per-epoch rotation and by the leaker being an attributable writer through the inner signature. No scheme fully prevents a member proxying writes.
- Cost: more sub-groups to maintain and rotate (one writer sub-group per role-write feed), plus per-write server verification
- Losing write role = Remove from the writer sub-group, triggering the Section 13.8 fan-out invariant
This generalizes the existing publisher group pattern for public feeds (Section 4.7) to any feed needing real write-role enforcement.
 
### 13.10 KeyPackage handling [Phase 1]
 
MLS uses KeyPackages to convey a member’s initial keying material when they’re added to a group. RFC 9420 specifies single-use semantics by default: a KeyPackage is consumed by the Welcome that adds the member, and the corresponding init_key is deleted after the Welcome is decrypted. An optional last-resort KeyPackage flag (`draft-ietf-mls-extensions`) allows a reusable KeyPackage to persist at the Delivery Service for cases where a member’s KeyPackage stash runs dry while they’re offline.
 
**Haven uses no last-resort KeyPackages anywhere.** Every join in Haven is exactly one of three types, and the trichotomy is exhaustive given the in-person membership model:
 
- **In-person-rooted joins.** The joiner is physically present (or remote-with-vouching for accessibility cases) and supplies a fresh single-use KeyPackage through the QR handshake. This covers primary community joins and first cross-instance joins (since federation establishes as a side-effect of the QR invite).
- **Parent-state-derived joins.** The joiner is already a member at some level; sub-group or feed adds derive from parent state (authenticated parent leaf, resumption PSK branch). No DS stash fetch needed. This covers all internal sub-group/feed adds, including existing cross-instance members being added to additional feeds.
- **Interactive joins.** The joiner is actively participating in their own join and supplies a fresh KeyPackage as part of that interaction. This covers multi-device adds and vouched remote onboarding (remote is not the same as offline-stash-fetch).
The one scenario async-stash-fetch (and therefore last-resort KeyPackages) exists for — adding an offline party with no existing group relationship — is exactly what the in-person membership gate forbids. No one is added to a community without being present for onboarding; no one is added to a sub-group without already being a parent member. The reusable KeyPackage is needed nowhere.
 
**Bootstrap KeyPackages are single-use and short-lived.** Lifetime is bounded to the validity of the invitation that produced them. Multi-use invite links are rejected — they would force a reusable bootstrap KeyPackage, reintroducing both the reuse-FS problem and a bypass of the in-person-trust gate.
 
**Stash exhaustion handling.** When a member’s KeyPackage stash at the DS runs dry, the platform handles this through proactive replenishment plus delayed-add semantics (“you’ve been added; it finalizes when you’re next reachable”). Civic-use-case rhythms make this acceptable; instant async adds aren’t required.
 
**Implementer note.** An off-the-shelf MLS stack offers last-resort KeyPackages as the default patch for stash exhaustion. Do not adopt this default reflexively. The reason last-resort is unnecessary in Haven (the trichotomy above) is not visible from the library; the discipline is platform-wide (“no standing reusable secret material at the untrusted layer,” consistent with the capture-forward principle in Section 13’s broader treatment).
 
**Reopening condition.** This analysis depends on the in-person / parent-derived / interactive trichotomy holding universally. If a future flow ever needs to add a party who is none of these — not present, not already a member, not interactively participating — last-resort would need reconsideration. Recorded as the reopening condition, not a current gap.
 
-----
 
## 14. Moderation
 
The platform’s moderation operates through member reports rather than platform scanning (end-to-end encryption makes scanning structurally impossible). Multi-level rule systems (community, alliance, instance, foundation) each have their own published rules; reports route to whichever levels have rules that were violated; each level acts according to its own judgment of its own rules.
 
### 14.1 Moderation actions [Phase 1]
 
- Hide content from feed (visible to moderators, not to members)
- Delete content (server-side deletion)
- Edit content with notation (in feeds where this is permitted)
- Flag content for review
- Mute member (their content not delivered to specific other members)
- Suspend member temporarily (no posting, may or may not have reading rights)
- Remove member from community
- Ban member (cannot rejoin)
### 14.2 Moderation models [Phase 2]
 
Different communities and different feeds within communities can use different moderation models:
 
- **Individual moderators**: specific people with the moderation permission act unilaterally
- **Moderation council**: a group of moderators decides actions collectively, with their own internal procedures
- **Random jury**: random selection from an eligible pool decides reported items, each jury empaneled per case
- **Hybrid**: different action types use different mechanisms (individual mods for quick removals, council for bans, jury for appeals)
- **Community vote**: significant moderation actions require community vote
- **Pre-publication review**: members post to an intake queue (a confidential-read feed readable by a reviewer sub-group); a reviewer republishes approved items into the open feed. Composed from the role-write pattern (Section 13.9) — the reviewer sub-group holds the open feed’s write-cap — so a community gets human pre-moderation while the operator stays blind. Constraints: the queue’s review property is disclosed to posters the way history status is (Section 5.9); review applies to a community’s own outbound contributions, never as an alliance-level chokepoint (alliance broadcast has no plaintext chokepoint, by design); and it pairs with digest cadence (Section 4.3) by default, because a real-time reviewed feed is a promise reviewers can’t staff.
### 14.3 Multi-level rule system [Phase 1]
 
Rules exist at multiple levels of the platform’s structure. Each level publishes its rules as human-readable text. Rules are not platform-enforced (the platform cannot read content); they are the criteria moderators apply when reviewing reports.
 
**Rule levels:**
 
- **Community rules**: specific to the community; reflect the community’s particular norms and purposes
- **Alliance rules**: shared across alliance member communities; apply within communities that have joined the alliance
- **Instance alliance rules**: apply to all communities hosted on the instance
- **Foundation baseline rules**: minimum rule set required for foundation certification; apply to all communities on certified instances
**Rule scope by level:**
 
- Local rules (community) are typically most numerous and detailed, reflecting specific community norms
- Higher-level rules (alliance, instance, foundation) are typically fewer and address broader concerns
- The foundation baseline is intentionally minimal — universally applicable prohibitions only
**Rule precedence:**
 
- Higher-level rules establish floors that lower levels cannot opt out of
- Communities can be more restrictive than their alliance/instance/foundation requires
- Lower-level rules cannot override higher-level rules (a community cannot permit what its instance forbids)
- Rules at the same level are coordinate (no hierarchy among community rules, for example)
**Rule changes by level:**
 
- Default thresholds: community rules by simple majority, alliance rules by 60% supermajority, instance alliance rules by two-thirds supermajority
- Each level can configure its own thresholds within governance bounds
- Foundation baseline rules change through Foundation Alliance governance (the math-enabled democracy framework)
**Rule versioning and prospective application:**
 
Rules apply prospectively, not retrospectively. Content posted under rule set version N can only be reported and evaluated against rule set version N — the rule set as it existed at the time the content was posted, not the current rule set.
 
This is fundamental fairness (no retroactive enforcement of rules that didn’t exist when the content was created) and a structural property of the moderation system, not a policy choice.
 
Implementation:
 
- Rule changes at every level (community, alliance, instance, foundation) are event-sourced; the rule state at any timestamp is reconstructible from the event log
- Reported content has a posting timestamp (cryptographically anchored in the message itself)
- The reporting flow reconstructs the rule sets at all applicable levels as of the content’s posting timestamp
- The reporter sees those rules (not current rules) when selecting which they believe were violated
- Moderators evaluate the report against the historical rule set
- Moderation actions are taken according to those historical rules
Edge cases:
 
- More restrictive rule changes: content legal when posted remains legal; cannot be retroactively criminalized
- Less restrictive rule changes: content illegal when posted remains reportable, though communities may choose not to act on historical violations
- Deleted rules: still reconstructable from event log for evaluation of past content; communities choose how to act
- Substantive edits to content: treated as new authored content with new timestamp, subject to rules in effect at edit time
- Minor edits (typo corrections, formatting): treated as modifications, original posting timestamp applies
- The distinction between substantive and minor edits is community-configurable
### 14.4 Member reports [Phase 1]
 
The reporting mechanism is the primary input for moderation. Members report content or behavior; reports route to appropriate moderators; moderators act.
 
**Report composition:**
 
- The reporting member’s identity (cryptographically verified within the community)
- The specific content or behavior being reported
- The community context where it occurred
- The reported member’s community-scoped identity (necessary for action to be taken)
- A cryptographic proof that the report references real platform content (prevents fabrication)
- The rules being invoked (see rule selection below)
- Optional explanation from the reporter
**Rule selection:**
 
- When initiating a report, the reporter sees the rules at all applicable levels (community, alliance, instance, foundation)
- The reporter selects which rules they believe have been violated
- The report routes to whichever levels had rules selected
- A report can invoke multiple rules across multiple levels
- The report includes the reporter’s reasoning for each rule selected
**Routing:**
 
- If only community rules are invoked, the report routes to community moderators
- If alliance rules are invoked, the report routes to alliance moderators
- If instance rules are invoked, the report routes to instance alliance moderators
- If foundation rules are invoked, the report routes to foundation moderation
- A single report can route to multiple levels simultaneously; each level processes independently
**This handles the “community in denial” case:** if a community has a violation that also breaks instance-level or foundation-level rules, the community cannot suppress the report. The instance and foundation each receive their own copy and can act according to their own rules.
 
### 14.5 Cryptographic verification of reports [Phase 1]
 
- Reports must include franking material — cryptographic proof linking the report to actual platform content
- Franking uses the asymmetric + sealed-sender + transcript construction (Hecate-style with sender pseudonymity preservation and pattern-of-behavior support)
- For community chat, working groups, and DMs: full franking with sealed-sender for recipient pseudonymity and transcript binding for pattern reporting
- For alliance broadcasts: asymmetric + transcript franking with sender attribution at community level
- For official encrypted channels: asymmetric franking only; senders attributed by default
- Verification keys are held by stable infrastructure at each governance layer (community, alliance, instance, foundation); jurors and moderators receive verified-evidence packages without needing to hold long-term keys
- Verification can be re-performed at higher governance layers for appeals or oversight
- Broken franking (bogus commitments from malicious senders) is itself reportable; recipients can escalate by revealing per-message decryption keys to allow infrastructure to verify the sender’s signature inside the encrypted payload
- Fabricated reports referencing nonexistent content are detectable and rejected
- The verification mechanism does not reveal additional information about the community
- Pattern of false reports affects reporter reputation (see Section 15)
- Specific committing AEAD primitive, key-reveal granularity, and composition with MLS PrivateMessages and sender-key broadcast layer are pending cryptographer review
### 14.6 Identity in reports [Phase 1]
 
- Reports must identify the reported member within the community context where the report applies
- Community-scoped identity is sufficient for community-level action
- For escalation across instances (e.g., CSAM-related actions), additional identity information is included as needed for federation
- Identity information is shared only with the levels that need it to act (community moderators see community-scoped identity; foundation sees what’s needed for federation effect)
- Per-community pseudonymity is preserved except when severe verified violations require cross-instance action
### 14.7 Moderation actions across levels [Phase 1]
 
Each level can take actions appropriate to its scope:
 
- **Community level**: actions within the community (hide, delete, suspend, remove, ban from community)
- **Alliance level**: actions within alliance member communities (alliance-wide ban, alliance content removal)
- **Instance level**: actions across the instance (instance-wide ban, community removal from instance)
- **Foundation level**: actions affecting federation (cross-instance flagging, federation effect for severe violations)
Higher-level actions are not automatic consequences of lower-level findings. Each level exercises its own judgment about its own rules. A community may decide content is acceptable while the instance decides it isn’t; the instance can still act according to its rules.
 
### 14.8 Due process [Phase 1]
 
- Reported members are notified of reports against them (with reporter identity protected per community policy)
- Reported members can respond before action is taken (except in cases of immediate harm)
- Moderators must have human review of reports before action; not automated processing
- Decisions are documented and available for appeal
- Patterns of false reporting trigger consequences for the reporter
### 14.9 Encrypted content moderation [Phase 1]
 
- Moderators see decrypted content for items reported to them
- Cryptographic proof that reported content came from claimed sender
- Server cannot perform proactive content moderation (cannot read content)
- All moderation is reactive and human-driven
- The platform does not employ AI-based content classification
- The platform does not engage in client-side scanning
### 14.10 CSAM specifically [Phase 1]
 
Child sexual abuse material is universally prohibited and handled with specific provisions:
 
- CSAM is included in foundation baseline rules; cannot be opted out of by any community or instance
- CSAM reports are processed at all applicable levels in parallel (community moderators, instance alliance, foundation)
- Verified CSAM reports trigger legally-required reporting to NCMEC (National Center for Missing & Exploited Children)
- Operators register with NCMEC as electronic service providers and maintain reporting capability
- Verified CSAM violators are banned at instance level and flagged for federation-wide effect
- Federation-wide flags propagate to peer instances through signed announcements; peer instances can act on the flag according to their own policies
- Evidence is preserved for legal process per applicable law
- Reporter identity is protected from the reported user; can be disclosed only through proper legal process
- The platform cooperates with law enforcement on valid legal process while maintaining technical limits (encrypted content cannot be produced because the platform cannot decrypt it)
### 14.11 Harassment handling [Phase 1-2]
 
Harassment is primarily a within-community problem because of the platform’s structural commitments. The locality-grounded community membership means a person cannot post in a community they are not a member of; therefore cross-community harassment in the form of an outsider attacking community members is structurally prevented.
 
What can happen:
 
- A community member harasses another member of the same community
- A person who is a member of multiple communities engages in problematic patterns across them
- Members harass each other across communities they’re both members of
How the platform handles it:
 
- Within-community harassment is community moderation responsibility; community removes harasser or takes lesser action per community rules
- Patterns across communities by the same person are detected through the credential chain (the institutions that vouched for the person carry reputation stakes)
- Cross-community correlation is not platform surveillance; it’s institutional accountability through the credential system
- Severe cases (threats, doxxing, sustained targeted harassment) can escalate to instance or foundation level if community fails to act and the conduct violates higher-level rules
- The platform does not provide cross-community harassment surveillance because that would require breaking per-community pseudonymity
### 14.12 Cross-community moderation through alliances [Phase 2]
 
- Alliances can maintain shared juror pools
- Member communities can request alliance juries for moderation decisions
- Alliance juries provide external perspective on community disputes
- Alliance moderation respects individual community sovereignty (alliances can recommend; communities can override per their own rules)
### 14.13 Jury system [Phase 2]
 
- Eligible jurors defined by configuration (community members, alliance pool, specific roles)
- Random selection of jurors per case
- Jurors review reported content and context
- Jurors deliberate (in a temporary feed) and decide
- Decision rendered by jury vote (configurable threshold)
- Jurors may be from outside the community in question (especially via alliance pools)
- Juror service can be opt-in or assigned
### 14.14 Honest framing on platform safety [All phases]
 
The platform’s commitment to end-to-end encryption and minimal data collection limits what platform-side enforcement is possible. The platform’s commitments include:
 
- Member reporting mechanism with cryptographic verification
- Multi-level moderation through published rules
- CSAM reporting to NCMEC per legal requirement
- Cooperation with valid legal process within technical limits
- Federation effect for severe verified violations
- Protection for victims, accused, and reporters
The platform does not:
 
- Scan content (impossible due to encryption)
- Use client-side scanning (rejected on privacy grounds)
- Employ AI-based detection (impossible on encrypted content)
- Surveil users across communities (per-community pseudonymity)
- Pre-emptively ban accounts based on patterns alone (always requires verified report)
The structural friction against criminal use comes from:
 
- In-person QR exchange required for community membership (significant onboarding friction)
- Locality-grounded credential chain (institutional accountability)
- Vouching with social-distance-weighted reputation accountability
- No anonymous account creation
- Real-world institutional credentialing for higher-trust activities
These properties make the platform structurally less attractive for criminal enterprises compared to platforms with low-friction account creation (Signal, WhatsApp, Telegram). Criminal use is not prevented absolutely, but the platform does not offer privacy benefits that justify the substantial onboarding friction. Bad actors typically choose paths of least resistance; this platform is not a path of least resistance.
 
-----
 
## 15. Reputation
 
### 15.1 Universal identity confidence [Phase 1]
 
- Derived from verified credentials and external attestations
- Reflects humanness and uniqueness, not behavior
- Visible to communities considering admission
- Updated as credentials are added or revoked
### 15.2 Per-community behavioral reputation [Phase 1]
 
- Specific to each community a member is part of
- Accumulates from participation, fulfilled offers/requests, attendance, vouches received
- Decays without continued participation
- Used internally by community for trust signals
- Does not transfer between communities
### 15.3 Vouching [Phase 1]
 
- Inviting a member is vouching for them
- Each vouching act produces a vouching credential held by both parties (the voucher’s record of having vouched; the vouchee’s record of being vouched for)
- Vouchers take reputation hits if vouchees misbehave
- Hits decay with social distance (deeper ancestors take smaller hits)
- Vouching relationships are recorded and traceable through the credential chain
- Withdrawing a vouch revokes the vouching credential and affects associated reputation
### 15.4 Reputation effects [Phase 1]
 
- Higher reputation may unlock additional roles or permissions in some communities
- Reputation thresholds can gate certain actions (configurable per community)
- Reputation visible to community members per visibility configuration
- Reputation does not appear within communities as visual hierarchy by default
### 15.5 Payment-related reputation signals [Phase 3]
 
- Payment processor events generate reputation signals about the paying entity
- Signals captured: chargebacks, fraud flags, account terminations by processors, persistent payment failures
- Signals are signed reputation events from authoritative sources (the operator’s payment adapter)
- Chargeback signals affect community/user reputation (not operator reputation)
- Single events have limited impact; patterns across multiple operators are stronger signals
- Voluntary refunds (operator agreed) do not generate negative signals
- Signals decay over time; old chargebacks weigh less than recent ones
- Operators and alliances configure how to weigh these signals in their trust decisions
- Reputation signals propagate through alliance channels where configured
### 15.6 Operator reputation [Phase 4]
 
- Operator-level reputation tracks operator behavior across hosted communities
- Signals: hosted communities migrating away, persistent service failures, contractual disputes, alliance-level shunning of the operator
- Operator reputation is visible to potential future communities considering hosting
- Operator reputation accumulates over time; brief issues weigh less than persistent patterns
- No platform-level global operator score; reputation is computed by interested parties (communities, alliances, the foundation)
- Operators can publish their own assessment of themselves (claimed practices, certifications)
-----
 
## 16. Templates
 
Templates are composable, allowing communities to mix and match standardized configurations across different aspects of community structure. Templates can be selected independently for each dimension.
 
### 16.0 Basic templates [Phase 1]
 
Phase 1 ships with a minimal but real template system supporting the governance templates from Section 7.0 and a small set of role/feed defaults. The pilot doesn’t get full custom template authoring, but it does get sensible starting configurations.
 
**Templates available in Phase 1:**
 
- Governance templates (Founder-led, Council, Member-vote on major decisions, Consensus — see Section 7.0)
- Role templates: a small set of starter role configurations matching each governance template
- Feed templates: standard feed sets (general discussion, announcements, events, governance)
- Community template combinations: pre-configured combinations like “small congregation” (council governance + clergy/elder/member roles + standard religious community feeds) or “neighborhood group” (member-vote governance + neighbor/coordinator roles + neighborhood feeds)
**Phase 1 template mechanics:**
 
- Communities select template combinations at creation
- Each template is independently customizable within phase-appropriate limits
- Templates ship with the platform (not user-authored in Phase 1)
- Default selections cover the pilot’s likely needs
**Not in Phase 1:**
 
- Custom template authoring (Phase 2)
- Template versioning and update following (Phase 2)
- Template marketplace or distribution (Phase 2)
- Cross-category template composition with arbitrary combinations (Phase 2)
- Author communities maintaining template libraries (Phase 2)
The goal is to give pilot communities working configurations without forcing them to design from scratch, while not committing to the full template ecosystem yet.
 
### 16.1 Template categories [Phase 2]
 
- **Governance templates**: define proposal types, ballot types, tally methods, voting eligibility, quorum requirements, including the asymmetric change thresholds for proposal-type configuration (Section 7.7)
- **Role templates**: define a set of roles with their permissions and assignment mechanisms
- **Permission templates**: define standard permission configurations for common role/feed combinations
- **Feed templates**: define a set of feeds with their configuration
- **Decision procedure templates**: define specific decision-making procedures that can be referenced by governance templates
- **Moderation templates**: define moderation models and rules
### 16.2 Template composition [Phase 2]
 
- A community starts by selecting one template from each category, or accepting defaults
- Templates can be mixed (Presbyterian governance + standard committee role structure + custom feed configuration)
- Each template is independently customizable after selection
- Customizations don’t affect the underlying template (each community has its own configuration derived from but independent of templates)
### 16.3 Template authoring [Phase 2]
 
- Templates can be authored by anyone with appropriate expertise
- Template authoring is itself a community activity (template author communities maintain templates)
- Templates are versioned; communities can choose to follow updates or stay on a specific version
- Templates have descriptions, suggested use cases, and documentation
### 16.4 Standard template library [Phase 2]
 
- Platform provides a baseline library of templates for common cases
- Religious community templates (various traditions)
- Civic association templates (neighborhood, professional, advocacy)
- Cooperative templates
- Educational/academic templates
- Mutual aid templates
- Activist organization templates
- Small group templates (book club, hobby group, etc.)
-----
 
## 17. Discovery and indexability
 
### 17.1 Public content discoverability [Phase 2]
 
- Public feeds are served as standard web pages with stable, permanent URLs
- Public content is indexable by standard search engine crawlers (Googlebot, Bingbot, etc.)
- The platform doesn’t optimize for search rankings; it serves content as standard web pages and lets search engines do their work
- Public feed URLs are designed to be human-readable where possible
### 17.2 Link metadata [Phase 2]
 
- Public pages serve Open Graph tags (og:title, og:description, og:image, og:type, og:url)
- Public pages serve Twitter Card tags for X/Twitter previews
- Public pages serve Schema.org structured data appropriate to content type (Event schema for events, Article for posts, Organization for community pages)
- Standard HTML metadata (title, description, canonical URL)
- Appropriate cache headers for crawler-friendly behavior
- Proper HTTP semantics (200, 301, 404, etc.)
### 17.3 Public community pages [Phase 2]
 
- Communities have public-facing pages with name, description, location, contact information, public feed list
- Community pages serve as the public web presence
- Communities can configure their public page (subject to platform constraints)
- Public pages link to public content within the community
### 17.4 Sharing [Phase 2]
 
- Public content URLs can be shared anywhere
- Platform provides share-link functionality (copy URL, native share sheet)
- Shared URLs render properly in other platforms via standard metadata
- No platform-specific integrations with external social networks
- No cross-posting features to specific external platforms
### 17.5 Community discovery [Phase 2]
 
- Locality registries (specialized alliances) curate community lists per locality
- Topic-based alliances curate communities by purpose or affinity
- Cross-registry alliances enable broader discovery
- Direct URL access to communities (if known)
- Search engine discovery of public community pages
### 17.6 What discovery does not include [Phase 2]
 
- No platform-level recommendation engine
- No algorithmic suggestion of communities to users
- No engagement-optimized content surfacing
- No advertising or sponsored discovery
- No analytics or feedback loops from search performance into platform behavior
-----
 
## 18. Archival and retention
 
### 18.1 Retention policies [Phase 1]
 
- Communities configure default retention periods
- Feeds can override community defaults
- Content types can have different retention defaults
- Retention applies to: messages, reactions, events, governance records
### 18.2 Content lifecycle states [Phase 2]
 
- Active: appears in normal views, fully accessible
- Archived: preserved but not in default views, accessible via explicit request
- Deleted: removed from server storage; may persist on member devices
### 18.3 Automatic transitions [Phase 2]
 
- Age-based archival (content older than X moves to archived)
- Activity-based archival (inactive feeds’ content moves to archived)
- Retention-policy based deletion (archived content older than Y deleted)
### 18.4 Manual archival [Phase 2]
 
- Members with appropriate permission can archive specific content
- Members can save content client-side before server-side deletion
- Communities can archive entire feeds via governance
- Important governance decisions can be marked for indefinite retention
### 18.5 External publication [Phase 2]
 
- Specific content can be published to durable external storage
- External publication creates content-addressed permanent record
- Publication is a deliberate act with appropriate governance
- Published content remains accessible even after server-side deletion
### 18.6 Member-controlled retention [Phase 2]
 
- Members can mark their own content for retention regardless of community defaults
- Members can request deletion of their own content (subject to community indelibility rules)
- Members can export their participation history at any time
-----
 
## 19. Privacy and metadata
 
### 19.1 Server visibility [Phase 1]
 
- Server sees: mailbox UUIDs, encrypted message volume per mailbox, timing of operations
- Server does not see: user identities (beyond authentication keys), community names, community categories, content, who wrote what, who’s reading what, social graph structure (for non-public content)
- Server cannot perform content-based moderation
- Server cannot deplatform based on community type
### 19.2 Mailbox model [Phase 1]
 
- No per-user inboxes
- Mailboxes are per-relationship or per-feed
- Server sees only “messages arrived at this mailbox”
- Mailbox UUIDs rotate to defeat correlation
- Mailbox creation is cheap and unattributable
### 19.3 Traffic analysis resistance [Phase 2]
 
- Mailbox rotation as primary defense
- Optional message batching to defeat timing correlation
- Optional cover traffic for high-threat scenarios
- Client-side filtering rather than server-side queries
- Encryption of metadata where possible (reactions, etc.)
### 19.4 Threat model [All phases]
 
- Server operator is honest-but-curious in primary threat model
- Compromised server operator cannot read content but can deny service
- External attacker without server access cannot read content or determine membership
- Compromised member’s keys do not retroactively expose non-retained content (forward secrecy via MLS epoch key deletion). For content in the institutional archive (Section 5.9), forward secrecy is deliberately and bounded-ly waived — retained content is protected by threshold-shared access control and audit, not by FS. The sacrifice is bounded, write-forward, threshold-gated, and audited.
- Compromised member’s keys do not permanently expose future content (post-compromise security via MLS Update)
### 19.5 Public content privacy implications [Phase 2]
 
- Content posted to public feeds is by definition not private
- Members posting to public feeds are notified that content is public
- Public content can be indexed, archived, quoted externally — same as any public website
- Communities choose what content is public; default is private
### 19.6 Operator correlation [Phase 2]
 
The platform protects different things to different degrees from the hosting operator:
 
**Strongly protected:**
 
- Content of any community communication (E2E encrypted via MLS)
- Member identities within communities (community membership rosters are not directly visible to operators)
- Credential issuance events (issuance happens through encrypted MLS channels; operator does not observe issuance)
- Internal credential presentation (credentials presented within communities are encrypted)
- Per-action authorship within encrypted contexts
**For communities without public faces (no public feeds):**
 
- Community-level pseudonymity is strong
- The operator sees MLS groups exist with members but cannot link MLS groups to a community identity through observable cryptographic information
- The community keypair signs credentials within encrypted channels; the operator does not observe the community-level linkage
- This is the strongest privacy property the architecture provides
**For communities with public faces (public feeds, registry listings):**
 
- Public content reveals the community’s public identity by design
- Publisher posts include authorization proofs that link to the community’s keypair (standard credential model)
- The operator can correlate the community’s MLS groups to its public identity via publisher credentials
- This is inherent to having a public identity; cryptographic mitigation requires issuer-hiding credentials (see Section 12.4a, Phase 5)
**Inherent limits applying everywhere:**
 
- Timing correlation: simultaneous activity across MLS groups can be correlated by statistical analysis
- Device patterns: the same client device connecting to multiple mailboxes correlates membership
- Resource patterns: large communities use proportionally more resources
- Media access patterns: media accessed on every message render (like avatars) is distinguishable from media accessed once (like attachments); the same user’s avatars across multiple communities are correlatable via access patterns even though ciphertext differs
- Out-of-band information: knowledge from other sources can defeat any cryptographic mitigation
The protection model is “inconvenient, not impossible.” A determined operator with sufficient resources can defeat the protections through statistical analysis. The architectural protections raise the cost of correlation without making it impossible. Communities needing absolute community-level pseudonymity should host their own instance with a trusted operator.
 
### 19.7 Legal and compliance [All phases]
 
- Server retains no information that would identify users to legal compulsion
- Subpoenas can produce metadata but not content or identities (for private content)
- Communities can configure for jurisdictions with specific requirements
- Right-to-be-forgotten supported by member-controlled deletion
-----
 
## 20. Instance operation
 
An instance is composed of two distinct but related entities: an operator entity (legal and technical authority) and an instance alliance (governance layer). This split separates decisions that require physical control of infrastructure from decisions that can be delegated to community governance.
 
### 20.1 Operator entity [Phase 1]
 
- The party legally and technically responsible for running the instance
- Holds infrastructure access, deployment authority, legal accountability
- Identified by an operator keypair (held by an organization or person)
- Authorized by the instance alliance through a signed attestation
- Cannot be delegated; the operator must be a concrete legal entity
- Responsibilities: infrastructure decisions, software updates, legal compliance, hosting capacity, funding, hiring (if applicable), continuation of operation
- Operator transitions are alliance actions — the alliance signs a new attestation transferring authorization to a new operator entity (see Section 20.17)
### 20.2 Instance alliance [Phase 1]
 
- The governance layer for the instance
- An MLS group whose member communities are the hosted communities of the instance
- One share per community; alliance actions require threshold multi-signature (default 2/3 of representative composition)
- Configurable governance: from benevolent dictator (single-community instance, founder makes decisions) to cooperative (multiple representative communities vote)
- Holds the instance’s federation identity collectively; threshold of representatives sign federation operations
- Authorizes the operator entity through signed attestation
- Responsibilities: terms of service, federation policy, moderation policy, dispute resolution, public statements, content policies for public feeds, operator authorization
### 20.3 Instance alliance anonymization [Phase 2]
 
- The instance alliance is structurally distinct from other alliances because the operator participates
- Member communities appear in the instance alliance under cryptographic identifiers distinct from their identifiers in other contexts
- Default visibility is opaque: communities appear as cryptographic IDs only
- Communities can choose alliance-specific handles through their own governance — opaque by default, revealing if they choose
- Operators see alliance state by alliance-specific identifiers, not by full community identities
- Participation happens through designated representatives, not as full communities
- Representatives discuss governance matters in alliance feeds without revealing their communities’ substantive activities
- The protection is “inconvenient, not impossible” — consistent with the platform’s broader pseudonymity model
- Timing correlation, resource patterns, and out-of-band information can still defeat anonymization for a determined hostile operator
- The architectural mitigation reduces casual visibility; operator trust remains the ultimate protection
### 20.4 Instance alliance moderation authority [Phase 2]
 
- The instance alliance has moderation authority above individual community governance
- Scope: anti-CSAM and other illegal content rules, compliance with applicable law, detection and removal of obvious abuse, decisions about communities violating broadly-shared norms, federation policy
- Moderation is reactive, not proactive: cannot scan content (E2E encryption prevents this), but can act on member reports with cryptographic proof
- Moderation council composed of representatives of member communities reviews reports and decides actions
- Possible actions: warnings, restrictions on access, removal from the instance, reporting to authorities
- These decisions are made through alliance governance, not unilaterally by the operator
### 20.5 Operator-alliance relationship [Phase 1]
 
- Operator entity is authorized by the alliance through signed attestation
- Operator transitions are alliance actions (threshold of representatives sign new attestation)
- Alliance identity is collectively held and durable across operator changes (federation identity stable)
- Most decisions are alliance-layer (governance-delegatable)
- A narrow set of decisions are operator-layer (irreducible to governance: infrastructure operation within authorization scope)
- The operator cannot unilaterally change their own authorization or claim alliance authority
- The alliance cannot unilaterally take over infrastructure but can revoke operator authorization and authorize a different operator
### 20.6 Default configuration [Phase 1]
 
- New instances default to operator as sole admin in the instance alliance
- Instance functions as traditional operator-led hosting in this mode
- All governance primitives exist but governance template is “benevolent dictator” with operator as dictator
- Operator can adjust governance configuration to democratize specific decision types
- Member communities can be granted voting rights, council seats, juror eligibility, etc. through governance template selection
### 20.7 Cooperative instances [Phase 2]
 
- Operators can configure the instance alliance for full cooperative governance
- Member communities have voting rights on alliance decisions per the configured template
- The operator entity remains for infrastructure responsibilities
- Demonstrates the democratically-governed instance model
### 20.8 Central instance [Phase 2]
 
- Platform foundation runs a default central instance
- Foundation is the operator entity for infrastructure responsibilities
- Instance alliance is configured for democratic governance by member communities
- Central instance is a default, not a destination
- No special features or pricing
- Explicit policies to limit dominance
- Can serve as failsafe for community migrations
- Demonstrates the cooperative model in practice
### 20.9 Public instance metadata [Phase 2]
 
- Each instance serves a public description accessible via standard web request
- Metadata includes: operator identity, alliance governance configuration, jurisdiction, terms of service, pricing, federation policy, contact information
- Metadata changes are versioned and dated
- Users and operators evaluate instances against their preferences using this metadata
- Metadata is served as standard web content (no federation required to view)
### 20.10 Multi-tenant operation [Phase 2]
 
- A single instance can host any number of communities
- Communities on the same instance don’t share state beyond what they explicitly share
- Resource isolation prevents one community from impacting others
- Instance operators cannot read community content (E2E encrypted)
### 20.11 Resource management [Phase 3]
 
- Operators set their own pricing, capacity limits, hosting policies
- Operators can decline to host based on resource limits
- Operators cannot decline based on content (cannot inspect content)
- Operators can decline based on alliance governance decisions (e.g., alliance has shunned a specific entity)
- Communities pay dues (if any) per the instance’s configuration
### 20.12 Community migration between instances [Phase 4]
 
- Communities can migrate between instances
- Migration is atomic with grace-period backups
- Migration preserves identity, content, alliances, federation, governance
- Migration tools are first-class, not afterthoughts
- When a community migrates to a new instance, it becomes a member of the new instance’s alliance and ceases to be a member of the old instance’s alliance
- Instance alliance composition changes are alliance actions on both sides (community joining, community leaving)
### 20.13 Instance reputation [Phase 4]
 
- Moderation records include the originating instance of affected users
- Instance operators and alliance members can aggregate moderation outcomes by source instance
- Aggregated data informs federation policy decisions
- Alliance member communities can share instance-level moderation data through alliance channels
- Instances may optionally publish their assessments of peer instances as signed public content
- Automated pattern detection can flag instances with concerning behavior for human review
- All reputation data is owned by the entity computing it; publication is voluntary
- No platform-level global reputation system; reputation is distributed and pluralistic
### 20.14 Defederation [Phase 4]
 
- An instance can defederate from a peer at any time
- Defederation is an alliance-layer decision (federation policy)
- Operator entity executes the decision (configures the systems)
- Defederation is unilateral with no negotiation required
- Local consequences: users on this instance lose access to communities on the defederated peer
- Communities on the defederated peer continue operating; they’re just unreachable from this instance
### 20.15 Operator relationships [Phase 3]
 
- Operator-community relationships are contractual, not protocol-enforced
- The platform provides standard operator agreement templates that operators can use
- Communities paying dues have a contractual relationship with their operator (subject to local law)
- Breach of operator obligations is recourse through civil court, not protocol
- The foundation can publish recommended operator standards (voluntary)
- Operators who adopt the standards appear differently in operator directories
- This is soft power: communities can prefer compliant operators, but the foundation has no direct authority
### 20.16 Trust limits with operators [Phase 4]
 
- Operators have irreducible powers: data hosting, service continuity, processing payments, technical infrastructure
- The platform cannot fully prevent operator misbehavior
- Architectural mitigations: portability, replication, cryptographic identity, event sourcing, reputation tracking
- Communities can migrate away from bad operators (real cost, but bounded)
- Members’ local replicas and credentials remain valid through operator changes
- Legal recourse (civil court, regulatory action, criminal prosecution where applicable) is the ultimate backstop
- This is a known trust requirement; the platform makes it as small and recoverable as possible
### 20.17 Operator transitions [Phase 4]
 
Operator transitions are alliance actions. The alliance signs an attestation transferring operator authority from one entity to another. The protocol handles cooperative, hostile, and abandoned-operator cases.
 
**Cooperative case:**
 
- Outgoing and incoming operators coordinate the change operationally
- Alliance representatives sign transition attestation on their own devices
- Incoming operator coordinates signature collection from representatives
- When threshold (default 2/3) reached, combined attestation is published to federated peers
- Peers verify against current alliance composition and update operator authorization
**Hostile case (operator refuses to cooperate):**
 
- Representatives recognize operator has gone hostile
- New operator identified (volunteer or chosen by alliance governance)
- Representatives sign on their own devices, not through hostile operator’s infrastructure
- Local MLS state copies provide alliance composition reference (representatives reconcile divergent state if operator suppressed updates)
- New operator coordinates signature collection out-of-band
- New operator begins serving from allied replica, backup, or fresh setup
- Communities that did not participate in migration remain on original infrastructure under original operator, forming effectively a new instance (continuity loss for dissenters, consistent with schism pattern)
**Abandoned case (operator has disappeared):**
 
- Same protocol as hostile case
- Data recovery depends on allied replicas or user-side backups since original instance may be unrecoverable
- This is why allied replicas matter as protection against sudden operator loss
**Composition changes:**
 
- Adding/removing representative communities is itself an alliance action through MLS proposals
- Composition changes propagate to federated peers through signed announcements
- Peers verify against previous epoch’s composition; chain of updates is verifiable from any known starting point
**Hardware migration:**
 
- Separate from operator transition but often coincident
- If operator is unchanged: data migration is operational; no alliance signatures needed
- URL changes: signed announcement from current operator, verifiable by alliance authorization
- If hardware change coincides with operator change: combined attestation handles both
**Limits:**
 
- Threshold compromise: if more than 1/3 of representatives are compromised, alliance can be captured
- Sufficient representatives offline: alliance can deadlock; recourse is schism path
- Bootstrap period: very early instances with few representatives have weaker protection
- Total infrastructure failure: degraded recovery through user-side backups
-----
 
## 21. Integrations
 
Most integrations with external systems are implemented through the platform’s adapter and extension point patterns. The adapter pattern (already established for payment adapters in Section 27) extends to other integration categories. Specific implementations of these integrations are plugins from Phase 5 onward; the adapter infrastructure that makes them possible is built earlier where needed.
 
### 21.1 Calendar export [Phase 2, full plugin support Phase 5]
 
- iCal feed URLs for personal, community, and feed-specific calendars
- Authentication on subscription URLs
- Updates reflected in subscribing calendar apps
- Standard format compatible with major calendar applications
- Basic iCal export ships as platform feature in Phase 2; richer calendar integration patterns (CalDAV, two-way sync, etc.) become plugins in Phase 5
### 21.2 Push notifications [Phase 1 infrastructure, Phase 5 policy plugins]
 
- APNs for iOS
- FCM for Android
- Web push for browser-based access
- Notification content encrypted; decryption happens on device
- Notification delivery infrastructure ships Phase 1 (essential for mobile UX)
- Notification policy (what to push, when, formatting, batching) is community-configurable from Phase 1; custom notification policy modules become plugins in Phase 5
### 21.3 Data export and import [All phases — see CCF-11]
 
- Users can export their data at any time in standard format
- Export includes credentials, message history, governance participation
- Export does not include content others have not shared with the exporting user
- Users can import previously exported data on new device
- Import preserves identity continuity through cryptographic verification
- Import can be partial (specific communities, specific date ranges)
- This is structural commitment, not a Phase 5 feature; see cross-cutting foundations
### 21.4 Web standards integration [Phase 1-2, with public web pages]
 
- Standard HTML/CSS/JS for public pages
- Open Graph metadata for link sharing
- Twitter Card metadata for X/Twitter previews
- Schema.org structured data for content types
- iCal for calendar export
- W3C Verifiable Credentials for credentials
- Standard HTTP semantics for crawlers and clients
- These are part of how public web pages work, not separately phased
-----
 
## 22. Constraints
 
### 22.1 End-to-end encryption [All phases]
 
- Content is end-to-end encrypted at all times within the platform
- Server operators cannot read content under any circumstances
- E2E is structural, not configurable
- Exception: content posted to public feeds is by definition public and not E2E protected
- Alliance messaging uses a two-layer design: MLS for representative governance, sender-key protocol for broader alliance content visible to all members of constituent communities (see Section 13.7); this matches the social reality that broad participation can’t deliver secrecy from members while still providing boundary confidentiality and sender authentication
- All non-public features must function within E2E constraints
### 22.2 No central authority [All phases]
 
- No single entity has platform-wide moderation power
- No single entity can deplatform users globally
- No single entity controls discovery
- Foundation governance includes mechanisms preventing capture
### 22.3 Community sovereignty [All phases]
 
- Communities define their own rules within protocol constraints
- Platform provides mechanisms, communities configure policy
- Communities can leave any instance, registry, or alliance
- Communities can dissolve themselves
### 22.4 No engagement optimization [All phases]
 
- No algorithmic feeds optimizing for time-on-platform
- No recommendation engines pushing content
- No notifications designed to maximize attention
- Defaults favor calm over engagement
### 22.5 No advertising [All phases]
 
- No ads anywhere in the platform
- No sponsored content
- No data sales to advertisers
- No engagement-based revenue
- Revenue from community dues and credential production
- The platform processes payments where it is the payee (dues, credential orders)
- The platform does not process payments for community-level commerce (ticketing, fundraising); those use external processors
### 22.6 In-person trust foundation [All phases]
 
- Joining communities requires in-person QR/NFC trust establishment
- No remote-only joining workarounds
- Accessibility cases handled through vouching by members who established in-person trust
- This commitment is foundational; there is no configuration that relaxes it
### 22.7 Accessibility [All phases]
 
- Platform usable by people with various disabilities
- Screen reader compatibility
- Keyboard navigation
- Text alternatives for media
- Configurable text size and contrast
### 22.8 Mobile-first [All phases]
 
- Primary platform is mobile (iOS and Android)
- Desktop access via web or native client
- All features available across platforms
- Cross-platform feature parity
### 22.9 Offline tolerance [All phases]
 
- Client functional with intermittent connectivity
- Queued operations sync when connection restored
- Local cache of recent content for offline viewing
- Clear indicators of sync state
### 22.10 Internationalization plumbing [All phases]
 
- All user-visible strings externalized for translation from Phase 1
- Date, time, number, and currency formatting use locale-aware libraries from Phase 1
- Text direction (LTR/RTL) handled in the UI layer from Phase 1
- Standard internationalization patterns (ICU MessageFormat, etc.)
- Specific language translations beyond English are Phase 5 deliverables
- This is structural: building i18n in later means refactoring every UI surface
-----
 
## 23. Security properties
 
### 23.1 Confidentiality [All phases]
 
- Content readable only by intended recipients (for non-public content)
- Membership in restricted feeds known only to members
- Cryptographic identity tied to user-controlled keys
- Server-side data is opaque encrypted blobs for non-public content
### 23.2 Integrity [All phases]
 
- Messages signed by senders within encrypted layer
- Tampering detectable via signature verification
- Group state changes authenticated via MLS commits
- Audit logs immutable for governance and moderation actions
### 23.3 Forward secrecy [All phases]
 
- For non-retained content: past messages unreadable if current keys compromised; achieved via MLS epoch keys deleted after use
- For retained content (institutional archive, per Section 5.9): forward secrecy is deliberately and bounded-ly waived — retained content is protected by access control and audit (threshold-shared archive key, every access logged to verifiable infrastructure), not by FS
- The FS sacrifice is bounded (only retention-configured feeds), write-forward (content encrypted to the archive key at post time, never by retaining epoch keys), threshold-gated (no single party can open the archive), and audited (every access undeniable)
- Onboarding context (per Section 5.9) is a separate, lighter waiver — time-bounded and self-healing (a joiner sees the recent window; later the content ages out)
- This carve-out states plainly what’s possible: no construction gives both “recoverable after losing only device” and “cryptographically unrecoverable after epoch keys are deleted” — these are contradictory. The work is bounding the FS sacrifice and making it visible and governed.
### 23.4 Post-compromise security [All phases]
 
- Future messages unreadable to attacker who compromised past keys
- Achieved via MLS Update proposals and key rotation
- Update cadence configurable by community
### 23.5 Authentication [All phases]
 
- Users authenticated by possession of identity private key
- Devices authenticated by device-specific keys derived from or authorized by identity
- Communities authenticated by community keypair
- Alliances authenticated by alliance keypair
- Issuers authenticated by issuer signing keys
### 23.6 Non-repudiation [All phases]
 
- Signed content cannot be repudiated by signer
- Signed governance votes attributable (for public votes)
- Signed credentials attributable to issuer
- Verified feed content cryptographically anchored
### 23.7 Privacy [All phases]
 
- Server cannot identify users
- Server cannot determine community categories (for private communities)
- Server cannot read content (for non-public content)
- Server cannot perform traffic analysis beyond mailbox-level patterns
- Metadata exposure minimized through mailbox model
-----
 
## 24. Data model entities
 
### 24.1 Core entities [Phase 1]
 
- User
- Device (associated with User)
- Community
- Membership (User in Community)
- Role
- Permission
- Role Assignment (User has Role in Community)
- Feed
- Message
- Reaction
- Event
- RSVP (User to Event)
- Proposal
- Vote
- Decision
- Mailbox (with composite address: cryptographic_entity_id + mailbox_role; authoritative instance is a mutable metadata pointer, not part of the address)
- MLS Group
- Instance (composed of Operator Entity and Instance Alliance — see below)
- Operator Entity (legal/technical authority over an instance; holds operator keypair)
- Instance Alliance (governance layer for an instance; specialized alliance whose members are hosted communities)
- Message Log Entry (signed, persistable record of state changes for event sourcing)
- Profile (per User per Community, with display name, avatar reference, bio, pronouns, contact info)
- Media (per Community, encrypted blob with ciphertext hash, uploader signature, content type, size)
### 24.2 Relationship entities [Phase 2]
 
- Vouch (User vouches for User in Community)
- Alliance (multi-community structure)
- Alliance Membership (Community in Alliance)
- Registry Listing (subset of Alliance Membership for registry alliances)
- Federation (Instance to Instance)
- Federation Relationship (with establishment time, status, established_via reference)
- Schism (Community split from Community)
- Migration (Community moved between Instances)
- Alliance Keypair (Community’s keypair for participation in a specific Alliance)
- Alliance Representative (Community role authorized to operate Alliance Keypair)
- Alliance Epoch State (current MLS state at the alliance level)
- Alliance Key Distribution (record of alliance epoch keys distributed to community members)
- Replica (record that this instance has a local replica of a mailbox owned by another instance)
- Instance Address Mapping (record of remote instance’s current URL, signed by the remote instance)
### 24.3 Credential entities [Phase 2]
 
- Credential (issued by Community/Issuer to User)
- Credential Type
- Issuer
- Trust Relationship (Community trusts Issuer)
- Revocation (Credential revoked by Issuer)
### 24.4 Content entities [Phase 2]
 
- Content (parent type for messages, events, etc.)
- Content State (active, archived, deleted)
- Attachment (media, file associated with Content)
- Reference (Content references other Content)
- Public Content (content with public visibility)
### 24.5 Governance entities [Phase 1]
 
- Proposal Type
- Ballot Type
- Tally Method
- Decision Procedure
- Moderation Rule
- Moderation Action
- Jury (for jury-based moderation)
- Juror Pool
- Template (governance, role, feed, etc.)
- Template Composition (community’s selected templates)
### 24.6 Audit entities [Phase 2]
 
- Governance Record (proposals, votes, decisions)
- Moderation Record (actions taken)
- Membership Record (joins, leaves, role changes)
- Configuration History (changes to community/feed/role configuration)
- Alliance Membership History
### 24.7 Payment entities [Phase 3]
 
- Payment Adapter (configuration for a specific payment processor on a specific instance/community)
- Payment Record (record of a payment processed through the platform)
- Recurring Payment Setup (configuration for recurring charges)
- Refund Record (record of a refund issued)
- Chargeback Record (record of a chargeback received from processor)
- Payment Event (signed event from payment adapter, used for reputation signaling)
- Billing Contact (role within a community whose holder handles payment information)
### 24.8 Ticketing entities [Phase 3]
 
- Ticket Configuration (event’s ticket setup: count, price, sybil tolerance, transfer policy, refund policy)
- Ticket Reservation (timed hold during payment)
- Ticket (credential representing a confirmed ticket purchase)
- Ticket Check-In Record (record of ticket presentation at event)
### 24.9 External integration entities [Phase 3]
 
- Integration Adapter (configuration for an external system integration, e.g., Tithely, manual treasurer flow)
- External Event (event received from an external system, signed by appropriate source)
- Integration Configuration (mapping from external events to platform actions)
-----
 
## 25. Integration points
 
### 25.1 Cryptographic libraries [Phase 1]
 
- MLS implementation (OpenMLS or equivalent)
- Verifiable credentials library
- Standard primitive libraries (libsodium, signing, hashing)
- Zero-knowledge proof library for selective disclosure
### 25.2 External services [Phase 3]
 
- Push notification services (APNs, FCM, Web Push)
- Email for backup/recovery only (not authentication)
- Time servers for trusted timestamps
- DNS for instance discovery
- Content-addressed storage for verified public content (IPFS or equivalent)
- Payment processors (Stripe, PayPal, regional providers, etc.) via pluggable adapters
### 25.3 Standards compliance [Phase 1]
 
- W3C Verifiable Credentials
- IETF MLS (RFC 9420)
- iCal/CalDAV for calendar export
- OAuth 2.0 / OpenID Connect for outbound auth
- SAML 2.0 for enterprise SSO
- Open Graph for link metadata
- Schema.org for structured data
- ActivityPub bridge (optional, for federation with broader fediverse)
- Matrix bridge (optional, for migration and interop)
- PCI-DSS compliance for payment adapters (handled by processors; operators must use compliant adapters)
### 25.4 Client integration [Phase 1]
 
- Operating system contacts (with explicit user permission)
- Operating system calendar (export only)
- Operating system camera (for QR codes, media capture)
- Operating system file system (for attachments, exports)
- Operating system share sheet (for sharing public URLs)
### 25.5 Operator infrastructure [Phase 1]
 
- Database (Postgres or equivalent)
- Object storage for media (S3-compatible)
- Backup services
- Monitoring and observability
- CDN for public content (optional, performance)
-----
 
## 26. Creator and audience roles
 
Several use cases initially considered for a separate “subscription” primitive — news organizations, artist collectives, political accountability feeds, religious publications, educational content — are better served by the platform’s existing role and feed primitives. Communities can define internal roles (e.g., “creator” and “audience”) with asymmetric permissions: creators have editorial authority and governance weight; audience members have read access to designated feeds but no editorial voice or voting rights on creator decisions.
 
This pattern reuses existing primitives:
 
- Role-based feed access (different roles see different feeds)
- Role-based voting weight (audience members may have zero weight in editorial decisions)
- Role-based credentialing (becoming a creator requires the community’s admission process)
- Configurable governance per community (the community decides how creators are admitted, how audience members participate, etc.)
The pattern serves the use cases without requiring new platform primitives:
 
- **News organizations** — journalists/editors are creators with editorial authority; readers/subscribers are audience members with access to published feeds
- **Artist collectives** — admitted artists are creators with curatorial and governance authority; followers/patrons are audience members with access to portfolio and exhibition feeds
- **Religious publications** — staff are creators producing content; readers are audience members receiving it
- **Educational content** — instructors are creators; students are audience with possibly some interaction roles
- **Political accountability feeds** — officials are creators publishing positions; constituents are audience members holding accountable through external mechanisms
What this is not: a new primitive. Communities that want subscription-payment-with-content-access patterns can compose existing payment adapter, credential, and role primitives. Subscription-specific mechanics (trials, promotional pricing, dunning workflows) can be implemented as plugins when communities want them.
 
What this is: an explicit articulation that communities are not monolithic — they can have internal role differentiation that supports many of the patterns previously imagined as needing separate community-to-community relationships.
 
The platform’s commitment is to support role-based feed access and role-based voting weight from Phase 1 (these are basic features of the governance and feed systems). Specific compositions for specific use cases are community decisions, supported by templates (Section 16) and potentially by plugins from Phase 5.
 
-----
 
## 27. Payments and external integrations
 
Payments on the platform fall into distinct categories with different platform involvement. The platform handles only what’s necessary; payment processing is offloaded to third-party providers via pluggable adapters. The platform never takes a transaction fee.
 
### 27.1 Payment categories [Phase 3]
 
**Platform processes payment for platform-owned resources:**
 
- Community dues to instances
- Credential production orders
- In all these cases, the platform owns the resource (knows what’s being sold, to whom, with what constraints) and uses a payment adapter for the money leg
**Platform receives signals from external systems (no money flows through platform):**
 
- Good standing tracking based on giving through external systems (Tithely, etc.)
- Attendance credentials based on off-platform payment confirmed by treasurer
- Any credential whose triggering payment happens entirely outside platform awareness
- Platform receives webhook/polling/manual confirmation, issues credential
**Platform stays out (no involvement):**
 
- Community-level commerce not tracked by the platform (general fundraising, member-to-member transactions)
- Communities use external systems directly
### 27.2 Pluggable payment adapters [Phase 3]
 
- The platform defines a payment adapter interface
- Operators and communities choose which payment processors to support
- Each adapter implements the interface for its specific processor
- Adapters exist or can be written for: Stripe, PayPal, Cash App, Venmo, Adyen, Square, Razorpay, regional providers, cryptocurrency processors, manual/check
- New adapters can be added without changing platform core code
- Communities and operators are not locked to a specific processor
- Different communities on the same instance can use different processors
### 27.3 Adapter interface [Phase 3]
 
The interface covers operations the platform requires:
 
- Initiate payment (one-time or recurring)
- Verify payment status
- Set up recurring payments
- Cancel recurring payments
- Issue refunds
- Handle webhooks from the provider
- Translate provider-specific events to platform events
- The platform code only knows the adapter interface, not provider specifics
### 27.4 Architectural commitment: no transaction fees [Phase 1]
 
- The adapter interface is structured such that the platform cannot intercept funds
- Money flows from payer to recipient’s account directly via the processor
- The platform sees the payment occurred (via webhook) but does not handle the money
- This is architecturally enforced rather than promised
- Future changes that would enable transaction fees would require fundamental restructuring (subject to foundation governance constraints)
### 27.5 Third-party processor as anonymity layer [Phase 3]
 
- The operator does not hold credit card information; the payment processor does
- The operator sees “this account paid” (via processor token); not raw payment details
- Linking real-world payment identity to platform identity requires both the operator’s data and the processor’s data
- A subpoena to the operator yields only “this account paid via [processor]”; the actual identifying information requires a separate legal action against the processor
- The processor anonymizes payments at the operator layer under normal circumstances
- Compelled disclosure from both processor and operator can defeat this; the protection is meaningful but not absolute
### 27.6 Recurring payments [Phase 3]
 
- Dues use recurring payment setups
- Recurring payment state is managed by the processor (via adapter)
- Platform stores dues state derived from processor events
- Failed payments trigger configured recovery flow
### 27.7 Privacy and pseudonymity [Phase 3]
 
- The community/user remains pseudonymous to the operator for non-payment context
- Payment necessarily involves identifying information at the processor
- The link between payment identity and platform identity is broken at the operator level
- Billing contact role: a specific role within the community whose holder handles payment; this role’s holder has identifying information visible to the processor but other community members do not
### 27.8 Operator and community responsibilities [Phase 3]
 
- Whoever receives the payment is the merchant of record (operator for dues, community for tickets/events)
- Merchant of record must be a legal entity capable of receiving payments
- Merchant handles tax compliance, reporting, accounting for received funds
- Merchant manages relationship with their chosen payment processor(s)
- Merchant’s legal jurisdiction determines applicable consumer protection laws
### 27.9 Multi-currency support [Phase 3]
 
- Adapters can handle multiple currencies per their processor’s support
- Prices are denominated in specific currencies
- Cross-currency transactions handled per processor capabilities
### 27.10 Refunds and chargebacks [Phase 3]
 
- Refunds initiated by recipient through their processor (via adapter)
- Platform records refund and updates relevant access state (revokes credential, removes ticket, etc.)
- Chargebacks initiated by the payer’s bank/processor
- Platform receives webhook, immediately revokes access, notifies recipient
- Recipient decides whether to contest the chargeback
- Chargebacks generate reputation signals about the disputing entity (see Section 15.5)
### 27.11 Failed payments [Phase 3]
 
- Adapter detects payment failure
- Platform initiates configured recovery flow
- Persistent failures result in service degradation per configuration
### 27.12 External integration adapters [Phase 3]
 
For cases where payment happens outside the platform’s awareness, the platform supports integration with external systems via three tiers:
 
**Tier 1: Webhook-based (real-time):**
 
- External system pushes events to the platform via HTTPS webhooks
- Available for: Stripe Billing, Eventbrite, GivingFire, and most modern payment systems
- Platform validates webhook signatures per the external system’s documentation
- Events trigger credential issuance, status updates, or other platform actions
- Best UX; near-real-time updates
**Tier 2: Polling-based (delayed):**
 
- Platform periodically queries external system’s API
- Used when the external system has APIs but no webhook support
- Available for: Tithely and similar polling-friendly systems
- Less real-time but still automated
- Configurable polling frequency
**Tier 3: Manual confirmation:**
 
- Designated role (typically community treasurer) manually confirms events through platform UI
- Works for any payment method including cash, check, off-platform digital payments
- Always available as fallback when no automation is possible
- Audit trail of manual confirmations
### 27.13 External integration use cases [Phase 3]
 
- Good standing tracking from external giving platforms (Tithely → “in good standing” credential renewal)
- Attendance credentials from off-platform event payment (treasurer confirms → attendance credential)
- Donor recognition credentials from external donation tracking
- Any credential whose triggering action happens outside the platform’s awareness
### 27.14 Ticketing [Phase 3]
 
- Tickets are platform-native: the platform owns the inventory, enforces sybil resistance, and is authoritative for who has a ticket
- Payment for tickets uses standard payment adapters (Stripe, etc.)
- Eventbrite-style external ticketing systems are not used; the platform implements ticketing directly
- This avoids cross-system reconciliation problems (overselling, identity verification mismatches)
- Communities focused on selling to non-platform users can still use external ticketing systems alongside platform credentials; integration adapters can be written for specific external ticketing platforms if needed (not core architecture)
#### 27.14.1 Ticket flow
 
1. Community creates event with ticket configuration (count, price, sybil tolerance, transfer policy, refund policy)
1. User clicks “Buy ticket”
1. Platform verifies user’s identity meets configured sybil tolerance
1. Platform checks ticket availability and any per-identity limits
1. Platform reserves a ticket slot (timed, default 15 minutes)
1. Platform initiates payment via the community’s configured adapter
1. User completes payment in adapter’s hosted flow (e.g., Stripe Checkout)
1. Adapter webhook returns to platform
1. Platform matches webhook to reservation, converts to confirmed ticket
1. Platform issues ticket credential to user’s identity
1. User sees ticket in their credential wallet
#### 27.14.2 Ticket properties
 
- Tickets are credentials cryptographically bound to the purchaser’s identity
- Non-transferable by default (transfer policy configurable per event)
- Verifiable at check-in (scan credential, verify signature, mark attended)
- Attendance can be tracked separately (credential presented = ticket valid; checked-in = attended)
- Refundable per event’s refund policy; refund revokes credential
#### 27.14.3 Sybil resistance for tickets
 
- Tickets are anonymous credentials with per-event nullifiers built on the same BBS per-verifier-pseudonym primitive used for voting
- Sybil tolerance configures which `pid` anchors the per-event nullifier — account `pid` (permissive), phone-verified `pid` (moderate), decency `pid` (“one per community member”), humanness `pid` (strict “one per person”)
- Common configurations: one ticket per cryptographic identity, one ticket per phone-verified identity, one ticket per community member, one ticket per person
- Per-event cardinality (N tickets per person) is enforced by application-layer counting at the ticketing authority — same construction as one-vote-per-decision in governance; voting is the cap-1 case, ticketing is the cap-N case
- Configuration applies at reservation time before payment
- Failed verification means user can’t reserve; they’re not charged
- This avoids the “paid but rejected” problem of post-payment verification
#### 27.14.4 Reservation timeouts
 
- Reservations expire after configured period (default 15 minutes)
- If payment doesn’t complete, slot returns to available inventory
- If webhook arrives after expiration but slot is still available, platform issues ticket anyway
- If webhook arrives after expiration and slot is gone, platform alerts community for manual reconciliation (refund or restore inventory)
#### 27.14.5 Free tickets
 
- Same flow without the payment step
- Reservation → confirm directly → issue credential
- Sybil tolerance still enforced (often the main point of free-but-limited events)
- Useful for limited-capacity events that aren’t selling tickets (capped RSVP, member-only access lists, etc.)
### 27.15 Compliance considerations [Phase 3]
 
- Payment processing is subject to consumer protection law in recipient’s jurisdiction
- EU and California have particularly strong consumer rights frameworks
- Merchants of record are responsible for legal compliance in their jurisdiction
- Platform supports compliance (e.g., disclosure mechanisms, refund window configuration)
- Some jurisdictions may require money transmitter licensing depending on payment flow structure
- Legal review recommended before production deployment of any new instance handling payments
### 27.16 Adapter extension points [Phase 5 plugin candidates]
 
The payment adapter pattern is one of the platform’s earliest plugin-shaped extension points. Additional adapter types and specific implementations are good candidates for the Phase 5 plugin system:
 
- Privacy-proxy adapters (an intermediary that mediates between payer and recipient with anonymization)
- Cryptocurrency processor adapters
- Alternative banking adapters (specific to jurisdictions or banking systems not covered by major processors)
- Member-credit / mutual-credit systems (for cooperatives that want internal accounting)
- Donation-platform adapters (specific to donation-focused processors)
- These are not platform commitments to build; they are extension points where the architecture supports community-developed adapters when communities want them
-----
 
## 28. Implementation stack
 
The platform is built on a small set of foundational technology choices. These are fixed; patterns within them are left to implementation.
 
### 28.1 Backend: Elixir / Phoenix [Phase 1]
 
- Phoenix framework for HTTP and WebSocket handling
- Phoenix Channels for real-time message push to connected clients
- OTP supervision trees for fault tolerance
- Per-connection, per-community, per-feed process isolation
- Built-in clustering support for horizontal scaling when needed
- Phoenix LiveDashboard for operational observability
- REST endpoints for request/response APIs; Channels for real-time bidirectional communication
- LiveView for server-driven UI on pages that benefit from it
### 28.2 Web client: React with live_react SSR [Phase 1]
 
- React components written in TypeScript
- live_react integration with Phoenix for server-side rendering
- Components render server-side via Node.js worker pool supervised by Phoenix
- Browser hydrates server-rendered HTML and takes over interactivity
- Same React components handle both public and authenticated rendering
- Public content rendered server-side with full content (indexable by search engines)
- Authenticated content rendered server-side as shell; encrypted content fetched and decrypted client-side after hydration
- Components written in SSR-compatible style throughout (no direct window/document access, proper hydration semantics)
- Single deployment unit; Node workers are part of the Phoenix application
### 28.3 Mobile client: React Native [Phase 1]
 
- React Native with TypeScript
- New architecture (Fabric/JSI) from the start
- Shares business logic, hooks, type definitions, and state shape with web React
- Visual implementation separate (React Native components vs React DOM)
- Talks to Phoenix via REST and Channels (same APIs as web client)
- FFI to Rust crypto via uniffi-rs or similar binding generator
- Native platform integration for push notifications, camera, file system, share sheet
### 28.4 Shared client patterns [Phase 1]
 
- TypeScript across web and mobile
- React Context for state management (multiple small contexts, not one monolith)
- Custom hooks encapsulate crypto operations, API calls, real-time subscriptions
- Type definitions shared between web and mobile clients
- Business logic in framework-agnostic modules where possible
- Visual components separate per platform; behavior and state shared
### 28.5 Design system [Phase 1]
 
- Tailwind CSS for styling
- Tailwind config as source of truth for design tokens (colors, spacing, typography, etc.)
- Phoenix templates use Tailwind classes directly
- React web components use Tailwind classes
- React Native uses NativeWind (Tailwind for React Native) for consistent design tokens
- Visual consistency across all client surfaces through shared design tokens
### 28.6 Cryptography: Rust (OpenMLS) [Phase 1]
 
- OpenMLS or equivalent Rust MLS implementation
- Compiled to multiple targets from one source: WASM (web), iOS native library, Android native library, native library (backend via NIFs or Ports)
- One implementation eliminates protocol divergence across platforms
- Custom credential type integrating with platform’s identity layer
- Same library handles verifiable credentials, signatures, primitive cryptography
- Backend integration via NIFs (performance) or Ports (safety) — implementation decision
### 28.7 Database: PostgreSQL [Phase 1]
 
- Relational storage for application data
- Stores encrypted blobs, not plaintext content
- Holds metadata: mailbox identifiers, community structure, role assignments, governance state, federation links, alliance structure
- Standard Phoenix Ecto integration
### 28.8 Object storage: S3-compatible [Phase 1]
 
- Storage for media attachments (images, video, audio, files)
- Stores encrypted media blobs
- Self-hosted (MinIO) or commercial (AWS S3, Backblaze B2, etc.) per instance operator choice
- Accessed via standard S3 API
### 28.9 Cross-component contract [Phase 1]
 
- MLS protocol logic identical across backend and clients (shared Rust library)
- Wire protocol uses MLS message formats directly
- Server routes opaque encrypted blobs through mailboxes; does not interpret content
- Push notifications carry encrypted payloads; decryption happens on device
- API surface (REST + Channels) is the same for web and mobile clients
### 28.10 Infrastructure expectations [Phase 1]
 
- Single instance can run on commodity hardware (VPS, bare metal, cloud VM)
- Vertical scaling sufficient for small-to-medium instances
- Horizontal scaling via Elixir clustering for larger instances
- Node SSR workers scale with Phoenix instances (no separate scaling concern)
- Standard PostgreSQL replication and backup patterns
- TLS via Let’s Encrypt or equivalent
- CDN for static assets and cacheable public content (optional, performance)
### 28.11 Fallback paths [Phase 1]
 
- If live_react proves inadequate for SSR needs, migration path is to a separate Next.js service for web SSR; component code remains largely unchanged
- If React Native crypto integration via FFI proves unworkable, fallback is custom native modules wrapping Rust manually
- If MLS performance at scale becomes a blocker, custom group key agreement built on MLS primitives is a research-grade option (not anticipated)
-----
 
## 29. Out of scope (explicitly)
 
These are intentionally not in the platform’s scope:
 
- Physical credential production (badges, NFC cards, key fobs) — a Strutco/Foundation service rather than platform software; the platform links digital credentials to physical artifacts but does not specify their manufacture
- Live streaming and live video
- Voice and video calls
- Payment processing as a platform function — the platform integrates external processors via adapters (Section 27) and never holds funds; community commerce outside platform-tracked resources is fully out of scope
- Advertising or sponsored content
- Algorithmic recommendation
- Public broadcast accounts (individual influencer model)
- Cross-posting integrations with specific external social networks
- SEO optimization tooling (platform serves indexable content; doesn’t actively optimize)
- AI-generated content as first-class feature
- NFT or blockchain integration
- Cryptocurrency as a platform-native payment mechanism (processor adapters may support it; see Section 27.2)
- Anonymous posting within communities (members are identifiable to community)
- External identity providers as authentication (no “sign in with Google”)
- Remote-only joining (in-person trust required)
-----
 
*This document enumerates features of the long-term vision. Pilot scope is a subset (see pilot scope document). Architectural reasoning is separate (see architecture document). This list is the input for technical decomposition into systems and capabilities.*
