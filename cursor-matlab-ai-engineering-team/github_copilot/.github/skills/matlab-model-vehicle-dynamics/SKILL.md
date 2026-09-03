---
name: matlab-model-vehicle-dynamics
description: 'Build and validate vehicle dynamics models in MATLAB/Simulink: longitudinal road-load dynamics, lateral bicycle model (yaw rate/sideslip), Pacejka Magic Formula tire modeling, and quarter-car ride/suspension models. Use when creating or reviewing vehicle dynamics plant models, tire models, handling/stability models, or ride-comfort suspension models. Toolbox-agnostic (base Simulink); does not assume Vehicle Dynamics Blockset licensing.'
---

# Vehicle Dynamics Modeling

## When to Use
- Building a longitudinal (traction/road-load) plant model for energy/performance simulation
- Building a lateral dynamics model (yaw/sideslip) for stability control or ADAS validation
- Selecting or implementing a tire force model
- Modeling ride comfort or suspension behavior (quarter-car)

## 1. Longitudinal Dynamics (Road-Load Equation)

```
F_traction = m·a + Crr·m·g·cos(θ) + 0.5·ρ·Cd·A·v² + m·g·sin(θ)
             [accel]  [rolling resist.]   [aero drag]      [grade]
```
- `Crr` (rolling resistance coefficient): ~0.008–0.015 for passenger tires.
- Combine with wheel rotational dynamics: `J_wheel·dω/dt = T_wheel − F_traction·r_wheel`, and slip ratio `κ = (ω·r − v)/max(v, ω·r, ε)` — always guard the denominator near zero speed to avoid divide-by-zero at standstill.
- Driveline losses: apply gear-ratio and efficiency maps (not a single constant efficiency) for gearbox/differential, since efficiency is torque- and speed-dependent, especially at low load.

## 2. Lateral Dynamics — Linear Bicycle Model

2-DOF single-track model, valid for moderate lateral acceleration (<0.4g) where tire forces stay in the linear region:

State vector `[vy, r]` (lateral velocity, yaw rate), input steering angle δ:
```
m·(v̇y + vx·r) = Fyf + Fyr
Iz·ṙ = a·Fyf − b·Fyr
Fyf = Cf·αf,   Fyr = Cr·αr   (linear cornering stiffness)
αf = δ − (vy + a·r)/vx,   αr = −(vy − b·r)/vx
```
- `Cf`, `Cr`: front/rear cornering stiffness (N/rad), from tire test data or Pacejka linearization at zero slip.
- **Understeer gradient**: `Kus = (m/L)·(b/Cf·L_f... )` — practically, compute via steady-state yaw-rate gain `r/δ` vs. speed; understeer if gain saturates below the neutral-steer value as speed increases, oversteer if it grows unbounded (indicates instability above the critical speed).
- **Critical speed** (oversteer vehicles only): speed above which the linear model becomes unstable — check eigenvalues of the state matrix `A` across the full speed range, not just at one operating point.

## 3. Tire Modeling — Pacejka Magic Formula

Preferred over a purely linear model whenever slip angles/ratios can exceed the linear region (aggressive maneuvers, low-µ surfaces, stability control validation):
```
F(κ or α) = D·sin(C·atan(B·x − E·(B·x − atan(B·x))))
```
- `B` (stiffness factor), `C` (shape factor), `D` (peak value, load-dependent), `E` (curvature factor) — fit from tire test data (e.g. TTC/Calspan) per vertical load `Fz`.
- **Load sensitivity**: `D` scales with `Fz` but saturates (peak friction coefficient decreases at very high load) — a linear `D ∝ Fz` assumption is only valid over a narrow load range; use the full nonlinear load-sensitivity form for weight-transfer-heavy maneuvers (braking, cornering).
- **Combined slip** (longitudinal + lateral simultaneously, e.g. braking-in-a-turn): use the friction-ellipse/combined-slip Pacejka extension rather than superimposing independent longitudinal and lateral formulas — independent superposition can produce force magnitudes exceeding the physical friction limit.
- Always clamp/verify that resultant tire force magnitude never exceeds `µ·Fz`.

## 4. Ride & Suspension — Quarter-Car Model

2-DOF lumped model: sprung mass `ms` (body) on spring `ks`/damper `cs`, over unsprung mass `mu` (wheel/tire) on tire stiffness `kt`:
```
ms·z̈s = −ks(zs−zu) − cs(żs−żu)
mu·z̈u = ks(zs−zu) + cs(żs−żu) − kt(zu−zr)
```
- Typical natural frequencies: sprung mode ~1–1.5 Hz (ride comfort), unsprung mode ~10–15 Hz (road holding/handling) — a well-separated pair of modes is the design target; frequencies too close together couple ride and handling undesirably.
- Use for ride-comfort (ISO 2631 weighted acceleration) or road-holding (dynamic tire load variation) trade-off studies; road input `zr` from measured or synthetic road profiles (ISO 8608 PSD classes A–H).

## Common Pitfalls
- Slip ratio computed without guarding near-zero-speed division → simulation instability/NaN at standstill or low speed.
- Linear bicycle model used at lateral accelerations beyond ~0.4g, where actual tire behavior is already nonlinear → misleading stability conclusions.
- Independent superposition of longitudinal/lateral tire forces during combined-slip maneuvers → nonphysical force exceeding the friction circle.
- Single constant driveline efficiency applied at all torque/speed points → energy model errors, especially at low load/regen.
- Checking lateral-dynamics stability (critical speed) at only one speed instead of across the full operating range.

## Verification Checklist
- [ ] All quantities in consistent SI units; slip ratio and slip angle bounded to physically valid ranges
- [ ] Tire model includes load sensitivity and (if combined-slip maneuvers are in scope) the friction-ellipse combined-slip formulation
- [ ] Lateral model's linear-region validity is checked against expected lateral acceleration range; switch to Pacejka if exceeded
- [ ] State-space eigenvalues checked for stability across the full relevant speed range
- [ ] Driveline efficiency modeled as a torque/speed-dependent map, not a fixed constant
- [ ] Suspension natural frequencies checked for adequate ride/handling mode separation
