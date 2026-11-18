<p align="center"><img src="assets/brand/logo.svg" width="166" height="166" alt="MEM logo"></p>

## MEM — Mesh Messenger

End-to-end encrypted messages between people who have approved each other, over **any** path
that exists: an animated QR code from screen to camera, sound for pairing, the local Wi-Fi
network, Bluetooth and Wi-Fi Direct, other people's phones as couriers, a MEM hub, e-mail and
any other app.

> **Delivery is our job; the risk choice is yours.** MEM prefers the most private channel that
> works. If only a less private one is available, MEM tells you exactly what it would expose and
> sends only with your consent. Every message shows the channel that really carried it.

This repository holds what MEM shares with testers, partners and readers: the current Android
build, the user-facing documentation and the presentation. The source code is not here.

**Status: beta, experimental.** MEM has had no external security review yet. Do not rely on it
where your safety depends on it.

### Install (Android 7.0 and later)

Download the APK for your phone from the latest release on the
[Releases page](../../releases/latest):

- `arm64-v8a`: nearly all phones from the last ten years;
- `armeabi-v7a`: older 32-bit phones;
- `x86_64`: emulators.

Compare the file with `SHA256SUMS` and the signing certificate in the release notes
(`sha256sum -c SHA256SUMS --ignore-missing`). Updates are signed with the same key, so they
install over the previous version. Only the newest release is kept here.

### What is here

| Path | Contents |
|---|---|
| [CHANGELOG.md](CHANGELOG.md) | What each release added, changed and fixed |
| [presentations/](presentations/) | The MEM presentation, in English and Ukrainian (PDF) |
| [docs/mini-apps.md](docs/mini-apps.md) | Building a Mini App: a small app inside MEM that talks to your service end to end |
| [SECURITY.md](SECURITY.md) | How to report a vulnerability |

### License

MEM is source-available, not open source. See [LICENSE.md](LICENSE.md).

The app builds, documentation and presentation are under the PolyForm Strict License 1.0.0
with an additional permission for security research. You may install and use the app and read
the documents for noncommercial purposes, but not redistribute or modify them without written
permission.
