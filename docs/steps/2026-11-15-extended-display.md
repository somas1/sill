**The Mac on an iPhone's or iPad's external display: #301 merged in, and the plan for an extra Mac screen sized to the monitor (2026-11-15, branch `extended-display` in the fork somas1/sill, from main at aea3547).**

- **What John asked for**: with an iPhone or iPad plugged into a monitor, TV or display glasses, the monitor shows the
  Mac instead of mirroring the phone; sometimes the Mac as it is, sometimes an extra Mac screen the monitor's size.
- **Merged in**: Saffsanity/sill#301 (addhumal, with kolyan94's iOS 27 commits 359facf4, ace802b3, 28e0c540) at
  218c8434, then main at aea3547 (d7433f2): no conflicts; the five files of #301 only.
- **The plan**: `docs/extended-display-plan.md`. Part 1 is #301, waiting for a test on real hardware. Part 2 is a
  small version of upstream's Virtual Desktop (#352) with its keys, sized to the monitor from #301's viewport and an
  optional `Viewport.externalScale`; option B there, the default unless John chooses otherwise. Part 3 (standalone
  glasses) is parked.
- **Verified**: nothing built or run (the shell this was done from has no Swift or Xcode).
- **Untested, for John**: every step of the plan's "Part 1: what is left".
