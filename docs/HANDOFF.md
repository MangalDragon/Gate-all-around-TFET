# Complete Handoff: GaSb / InAs / InGaAs Broken-Gap GAA TFET

This document is the end-of-session handoff for the broken-gap Gate-All-Around
TFET project. It records what was tried, what worked, what is fundamentally
blocked in ATLAS 2019, and the ranked set of paths forward (including the
already-downloaded Sentaurus N-2017 and free atomistic alternatives).

## 1. The Device

| Field         | Value                                                     |
|---------------|-----------------------------------------------------------|
| Source        | GaSb, p+ 5x10^19 cm^-3                                    |
| Channel       | InAs, n- ~10^15 cm^-3                                     |
| Drain         | In0.53Ga0.47As, n+ 5x10^19 cm^-3                          |
| Geometry      | Cylindrical GAA nanowire                                  |
| Radius        | 5 nm                                                      |
| Gate length   | 20 nm                                                     |
| Gate oxide    | HfO2, 2 nm                                                |
| Junction type | Type-III broken-gap: E_V(GaSb) ~114 meV above E_C(InAs)   |

That last row is the entire problem. Confirmed numerically from a `.dat`
cross-section export of the saved structure: junction overlap = +104 meV,
bulk-to-bulk overlap = +199 meV. The device is genuinely broken-gap, not
Type-II.

## 2. Verified Working in ATLAS 2019

`simulations/gaa_iiiv_hj_tfet_hurkx_production.in` (or its earlier sibling
`gaa_iiiv_hj_tfet.in`) runs end-to-end and produces:

| Metric                       | Value           |
|------------------------------|-----------------|
| V_T (V_DS = 0.5 V)           | 1.07 V          |
| SS (min)                     | 103 mV/dec      |
| I_ON (V_GS = 1.5 V, V_DS=0.5)| 2.34 pA         |
| I_OFF                        | 8.5x10^-17 A    |
| I_ON / I_OFF                 | ~10^3           |

The curve shape is correct (monotonic turn-on, sub-thermal SS region,
saturating Id-Vd family). The I_ON of 2.34 pA is the problem - published
TCAD studies of the same device family report I_ON in the nA-uA range,
three to six orders of magnitude higher.

## 3. The Three Fundamental Challenges in ATLAS 2019

### Challenge 1 - BBT.NONLOCAL aborts with WKB Code 2

The non-local Kane / WKB integrator computes

    T(E) ~ exp(-2 * integral sqrt(2 m (V(x) - E) / hbar^2) dx)

which requires a forbidden barrier `V(x) - E > 0` along the tunnel path.
For Type-III, E_V(GaSb) > E_C(InAs), so the radicand goes negative and
ATLAS aborts.

Why this can't be patched: the WKB approximation itself fundamentally
requires a barrier. Removing the abort would just produce nonsense.
ATLAS is correctly refusing to simulate.

### Challenge 2 - NEGF_MS returns T(E) ~ 0 (numerical noise)

ATLAS effective-mass mode-space NEGF builds the device Hamiltonian on a
per-slice transverse Schroedinger basis from the conduction-band
effective masses. There is no off-diagonal matrix element coupling the
GaSb valence band to the InAs conduction band, so the retarded Green's
function returns T(E) = 0 across the broken-gap junction.

The Id-Vg from this run was 10^-21 A oscillating randomly through zero -
the round-off noise of an inverter computing exactly zero transmission.
The saved `.str` contained no carrier solution because NEGF's internal
arrays were never populated.

Why this can't be patched: it's a model-class limitation, not a tuning
knob. Changing `NPRED.NEGF`, `ESIZE.NEGF`, or `EIGEN` cannot introduce
CB-VB coupling that the Hamiltonian doesn't contain. Only an atomistic
or k.p multi-band Hamiltonian has that coupling.

### Challenge 3 - local-Hurkx underpredicts I_ON by orders of magnitude

