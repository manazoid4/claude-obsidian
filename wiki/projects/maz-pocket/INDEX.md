---
title: MAZ Pocket
created: 2026-08-13
updated: 2026-08-13
type: project
tags: [projects, maz-pocket, hardware, firmware, cardputer, esp32, voice, github]
---

# MAZ Pocket

Standalone firmware for the **M5Stack Cardputer ADV**, installable through
M5Launcher alongside Bruce and Nemo. The hardware front end for the MAZ
assistant stack: hold SPACE, talk, and the WAV lands on the card.

## Locations

- Local repo: `C:\Users\manaz\maz-pocket`
- GitHub: `https://github.com/manazoid4/maz-pocket`
- Binary: `.pio/build/cardputer-adv/firmware.bin`

## Why it exists

v0.1 is deliberately local-only. Its job is to make the ADV's voice, audio,
storage and network foundations real so that v0.2 can swap
`record -> local playback` for `record -> Wi-Fi -> [[openflowkit]] -> [[mazos]]
-> TTS` by changing a `voice::Sink` implementation and nothing else.

The strategic point, stated plainly: **on its own this is a nicer notepad**.
Its value is as the physical capture end of OpenFlowKit. See
[[2026-08-13-maz-pocket-v0.1-build]] for the honest utility assessment.

## Hardware facts worth remembering

- The ADV is **not** a Cardputer with a bigger battery. Its keyboard moved
  behind a **TCA8418 I2C expander** (addr 0x34, INT on G11) — which is why
  pre-ADV firmware boots on an ADV with a **completely dead keyboard**. MAZ
  Pocket ships its own driver.
- Audio moved to an **ES8311 codec** (I2C 0x18). One codec serves both
  directions, so mic and speaker cannot be live at once.
- **No PSRAM.** 512KB SRAM total, so audio streams to storage block by block.
- SD CS is **G12**, not the G5 printed on the M5 docs page. M5Unified's ADV
  table and M5Stack's own ADV UserDemo both say G12.
- `M5Unified` 0.2.19 is the first registry release with
  `board_M5CardputerADV`. Pinned exactly; floating it silently breaks audio.

## Current status

- [[2026-08-13-maz-pocket-v0.1-build]] — v0.1 built and committed. Compiles
  clean (RAM 16.1%, flash 34.4%). **No hardware test has happened** — no device
  was attached. `docs/VERIFICATION.md` lists the eight checks that need the
  device, in order, with the keyboard driver flagged as the highest risk.

## Next

Three things, and only three, before anything else is added:

1. Run the hardware checklist and fix whatever the keyboard driver got wrong.
2. Wire `voice::Sink` to OpenFlowKit over the LAN — the whole point.
3. Decide by the seven-day carry test whether this project continues.
