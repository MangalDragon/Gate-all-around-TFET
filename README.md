# Gate-All-Around III-V Heterojunction TFET (ATLAS 2019)

Cylindrical gate-all-around tunnel FET in Silvaco ATLAS:

- **Source**:  GaSb (p+, 5e19 cm-3) - broken-gap
- **Channel**: InAs (n-, 1e15 cm-3)
- **Drain**:   In0.53Ga0.47As (n+, 5e19 cm-3)
- **Gate ox**: 2 nm HfO2
- **R = 5 nm, Lg = 20 nm**

The GaSb/InAs source/channel is the classic Type-III (broken-gap)
heterojunction that gives the high I_ON of III-V TFETs. The InAs/InGaAs
drain is staggered and helps suppress ambipolar conduction.

## Repository layout

```
simulations/
  gaa_iiiv_hj_tfet.in   <- recommended deck (BBT.NONLOCAL + TAT.NONLOCAL + BQP)
  original_v9.in        <- original deck, preserved for comparison
docs/
  REVIEW.md             <- review of original_v9.in
  MODELS.md             <- model rationale and references
README.md
```

## How to run

```bash
deckbuild -run -ascii simulations/gaa_iiiv_hj_tfet.in
```

Outputs:

| File                                         | Content              |
|----------------------------------------------|----------------------|
| `gaa_iiiv_hj_tfet_eq_DD.str`                 | DD-only equilibrium  |
| `gaa_iiiv_hj_tfet_eq.str`                    | Full-physics equilibrium |
| `gaa_iiiv_hj_tfet_idvg.log`                  | Transfer (Id-Vg @ Vd=0.5) |
| `gaa_iiiv_hj_tfet_on.str`                    | On-state structure   |
| `gaa_iiiv_hj_tfet_idvd_vg{05,10,15}.log`     | Output family        |

## What changed vs. the original

See `docs/REVIEW.md`. Summary:

1. Local Kane parameters `bbt.a`/`bbt.b` removed (inert with non-local model).
2. `TAT.NONLOCAL` added (sets the realistic SS floor).
3. `BQP.N`/`BQP.P` added (quantum confinement in a 10 nm-diameter wire).
4. Si-specific mobility/BGN models replaced with III-V parameters.
5. Robust solve sequence: DD equilibrium -> warm-start full physics -> ramp.
6. Tighter convergence (`climit=1e-5`, `dvmax=0.2`, `gummel+newton+autonr`).

## Expected behavior

At V_DS = 0.5 V, room temperature, the transfer curve should show:

- I_ON in the 100 uA/um - 1 mA/um range (per nominal perimeter)
- I_OFF in the 1e-5 - 1e-3 nA/um range (TAT-limited)
- SS_min in the 30-80 mV/dec range
- V_T around 0.2-0.5 V depending on quantum-correction calibration

If your curve is far from these ranges, the most common culprit is the
BQP gamma/alpha pair - re-tune those against an atomistic or ETB
reference for your specific diameter.
