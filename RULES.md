# Standing rules

These are project constraints, not suggestions.

## Scope

- Vita MEMORY slot only (`SceMsif` / MSIF).
- Not SD2Vita, not the game-card slot (`SceSdif` / gamesd / YAMT / StorageMgr).
- Not PSVSD / 3G-modem USB path.
- Not a redesign of NatalieTheNerd’s PSP Go adapter.
- Do not promise a passive flex “just like Natalie.” Passive wiring cannot speak Vita MEMORY auth.

## IP

- Original layout only.
- Natalie product and install method may be cited as *method inspiration* (desolder proprietary slot → flex → relocated push-push microSD).
- Do not copy her Gerbers, silkscreen, mechanical outline, or product photos into a derivative board.

## Money and fab

- No order, checkout, PCB fab submit, or component buy without the owner’s explicit OK.
- Daily storage stays SD2Vita + StorageMgr until a MEMORY bridge actually works.
- Fab quotes stay with PCB Fab / Dan as before. This repo does not place them.

## Publication

- Prefer public dumps + notes early over a private dead end.
- Capture traces under `captures/` are CC0 so someone else can continue.
- Do not publish Sony firmware blobs, full card NAND images, or keys as if they were ours.

## Firmware / CFW

- A modded Vita helps kernel probes (`SceMsif`, `rmauth_sm.self`, F00D/Cmep calls).
- CFW does **not** by itself unlock Sony card-auth secrets or give you a drop-in fake card.
- “Format this memory card?” is a detection/mount failure mode, not success.

## Hardware safety

- Probe a sacrificial official card or a breakout first.
- Motherboard MEMORY pads are last-resort. Pitch and pin-1 orientation are still unverified on-bench for 1000 vs 2000.
- ESD strap. Flux, 30 AWG magnet wire, strain relief. Do not hot-plug the analyzer ground as an afterthought.
