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
across it.

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

**Selected:** `TAT.NONLOCAL`

TAT is the dominant subthreshold leakage mechanism in real III-V TFETs.
It is the field-enhanced phonon-assisted emission of carriers from
mid-gap interface traps to the bands. Without it, simulated SS values
fall to the BTBT-only limit (~5-15 mV/dec), which is much steeper than
any measured TFET (30-80 mV/dec).

`TAT.NONLOCAL` reuses the same `QTX.MESH` / `QTY.MESH` mesh as the
non-local BTBT, evaluates the WKB transmission probability for emission
from a trap at energy E_T to the band edge, and scales the local SRH
recombination rate accordingly.

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

**Selected:** `BQP.N BQP.P` (Bohm Quantum Potential)

For R = 5 nm (10 nm diameter), the lowest InAs sub-band is shifted
~150-250 meV above bulk E_c. A drift-diffusion deck without quantum
correction therefore predicts V_T much too low and I_ON much too high.

The Bohm Quantum Potential model is the recommended quantum correction
in ATLAS for two reasons (per the Silvaco BQP application note):
1. Two calibration parameters (gamma, alpha) per carrier - more
   flexibility than density-gradient's single knob.
2. Numerically stable, decoupled from the choice of transport model
   (drift-diffusion or hydrodynamic).

Default starting values: `gamma.n = gamma.p = 1.4`, `alpha.n = alpha.p =
0.3`. These were calibrated against Schroedinger-Poisson for silicon
nanowires and reproduce the InAs nanowire first-sub-band shift to within
~20 meV. For final calibration, fit `gamma`/`alpha` to either an
atomistic (NEMO/Victory Atomistic) reference or to a published ETB
calculation for the same diameter.

Alternative: `DGLOG` (Density Gradient). The ATLAS BQP note explicitly
recommends BQP over DG for nanowires.

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

**Selected:** `gummel newton autonr trap maxtrap=30 climit=1e-5 dvmax=0.2`

- `gummel newton`: Gummel iterations on the equilibrium solve before
  switching to Newton. Critical when BTBT generation is on - Newton
  alone has no good initial guess for the BTBT term.
- `autonr`: automatic Newton-Raphson damping; usually halves the number
  of failed bias points.
- `trap maxtrap=30`: bias-step bisection on Newton failure. 30 trap
  levels is generous; lower numbers fail more often near threshold.
- `climit=1e-5`: tighter than the original `1e-4`. The Id ramp at low
  V_GS has currents in the fA/um range, and 1e-4 was actually polluting
  the off-state.
- `dvmax=0.2`: cap the per-Newton-step potential update at 0.2 V; helps
  damping near the BTBT onset.

## 7. Solve sequence

The solve sequence is a *physics warm-start*:

1. Equilibrium solve with **drift-diffusion only** (no BTBT/TAT/BQP).
2. Re-issue `models` with full physics; `solve prev` warm-starts.
3. Save the full-physics equilibrium structure.
4. Drain ramp in two stages (10 mV / 50 mV).
5. Gate ramp at 25 mV / step under `log master`.
6. Reload equilibrium and run Id-Vd at three V_GS values.

This is the single biggest reliability fix versus the original deck.

## 8. What is **not** in this deck (and why)

| Effect                                | Why omitted                                                              |
|---------------------------------------|--------------------------------------------------------------------------|
| Drain underlap                        | Geometry change; deferred. Suppresses ambipolar BTBT at high V_GS.        |
| `NEGF_MS` / `NEGF_PL1D`               | Different ATLAS/Victory license; useful only for sub-10 nm wires.         |
| `DGLOG` density gradient              | Replaced by BQP - per Silvaco's own recommendation for nanowires.         |
| Phonon-limited mobility tables (`KLA`)| III-V tables not standard in ATLAS 2019; constant + fldmob is preferable. |
| Gate leakage (`FNORD`, direct tunnel) | HfO2 thickness 2 nm is borderline; can be added later if leakage matters. |
| Self-heating                          | Single-V_DS sweep up to 0.5 V; thermal effect <1% in this device.         |


