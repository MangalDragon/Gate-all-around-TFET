# Physics models for III-V GAA Heterojunction TFETs in Silvaco ATLAS

This document explains why each model in
`simulations/gaa_iiiv_hj_tfet.in` is selected, with references. It is
written for ATLAS 2019 (the same model set is valid in newer releases;
flag names have not changed).

> Content of cited Silvaco notes is paraphrased for licensing compliance.

## 1. Band-to-band tunneling (BTBT) - the primary current

**Selected:** `BBT.NONLOCAL BBT.NLDERIVS`

A TFET turns on by interband tunneling at the source/channel junction.
That tunneling is fundamentally non-local: the carrier is generated on
one side of the junction and recombines on the other, and the WKB
integral has to be evaluated along the actual band-bending path, not
locally at one point.

The local alternatives in ATLAS - `BBT.STD` (Kane), `BBT.KANE`,
`BBT.KL` (Klaassen), and `SCHENK.BBT` - all approximate the tunneling
generation rate as a function of the local electric field and band gap.
For a heterojunction TFET that reduces accuracy by 1-2 orders of
magnitude on I_ON because the broken-gap alignment is fundamentally a
spatial-band-edge effect.

`BBT.NONLOCAL` evaluates the WKB transmission on the **non-local mesh**
defined by `QTX.MESH` / `QTY.MESH`. The mesh must enclose the source/
channel junction with sub-nm spacing in the tunneling direction. In
this deck Y=0.020 is the junction and we have `SPACING=0.0001` (= 0.1 nm)
across it. *Going below 0.1 nm is counter-productive*: in the previous
revision we tried 50 pm and the WKB integrator started returning Code 2
warnings (over-discretization). 0.1 nm with 0.2 nm transition zones
matches Silvaco's own GAA TFET examples.

`BBT.NLDERIVS` is a numerical option, not a physics one: it tells the
solver to keep the analytic Jacobian of the BTBT generation rate with
respect to the potential. Without it, Newton sees BTBT as a "frozen"
right-hand-side and convergence collapses near threshold.

When *not* to use `BBT.NONLOCAL`:
- An indirect-gap material (Si, Ge) where phonon assistance is critical:
  prefer `SCHENK.BBT`.
- A tunneling problem where the relevant junction is curved on a scale
  finer than the mesh can resolve - then the non-local algorithm picks
  spurious tunnel paths.

**Tunneling masses.** The non-local model uses `me.tunnel` / `mh.tunnel`
as the isotropic effective tunneling mass. Recommended values used in
the deck (Vurgaftman 2001 + Avci 2015 + Bahuguna 2017):

| Material | m_e^tun | m_h^tun |
|----------|---------|---------|
| InAs     | 0.023   | 0.026   |
| GaSb     | 0.039   | 0.40    |
| In0.53Ga0.47As | 0.041 | 0.050 |

The `bbt.a` and `bbt.b` Kane coefficients are **inert** when
`BBT.NONLOCAL` is on - they belong to the local Kane formula. Leaving
them in the material card is harmless but misleading; the upgraded deck
removes them.

