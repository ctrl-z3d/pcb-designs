# 5V Linear Power Supply (AC-DC, L7805-based)

I designed a small, DRC-clean KiCad PCB that converts low-voltage AC (from a transformer) into a regulated 5V DC output, using a full-wave bridge rectifier and an L7805 linear regulator.


## Overview

![3D View](3D%20View.png)

This board takes AC input from a Source, rectifies it, smooths it, and regulates it down to a clean 5V DC rail suitable for powering microcontrollers, sensors, relays, or other low-voltage electronics.

*Input:* 9–12V AC
*Output:* 5V DC, up to 1.5A
*Topology:* Bridge rectifier → smoothing capacitor → linear regulator → output filtering

AC IN → [Bridge Rectifier D1] → [C3 smoothing] → [L7805] → [C1/C2 decoupling] → DC OUT

## Schematic

| Ref | Part | Value / Part Number | Purpose |
|---|---|---|---|
| J1 | Screw terminal (2-pin) | General 2 pin | AC input |
| D1 | Bridge rectifier | KBP206 | Full-wave AC→DC rectification |
| C3 | Radial electrolytic | 1000µF | Bulk smoothing / ripple filtering |
| U1 | Linear regulator | L7805 (TO-220) | Regulates to 5V DC |
| C1 | Film/ceramic cap | 0.33µF | Regulator input decoupling (per datasheet) |
| C2 | Ceramic cap | 0.1µF | Regulator output decoupling (per datasheet) |
| J2 | Screw terminal (2-pin) | General 2 pin | Regulated 5V DC output |

Signal flow is a standard capacitor-input linear supply:

AC IN → [Bridge Rectifier D1] → [C3 smoothing] → [L7805] → [C1/C2 decoupling] → DC OUT

## Design notes

### KBP206 3D Model
*IMPORTANT NOTE: The KBP206 3D file is located within the footprints and schematics, kindly add it via KiCAD to view the 3D view of the Rectifier.*

### Why 9–12V AC input

The L7805 needs a minimum ~2V headroom above 5V to regulate correctly, even accounting for ripple sag between rectifier charging pulses. After accounting for bridge diode drops (~1.4V) and ripple, an AC input in the 9–12V RMS range lands the DC rail comfortably in the L7805's safe regulation window without excessive voltage headroom — excess headroom is wasted entirely as heat (see below), so this range was chosen deliberately rather than just "whatever transformer was on hand."

### Thermal design

Linear regulators dissipate power as heat according to:

P = (Vin − Vout) × Iout

At the top of the design envelope (12V AC → ~14V DC, 1.5A load), this comes out to roughly 13W dissipated in the TO-220 package — well beyond what the bare package can shed to ambient air. The board reserves a silkscreen-outlined footprint and mounting hole pattern (M3, 3.2mm clearance) for a bolt-on aluminum heatsink; a bare TO-220 or a small clip-on heatsink is *not sufficient* at this current level. See [Bill of Materials](#bill-of-materials) for the heatsink spec used.

### PCB layout

- *2-layer board.* All power routing (rectifier → smoothing cap → regulator → output) is on the bottom copper layer (B.Cu) at 1mm trace width, sized for the ripple/peak charging current through the bridge rectifier and smoothing cap, not just average load current.
- *Ground poured solid on both layers* (F.Cu and B.Cu), stitched together with vias, and set to *solid pad connection* (not thermal relief) rather than the KiCad default. This was a deliberate choice: thermal reliefs exist to make hand-soldering easier by limiting heat sinking into the pour, which is the opposite of what's wanted at the regulator's ground/tab connection, where good copper contact helps both current return and heat spreading.
- *Decoupling caps (C1, C2)* placed close to the regulator's IN/OUT pins to minimize trace inductance and keep them effective against fast transients, per the L7805 datasheet's recommended values.
- *Mounting holes* (M3, standard 3.2mm close-fit clearance) added at two points for mechanical stability, since the screw terminals will see repeated mechanical stress from wiring.

## Bill of Materials

| Qty | Part | Notes |
|---|---|---|
| 1 | KBP206 bridge rectifier | |
| 1 | L7805 (TO-220) | |
| 1 | 1000µF electrolytic capacitor | Radial, D12.5mm |
| 1 | 0.33µF capacitor | Per L7805 datasheet |
| 1 | 0.1µF capacitor | Per L7805 datasheet |
| 2 | 2-pin screw terminal block | 5.08mm pitch |
| 1 | TO-220 heatsink | Sized for ~9°C/W or better at full 1.5A load — see [Thermal design](#thermal-design) |
| 1 | M3 screw + hardware | Heatsink and/or board mounting |

## Status

- ✅ Schematic complete
- ✅ PCB layout routed
- ✅ DRC clean
- ⏳ Not yet fabricated / bench-tested