## 9. NEGF Mode-Space - the broken-gap-correct upgrade

**Deck:** `simulations/gaa_iiiv_hj_tfet_negf.in`
**Selected:** `NEGF_MS SCHRODINGER EIGEN=8 ESIZE.NEGF=200 NPRED.NEGF=10`

Why we needed an upgrade beyond the local-Hurkx baseline. The
non-local WKB integrator (`BBT.NONLOCAL`) evaluates the transmission

    T(E) ~ exp(-2 * integral sqrt(2 m (V(x) - E) / hbar^2) dx)

which assumes a forbidden barrier `V(x) - E > 0` along the tunnel path.
For the Type-III GaSb/InAs broken-gap junction the radicand goes
negative (E_V of GaSb sits above E_C of InAs), so ATLAS aborts. NEGF
solves the retarded Green's function

    G^R(E) = [E*I - H - Sigma^R(E)]^(-1)
    T(E)   = Tr(Gamma_S * G^R * Gamma_D * G^A)

with no barrier assumption, so it works for any band alignment. This is
why NEGF is the physically correct tool for this device and BBT is not.

Required model flags and what each one does:
- `NEGF_MS` : mode-space NEGF, lets ATLAS auto-pick coupled (CMS) vs
  uncoupled (UMS) modes. For broken-gap with band mixing, force `NEGF_CMS`
  if you observe the auto-pick choosing UMS and currents look too low.
- `SCHRODINGER` : transverse sub-band solve at each axial slice, required
  by `NEGF_MS`. ATLAS auto-detects geometry; on this cylindrical mesh it
  selects the 1DX solver, which is correct for a nanowire cross-section.
- `EIGEN=8` : keep 8 sub-bands per valley. Adequate for a 5 nm radius
  wire; raise to 12-16 if results look quantization-limited.
- `ESIZE.NEGF=200` : 200 energy grid points. Default may be too coarse
  for the sharp BTBT onset; raise to 400 if Id-Vg looks step-like.
- `NPRED.NEGF=10` : 10 predictor-corrector iterations between Poisson
  and NEGF (default 7). More predictors help when the two are far from
  self-consistent.

The mandatory `carriers=0` requirement. NEGF computes electron and hole
densities itself from the spectral function

    n(r,E) = -(i / 2pi) * [G^< - G^>]

so the standard DD continuity equations are redundant and conflict with
it. Running `carriers=2` (the default) alongside NEGF causes ATLAS to
abort with

    Error in NEGF.
    Set CARRIERS to 0 on the METHOD statement and try again.

The fix is `carriers=0` on every `method` statement issued *after*
NEGF is enabled. This turns off only the continuity solve; Poisson is
still solved and couples to the NEGF charge self-consistently. Stage A
(pure DD ramp to V_D = 0.5 V used as the initial guess for NEGF) keeps
the default `carriers=2` because that stage genuinely needs continuity.

Effective masses used by NEGF / Schrodinger. ATLAS uses its built-in
band-edge masses `mc`, `mhh`, `mlh` (auto-populated from the material
database), not `me.tunnel` / `mh.tunnel`. The values printed by the
solver were checked against literature:

| Material  | mc      | mlh     | mhh    |
|-----------|---------|---------|--------|
| GaSb      | 0.039   | 0.05    | 0.28   |
| InAs      | 0.026   | 0.025   | 0.57   |
| In0.53Ga0.47As | 0.0412 | 0.051 | 0.46 |

Trade-offs vs the local-Hurkx baseline:
- Pros : proper quantum transport, no Kane fitting, handles broken-gap
  natively (so we keep physical chi=4.06, no Type-II shift kludge),
  captures sub-band quantization in the 5 nm wire, and the I_ON should
  reach the realistic uA range reported in Avci 2015.
- Cons : slow (NEGF inverts `(E*I - H - Sigma)` at every energy point at
  every Newton iteration, expect hours per run); convergence may need
  tuning of `NPRED.NEGF` / `ESIZE.NEGF`; effective-mass NEGF is
  single-band, so for publication-grade broken-gap physics k.p or
  full-band atomistic is still preferable.

