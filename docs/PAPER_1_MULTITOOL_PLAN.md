# Paper 1 (Active Plan): Multi-Tool Analysis with Silvaco ATLAS + ANSYS + Keysight ADS

**Status:** This is the **active** Paper 1 plan, replacing the Sentaurus-comparison plan in `PAPER_1_PLAN.md`.
The Sentaurus plan is preserved as a backup or potential Paper 2 direction.

**Project:** Multi-physics analysis of GaSb/InAs/InGaAs broken-gap GAA tunnel FET
**Author target:** BE final-year student
**Tools:** Silvaco ATLAS 2019 + ANSYS (Icepak/Mechanical) + Keysight ADS
**Target venue:** Microelectronics Journal or Microsystem Technologies (Springer/Elsevier)
**Realistic timeline:** 20 weeks (5 months) part-time

---

## 0. WHY THIS PLAN OVER THE SENTAURUS COMPARISON

The Sentaurus-comparison plan (in `PAPER_1_PLAN.md`) had several risks:
- Sentaurus environment may not be configured on user's machine
- 3+ weeks learning curve before any results
- Convergence risk for broken-gap with full Sentaurus models
- Requires user to be comfortable in Linux terminal

This multi-tool plan is strictly better for the BE timeline because:

| Consideration | Sentaurus comparison | Multi-tool ATLAS+ANSYS+ADS |
|---------------|---------------------|-----------------------------|
| Tool count | 2 (both TCAD) | 3 (TCAD + thermal + circuit) |
| Sentaurus risk | High dependency | Zero — not used |
| Multi-disciplinary appeal | Low | High |
| Industry workflow | No | Yes (real engineering co-simulation) |
| Defendable for BE | Modest | Strong |
| Risk of failure | Tool environment | Just compute time |

The Sentaurus angle is preserved as Paper 2 fallback if user wants a follow-up.



---

## 1. CENTRAL THESIS AND CONTRIBUTIONS

**Working title:** *"Multi-physics analysis of GaSb/InAs/InGaAs broken-gap gate-all-around tunnel FET: device, thermal, and circuit-level evaluation using Silvaco ATLAS, ANSYS, and Keysight ADS"*

**Thesis statement:** A complete engineering characterization of the Type-III broken-gap GAA tunnel FET requires not only device-level TCAD simulation but also thermal analysis (to capture self-heating in the nm-scale wire) and circuit-level evaluation (to project to logic and memory applications); we present an integrated three-tool methodology covering all three domains.

**Defensible contributions:**

1. **Comprehensive device-level analysis** of the GaSb/InAs/InGaAs broken-gap GAA TFET in Silvaco ATLAS 2019, including spatial BTBT generation maps, parametric sensitivity to all major design parameters, AC small-signal characterization, and figures-of-merit at multiple operating points.

2. **Self-heating analysis** of the device using ANSYS thermal simulation, demonstrating that even at low absolute current the small thermal mass of a 5 nm radius wire produces non-negligible local temperature rise that feeds back into device behavior. (Few or no published TFET papers do this with ANSYS specifically.)

3. **Circuit-level performance projection** through compact-model extraction in Keysight ADS, including TFET inverter VTC, ring oscillator characteristics, or SRAM cell static noise margin and read/write delay.

4. **Multi-tool methodology** documenting the device-thermal-circuit co-simulation workflow as a reusable template for future III-V TFET design studies.

**What is NOT claimed:**
- Atomistic-grade absolute I_ON. (Documented as ATLAS structural ceiling.)
- Novel device structure. (The device is well-published.)
- Experimental validation. (No fab in scope.)
- Cross-tool TCAD comparison. (Saved for Paper 2 with Sentaurus.)



---

## 2. TOOL INVENTORY

### Required (must verify access on Day 1)

| Tool | Version | Purpose | Status to verify |
|------|---------|---------|------------------|
| Silvaco ATLAS | 2019 | Device simulation, baseline | Already working — confirmed |
| DeckBuild + TonyPlot | matching ATLAS | Run/edit decks, view output | Already working |
| ANSYS Icepak OR Mechanical Thermal | any modern version | Self-heating analysis | **Verify license + tutorial access** |
| Keysight ADS | any modern version (2019+) | Compact model + circuit simulation | **Verify license + tutorial access** |
| Linux/Windows terminal | any | Run commands | Already have |
| Text editor | gedit, Notepad++, VS Code | Edit deck files | Already have |

