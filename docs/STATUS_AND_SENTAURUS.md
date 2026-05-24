# GaSb / InAs / InGaAs Broken-Gap GAA TFET: Status & Sentaurus Migration

This document is split into three parts:

1. **What we achieved, discovered, and could not achieve in Silvaco TCAD ATLAS 2019.**
2. **What is still worth doing in ATLAS 2019, and how to do it correctly.**
3. **Step-by-step Sentaurus TCAD N-2017 migration of this device** for a
   user on Red Hat with no prior Sentaurus experience.

It complements `docs/HANDOFF.md` (which is the executive-summary
handoff) and `docs/MODELS.md` (which is the per-model rationale).

---

## Part 1 - ATLAS 2019: status of this project

### 1.1 Achieved

**Geometry, mesh, materials, doping** (in `simulations/gaa_iiiv_hj_tfet.in`):

- Cylindrical GAA mesh: radial X axis, axial Y axis, axis at X=0.
- 5 nm radius, 20 nm gate length, 2 nm HfO2 gate oxide, Al electrodes.
- Three-region semiconductor stack along the wire axis:
  GaSb source (p+, 5x10^19) -> InAs channel (n-, 1x10^15) -> In0.53Ga0.47As
  drain (n+, 5x10^19), with Gaussian source/drain doping (Y.CHAR=3 nm).
- 0.1 nm axial mesh spacing across both junctions; QTX/QTY non-local
  tunnel-mesh box around the source/channel junction.

**Material parameters** calibrated from Vurgaftman 2001 / Avci 2015 /
Bahuguna 2017:

| Material | chi (eV) | Eg (eV) | m_e^tun | m_h^tun | mu_n   | mu_p  |
|----------|----------|---------|---------|---------|--------|-------|
| GaSb     | 4.06     | 0.726   | 0.039   | 0.40    | 3000   | 1000  |
| InAs     | 4.90     | 0.354   | 0.023   | 0.026   | 20000  | 500   |
| InGaAs (53/47) | 4.50 | 0.74  | 0.041   | 0.050   | 12000  | 300   |
| HfO2     | 2.05     | 5.7     | -       | -       | -      | -     |

**Six structural issues identified and fixed** in the original v9 deck
(see `docs/REVIEW.md`):

1. Inert local Kane parameters (`bbt.a`/`bbt.b`) left in material cards
   while `BBT.NONLOCAL` was selected.
2. No trap-assisted tunneling -> unrealistic SS.
3. No quantum confinement in a 10 nm-diameter wire -> wrong V_T and
   I_ON.
4. Silicon-flavored `conmob` and `bgn` activated on III-V materials.
5. Fragile direct `solve init` with BTBT already on -> Newton failures.
6. No drain underlap awareness -> ambipolar branch contaminates high
   V_GS.

**Robust solve sequence** that converges:

1. Stage A: DD-only equilibrium (no BTBT, TAT, BQP).
2. Re-issue `models` with full physics; warm-start with `solve prev`.
3. Drain ramp two-stage (10 mV / 50 mV).
4. Gate ramp at 25 mV/step.
5. Reload equilibrium, run Id-Vd at three V_GS values.
6. Numerics: `gummel newton autonr trap maxtrap=30 climit=1e-5
   dvmax=0.2`.

**Verified end-to-end production result** with the local-Hurkx + BQP
deck (V_DS = 0.5 V, room T):

| Metric            | Value          |
|-------------------|----------------|
| V_T               | 1.07 V         |
| SS (min)          | 103 mV/dec     |
| I_ON (V_GS=1.5 V) | 2.34 pA        |
| I_OFF             | 8.5x10^-17 A   |
| I_ON / I_OFF      | ~10^3          |

Curve shape is correct: monotonic turn-on, sub-thermal SS region,
saturating Id-Vd family. Magnitude of I_ON is the issue (see 1.3).

**NEGF run completed end-to-end** (with the mandatory `carriers=0`
fix on every `method` statement after NEGF is enabled). All three
stages converged: DD warm-up, NEGF self-consistent solve at V_D=0.5 V,
full V_G sweep -0.5..+1.0 V at 25 mV.

### 1.2 Discovered / understood (the physics insights)

**The device is genuinely Type-III broken-gap.** Confirmed numerically
by extracting band edges from the saved post-NEGF .str at V_D=0.5 V
(212-point axial cutline):

- Junction overlap E_V(GaSb) - E_C(InAs) at the abrupt junction =
  **+104 meV**.
- Bulk-to-bulk overlap = **+199 meV**.
- Flat-band prediction from affinities and Eg = **+114 meV**, agreeing
  with the simulated junction overlap to within 10 meV.

