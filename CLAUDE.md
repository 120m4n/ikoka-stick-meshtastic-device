# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A KiCad hardware project (no firmware, no build/test commands): a Meshtastic LoRa node PCB that takes a socketed Seeed XIAO module (nRF52840 or ESP32-S3), an EBYTE E22 LoRa module (400/900 MHz, 22/30/33 dBm variants), an optional SSD1306 OLED, and an optional 21700 or LiPo battery. The README holds the GPIO map, parts options and Meshtastic firmware compatibility notes. Update it when pin assignments or part options change.

## Layout

- Single flat design: `ikoka-stick-meshtastic-device.kicad_sch` (one sheet) and `.kicad_pcb`, plus `.kicad_pro`.
- `libraries/` holds project symbol/footprint libs (`Battery 21700`, `E22-900M30S`) and the `OPL_Kicad_Library` git submodule (Seeed). Run `git submodule update --init` after cloning. `sym-lib-table` and `fp-lib-table` reference these.
- `3dmodels/` holds STEP models referenced by footprints.
- `jlcpcb/` holds `rotations.csv` (per-part rotation corrections for JLCPCB PCBA pick-and-place) and `project.db` (JLCPCB KiCad plugin part database). Generated output (`jlcpcb/gerber`, `jlcpcb/production_files`) and exported BOM/netlist files (`*.csv`, `*.xml`, `*.net`) are gitignored. `rotations.csv` is the exception that is tracked.
- Part orientation changes for assembly (e.g. "Rotate L1 to be upright on JLCPCB PCBA PNP") are handled via `rotations.csv`, not by rotating the footprint.

## Working with the files

- KiCad files are S-expression text. Prefer editing through KiCad or the Konnect MCP tools (see the `konnect` skill) over hand-editing, to avoid corrupting them.
- Commit style: conventional prefixes (`feat:`, `chore:`), and issue refs such as `(fixes #18)`.
- CI (`.github/workflows/pcb_image.yaml`) renders the PCB with `linalinn/kicad-render` on push to `main` and publishes top/bottom/GIF images to GitHub Pages, which the README embeds.

## JP1 (E22P EN jumper, v3) — reference for firmware flashing (MeshCore/Meshtastic)

JP1 is a 3-pad solder jumper that picks what drives pin 6 of the LoRa module (U2):

```
 pad 1        pad 2 (center)      pad 3
 +3V3  ═════  U2 pin 6            XIAO D5 (pin 6)
       factory
        trace
```

- Pad 1: 3.3 V.
- Pad 2 (center): U2 pin 6 (`EN` on the E22P, `RXEN` on the E22).
- Pad 3: XIAO D5 (pin 6), the pin the firmware uses as `SX126X_RXEN` (nRF52840 P0.05, ESP32-S3 GPIO6).

The footprint is "bridged 1–2": the board ships with a thin copper trace joining pads 1 and 2. Pad 3 is left open.

**Default, 1–2 (for the E22P)**
- The E22P `EN` pin is tied to 3.3 V, so the module is always enabled.
- No rework and no firmware change needed.
- The firmware still toggles D5 as if it were `RXEN`, but D5 only reaches pad 3, which is open, so it has no effect.

**2–3 (cut the 1–2 trace with a hobby knife, then put a solder blob across pads 2 and 3)**
- U2 pin 6 now follows D5.
- **Original E22: mandatory.** The E22 needs `RXEN` toggled (high in RX, low in TX). Tied to 3.3 V, the RX path stays on while transmitting: poor performance and possible damage. With 2–3 the board is electrically the same as v2.
- **E22P with software power-down (e.g. to save battery in sleep): careful.** Stock firmware drives `RXEN` low during TX, which on the E22P **disables the module exactly when it must transmit**. Only use this with firmware that treats D5 as an enable (held high in TX and RX), such as the washtastic variant tahnok links to (https://github.com/yellowcooln/MeshCore-washtastic-e22p/commit/3ebd5f18f94eb0c87fe2e88f7eaa14b53440a2b4). With stock firmware and an E22P, keep 1–2.

**Cautions**
- Never bridge all 3 pads: that shorts 3.3 V to D5, and when the XIAO drives D5 low the MCU pin can be damaged.
- After cutting, check with a multimeter in continuity mode that pads 1 and 2 are no longer connected before bridging 2–3.
- JP1 sits under the XIAO; pull the XIAO from its sockets to change it.

| Module | JP1 | Firmware |
|---|---|---|
| E22P | 1–2 (factory default) | Stock Meshtastic/MeshCore |
| E22 | 2–3 (cut and bridge) | Stock Meshtastic/MeshCore |
| E22P with software power-down | 2–3 | Firmware that treats D5 as an enable |

When configuring/flashing firmware for this board: with an E22P on 1–2, the `SX126X_RXEN` (D5) define is harmless and can stay or be set to unused; U2 pin 7 (`T/R CTRL` on the E22P, `TXEN` on the E22) is wired to the radio's DIO2, so check that the firmware variant uses DIO2 as the RF switch control (`SX126X_DIO2_AS_RF_SWITCH`).
