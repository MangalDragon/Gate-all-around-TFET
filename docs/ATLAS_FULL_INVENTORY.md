# Complete Inventory: Everything Possible in Silvaco ATLAS 2019 for the GaSb/InAs/InGaAs Broken-Gap GAA TFET

This is a working checklist organized by category. Status legend:

- ✅ Done
- 🟡 Partially done
- ⬜ Not yet done (possible to do)
- 🚫 Blocked by ATLAS 2019 capability

---

## CATEGORY 1: STRUCTURE & GEOMETRY VERIFICATION

| # | Item | Status |
|---|------|--------|
| 1.1 | Cylindrical GAA mesh | ✅ |
| 1.2 | Material regions (GaSb/InAs/InGaAs/HfO2/Air) | ✅ |
| 1.3 | Three electrodes (source, drain, gate) | ✅ |
| 1.4 | Gaussian doping profiles | ✅ |
| 1.5 | LDD spacer at drain side | ✅ |
| 1.6 | Mesh quality verification (visual inspection) | ⬜ |
| 1.7 | Doping profile axial cutline | ⬜ |
| 1.8 | Doping profile radial cutline | ⬜ |
| 1.9 | Verify electrode coverage in TonyPlot | ⬜ |

---

## CATEGORY 2: CUTLINES & SPATIAL ANALYSIS

### 2A. Axial cutlines (along the wire, X=0.001)

| # | Cutline | Status |
|---|---------|--------|
| 2.1 | Band edges (E_C, E_V) at equilibrium | ✅ via diag deck |
| 2.2 | Band edges at V_DS = 0.5 V, V_GS = 0 (OFF) | 🟡 saved in .str |
| 2.3 | Band edges at V_DS = 0.5 V, V_GS = 1.5 V (ON) | 🟡 saved in .str |
| 2.4 | Band edges at threshold V_GS = 1.07 V | ⬜ |
| 2.5 | Quasi-Fermi levels (E_Fn, E_Fp) | 🟡 |
| 2.6 | Electric field magnitude | 🟡 |
| 2.7 | BTBT generation rate U.BBT | 🟡 most important for TFET |
| 2.8 | SRH recombination U.SRH | 🟡 |
| 2.9 | Carrier concentrations (n, p) | ⬜ |
| 2.10 | Total current density (J) | 🟡 |
| 2.11 | Electron / hole current density (J_n, J_p) | 🟡 |

### 2B. Radial cutlines (across the wire)

| # | Cutline | Status |
|---|---------|--------|
| 2.12 | Radial band diagram through channel center (Y=0.030) | ⬜ |
| 2.13 | Radial band diagram through tunnel junction (Y=0.020) | ⬜ |
| 2.14 | Radial carrier density at channel center | ⬜ |
| 2.15 | Radial electric field | ⬜ |

### 2C. 2D contour maps

| # | Contour | Status |
|---|---------|--------|
| 2.16 | 2D conduction band map | ⬜ |
| 2.17 | 2D BTBT generation rate map | ⬜ |
| 2.18 | 2D electric field magnitude map | ⬜ |
| 2.19 | 2D current flow vector plot | ⬜ |
| 2.20 | 2D potential map | ⬜ |
| 2.21 | Flowlines | ✅ |

### 2D. Probe-based extraction during bias sweep

| # | Item | Status |
|---|------|--------|
| 2.22 | Probes at fixed locations (logged at every bias step) | ⬜ |
| 2.23 | Probe E_C, E_V on both sides of junction | 🟡 |
| 2.24 | Probe E-field at junction | ⬜ |

---

## CATEGORY 3: DC CHARACTERIZATION

