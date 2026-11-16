# The Mac on an external display: the plan

2026-11-15. Fork `somas1/sill`, branch `extended-display`, from main at aea3547 with Saffsanity/sill#301 merged in.
A plan only: no feature code. Nothing here was built or run: the shell this was written from has no Swift or Xcode,
so every claim about behaviour is read from the code or quoted from a pull request, and says which.

## What John asked for

An iPhone or iPad running Sill and plugged into an external display (a USB-C monitor, a TV over AirPlay, or display
glasses) should act as if the Mac had an extended display, not mirror the phone's screen:

1. **Sometimes** the monitor should show the Mac as it is (a window, or the Mac's own screen), the phone becoming the
   trackpad and keyboard.
2. **Sometimes** it should show an **extra Mac screen sized to the monitor**, one the Mac did not have before, so the
   phone and monitor add room to the Mac rather than repeat one of its screens.
3. **Devices**: iPhone first (mostly in the car), iPad, and glasses: XREAL-style glasses that plug into USB-C, and
   later a RayNeo X3 Pro and perhaps Snap Specs, which are standalone and wireless.

## What exists

- **Part 1 is Saffsanity/sill#301** (addhumal, with kolyan94's iOS 27 commits), merged into this branch as it is. The
  monitor's scene (`windowExternalDisplayNonInteractive`: a USB-C monitor with Stage Manager off, or AirPlay) gets a
  window of its own (`ExternalDisplaySceneDelegate`); the stream's picture moves there and back as the monitor comes
  and goes (`ExternalDisplay` in SillAppDelegate.swift). The phone or iPad shows the trackpad, with "The Mac is on the
  external display." On iOS 27 the stream screen registers a scene accessory (`ExternalDisplayAccessory`), without
  which iOS connects no external scene at all; overscan is off; auto-lock is held off through `DeviceIdleTimer`.
  **The viewport (kind 9) is the monitor's size in points** (`scene.screen.bounds.size`), sent again as a monitor
  comes or goes, so the Mac sizes a streamed window to the monitor. No wire change. kolyan94 tested it on an iPhone 17
  Pro on iOS 27.2 with XREAL One Pro glasses; nobody has reported a test on an iPad with a real monitor.
- **Part 2 is upstream's Virtual Desktop** (`docs/virtual-desktop-plan.md`, Saffsanity/sill#352, a draft with Noah's
  rulings of 2026-11-12): a display of Sill's own, off by default, made while a device shows it and removed 30 s
  after, its windows moved to the main display when it goes and back when it returns, switched on and sized from the
  Mac or a paired device through kinds 16 and 17. Its **phase 4, "Match this device"**, sizes it from the viewport the
  device already sends. With #301 that viewport is the monitor's, so upstream's phase 4 is, nearly for free, "an extra
  Mac screen sized to the monitor". Phase 1 (the Desktop button stepping through the Mac's screens, #355) is in main;
  phases 2 to 4 are not built, and phase 2 waits for #200 (the headless display), which is not merged.
- **Older takes on part 2**: Saffsanity/sill#156 (dr-apple: a display of the device's own size, made the Mac's main
  display while the Desktop streams; 325 commits behind main, 7 commits, 765 lines) and RaffaelFerro's fork ("As a
  Monitor"). Upstream's plan credits them but does not take their host halves: it never makes the device's display the
  main display (§7, §9 there).

## Decision for John: how to get part 2

| | What it is | Gets you | Costs |
|---|---|---|---|
| **A. Wait for upstream** | Use #301 now; take the Virtual Desktop when Noah ships phases 2 to 4 | No code of ours to keep | No date; phase 2 waits on #200 |
| **B. A small version in our fork, upstream's names** (recommended) | §"Part 2 in this fork" below | Part 2 within a few sessions | Our code to keep until upstream's lands, then dropped for theirs |
| **C. Port #156 onto #301** | dr-apple's host side, as it is | Fastest to try | Makes the monitor the Mac's main display; 325 commits of conflicts; the design upstream turned down |

**Default taken if John says nothing: B.** It is built so that upstream's later phases replace it rather than fight
it: the same JSON keys, the same size id, the same lifetime and window rules, so dropping our version for theirs is a
merge, not a migration.

## Part 1: what is left

Nothing to write before a test on real hardware. The checks, in this order, on the M6 in Claude Code or Xcode:

