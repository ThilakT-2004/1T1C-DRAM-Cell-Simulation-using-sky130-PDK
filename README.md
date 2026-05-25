# 1T1C DRAM Cell — SPICE Simulation (sky130 PDK)

A complete ngspice simulation of a 1-Transistor 1-Capacitor (1T1C) DRAM cell using the open-source sky130 process design kit.

## Circuit Overview

The 1T1C DRAM cell consists of:
- **M1** — NMOS access transistor (`sky130_fd_pr__nfet_01v8`, W=1µm, L=0.15µm)
- **Cs** — Storage capacitor (30 fF)
- **Sense amplifier** — Cross-coupled CMOS latch (XM2–XM5, PMOS W=1.5µm, NMOS W=1µm)

```
        WL
        |
BL ----[M1]---- cs
                |
               [Cs]
                |
               GND
```

---

## Simulation Results

### DC Analysis — Stability / Butterfly Curve
**File:** `dc/1t1c_dram.spice`

Sweeps the `cs` node from 0 → 1.8V with `blb` free to respond. Plots the sense amplifier transfer characteristic to verify two stable states.

![DC Butterfly](screenshots/1t1c_dram.png)

| Parameter | Value |
|---|---|
| VDD | 1.8 V |
| Trip point (Vtrip) | ~0.8 V |
| Stable state '1' | cs = 0V, blb = 1.8V |
| Stable state '0' | cs = 1.8V, blb = 0V |
| PMOS W (sense amp) | 1.5 µm (sized to centre trip point) |

---

### Transient Analysis 1 — Basic Cell (Source Follower)
**File:** `tran/1t1c.spice`

Basic 1T1C cell with WL fixed HIGH and BL pulsing. Verifies M1 passes BL to output (cs) with NMOS threshold drop.

![Basic Tran](screenshots/1t1c.png)

| Parameter | Value |
|---|---|
| WL | Fixed at 1.8V |
| BL | PULSE 0→1.8V, 1µs period |
| V(out) high | ~1.35V (= VDD − Vt) |
| Vt (sky130 NFET) | ~0.45V |

---

### Transient Analysis 2 — Write Operation
**File:** `tran/1t1c_dramW.spice`

WL pulses HIGH while BL = 1.8V (write '1'). Storage capacitor charges through M1.

![Write Tran](screenshots/1t1c_dramW.png)

| Parameter | Value |
|---|---|
| BL | 1.8V (write '1') |
| WL | PULSE, 5ns delay, 20ns ON |
| V(cs) after write | ~1.35V (VDD − Vt threshold drop) |
| Write time | ~3–5 ns |
| V(cs) during hold | Flat — negligible leakage |

> **Note:** V(cs) does not reach full VDD (1.8V) due to the NMOS threshold drop. This is a fundamental characteristic of 1T1C DRAM — the access transistor stops conducting when Vgs = Vt.

---

### Transient Analysis 3 — Read Operation
**File:** `tran/1t1c_dramR.spice`

BL precharged to VDD/2 (0.9V), WL asserts, cs shares charge with BL creating a small ΔV, sense amplifier detects and amplifies to full rail.

![Read Tran](screenshots/1t1c_dramR.png)

| Parameter | Value |
|---|---|
| BL precharge | 0.9V (VDD/2) |
| BLB reference | 0.9V (soft, via 10kΩ) |
| V(cs) initial | 1.35V (stored '1') |
| ΔV on BL | ~60 mV |
| V(blb) after sense | 1.8V (sense amp latches) |
| V(cs) after sense | 1.8V |
| Read type | Destructive (requires write-back) |

---

## Repo Structure

```
1t1c_dram_sim/
├── README.md
├── dc/
│   └── 1t1c_dram.spice       # DC stability / butterfly curve
├── tran/
│   ├── 1t1c.spice             # Basic cell transient (source follower)
│   ├── 1t1c_dramW.spice       # Write operation transient
│   └── 1t1c_dramR.spice       # Read operation transient
└── screenshots/
    ├── 1t1c.png               # Basic cell waveform
    ├── 1t1c_dram.png          # Butterfly curve
    ├── 1t1c_dramW.png         # Write waveform
    └── 1t1c_dramR.png         # Read waveform
```

---

## How to Run

**Prerequisites:**
- ngspice installed
- sky130 PDK installed at `/home/<user>/Desktop/eda_tools/open_pdks/sky130/`

**Run any simulation:**
```bash
ngspice dc/1t1c_dram.spice       # DC stability
ngspice tran/1t1c_dramW.spice    # Write transient
ngspice tran/1t1c_dramR.spice    # Read transient
ngspice tran/1t1c.spice          # Basic cell
```

---

## Key Design Parameters

| Component | Value | Notes |
|---|---|---|
| Technology | sky130 (180nm) | Open-source PDK |
| VDD | 1.8 V | Nominal supply |
| Access NFET W/L | 1µm / 0.15µm | Minimum length |
| Storage cap (Cs) | 30 fF | On-schematic cap |
| BL parasitic (Cbl) | 200 fF | Modelled cap |
| Sense amp PMOS W | 1.5 µm | Sized for trip point ~0.9V |
| Sense amp NMOS W | 1.0 µm | Minimum width |
| ΔV (read swing) | ~60 mV | Cs/(Cs+Cbl) × (Vcs−VDD/2) |