### Useful (nice to have)

| Tool | Purpose |
|------|---------|
| Python 3 + matplotlib | Generate publication-quality plots from exported ASCII |
| MATLAB or Origin | Alternative plotting if available |
| Excel / LibreOffice | Compile FoM tables |
| LaTeX (Overleaf or local TeX Live) | Manuscript preparation |
| Reference manager (Mendeley/Zotero) | Bibliography |
| Git + GitHub | Version control (already in use) |

### NOT used in this plan

- Synopsys Sentaurus (saved for Paper 2)
- KWANT or atomistic tools
- Any other commercial TCAD



---

## 3. PHASE A: COMPLETE ATLAS DEVICE CHARACTERIZATION (Weeks 1-6)

This phase finishes everything ATLAS 2019 can do for this device. After this phase, you have a comprehensive device-level dataset; ANSYS and Keysight build on top of it.

### Track A.1 — Spatial visualization (Week 1, ~5 days)

**Goal:** Generate 8-12 publication figures from existing saved .str files. No new simulations needed.

**Existing .str files in your repo (or regenerable from existing decks):**
- `gaa_iiiv_hj_tfet_eq_DD.str` — DD-only equilibrium
- `gaa_iiiv_hj_tfet_dd_vd05.str` — DD-only at V_DS=0.5V
- `gaa_iiiv_hj_tfet_btbt_vd05.str` — full physics at V_DS=0.5V, V_GS=0
- `gaa_iiiv_hj_tfet_on.str` — ON state at V_DS=0.5V, V_GS=1.5V

**Tasks:**

| Task | Time | Output |
|------|------|--------|
| A.1.1 — Axial cutline of E_C, E_V at V_GS=0 (OFF state) | 30 min | Figure 1: OFF-state band diagram |
| A.1.2 — Axial cutline at V_GS=1.5V (ON state) | 30 min | Figure 2: ON-state band diagram |
| A.1.3 — Save .str at V_GS=1.07V (V_T) and cutline | 1 hour | Figure 3: turn-on band diagram |
| A.1.4 — Cutline of E-field magnitude along axis | 30 min | Figure 4: peak field localization |
| A.1.5 — **Cutline of U.BBT (BTBT generation rate)** | 30 min | **Figure 5: where tunneling actually happens** (critical) |
| A.1.6 — Radial cutline of E_C at mid-channel | 30 min | Figure 6: quantum confinement cross-section |
| A.1.7 — 2D contour map of conduction band | 30 min | Figure 7: 2D band visualization |
| A.1.8 — **2D contour map of U.BBT** | 30 min | **Figure 8: spatial BTBT 2D** (critical) |
| A.1.9 — 2D contour map of E-field | 30 min | Figure 9: peak-field zones |
| A.1.10 — 2D contour map of potential | 30 min | Figure 10: gate control verification |
| A.1.11 — 2D vector plot of total current density | 30 min | Figure 11: current flow path |
| A.1.12 — Export curve data to ASCII for Python re-plotting | 1 hour | Reusable raw data |

**TonyPlot procedure for cutlines:**
```
tonyplot gaa_iiiv_hj_tfet_on.str &
# Tools → Cutline → Y direction
# Click at X = 0.001 (axial cutline through wire core)
# Right-click on plot → Display → Add Plot
# Choose: Conduction Band Energy, Valence Band Energy, U BBT, E Field, etc.
# File → Export → save .dat for Python re-plotting
```

**Decision gate end of Week 1:** All 12 figures generated and saved. If TonyPlot has issues, fix before moving on.



### Track A.2 — Parametric sweeps (Weeks 2-3, ~10 days)

**Goal:** Generate parametric trend curves (V_T, SS, I_ON, I_OFF vs design parameter).

**Sweep matrix:**

