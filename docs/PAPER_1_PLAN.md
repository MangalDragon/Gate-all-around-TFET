# Paper 1: Comprehensive Plan and Handoff Document

**Project:** Cross-tool TCAD comparison of GaSb/InAs/InGaAs broken-gap GAA tunnel FET
**Author target:** BE final-year student, single-author with supervisor
**Tools:** Silvaco ATLAS 2019 (have), Synopsys Sentaurus N-2017 (have)
**Target venue:** Microelectronics Journal (Elsevier) or equivalent mid-tier
**Realistic timeline:** 4-5 months (20 weeks) part-time alongside BE coursework

---

## 0. STATUS AT HANDOFF (what's done, what isn't)

### Done — ATLAS side (2 sessions of work, in repo)

- [x] Cylindrical GAA mesh, 5 nm radius, 20 nm gate length (`gaa_iiiv_hj_tfet.in`)
- [x] Three-region structure: GaSb p+ source, InAs n- channel, In0.53Ga0.47As n+ drain
- [x] HfO2 gate dielectric, 2 nm thick
- [x] Gaussian doping profiles with Y.CHAR=0.001 (1 nm sharpness)
- [x] LDD spacer at drain side (5 nm undoped InGaAs region)
- [x] Calibrated material parameters from Vurgaftman 2001 + Avci 2015 + Bahuguna 2017
- [x] Two-stage solve sequence (DD eq → drain ramp → BTBT turn-on → gate sweep)
- [x] Verified production result: V_T = 1.07 V, SS = 103 mV/dec, I_ON = 2.34 pA, I_OFF = 8.5e-17 A
- [x] Documented three failure modes:
  - BBT.NONLOCAL aborts (WKB Code 2 — needs forbidden barrier)
  - NEGF_MS returns T(E)=0 (single-band Hamiltonian, no CB-VB coupling)
  - BBT.HURKX converges but underpredicts I_ON by 3-6 orders
- [x] Verified +104 meV broken-gap junction overlap from band-edge cutline
  (.dat extraction from saved .str confirms the Type-III character matches Anderson-rule
  prediction within 10 meV)
- [x] Six deck variants saved in `simulations/`:
  - `gaa_iiiv_hj_tfet.in` (production, local Hurkx)
  - `gaa_iiiv_hj_tfet_v9_fixed.in` (BBT.NONLOCAL + BQP attempt)
  - `gaa_iiiv_hj_tfet_nonlocal.in` (Type-II kludge option)
  - `gaa_iiiv_hj_tfet_negf.in` (NEGF_MS attempt)
  - `gaa_iiiv_hj_tfet_negf_diag.in` (band-edge probe deck)
  - `gaa_iiiv_hj_tfet_improved.in` (workfunction sweep + DIBL)
- [x] Full documentation in `docs/`:
  - `MODELS.md` (per-model rationale + NEGF closing analysis)
  - `REVIEW.md` (original v9 deck audit, six bugs identified)
  - `HANDOFF.md` (executive summary)
  - `THEORY_AND_DESIGN.md` (graduate-level physics)
  - `COMPLETE_GUIDE.md` (Sentaurus migration tutorial)
  - `ATLAS_FULL_INVENTORY.md` (capability checklist)
  - `STATUS_AND_SENTAURUS.md` (project status)

### NOT done — Sentaurus side (this is the new work)

- [ ] Sentaurus environment setup on Red Hat
- [ ] sde geometry build (2D axisymmetric cylindrical)
- [ ] sdevice command file with NonlocalPath BTBT
- [ ] sdevice.par material parameters file
- [ ] First clean DD equilibrium solve
- [ ] First Id-Vg with NonlocalPath BTBT
- [ ] Calibration to match ATLAS Hurkx parametric trends
- [ ] Parametric sweeps (workfunction, doping, temperature, geometry)
- [ ] Cross-tool comparison plots and tables
- [ ] Paper draft

### Missing infrastructure
- [ ] Confirmation that Sentaurus N-2017 is installed and licensed on user's Red Hat machine
- [ ] Confirmation user has terminal/Linux basics (covered in COMPLETE_GUIDE.md Chapter 0)