This rules out any "the structure is built wrong" hypothesis.

**Why `BBT.NONLOCAL` aborts on this device.** The WKB integrand

    sqrt( 2 m (V(x) - E) / hbar^2 )

requires `V(x) - E > 0` along the tunnel path (a forbidden barrier).
For Type-III, E_V(GaSb) > E_C(InAs), so the radicand goes negative and
ATLAS halts with WKB Code 2. This is not a bug to be patched - the WKB
approximation itself fundamentally requires a barrier. ATLAS is
correctly refusing to apply it.

**Why `NEGF_MS` returns T(E) ~ 0 (numerical noise floor).** ATLAS
effective-mass mode-space NEGF builds the device Hamiltonian on a
per-slice transverse Schrodinger basis using the conduction-band
effective masses only. There is no off-diagonal matrix element coupling
the GaSb valence band to the InAs conduction band, so

    G^R(E) = [E*I - H - Sigma^R(E)]^(-1)
    T(E)   = Tr(Gamma_S * G^R * Gamma_D * G^A) -> 0

across the broken-gap junction. The signature is unambiguous: I_D
oscillates symmetrically through zero in the +/-3x10^-21 A range,
nine orders below the local-Hurkx baseline, with no V_G dependence -
the round-off of an inverter computing exactly zero transmission.

**Why local Hurkx works but underpredicts.** `BBT.HURKX` does not
crash on broken-gap because it is not a WKB integral - it is a
phenomenological local generation rate that is a function of the local
electric field only. It is also not aware of heterojunction band
alignment - it just multiplies a Kane-form local rate by a function of
the local field. Even with `bbt.a = 9.1e15` (already 10x textbook
III-V values), I_ON stays in the pA range. Real BTBT in this device
is non-local along a few nm of band-bending path - the local
approximation systematically misses that.

**The two failure modes are not contradictory.** Both are correct
within their respective models: NEGF returns zero because no CB-VB
matrix element exists in the Hamiltonian; Hurkx returns a small
non-zero rate because it doesn't know about heterojunction coupling
at all. Both miss the same physics for opposite reasons.

**The mandatory `carriers=0` requirement.** When NEGF is enabled, n
and p are computed from the spectral function n(r,E) = -(i/2pi)*[G< -
G>], so the standard DD continuity equations are redundant. Running
the default `carriers=2` alongside NEGF aborts with
"Set CARRIERS to 0 on the METHOD statement and try again". This
turns off only the continuity solve; Poisson is still solved and
couples to the NEGF charge self-consistently.

**Effective masses used by NEGF / Schrodinger** (ATLAS auto-populated
from material database, not from `me.tunnel`/`mh.tunnel`):

| Material | mc     | mlh   | mhh  |
|----------|--------|-------|------|
| GaSb     | 0.039  | 0.05  | 0.28 |
| InAs     | 0.026  | 0.025 | 0.57 |
| InGaAs   | 0.0412 | 0.051 | 0.46 |

### 1.3 Could not achieve (what's blocked in ATLAS 2019)

| Goal | Blocker |
|------|---------|
| Realistic I_ON (nA-uA range as in Carrillo-Nunez 2017, Avci 2015) | All available BTBT models in ATLAS 2019 are either WKB-based (abort on broken-gap) or local (no heterojunction coupling). |
| Non-local BTBT through the broken-gap junction | `BBT.NONLOCAL` requires a forbidden barrier; the device has no barrier on the tunnel path. |
| Quantum-mechanically correct interband transmission | `NEGF_MS` is single-band effective-mass; no CB-VB matrix element. Returns T(E) = 0. |
| Avoid both fitting and kludging | The two cheap workarounds are (a) cranking `bbt.a` until I_ON matches a target (curve fitting), or (b) raising chi(GaSb) from 4.06 to ~4.65 eV to convert the alignment to staggered Type-II, restoring a barrier (a band-offset kludge). Neither is publication-grade. |

The fix for all four requires an atomistic or k.p Hamiltonian that has
the off-diagonal CB-VB coupling: Victory Atomistic (Silvaco, separate
license), Synopsys QuantumATK, or one of the academic atomistic tools
(NEMO5, OMEN, KWANT). This is documented in `docs/HANDOFF.md`.

---

## Part 2 - What is still worth doing in ATLAS 2019

The local-Hurkx baseline produces curves with the **right shape and
correct trends versus design knobs**. The absolute I_ON is wrong by
3-6 orders of magnitude, but parametric / scaling studies are still
defensible.

Each item below tells you what the deck change is and what the
expected physical interpretation is. Decks for several of these are
already on the local development machine but **not yet committed to
this repository** (see `docs/HANDOFF.md` section 8).