| Parameter | Values | # sims | Compute time |
|-----------|--------|--------|--------------|
| Temperature T | 200, 250, 300, 350, 400 K | 5 | ~2.5 hours |
| Junction sharpness Y.CHAR | 0.5, 1.0, 2.0, 3.0 nm | 4 | ~2 hours |
| Source doping N_A | 2e19, 5e19, 1e20 cm^-3 | 3 | ~1.5 hours |
| Drain doping N_D | 2e19, 5e19, 1e20 cm^-3 | 3 | ~1.5 hours |
| Gate workfunction | 4.85, 5.00, 5.10, 5.20, 5.30, 5.40 eV | 6 | ~3 hours |
| Channel doping | 1e15, 1e16, 1e17 cm^-3 | 3 | ~1.5 hours |

**Total: 24 simulations, ~12 hours of compute** (overnight in 2-3 batches).

**Implementation:**
For each sweep, create a deck variant by changing the relevant `set` line. Run in batch:
```
deckbuild -ascii -run gaa_T200.in &
deckbuild -ascii -run gaa_T250.in &
...
```

Or use ATLAS's `loop` construct if you've gotten it to work in your version.

**Extraction script (paste into each deck):**
```
extract init infile="idvg_<case>.log"
extract name="V_T"          xintercept(maxslope(curve(abs(v."gate"), abs(i."drain"))))
extract name="SS_min"       1000.0/slope(maxslope(curve(abs(v."gate"), log10(abs(i."drain")))))
extract name="I_ON"         max(abs(i."drain"))
extract name="I_OFF"        min(abs(i."drain"))
extract name="ION_OFF"      $"I_ON"/$"I_OFF"
```

**Output:** 6 parametric trend figures (one per parameter).

### Track A.3 — Geometry sweeps (Weeks 4-5, ~10 days)

**Goal:** Show device performance vs physical dimensions.

**Sweep matrix (each requires a new deck with edited mesh and regions):**

| Parameter | Values | # sims | Setup effort |
|-----------|--------|--------|--------------|
| Wire radius R | 3, 4, 5, 6, 8 nm | 5 | 1 day per geometry |
| Gate length L_g | 10, 15, 20, 30 nm | 4 | 1 day per geometry |
| Oxide thickness t_OX | 1.0, 1.5, 2.0, 2.5 nm | 4 | 1 day per geometry |
| LDD spacer length | 0, 3, 5, 8 nm | 4 | half day per geometry |

**Total: 17 simulations, ~17 days setup, ~17 hours compute.**

**For each radius change:** The X.MESH must extend to the new R+tOX+tGate. The REGION X.MAX values must be updated. The contact placement must follow.

**Output:** 4 scaling-trend figures (V_T, SS, I_ON vs each parameter).

### Track A.4 — AC analysis and small-signal (Week 6, ~5 days)

**Goal:** Extract gm, gds, capacitances, f_T at the operating point.

**ATLAS AC syntax:**
```
# After DC operating point converges (V_DS=0.5, V_GS=1.07 → 1.5)
solve init
solve name=DRAIN vdrain=0.0 vstep=0.05 vfinal=0.5
solve name=GATE  vgate=0.0 vstep=0.025 vfinal=1.5
# Now do AC sweep
solve ac freq=1e6 vstep=0.025 vfinal=1.5 name=GATE
log outfile=cv_gate_sweep.log master
log off

# Extract g_m and capacitances
extract init infile="cv_gate_sweep.log"
extract name="gm_max"       max(slope(curve(abs(v."gate"), abs(i."drain"))))
extract name="Cgg_at_VG_1V" y.val from curve(abs(v."gate"), abs(c."gate"."gate")) where x.val=1.0
extract name="fT_GHz"       $"gm_max" / (2.0 * 3.14159 * $"Cgg_at_VG_1V") / 1e9
```

**Tasks:**
- A.4.1 — Set up AC sweep deck (1 day)
- A.4.2 — Run C_gg, C_gd, C_gs vs V_GS (1 day, 3 simulations)
- A.4.3 — Extract g_m and g_ds from numerical derivative of DC sweeps (1 day)
- A.4.4 — Compute f_T = g_m/(2π C_gg), g_m/I_D efficiency (1 day)
- A.4.5 — Plot small-signal figures (1 day)