References:
- [Silvaco TCAD Simulations of TFET and Tunneling Diode](https://silvaco.com/simulation-standard/tcad-simulations-of-tfet-and-tunneling-diode/)
- [Tunneling FET Calibration Issues - Sentaurus vs Silvaco TCAD](https://www.researchgate.net/publication/347929897_Tunneling_FET_Calibration_Issues_Sentaurus_vs_Silvaco_TCAD)

## 2. Trap-assisted tunneling (TAT) - the SS floor

**Selected:** `TAT.NONLOCAL` - but turned on **after** the operating
point is established, not at zero bias.

TAT is the dominant subthreshold leakage mechanism in real III-V TFETs.
It is the field-enhanced phonon-assisted emission of carriers from
mid-gap interface traps to the bands. Without it, simulated SS values
fall to the BTBT-only limit (~5-15 mV/dec), which is much steeper than
any measured TFET (30-80 mV/dec).

`TAT.NONLOCAL` reuses the same `QTX.MESH` / `QTY.MESH` mesh as the
non-local BTBT, evaluates the WKB transmission probability for emission
from a trap at energy E_T to the band edge, and scales the local SRH
recombination rate accordingly.

**Why TAT is staged after BTBT.** For a broken-gap heterojunction, the
zero-bias equilibrium has E_V(GaSb) ~114 meV above E_C(InAs). At that
configuration the TAT WKB integrator can hit invalid energy ranges and
emit `Code 2 in GetTransmissionProbability` warnings; layered on top of
the broken-gap BTBT current, Newton then fails to find a self-consistent
zero-bias equilibrium. Empirically the cleanest path is:

1. DD-only equilibrium.
2. Add `BBT.NONLOCAL`, ramp drain to operating bias (V_DS = 0.5 V).
3. Now add `TAT.NONLOCAL` and `solve prev`.
4. Run the gate sweep.

This is what `simulations/gaa_iiiv_hj_tfet.in` does (sections 9, 10, 11).

Local fallback: `TRAP.TUNNEL` modifies the SRH lifetimes by a Hurkx-style
field enhancement factor. Less accurate but cheaper and does not require
a non-local mesh - a reasonable choice for fast design-space sweeps.

The interface trap density is currently expressed indirectly through the
finite surface recombination velocity `s.n=s.p=5e2 cm/s` on the
`interface` statement. To pin a specific D_it, replace the
`interface s.n s.p` line with explicit `interface qf=... nti=... ntd=...`
or `intdefects` blocks.

References:
- [Silvaco - New Thermionic Emission and Tunneling Models in ATLAS](https://silvaco.com/simulation-standard/new-thermionic-emission-and-tunneling-models-in-atlas/)
- [Modified Hurkx BTBT model - TCAD perspective](https://www.researchgate.net/publication/337567515_Modified_Hurkx_band-to-band-tunneling_model_for_accurate_and_robust_TCAD_simulations)
- [Quantum Tunneling Model of a P-N Junction in Silvaco](https://www.researchgate.net/publication/235182791_Quantum_Tunneling_Model_of_a_P-N_Junction_in_Silvaco)

## 3. Quantum confinement - V_T correction in 5 nm wires

**Selected:** *not* enabled. V_T calibration is delegated to gate
workfunction. The reasoning below explains why.

For R = 5 nm (10 nm diameter), the lowest InAs sub-band is shifted
~150-250 meV above bulk E_c. A drift-diffusion deck without quantum
correction therefore predicts V_T much too low and I_ON much too high.

The natural ATLAS quantum correction is the **Bohm Quantum Potential**
(`BQP.N BQP.P`). It is recommended over `DGLOG` (density gradient) by
the Silvaco BQP application note for nanowires.

**ATLAS 2019 caveat: BQP cannot be combined with BBT.NONLOCAL on the
same solve.** Two failure modes occur in this 5.28.1.R build:

1. *Method conflict.* Activating `BQP.N`/`BQP.P` triggers
   `Must specify BLOCK for Bohm Quantum Potential`,
   `Setting solution method to BLOCK`. Any subsequent `method newton`
   is silently overridden.
2. *Block-iteration divergence.* When the BLOCK iteration carries the
   BQPn/BQPp auxiliary unknowns *and* the BTBT generation rate, the
   non-linear residuals grow rather than shrink. The solver then
   aborts with a misleading `Need to specify NEWTON or BLOCK method
   to use BBT.NONLOCAL model` error - the real cause is divergence.

The standard published TFET decks (including Silvaco's own simulation-
standard examples) handle this by **not combining BQP with BTBT**.
Instead, V_T is anchored by tuning the gate workfunction (or a fixed
interface charge). Both knobs shift V_T monotonically and are easier
to calibrate against measured or atomistic-reference data than the BQP
gamma/alpha pair.

The deck exposes `set gate_wf = 4.35` for that purpose. As a guideline:

| Target V_T shift | Delta workfunc to apply |
|------------------|-------------------------|
| +100 mV          | +0.10 eV                |
| +200 mV          | +0.20 eV                |

(In the 100-300 meV range it is approximately 1:1 because the gate is
wrapped on a thin wire.)

For ultimate accuracy in sub-10 nm wires, the next step is NEGF mode-
space (`NEGF_MS`) or full Victory Atomistic (`NEGF_PL1D`). Both are
covered by separate licenses; they are not used in this deck.

References:
- [Silvaco - A New Efficient Quantum Method: The Bohm Quantum Potential Model](https://silvaco.com/simulation-standard/a-new-efficient-quantum-method-the-bohm-quantum-potential-model/)
- [Silvaco - Comparison of 3-D Quantum Effects in Nano Devices Using BQP](https://silvaco.com/simulation-standard/comparison-of-3-dimensional-quantum-effects-in-nano-devices-using-the-atlas-3d-bqp-model/)
- [Silvaco - Quantum Transport Simulation at Atomistic Accuracy of a Nanowire FET](https://silvaco.com/simulation-standard/quantum-transport-simulation-at-atomistic-accuracy-of-a-nanowire-fet/)

## 4. Mobility - III-V appropriate

**Selected:** `FLDMOB` only, with explicit `mun` / `mup` and `vsatn` /
`vsatp` per material card.

Dropped:
- `CONMOB`: silicon doping-dependent mobility tables. ATLAS has no
  III-V tables under that name.
- `BGN`: Slotboom band-gap narrowing model. Its parameters are
  Si-specific.

Set per material:

| Material  | mu_n (cm^2/V/s) | mu_p (cm^2/V/s) | v_sat,n (cm/s) | v_sat,p (cm/s) |
|-----------|-----------------|-----------------|----------------|----------------|
| InAs      | 20000           | 500             | 4.0e7          | 4.0e7          |
| GaSb      | 3000            | 1000            | 7.0e6          | 7.0e6          |
| In0.53Ga0.47As | 12000      | 300             | 2.5e7          | 2.5e7          |

For doping-dependent III-V mobility you can layer in `ALBRCT.N` (Albrecht
ionized-impurity scattering) or replace the constant `mun`/`mup` with a
`mob` block; both are out of scope for the first cut.

## 5. Statistics

**Selected:** `FERMI`

Source and drain are degenerate (5e19), so Fermi-Dirac is mandatory; a
Boltzmann statistic would over-estimate carrier density by a factor of
~e at these doping levels. `NI.FERMI` is implied by `FERMI` and is not
specified separately in the upgraded deck.

## 6. Numerical method

**Two method statements, one per stage of physics:**

| Stage | Models | `method` line |
|-------|--------|---------------|
| A (DD only) | `fermi srh auger fldmob` | `gummel newton autonr trap maxtrap=10 itlimit=50 climit=1e-5 dvmax=0.5` |
| B,C,D,E (BTBT on) | `+bbt.nonlocal +tat.nonlocal` | `newton autonr trap maxtrap=30 itlimit=100 climit=1e-5 dvmax=0.05` |

The Stage A method may use Gummel because DD has no non-local terms.

**Stage B onward MUST drop the `gummel` keyword.** ATLAS 2019 errors out
immediately with `Need to specify NEWTON or BLOCK method to use
BBT.NONLOCAL model` if `gummel` appears in the method line while
BBT.NONLOCAL is enabled. The reason is structural: Gummel decouples the
electron and hole continuity equations, but BBT.NONLOCAL contributes
off-diagonal Jacobian terms (the `BBT.NLDERIVS` flag exists precisely
because those terms are essential), so it must run inside the carrier-
coupled Newton solver.

- `newton autonr trap`: carrier-coupled Newton with auto-NR damping and
  bias-step bisection on failure.
- `maxtrap=30`: 30 levels of bisection. Generous; lower numbers fail
  more often near threshold.
- `climit=1e-5`: tighter than ATLAS default `1e-4`. The off-state Id
  is in the fA/um range and looser tolerances pollute it.
- `dvmax=0.05`: cap per-Newton-step potential update at 50 mV. Helps
  damping near the BTBT onset.

## 7. Solve sequence

**This is the most important section in the document.** The deck never
solves a BTBT-aware equilibrium. The math behind that decision:

```
  E_V(GaSb)             = -chi - Eg = -4.06 - 0.726 = -4.786 eV
  E_C(InAs)             = -chi      = -4.900 eV
  E_V(GaSb) - E_C(InAs) = +0.114 eV    (Type-III broken-gap)

  E_F(GaSb p+, 5e19) ~= E_V(GaSb)   = -4.786 eV
  E_F(InGaAs n+,5e19)~= E_C(InGaAs) = -4.500 eV
  V_bi               ~= 0.286 V across source/drain
```

E_V(GaSb) is 114 meV ABOVE E_C(InAs). The barrier height between the
GaSb VB and the InAs CB at the metallurgical junction is therefore
*negative* at zero bias, which makes the WKB transmission integral
sign-indefinite. ATLAS reports this as `Code 2 in
GetTransmissionProbability` and Newton diverges from any DD-only
initial guess. There is also already a 0.286 V built-in tilt across
the device.

The reliable sequence we use:

1. **Stage A.** DD-only `solve init`, then DD-only drain ramp from 0
   to V_DS = 0.5 V. DD physics is unconditionally well-conditioned
   at any bias.
2. **Stage B.** Now turn on `BBT.NONLOCAL` + `BBT.NLDERIVS`,
   `method newton` (no Gummel), `solve prev` at V_DS = 0.5 V. The
   bands are tilted by V_bi + V_DS ~ 0.79 V, the BTBT direction is
   unambiguous (forward at the source), and Newton sees BTBT as a
   small perturbation.
3. **Stage C.** Add `TAT.NONLOCAL`, `solve prev` again. The carrier
   distribution is settled, so the non-local TAT integrator has a
   sensible WKB path.
4. **Stage D.** Sweep gate 0 -> 1.5 V at 25 mV steps under
   `log master`.
5. **Stage E.** For Id-Vd at V_GS = 0.5, 1.0, 1.5 V: from the Id-Vg
   endpoint, walk drain DOWN to 0 V in small steps, walk gate to the
   target V_GS, then sweep drain.

Lifetimes are relaxed from `taun0 = taup0 = 1e-9 s` to `1e-7 s`.
The shorter value made the broken-gap equilibrium artificially stiff
(BTBT generation balanced by very fast SRH recombination); 100 ns is
the typical InAs/GaSb literature value.

## 8. What is **not** in this deck (and why)

| Effect                                | Why omitted                                                              |
|---------------------------------------|--------------------------------------------------------------------------|
| Drain underlap                        | Geometry change; deferred. Suppresses ambipolar BTBT at high V_GS.        |
| `NEGF_MS` / `NEGF_PL1D`               | Different ATLAS/Victory license; useful only for sub-10 nm wires.         |
| `DGLOG` density gradient              | Replaced by BQP - per Silvaco's own recommendation for nanowires.         |
| Phonon-limited mobility tables (`KLA`)| III-V tables not standard in ATLAS 2019; constant + fldmob is preferable. |
| Gate leakage (`FNORD`, direct tunnel) | HfO2 thickness 2 nm is borderline; can be added later if leakage matters. |
| Self-heating                          | Single-V_DS sweep up to 0.5 V; thermal effect <1% in this device.         |
