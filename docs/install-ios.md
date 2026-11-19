# Installing the MEM beta on an iPhone

Each MEM release on GitHub (`mem.<number>`) carries an iPhone build,
`mem-<number>-ios-unsigned.ipa`. It is **not signed by Apple or by us**: an iPhone refuses to
install an app that nobody signed, so you sign it yourself, with your own Apple ID, using a
sideloading tool. This is the only way to install an iPhone app from a download without the
App Store or TestFlight.

**Beta status:** the iPhone build has not been tested on a real iPhone yet. Expect rough edges
and please report what you find.

## What you need

- An iPhone with iOS 15 or later (tested target: iPhone 14 Pro Max, iOS 27).
- An Apple ID (a free one is enough).
- Once, a Mac or Windows PC for the sideloading tool.

## Install

1. Download `mem-<number>-ios-unsigned.ipa` from the release on GitHub.
   Optional: compare its SHA-256 with the line in `SHA256SUMS`
   (macOS: `shasum -a 256 mem-<number>-ios-unsigned.ipa`).
2. Install a sideloading tool and follow its own setup:
   - **AltStore** (altstore.io): AltServer on the computer, AltStore on the iPhone; it
     re-signs MEM automatically while the iPhone and the computer share a Wi-Fi network.
   - **SideStore** (sidestore.io): like AltStore, but refreshes from the iPhone itself after
     the first setup.
   - **Sideloadly** (sideloadly.io): connect the iPhone by cable, drop the IPA in, enter your
     Apple ID.
3. Open the IPA with the tool and sign in with your Apple ID when it asks. The tool sends your
   Apple ID to Apple only to get a signing certificate for your own device.
4. On the iPhone: **Settings, Privacy & Security, Developer Mode**, turn it on and restart
   when asked (iOS 16 and later).
5. On the iPhone: **Settings, General, VPN & Device Management**, tap your Apple ID under
   "Developer App" and trust it.
6. Open MEM.

## Keeping it running

- With a **free** Apple ID the signature lasts **7 days**. Refresh MEM in AltStore or SideStore
  (or install again with Sideloadly) before then; otherwise iOS will not open MEM until you
  do. Your messages stay on the phone either way.
- A free Apple ID may have at most 3 sideloaded apps at a time.
- A paid Apple Developer account signs for a year.

## Updating

Install the IPA of the newer release the same way, with the same Apple ID: the app and its
data are kept. Make a backup first (Settings, Privacy and security, Backup), in case iOS
treats the new build as a different app.

## What works on iPhone, and what does not yet

| Built for iPhone (not yet tried on a device) | Not on iPhone yet |
|---|---|
| Pairing by QR code, messages by animated QR code | Bluetooth and Wi-Fi Direct |
| Messages through a hub | Pairing by sound |
| Groups, attachments, voice messages | Calls |
| Backup and restore | Push wake-ups: MEM receives only while it is open |
| | "Stay reachable" in the background |

Finding contacts on the same Wi-Fi network may not work: iOS lets an app send to nearby
devices on the local network only with a permission that Apple grants to signed developer
accounts. Connecting with a code (QR or pasted) still works.

**Know what this build does differently:** on Android the key that encrypts MEM's data on the
phone is held by the Android Keystore. In this iPhone build it is still a file inside MEM's
own app storage, protected by iOS's file protection but not yet by the iOS Keychain. Someone
who gets a full copy of the phone's app data can read MEM's data.
