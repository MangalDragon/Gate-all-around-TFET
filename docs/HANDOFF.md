# Project Handoff: GaSb/InAs/InGaAs Broken-Gap GAA TFET

## Status Summary

This project simulates a Gate-All-Around III-V heterojunction Tunnel FET
with a Type-III (broken-gap) GaSb/InAs source/channel junction in Silvaco
ATLAS 2019. The investigation has reached a definitive conclusion:

**ATLAS 2019 cannot produce publication-grade BTBT current for this device.**

Three independent model approaches were exhausted:
1. `BBT.NONLOCAL` — aborts with WKB Code 2 (no forbidden barrier in Type-III)
2. `NEGF_MS` — returns T(E)≈0 (no CB-VB coupling in effective-mass Hamiltonian)
3. `BBT.HURKX` — converges but gives I_ON = 2.34 pA (orders of magnitude low)

## Verified Baseline Result

The local-Hurkx production deck produces:
- V_T = 1.07 V (at V_DS = 0.5 V)
- SS = 103 mV/dec
- I_ON = 2.34 pA (at V_GS = 1.5 V, V_DS = 0.5 V)
- I_OFF = 8.5×10⁻¹⁷ A

Published literature for this device family reports I_ON in the nA–µA range.

## Recommended Next Steps

1. **Synopsys Sentaurus N-2017** — try `Schenk` non-local BTBT model + `NonlocalPath`
2. **KWANT** (free, Python) — prototype broken-gap transmission as sanity check
3. **OMEN or QuantumATK** (academic licenses) — for publication-grade atomistic NEGF
4. **Silvaco Victory Atomistic** (2022+) — if license upgrade is feasible

## Key Reference

Carrillo-Nuñez et al., "An efficient tight-binding mode-space NEGF model
enabling up to million atoms III-V nanowire MOSFETs and TFETs simulations"
(2017), arXiv:1705.00909 — canonical study of this exact device family.

## Repository Contents

See docs/MODELS.md for detailed per-model rationale and the full NEGF
investigation record (sections 9 and 10).