| # | Item | Status |
|---|------|--------|
| 3.1 | Id-Vg at V_DS=0.5V, V_GS = -1.0 to +1.5 V | ✅ |
| 3.2 | Id-Vd at V_GS = 1.1, 1.3, 1.5 V | ✅ |
| 3.3 | Id-Vg at V_DS=0.05V (DIBL) in improved.in | ✅ |
| 3.4 | Id-Vg at V_DS=0.05V on production deck | ⬜ |
| 3.5 | Id-Vg family at V_DS=0.1, 0.2, 0.3, 0.4 V | ⬜ |
| 3.6 | Reverse V_GS sweep (hysteresis) | ⬜ |
| 3.7 | Negative V_DS (drain-side BTBT) | ⬜ |
| 3.8 | Wider Id-Vd range (V_DS up to 1.0 V) | ⬜ |
| 3.9 | Fine V_GS step around V_T (5 mV) | ⬜ |
| 3.10 | Id-Vd at lower V_GS (0.5, 0.7, 0.9 V) | ⬜ |

---

## CATEGORY 4: PARAMETRIC SWEEPS — GEOMETRY

| # | Sweep | Status |
|---|-------|--------|
| 4.1 | Wire radius R (3, 4, 5, 6, 8 nm) | ⬜ |
| 4.2 | Gate length Lg (10, 15, 20, 30 nm) | ⬜ |
| 4.3 | Gate oxide thickness tOX (1.0-3.0 nm) | ⬜ |
| 4.4 | LDD spacer length (0, 2, 5, 8 nm) | ⬜ |
| 4.5 | Source length Ls | ⬜ |
| 4.6 | Source-side underlap | ⬜ |
| 4.7 | Different oxide material (Al2O3 vs HfO2) | ⬜ |
| 4.8 | Asymmetric source/drain radii | ⬜ |

---

## CATEGORY 5: PARAMETRIC SWEEPS — DOPING

| # | Sweep | Status |
|---|-------|--------|
| 5.1 | Source doping N_A (1e19 to 1e20) | ⬜ |
| 5.2 | Drain doping N_D | ⬜ |
| 5.3 | Channel doping N_ch (1e15 to 1e17) | ⬜ |
| 5.4 | Junction sharpness Y.CHAR | ⬜ |
| 5.5 | Source pocket doping (line tunneling) | ⬜ |
| 5.6 | Drain pocket / halo doping | ⬜ |
| 5.7 | Asymmetric Y.CHAR | ⬜ |

---

## CATEGORY 6: PARAMETRIC SWEEPS — MATERIAL / WORKFUNCTION

| # | Sweep | Status |
|---|-------|--------|
| 6.1 | Gate workfunction (4.85-5.40 eV) | 🟡 done at 4.35/4.65/4.80 |
| 6.2 | Source affinity (Type-II sensitivity) | 🟡 in nonlocal.in |
| 6.3 | Source bandgap | ⬜ |
| 6.4 | Channel material variant (InAsSb) | ⬜ |
| 6.5 | Drain InGaAs composition | ⬜ |
| 6.6 | Tunneling masses sensitivity | ⬜ |
| 6.7 | Mobility sensitivity | ⬜ |

---

## CATEGORY 7: PARAMETRIC SWEEPS — OPERATING CONDITIONS

| # | Sweep | Status |
|---|-------|--------|
| 7.1 | Temperature (200-400 K) | ⬜ |
| 7.2 | TAT vs BTBT separation via T | ⬜ |
| 7.3 | V_DD sensitivity (0.3, 0.5, 0.7 V) | ⬜ |

---

## CATEGORY 8: TRAP / INTERFACE PHYSICS

| # | Item | Status |
|---|------|--------|
| 8.1 | Surface recombination velocity | ✅ s.n=s.p=5e2 |
| 8.2 | s.n / s.p sweep | ⬜ |
| 8.3 | Explicit Dit via interface qf/nti/ntd | ⬜ |
| 8.4 | INTDEFECTS block | ⬜ |
| 8.5 | Trap energy level sweep | ⬜ |
| 8.6 | TAT.NONLOCAL vs local TRAP.TUNNEL | 🟡 |
| 8.7 | Acceptor vs donor traps | ⬜ |