**Output:** 4-5 small-signal figures (C-V, g_m, f_T, g_m/I_D).

### Phase A summary

**End of Phase A (Week 6):**
- ~20 figures total from ATLAS work
- ~45 simulations completed
- All FoM (V_T, SS, I_ON, I_OFF, DIBL, g_m, f_T, capacitances) tabulated
- Compact model data ready (Id-Vg-Vd grid for export to Keysight ADS)

**Decision gate:** Have all data needed for Phases B and C? If yes, proceed.



---

## 4. PHASE B: ANSYS THERMAL ANALYSIS (Weeks 7-10)

**Goal:** Demonstrate self-heating in the nm-scale wire and its feedback into device performance.

**Why this is interesting for a TFET:**
The 5 nm radius nanowire has tiny thermal mass. At V_DS = 0.5 V × I_D = 1 nA = 0.5 nW total dissipation, but the *local* dissipation density at the BTBT junction is enormous (current squeezed into ~10 nm³). Steady-state local temperature can rise 10-50 K above ambient. This is real physics that ATLAS doesn't model well in 3D.

### Choice: Icepak vs Mechanical Thermal

**Icepak** is the dedicated thermal analysis tool — better for fluid-cooled or convective scenarios. For a chip with conduction-dominated heat transfer, **Mechanical Thermal** (or Steady-State Thermal in Workbench) is fine and easier to learn.

**Recommendation for BE:** Use ANSYS Mechanical (Steady-State Thermal). Easier setup, faster turnaround.

### Workflow

#### B.1 — Set up ANSYS environment (Week 7, 5 days)

Tasks:
- Verify ANSYS Workbench launches
- Run the standard Steady-State Thermal tutorial (any geometry — just confirm tool works)
- Familiarize with: geometry import, mesh, boundary conditions, solve, post-process
- If this fails → escalate or fall back to ATLAS-only paper

#### B.2 — Build thermal model (Week 8, 5 days)

Tasks:
- B.2.1 — Build a 3D model of the nanowire device in DesignModeler or SpaceClaim
  - Cylindrical InAs core (5 nm radius, 60 nm long)
  - HfO2 cylindrical shell (2 nm thick, 20 nm long, around channel only)
  - Gate metal cylindrical shell (2 nm thick, around oxide)
  - Source/drain contact regions (Aluminum cylinders at ends)
- B.2.2 — Apply material thermal properties:
  - InAs thermal conductivity: 27 W/m·K
  - GaSb thermal conductivity: 32 W/m·K
  - In0.53Ga0.47As thermal conductivity: 5 W/m·K (much lower due to alloy scattering)
  - HfO2 thermal conductivity: 1.0 W/m·K (or 0.5-1.5, literature varies)
  - Aluminum: 237 W/m·K
- B.2.3 — Mesh the geometry (sweep mesh in radial + axial directions)

#### B.3 — Apply heat source from ATLAS (Week 9, 5 days)

Tasks:
- B.3.1 — From ATLAS, extract the spatial dissipation profile P(x,y) = J · E at the V_DS=0.5V, V_GS=1.5V operating point
  - In TonyPlot, plot J_total · E_field (need to do as user-defined function)
  - Or: export J_total and E_field separately, multiply in post-processing
- B.3.2 — Convert to volumetric heat generation (W/m³) and apply as a "Heat Generation" load in ANSYS
- B.3.3 — Apply boundary conditions:
  - Source/drain metal contacts: fixed at 300 K (heat sink)
  - Outer surface of HfO2: convection or fixed temperature (whatever your professor prefers)
  - Surface of nanowire (between contacts): adiabatic or weak coupling

#### B.4 — Solve and extract results (Week 10, 5 days)

Tasks:
- B.4.1 — Solve steady-state thermal
- B.4.2 — Plot T(x,y) — temperature distribution along nanowire axis
- B.4.3 — Identify peak temperature ΔT_peak
- B.4.4 — Plot temperature contour (2D and 3D)
- B.4.5 — Compute thermal resistance R_th = ΔT_peak / P_total

