> **2026-10-21 branding update:** the active native project is now `NativeReference/iLightCraft.xcodeproj`, scheme `iLightCraft`. Source and test folders, executable, engine bridge and scripts use iLightCraft. The chronology below retains the original identifiers and test evidence; see `NativeReference/Docs/APP-STORE-READINESS.md` for current Release settings.

# Native Apple app direction

Last updated: 2026-10-26

The user selected Unveil as the foundation again, with iPad, Mac and iPhone apps.
Active source is the separate Git checkout in `NativeReference/` (base `79be4e4`).
The parent Rust checkout remains an upstream reference. At the user's request,
`AppleWrapper/` and obsolete app/build copies were moved to Trash on 2026-10-26.
NativeReference is the only active Apple app project. Device backups, originals,
current source/build inputs and submitted release archives remain preserved.
See [cleanup and recovery details](NativeReference/Docs/Verification/2026-10-26-project-cleanup.md).
The historical local `plan/` files are absent from this checkout.

## Platform foundation

| Platform | Implementation | Evidence / remaining check |
|---|---|---|
| iPad | SwiftUI editor and Metal canvas; separate 292-point adjustment dock | New dock visually checked in simulator; signed update awaits device connectivity |
| iPhone | Photo above a compact adjustment dock, 48-point histogram, top Edit/Done control | New layout visually checked in simulator; 13 editor tests passed; previous build installed, new signed update awaits device connectivity |
| Mac | Mac Catalyst, arm64 engine slice, separate Mac sandbox/Photos entitlements | Build passed; launch requested; visual and workflow check pending because Mac locked, user elected to continue without it |

`NativeReference/scripts/build-xcframework.sh` builds the three Rust engine slices.
`NativeReference/scripts/build-apple-local.sh` accepts `simulator`, `ios` or `mac`.
It builds unsigned with a separate `com.cwallyman.UnveilNative` identity so this development
app cannot replace `com.cwallyman.LightCraftWebPad` or the former iPad prototype.
No App Store submission or automatic library migration occurred.

On 2026-10-19, at the user's request, the development-signed native app was installed on
the paired iPad Air 11-inch (M3), launched, and independently confirmed in the device
process list. Device inventory confirmed this new bundle identity was not
already installed, so there was no existing native app container to back up or overwrite.
The existing iLightCraft and earlier native prototype app identities were preserved.
The wildcard development profile omits increased-memory entitlement; large RAW memory behavior
still requires physical-device testing.

On 2026-10-19, at the user's request, the same signed universal device build was also
installed and launched on the paired iPhone 17 Pro. The embedded development profile was
verified to include this device, the signature verified, and `UIDeviceFamily` includes both
1 (iPhone) and 2 (iPad). Inventory confirmed no existing `com.cwallyman.UnveilNative` app,
so this installation replaced no native app data. No personal photos were opened or modified.
Installation and launch were confirmed by devicectl. A later process-list check could not
connect to the phone (CoreDevice 4016), so sustained runtime was not independently verified.
Physical iPhone import/edit/export, layout and relaunch persistence remain hands-on checks.

## Docked editing layout — 2026-10-19

The user's physical iPhone screenshot showed the old adjustment card covering the photo.
The editor now reserves distinct layout space, preserving the same Metal canvas while it
resizes: controls below the photo on iPhone/narrow windows and beside it on iPad/Mac.
The iPhone dock is capped at 340 points and 48% of available height. The side dock is
292 points wide. Edit/Done moved to the top toolbar. The histogram is 48 points high on
iPhone and 68 in the side dock. Tighter row spacing and caption labels retain 44-point
stepper/tab targets. Touching the photo no longer dismisses the editing controls.

Verification for this update:
- All 13 `EditorViewTests` passed, including both dock directions, canvas identity and
  touch routing, compact widths, histogram dimensions, alerts and histogram refresh.
  The initial run caught an inferred-square mini histogram; corrected before the passing run.
- Device and Mac Catalyst builds passed; simulator app built through the test run.
- Real simulator editor captures with the original generated fixture and +0.50 EV were
  visually inspected on iPhone and iPad. The photo is unobstructed, the dock is readable,
  and the top editing control and compact histogram are visible.
  Captures: `LocalVerification/unveil-docked-iphone.png` and
  `LocalVerification/unveil-docked-ipad.png` (local only).
- Device app signed with the existing development identity and verified using
  `codesign --verify --deep --strict` outside the sandbox.
