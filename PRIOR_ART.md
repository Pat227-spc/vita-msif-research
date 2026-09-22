# Prior art

Cite these. Do not pretend they are this project.

## Hardware pinout and early bus notes

### Vita Developer Wiki - Memory Card

- https://www.psdevwiki.com/vita/Memory_Card
- 9-pin MEMORY card: `INS SCLK VCC D3 D2 D1 D0 BS VSS`
- Pinout attributed to [@Asdron_](https://twitter.com/Asdron_/status/781948076281954304), confirmed by RichDevX.
- INS (pin 1): host input, pulled up; card side is grounded (RichDevX).
- Soft RE note from RichDevX: `magicgate.skprx` did **not** appear to be in the idle/mount path he looked at. That does **not** mean there is no MagicGate-class auth; henkaku-wiki later documents a dedicated MC init/auth path in `SceMsif` + `rmauth_sm.self`.
- HW RE (RichDevX): PulseView waveforms; "based on Memory Stick Pro, maybe a different command set."
- ViMC-Decoded: minimal protocol decoder (RichDevX / @gameshack_ logo). Binary capture format note:
  `7_________________________0` / `[X] [X] [X] [D3][D2][D1][D0][BS]`
- Also: https://www.psdevwiki.com/vita/Media (gamecard 10-pin vs memory card 9-pin; SD pinout for contrast).

### Vita Developer Wiki - Memory Card Development Board

- https://www.psdevwiki.com/vita/Memory_Card_Development_Board
- RichDevX breakout: insert into Vita MEMORY slot, official card plugs into the board, tap signals with an analyzer.
- This is the preferred first-capture topology (do not lift MB pads first). Gerbers from that 2016-era board are not assumed available; remake from photos + pinout if needed.

## Software RE (auth is here)

### henkaku wiki - Memory Card

- https://wiki.henkaku.xyz/vita/Memory_Card
- Driver: `SceMsif`.
- Partitions on a real card:
  - `0xD` raw @ `0x400000` size `0x400000` (mediaid / "some data")
  - `0x8` exFAT `ux0` @ `0x800000` to end of card
- `mediaid` is reachable as `sdstor0:mcd-lp-act-mediaid`. Includes `sleep_session_id` written before sleep; mismatch on wake -> error / power off. You cannot hot-swap a MEMORY card in sleep.
- Hypothesis (still labeled unconfirmed on the wiki): Vita MC is a new-generation Memory Stick + extra auth layer. Evidence cited there:
  - TPC protocol support in `SceMsif`
  - driver handles Memory Stick "Model Name Device Information Entry"
  - dumped card has a valid Attribute Information Area Confirmation Structure
- Vendor-ish commands `0x48 0x49 0x4A 0x4B` (wiki analogy: SD CMD56).
  - `0x48` + `0x49` required before the card can be read
  - `0x4A` + `0x4B` hypothesized as genuine-check; not fully tested as required for raw I/O
- Init also uses `rmauth_sm.self` functions 1 and 2:
  - fn 1: obtain 1 byte from Cmep
  - fn 2: take a key returned by the card, install into SceSblDMAC5 keyring `0x1C`
- Cipher during MC auth: **3-DES-CBC-CTS**
- Card-unique key is **ECDSA-224 signed on a custom curve**
- That last sentence is why a COTS MS<->SD bridge chip is not enough, and why richdapter died.

### motoharu-gosuto

Software RE, **not** a finished fake-card firmware.

| Repo | Why it matters |
| --- | --- |
| https://github.com/motoharu-gosuto/psvcmd56 | Kitchen-sink reversed drivers. README lists **SceMsif - auth protocol of memory card**. Also SceSdif / CMD56 (game card - different bus). |
| https://github.com/motoharu-gosuto/psvemmc | User+kernel path to read eMMC / game cart / memory cart at sector level via SceSdif **and** SceMsif APIs. Useful later for comparing what the official card returns after auth. |
| https://github.com/motoharu-gosuto/psvgamesd | Game-card dump/run. Do not confuse CMD56 (game) with MC `0x48-0x4B`. |
| https://github.com/motoharu-gosuto/psvkirk | F00D/Kirk proxy used as a black box for **game-card** CMD56. Same *idea* (crypto in F00D) may apply to MC `rmauth_sm`, but it is not a copy-paste. |

## Why the MEMORY adapter died and SD2Vita won

### Cimmerian - SD2Vita history

- https://github.com/Cimmerian-Iter/Vita-troubleshooting-guide/blob/master/sd2vita-and-memory-corruption/sd2vita-history.md
- Pre-SD2Vita options: pay Sony for official MC, or PSVSD (3G Vita: replace modem, USB mass-storage path).
- **richdapter** (RichDevX): explicit attempt to clone the *PSP Go microSD-adapter idea* onto Vita MEMORY.
- Cimmerian's post-mortem:
  - overpriced / more talk than progress
  - public demo that made the Vita ask to **format** the card - that only proves the host saw *something*, not that MicroSD contents were readable
  - **memory-card authentication protocol was not totally reversed**
  - project died; no product
- SD2Vita succeeded on a different connector: game-card slot is "SD + CMD56 auth." xyz hooked the prototype/devkit SD driver path to that slot; motoharu helped on protocol; gamesd.skprx shipped; Gadorach / others made the cheap PCB. That stack is **solved storage**. It does not make a MEMORY-slot fake card.

## Method inspiration (not to copy)

### NatalieTheNerd - PSP Go microSD

- Product: https://nataliethenerd.com/products/psp-go-micro-sd
- Install: https://nataliethenerd.com/pages/psp-go-micro-sd-adaptor
- Method used here as inspiration only: desolder proprietary slot, solder a flex to the gold fingers, relocate a push-push microSD.
- PSP Go talks Memory Stick Micro (M2). Commercial or discrete MS<->SD bridges exist for that generation. Vita MEMORY does not have an equivalent COTS auth-speaking chip.
- No public Gerbers found 2026-09-21. Do not recreate her outline.

## Memory Stick physical / TPC (generic, pre-Vita)

Useful for interpreting `SCLK` + `BS` + `D[3:0]` once capture #1 exists. Vita may diverge.

- Dmitry.GR Memory Stick notes: https://dmitry.gr/index.php?r=05.Projects&proj=31.%20Memory%20Stick
  - BS marks bus state; CLK host-driven; data bidirectional; direction turnaround is one bit-time; clock idle low; data latched rising edge
  - TPC: 8-bit code, low nibble is inverse of high nibble
- Classic MS 4-bit parallel uses `SCLK, BS, DATA[3:0]` - same *names* as Vita MEMORY.
- Classic MS pin **order** is **not** the Vita MEMORY order. Do not wire by name-to-name from an MS pinout table onto a Vita slot.

## What this project is allowed to reuse

| Source | Reuse |
| --- | --- |
| psdevwiki pin names / RichDevX INS note | Cite + verify on a real card |
| henkaku wiki command/crypto outline | Cite; treat numbers as hypotheses until a dump matches |
| motoharu reversed C | Read, cite, do not vend as our firmware |
| Cimmerian history | Cite why we are not repeating richdapter's "format?" demo |
| Natalie method | Mechanical *idea* only |
| SD2Vita / gamesd / YAMT / StorageMgr | Out of scope except as the daily-storage workaround |

## What does *not* exist (checked 2026-09-21)

- A public, working Vita MEMORY <-> microSD active bridge
- A COTS chip documented to speak Vita `0x48-0x4B` + ECDSA-224 card cert + 3-DES-CBC-CTS session
- Natalie's Gerbers
- Complete RichDevX richdapter source / ViMC-Decoded tree in a durable public repo (Sendspace-era links on psdevwiki are stale risk)