---

## CATEGORY 9: AC / SMALL-SIGNAL ANALYSIS

| # | Item | Status |
|---|------|--------|
| 9.1 | C_gg vs V_GS | ⬜ |
| 9.2 | C_gd vs V_DS | ⬜ |
| 9.3 | C_gs vs V_GS | ⬜ |
| 9.4 | Frequency dependence | ⬜ |
| 9.5 | gm (transconductance) | ⬜ |
| 9.6 | gds (output conductance) | ⬜ |
| 9.7 | f_T cutoff frequency | ⬜ |
| 9.8 | f_max | ⬜ |
| 9.9 | gm/I_D efficiency | ⬜ |

---

## CATEGORY 10: QUANTUM CONFINEMENT

| # | Item | Status |
|---|------|--------|
| 10.1 | BQP without BTBT (DD only) | ⬜ |
| 10.2 | Compare V_T with vs without BQP | ⬜ |
| 10.3 | BQP gamma sensitivity | ⬜ |
| 10.4 | BQP alpha sensitivity | ⬜ |
| 10.5 | Density gradient (DGLOG) | ⬜ |
| 10.6 | BQP + BBT.HURKX (local) test | ⬜ |

---

## CATEGORY 11: TUNNELING MODEL COMPARISONS

| # | Item | Status |
|---|------|--------|
| 11.1 | BBT.HURKX (current) | ✅ |
| 11.2 | BBT.STD (Kane) | 🟡 tried |
| 11.3 | BBT.KL (Klaassen) | ⬜ |
| 11.4 | BBT.NONLOCAL with Type-II kludge | 🟡 in nonlocal.in |
| 11.5 | bbt.a sensitivity | ⬜ |
| 11.6 | bbt.b sensitivity | ⬜ |
| 11.7 | Source-only BTBT | ✅ |
| 11.8 | Forward vs reverse BTBT | ⬜ |

---

## CATEGORY 12: MIXED-MODE / CIRCUIT-LEVEL

| # | Item | Status |
|---|------|--------|
| 12.1 | TFET inverter (with PMOS) | ⬜ |
| 12.2 | TFET-TFET inverter | ⬜ |
| 12.3 | NAND/NOR gate | ⬜ |
| 12.4 | SRAM cell | ⬜ |
| 12.5 | Compact model extraction | ⬜ |

---

## CATEGORY 13: EXTRACTIONS & FIGURES OF MERIT

| # | Item | Status |
|---|------|--------|
| 13.1 | V_T (max-slope x-intercept) | ✅ |
| 13.2 | V_T (constant-current method) | ⬜ |
| 13.3 | V_T (linear extrapolation) | ⬜ |
| 13.4 | SS minimum | ✅ |
| 13.5 | SS averaged over 4 decades | ⬜ |
| 13.6 | I_ON | ✅ |
| 13.7 | I_OFF (at V_GS=0) | ⬜ |
| 13.8 | I_ON / I_OFF ratio | ✅ |
| 13.9 | DIBL | ✅ |
| 13.10 | V_DSAT (saturation voltage) | ⬜ |
| 13.11 | Maximum gm | ⬜ |
| 13.12 | SS vs V_GS plot (sub-thermal regions) | ⬜ |

---

## CATEGORY 14: ADVANCED STUDIES

| # | Item | Status |
|---|------|--------|
| 14.1 | Source-pocket TFET variant | ⬜ |
| 14.2 | Vertical/L-shaped TFET geometry | ⬜ |
| 14.3 | Hetero-dielectric gate stack | ⬜ |
| 14.4 | Asymmetric oxide thickness | ⬜ |
| 14.5 | Dual-material gate | ⬜ |
| 14.6 | Counter-doped channel | ⬜ |
| 14.7 | Type-II kludge sensitivity sweep | 🟡 |
| 14.8 | Ambipolar branch detailed study | 🟡 |
| 14.9 | Output resistance r_o | ⬜ |

