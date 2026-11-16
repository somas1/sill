**The Mac on an iPhone's or iPad's external display: #301 merged in, and the plan for an extra Mac screen sized to the monitor (2026-11-15, branch `extended-display` in the fork somas1/sill, from main at aea3547).**

- **What John asked for**: with an iPhone or iPad plugged into a monitor, TV or display glasses, the monitor shows the
  Mac instead of mirroring the phone; sometimes the Mac as it is, sometimes an extra Mac screen the monitor's size.
- **Merged in**: Saffsanity/sill#301 (addhumal, with kolyan94's iOS 27 commits 359facf4, ace802b3, 28e0c540) at
  218c8434, then main at aea3547 (d7433f2): no conflicts; the five files of #301 only.
- **The plan**: `docs/extended-display-plan.md`. Part 1 is #301, waiting for a test on real hardware. Part 2 is a
  small version of upstream's Virtual Desktop (#352) with its keys, sized to the monitor from #301's viewport and an
  optional `Viewport.externalScale`; option B there, the default unless John chooses otherwise. Part 3 (standalone
  glasses) is parked.
- **Verified** (2026-11-16, John in his car): the iOS app built from this branch (Debug, for the Duo "Duopus",
  signed on John's team by command-line overrides only: `DEVELOPMENT_TEAM=227FKZG73J
  PRODUCT_BUNDLE_IDENTIFIER=com.somas1.sill -allowProvisioningUpdates`, the project untouched; Xcode 27.2 through
  `DEVELOPER_DIR`, since xcode-select names the Command Line Tools). The Duo on a car monitor, over Tailscale to the M2
  and the M6 (Sill.app 0.6.0 unchanged: #301 is iOS only): the picture on the monitor, the phone the trackpad, "fairly
  usable as is". The M2's Desktop filled the monitor; the M6's had bars at the sides, as expected: its Desktop is
  GLKVM's 2560 × 1600 (16:10) fitted to a wider monitor, and #301's viewport sizes only a window. Sessions away from
  home ran at Low · Retina, 4 Mbps.
- **John's main use**: the car, the Duo on the car's monitor, over Tailscale to the Macs at home. The Duo is then out of
  devicectl's reach, so the M6's Sill.log is the evidence (a picked window's virtual display line gives the monitor's
  size from the viewport).
- **Found and fixed (2026-11-16)**: the car monitor reports 1920 × 1080 points (Sill's virtual display was asked
  3840 × 3870, the square of 1920 ÷ 0.5 and a 30-point menu bar). macOS ran that display with no modes ("backing
  0×0 px") five times, so every picked window fell back to its own shape, bars on the monitor. `VirtualDisplaySize.
  maxSide` goes from 4096 to 2720 points (2720 × 2750 is the largest Sill.log shows at 2x); a 1080p monitor now gets
  2720 × 2720, with room for its 1920 × 1080 window at Aa's 1x. Checks: `capture-scale` 179 of 179, its 51 mutants
  caught (V9 now raises the ceiling back to 4096), `stage-release`, `repo-rules`, `swift build -c release`.
  Not installed or tried live yet: `make-app.sh --install` quits Sill.app, so it waits until no device is connected;
  then `--virtual-display-selftest` at sizes between 2720 and 3840 finds macOS's real limit and the cap is raised to it.
  Upstream (Saffsanity/sill main) has the same 4096 and no issue for it: one to offer back.
- **Untested**: a window filling the monitor once the fix is installed, an iPad, AirPlay, glasses.
