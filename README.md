# vita-msif-research

Open reverse-engineering of the **PlayStation Vita MEMORY-slot bus** (MSIF + MagicGate-class auth) toward an original **active MSIF \u2194 microSD bridge** (MCU/FPGA on flex or rigid-flex).

This is **not** SD2Vita. SD2Vita lives in the **game-card** slot (`SceSdif` / gamesd / YAMT / StorageMgr). Daily storage on a hacked Vita stays on SD2Vita + StorageMgr until a MEMORY-slot bridge actually works.

**Status (2026-09-21): research pack + capture protocol only. No working fake card. No fab. No purchase.**

## Hard electrical truth

- Vita MEMORY contacts **are not** microSD pins. Passive wiring will not work.
- PSP Go had a workable protocol-bridge silicon path (M2 / Memory Stick Micro \u2194 SD). Vita MEMORY needs **host-visible authentication** that no known COTS MS\u2194SD chip speaks.
- Prior attempt **richdapter** (RichDevX) stalled: incomplete auth RE. A Vita \u201cformat this memory card?\u201d prompt is **not** a working card.
- Community solved *storage* via SD2Vita instead. Do not merge those requirements into this repo.

## What this repo is for

1. Publish bus dumps and notes so the work survives if the original bench stalls.
2. Keep pinout, mechanical, and prior-art claims **labeled as verified / wiki / unverified**.
3. Specify capture #1 so the first logic-analyzer session is repeatable.
4. Track auth RE status honestly versus \u201cSD2Vita is good enough for storage.\u201d

Hardware layout for a future adapter is **original**. NatalieTheNerd\u2019s PSP Go microSD install method (desolder proprietary slot \u2192 flex \u2192 relocated push-push microSD) is inspiration only. Do not 1:1 copy her Gerbers or product IP.

- Product (methods only): https://nataliethenerd.com/products/psp-go-micro-sd
- Install notes: https://nataliethenerd.com/pages/psp-go-micro-sd-adaptor
- No public Gerbers found as of 2026-09-21.

## Repo map

| Path | Purpose |
| --- | --- |
| [PRIOR_ART.md](PRIOR_ART.md) | Citations: psdevwiki, henkaku wiki, RichDevX, Cimmerian, motoharu, Natalie (methods) |
| [PINMAP.md](PINMAP.md) | MSIF pinout, INS polarity, what is still unprobed |
| [CAPTURE_SETUP.md](CAPTURE_SETUP.md) | Analyzer, wiring, sample rate, session checklist |
| [AUTH_STATUS.md](AUTH_STATUS.md) | What software RE already knows vs what a fake card still cannot do |
| [RULES.md](RULES.md) | Standing rules: no pay/fab without explicit OK; Vita-only; public dumps |
| [captures/](captures/) | `.sr` / Saleae / PulseView + `notes.json` \u2014 empty until capture #1 |
| [hardware/](hardware/) | Electrical/mechanical constraints for a future original bridge (not fab-ready) |
| [docs/](docs/) | Glossary + non-goals |

## Next milestone (capture #1)

Do **not** lift motherboard pads first.

1. USB logic analyzer (Saleae Logic 8 comfort, or DreamSourceLab DSLogic Plus budget) + 30 AWG magnet wire + flux + test hooks + ESD strap.
2. Prefer a **sacrificial official Sony Vita MC** with wires on the card contacts, or a remade RichDevX-style interposer.
3. Capture cold-insert / mount traffic: `INS`, `SCLK`, `BS`, `D0\u2013D3`, `GND`. Sample ~50\u2013100 MS/s.
4. Publish under `captures/YYYYMMDD_model_*/` with `notes.json`.
5. **Only after dumps:** choose bridge silicon (U1) and attack auth framing.

## License

- Documentation in this tree: [CC BY 4.0](LICENSE).
- Capture traces and `notes.json` under `captures/`: [CC0 1.0](LICENSE-CAPTURES) (reuse without asking).
- Do not drop third-party Gerbers, firmware blobs, or Sony dump images into this repo.

## Maintainer notes

Orange County bench. No Amazon / PCB fab / component order without an explicit \u201cpay\u201d / \u201cfab\u201d from the owner. Vita-only. Fab quotes stay with PCB Fab / Dan as before.