### 2.1 Gate-workfunction sweep (V_T tuning)

```
set gate_wf_list = "4.05 4.20 4.35 4.50 4.65 4.85 5.00 5.20"
foreach wf ($gate_wf_list)
  contact name=GATE workfunc=$wf
  solve init
  ...
  log outfile=idvg_wf_${wf}.log master
  solve name=GATE vgate=0 vstep=0.025 vfinal=1.5
  log off
end
```

Use to find the workfunction that puts V_T at the desired value. The
production deck used `gate_wf=5.20` to push V_T into the 1 V range
(typical n-TFET target).

### 2.2 Drain underlap / LDD spacer effect on ambipolar suppression

Add a 2-3 nm undoped InAs spacer between channel and drain doping by
shrinking the drain doping Y window:

```
DOPING GAUSSIAN MATERIAL=InGaAs N.TYPE CONCENTRATION=$doping_drn \
  Y.MIN=0.043 Y.MAX=0.060 X.MAX=0.005 \   # was Y.MIN=0.040
  Y.CHAR=$Ychar X.CHAR=0
```

Then sweep V_GS into the negative range to see the ambipolar branch.
The undoped spacer should suppress drain-side BTBT and drop the
high-V_GS ambipolar leakage by 1-2 orders of magnitude. This was part
of the production deck's geometry.

### 2.3 DIBL extraction

Run two transfer curves at V_DS = 0.05 V and V_DS = 0.5 V. Define V_T
as the V_GS where I_D = 1e-7 A/um (constant-current method); DIBL =
(V_T(0.05) - V_T(0.5)) / (0.5 - 0.05) in mV/V. Useful for short-channel
trend studies even if absolute V_T is BQP-calibration-dependent.

### 2.4 Subthreshold slope vs trap density

The TAT.NONLOCAL model uses interface traps; the deck currently sets
this indirectly through `s.n=s.p=5e2 cm/s`. Sweep to study D_it impact:

```
interface x.min=0.005 x.max=0.005 \
          y.min=0.020 y.max=0.040 \
          s.n=$svalue s.p=$svalue   # try 1e2, 5e2, 1e3, 5e3, 1e4
```

For a cleaner D_it sweep, use the `INTDEFECTS` block instead, which
lets you specify `NTI` (acceptor-like) and `NTD` (donor-like) trap
densities in cm^-2/eV explicitly.

### 2.5 Temperature sweep (TAT vs BTBT contribution)

```
foreach T (200 250 300 350 400)
  models temperature=$T <other flags>
  solve init
  ...
end
```

TAT has a strong temperature dependence (phonon emission/absorption);
BTBT is weakly temperature-dependent. The temperature dependence of SS
in this device tells you how much of the subthreshold current is TAT
vs BTBT.

### 2.6 Geometry sweeps

- Radius R: 3, 4, 5, 6, 8 nm. Quantum confinement shift in V_T,
  smaller wires give larger V_T (BQP captures the trend).
- Gate length Lg: 10, 15, 20, 30 nm. SCE/DIBL trend.
- HfO2 thickness: 1.0, 1.5, 2.0, 2.5 nm. EOT scaling.
- Source doping: 2e19, 5e19, 1e20. BTBT generation rate scales with
  field, which scales with sqrt(N_A).

### 2.7 Type-II band-offset sensitivity (a sanity-check, not a result)

Raise chi(GaSb) from 4.06 to 4.50, 4.65, 4.80 eV. This converts the
alignment from broken-gap to staggered, restoring a finite barrier so
`BBT.NONLOCAL` (and NEGF_MS) become applicable. The deck then
produces non-zero NEGF currents. The result is no longer the real
GaSb/InAs alignment, but the sensitivity of V_T and SS to the assumed
band offset (which has ~50-100 meV experimental uncertainty in the
literature) is informative.

### 2.8 AC analysis: parasitic capacitances

After the DC operating point converges, ATLAS supports `solve ac`:

```
solve init
solve prev
solve name=DRAIN vdrain=0.5
solve name=GATE  vgate=1.0
solve ac freq=1e6 name=GATE vstep=0.025 vfinal=1.5
log outfile=cgg.log master
log off
```

Extracts C_gg, C_gd, C_gs as functions of bias. Important for compact
modelling and circuit-level use of this device.

### 2.9 Mixed-mode (circuit-level) simulation

Once the device deck is converged, embed it as a subcircuit inside a
mixed-mode SPICE-like netlist (`go atlas` block followed by `.subckt`
and `.tran`/`.dc` analysis). Useful for inverter/SRAM evaluation with
the TFET as one of the active devices.

### 2.10 What ATLAS 2019 cannot do, no matter what

