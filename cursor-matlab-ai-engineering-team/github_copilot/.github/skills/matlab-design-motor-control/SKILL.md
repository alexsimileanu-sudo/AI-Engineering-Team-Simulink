---
name: matlab-design-motor-control
description: 'Design and model automotive traction inverter control in MATLAB/Simulink: PWM generation (SPWM, SVPWM), dead-time insertion/compensation, switching loss estimation, and field-oriented control (FOC) of PMSM/induction traction motors (Clarke/Park transforms, current-loop PI tuning, MTPA, field weakening). Use when building or reviewing motor control models, inverter PWM stages, torque control loops, or e-drive Simulink models. Toolbox-agnostic (base Simulink/Stateflow); does not assume Motor Control Blockset or Simscape Electrical licensing.'
---

# Automotive Motor Control & PWM Design

## When to Use
- Building or reviewing a traction inverter model (PWM stage, current loops, torque control)
- Choosing between SPWM/SVPWM, setting switching frequency, or adding dead-time
- Implementing FOC for a PMSM or induction motor traction drive
- Reviewing torque-control loop stability, bandwidth separation, or safety monitoring

## Domain Facts (why defaults matter)

| Parameter | Typical automotive range | Notes |
|---|---|---|
| DC bus voltage | 400 V or 800 V | 800V systems halve current for same power → smaller cabling, faster DC charging |
| Switching frequency | 8–20 kHz (Si IGBT), up to 40+ kHz (SiC) | Audible-noise floor ~16-20kHz drives NVH decisions |
| Current-loop bandwidth | 500 Hz–2 kHz | Must be ≥5–10x slower than switching freq to avoid aliasing |
| Torque-loop bandwidth | 50–200 Hz | Outer loop, 5–10x slower than current loop |
| Dead-time | 1–4 µs | Trades shoot-through safety vs. output voltage distortion |

## 1. PWM Generation

**SPWM (sinusoidal, carrier-based)**: compare 3 sinusoidal references (120° apart) against a triangular/sawtooth carrier. Simple, but only achieves ~86.6% of the DC-bus utilization of SVPWM without third-harmonic injection.

**SVPWM (space-vector)**: synthesize the reference voltage vector from the two adjacent active switching vectors + zero vectors within each sector (60° sectors, 6 total). Gives ~15% higher DC-bus utilization than plain SPWM — always prefer SVPWM (or SPWM + 3rd-harmonic injection) for traction inverters unless the model explicitly targets legacy/simplified analysis.

Modeling pattern (base Simulink, no toolbox):
1. Compute reference dq voltages from current controllers → inverse Park to αβ → inverse Clarke or direct sector calculation to get duty cycles.
2. Use a `Repeating Sequence`/counter-based carrier generator or `Simulink PWM Generator` at the target switching frequency; compare against duty cycle to produce raw gate signals.
3. Insert dead-time (see below) before driving switch models or S-Function gate outputs.
4. Sample current feedback synchronously with the PWM carrier (top/bottom of carrier) to avoid ripple-induced measurement error — this is the single most common mistake in home-grown PWM models.

## 2. Dead-Time

Both switches in a leg must never conduct simultaneously (shoot-through). Insert a dead-time gap (1–4 µs) between turning one switch off and the complementary switch on.

- **Effect**: dead-time distorts the average output voltage — a well-known nonlinearity that causes low-order harmonics (5th/7th) and torque ripple at low speed/light load.
- **Compensation**: feedforward voltage compensation based on current polarity (add/subtract a fixed voltage-error term per phase depending on measured/estimated current sign), or model-based compensation using duty-cycle correction ΔD = td·fsw·sign(i_phase).
- **Verification**: check compensated vs. uncompensated phase voltage THD in simulation; dead-time distortion should visibly shrink after compensation is enabled.

## 3. Switching Loss Estimation

Model switching losses as energy-per-switching-event lookup tables (Eon, Eoff, Erecovery vs. current and bus voltage), scaled by switching frequency:
```
P_sw ≈ fsw * (Eon(I,Vbus) + Eoff(I,Vbus) + Erec(I,Vbus))  [summed over 6 switches]
```
Use 2-D lookup tables populated from datasheet curves (not analytical estimates) — datasheet Eon/Eoff already includes parasitic ringing effects that simple RC switching models miss. Combine with conduction loss (I²·Rds_on or Vce_sat·I) for total inverter loss feeding thermal/efficiency models.

## 4. Field-Oriented Control (FOC)

Standard cascaded structure for PMSM/IM traction motors:

1. **Transforms**: Clarke (abc→αβ) then Park (αβ→dq) using measured rotor angle θ (from resolver/encoder or observer). Inverse Park + inverse Clarke (or SVPWM sector logic) to return to abc/PWM duty.
2. **Current loops (inner, fastest)**: PI controllers on Id, Iq with cross-coupling decoupling feedforward terms (ωe·Lq·Iq and ωe·Ld·Id + ωe·λpm) — omitting decoupling is a common source of poor high-speed transient response.
3. **MTPA (Maximum Torque Per Ampere)**: below base speed, compute Id*, Iq* reference split via MTPA lookup table (function of torque command) for salient-pole PMSMs (Ld ≠ Lq) to minimize copper loss for a given torque.
4. **Field weakening**: above base speed, inject negative Id to reduce back-EMF and stay within the voltage limit circle (Id² + Iq² ≤ Imax², and Vd² + Vq² ≤ Vbus_limit²). Implement via voltage-limit feedback (reduce Id reference as modulation index saturates), not a fixed schedule — fixed schedules break under DC-bus sag.
5. **Torque command chain**: torque request → MTPA/field-weakening lookahead → Id*/Iq* → current PI → dq voltage → SVPWM.

## Common Pitfalls
- Sampling current feedback asynchronously to the PWM carrier → aliased ripple in the control loop.
- No dead-time compensation → torque ripple and audible noise at low speed, often misdiagnosed as a control-tuning problem.
- Field weakening implemented as an open-loop speed-based schedule instead of voltage-limit closed loop → torque loss under bus-voltage sag (regen, cold battery).
- Missing anti-windup on current-loop PI when duty saturates → integrator windup causes overshoot on step torque commands.
- Torque-loop bandwidth too close to current-loop bandwidth (<5x separation) → coupled oscillation between loops.

## Verification Checklist
- [ ] Switching frequency, current-loop bandwidth, and torque-loop bandwidth maintain ≥5–10x separation at each stage
- [ ] Dead-time value matches switch datasheet turn-off + margin; compensation demonstrably reduces low-speed THD
- [ ] MTPA/field-weakening respects both current limit circle and voltage limit circle across the full speed range
- [ ] Current controllers include anti-windup and decoupling feedforward
- [ ] Switching/conduction losses use datasheet-derived lookup tables, not idealized analytical formulas
- [ ] Torque monitoring/plausibility check exists if this feeds a safety-relevant (ASIL) torque path — see `safety-compliance-engineer` role for standards coverage