### Optional B.5 — Feedback to ATLAS (Bonus, 1 week extra)

If you want to close the loop:
- Use the temperature distribution from ANSYS as the temperature in ATLAS (`temperature=...` block with spatial profile if supported, or zone-by-zone temperature)
- Re-run ATLAS with temperature-corrected operating point
- Show the I_D shift due to self-heating

This is optional but strongly impressive — true co-simulation.

### Phase B output

**Figures from Phase B:**
- Figure: 3D nanowire thermal model
- Figure: Heat generation profile (from ATLAS, mapped to ANSYS)
- Figure: Temperature distribution T(x,y) along wire
- Figure: Peak temperature vs operating bias
- Figure (optional): Temperature-corrected I_D-V_GS comparing isothermal vs self-heating

**Tables from Phase B:**
- Material thermal properties used
- Thermal resistance breakdown (semiconductor / oxide / metal)
- Peak T at different operating points

---

## 5. PHASE C: KEYSIGHT ADS CIRCUIT-LEVEL (Weeks 11-13)

**Goal:** Project the device-level performance to circuit applications (inverter, ring oscillator, or SRAM).

### Workflow

#### C.1 — Compact model extraction from ATLAS (Week 11, 5 days)

Tasks:
- C.1.1 — Run a DC sweep grid in ATLAS:
  - V_GS: -1.0 to +1.5 V at 25 mV step (101 points)
  - V_DS: 0.0, 0.05, 0.1, 0.2, 0.3, 0.4, 0.5 V (7 points)
  - Total: 707 (V_GS, V_DS) operating points → 7 Id-Vg curves
- C.1.2 — Export Id(V_GS, V_DS) as ASCII table
- C.1.3 — Format as Keysight ADS lookup-table model (`.dscr` file or CITIfile)
  - Two-port model: gate, drain (source as reference)
  - Two-dimensional table I_D(V_GS, V_DS)

**Alternative:** Fit a Verilog-A formula. More work but more flexible. For BE timeline, lookup table is fine.

#### C.2 — TFET inverter design and simulation (Week 12, 5 days)

For an inverter you need both n-TFET and p-TFET. The user's existing device is the n-TFET (electrons tunnel from GaSb VB to InAs CB). 

**For the p-TFET counterpart, two options:**

**Option A — Reverse the doping** (more accurate, more work):
- New device with p+ source (use n-doped GaSb), n+ drain (use p-doped InGaAs)
- Re-run TCAD calibration for this device
- Extract its compact model
- More accurate but doubles the TCAD work

**Option B — Symmetric mirror** (simpler, common in BE projects):
- Assume p-TFET is symmetric to n-TFET with reversed signs
- Use the same I_D model but flip V_GS, V_DS, I_D signs
- Simpler but less rigorous; explicitly note as a limitation

**For BE timeline: Option B.** State the assumption clearly in the methodology.

Tasks:
- C.2.1 — Set up Keysight ADS schematic with the n-TFET pull-down + symmetric p-TFET pull-up
- C.2.2 — DC sweep of input voltage 0 to V_DD
- C.2.3 — Plot output VTC (Voltage Transfer Characteristic)
- C.2.4 — Extract V_IH, V_IL, V_M (mid-point), gain at V_M
- C.2.5 — Compare to a same-V_DD MOSFET inverter (use ADS built-in MOSFET model at 22 nm node)

Output figures:
- Inverter VTC for V_DD = 0.3, 0.5 V
- Comparison TFET vs MOSFET at same V_DD
- Static power dissipation comparison

#### C.3 — Ring oscillator or SRAM cell (Week 13, 5 days)

**Choose ONE based on supervisor preference:**

**Option C.3.A — Ring oscillator:**
- Cascade 5, 7, or 9 inverters in a ring
- Transient simulation, measure oscillation period and frequency
- Compute average power dissipation
- Compute power-delay product (PDP) and energy-delay product (EDP)
- Compare to MOSFET ring oscillator

**Option C.3.B — SRAM cell:**
- 6T SRAM with the TFET inverters
- Static noise margin (SNM) analysis
- Read/write delay measurement
- Hold stability at low V_DD