---

## 1. PAPER 1 — CENTRAL THESIS

**Title (working):** *"Comparative TCAD analysis of Type-III broken-gap GaSb/InAs/InGaAs gate-all-around tunnel FET in Silvaco ATLAS 2019 and Synopsys Sentaurus N-2017"*

**Thesis statement (one sentence):** Two industry-standard commercial TCAD tools, applied to the same Type-III broken-gap GAA TFET design, yield distinct convergence behaviors and quantitative results that are dictated by the underlying band-to-band tunneling model architecture; we document the calibration procedures, parametric sensitivities, and physical interpretation of these tool-dependent and tool-independent findings.

**Contributions claimed (must be defensible):**

1. First systematic side-by-side TCAD comparison of ATLAS 2019 and Sentaurus N-2017 on a Type-III broken-gap GAA TFET. *(Defensibility: search Google Scholar for the specific tool pair on this device; if no hit, the claim holds.)*
2. Documented model-by-model failure analysis of ATLAS for broken-gap heterojunction (WKB-based BBT.NONLOCAL aborts; effective-mass single-band NEGF_MS returns zero transmission; local Hurkx converges but underpredicts).
3. Demonstrated convergence path in Sentaurus using dynamic NonlocalPath BTBT.
4. Cross-tool agreement on V_T and SS trends across design parameters (workfunction, junction sharpness, temperature, geometry) despite divergent absolute I_ON values.
5. Calibration recipe for both tools against the atomistic NEGF reference of Carrillo-Nuñez et al. 2017.

