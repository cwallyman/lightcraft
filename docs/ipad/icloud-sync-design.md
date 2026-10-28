# Cross-device iCloud editing

Date: 2026-10-17. Status: development prototype implemented in AppleWrapper; user confirmed basic Mac ↔ iPad original/edit/rating continuity. Production readiness remains unverified.

## Required experience

Edit a photo in our Mac app, see it finish syncing, open our iPad app signed into the same
Apple Account and continue with the same non-destructive settings. Edits on iPad must return
to Mac. This must not require exporting a flattened JPEG, manually moving a backup, or opening
the same mutable catalog on both devices. Both apps need our sync integration; an unmodified
upstream LightCraft Mac build will not participate automatically.

Each device keeps a local WebKit catalog; the native CloudKit bridge now transfers originals and immutable edit revisions. Universal purchase/App Store listing
and iCloud device backup do not provide this application-level synchronization.

## Proposed design

Use CloudKit's private database and CKSyncEngine in a native Apple sync module shared between
macOS and iPadOS. Keep the LightCraft engine/UI and each device's working catalog local.
A small explicit, versioned bridge connects the web host to the native sync module. The native
Mac Rust host needs an Apple bridge to the same sync protocol; alternatively the same web
wrapper can host the Mac app. Do not change the Mac host solely to avoid defining the protocol.