1. Build the iOS app (Debug, for a device) and Sill.app (`Scripts/make-app.sh`) from this branch.
2. iPhone on a USB-C monitor: the picture moves to the monitor; the phone is a trackpad, sideways and upright; the
   hardware keyboard keeps working; unplug, and the picture comes back.
3. The viewport the Mac logs ("viewport: … pt" in the device console, the window's fit in Sill.log) is the monitor's
   size in points. **Record what iOS reports for a 1080p and a 4K monitor** (`scene.screen.bounds` and
   `scene.screen.scale`): part 2's size rule depends on it.
4. The same on an iPad with Stage Manager off; with it on, #301 does nothing (iPadOS extends its own screen), which
   is expected, not a bug.
5. AirPlay to a TV. XREAL-style glasses if one is at hand.
6. Car: iPhone on the car's display only if it is a plain monitor input; CarPlay is not an external display to iOS.

Anything that fails here is fixed on this branch before part 2 starts, and offered back on #301.

## Part 2 in this fork (option B)

A **Monitor Desktop** (our name in the words; the code uses upstream's `virtualDesktop` keys): while a device shows
its picture on an external display and the setting is on, the Mac makes one display of Sill's own, sized to that
monitor, and the Desktop streams it. Upstream's plan is the reference for every rule not written here; where this
differs, it says so.

### The rule

- **Made** when all hold: the setting is on; a device is streaming the Desktop with its picture on an external display
  (the device says so, below); this account is on an unlocked console; Screen Recording granted; the private API
  present (`VirtualDisplay.checkPrivateAPI`); Sill.app's AppKit loop running; not a synthetic host (a test hook stands
  in, below).
- **Size**: from that device's monitor, in points, at the monitor's own scale when iOS reports one above 1 (else 1x),
  each side at least 800 points at 2x (`VirtualDisplaySize`'s rule: macOS ran smaller ones at 1x), the backing store
  held inside 3840 × 2160 pixels with its shape kept, as upstream's §2 caps its default. Rate: the device's, at most
  60. Worked guesses, to check against step 3 above: a 1080p monitor that iOS reports as 1920 × 1080 at 1x gives
  1920 × 1080 at 1x; a 4K one reported as 3840 × 2160 at 1x gives 3840 × 2160 at 1x (small text: phase 3 of upstream
  would offer 1920 × 1080 at 2x).
- **Placed** to the right of the Mac's main display in the arrangement, never made the main display (upstream §7).
- **Kept** while the stream shows it, and 30 s after: a reconnect, or the phone locked a moment, comes back to the
  same desktop and its windows.
- **Removed** 30 s after no device shows it, at once when the setting goes off, when the monitor is unplugged (the
  device's viewport says so) for 30 s, at Quit, off console; its windows first moved to the main display (upstream §6,
  `evictForeignWindows`'s way), and put back if it returns in the same run of Sill and the person did not move them.
- **Locked**: never made; one that exists is kept until the unlock (upstream §5, #298).
- **One at a time**: one Monitor Desktop, for the device in use (PointerControl's controller). A second device on a
  monitor sees the same desktop fitted, with bars.

### The wire (additive, upstream's keys)

- **Device → host, the hello or the viewport** gains one optional field saying the picture is on an external display
  and that display's scale: `Viewport.externalScale: Double?` (nil: the picture is on the device's own screen). An older
  host skips it (JSONDecoder ignores unknown keys). Its bounds join `ViewportBounds` (0.5 to 4, else refused whole, as
  the other fields are).
- **Kinds 16 and 17** carry upstream's `virtualDesktop: Bool?` (the switch) and `virtualDesktopSize: String?` with
  only one value from us, `"device"` (upstream's id for Match this device). `HostSettingsState.virtualDesktop` with
  `shown` and `note`, as upstream's plan has them.
- **Who may switch it from a device**: a paired session only (`sessionPaired`, Direct Wireless's bar), at home and
  away, as upstream's §3a decides.
- Nothing else: no new kind, `StreamSource` unchanged (`.desktop` is the Desktop on whichever display the host shows).
- **Mixed builds**: an upstream Sill.app ignores the new viewport field and never sends `virtualDesktop`, so our device
  shows no switch for it (the ledger's rule 9); our Sill.app with an upstream device never sees `externalScale`, so it
  makes nothing.

### The host

- `Sources/StreamProtocol/Viewport.swift`: `externalScale` and its bound. `HostSettings.swift`: the two settings and
  the state, in every place the file's header lists.
- `Sources/SillHost/CaptureScale.swift`: `MonitorDesktopSize` (pure: viewport and scale in, points, scale and rate
  out), beside `VirtualDisplaySize`, with a check `Tests/checks/monitor-desktop` and its mutants.
- A `MonitorDesktop` owner beside `VirtualStage`, using `VirtualDisplay` as the stage does: make, wait online, place,
  remove; its life as a pure `MonitorDesktopLife` (events and a clock in; make, keep, remove out), checked by the same
  check. Window moves reuse WindowRestore's tolerance and the stage's eviction.
- `DesktopDisplays.swift` (phase 1, in main): the Monitor Desktop becomes a stop after the real monitors, where its
  `// PHASE 2` line says the Virtual Desktop goes, and the Desktop moves to it when it is made for the device in use.
- `StreamCoordinator`: reads the viewport's `externalScale`, applies the settings through `DeviceSettings`, owns the
  `MonitorDesktop`'s life, logs one line per make and removal ("Monitor Desktop: made 1920 × 1080 at 1x for iPhone
  (iPhone17,1)").
- Test hook: `SILL_TEST_MONITOR_DESKTOP=1` on a synthetic host stands a fake stop in through `DesktopDisplaysHook`, so
  everything but the display itself runs on loopback.

### Sill.app

- Settings › Displays: "Make an extra screen for a device on an external monitor", off by default, with the words
  "When your iPhone or iPad shows Sill on a monitor, your Mac gets a screen the monitor's size. Its windows move here
  when it goes." The menu's subtitle while one exists: "Monitor Desktop for iPhone: 1920 × 1080".

### The device

- `ExternalDisplay` sets `Viewport.externalScale` from `scene.screen.scale` while the picture is on a monitor, nil
  otherwise.
- The Settings panel, in the Mac's section, while the Mac reports `virtualDesktop`: the switch "Extra Mac Screen on
  Monitor", footnote "Your Mac gets a screen the size of your monitor while Sill shows on it." Disabled on an unpaired
  session with Direct Wireless's words.
- When the Mac makes it, the Desktop is on it; the Desktop button steps through the Mac's screens as phase 1 does.

### Gates

- `Tests/checks/run-all.sh`, with the new check and its mutants; `swift build -c release`; the iOS app Debug and
  Release for the simulator and Debug for a device.
- Live on loopback: a synthetic host with the hook, `sillclient.py` sending a viewport with and without
  `externalScale`, `--set=virtualDesktop=1` from a paired and an unpaired test client (shown, refused), the 30 s
  grace across a reconnect.
- On the M6 at rest, never while a device is connected to John's own Sill.app: `--virtual-display-selftest` at the
  sizes step 3 of part 1 recorded.
- On the hardware: an iPhone on a monitor shows the extra screen; drag a window onto it from the Mac; unplug, wait
  30 s, the window is back on the main display; plug in, it returns.

## Part 3: glasses

- **USB-C display glasses** (XREAL-style, RayNeo Air): iOS treats them as an external display, so parts 1 and 2 cover
  them; kolyan94's test already did.
- **Standalone glasses** (RayNeo X3 Pro, likely Snap Specs): they do not take video from the phone's USB-C port; the
  X3 Pro connects wirelessly (RayNeo's own guide). Sill would need a client running on the glasses. Parked until parts
  1 and 2 work; a few forks have started Android clients (andrewgribben, ErickXavier), which would be the starting
  point.

## Upstream

#301 is upstream's to merge; fixes found in part 1's tests go back to it. Part 2 is upstream's plan in a smaller form:
before building it, John may want to tell Noah on #352 that a fork is building "Match this device" for external
monitors, and offer it back as phase 4's first half.

## Progress

- 2026-11-15: #301 merged into this branch with main at aea3547 (no conflicts); the plan written. Nothing built or run.
- 2026-11-16: part 1 tried on the hardware: the Duo on a car monitor over Tailscale, to the M2 and the M6, usable as is.
  The M6's Desktop shows bars at the sides (16:10 on a wider monitor), which part 2 removes. Step 3's sizes still to
  record. Main use, for part 2's design: the car, away from home, so the away quality sets what a monitor-sized
  desktop costs in cellular data.
