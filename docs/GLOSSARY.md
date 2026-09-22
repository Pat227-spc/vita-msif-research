# Glossary

| Term | Meaning here |
| --- | --- |
| MEMORY slot | Vita proprietary memory-card connector (9 pins). This project. |
| Game-card slot | Vita game cartridge connector (10 pins). SD2Vita lives here. |
| MSIF | Host interface / driver world for the MEMORY slot (`SceMsif`). |
| SceSdif | SD host / game-card driver world. Not this bus. |
| TPC | Transfer Protocol Command. Memory Stick transaction primitive framed by BS. |
| BS | Bus State. Host line that marks TPC vs data phases. |
| INS | Insert detect. Host input, pulled up; official card grounds it. |
| MagicGate-class | Shorthand for Sony\u2019s card-auth family. Vita MC auth is documented on henkaku wiki as `SceMsif` + `rmauth_sm` + 3-DES-CBC-CTS + ECDSA-224, not necessarily `magicgate.skprx`. |
| CMD56 | SD vendor command. Vita **game cards** use a CMD56 handshake. Different from MC `0x48\u20130x4B`. |
| SD2Vita | Passive-ish game-slot adapter + kernel plugin that remaps a microSD to ux0. Solved storage. |
| StorageMgr / YAMT / gamesd | CFW plugins that mount SD2Vita (and remap official MC / imc0 / uma0). Daily driver until MEMORY bridge works. |
| PSVSD | 3G Vita modem-bay USB/SD adapter. Out of scope. |
| richdapter | RichDevX MEMORY-slot microSD attempt. Died on incomplete auth. \u201cFormat?\u201d \u2260 working. |
| mediaid | Raw partition on official MC; holds `sleep_session_id` among other fields. |
| U1 | Placeholder part designator for the future active bridge. |
