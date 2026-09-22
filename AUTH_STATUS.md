# Auth RE status vs SD2Vita "good enough for storage"

Short version for the bench owner:

**You already have working cheap storage.** SD2Vita + StorageMgr (or YAMT / gamesd) mounts a microSD in the **game-card** slot. That path is `SceSdif` + CMD56. It is a different connector, a different driver, and a solved problem. Keep using it for daily ux0.

**A MEMORY-slot microSD bridge is a different product.** The host (`SceMsif`) expects a Sony-shaped Memory Stick cousin that can finish an authentication handshake. No known COTS MS<->SD chip does that handshake. Until dumps + a bridge that speaks it exist, the MEMORY slot stays official-card-only.

## What is already known (software RE)

From henkaku wiki Memory Card page and motoharu's `psvcmd56` tree (SceMsif listed; not a finished fake-card):

| Piece | Status | Meaning for a fake card |
| --- | --- | --- |
| Physical bus names | wiki pinout | Need capture to confirm timing, idle levels, bit order |
| TPC / Memory Stick-like framing | hypothesized + driver evidence | Decoder target for capture #1 |
| Attribute / model-name structures | wiki says real dumps look like MS | A bridge may need to emulate MS attribute area, not only SD sectors |
| Commands `0x48`, `0x49` | required before reads | Must implement or the host never talks storage |
| Commands `0x4A`, `0x4B` | probable genuine-check | May be skippable for raw I/O; likely **not** skippable if you want the OS to trust the card |
| `rmauth_sm.self` fn 1 / 2 | wiki | Host-side: 1 byte from Cmep; card-supplied key loaded to DMAC5 keyring `0x1C` |
| 3-DES-CBC-CTS | wiki | Session crypto. Bridge has to keep pace, not just ACK TPC |
| ECDSA-224, custom curve, per-card key | wiki | This is the wall. A random MCU cannot mint a Sony-trusted cert. Options are: protocol-level bypass on a *modded* host, a leak of the verify path, or living with "unofficial card" behavior - none of those are done |
| `mediaid` + `sleep_session_id` | wiki | Even a "working" card must persist a small raw partition or sleep/wake breaks |

RichDevX circa 2016: `magicgate.skprx` did not look central to the traces he had. That observation is compatible with auth living in `SceMsif` + `rmauth_sm` instead. It is **not** evidence that auth is absent.

## What richdapter proved, and what it did not

Cimmerian's history: RichDevX aimed at a PSP-Go-style MEMORY adapter. Public demo: Vita asked to **format** the card.

That UI means:

- INS (or equivalent) convinced the host a card was present, and/or
- enough of the bus wiggled that `SceMsif` started init and then failed

That UI does **not** mean:

- TPC init completed
- `0x48`/`0x49` succeeded
- the ECDSA-signed card key was accepted
- the host could read the exFAT at `0x800000`

Treat "format?" as a **fail code**, same class as "the card is damaged."

## SD2Vita is not a shortcut to this

| | Official MEMORY | SD2Vita |
| --- | --- | --- |
| Slot | MEMORY (9-pin) | Game card (10-pin) |
| Driver | `SceMsif` | `SceSdif` + plugin remap |
| Auth | MC `0x48-0x4B` + rmauth + 3DES + ECDSA-224 | Game-card CMD56 (also proprietary, **already bypassed in software** by gamesd/YAMT) |
| Needs CFW | No (official card) | Yes |
| Cheap microSD today | No | Yes |
| Frees MEMORY slot | n/a | Yes - official card can stay out or become uma0 |

Building a MEMORY bridge so a *stock* Vita can use microSD is the hard, interesting problem. Building a MEMORY bridge that only works on a hacked Vita is easier *if* `SceMsif` can be patched to skip auth - that is a firmware project, not a passive flex, and it still needs a correct TPC/storage emulator. Neither is in this repo yet.

## Honest options (none selected)

1. **Full fake card on stock firmware** - implement TPC + attribute area + `0x48-0x4B` + whatever verify F00D expects. Blocked on dumps + crypto. Highest value, highest risk of stalling the same way richdapter did.
2. **CFW skip-auth + dumb MS-like storage emulator** - patch `SceMsif` / rmauth to accept a non-Sony card, then implement enough TPC that reads/writes hit a microSD. Requires a hacked Vita forever. Still needs capture #1 so the emulator is not guesswork.
3. **Stop and keep SD2Vita** - already good enough for storage. MEMORY slot work is optional R&D.

Capture #1 is required before choosing 1 vs 2. Choosing a U1 (RP2040, small FPGA, etc.) before framing dumps is how you waste a board spin.

## What a modded Vita gives you

Useful:

- Dump `SceMsif` / `rmauth_sm.self` and compare to motoharu
- Log TPC at the driver instead of only on the wire
- Experiment with skip-auth patches *after* you can see the handshake

Not useful by itself:

- "I have Enso, therefore I can solder a microSD to the MEMORY pads"
- Kernel access does not mint Sony ECDSA-224 signatures
