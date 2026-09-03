---
name: matlab-tune-automotive-controllers
description: 'Tune and validate control loops shared across automotive Simulink models: cascaded PID loop design and bandwidth separation, state-space/LQR gain scheduling, and MPC constraint handling for real-time ECU deployment. Applies to motor current/torque loops, HV charging current control, and vehicle stability control. Use when designing, tuning, or reviewing any closed-loop controller in an automotive model, or when checking loop stability margins and real-time feasibility. Toolbox-agnostic (base Simulink/Control System Toolbox concepts); does not assume MPC Toolbox licensing for the core patterns.'
---

# Automotive Controller Tuning Patterns

## When to Use
- Tuning a PID/cascaded control loop (motor current, torque, charge current, vehicle stability)
- Choosing between PID, LQR/state-space, and MPC for a given control problem
- Checking stability margins, bandwidth separation, or real-time feasibility of a controller before deployment
- Reviewing gain-scheduled controllers across an operating envelope (speed, temperature, SOC)

This skill is cross-cutting: pair it with `matlab-design-motor-control`, `matlab-design-hv-charging`, or `matlab-model-vehicle-dynamics` for the plant-specific context.

## 1. Cascaded PID Structure

Automotive control is almost always cascaded (inner loop fast/simple, outer loop slower, sets inner loop's reference):
```
Outer (e.g. torque/speed/charge-voltage) → reference for → Inner (e.g. current)
```
- **Bandwidth separation rule of thumb**: outer loop bandwidth ≤ 1/5 to 1/10 of inner loop bandwidth. Violating this causes the loops to interact and produce oscillation or instability that looks like a "tuning problem" but is actually a structural bandwidth-separation problem.
- **Anti-windup**: mandatory on every PI/PID whose output can saturate (duty cycle, current limit, torque limit) — use back-calculation or clamped integration; without it, saturation during a large step command causes overshoot on recovery.
- **Feedforward/decoupling**: wherever a known disturbance or cross-coupling exists (e.g. dq cross-coupling in FOC, back-EMF feedforward, road-grade feedforward for cruise control), add it explicitly rather than relying on integral action alone — integral-only rejection is slower and can violate the bandwidth-separation assumption under large disturbances.

## 2. State-Space / LQR

Prefer over independent SISO PIDs when the plant has strong cross-coupling (e.g. MIMO dq current dynamics, coupled lateral/yaw states):
- Formulate `ẋ = Ax + Bu`, choose `Q`/`R` weighting to trade state error vs. control effort, solve for `K` via LQR (`lqr` in Control System Toolbox, or hand-derived Riccati solution if toolbox-constrained).
- **Gain scheduling**: automotive plants are rarely LTI across the full operating range (motor inductance vs. saturation current, tire stiffness vs. load, battery resistance vs. temperature/SOC) — schedule gains on the dominant varying parameter (speed, temperature, SOC) rather than using one fixed-point linearization for the whole envelope. Verify smooth gain interpolation (no discontinuous jumps) between schedule breakpoints.

## 3. MPC for Constraint-Heavy Problems

Use MPC when hard constraints must be respected simultaneously with tracking performance and a PID/LQR would require ad-hoc limiting logic — common cases: battery charge current subject to simultaneous voltage/thermal/EVSE-capability limits, torque command subject to simultaneous current/voltage/thermal limits.

- **Horizon and real-time feasibility**: automotive ECUs typically cannot run a long-horizon QP at motor-control rates (100 µs–1 ms); use explicit MPC (precomputed piecewise-affine control law) or a short-horizon (2–10 step) implicit MPC only at slower control rates (charging: 100 ms–1 s; vehicle-level: 10–100 ms).
- Always specify constraints as **hard** (must never be violated, e.g. cell voltage limit) vs. **soft** (penalized but violable, e.g. comfort-related) — conflating the two leads to either infeasible QPs (all hard) or safety-relevant limits being violated (all soft).

## 4. Automotive Control-Loop Rate Reference

| Loop | Typical rate | Notes |
|---|---|---|
| Motor current (FOC inner loop) | 10–20 kHz (matches/exceeds PWM freq /10) | Fastest loop in the system |
| Motor torque/speed (FOC outer loop) | 1–2 kHz | |
| HV charging current-demand loop | 1–10 Hz | Limited by CAN message cadence between vehicle/EVSE |
| BMS SOC/thermal estimation | 1–10 Hz | Slower than charging current loop is acceptable; estimator dynamics are slow |
| Vehicle dynamics / stability control | 100 Hz–1 kHz | |
| Cruise/ADAS longitudinal control | 10–50 Hz | |

## Common Pitfalls
- Tuning an outer loop without checking bandwidth separation from the inner loop it commands.
- Missing anti-windup on any saturating PI/PID, discovered only when a large step command produces overshoot.
- Fixed-point LQR/state-space gains applied across an operating range where the plant is materially nonlinear (no gain scheduling).
- Choosing implicit long-horizon MPC for a control loop whose real-time budget cannot support solving a QP every cycle.
- Not distinguishing hard vs. soft constraints in an MPC formulation, leading to either infeasibility or safety-limit violations.

## Verification Checklist
- [ ] Cascaded loops maintain ≥5–10x bandwidth separation between adjacent loops
- [ ] Every saturating PI/PID has anti-windup
- [ ] Known disturbances/cross-couplings have explicit feedforward, not just integral rejection
- [ ] Gain-scheduled controllers interpolate smoothly between breakpoints; scheduling variable matches the dominant plant nonlinearity
- [ ] MPC (if used) is confirmed solvable within the target loop's real-time budget on the target ECU, and constraints are explicitly classified hard vs. soft
- [ ] Stability margins checked (phase margin, gain margin) at multiple operating points across the relevant envelope, not just nominal
