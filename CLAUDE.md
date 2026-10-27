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
