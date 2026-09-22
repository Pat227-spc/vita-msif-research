# Capture setup - session #1 and after

Goal of capture #1: a complete cold-insert -> mount (or mount-fail) trace of an **official** Sony Vita memory card, with enough timing to decode TPC framing. Auth payload interpretation comes later.

Do not start by desoldering the motherboard slot.

## Bill of materials (bench)

Owner already has a deep parts drawer. Do **not** check out an Amazon cart unless he says pay.

Minimum:

- USB logic analyzer
  - Comfort: Saleae Logic 8 (~$499 class)
  - Budget: DreamSourceLab DSLogic Plus (~$149 class)
- 30 AWG magnet wire or enamel-free wrap wire
- RMA flux, fine iron or hot-air for *card-side* taps only
- Test hooks / pogo / 0.1" header on a breakout
- ESD wrist strap, grounded mat
- Sacrificial official Sony Vita MEMORY card (preferred) **or** a remade RichDevX-style interposer
- Second known-good official card kept uncut for the daily machine
- Vita under test (note model: 1000 OLED vs 2000 Slim, firmware, CFW yes/no)

Optional later: second analyzer / analog channel on VCC to catch brownouts.

## Topology (in order of how much you are allowed to destroy)

1. **Best:** official card contacts wired, card still inserts into an unmodified Vita. Strain-relieve wires so the slot lid does not shear them.
2. **Good:** RichDevX-style breakout - Vita slot <- breakout <- official card. Tap the breakout.
3. **Last:** lift the motherboard connector and solder a flex. Pitch / pin-1 still unverified. Do not do this for capture #1.

## Analyzer settings

| Setting | Start here | Why |
| --- | --- | --- |
| Sample rate | 50-100 MS/s | Classic MS CLK is often ~20 MHz; 4-5 samples/period is the floor. If SCLK is slower, you can drop rate on capture #2. |
| Threshold | 1.65 V as first guess on a 3.3 V rail | Confirm VCC with a meter first. If I/O is 1.8 V, drop threshold. |
| Channels | SCLK, BS, D0, D1, D2, D3, INS | See PINMAP.md |
| Trigger | falling INS, **or** first rising SCLK after INS low | Record both a pre-roll (20-50 ms) and several hundred ms after clocks start |
| Depth | as deep as the box allows | Init + auth is a burst, then idle, then more. Better to over-record. |

Save native session **and** an export PulseView can open (`.sr` / VCD / CSV of the burst). Saleae `.sal` alone is a vendor lock-in risk.

## Session script (cold insert)

Power the Vita off. Analyzer armed. Then:

1. Photograph the wiring. Include a ruler and a note of which wire is which color.
2. Fill `captures/YYYYMMDD_model_*/notes.json` *before* you forget voltages.
3. Power on with **no** card (optional baseline: confirm INS stays high).
4. Power off.
5. Insert instrumented official card. Power on. Let the OS sit on the live area / settings -> format-or-mount path. Do not yank the card.
6. Stop capture. Save.
7. Repeat once. Two files cost nothing; a single clipped burst is useless.
8. If CFW is present, a *second* pair of captures with taiHEN plugins disabled vs enabled is useful later. Label them. CFW is not required for capture #1.

Also capture, when you have time:

- Warm reboot with card already inserted
- Sleep / wake (mediaid `sleep_session_id` path on henkaku wiki)
- Settings -> format flow on a junk official card (do not format the daily card)

## What "good" looks like

A usable capture #1 has:

- INS falling (or already low) before SCLK activity
- Clean SCLK with no threshold chatter
- BS toggling in a repeating frame around data
- D[3:0] changing in lockstep with SCLK during bursts
- A notes.json that states model, FW, analyzer, rate, channel map, and whether the UI mounted the card or asked to format

A pretty Saleae screenshot with 2 channels and no notes is not a capture.

## Publish layout

```
captures/YYYYMMDD_pch1000_fw365_coldinsert/
  notes.json
  capture.sr            # or .sal + exported .sr
  photo_wiring.jpg
  README.md             # optional one-paragraph human summary
```

License for everything under `captures/`: CC0 (see LICENSE-CAPTURES).

## Decoder work *after* the file exists

1. Identify TPC bytes on the BS-high phase (classic MS: 4 bits + 4 inverted bits, with BS early-flip nastiness - verify, do not assume).
2. Mark the first frames after insert. Those should contain or precede `0x48` / `0x49` if henkaku wiki mapping is right.
3. Do **not** pick U1 silicon until this framing is boringly repeatable.

## Safety

- Magnet wire on a live 3.3 V bus is fine; shorting VCC to a data pin is not.
- Official cards are scarce and expensive. Cut one, keep one.
- The motherboard MEMORY connector is harder to replace than a $80 used card. Act like it.
