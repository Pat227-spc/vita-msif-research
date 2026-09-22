# PINMAP - Vita MEMORY (MSIF)

**Verification state:** signal *names* and INS polarity are wiki/Twitter-era (Asdron + RichDevX).
**Unverified on this bench:** left->right pad order on the motherboard after slot removal, pitch, pin-1 orientation for PCH-1000 OLED vs PCH-2000 Slim.

Do not treat this file as a fab netlist.

## Card-edge signals (psdevwiki)

Source: https://www.psdevwiki.com/vita/Memory_Card
Pinouts by Asdron, confirmed by RichDevX.

| Pin | Signal | Direction (typical) | Notes |
| --- | --- | --- | --- |
| 1 | INS | Card -> host detect | Host controller input, pulled up high. Card ties this to ground when inserted (RichDevX). Treat as active-low card-present. |
| 2 | SCLK | Host -> card | Serial clock. Memory Stick family: host-driven, idle low in classic MS. **Measure idle level on Vita before writing a decoder.** |
| 3 | VCC | Host -> card | Card VDD. Measure voltage on a live official card before designing a regulator (expect ~3.3 V class; do not assume 1.8 V I/O). |
| 4 | D3 | Bidirectional | Data 3 |
| 5 | D2 | Bidirectional | Data 2 |
| 6 | D1 | Bidirectional | Data 1 |
| 7 | D0 | Bidirectional | Data 0. Classic serial MS used D0 as SDIO; Vita is documented as 4-bit parallel names. |
| 8 | BS | Host -> card | Bus State. Frames TPC vs data in Memory Stick. |
| 9 | VSS | Ground | Ground |

9 pins total. Game-card slot is a **different** 10-pin connector. Do not mix.

## What INS actually tells you

- Host sees insert when INS is pulled down by the card.
- A dummy grounded INS + floating data lines can make the OS notice "a card" and throw UI (including "format?"). That is **not** auth success.
- For capture #1, INS is a trigger channel, not a protocol channel.

## microSD is a different pin language

Do not 1:1 these.

| microSD (SD mode, shorthand) | Vita MEMORY |
| --- | --- |
| DAT3 / DAT2 / DAT1 / DAT0 | D3-D0 are *named* similarly and are still not the same protocol |
| CMD | no CMD pin - Vita uses TPC over D[3:0] framed by BS |
| CLK | not the same clock domain or command set as SCLK |
| VDD / VSS / CD | voltages and detect are host-specific |

A passive flex that shorts "DAT0<->D0, CLK<->SCLK" will produce garbage or a format prompt. That is the richdapter failure mode.

## Motherboard pads after slot removal

**Status: UNCERTAIN until probed.**

Need photographs + caliper + continuity from each gold finger / spring to a known card-edge pin, separately for:

- PCH-1000 / 11xx OLED
- PCH-2000 Slim

Record in the first hardware probe session:

- [ ] Pin-1 end (which physical end of the connector is INS)
- [ ] Whether the MB pad row is mirrored vs the card-edge drawing
- [ ] Pitch (mm), pad count, extra mechanical / shield / detect springs
- [ ] Presence of series resistors or EMI parts between connector and SoC
- [ ] VCC rail source and measured voltage at the connector under insert

Until those boxes are ticked, any KiCad host-zone footprint is a **placeholder**.

## Capture channel assignment (suggested)

Analyzer has 8 digital channels on the cheap/comfort units named in CAPTURE_SETUP.md. Map:

| LA CH | Signal | Why |
| --- | --- | --- |
| 0 | SCLK | clock; decoder sample clock |
| 1 | BS | transaction framing |
| 2 | D0 | |
| 3 | D1 | |
| 4 | D2 | |
| 5 | D3 | |
| 6 | INS | insert / trigger |
| 7 | spare / VCC-valid / card-reset if found | do not waste CH7 on a second ground |

GND of the analyzer **must** bond to card VSS / host ground at the breakout, not through a random shield.

## Classic Memory Stick vs Vita MEMORY

Same *family* of wire names (`BS`, `SCLK`, 4-bit data).
Different: connector pin order, card form factor, and the `0x48-0x4B` + ECDSA-224 + 3-DES-CBC-CTS layer.

Use Dmitry.GR / MS TPC docs only as a starting decoder, then diff against capture #1.