**Recommendation for BE:** Ring oscillator is simpler, more visual, easier to defend at viva.

Tasks:
- C.3.1 — Set up ADS schematic
- C.3.2 — Transient simulation
- C.3.3 — Extract metrics
- C.3.4 — Plot waveforms and FoM

Output figures:
- Schematic of ring oscillator (or SRAM)
- Output voltage waveform (transient)
- Frequency vs V_DD plot
- Power-delay product comparison

### Phase C output

**Figures from Phase C:**
- Compact model fit quality (TCAD points vs ADS lookup output)
- Inverter VTC (TFET vs MOSFET)
- Ring oscillator waveform
- Frequency vs V_DD
- Power-delay product comparison

**Tables from Phase C:**
- Compact model parameters (or LUT statistics)
- Inverter FoM (gain, V_M, switching threshold)
- Ring oscillator FoM (frequency, power, PDP)



---

## 6. PHASE D: PAPER WRITING (Weeks 14-17)

### Section-by-section outline

#### Abstract (last to write, 200-250 words)
- 1 sentence: motivation (broken-gap TFET importance)
- 2 sentences: device under study
- 2 sentences: three-tool methodology
- 3 sentences: key findings (device-level + thermal + circuit-level)
- 1 sentence: implications

#### 1. Introduction (1.5-2 pages)
- Boltzmann tyranny, TFET, broken-gap, our specific device
- Need for multi-physics characterization (thermal + circuit)
- Existing literature: device-level only, no thermal or circuit-level studies
- Contributions of this work (4-5 bullets)

#### 2. Device Structure and Physics (1.5 pages)
- Device dimensions, materials, doping
- Anderson-rule band alignment (with calculation showing +114 meV)
- Why GAA cylindrical, why InGaAs drain
- Cite Carrillo-Nuñez 2017 atomistic, Memisevic 2017 experimental

#### 3. Methodology (2-2.5 pages)

**3.1 Device-level TCAD (Silvaco ATLAS 2019)**
- Mesh, regions, materials, doping
- Model selection: BBT.HURKX with calibrated bbt.a, bbt.b
- Two-stage solve sequence
- Why local Hurkx (with footnote on BBT.NONLOCAL/NEGF_MS limitations)

**3.2 Thermal analysis (ANSYS Mechanical Thermal)**
- Geometry import workflow
- Material thermal properties
- Heat source from ATLAS dissipation
- Boundary conditions
- Mesh convergence

**3.3 Circuit-level (Keysight ADS)**
- Compact model extraction approach
- Inverter and ring oscillator topology
- Reference MOSFET model used for comparison

#### 4. Results (3-4 pages)

**4.1 Device-level characteristics**
- Equilibrium and biased band diagrams (Figs 1-3)
- Spatial BTBT generation map (Fig 5)
- Id-Vg, Id-Vd at multiple V_DS (Fig 4)
- Parametric sensitivity: V_T, SS, I_ON vs T, doping, workfunction, junction sharpness (Fig 6-9)
- AC and small-signal: C-V, g_m, f_T (Figs 10-12)

**4.2 Thermal analysis**
- Heat dissipation profile (Fig 13)
- Temperature distribution along wire (Fig 14)
- Peak temperature rise vs operating bias (Fig 15)
- Thermal resistance components (Table)

**4.3 Circuit-level results**
- Compact model fit verification (Fig 16)
- Inverter VTC: TFET vs MOSFET at same V_DD (Fig 17)
- Ring oscillator output and frequency vs V_DD (Fig 18)
- Power-delay product comparison (Fig 19)

#### 5. Discussion (1.5 pages)
- Multi-physics insights: how thermal feeds back into device, how device feeds into circuit
- Comparison with published atomistic predictions and experimental data
- Limitations: effective-mass single-band, isothermal device-level (corrected by Phase B), look-up table circuit model
- Practical implications for low-power TFET design

#### 6. Conclusion (0.5-1 page)
- Summary of multi-tool methodology
- Key numerical results
- Future work: cross-tool TCAD comparison (Sentaurus), atomistic validation, fab integration