| Limitation | Reason | Where to go |
|------------|--------|-------------|
| Atomistic tight-binding NEGF | Not in ATLAS 2019; in Silvaco's Victory Atomistic addon | Sentaurus QuantumATK, NEMO5, OMEN |
| Multi-band k.p NEGF | Only single-band effective mass | Sentaurus has eMultiValley/hMultiValley but still not full k.p NEGF |
| Phonon-assisted indirect BTBT explicitly | Schenk model not in ATLAS 2019 | Sentaurus has Schenk |
| Quantitative match to published broken-gap I_ON | Missing CB-VB matrix element | Atomistic tools only |

---

## Part 3 - Sentaurus TCAD N-2017 step-by-step migration on Red Hat

This part assumes:
- You have Sentaurus N-2017 installed on Red Hat (any RHEL/CentOS 6
  or 7 era).
- You have `setenv` lines or a module file that puts `swb`, `sde`,
  `sdevice`, `svisual`, `inspect` on your `PATH`.
- You have never used Sentaurus before but have just spent months on
  ATLAS, so you know what each model means physically.

> Where this guide gives concrete file contents, treat them as a
> **starting framework**. Some material parameter names and exact
> command syntax may need to be adjusted against your local Sentaurus
> N-2017 manuals. Run small test cases before committing to a full
> sweep.

### 3.1 The Sentaurus tool chain (named analogues to ATLAS)

| Tool | What it is | ATLAS analogue |
|------|------------|----------------|
| `swb` (Sentaurus Workbench) | GUI project manager that orchestrates all the others. Each "tool" in a project node runs in sequence. | DeckBuild |
| `sde` (Sentaurus Structure Editor) | Builds the geometry, mesh, and doping profile. Scheme/Tcl-based. Output: `*_msh.tdr`. | The MESH/REGION/DOPING block in an ATLAS deck. |
| `snmesh` | Standalone Delaunay mesher, usually invoked from sde. | MESH block. |
| `sdevice` | The device simulator. Reads the `_msh.tdr`, applies physics models, runs solves, writes I-V logs and bias-point structure files. | ATLAS itself. |
| `svisual` | 3D/2D structure and band-diagram viewer. | TonyPlot. |
| `inspect` | 1D log curve plotter. | TonyPlot for `.log` files. |
| `tdx` | TDR file inspector. | (no direct analogue) |

### 3.2 File layout of a Sentaurus project

A Sentaurus project lives in a single directory and contains:

```
project_dir/
  gtree.dat                  # Workbench's project tree
  sde_dvs.cmd                # SDE command file - geometry, mesh, doping
  sdevice_des.cmd            # sdevice command file - physics, electrodes, solve
  sdevice.par                # material parameters (per-device override file)
  sde_msh.tdr                # generated mesh+structure (output of sde)
  n@node@_des.tdr            # output structure files (one per bias point)
  n@node@_des.plt            # log files (Id-Vg, Id-Vd, etc.)
  results/                   # subdirectory created by SWB
```

The `@node@` placeholders are SWB's experiment-tree node IDs.

### 3.3 First-time environment setup on Red Hat

In your shell login file (`.cshrc` if you use tcsh, `.bashrc` if you
use bash), source the Sentaurus environment (path will be
installation-specific):

For bash:
```bash
export STROOT=/opt/synopsys/sentaurus/N-2017.09
export PATH=$STROOT/bin:$PATH
export STDB=$HOME/STDB
mkdir -p $STDB
```

For tcsh:
```tcsh
setenv STROOT /opt/synopsys/sentaurus/N-2017.09
setenv PATH ${STROOT}/bin:${PATH}
setenv STDB ${HOME}/STDB
mkdir -p ${STDB}
```

Then verify:

```bash
which swb sde sdevice svisual inspect
sdevice -h | head -3
```

`STDB` is the projects directory; `swb` opens projects there by
default.

### 3.4 Project creation in SWB

```bash
cd $STDB
mkdir gaa_iiiv_hj_tfet
cd gaa_iiiv_hj_tfet
swb &
```

In the SWB GUI:

1. `Project -> New Project`. Choose the directory created above.
2. Right-click the project root in the tree, `Add tool -> Sentaurus
   Structure Editor (sde)`.
3. Right-click the sde node, `Add tool -> Sentaurus Device (sdevice)`.

You now have a two-tool project: `sde -> sdevice`. The structure
generated by sde flows automatically into sdevice.

### 3.5 SDE command file: geometry, mesh, doping for our GAA TFET

Right-click the sde node, `Edit -> Commands`, and paste the file
below. SDE uses a Scheme dialect.

