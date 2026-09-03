---
name: matlab-model-vehicle-aerodynamics
description: 'Model vehicle aerodynamic loads in MATLAB/Simulink for range, energy, and stability studies: drag/downforce coefficient modeling, yaw-angle sensitivity, crosswind composition, and CFD-lite lookup-table surrogates for system-level (non-CFD) simulation. Use when building or reviewing an aerodynamic force model feeding an energy/range model or a vehicle dynamics stability model. Toolbox-agnostic (base Simulink); does not run or replace CFD.'
---

# Vehicle Aerodynamics Modeling (System-Level)

## When to Use
- Adding aerodynamic drag/downforce to a longitudinal energy or range model
- Modeling crosswind/yaw sensitivity for stability or ADAS validation
- Integrating CFD-derived coefficients into a system-level Simulink model without re-running CFD

## 1. Aerodynamic Drag

```
F_drag = 0.5 · ρ · Cd · A · v_rel²
```
- `Cd` (drag coefficient): ~0.20–0.35 for modern passenger EVs (lower is better for range); varies with yaw angle — do not treat `Cd` as constant if crosswind or high-yaw maneuvers matter.
- `A`: frontal area (m²), from vehicle geometry — verify it matches the same reference area used when `Cd` was measured/derived (wind-tunnel vs. CFD conventions can differ).
- `v_rel`: relative airspeed = vehicle ground speed vector minus wind vector, not ground speed alone — always compose wind speed/direction into the relative airspeed before computing drag, particularly for range models validated against real-world (windy) driving data.
- Aerodynamic power: `P_aero = F_drag · v` — feeds directly into the vehicle's total propulsion power/energy model alongside rolling resistance and driveline losses.

## 2. Downforce / Lift

```
F_lift = 0.5 · ρ · Cl · A · v_rel²
```
- `Cl` (lift coefficient, negative = downforce): relevant mainly for performance/sports vehicles; affects tire vertical load `Fz` and therefore max available tire force (see `matlab-model-vehicle-dynamics` tire load sensitivity) — for any performance-handling study, feed `Fz = Fz_static ± F_lift/4` (front/rear split per aero balance) into the tire model rather than assuming constant static load.
- Ground-effect: downforce grows sharply as ride height decreases — if ride height varies materially with load/speed (e.g. active suspension, high-speed squat), a constant `Cl` is not sufficient; use a `Cl(ride_height)` table.

## 3. CFD-Lite Surrogate Modeling for System-Level Simulation

System-level Simulink models should **never re-run full CFD**; instead, ingest CFD/wind-tunnel-derived coefficients as lookup tables:

- **Cd/Cl vs. yaw angle**: 1-D lookup table, since real vehicles show meaningfully different drag at even small yaw angles (2–10°) typical of real-world crosswind driving — using only the zero-yaw `Cd` under-predicts real-world energy consumption.
- **Cd/Cl vs. ride height** (if active aero/suspension is modeled): 1-D or 2-D table (yaw × ride height).
- **Air density correction**: `ρ = f(altitude, temperature, humidity)` via the ISA standard atmosphere model (or measured local conditions) — do not hardcode sea-level `ρ = 1.225 kg/m³` in a range model meant to cover varied climates/altitudes.
- **Lookup table extrapolation**: explicitly set extrapolation behavior (clamp, not linear-extrapolate) for yaw/ride-height tables — CFD data is rarely characterized far outside the tested range, and linear extrapolation can produce nonphysical coefficients.

## 4. Validation

- Validate the assembled model (drag + rolling resistance + driveline) against **coastdown test data**: fit the standard ABC road-load coefficients (`F = A + B·v + C·v²`) from measured deceleration and compare to the model's predicted force at matching speeds — this is the industry-standard cross-check before trusting an aero model for range certification-style claims.
- Cross-check total propulsion power at a few reference speeds (e.g. 50/70/90 km/h constant-speed cruise) against published or measured energy-consumption figures.

## Common Pitfalls
- Treating `Cd`/`Cl` as constant regardless of yaw angle → underestimates real-world drag/energy use in crosswind conditions.
- Using ground speed instead of wind-composed relative airspeed → systematic energy-model bias vs. real-world data.
- Hardcoding sea-level air density in a range model intended for varied climates/altitudes.
- Allowing lookup tables to linearly extrapolate beyond the CFD/wind-tunnel-tested yaw or ride-height range.
- Feeding tire models a constant static vertical load when downforce/lift materially changes it with speed.

## Verification Checklist
- [ ] Drag/downforce use relative airspeed (ground speed composed with wind vector), not ground speed alone
- [ ] Cd/Cl modeled as a function of yaw angle (and ride height, if active aero is in scope), not a single constant
- [ ] Air density computed from altitude/temperature rather than hardcoded sea-level value
- [ ] Lookup tables clamp rather than extrapolate beyond characterized range
- [ ] Assembled road-load model cross-checked against coastdown (ABC coefficient) test data
