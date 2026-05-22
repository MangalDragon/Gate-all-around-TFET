# Review of `simulations/original_v9.in`

**TL;DR:** The geometry, mesh, doping, and base material parameters of the
original deck are sound. As a *transfer-curve generator* it has six
structural problems that, together, are why the I_D-V_GS curve will not
look like measured GaSb/InAs broken-gap GAA TFETs.

The corrected deck is in `simulations/gaa_iiiv_hj_tfet.in`. The model
rationale is in `docs/MODELS.md`.

---

## What was right

| Item | Status | Notes |
|------|--------|-------|
| `MESH CYLINDRICAL`, axis at X=0 | OK | Correct GAA cylindrical convention. |
| Region stack along Y axis | OK | GaSb / InAs / InGaAs along the wire. |
| Type-III broken-gap alignment | OK | Affinities (4.06 / 4.90 / 4.50) and Eg (0.726 / 0.354 / 0.74) reproduce the published GaSb/InAs band offset (~150 meV broken). |
| Mesh refinement at Y=0.020 and Y=0.040 | OK | 0.1 nm spacing across the tunnel junction. |
| `QTX.MESH` / `QTY.MESH` rectangle | OK | Encloses source/channel junction. |
| Gaussian source & drain doping | OK | Y.CHAR = 3 nm produces a slightly graded junction, helps tunneling. |
| Aluminum source/drain electrodes + GATE workfunc=4.35 eV | OK | Reasonable starting point for n-type TFET. |

## What was wrong (or generic)

### 1. Local Kane parameters (`bbt.a`, `bbt.b`) left in the material cards

`BBT.NONLOCAL` does not consume `bbt.a` / `bbt.b`. The non-local solver
uses `me.tunnel` / `mh.tunnel` plus the local Eg and affinity to evaluate
the WKB transmission along the tunnel path. Those Kane coefficients are
silently discarded - keeping them in the deck mis-represents the
calibration.

**Fix:** removed from `material` cards in the new deck.

### 2. No trap-assisted tunneling

In a real HfO2 / InAs interface, the subthreshold current is dominated by
phonon-assisted tunneling through interface traps - that is what sets the
SS floor (typically 30-80 mV/dec). With only `BBT.NONLOCAL` enabled, off-state
current is limited by SRH+Auger only and the simulated SS will be
unrealistically steep (often single-digit mV/dec).

**Fix:** added `TAT.NONLOCAL` (uses the same non-local mesh) plus a
finite interface recombination velocity (`s.n=s.p=5e2`).

### 3. No quantum confinement for a 5 nm-radius wire

InAs has m_e* = 0.023; in a 10 nm-diameter wire the lowest sub-band sits
~150-250 meV above bulk Ec. A pure drift-diffusion deck pretends the wire
is bulk and therefore predicts V_T much too low and I_ON much too high.

**Fix:** added `BQP.N` and `BQP.P` (Bohm Quantum Potential), with the
calibrated gamma = 1.4 / alpha = 0.3 from the Silvaco BQP application
note as a starting point for III-V channels.

### 4. Silicon-flavored mobility / BGN

`conmob` (concentration-dependent mobility tables) and `bgn` (Slotboom
band-gap narrowing) are calibrated for silicon. Activating them on
InAs/GaSb/InGaAs causes the simulator to fall back to silicon defaults or
flat behavior - either way the channel mobility used in the
drift-diffusion part of the current is wrong.

**Fix:** dropped `conmob` and `bgn`; specify `mun`/`mup` per material
(InAs 20000/500, GaSb 3000/1000, InGaAs 12000/300 cm^2/V/s); keep
`fldmob` because we already supply `vsatn`/`vsatp`.

### 5. Fragile convergence

The original uses `solve init` directly, with `bbt.nonlocal` already on,
followed by `vstep=0.05` ramps under Newton. BTBT generation has no good
zero-bias initial guess, so Newton routinely fails on the first or second
non-zero bias point.

**Fix in the new deck:**
- Stage A `solve init` is run with **drift-diffusion only** (no BTBT, no TAT,
  no BQP).
- Then `models` is *re-issued* with the full physics turned on, and
  `solve prev` warm-starts from the DD equilibrium.
- `method newton autonr trap maxtrap=30 climit=1e-5 dvmax=0.2` plus a
  `gummel newton` first pass.
- Drain is ramped in two stages: `vstep=0.01` from 0 to 50 mV, then 0.05
  V steps to 0.5 V.

### 6. Single QTREGION with no drain underlap

The QTREGION only covers Y=0.015..0.025 (source side), which is correct.
But there is no **drain** underlap, so at high V_GS the InAs/InGaAs drain
junction will start to BTBT in the local model, polluting the high-V_GS
end of the transfer curve with ambipolar leakage that the deck did not
intend to capture.

**Fix:** keep QTREGION on the source side only (already correct), and
**add a TODO** in the deck: physical fix is to introduce a 2-3 nm undoped
spacer at the drain edge of the channel. The new deck does not change
the geometry to keep the diff small, but it is documented in
`docs/MODELS.md`.

## Minor cleanups

- Removed `ni.fermi` (redundant with `fermi`).
- Set HfO2 permittivity to 22.0 (more representative than 21.0 for ALD HfO2)
  and added Eg / affinity so the dielectric band offset is well defined.
- Added `u.bbt u.srh u.aug` to `output` so generation/recombination
  components can be plotted directly in TonyPlot.
- Renamed output files with a consistent `gaa_iiiv_hj_tfet_*` prefix.

## Verdict

**Original:** generic textbook deck. Will run, but the transfer curve it
produces is not predictive for a III-V GAA HJ TFET.

**Upgraded:** physics-complete for classical drift-diffusion + BTBT +
TAT + quantum-correction. For sub-10 nm wires where ballistic transport
dominates, the next step would be NEGF in Victory Atomistic - that is a
separate license and is mentioned in `docs/MODELS.md`.
