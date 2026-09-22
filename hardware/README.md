# Hardware \u2014 constraints only

There is **no fab-ready board in this repository.**

An earlier conversation referenced a KiCad scaffold at `/workspace/vita-mem-microsd-adapter/` (host zone + flex neck + stiffener island, U1 = bridge placeholder). That tree was **not** present in this sandbox when the public research pack was created (2026-09-21). If it still exists on the owner\u2019s disk, copy it here later as `hardware/kicad/` \u2014 original layout only.

Until capture #1 and a U1 choice exist, treat any PCB as a mechanical cartoon.

## Intended shape (when we get there)

Inspired by NatalieTheNerd\u2019s *method*, not her files:

- Desolder Vita MEMORY connector (last, after pad map is verified)
- Flex or rigid-flex from those pads
- Stiffener island elsewhere in the shell for:
  - bridge MCU/FPGA (U1)
  - push-push microSD
  - decoupling, 1.8/3.3 regulation if needed

Requirements that make this unlike a PSP Go flex:

- U1 is an **active protocol bridge**, not a pin map
- Host must see MSIF timing + (eventually) auth
- microSD side is standard SD/SPI to whatever U1 you pick
- INS on the host side must still look like a card is inserted

## Electrical constraints

See [CONSTRAINTS.md](CONSTRAINTS.md) and `../PINMAP.md`.

## What is allowed to be designed now

- Mechanical envelope sketches
- Breakout / interposer for capture (RichDevX-style) \u2014 this is the useful next PCB if any
- Test-hook land pattern

## What is not allowed now

- Fab submit
- \u201cJust like Natalie\u2019s Go board but Vita silk\u201d
- Passive D0\u2013D3 \u2194 DAT0\u2013DAT3 wiring sold as a product
- Buying U1 reels \u201cso we have them\u201d
