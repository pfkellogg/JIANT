# JIANT FM + FM-1 Big Screen

[JIANT FM](https://github.com/juanjiant-bit/JIANT) by juanjiant (a fork of Leo Kuroshita's
[Felucca](https://github.com/hugelton/Felucca), with its song system from [SLOOP](https://github.com/isod89/sloop-fm1)),
with one addition: while an app asks for it, the FM-1 sends its screen over its USB cable, so
[FM-1 Mirror](https://github.com/pfkellogg/fm1-mirror) (Mac, Windows, Linux) and
[FM-1 Mirror + Spyglass](https://github.com/pfkellogg/fm1-mirror-spyglass-android) (Android) show it big, live. For
players who can't read the FM-1's 1.54-inch screen.

This is an **unofficial** build, not a JIANT release, and juanjiant has nothing to do with it. Everything else is
JIANT FM **Beta V10.49** (commit `cf1bf3d`, 2026-11-03) as published. Tested on a real FM-1 (2026-11-03): HOME, the
pages, the PRESETS list, with FM-1 Mirror on a Mac.

**Watch it:** [JIANT FM on a Big Screen: A Second Community Firmware with FM-1 Mirror](https://youtu.be/7o5s0EDJX5c) (4:57): every install step, slowly, and the presets on the big screen.

A fourth, not released yet: **SLOOP + FM-1 Big Screen**, tested on a real FM-1 (2026-11-04). Watch it: [SLOOP on a Big Screen](https://youtu.be/Pd4Bm6JvtkU).

A third community firmware with the same addition: **[Skiff + FM-1 Big Screen](https://github.com/pfkellogg/skiff/blob/big-screen/BIG_SCREEN.md)**;
all three come from Felucca ([the family tree](https://github.com/pfkellogg/skiff/blob/big-screen/FIRMWARE_FAMILY.md), [video](https://youtu.be/lmMT-FtAoSE)).

## JIANT has daily builds

JIANT changes almost every day: a new beta, new features, fixes. This build is **one day's JIANT** (V10.49) plus the
screen. It doesn't update itself. When JIANT moves on, this build stays at V10.49 until the screen is added to a newer
JIANT and released here again (the release names say which JIANT is inside).

You can always go back to the newest JIANT: its [web installer](https://juanjiant-bit.github.io/JIANT/webapp/installer/)
installs it over this one as an ordinary JIANT update (your projects stay). That JIANT has no big screen.

## Installing it

1. **If the FM-1 runs another custom firmware** (ChoralRoot, Felucca, Skiff, M-VAVE's Open firmware, Big Screen): put
   M-VAVE's original V15 back first, on M-VAVE's [FM1 Workbench](https://open.m-vave.com/fm1-workbench) (Chrome or
   Edge): **Connect device**, then **Device update › Restore original V15**. JIANT says so too: written straight over
   another custom firmware, it can leave the FM-1 unable to start. (On V15 or JIANT already: skip this.)
2. **Charge the FM-1, plug it straight into the computer** (no hub), close every app that uses it.
3. With this repository and Python 3 (`pip3 install mido python-rtmidi`), from its folder:

   ```
   python3 tools/fm1_install.py JIANT-V10.49-BigScreen.fwsc --yes
   ```

   (the `.fwsc` from this fork's Releases). It writes for about a minute, then the FM-1 restarts.
4. **Unplug the USB cable and plug it back in** if the computer doesn't see the FM-1 after the restart.

## Seeing the screen

Open **FM-1 Mirror** (Mac, Windows, Linux) or **FM-1 Mirror + Spyglass** (Android) with the FM-1 plugged in by USB:
the screen appears, big, in colour or high-contrast looks.

**The sound playing, on top of the spyglass.** The FM-1 also tells the app what's playing: the selected track, its
engine and its sound, as JIANT's header shows it (`T1 FM6 ICE BELL`). FM-1 Mirror shows it in gold on top of its
spyglass and follows the PRESETS knob and track changes. (It's sent only while an app asks, like the screen.)

**USB only.** JIANT doesn't use the FM-1's Bluetooth MIDI, so **FM-1 Mirror Bluetooth** can't show it. Bluetooth
needs firmware with Bluetooth MIDI: the FM-1 Big Screen firmware (M-VAVE's Open firmware + the screen) has it.

## What changed in the code

The same drop-in as [ChoralRoot + Big Screen](https://github.com/pfkellogg/ChoralRootFM1/blob/big-screen/BIG_SCREEN.md):
JIANT and ChoralRoot are both built on Felucca, so the screen goes in at the same places. **38 lines in JIANT's own
files**, and the same **7 of them do the work**; the rest are an on/off switch, `JIANT_BIG_SCREEN` (1 in `felucca.c`):

| File | Lines | What |
|---|---|---|
| `firmware/src/lcd.c` | 9 | `mirror_lcd(...)` in `lcd_fill`, `lcd_blit` and `lcd_blit_stride`: every rectangle on its way to the LCD |
| `firmware/src/usb.c` | 14 | where `sysex_byte` finishes a message: a 6-byte `F0 7D 'F' 'S' .. F7` goes to `sm_sysex`; the SysEx send ring's size (`SXQ`) can be set from outside |
| `firmware/src/main.c` | 3 | `mirror_service()` once a main-loop pass |
| `firmware/src/felucca.c` | 12 | the switch, `#include "jiant_mirror.c"`, and `SXQ 512` (the screen at ~67 KB/s, not ~25) |

Two differences from ChoralRoot: JIANT also draws parts of a wider buffer (`lcd_blit_stride`), so the adapter takes a
row stride; and JIANT only redraws the cells that changed, so a new viewer's whole screen also marks every cell stale
(`dc_stale = 1` with `ui.force = 1`).

Two new files, both MIT:

- `firmware/src/jiant_mirror.c` (89 lines): the adapter. Each row goes back into RGB565 for `sm_rows`; each message
  goes into usb.c's SysEx queue only when it has room (never waiting on USB); a new viewer, or rows that had to be
  dropped, get the whole screen again (at most twice a second). A draw never waits more than 0.4 s. Every 100 ms it also puts
  the sound playing together (JIANT's own `sound_name()`, as the header does) and hands it to `sm_name`.
- `firmware/src/screen_mirror.c` / `.h`: the firmware-neutral screen mirror from
  [pfkellogg/fm1_open, `screen_mirror/`](https://github.com/pfkellogg/fm1_open/tree/screen-out/screen_mirror), with a
  6 KB buffer.

Size: image 569,872 of 584,336 bytes; RAM 88,836 of 98,304 bytes.

## Building it

As JIANT (BUILDING.md), or on a Mac without Docker: `tools/build.py` also takes JieLi's **Windows** toolchain 2.5.3
under Wine (the same clang 4.0.1 for pi32v2):

```
JIELI_WINE=/path/to/wine JIELI_WIN_BIN=/path/to/JL/pi32/bin ./build.sh
```

## Licence

JIANT and Felucca: GPL-3.0-only (LICENSE, LICENSING.md), and so is this build as a whole; its source is this branch.
`jiant_mirror.c`, `screen_mirror.c` and `screen_mirror.h`: MIT, © 2026 Patrick F. Kellogg, so any FM-1 firmware can
use them. M-VAVE and FM-1 are M-VAVE's names.
