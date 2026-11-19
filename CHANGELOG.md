# Changelog

All notable changes to MEM. Versions follow [SemVer](https://semver.org/); before 1.0 the wire
format may change between minor versions.

## [mem.107] — 2026-11-19

### Added

- **Barcode scanner for Mini Apps:** a Mini App can show a scan button. The camera opens on
  MEM's own screen, and only when you tap it. The Mini App gets only the codes you scan, never
  the picture, and never MEM's own codes such as contact cards. It reads QR, Data Matrix,
  Code 128, EAN, UPC, ITF and Code 39, including GS1 DataMatrix and GS1-128 on medicine packs
  and shipping labels. You can send one code, or scan many and send them together, with equal
  codes counted. The install screen says when a Mini App can ask you to scan.
- **Sample Mini App "kit check"** (`samples/miniapp-stock`): scan a bag's contents in one go;
  its service compares them with the bag's list and reports missing, expired and foreign items.
- **iPhone beta:** each release now carries an unsigned iPhone build
  (`mem-<number>-ios-unsigned.ipa`), installed with AltStore, SideStore or Sideloadly and your
  own Apple ID (docs/install-ios.md). Not yet tested on a real iPhone. Bluetooth, Wi-Fi Direct,
  pairing by sound, calls and push wake-ups are Android-only for now.

### Fixed

- **The built-in hub needed an address:** MEM's own hub is built in, but the hub switch stayed
  locked until you typed an address. Now the hub screen shows the built-in address and the
  switch works with the field empty; enter an address only to use another hub. The hub stays
  off until you turn it on.

### Changed

- **Networking library updated** (iroh 1.4.2), needed for the iPhone build.

## [mem.106] — 2026-11-18

### Added

- **Voice messages:** with the text field empty, the button next to it is a microphone. Tap to
  record, tap again (or Send) to send, Delete to throw the recording away. Recording goes on
  with the screen off; the notification offers Send and Cancel. A minute of speech stays small
  enough for every channel. Voice messages have their own switch under Shared files and media.
- **Audio plays inside the message:** play or pause, a seek bar, the time and the speed (1×,
  1.5×, 2×). Starting one player pauses any other.
- **Speech to text, on the phone:** turn on Settings → Speech to text, download a speech model
  once, and the microphone can type for you. The words appear in the message field as you
  speak, go in where the cursor is, and can be corrected while you keep talking; only the text
  you send leaves the phone, never the audio. It works with the screen off. Ten languages:
  English, Ukrainian, German, Spanish, French, Finnish, Italian, Polish, Romanian and Dutch.
  - **Speech models:** a minimal model (Whisper base, 153 MB) and the recommended one
    (Parakeet TDT 0.6B v3, 640 MB). Downloads come from Hugging Face (MEM says once that
    Hugging Face sees your IP address), resume after a broken connection, and are checked
    byte for byte before use. Models can be disabled or deleted; sizes your phone has too little
    memory for are refused with the reason.
  - **The microphone button:** a long press opens a list above it (voice message, speech to
    text); a setting chooses what a plain tap does.
  - **New models without an update:** with the hub on, MEM asks it now and then for the
    current list of speech models. The list is signed by the operator key the app trusts and
    is used only if that signature checks out.
  - **Timing settings:** when speech is cut into pieces for recognition (longer pieces are more
    accurate, shorter ones show text sooner), processor threads, and whether the bar above the
    field shows recognition progress. All within safe bounds, with a reset.
- **Unread counts:** each chat shows how many messages arrived since you last read it, and the
  app icon shows how many chats have unread messages (where the launcher supports it; a switch
  under MEM display turns it off).
- **A taller message field:** swipe up on the grip above the field to open it for a long text,
  down (or a tap) to close it. How tall it gets is a setting under MEM display (default 75 % of
  the screen); with the keyboard up it stays below the chat header.
- **Undo and Redo** in the message editor.
- **A built-in MEM hub:** with Hub turned on and no address of your own, MEM reaches the
  operator's hub without typing an address. The list of hubs is signed by the operator key the
  app trusts.

### Changed

- **Messages sent straight to a contact nearby** (Wi-Fi, Bluetooth, Wi-Fi Direct) now wait for
  the contact's confirmation before also going to the hub; if it arrives in time, the hub never
  sees the message. The wait is a setting under Messages and delivery (0–60 s, 4 s by default;
  0 uploads at once as before).
- **The message editor** takes the whole screen above the keyboard on a phone, without a title,
  with compact buttons; the time left to edit shows as a clock with an ⓘ that explains it.
- **Explanations stay out of the way:** on the contact screen and on the Speech models screen,
  long explanations open from a "?" or ⓘ button instead of filling the screen.

### Fixed

- **E-mail channel:** the e-mail library is updated to fix published flaws: a network
  attacker could forge server replies after STARTTLS, a crafted reply could crash the app or
  fill its memory, and some addresses could be sent to the wrong place.
- **Mini Apps (test):** an update must now be signed by the same developer as the installed
  Mini App; a file with the same name from anyone else is refused.
- **Chats with pictures** open at the newest message and no longer jump when you long-press a
  message; pictures show whole, in their own proportions.
- **The list of channels under a message** no longer grows with every resend.
- **A long text in the edit dialog** no longer overflows the screen.
- **Backups** no longer change when nothing in them has.

## [mem.105] — 2026-11-16

### Fixed

- **Groups looked empty:** people you invite become members only once they accept on their own
  phone. The group list and the chat now show who is invited and still has to accept ("1
  member · 2 invited"), invitations appear on the home list (not only under Requests), and
  writing to a group nobody has joined yet says why.
- **Long-press menus** no longer fail on some builds.

### Added

- **A menu for groups:** long-press a group on the home list, or tap its name in the chat, for
  group info, inviting, shared files, leaving (or ending the group as its admin) and deleting.
- **Deleting a group** removes it and its messages from your phone; if you still belong to it
  you leave it first (as admin: it ends for everyone), and an invitation is declined.

## [mem.104] — 2026-11-15 — beta 2

### Changed

- **Home screen:** messages on the top line (hub state, messages carried for others, notices
  after an action) now fade into the background after a few seconds and come back only when
  what they report changes.
- **Search:** a tap outside the field hides the keyboard and keeps what you typed; the
  field's rainbow border is thinner.
- **Under the hood:** every library and toolchain updated to its latest release (Rust 1.100,
  the newest encryption, networking, database and Android build components), while Android
  7.0 stays supported. The Android Keystore bridge that protects your local data was rewritten
  for the new interface and checked on Android 7 and 15: existing data opens after the update.

## [mem.99] — 2026-11-15 — beta 1

The first build meant for beta testing on Android. It is the starting point: everything below
works on our test phones, but MEM has had **no external security review yet**. Do not rely on
it where your safety depends on it.

### Installing

Download the APK for your phone from the GitHub release (`arm64-v8a` for nearly all phones,
`armeabi-v7a` for older 32-bit ones, `x86_64` for emulators) and compare it with
`SHA256SUMS` and the signing certificate in the release notes. This is the first build signed
with the MEM release key: earlier test builds were signed with a debug key, so remove them
first (make a backup in MEM, uninstall, install, restore).

### What you can test

- **Contacts:** pairing in person by QR code or by sound, contact requests by card, trust
  levels, a name of your own and a picture for each contact (kept only on your phone).
- **Messages over any channel,** each labelled with the channel and privacy grade that really
  carried it: animated QR codes, the local network, Bluetooth and Wi-Fi Direct (off by
  default), other MEM phones as couriers, a MEM hub you choose, a phone of a contact acting as
  a hub, email, and any other app through the share sheet. Anything below a contact's privacy
  floor asks first and is listed in the Privacy log.
- **Couriers and the hub:** an optional setting (off by default) lets an online courier hand
  the messages it carries to the hub and fetch messages for phones it met, within amounts you
  set. What it exposes and what it cannot promise is shown before you turn it on.
- **Groups** of up to 16, **files** (small ones through any channel, large ones pulled
  directly), shared folders for a contact or group, **voice calls** over direct paths,
  editing and deleting sent messages within a limit both sides agree on.
- **Several devices** for one identity, encrypted **backups** with a recovery phrase,
  post-quantum encryption for contacts that support it.
- **New screens:** Settings grouped into pages with a search; a status strip and a menu for
  selecting contacts on the home screen; a search field on the home bar; long-press a contact
  for quick actions, or the name in a chat for all actions.
- **Experimental (off by default):** Mini Apps, small apps of outside services that work
  through MEM even offline, with an optional web page after a disclosure.

### Known gaps

- Android only; iOS is not built. Desktop and web builds are untested.
- Bluetooth and Wi-Fi Direct have been checked on few phones.
- "Mute" on a contact is shown but not available yet.
- There is no MEM hub set by default: to use one, enter its address under Channels.

## [mem.107] — 2026-11-19

### Added — encrypted backup and recovery phrase

- **Recovery phrase:** turning on backups shows a generated phrase, either 12 English words (BIP-39)
  or 9 Latin syllables (proquints, language-neutral; the default for the Ukrainian UI).
  - You confirm it by typing 3 of the words.
  - Either form opens every backup, because keys come from the 128-bit secret, not the words.
  - A single mistyped syllable letter is corrected when only one fix fits.
  - The phrase screens block screenshots and screen recording (`FLAG_SECURE`), checked on the
    emulator.
- **Backup:** Argon2id (64 MiB, 3 passes) then XChaCha20-Poly1305 over the padded state, with or
  without message history.
  - It can be saved as a `.membackup` file through the system picker (no storage permission),
    sent through another app (with a disclosure), or shown as an animated QR to a new phone
    (64 KiB, "without history" if larger).
  - MEM keeps no server copy.
- **Restore:** from a QR scan, a file or pasted text. It is a move: same identity, contacts and
  keys.
  - Restoring over an identity that has contacts asks first, and keeps the old state encrypted as
    `replaced-<time>.bin`; nothing is deleted.
- **Automatic backups:** off by default, every N days (1–90) or after each new contact, into a
  folder you pick once. Cloud-synced folders are allowed, with a note about what the provider sees.
- **Fixed:** the runtime now persists state changes made by clock ticks (expiry, acks).
- **Engineering:**
  - crates `argon2` (MIT/Apache-2.0) and `bip39` (CC0-1.0);
  - `spec/backup.cddl`, a published vector, and a CI validation step;
  - fuzz target `backup_decode`;
  - invariant tests for restore equivalence, both phrase forms, wrong phrases and hostile KDF
    parameters;
  - a test that only two FFI functions carry phrase words;
  - the Linux end-to-end test turns backups on through the UI and restores a fresh profile.

### Added — delivery confirmations

- **No more silent message loss.** A message is `Delivered` (two ticks) only when the recipient's
  signed, encrypted ack arrives, sent after the message is stored.
  - Until then it stays in the outbox and is resent at every Wi-Fi encounter, and every 30 s
    within one (at most 3 sends per encounter).
  - Over QR and share, the chat says "Not confirmed yet — tap to send again".
  - Fixes a message written into a dead Wi-Fi connection being marked Sent and lost.
- **How acks travel:** piggybacked in the next message (inner field 10, any channel), or as a
  standalone ack bundle (kind 1, content type 2) over Wi-Fi within the floor.
  - A retransmitted copy is still refused by its tombstone, but acknowledged again from a
    bounded delivered index.
- **Faster failure detection on Wi-Fi:** QUIC keep-alive 3 s, idle timeout 10 s. Sending prefers the
  newest encounter with a contact.
- **Retry period:** the message lifetime defaults to 30 days, and is a setting in Profile within
  1–30 days.
- **Wire format:** inner field 10 and content type 2 are new. Builds before this change cannot
  read messages that carry them; update all devices together.
- **Engineering:**
  - invariant tests `tests/invariants/tests/delivery_acks.rs`;
  - fuzz target `inner_decode`;
  - the runtime ticks every second while a peer is connected.

### Added — animated QR and Wi-Fi

- **Animated QR (P0)**:
  - Messages of any size up to the 64 KiB bundle cap travel as a loop of fountain-coded QR frames
    (RaptorQ, RFC 6330), 8 frames/s, QR version 15.
  - The receiver's camera can miss frames and catch them in any order; any K frames complete the
    transfer. A typical message needs 1–2 frames, and the maximum is about 21 s.
  - Frame format: `spec/frame.cddl`.
- **Scanning in Rust**:
  - The camera feeds its luma plane to `rxing` in the core.
  - This replaces `mobile_scanner`, which uses Google ML Kit and reports usage to Google, contrary
    to the P0 exposure profile.
  - Pairing-card QR codes (`memcard0:`) still work.
- **Wi-Fi delivery (Iroh, P1)**:
  - Phones on the same local network find each other by mDNS and connect directly over QUIC.
  - There are no relays, no n0 discovery and no port mapping, and the transport key is random on
    every start.
  - Queued messages go out by themselves when the contact comes into range, but only when the
    channel is within the contact's privacy floor (or a standing rule allows it). Automatic sends
    never ask and never go below the floor.
  - A message typed while the contact is nearby is sent immediately.
- **Encounter protocol, subset**: `hello`, `tags`, `bundles` and `bye`.
  - Only today's receiving tags are advertised, padded with random decoys and sorted, so tags of
    different days stay unlinkable.
  - The per-encounter quota is 256 bundles.
- **App**:
  - A Wi-Fi status icon, and a "nearby" mark on contacts and in the chat header.
  - A **Channels** screen showing each channel's exposure profile, with a switch for Wi-Fi delivery.
  - "Wait for Wi-Fi" in the channel picker.
  - Live updates when messages arrive over the network.
  - `Wi-Fi · P1` provenance badges.
  - en/uk strings.
- **Android permissions**: network state, Wi-Fi state, multicast (with a foreground-only multicast
  lock) and camera.
- **Engineering**:
  - New crates: `mem-fountain`, `cla-qr`, `cla-iroh`.
  - `mem-runtime::Node`: Tokio, the Iroh channel, a clock and an event stream.
  - New fuzz targets: `frame_decode`, `encounter_decode`, `fountain_assemble` and `qr_luma`.
  - Vectors for frames and encounter messages, validated against CDDL in CI.
  - `deny.toml` gains narrow licence exceptions for Iroh's transitive dependencies:
    CDLA-Permissive-2.0 (CA root data) and Unlicense (wasm-only crates). The `paste` "unmaintained"
    advisory is ignored; it is a build-time macro with no vulnerability.

### Changed — wire format (not compatible with 0.1.0 bundles)

- Every bundle now carries a `purge_commitment` (envelope field 10, bound into the HPKE AAD). The
  encrypted inner payload carries the matching `purge_token` (field 9, covered by the signature).
- The receiver verifies that the token matches the commitment; a mismatch means nothing is
  delivered. After delivery it keeps a tombstone (30 days, quota of 32,768). Later copies of the
  same bundle are refused before decryption. Tombstones survive restarts.
- Bundles made by 0.1.0 are rejected as malformed. Pairing cards are unchanged, so contacts stay
  paired after the update. Messages still queued in a 0.1.0 outbox are dropped on upgrade.
- Snapshots from 0.1.0 still load. The purge key is created on the first send.

## [0.1.0] — 2026-10-15 — experimental

First usable build. **Experimental: do not rely on it for high-risk communication.** No
external security review has been done.

### What works

- **Identity**: Ed25519 identity, X25519 mailbox key, base32 fingerprint. Created on first launch.
- **Pairing**: exchange signed contact cards by scanning a QR code (Android) or
  pasting text (any platform). Both devices show the same 6-emoji short authentication string
  to compare. Contacts record whether you met in person or paired remotely.
- **End-to-end encrypted 1:1 text messages**: sign-then-encrypt with the recipient's key in the
  signed data. HPKE (RFC 9180, X25519 / HKDF-SHA256 / ChaCha20-Poly1305), with the envelope
  bound as associated data. Padding buckets hide message length. Daily-rotating blinded
  rendezvous tags. The envelope carries no identities or timestamps.
- **Two channels**:
  - **QR code on screen (P0)**: the receiver scans it. Needs no network and no other app.
  - **Share as text (P4)**: an armored `mem0:` string sent through any app (Signal, WhatsApp,
    email…) and pasted into MEM → Receive.
- **Consent before downgrade**: sending below the contact's privacy floor
  (default P2) shows a disclosure generated from the channel's exposure profile. That covers who
  observes, what they learn, and what stays private. You can allow once, or create a standing
  permission for the contact (with an extra confirmation). Every below-floor send goes into the
  **Privacy log**. Permissions are revocable.
- **Honest provenance**: each message shows which channel carried it and its grade. Text
  you paste in is shown as P4, because MEM cannot know how it reached you.
- **Deny by default**: only bundles that decrypt and verify as coming from an approved
  contact become messages.
- **Local storage**: one snapshot, encrypted with XChaCha20-Poly1305 and written atomically.
  Android backup and device transfer are disabled.
- **Languages**: English and Ukrainian.
- **Platforms**: Android (arm64, armv7, x86_64) and Linux desktop. Linux has no camera scanning;
  paste instead.

### Known gaps

| Gap | Impact |
| --- | --- |
| One mailbox key, never rotated | A key compromise exposes all past messages (no forward-secrecy window) |
| Storage key in an app-private file, not Android Keystore | A rooted device or forensic access to app data can read the history |
| Pairing nonce is long-lived | Re-pairing yields the same secret unless you tap "New pairing code" |
| No delivery acknowledgements | "Sent" means handed to a channel, not received |
| Single-frame QR only (≈ 820-byte bundle) | Long messages need the text channel |
| No network, courier, hub, BLE or Iroh delivery | Every message needs a user action |
| No backup/restore, single device | Losing the device loses the identity and history |
| Wire format was experimental | v0.1.0 bundles are not accepted by later versions |
| Android release is signed with the debug key | Not suitable for store distribution; Android won't accept it as an update to a differently signed build |

### Engineering

- Rust workspace: `mem-proto` (bounded, canonical-only CBOR decoders, fuzzed), `mem-crypto`,
  `mem-cla`, `mem-core` (sans-IO agent), `mem-runtime`, `mem-ffi` (flutter_rust_bridge 2.13).
- 71 Rust tests, including the cross-crate invariant tests. Plus a Flutter integration test
  driving the real app end to end.