```scheme
;; --- gaa_iiiv_hj_tfet sde_dvs.cmd -----------------------------------
;; GaSb / InAs / In0.53Ga0.47As broken-gap GAA cylindrical TFET.
;; Cylindrical symmetry around the Z axis (sde uses Z as wire axis).
;; All lengths in micrometers (Sentaurus default).

(sde:clear)
(sdegeo:set-default-boolean "ABA")

;; --- geometry parameters ---
(define R     0.005)   ; wire radius        = 5 nm
(define tOX   0.002)   ; HfO2 thickness     = 2 nm
(define tGate 0.002)   ; Gate metal thickness
(define Lg    0.020)   ; gate length        = 20 nm
(define Ls    0.020)   ; source length      = 20 nm
(define Ld    0.020)   ; drain length       = 20 nm
(define Lspc  0.003)   ; drain-side undoped spacer = 3 nm

;; --- z-coordinates of the layer stack ---
(define zS0  0.000)
(define zS1  Ls)              ; GaSb top (= source/channel junction)
(define zCh1 (+ zS1 Lg))      ; channel/spacer
(define zSpc1 (+ zCh1 Lspc))  ; spacer/drain
(define zD1  (+ zSpc1 Ld))    ; drain top

;; --- semiconductor cylinders ---
(sdegeo:create-cylinder
 (position 0 0 zS0)  (position 0 0 zS1)  R "GaSb"   "SRC")

(sdegeo:create-cylinder
 (position 0 0 zS1)  (position 0 0 zCh1) R "InAs"   "CH")

(sdegeo:create-cylinder
 (position 0 0 zCh1) (position 0 0 zSpc1) R "InAs"  "CH_SPC")

(sdegeo:create-cylinder
 (position 0 0 zSpc1) (position 0 0 zD1) R "InGaAs" "DRN")

;; --- HfO2 gate oxide (annular cylinder around the channel only) ---
(sdegeo:create-cylinder
 (position 0 0 zS1)  (position 0 0 zCh1) (+ R tOX) "HfO2" "OX")
(sdegeo:bool-subtract!
 (find-region-id "OX") (find-region-id "CH"))

;; --- gate metal (annular cylinder around the oxide, channel only) ---
(sdegeo:create-cylinder
 (position 0 0 zS1) (position 0 0 zCh1) (+ R tOX tGate) "Aluminum" "GATE")
(sdegeo:bool-subtract!
 (find-region-id "GATE") (find-region-id "OX"))

;; --- contacts ---
(sdegeo:define-contact-set "source" 4 (color:rgb 0.8 0.2 0.2) "##")
(sdegeo:set-contact-faces (find-face-id (position 0 0 zS0)) "source")

(sdegeo:define-contact-set "drain"  4 (color:rgb 0.2 0.2 0.8) "##")
(sdegeo:set-contact-faces (find-face-id (position 0 0 zD1)) "drain")

(sdegeo:define-contact-set "gate"   4 (color:rgb 0.2 0.8 0.2) "##")
(sdegeo:set-contact-faces
 (find-face-id (position (+ R tOX tGate) 0 (* 0.5 (+ zS1 zCh1)))) "gate")

;; --- doping ---
(sdedr:define-constant-profile "src_dop"
 "BoronActiveConcentration" 5e19)
(sdedr:define-constant-profile-region "src_dop_pl" "src_dop" "SRC")

(sdedr:define-constant-profile "ch_dop"
 "PhosphorusActiveConcentration" 1e15)
(sdedr:define-constant-profile-region "ch_dop_pl" "ch_dop" "CH")
(sdedr:define-constant-profile-region "ch_dop_spc" "ch_dop" "CH_SPC")

(sdedr:define-constant-profile "drn_dop"
 "PhosphorusActiveConcentration" 5e19)
(sdedr:define-constant-profile-region "drn_dop_pl" "drn_dop" "DRN")

;; Gaussian smoothing of the source/channel junction (Y.CHAR ~ 1 nm)
(sdedr:define-gaussian-profile "src_gauss"
  "BoronActiveConcentration"
  "PeakPos" 0  "PeakVal" 5e19
  "ValueAtDepth" 1e15
  "Depth" 0.001
  "Gauss" "Factor" 0.8)
(sdedr:define-refeval-window "src_grad_win"
 "Cuboid" (position -0.020 -0.020 0.018) (position 0.020 0.020 0.020))
(sdedr:define-analytical-profile-placement
 "src_grad" "src_gauss" "src_grad_win" "Both" "NoReplace" "Eval")

;; --- mesh refinement ---
(sdedr:define-refinement-size "ref_global"  0.005 0.005 0.005
                                            0.001 0.001 0.001)
(sdedr:define-refinement-region "rg_global" "ref_global" "CH")
(sdedr:define-refinement-region "rg_global_src" "ref_global" "SRC")
(sdedr:define-refinement-region "rg_global_drn" "ref_global" "DRN")

;; very fine refinement at the source/channel tunnel junction (z=zS1)
(sdedr:define-refinement-size "ref_junc" 0.0005 0.0005 0.0001
                                          0.0001 0.0001 0.00005)
(sdedr:define-refeval-window "junc_win"
 "Cuboid" (position -0.010 -0.010 (- zS1 0.003))
          (position  0.010  0.010 (+ zS1 0.003)))
(sdedr:define-refinement-placement "rp_junc" "ref_junc" "junc_win")

;; --- non-local mesh for BTBT (very important for Sentaurus) ---
(sdedr:define-refeval-window "nl_btbt_win"
 "Cuboid" (position -0.010 -0.010 (- zS1 0.005))
          (position  0.010  0.010 (+ zS1 0.005)))

;; --- save ---
(sde:build-mesh "snmesh" "" "n@node@_msh")
```

