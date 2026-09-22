# Electrical / mechanical constraints

All numbers below that are not from a meter reading are **assumptions to verify**.

## Power

- Measure VCC at the official card during insert and during a big read.
- Expect a 3.3 V-class rail until proven otherwise. Do not drop a 1.8 V-only MCU on VCC.
- Budget headroom for U1 + microSD inrush. Official cards are tiny; a microSD + MCU is greedier. If the host rail sags, add a cap farm on the island and/or a separate regulated rail \u2014 only after measuring.

## Signal

- 4-bit parallel + BS + SCLK. Half-duplex data.
- Level-shift if U1 I/O \u2260 measured bus voltage.
- Series 22\u201333 \u03a9 placeholders on SCLK/BS/D[3:0] near U1 until edge rates are known.
- Keep SCLK short on the host side of U1. The flex neck is the hostile part.

## Detect

- INS: host pulled up, card shorts to GND (RichDevX). A bridge can hard-tie INS to GND on the host flex, or drive it from U1 if you want removable-microSD semantics.
- Removable microSD on the island is a product feature, not a protocol requirement. Natalie\u2019s Go notes: bigger cards slowed sleep-wake; Vita has its own sleep_session_id rule \u2014 do not assume the same symptom.

## Mechanical (unverified)

Need calipers on a real 1000 and 2000:

- Slot depth, lid interference, nearby shields
- Whether Slim 2000 has less Z for a stiffener island
- Connector pitch and pin-1

1000 vs 2000 **will** differ. One flex may not fit both.

## U1 (do not pick yet)

Candidates only after framing dumps:

- Mid-range MCU with 8+ fast GPIOs and a hardware SD host (RP2040/RP2350 PIO is tempting; 3-DES-CBC-CTS + ECDSA-224 custom curve is not what PIO is for)
- Small FPGA if TPC state machine + crypto offload wants deterministic I/O
- Do not buy a \u201cMemory Stick controller\u201d IC because a seller listed \u201cMS to SD\u201d \u2014 those speak old MS/MS-Pro, not Vita `0x48\u20130x4B`

## ESD / assembly

- Human-insert slot. TVS on the island if the microSD is user-facing.
- Magnet-wire capture harness is not the production interconnect.
- No order / fab without explicit OK.