**What is NOT claimed:**
- Atomistic-grade quantitative I_ON. *(That's outside the scope of effective-mass commercial TCAD.)*
- Novel device structure. *(The device is well-published; we're benchmarking tools against it.)*
- Experimental validation. *(No fab in scope.)*

---

## 2. TOOL INVENTORY (everything you need)

### Required (must have)

| Tool | Version | Purpose | License source |
|------|---------|---------|----------------|
| Silvaco ATLAS | 2019 (5.28.1.R or similar) | First simulator, baseline | Already have |
| DeckBuild | matching ATLAS | Run/edit ATLAS decks | Bundled with ATLAS |
| TonyPlot | matching ATLAS | View ATLAS output (.log, .str) | Bundled with ATLAS |
| Synopsys Sentaurus | N-2017.09 | Second simulator, primary novel work | Already have, need to confirm operational |
| sde (Sentaurus Structure Editor) | N-2017.09 | Build geometry/mesh in Sentaurus | Bundled |
| sdevice | N-2017.09 | Run physics simulation | Bundled |
| svisual | N-2017.09 | View Sentaurus output (.tdr) | Bundled |
| inspect | N-2017.09 | Plot Sentaurus I-V curves (.plt) | Bundled |
| swb (Sentaurus Workbench) | N-2017.09 | GUI project manager | Bundled, optional but helpful |
| Linux terminal | Red Hat (any modern version) | Run commands | Already have |
| gedit or similar text editor | any | Edit command files | Already have |

### Useful (nice to have)

| Tool | Purpose |
|------|---------|
| Python 3 with matplotlib | Generate publication-quality comparison plots from exported ASCII data |
| Excel or LibreOffice Calc | Compile tables of extracted FoM |
| Reference manager (Mendeley, Zotero) | Manage bibliography for paper writing |
| LaTeX (TeX Live or Overleaf) | Manuscript preparation |
| Git + GitHub | Version control of all files (already in use) |

### NOT needed for this paper

- ANSYS (no thermal/EM analysis required for paper 1)
- Keysight ADS (no circuit simulation required for paper 1)
- KWANT or any atomistic tool (out of scope)
- Victory Atomistic (out of scope)

---

## 3. PHASE-BY-PHASE EXECUTION PLAN

### PHASE 0 — Environment verification (Week 1, 5 days)

**Goal:** Confirm Sentaurus N-2017 actually runs on the user's Red Hat machine before any device work.

**Day 1: Sentaurus PATH and license**
```bash
# In a Red Hat terminal:
which sde sdevice svisual inspect swb
# All four should print full paths. If any says "no command", contact IT.

sdevice -h | head -10
# Should print version info. If license error, contact IT.

echo $STROOT
echo $LM_LICENSE_FILE
# These should be set. If empty, environment is not loaded.
```

**Day 2-3: Run a tutorial example**
Find an example in `$STROOT/tcad/current/lib/sdevice` or similar. Run it end-to-end.
Expected outcome: tutorial completes, produces .tdr and .plt files, svisual opens the structure.
If this fails, **stop and resolve before continuing**.

**Day 4: Set up project directory**
```bash
mkdir -p ~/sentaurus_projects/gaa_tfet_paper1
cd ~/sentaurus_projects/gaa_tfet_paper1
```

**Day 5: Decision gate.** Either:
- (a) Sentaurus works → proceed to Phase 1
- (b) Sentaurus doesn't work → escalate to supervisor; backup plan is ATLAS-only methodology paper

### PHASE 1 — Sentaurus device build (Weeks 2-3, ~10 days)

**Goal:** Build the same physical device in Sentaurus that we have in ATLAS, with no physics yet.

**Reference document:** `docs/COMPLETE_GUIDE.md` Chapter 3 has the exact `sde_dvs.cmd` template.

**Tasks:**

1. **Day 1:** Copy the `sde_dvs.cmd` from `COMPLETE_GUIDE.md` into the project folder.
   Customize dimensions to match ATLAS deck:
   - R = 0.005 (5 nm radius)
   - tOX = 0.002 (2 nm HfO2)
   - Source GaSb = 17 nm long (Y from 0.003 to 0.020 to match ATLAS Y.MIN=0.003 Y.MAX=0.020)
   - Channel InAs = 20 nm long
   - Drain InGaAs = 17 nm long with 5 nm LDD spacer
   
2. **Day 2:** Run `sde -e -l sde_dvs.cmd`. Debug syntax errors (parenthesis matching, material name spelling).

3. **Day 3:** Open the resulting `gaa_tfet_msh.tdr` in svisual. Verify visually:
   - 4 colored regions (GaSb, InAs, InGaAs, HfO2) in correct positions
   - Mesh density: fine at junctions, coarser elsewhere
   - Three contacts visible (source, drain, gate)
   - Doping concentrations correct in each region

4. **Day 4-5:** Build `sdevice.par` material parameters file.
   Reference: `COMPLETE_GUIDE.md` Chapter 5.
   For each material, specify:
   - Bandgap (Chi0 = electron affinity, Eg0 = bandgap)
   - Epsilon (relative permittivity)
   - Effective masses (eDOSMass, hDOSMass)
   - Band2BandTunneling parameters
   
   **Critical:** All values must match ATLAS material cards. Use Vurgaftman 2001 values:
   - GaSb: chi=4.06, Eg=0.726, eps=15.7, mc=0.039, mhh=0.40, mlh=0.05
   - InAs: chi=4.90, Eg=0.354, eps=15.15, mc=0.023, mhh=0.026, mlh=0.025
   - InGaAs: chi=4.50, Eg=0.74, eps=13.9, mc=0.041, mhh=0.46, mlh=0.05
   - HfO2: chi=2.05, Eg=5.7, eps=22.0

5. **Day 6:** First Sentaurus simulation: pure Poisson + DD, no BTBT.
   Use a stripped-down `sdevice_des.cmd` (remove the Band2Band lines from the template).
   Just do equilibrium solve and ramp drain to 0.1 V.
   Expected: converges in ~5 minutes. Check svisual: Conduction Band Energy should show
   Type-III broken-gap alignment (GaSb VB above InAs CB at the junction).

6. **Day 7:** Verify the broken-gap alignment quantitatively.
   In svisual, do a cutline along the wire axis.
   Plot Conduction Band Energy and Valence Band Energy together.
   At the GaSb/InAs interface, measure E_V(GaSb) - E_C(InAs).
   This should be ~+114 meV (matching Anderson rule prediction).
   This **must** match the +104 meV we measured in ATLAS.
   If it disagrees by more than ~20 meV, fix the material parameters before continuing.

7. **Day 8-10:** Buffer for debugging.

**Decision gate at end of Phase 1:**
- (a) Same device geometry, same band alignment in Sentaurus → proceed to Phase 2
- (b) Geometry doesn't build, or band alignment is off → debug; if stuck >2 weeks, fall back to ATLAS-only paper

### PHASE 2 — Add BTBT physics in Sentaurus (Weeks 4-6, ~15 days)

**Goal:** Get a working Id-Vg curve in Sentaurus with NonlocalPath BTBT model active.

**Tasks:**

1. **Day 1-2:** Add `Band2Band(Model=NonlocalPath)` to the Physics section.
   Add the `NonLocalMesh` block defining the tunnel path region.
   Reference: `COMPLETE_GUIDE.md` Chapter 4.
   
2. **Day 3-4:** Add `eQuantumPotential` and `hQuantumPotential` for quantum confinement.
   This is a Sentaurus advantage — these models work alongside BTBT, unlike ATLAS BQP.

3. **Day 5:** Set up the staged Solve sequence:
   - Stage 1: Coupled(Iterations=100) {Poisson}
   - Stage 2: Coupled {Poisson Electron Hole}
   - Stage 3: Coupled {Poisson Electron Hole eQuantumPotential hQuantumPotential}
   - Stage 4: Quasistationary drain ramp to 0.5 V
   - Stage 5: Quasistationary gate sweep 0 to 1.5 V
   
4. **Day 6-7:** First full Id-Vg run. Expect 30-60 minutes of compute time.
   Save the output to `idvg_sentaurus_baseline.plt`.

5. **Day 8:** Open `idvg_sentaurus_baseline.plt` in inspect.
   Plot drain TotalCurrent vs gate OuterVoltage on log scale.
   Compare visually to ATLAS Hurkx Id-Vg from `gaa_iiiv_hj_tfet_idvg.log`.
   
   Expected differences:
   - Sentaurus I_ON should be 100× to 1000× higher than ATLAS Hurkx (tens of nA vs 2.34 pA)
   - V_T should be similar (within 100-200 mV) — both controlled by gate workfunction
   - SS should be similar or better in Sentaurus (NonlocalPath integrates over physical path)
   - Curve shape should be similar

6. **Day 9-10:** Calibration adjustments.
   The first Sentaurus run uses default A_Kane and B_Kane parameters.
   To match ATLAS calibration philosophy (calibrated against atomistic published results):
   - Read out current Sentaurus I_ON
   - Compare to Carrillo-Nuñez 2017 prediction (~100 µA/µm at V_DD=0.3V, scaled for V_DD=0.5V)
   - Adjust A_Kane and B_Kane in `sdevice.par` to land in the right magnitude range
   - Re-run

7. **Day 11-15:** Buffer for convergence debugging.

**Common Sentaurus issues and fixes:**

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| "Cannot open grid file" | Mesh file wrong path | Edit File { Grid = "..." } |
| "Unknown material" | Material name typo | Check exact spelling: "GaSb", "InAs", "InGaAs", "HfO2" |
| Newton not converging | Step too large | Reduce MaxStep in Quasistationary block |
| Path integration error in NonlocalPath | Tunnel path region too small | Increase Length in NonLocalMesh block (try 10e-3 → 15e-3) |
| Parameter rejected | Wrong name in your version | Run `sdevice -P -m GaSb` to see valid names |

**Decision gate at end of Phase 2:**
- (a) Sentaurus produces a clean Id-Vg with monotonic turn-on and physically reasonable I_ON → proceed to Phase 3
- (b) Sentaurus doesn't converge → debug; if stuck >3 weeks, fall back to ATLAS-only paper

### PHASE 3 — Parametric sweeps in both tools (Weeks 7-10, ~20 days)

**Goal:** Run identical parametric sweeps in both ATLAS and Sentaurus to enable side-by-side comparison.

**Sweep matrix:**

| Parameter | Values | Why this sweep matters |
|-----------|--------|------------------------|
| Gate workfunction | 4.85, 5.00, 5.20, 5.40 eV | V_T tuning sensitivity |
| Y.CHAR (junction sharpness) | 0.5, 1.0, 2.0, 3.0 nm | BTBT sensitivity to abruptness |
| Source doping N_A | 2e19, 5e19, 1e20 cm^-3 | BTBT field dependence |
| Drain doping N_D | 2e19, 5e19 cm^-3 | Ambipolar suppression |
| Wire radius R | 4, 5, 6 nm | Quantum confinement scaling |
| Gate length L_g | 15, 20, 25 nm | Short-channel effects |
| Temperature | 250, 300, 350 K | Thermal sensitivity (TAT vs BTBT) |
| V_DS (for DIBL) | 0.05, 0.5 V | Drain-induced barrier lowering |

That's 8 parameters × 3-4 values each ≈ 25 sweeps per tool, but you only do BASE + ONE sweep at a time so ~8 + 8 = 16 simulations per tool ≈ 32 total.

**For each sweep, extract:**
- V_T (max-slope x-intercept, **and** constant-current at I_D=1e-9 A for cross-comparison)
- SS_min
- SS averaged over 2 decades
- I_ON at V_GS = 1.5 V
- I_OFF (minimum value over the curve)
- I_ON/I_OFF ratio
- DIBL (only for the V_DS sweep)

**Tasks:**

1. **Day 1-2:** Set up automated parameter sweeps.
   For ATLAS: Use `set parameter_name = $value` and `loop steps=N` if you can get it working; otherwise just multiple deck variants.
   For Sentaurus: SWB experiment-tree handles this naturally with `@variable@` placeholders.
   
2. **Day 3-12:** Run the sweep matrix. Each individual run takes 30 min to 2 hours; total ~50-100 hours of compute. Run overnight, batch carefully.

3. **Day 13-15:** Extract metrics. For each sweep, pull the FoM into a table.
   Use `extract` blocks in ATLAS and `inspect` scripts or post-processing in Sentaurus.

4. **Day 16-20:** Buffer for re-runs and debugging.

**Decision gate at end of Phase 3:**
- All sweep results in two tables (one per tool), ready for plotting → proceed to Phase 4

### PHASE 4 — Comparison analysis and figure preparation (Weeks 11-13, ~15 days)

**Goal:** Generate the figures and tables that will appear in the paper.

**Required figures (planned for paper):**

| Fig | Content | Source |
|-----|---------|--------|
| 1 | Device structure: 3D rendering + 2D cross-section + parameter table | Schematic from svisual + ASCII drawing |
| 2 | Energy band diagram at V_DS=0, V_GS=0 — overlay ATLAS and Sentaurus | Cutlines from .str (ATLAS) and .tdr (Sentaurus) |
| 3 | Energy band diagram at V_DS=0.5 V — overlay both tools | Same procedure at biased operating point |
| 4 | ATLAS Id-Vg with three BTBT models (BBT.NONLOCAL [aborts], NEGF_MS [noise], BBT.HURKX [works]) | ATLAS log files; show the failures explicitly |
| 5 | Sentaurus Id-Vg with NonlocalPath BTBT — log scale | inspect plot of `idvg.plt` |
| 6 | Direct overlay: ATLAS Hurkx vs Sentaurus NonlocalPath Id-Vg | Both on one plot, log scale |
| 7 | V_T vs gate workfunction — overlay both tools | Extract sweeps in tabular form, plot |
| 8 | SS_min vs Y.CHAR — overlay both tools | Same |
| 9 | I_ON vs source doping — overlay both tools | Same |
| 10 | Id-Vd family at V_GS=1.1, 1.3, 1.5 V — both tools side by side | Output sweeps in both tools |
| 11 | 2D BTBT generation rate map at V_GS=1.5 V — side by side both tools | Surface plots from svisual / TonyPlot |
| 12 | DIBL extraction summary table (numerical) | Comparison table |

**Required tables:**

| Table | Content |
|-------|---------|
| 1 | Material parameters used in both tools (Vurgaftman 2001 values) |
| 2 | Model selection: which BTBT model in which tool, with status (works/aborts/noise) |
| 3 | Calibration parameters: ATLAS bbt.a/bbt.b vs Sentaurus A_Kane/B_Kane |
| 4 | Extracted FoM comparison: V_T, SS, I_ON, I_OFF, I_ON/I_OFF, DIBL — both tools side by side |
| 5 | Comparison with published atomistic results (Carrillo-Nuñez 2017) and experimental (Memisevic 2017) |

**Tasks:**

1. **Day 1-3:** Generate raw plots in TonyPlot/inspect for each figure.
2. **Day 4-7:** Export raw curve data to ASCII/CSV files.
3. **Day 8-12:** Re-plot in Python matplotlib for publication quality. (Or use Origin if you have it.)
4. **Day 13-15:** Compile tables in Excel/LibreOffice; format for paper.

**Decision gate at end of Phase 4:**
- 12 publication-quality figures and 5 tables ready → proceed to Phase 5

### PHASE 5 — Paper drafting (Weeks 14-17, ~20 days)

**Goal:** Complete first draft of manuscript.

**Section-by-section plan:**

#### Abstract (last to write, 200-250 words)
- 1 sentence: motivation (broken-gap TFET importance)
- 2 sentences: device under study
- 2 sentences: tools compared
- 2 sentences: key findings
- 1 sentence: implication

#### 1. Introduction (1.5-2 pages)
- Boltzmann tyranny and the TFET concept (1 paragraph)
- Heterojunction TFETs and broken-gap (1 paragraph)
- Specific device family (GaSb/InAs/InGaAs GAA) and its prior characterization (1 paragraph)
- TCAD landscape: drift-diffusion vs atomistic; commercial vs free (1 paragraph)
- Gap in literature: no published TCAD comparison for this device class (1 paragraph)
- Contributions of this work (bullet list, 4-5 items)

#### 2. Device Structure and Physics (1-1.5 pages)
- Device dimensions and material stack (with Figure 1)
- Anderson-rule band alignment calculation showing Type-III broken-gap (with equations)
- Why InGaAs drain (suppresses ambipolar)
- Why cylindrical GAA topology (electrostatic integrity)
- Why these specific dimensions (citing Carrillo-Nuñez 2017)

#### 3. TCAD Modeling Methodology (2-3 pages)

**3.1 ATLAS modeling**
- Mesh and refinement strategy
- Material parameters (Table 1)
- Three BTBT models attempted: BBT.NONLOCAL, NEGF_MS, BBT.HURKX
- Each model: brief description, observed behavior, why it works/fails (Figure 4)
- Two-stage solve sequence rationale
- Calibration: bbt.a, bbt.b values and how they were chosen

**3.2 Sentaurus modeling**
- sde geometry build
- NonlocalPath BTBT model description and theory
- eQuantumPotential and hQuantumPotential
- Solve sequence rationale
- Calibration: A_Kane, B_Kane values

**3.3 Cross-tool consistency checks**
- Band alignment verification (both tools should give +114 meV broken-gap, Figure 2-3)
- Parameter equivalence table

#### 4. Results and Discussion (3-4 pages)

**4.1 Equilibrium and OFF-state band diagrams**
- Figure 2 (V=0) and Figure 3 (V_DS=0.5)
- Both tools confirm Type-III alignment numerically

**4.2 Transfer characteristic comparison**
- Figure 5 (Sentaurus alone), Figure 6 (overlay)
- I_ON, V_T, SS, I_OFF in Table 4
- Discussion of why Sentaurus I_ON > ATLAS I_ON (NonlocalPath integration vs local Hurkx)

**4.3 Output characteristics**
- Figure 10
- Discussion of saturation behavior

**4.4 Parametric sensitivity**
- V_T vs workfunction (Figure 7)
- SS vs Y.CHAR (Figure 8)
- I_ON vs source doping (Figure 9)
- Key observation: trends agree across tools even though absolute values differ

**4.5 Comparison with published literature**
- Table 5: our ATLAS, our Sentaurus, Carrillo-Nuñez 2017 atomistic, Memisevic 2017 experimental
- Discussion of where each falls and why

**4.6 Spatial generation rate distributions**
- Figure 11
- Both tools localize BTBT at the source-channel junction (qualitatively agree)

#### 5. Discussion (1.5-2 pages)
- Tool-dependent vs tool-independent findings
- When to use which tool: methodology guidance for future researchers
- Limitations: effective-mass single-band physics in both tools
- What atomistic simulation would add (cite NEMO5/OMEN)
- Implications for TFET design optimization

#### 6. Conclusion (0.5-1 page)
- Summary of findings (3-4 bullet points)
- Methodology recommendations
- Future work directions

#### References (1-2 pages, ~25-30 references)
Pull from `THEORY_AND_DESIGN.md` Section 15 reference list. Add specific tool manual citations (Silvaco ATLAS User's Manual, Sentaurus Device User's Manual).

**Tasks:**

1. **Day 1-3:** Outline expansion. Convert each section above into 5-10 bullet points of content.
2. **Day 4-12:** Section-by-section drafting. ~1 day per section.
3. **Day 13-15:** Figure integration, caption writing.
4. **Day 16-18:** Reference cleanup, bibliography compilation.
5. **Day 19-20:** Internal proofreading.

**Decision gate at end of Phase 5:**
- Complete first draft (8-12 pages, all figures and tables included) → proceed to Phase 6

### PHASE 6 — Review, revision, submission (Weeks 18-20, ~10 days)

**Tasks:**

1. **Day 1-3:** Supervisor review. Get written feedback.
2. **Day 4-7:** Revise based on feedback.
3. **Day 8:** Format for target journal (use journal LaTeX template).
4. **Day 9-10:** Submit.

---

## 4. VENUE SELECTION (where to submit)

**Primary target:** Microelectronics Journal (Elsevier)
- IF ~2.4
- Accepts methodology comparison papers
- Mid-tier, achievable for BE
- 8-16 week review cycle typically
- Average citation: 5-15

**Secondary target if Microelectronics Journal rejects:** Solid-State Electronics (Elsevier)
- IF ~1.6
- Similar scope
- Slightly easier acceptance

**Tertiary target:** Journal of Computational Electronics (Springer)
- IF ~2.0
- More TCAD-focused
- Methodology papers welcome

**IEEE conference fallback:** IEEE INDICON or IEEE iSES
- Conference proceedings (less prestigious than journal)
- Faster turnaround
- Can lead to journal extension later

**Avoid (too high tier for BE methodology paper):**
- IEEE TED, IEEE EDL — they want device physics novelty, not tool comparison
- Nature Sci. Reports — too broad-audience for tool-specific work

---

## 5. RISK REGISTER (what could go wrong)

### Risk 1: Sentaurus license expires or unavailable
**Mitigation:** Verify Day 1. If unavailable, escalate to supervisor. Backup: ATLAS-only methodology paper (still publishable but lower impact).

### Risk 2: Sentaurus convergence issues with broken-gap
**Mitigation:** Reduce step sizes, use staged solve, increase mesh density. If stuck >3 weeks, simplify to drift-diffusion-only Sentaurus comparison (less impact but completable).

### Risk 3: Sentaurus and ATLAS results too similar (no contrast)
**Mitigation:** This itself is a finding! Frame as "both tools converge to similar trends, demonstrating tool-agnostic physical insight."

### Risk 4: Sentaurus and ATLAS results too different (calibration mismatch)
**Mitigation:** Investigate why. Material parameter differences? Mesh differences? This becomes a longer Discussion section.

### Risk 5: Reviewer rejects as "just a methodology comparison, no novel physics"
**Mitigation:** Reframe as methodology guidance for future researchers; cite that this is a documented gap in the literature; provide tabular calibration recipes that have practical reuse value.

### Risk 6: BE coursework consumes time
**Mitigation:** Plan for 10-15 hours/week, not 40 hours/week. Stretch timeline to 6 months if needed.

### Risk 7: Supervisor wants different angle
**Mitigation:** Discuss this plan with supervisor in week 1 before committing. Adapt thesis statement if needed.

---

## 6. SPECIFIC FIRST ACTIONS FOR THE NEXT AI / NEXT SESSION

When the next chat session begins, the next AI should:

### Step 1: Confirm status
Read this document (`docs/PAPER_1_PLAN.md`) plus:
- `docs/HANDOFF.md`
- `docs/MODELS.md`
- `docs/COMPLETE_GUIDE.md`
- `docs/THEORY_AND_DESIGN.md`

This gives complete context.

### Step 2: Ask the user where they are in the plan
Specifically:
- "Have you completed Phase 0 (Sentaurus environment verification)?"
- "If yes, are you in Phase 1 (sde geometry build) or further?"
- "Are there any specific issues you're stuck on?"

### Step 3: Provide concrete deck modifications and command-line steps
Don't write theory documents — the theory is already documented. Write:
- Specific shell commands to run
- Specific deck modifications
- Specific extraction syntax

### Step 4: Suggest the next single action
Always end with: "your next single action is [X]". Avoid overwhelming with options.

### Step 5: Maintain the repository
Commit frequently. Each working deck variant should be a git commit. Branch by phase if needed.

---

## 7. WHAT THE FINAL PAPER WILL LOOK LIKE (sanity check)

A successful Paper 1 produces:
- 8-12 pages of manuscript
- 12 figures, 5 tables
- 25-30 references
- Submitted to Microelectronics Journal or equivalent
- Realistic outcome: accepted with minor revisions, published 6-12 months from submission
- Citation count after 2 years: 5-15
- Acknowledgment in user's BE thesis

This is a respectable BE-level outcome. It is NOT going to change the world. It IS going to:
- Get the user a published paper
- Demonstrate two-tool TCAD competence on the user's CV
- Provide a foundation for Paper 2 (cryogenic study or other application)
- Be a defensible thesis chapter

---

## 8. QUICK-REFERENCE CHEAT SHEET FOR THE USER

When the user asks "what should I do today?", here's the table to consult:

| Phase | Activity | Output | Time |
|-------|----------|--------|------|
| 0 | Verify Sentaurus environment | Working `sdevice -h` | 1 week |
| 1 | Build geometry in sde | `gaa_tfet_msh.tdr` | 2 weeks |
| 2 | Add NonlocalPath BTBT | First Id-Vg in Sentaurus | 3 weeks |
| 3 | Parametric sweeps both tools | 32 simulation results, FoM tables | 4 weeks |
| 4 | Generate figures/tables | 12 figures + 5 tables | 3 weeks |
| 5 | Write paper | 8-12 page draft | 4 weeks |
| 6 | Review and submit | Submitted manuscript | 2 weeks |

**Total: ~20 weeks ≈ 5 months part-time.**

---

## 9. PROJECT CONTEXT FOR FUTURE AI

If you (the next AI) are reading this for the first time, here is the executive summary:

- **User:** BE final-year student, pursuing one published paper
- **Device:** GaSb/InAs/InGaAs broken-gap GAA TFET (5 nm radius, 20 nm gate)
- **Status:** ATLAS work done. Sentaurus side is the new work for this paper.
- **Tool access:** ATLAS 2019 + Sentaurus N-2017 confirmed available
- **Paper plan:** This document
- **All other context:** in `docs/` folder of this repo

The single most important thing to know: **the user's I_ON of 2.34 pA in ATLAS is NOT a bug**. It is the documented ceiling of effective-mass commercial TCAD for Type-III broken-gap. We have already documented this exhaustively in `docs/MODELS.md` and `docs/COMPLETE_GUIDE.md`. The paper's contribution is the cross-tool comparison and methodology, NOT atomistic-grade I_ON.

Never tell the user "your I_ON is too low — let's increase A_Kane" without checking the documented ceiling. The user knows this. The next AI should know this.

---

## 10. DOCUMENT VERSION

Version: 1.0
Date: at handoff
Status: complete plan, ready for execution starting Phase 0