Sources: [CKSyncEngine](https://developer.apple.com/documentation/cloudkit/cksyncengine-4b4w9?language=objc),
[Apple sync sample](https://github.com/apple/sample-cloudkit-sync-engine),
[iCloud capability configuration](https://developer.apple.com/documentation/Xcode/configuring-icloud-services).

### Data to transfer

- Original photo as an immutable asset, with a full-file checksum, byte count and type.
  Import leaves the user's source file untouched. Upload once, fetch/cache as needed.
- Versioned complete edit settings, including rendering process version, crop, masks, healing
  and references to any required auxiliary assets. Preserve an unknown field/schema on disk;
  refuse an unsupported edit revision instead of silently dropping adjustments.
- Separate stable UUIDs for library, image asset, editable photo/virtual copy, device and revision.
  Local numeric PhotoId values are mapped to those IDs and never treated as global IDs.
- Rating, flags, keywords, album membership and edit versions with explicit revision semantics.
- Last open photo as a lightweight resume hint. Zoom, scroll and panel state should be separate
  from image edits; restoring them must not change the photo or overwrite a device's active work.
- Previews/thumbnails for browsing before originals download. Full originals are the default
  basis for correct editing/export; proxy-only behavior is a separate explicit option.

Native catalog hashes can be sampled; web hashes use a separate engine hash. Neither is used as the sync identity. Sync assets use a full-file SHA-256 computed by the native bridge.

### Revisions and conflicts

Persist a durable local outbox before reporting an edit queued. Upload immutable revisions with
parent revision IDs; update a photo's advertised head using CloudKit change tags. Retrying after a
network failure must be idempotent. Downloaded edits are persisted locally before acknowledging
receipt. A settings change received from sync must not be uploaded as a brand-new local edit.

Concurrent edits descending from the same parent produce two preserved versions and a clear
choice, not last-device-wins. Preserve the whole coherent settings state initially; automatic
field-level merging can produce surprising interactions among crops, masks and healing.
Retain a pending local revision until CloudKit confirms it, and distinguish an edit acknowledgment
from completion of the original-photo upload. Use tombstones for deletions; handle edit-vs-delete
conflicts without destroying either the local source or an unsynced edit. Do not propagate permanent
asset deletion in the first milestone.

### User-visible states

Saved on this device; Uploading original; Syncing edits; Up to date; Offline (changes queued);
Downloading original; iCloud unavailable; Storage full; Conflicting versions; Update required.
An open destination app should fetch changes on activation, with a manual Sync Now action.
Do not promise instant handoff: upload/download completion depends on connectivity, iCloud
scheduling, photo sizes and account state. Never label a local save as successfully synced.
Account sign-out/switch pauses sync and preserves local data; data from different accounts must
never be uploaded into one another's cloud library without an explicit migration choice.

## Bridge work required

Existing commands `develop.get`, `photo.inspect` and `develop.merge` are useful building blocks,
but not a complete sync API. `develop.merge` merges a partial object and changes history; it must
not be assumed to atomically replace a remote revision, remove absent masks, or restore history.

Add narrow engine/host operations for export of a coherent photo state; original/auxiliary byte
transfer by asset ID; import with globally stable IDs; transactional remote revision application
with an expected local parent; and a committed-change feed. Include the engine/process/schema
version. Queue snapshots after the user commits an adjustment, coalescing slider previews.
Do not poll only the selected image: background metadata edits, imported photos, virtual copies,
albums and deletions also need to be observed.

OPFS stays local; arbitrary CloudKit code cannot directly read WKWebView's persistent storage.
Large originals should be transferred in bounded chunks or a streamed local bridge, not one
base64 message that duplicates a RAW file in JS and Swift memory. Validate every native bridge
request and restrict it to the bundled main-frame origin. A bulk backup ZIP is recovery tooling,
not the synchronization protocol.

## Why not sync the live catalog folder through iCloud Drive?

The catalog has paired snapshot/journal files, ordered atomic replacements, local PhotoId values
and process-local locking. Browser writes are asynchronous. Cloud delivery of individual files
cannot be assumed to preserve that multi-file transaction or a cross-device lock. Each device
therefore owns a local catalog and exchanges photo/revision records. Coordinated project documents
could be a separate simpler workflow if library-wide sync is not wanted.

Sources: crates/catalog/src/store.rs, crates/catalog/src/lock.rs,
apps/lightcraft-web/src/files.rs and [Apple's file coordination guidance](https://developer.apple.com/documentation/technologyoverviews/shared-data).

## Implementation milestones and acceptance

1. Define/test the stable-ID/revision envelope, durable outbox and replay/conflict logic locally.
2. Add the narrow engine/web/native bridge and prove a synthetic image/settings round trip on
   one device without touching an existing library.
3. Configure one shared CloudKit container, development schema and both signed app targets.
   Current wildcard signing is insufficient for iCloud. A healthy signed-in Xcode developer
   account and iCloud-capable provisioning profiles are needed for real-device CloudKit tests.
4. Mac -> iPad: original plus exposure, crop and mask render correctly; persisted revision and
   values match after relaunch. iPad -> Mac passes the same check. Compatible engine versions
   are required; CPU/GPU output equality is measured rather than assumed.
5. Test offline edits on both devices, concurrent revisions, interrupted uploads, process death,
   duplicate retries, missing original, quota exhaustion and account changes. No revision loss.
6. Validate background/foreground fetch behavior, library/album metadata, virtual copies and
   tombstones, then deploy the reviewed schema to CloudKit production and repeat through TestFlight.

Release gate: two physical devices, same Apple Account, real round trips and offline/conflict
recovery. A successful build, CloudKit entitlement or single-device mock does not establish sync.

## Scope decisions

Start with private same-person sync, original photos plus edits, and compatible engine versions.
Shared family/team libraries, collaborative live editing and restoring the full platform-specific
Undo stack are separate work. No sync behavior is enabled in the current installed prototype.

## Implemented prototype scope (2026-10-17)

Both apps use the shared Mac Catalyst/iPad wrapper. PhotoSyncCore persists immutable revision graphs and pending queues atomically. SyncCoordinator maps device-local IDs and the JavaScript bridge transfers originals in chunks and applies develop settings with readback validation. CloudKit uses the private PhotoLibrary zone and CKAsset originals/revision payloads.

Implemented metadata subset: rating, flag, label, title, caption, creator and keywords. Albums, named version/history collections, auxiliary-asset synchronization and last-open-photo restoration remain requirements, not completed features. Basic original transfer and exposure/rating edits in both directions were user-confirmed using development-signed builds; offline recovery, simultaneous conflicts, account changes and production/TestFlight remain unverified.