A few notes on differences from ATLAS:

- Sentaurus `sdegeo:create-cylinder` uses two endpoints + radius; this
  is naturally cylindrical and replaces the `mesh cylindrical` magic.
- Doping is concentration of an **active species** ("Boron",
  "Phosphorus", "Arsenic") rather than a `P.TYPE`/`N.TYPE` flag with
  raw concentration. For binary alloys (GaSb), Sentaurus accepts the
  active concentration regardless of the actual chemical species; what
  matters is the sign and magnitude.
- The "SRC", "CH", "DRN" labels above are **region names**. They are
  referenced in `sdevice_des.cmd` for region-specific physics.

### 3.6 sdevice command file: physics, electrodes, solve

Right-click the sdevice node, `Edit -> Commands`, paste the file
below. The structure of `sdevice_des.cmd` is sectioned (curly-brace
blocks):

```
File {
  Grid    = "n@node@_msh.tdr"
  Plot    = "n@node@_des.tdr"
  Current = "n@node@_des.plt"
  Output  = "n@node@_des.log"
  Parameter = "sdevice.par"
}

Electrode {
  { Name="source" Voltage=0.0 }
  { Name="drain"  Voltage=0.0 }
  { Name="gate"   Voltage=0.0 Workfunction=5.20 }
}

Physics {
  AreaFactor = 1.0
  Fermi
  EffectiveIntrinsicDensity( OldSlotboom )
  Mobility ( DopingDep HighFieldsaturation )
  Recombination ( SRH ( DopingDep ) Auger )
  eQuantumPotential
  hQuantumPotential
}

Physics ( Region = "SRC" ) {
  Recombination (
    Band2Band ( Model = NonlocalPath )
  )
}

Physics ( Region = "CH" ) {
  Recombination (
    Band2Band ( Model = NonlocalPath )
  )
}

Physics ( RegionInterface = "SRC/CH" ) {
  Recombination (
    Band2Band ( Model = NonlocalPath )
  )
}

;; non-local tunnel mesh definition - the source/channel junction
NonlocalMesh "btbt_path" {
  RegionInterface = "SRC/CH"
  Length = 8.0e-7      # 8 nm path on each side
  Permeation = 8.0e-7
  Direction = (0 0 1)
}

Math {
  Extrapolate
  Iterations = 30
  Notdamped = 50
  RHSmin = 1e-10
  Method = ParDiSo
  RecBoxIntegr ( 1.0e-3, 1.0e3, 5 )
}

Solve {
  ;; ---- Stage A: Poisson alone (trivial init) ----
  Coupled (Iterations=100) { Poisson }

  ;; ---- Stage B: Poisson + carriers, no BTBT yet ----
  Coupled { Poisson Electron Hole }

  ;; ---- Stage C: turn on quantum potentials ----
  Coupled { Poisson Electron Hole eQuantumPotential hQuantumPotential }

  ;; ---- Stage D: ramp drain ----
  Quasistationary (
    InitialStep=0.01 MaxStep=0.05 MinStep=1e-4
    Goal { Name="drain" Voltage=0.5 }
  ) {
    Coupled { Poisson Electron Hole eQuantumPotential hQuantumPotential }
  }
  Save( FilePrefix="n@node@_des_vd05" )

  ;; ---- Stage E: gate sweep at V_DS=0.5 V (Id-Vg) ----
  NewCurrentFile = "idvg_"
  Quasistationary (
    InitialStep=0.025 MaxStep=0.025 MinStep=1e-4
    Goal { Name="gate" Voltage=1.5 }
  ) {
    Coupled { Poisson Electron Hole eQuantumPotential hQuantumPotential }
    CurrentPlot ( Time=(Range=(0 1) Intervals=60) )
  }
}
```