- **Update not installed on physical devices.** Both remain unavailable in devicectl and
  xcdevice; app-container backups failed before copying with CoreDevice 4016. The user
  confirmed they are remote over Tailscale. No device app/data was replaced. Resume by
  establishing USB/local Xcode connectivity, backing up each existing app container, then
  installing and launching the signed build. Physical layout and persistence remain unverified.
- Mac visual check remains deferred at the user's request. No upstream CI run or commit.

## Next work, in order

1. Verify the native RAW/HDR and Photos update on iPhone; install the current update on iPad when requested. Verify import/edit/reopen
   and compact controls on a physical iPhone; verify the native Mac
   Files/Photos launch screen, resize, keyboard/mouse and editing on an unlocked Mac.
2. Native export and crop are implemented and installed on iPhone. Finish the saved hands-on
   import/edit/export/reopen checks; see `NativeReference/Docs/Verification/export-and-crop.md`.
3. Native Files/Photos library choice and same-item Photos saving are implemented, including
   revision checks, complete adjustment data and archive/reload conflict recovery. Physical
   same-item save/reopen and external-edit checks remain user acceptance tasks.
4. Apple RAW decoding, HDR input, live float EDR frames and HDR TIFF/HEIC export are implemented.
   Device/simulator builds passed and the generated simulator HDR editing view was visually checked.
   Camera qualification, physical HDR brightness and export readback remain user checks.
5. Migrate optional private CloudKit sync across all three devices, including conflict and
   offline recovery checks. Apple Photos mode should continue to use iCloud Photos.
6. Add remaining native editing controls incrementally, with shared command/settings behavior
   and platform-specific layout. The Rust engine's feature checklist is not native UI parity.

## Earlier foundation verification

- Rust bridge: 65 tests passed, 0 failed (`cargo test --locked -p unveil-ffi --features test-hooks`).
- Rust release engine builds passed for device, simulator and Mac Catalyst arm64.
- iPhone/iPad device, iOS simulator and Mac Catalyst app builds passed (Xcode 27).
  The Mac build predates only the subsequent home-screen Photos permission timing change.
- Swift compact layout regression passed at widths 296, 351 and 378 points with 280-point
  available height. The initial Xcode test-name filter selected zero tests; rerun using
  `compactCardFitsPhone()` executed and passed one test.
- Initial simulator screenshots were blocked by a Photos permission prompt, not successful
  editor evidence. Photos access is now requested only when the Photos source is selected.
  A subsequent iPad simulator render exposed a Metal abort: the simulator rejects textures
  backed by shared buffers. A simulator-only texture copy now preserves the device zero-copy
  path. All 4 focused Swift tests passed, including pixel readback through the corrected
  simulator texture path and compact card sizing. The final iPad simulator screenshot was
  visually inspected: generated image, +0.50 EV, histogram and floating Light controls are
  visible without dialogs. iPhone editor capture is not verified: a stale simulator Photos
  dialog persisted, then a capture immediately after simulator restart showed a blank launch
  screen. The compact view unit test passes, but this is not end-to-end iPhone proof. Mac visual verification
  was explicitly deferred by the user.
- No full upstream `cargo xtask ci` run and no commits made. Existing local changes preserved.

## Revision history

| Date | Change |
|---|---|
| 2026-10-18 | Restored Unveil as the active native foundation and added iPhone/Mac Catalyst platform configuration and compact layout. |
| 2026-10-19 | Passed device build and compact layout test; installed separate native Unveil on the user's iPad. |
| 2026-10-19 | Installed and launched the signed universal Unveil build on the user's iPhone 17 Pro; existing apps and libraries preserved. |
| 2026-10-19 | Docked controls outside the photo; moved Edit/Done to toolbar; added mini histogram; passed 13 editor tests and simulator visual checks; physical update pending connectivity. |

## iLightCraft native branding, 2026-10-19

The native app display name is iLightCraft. Its home shows the existing winking
lynx above the title with gentle native layer motion, replacing the photo cards.
The accent uses LightCraft amber #F2A516. The app icon uses the same winking lynx.
Native iPhone and simulator builds passed, and the simulator home was visually
reviewed at `LocalVerification/ilightcraft-home-iphone.png`. Source, asset license
and verification notes are in `NativeReference/Docs/Verification/ilightcraft-branding.md`.

The branding update was signed, installed and launched successfully on the iPhone 17 Pro.