#### References (1-2 pages, ~25-30 references)

### Tasks for Phase D

| Task | Time |
|------|------|
| D.1 — Outline expansion (each section into bullets) | 2 days |
| D.2 — Section-by-section drafting (1 day per section × 6) | 6 days |
| D.3 — Figure integration and caption writing | 3 days |
| D.4 — Reference cleanup and bibliography | 2 days |
| D.5 — Internal proofreading | 2 days |
| D.6 — Buffer | 5 days |

**Total: ~20 days, 4 weeks.**



---

## 7. PHASE E: SUBMISSION (Weeks 18-20)

### Venue selection (multi-tool methodology paper)

**Primary target:** Microelectronics Journal (Elsevier)
- IF ~2.4
- Multi-physics methodology fits well
- 8-16 week typical review cycle

**Secondary:** Microsystem Technologies (Springer)
- IF ~2.0
- Strong fit for multi-tool MEMS/nano-device papers
- Welcomes BE/MS-level contributions

**Tertiary:** Silicon (Springer)
- IF ~2.6
- Broader semiconductor scope
- Decent acceptance rate

**Conference fallback:** IEEE INDICON, IEEE iSES, IEEE VLSID
- Conference proceedings (less prestigious than journal)
- Faster turnaround (~3 months)
- Can lead to journal extension later

**Avoid:**
- IEEE TED, EDL — they want device physics novelty, not multi-tool methodology
- Nature Sci. Reports — too broad-audience for this scope

### Tasks for Phase E

| Task | Time |
|------|------|
| E.1 — Supervisor review | 5 days |
| E.2 — Revise based on feedback | 3 days |
| E.3 — Format for journal template (Microelectronics Journal LaTeX) | 1 day |
| E.4 — Final checks (figures resolution, captions, references) | 1 day |
| E.5 — Submit | 1 day |

---

## 8. SPECIFIC FIRST ACTIONS (your next steps)

### This week (Day 1 onwards)

**Day 1: Tool verification**

Open three terminals/instances and verify:

```
# 1. ATLAS (you already know this works)
cd ~/your_atlas_workspace
deckbuild &

# 2. ANSYS Workbench
ansysWB &
# Or: navigate to Start Menu → ANSYS xx.x → Workbench
# Confirm Steady-State Thermal module is available

# 3. Keysight ADS
ads &
# Confirm it launches and you can create a new schematic
```

**Day 2: Tutorial run (each tool)**

- ATLAS: any tutorial example you've already done
- ANSYS Mechanical: run the "Steady-State Thermal" tutorial (heat conduction in a simple block)
- Keysight ADS: run the basic transistor amplifier tutorial

**Day 3: Decision**

- (a) All three tools work → start Phase A
- (b) ANSYS or ADS doesn't work → contact IT/supervisor; evaluate fallback (ATLAS-only paper)

### Week 1 onwards: Phase A.1 (visualization)

Start with Track A.1 because it's fastest, lowest risk, and produces immediate paper-ready figures from data you already have. By end of Week 1 you should have 10-12 figures in hand.

---

## 9. RISK REGISTER

### R1 — ANSYS license unavailable
**Mitigation:** Verify Day 1. If unavailable, Phase B can be replaced by:
- Simpler analytical thermal estimate (not as impressive)
- Different bonus (e.g., the existing AC small-signal section can be expanded)

### R2 — Keysight ADS license unavailable
**Mitigation:** Verify Day 1. Fallback: use ATLAS mixed-mode (limited but works), or use free LTspice with hand-extracted compact model.

### R3 — Compact model extraction harder than expected
**Mitigation:** Start with simple lookup-table (LUT) instead of Verilog-A. ADS supports LUT models natively.

### R4 — ANSYS thermal coupling diverges
**Mitigation:** Use simpler 1D thermal model (radial heat conduction analytical formula). Loses Phase B.5 feedback but keeps the basic thermal characterization.

### R5 — BE coursework consumes time
**Mitigation:** Plan for 10-15 hours/week, not 40. Extend to 6 months if needed. Phase A is the floor — even if Phases B and C drop, Phase A alone can publish in a lower-tier venue.