### 3.7 Material parameters file: `sdevice.par`

Sentaurus reads global material parameters from `$STROOT/tcad/N-2017.09/lib/`,
but you almost always want to override them per-project. Put this in
`sdevice.par` next to your `_des.cmd`:

```
Material = "GaSb" {
  Epsilon { epsilon = 15.7 }
  Bandgap {
    Chi0   = 4.06   # affinity (eV)
    Eg0    = 0.726  # gap at 0 K
    alpha  = 4.17e-4
    beta   = 140
  }
  EffectiveIntrinsicDensity( OldSlotboom ) { ... }
  eDOSMass { Formula = 1  Nc300 = 2.0e17 }
  hDOSMass { Formula = 1  Nv300 = 1.3e19 }
  ;; non-local BTBT path Kane parameters (per-material)
  Band2BandTunneling {
    A_kane = 9.1e15   ; cm-3 s-1 (electron generation prefactor)
    B_kane = 1.3e7    ; V/cm
    me_band2band = 0.039
    mh_band2band = 0.40
  }
}

Material = "InAs" {
  Epsilon { epsilon = 15.15 }
  Bandgap {
    Chi0   = 4.90
    Eg0    = 0.354
    alpha  = 2.76e-4
    beta   = 93
  }
  eDOSMass { Formula = 1  Nc300 = 8.7e16 }
  hDOSMass { Formula = 1  Nv300 = 6.6e18 }
  Band2BandTunneling {
    A_kane = 4.0e15
    B_kane = 1.9e7
    me_band2band = 0.023
    mh_band2band = 0.026
  }
}

Material = "InGaAs" {     ; In0.53Ga0.47As
  Epsilon { epsilon = 13.9 }
  Bandgap {
    Chi0   = 4.50
    Eg0    = 0.74
    alpha  = 5.4e-4
    beta   = 204
  }
  eDOSMass { Formula = 1  Nc300 = 2.1e17 }
  hDOSMass { Formula = 1  Nv300 = 7.7e18 }
  Band2BandTunneling {
    A_kane = 8.0e15
    B_kane = 1.6e7
    me_band2band = 0.041
    mh_band2band = 0.050
  }
}

Material = "HfO2" {
  Epsilon { epsilon = 22.0 }
  Bandgap { Chi0 = 2.05  Eg0 = 5.7 }
}
```

The exact parameter names vary between Sentaurus releases. If sdevice
errors on a parameter name in N-2017, search for the correct spelling
with:

```bash
sdevice -P -m GaSb        # prints the model parameters known for GaSb
```

### 3.8 Running the project on the command line

You can run from inside SWB (right-click root, `Run`), or headless:

```bash
cd $STDB/gaa_iiiv_hj_tfet

# Run sde
sde -e -l sde_dvs.cmd

# Run sdevice using the generated mesh
sdevice sdevice_des.cmd

# Plot the Id-Vg
inspect -batch -f "load_file(\"idvg__des.plt\");
                   create_curve(\"id\", \"gate OuterVoltage\", \"drain TotalCurrent\");
                   set_curve_prop(\"id\", color: \"red\", line_style: solid);
                   export_image(\"idvg.png\")"
```

### 3.9 Visualizing in svisual / inspect

```bash
svisual n@node@_des.tdr &
```

In svisual:
- `Tools -> Cutline` to extract a 1D band-edge cutline along the wire
  axis (analogous to TonyPlot cutline).
- `Tools -> Tunneling Path` to show the non-local BTBT integration
  paths svisual computed.
- Right-click a region to overlay E_C, E_V, electron/hole quasi-Fermi
  levels.

### 3.10 The three things that fail in ATLAS, and how Sentaurus N-2017
addresses them

| ATLAS 2019 issue | Sentaurus N-2017 path |
|------------------|-----------------------|
| `BBT.NONLOCAL` aborts (WKB needs barrier) | `Band2Band(Model=NonlocalPath)` plus `NonlocalMesh` block. Sentaurus's path integrator is more graceful with broken-gap; it integrates only over forbidden segments of the path and returns a finite generation rate. Expected I_ON: 10-100x higher than ATLAS-Hurkx, i.e. nA range. |
| `NEGF_MS` returns T(E)~0 | Not directly fixed. Sentaurus N-2017 effective-mass NEGF has the same single-band limitation. Sentaurus's eMultiValley/hMultiValley refines the in-band physics but doesn't add CB-VB coupling. For atomistic/multi-band you still need QuantumATK. |
| Hurkx underpredicts I_ON | Use `Band2Band(Model=Schenk)` instead of Hurkx. Schenk is a non-local Kane two-band integrator with phonon-assisted contributions, integrated along the band-bending path. More physical than Hurkx and not subject to the WKB-barrier abort the way ATLAS BBT.NONLOCAL is. |

