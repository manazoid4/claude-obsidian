---
title: MAZ Pocket v0.1 build
created: 2026-08-13
updated: 2026-08-13
type: session
tags: [maz-pocket, firmware, cardputer, esp32, voice, research]
---

# MAZ Pocket v0.1 — build session, 2026-08-13

Built the first version of [[maz-pocket]]: standalone Cardputer ADV firmware,
launchable via M5Launcher, architected as the hardware end of the
[[openflowkit]] / [[mazos]] voice pipeline.

## What shipped

Modular C++/PlatformIO firmware, ~3,400 lines across `core/ ui/ apps/ audio/
input/ net/ storage/`:

- **Custom TCA8418 keyboard driver** — the ADV-specific work most third-party
  firmware gets wrong.
- **`voice::Sink` pipeline** — WAV today, network next. Call/Capture/Recorder
  all sit on it and none of them know where the audio goes.
- **Shell** — app registry, `Ctrl+K` command palette with subsequence matching,
  one notification system, a Focus timer that survives navigation, screen
  timeout, dirty-flag repainting.
- **Apps** — Call, Capture, Notes, Focus, Tasks, Recorder, Tools, Settings,
  Connections, Help, plus six research-backed utilities (Calculator, Stopwatch,
  QR, Text Viewer, Generator, Snippets).
- **Storage that degrades** — SD, else LittleFS, else settings-only via NVS.
  This matters because M5Launcher flashes with *its* partition table, so our
  LittleFS partition may not exist at runtime.
- **All artwork drawn in code.** No third-party assets, no Bruce branding, so
  MIT stands cleanly against Bruce's AGPL and Nemo's GPL.

Build: `pio run` SUCCESS first attempt. RAM 16.1%, flash 34.4% of 3MB.

## The research finding that changed the product

Community research (r/CardPuter, M5Stack forum, RPi magazine review) says
something uncomfortable: **most Cardputer owners cannot name a daily use.** The
most-cited actual activity is *hopping between firmware*. Nobody describes
carrying one for note-taking.

Two consequences, both acted on:

1. Tools was cut down to pure diagnostics rather than grown into a scanner
   suite — that space is fully served by Bruce, and duplicating it would make
   MAZ Pocket less distinct, not more useful.
2. The whole bet rests on voice. A physical push-to-talk button on a device with
   no feed behind it is the one thing the ADV does that a phone makes awkward.

Rejected despite being popular: MP3 player (duplicates a phone, no PSRAM),
Meshtastic (needs a LoRa cap, dedicated firmware exists), all offensive security
tooling (Bruce's job), emulators. Deferred: IR remote and a game — both
unverifiable without hardware in hand.

## Honest assessment of value

Asked directly how useful this is. The answer given, and worth keeping:

- **On its own, low.** A phone beats it at notes, tasks, calculator and QR.
- **As the capture end of OpenFlowKit, real.** The WAVs it writes are already
  the exact input OpenFlowKit consumes. That makes it dogfood plus a stronger
  portfolio artefact than another dashboard.
- **The risk is real too.** This is project #9 while [[flowlens]] needs revenue.
- **The test:** carry it for seven straight days. If yes, build v0.2 at once. If
  no, it was a good weekend and it stops.

## What is NOT true yet

No hardware test happened — no device was attached during the build. The
firmware compiles and the pin map came from vendor source rather than guesswork,
but nothing has been observed running. The keyboard driver is the highest-risk
component: written from the TI datasheet plus M5's ADV remap arithmetic, never
executed.

## Next three (and only three)

1. Hardware checklist in `docs/VERIFICATION.md`, keyboard first.
2. Network `voice::Sink` to OpenFlowKit on the LAN.
3. Multi-line note editor — the 400-character single-line field is the weakest
   part of the daily-use story.