### R6 — Reviewer rejects "just multi-tool engineering, no novel physics"
**Mitigation:** Reframe contribution as "engineering methodology for III-V TFET integration." Cite this is a documented gap. The thermal feedback (if Phase B.5 is done) is genuinely novel because few/no published TFET papers couple ATLAS dissipation to ANSYS thermal.

### R7 — Supervisor wants different angle
**Mitigation:** Discuss this plan in Week 1. Adapt structure (e.g., focus on circuit-level as the headline, not thermal).

### R8 — Compute time exceeds available resources
**Mitigation:** Most ATLAS sims are <1 hour. ANSYS thermal is typically <30 min. ADS circuit sims are seconds to minutes. Total compute is well under 100 hours; manageable on a workstation.

---

## 10. DECISION GATES (at each phase boundary)

After each phase, answer YES/NO to proceed:

**End of Day 3 (tool verification):**
- ATLAS works? (Y/N)
- ANSYS thermal works? (Y/N)
- Keysight ADS works? (Y/N)
- If all Y → proceed
- If any N → fallback plan (skip that phase or fall back to ATLAS-only paper)

**End of Week 1 (Phase A.1):**
- 10-12 figures generated from existing .str files? (Y/N)
- Cutlines and contour maps look physically reasonable? (Y/N)
- If Y → proceed to A.2

**End of Week 6 (Phase A complete):**
- All 20+ ATLAS figures done?
- All FoM tabulated?
- Compact model data ready (Id-Vg-Vd grid exported)?
- If Y → proceed to Phase B

**End of Week 10 (Phase B complete):**
- Thermal model converged?
- Peak temperature extracted?
- If Y → proceed to Phase C

**End of Week 13 (Phase C complete):**
- Compact model in ADS verified against TCAD?
- Inverter VTC and ring oscillator simulated?
- Comparison with MOSFET reference done?
- If Y → proceed to Phase D

**End of Week 17 (Phase D complete):**
- 8-12 page draft complete with all figures?
- All references compiled?
- Internal proofread done?
- If Y → submit

---

## 11. FOR THE NEXT AI / NEXT SESSION

### Read-this-first list

When the next AI session begins, read in this order:

1. **`docs/PAPER_1_MULTITOOL_PLAN.md`** (this document) — the active execution plan
2. `docs/PAPER_1_PLAN.md` — backup plan (Sentaurus comparison) and original status
3. `docs/HANDOFF.md` — executive summary
4. `docs/MODELS.md` — per-model rationale and ATLAS failure analysis
5. `docs/COMPLETE_GUIDE.md` — Sentaurus migration tutorial (NOT used in this plan, but contains useful Linux/TCAD basics)
6. `docs/THEORY_AND_DESIGN.md` — graduate-level physics

### Critical context

- **The user's I_ON of 2.34 pA in ATLAS is NOT a bug.** It is the documented ceiling of effective-mass commercial TCAD for Type-III broken-gap. The paper builds on this finding.
- **This plan does NOT use Sentaurus.** That's intentional. Sentaurus comparison is a future direction.
- **The user has access to ANSYS and Keysight ADS** but tool versions and licenses must be verified Day 1.
- **The user is BE final-year**, working part-time (10-15 hours/week).
- **Realistic timeline is 5-6 months**, not 3.

### What the next AI should do

1. Check user's progress against this phase plan
2. Provide concrete deck modifications (not theory)
3. Help debug specific simulator issues
4. Help format figures for paper
5. Help review draft sections

### What the next AI should NOT do

1. Pivot back to Sentaurus comparison (already decided against)
2. Suggest a new device (already decided against)
3. Push for atomistic tools (out of scope for BE)
4. Aim for top-tier venues (BE-level scope)

---

## 12. DOCUMENT STATUS

- Version: 1.0 (multi-tool plan)
- Status: ACTIVE plan. Replaces Sentaurus-comparison plan in `PAPER_1_PLAN.md`.
- Sentaurus plan: kept as backup/Paper 2 in `PAPER_1_PLAN.md`.
- Last update: at handoff