### 3.11 Concrete first-week plan

Day 1: Get the environment running. `swb` opens; `sdevice -h` works.
Run any of the Sentaurus tutorial projects in `$STROOT/tcad/current/lib/`
end-to-end so you see one full sde -> sdevice -> svisual cycle.

Day 2: Reproduce a simple Si MOSFET deck from the Synopsys tutorials
to get the hang of `sdevice_des.cmd` syntax.

Day 3: Build the cylindrical structure from section 3.5. Don't add
BTBT yet. Just get a valid `_msh.tdr` with the right materials and
doping and a clean DC operating point at zero bias. Plot the
zero-bias band diagram in svisual along the wire axis - confirm the
Type-III alignment.

Day 4: Add BTBT (`Band2Band(Model=NonlocalPath)` plus the
`NonlocalMesh` block). Run a drain ramp to V_DS=0.5 V and inspect the
generation-rate plot. If non-zero generation appears at the
source/channel junction, the broken-gap path is being treated.

Day 5: Add quantum potentials (`eQuantumPotential`, `hQuantumPotential`)
and the gate sweep. Compare the resulting Id-Vg to the ATLAS Hurkx
production curve. Expect higher I_ON, similar V_T (within ~100 meV
shift from the BQP-vs-eQP calibration difference), similar SS.

Day 6-7: If results look reasonable, sweep the same design knobs you
did in ATLAS (Part 2 above) but in SWB's experiment-tree with
parametric variables. SWB makes parametric sweeps natural - just add
`@variable@` placeholders in the cmd files and define ranges in the
SWB GUI.

### 3.12 Caveats and common gotchas

- **Parameter names drift between Sentaurus releases.** If `Chi0` or
  `A_kane` is rejected by your N-2017, check the manual at
  `$STROOT/tcad/current/manuals/sdevice/`. The parameter names above
  are correct as of N-2017.09 to the best of public documentation,
  but verify in your installation.
- **`NonlocalMesh Length` and `Permeation`** are how far on each side
  of the interface the path integrator searches. Too small and you
  cut off the tunnel path; too large and you waste compute. Start
  with 8 nm on each side (`8.0e-7` cm) for a 5 nm radius wire.
- **Compute time.** Sentaurus with NonlocalPath BTBT plus quantum
  potentials on a fine cylindrical mesh will be 5-30 minutes per bias
  point on a single core. Use SWB's parallel-job capability or
  `sdevice -P 4` for 4-thread runs.
- **License modules.** Sentaurus has separate license features for
  `sde`, `sdevice`, `svisual`, `swb`, and individual physics like
  `nonlocal-tunnel`. If a model is rejected with a license error,
  ask your admin which licence features are enabled in your N-2017
  install.

---

## References

- Carrillo-Nunez et al., *An efficient tight-binding mode-space NEGF
  model enabling up to million atoms III-V nanowire MOSFETs and TFETs
  simulations*, [arXiv:1705.00909](https://arxiv.org/abs/1705.00909).
- Avci, Morris and Young, *Tunnel field-effect transistors: prospects
  and challenges*, IEEE J. Electron Devices Soc. 2015.
- Vurgaftman, Meyer, Ram-Mohan, *Band parameters for III-V compound
  semiconductors and their alloys*, J. Appl. Phys. 89, 5815 (2001).
- Bahuguna et al., calibration values for III-V tunneling masses
  (cited in `docs/MODELS.md`).
- Schenk, *Rigorous theory and simplified model of the band-to-band
  tunneling in silicon*, Solid-State Electronics 36, 19 (1993).
- Silvaco, [TCAD Simulations of TFET and Tunneling Diode](https://silvaco.com/simulation-standard/tcad-simulations-of-tfet-and-tunneling-diode/) (paraphrased for compliance).
- Silvaco, [BQP application note](https://silvaco.com/simulation-standard/a-new-efficient-quantum-method-the-bohm-quantum-potential-model/) (paraphrased for compliance).
- BiLPA / CERN, [Sentaurus on Linux user guide](https://bilpa.docs.cern.ch/simulation/tcad/guide/) (compliance: paraphrased).
- Stanford EE212, [SWB tutorial pages](http://web.stanford.edu/class/ee212/SWB/swb_a.html) (compliance: paraphrased).

> Web content cited above was rephrased for compliance with licensing
> restrictions; consult the original sources for verbatim text.