`BBT.HURKX` works (it's a local generation rate, not a WKB integral, so
it doesn't crash on broken-gap) and produces the V_T = 1.07 V, SS = 103
mV/dec, I_ON = 2.34 pA result. But Hurkx is a fitted local approximation
with parameters `bbt.a` (rate prefactor) and `bbt.b` (Kane field
exponent). Even at `bbt.a = 9.1e15` (already 10x the textbook III-V
value), I_ON is in the pA range, not nA-uA.

Why this can't be patched cleanly: raising `bbt.a` further is just a
fit. Real BTBT in this device is non-local along a few nm of path
through the actual band-edge profile, and the Hurkx local approximation
systematically misses that. You can keep cranking the prefactor until
I_ON matches a target, but it stops being calibrated physics and
becomes pure curve-fitting.

### Why the noise floor and Hurkx results aren't contradictory

Both are correct within their model. NEGF returns zero because no CB-VB
matrix element exists in the Hamiltonian. Hurkx returns a small nonzero
current because it's a phenomenological generation rate that doesn't
know about heterojunction coupling at all - it just multiplies a local
rate by a function of the local field.

## 4. Solution Paths, Ranked

### Path A - Sentaurus TCAD N-2017 (downloaded, most actionable)

Sentaurus is meaningfully better than ATLAS 2019 for this class of
device, but it does not solve the problem completely. Try in this order:

**A.1 Sentaurus's Schenk non-local BTBT model.** The Schenk full-band
BTBT model is constructed to handle heterointerfaces and degenerate
doping more robustly than ATLAS's WKB-based `BBT.NONLOCAL`. It computes
the BTBT generation rate using the Kane two-band model integrated over
the actual band-bending path, including phonon-assisted as well as
direct contributions. May converge where ATLAS aborts. Worth the first
try.

**A.2 Sentaurus's NonlocalPath BTBT mesh.** Sentaurus lets you define
non-local tunnel paths explicitly via `NonlocalPath` regions, with
finer control over path geometry than ATLAS's `QTX.MESH` / `QTY.MESH`.
For a broken-gap junction, you can constrain the integration path so it
doesn't cross regions where the radicand goes negative. Likely how a
working non-local BTBT result is achieved. Expected I_ON: probably
10-100x higher than ATLAS local-Hurkx, so nA range. Not yet uA.

**A.3 Sentaurus's multi-band eMultiValley / hMultiValley for III-V.**
More developed multi-valley models for III-V CB structure (Gamma, L, X)
and split-off VB. Marginal for broken-gap (dominant tunneling is
between Gamma-valleys), but more accurate than single-valley.

**A.4 Sentaurus density-gradient quantum confinement.** More mature DG
and BQP than ATLAS. Improves V_T accuracy by ~50-100 meV. Doesn't
change the fundamental BTBT issue.

**What N-2017 cannot do.** It still doesn't do atomistic tight-binding
NEGF. For publication-grade quantitative match (uA range that
Avci 2015 / Carrillo-Nunez 2017 report) you'd need Synopsys QuantumATK
addon or a different tool. N-2017 will probably get you from pA to nA,
not from pA to uA.

**Sentaurus migration steps (sketch):**

1. Convert geometry. Sentaurus uses Mesh (`sde`), structure description
   files (`*_msh.tdr`). Cylindrical mesh is supported via `Refinement`
   blocks with `Cylindrical` axis.
2. Material parameters move to `Sentaurus/Models/Materials/` as `.par`
   files. Same physical values (chi=4.06 for GaSb etc.) but different
   syntax.
3. Physics block replaces ATLAS models. Equivalent of the Hurkx
   baseline:
   ```
   Physics(Region="SRC_GaSb"){
     Recombination(SRH Auger Band2Band(Schenk))
     EffectiveIntrinsicDensity(BandGapNarrowing(Slotboom))
     Mobility(DopingDep HighFieldSaturation)
   }
   ```
4. Solve sequence in Sentaurus Workbench (`*_des.cmd`):
   `Solve { Coupled{Poisson Electron Hole} }` for DD-only first, then
   `Coupled{Poisson Electron Hole eQuantumPotential hQuantumPotential}`
   for quantum.
5. Use Sentaurus Visual instead of `tonyplot`.

### Path B - Silvaco upgrade to 2022+ (Victory Atomistic)

Victory Atomistic is Silvaco's port of the Nemo5 codebase. It does
atomistic tight-binding NEGF with full sp3d5s* basis and handles
broken-gap natively. Same physics as Carrillo-Nunez 2017.

Pros: publication-quality results, drop-in continuation of existing
decks. Cons: new license cost, hours per Id-Vg point on a 5 nm GAA
nanowire. Academic licensees may be able to swap modules within an
existing license.

### Path C - Free / academic atomistic tools

These are the tools the published broken-gap TFET literature actually
uses.

| Tool        | Origin              | Capability                           | Notes |
|-------------|---------------------|--------------------------------------|-------|
| NEMO5       | Purdue (Klimeck)    | Atomistic TB-NEGF (sp3d5s*), broken-gap native | Used by Carrillo-Nunez 2017, Avci 2015. XML input, HPC-only, hours per bias point. |
| OMEN        | ETH Zurich (Luisier)| Atomistic TB-NEGF + DFT-NEGF, GPU    | More actively maintained than Nemo5. |
| KWANT       | TU Delft / Madrid   | TB transport, Landauer-Buttiker      | Pure Python, `pip install kwant`. Best for prototyping the broken-gap transmission spectrum. |
| NESS        | Glasgow (Asenov)    | 3D Poisson + DD/MC/NEGF, broken-gap  | More approachable than Nemo5/OMEN. |
| TBtrans / TranSiesta | -          | DFT-NEGF via Siesta + Python (SISL)  | Days per bias point for full device, useful for small reference calculations. |
| QuantumATK  | Synopsys (ex-QuantumWise) | DFT-NEGF + TB-NEGF + multiscale | Free academic licenses at many universities. |
| Genius / VisualTCAD | Cogenda     | Full TCAD, non-local BTBT, DG        | Free academic. Less polished. |
| TiberCAD    | Rome Tor Vergata    | Multiscale, k.p + DD coupling        | Built for III-V quantum-confined devices. |

### Path D - Pragmatic hybrid: keep ATLAS Hurkx, cite atomistic literature

What an honest TCAD-based publication would do. Methodology section
reads:

> BTBT current was simulated using the local Hurkx model in Silvaco
> ATLAS 2019, calibrated to bbt.a = 9.1e15, bbt.b = 4.0e6 following
> [Bahuguna 2017]. The local-Hurkx approach is known to underestimate
> broken-gap I_ON because it lacks the non-local CB-VB coupling that
> atomistic tight-binding NEGF treats explicitly [Carrillo-Nunez 2017].
> Quantitative I_ON for this device family in the uA range has been
> reported using TB-MS-NEGF [Carrillo-Nunez 2017, Avci 2015]; we use
> ATLAS for the V_T and SS trends across design-space sweeps and
> reference the atomistic literature for absolute I_ON values.

That paragraph plus the working ATLAS deck for parametric sweeps plus
the cited atomistic numbers for absolute calibration is a defensible
thesis chapter or paper. Many published TCAD-only studies of this
device family do exactly this.

## 5. Recommendation Ranking

For someone with Sentaurus N-2017 downloaded and ATLAS 2019 already
used, wanting to move forward:

1. **Try Sentaurus N-2017's Schenk BTBT first.** Already downloaded;
   non-local BTBT will probably get from 2.34 pA to ~100 pA - 1 nA
   range without any new tooling. That alone makes the result more
   publishable.
2. **In parallel, install KWANT** (`pip install kwant`) and prototype
   the GaSb/InAs broken-gap transmission in 1D as a sanity check on
   whatever Sentaurus produces. Fast to learn, free atomistic
   reference.
3. **For publication-grade I_ON (uA range):** pursue OMEN academic
   license or QuantumATK academic license. The right tools long-term.
4. **For all of the above:** keep the ATLAS Hurkx baseline as the
   trend-line / parametric-sweep deck. Verified, fast, gives V_T/SS
   scaling that Sentaurus or atomistic tools would give absolute I_ON
   for.
5. **Don't waste more time on ATLAS NEGF_MS.** It cannot, by
   construction, simulate broken-gap BTBT. The `.dat` extracted proves
   the structure is correct; the model just doesn't have the matrix
   element.

## 6. What to Bring to the New Chat

- This document (`docs/HANDOFF.md`).
- PR #6: https://github.com/MangalDragon/Gate-all-around-TFET/pull/6
- Verified ATLAS baseline numbers: V_T = 1.07 V, SS = 103 mV/dec,
  I_ON = 2.34 pA at V_DS = 0.5 V.
- Working production deck (`simulations/gaa_iiiv_hj_tfet_hurkx_production.in`
  once committed, or `gaa_iiiv_hj_tfet.in`).
- The `.dat` band-cutline confirming +104 meV junction overlap (on disk
  locally).
- Reference: Carrillo-Nunez et al., arXiv:1705.00909 (2017) -
  atomistic-NEGF study of this exact device family.
- Reference for Schenk model: Schenk, Solid-State Electronics 36, 19
  (1993), used by Sentaurus.
- Note that Sentaurus N-2017 is installed and ready to use.

## 7. Repository File Map

| File                                                     | Purpose                                                              |
|----------------------------------------------------------|----------------------------------------------------------------------|
| `simulations/gaa_iiiv_hj_tfet.in`                        | Local-Hurkx + BQP + TAT baseline (`gate_wf=4.35`)                    |
| `simulations/gaa_iiiv_hj_tfet_negf.in`                   | NEGF Mode-Space deck - runs but produces noise floor                 |
| `simulations/gaa_iiiv_hj_tfet_negf_diag.in`              | Probe-based band-cutline diagnostic                                  |
| `simulations/gaa_iiiv_hj_tfet_v9_fixed.in`               | Corrected v9 with two-stage solve, BQP, full ambipolar VG sweep      |
| `simulations/gaa_iiiv_hj_tfet_improved.in`               | Workfunction-sweep + DIBL deck                                       |
| `simulations/original_v9.in`                             | Original v9 (preserved for reference)                                |
| `simulations/gaa_iiiv_hj_tfet_hurkx_production.in`       | Production Hurkx deck that produced the verified numbers (gate_wf=5.20, Ychar=0.001, LDD spacer, bbt.a=9.1e15, bbt.b=4.0e6) - to be committed |
| `docs/MODELS.md`                                         | Per-model rationale + NEGF closing analysis (sections 9, 10)         |
| `docs/REVIEW.md`                                         | Earlier physics review                                               |
| `docs/HANDOFF.md`                                        | This file                                                            |

## 8. Status of Files in the Repo (as of this commit)

Not all of the files listed in section 7 are currently committed to
`main`. As of the commit that adds this document:

- On `main`: `gaa_iiiv_hj_tfet.in`, `original_v9.in`, `docs/MODELS.md`,
  `docs/REVIEW.md`.
- In open PR #6 (branch `negf-carriers-fix`): updates to `docs/MODELS.md`
  adding the NEGF closing analysis (sections 9 and 10).
- Not yet pushed: `gaa_iiiv_hj_tfet_negf.in`,
  `gaa_iiiv_hj_tfet_negf_diag.in`, `gaa_iiiv_hj_tfet_v9_fixed.in`,
  `gaa_iiiv_hj_tfet_improved.in`,
  `gaa_iiiv_hj_tfet_hurkx_production.in`. These exist on the local
  development machine but have not been committed to GitHub. They
  should be added in a follow-up commit so the repo state matches the
  description in section 7.