---

## CATEGORY 15: VISUALIZATION TECHNIQUES

| # | Technique | Status |
|---|-----------|--------|
| 15.1 | Linear Id-Vg plot | ✅ |
| 15.2 | Log-y Id-Vg plot | ✅ |
| 15.3 | Id-Vd family overlay | ✅ |
| 15.4 | 1D cutline | ⬜ |
| 15.5 | 2D contour (surface) plot | ⬜ |
| 15.6 | Vector plot | ⬜ |
| 15.7 | Animation across bias points | ⬜ |
| 15.8 | Multi-curve overlay | ✅ |
| 15.9 | Probe values vs bias | ⬜ |
| 15.10 | ASCII export for external plotting | ⬜ |

---

## CATEGORY 16: STRUCTURALLY BLOCKED IN ATLAS 2019

| # | Item | Why blocked |
|---|------|-------------|
| 16.1 | True non-local BTBT through Type-III | 🚫 WKB needs barrier |
| 16.2 | Multi-band atomistic NEGF | 🚫 NEGF_MS single-band |
| 16.3 | k.p NEGF | 🚫 Not in ATLAS 2019 |
| 16.4 | Phonon-assisted indirect BTBT | 🚫 No Schenk model |
| 16.5 | True sub-60 mV/dec SS | 🚫 Local Hurkx limitation |
| 16.6 | Quantum confinement + non-local BTBT | 🚫 BLOCK divergence |
| 16.7 | Quantitative match to atomistic I_ON | 🚫 Model-class ceiling |
| 16.8 | True 1D quantum capacitance | 🚫 Effective-mass only |

---

## RECOMMENDED PRIORITY ORDER

### Tier 1 — No new simulation needed, just TonyPlot on existing .str files
- 2.1, 2.3, 2.7 — Band edges + BTBT generation rate
- 2.16, 2.17, 2.20 — 2D contour maps

### Tier 2 — One-line deck changes
- 7.1 Temperature sweep
- 6.1 More workfunction points
- 5.4 Y.CHAR sweep
- 13.7, 13.10 Add I_OFF and V_DSAT extracts
- 9.1 C_gg vs V_GS

### Tier 3 — Mesh edits required
- 4.1 R sweep
- 4.2 Lg sweep
- 4.3 tOX sweep

### Tier 4 — Deck restructuring
- 14.1 Source pocket
- 12.1 Mixed-mode inverter
- 10.2 BQP-only V_T comparison

---

## CUTLINE COMMAND REFERENCE

How to generate cutlines from any saved .str file:

### In TonyPlot interactively:
1. `tonyplot gaa_iiiv_hj_tfet_on.str &`
2. Tools → Cutline → choose Y direction
3. Click anywhere at X=0.001 (axial cutline through wire core)
4. New window opens with 1D plot vs Y
5. Right-click on plot → Display → Add Plot → choose variable (E_C, E_V, U BBT, etc.)

### In a deck (automated):
You cannot generate cutlines from inside a deck directly, but you can:
- Save .str at the bias point of interest with `save outfile=at_VG=1.07.str`
- Run tonyplot in batch mode with `tonyplot file.str -script myscript.tcl`
- Or use `probe` statements to get pointwise values logged into the run output

### Useful probe statements for this device:
```
probe name=EC_GaSb_bulk    x=0.001 y=0.010 con.band
probe name=EV_GaSb_bulk    x=0.001 y=0.010 val.band
probe name=EC_InAs_bulk    x=0.001 y=0.030 con.band
probe name=EV_InAs_bulk    x=0.001 y=0.030 val.band
probe name=EC_InAs_junction  x=0.001 y=0.0205 con.band
probe name=EV_GaSb_junction  x=0.001 y=0.0195 val.band
probe name=Efield_junction   x=0.001 y=0.020 e.field
probe name=Ubbt_junction     x=0.001 y=0.020 u.bbt
```

These will print values to the run log at every bias step.