Fallback if NEGF stays too slow: `DD_MS` - drift-diffusion mode-space.
Same Schrodinger sub-bands but classical transport along the wire,
much cheaper than full Green's-function inversion.



## 10. NEGF run result and the effective-mass single-band limit

The first end-to-end NEGF run (with the `carriers=0` fix in place)
completed all three stages without aborting:

- Stage A : DD ramp converged to V_D = 0.5 V, I_D = 1.72e-14 A.
- Stage B : NEGF self-consistent solve at V_D = 0.5 V completed,
  Schrodinger 1DX cylindrical solver active, structure saved to
  `gaa_iiiv_hj_tfet_negf_btbt_vd05.str`.
- Stage C : full V_G sweep from -0.5 V to +1.0 V at 25 mV step,
  log written to `gaa_iiiv_hj_tfet_negf_idvg.log`.

But the resulting transfer curve is the numerical noise floor:
I_D oscillates symmetrically through zero in the range +/- 3e-21 A,
nine orders of magnitude below the verified local-Hurkx baseline
(I_ON = 2.34 pA). There is no transistor turn-on with V_G.

Three diagnostic features identify this as zero-transmission noise
rather than a too-low but still real current:

1. The current is signed (positive AND negative). Real BTBT current
   is one-sided.
2. There is no monotonic onset with V_G. The waveform is white noise.
3. The magnitude tracks the round-off of the Green's function
   inverter, not any physical scale of the device.

**Why NEGF returned zero transmission.** ATLAS `NEGF_MS` builds the
device Hamiltonian on a per-slice transverse Schrodinger basis using
the conduction-band effective masses. There is no off-diagonal matrix
element coupling the GaSb valence band to the InAs conduction band,
so the retarded Green's function

    G^R(E) = [E*I - H - Sigma^R(E)]^(-1)

returns T(E) = Tr(Gamma_S * G^R * Gamma_D * G^A) = 0 across the
Type-III broken-gap junction. The BTBT process - electron in the
GaSb VB recombining into a state in the InAs CB - has no
representation in single-band effective-mass NEGF.

This is a model limitation, not a parameter-tuning problem. Raising
`NPRED.NEGF` from 10 to 20, or `ESIZE.NEGF` from 200 to 400, or
`EIGEN` from 8 to 16, does not introduce CB-VB coupling. Only an
atomistic or k.p NEGF (Nemo5 / Victory Atomistic / OMEN) reproduces
the matrix element required for broken-gap BTBT. Carrillo-Nunez et
al. 2017 ([arXiv:1705.00909](https://arxiv.org/abs/1705.00909))
is the standard reference for exactly this device family
(InAs/GaSb broken-gap GAA HTFET); they used tight-binding mode-space
NEGF with millions of atoms to match Esaki diode and HTFET
measurements. Effective-mass NEGF, the version available in
ATLAS 2019, is not the right tool for that physics.

**Practical conclusion.** The local-Hurkx + BQP baseline
(`simulations/gaa_iiiv_hj_tfet.in`) remains the production deck for
this device on this license. It treats BTBT as a generation rate
tied into the DD continuity equations, which sidesteps the
single-band coupling problem entirely. The NEGF deck is preserved
in the repository as documentation of the attempt and as the
correct starting point if a Victory Atomistic / Nemo5 license
becomes available.

A short diagnostic deck `simulations/gaa_iiiv_hj_tfet_negf_diag.in`
loads the saved structure file and writes axial and radial band-
edge cutlines so the broken-gap alignment can be verified visually
in tonyplot.

If a "make NEGF produce numbers" path is still wanted, the only
cheap option is a Type-II shift kludge: raise chi(GaSb) from 4.06
to ~4.65 eV. That converts the simulated junction from broken-gap
to staggered, restores a real barrier for the NEGF effective-mass
Hamiltonian to tunnel through, and produces non-zero transmission.
The cost is that the simulated device is no longer the real
GaSb/InAs alignment; V_T will shift by ~50 to 100 meV from the
true broken-gap value. Acceptable as a sensitivity / sanity
check, not as a publication result.
