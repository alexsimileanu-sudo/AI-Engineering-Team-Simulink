---
name: matlab-design-hv-charging
description: 'Design and model EV high-voltage charging control in MATLAB/Simulink/Stateflow: CC-CV charge algorithms, DC fast-charge communication state machines (CCS/ISO 15118, CHAdeMO current-demand loops), and BMS state estimation (SOC via Coulomb counting + EKF, SOH via capacity/resistance fade) and cell thermal management. Use when building or reviewing HV charging controllers, charge state machines, BMS estimators, or thermal derating logic. Toolbox-agnostic (base Simulink/Stateflow); does not assume Simscape Battery licensing.'
---

# HV Charging Control & BMS Estimation

## When to Use
- Modeling a CC-CV charge profile or charger/vehicle current-demand control loop
- Building a DC fast-charge (CCS/CHAdeMO) handshake and charge state machine in Stateflow
- Implementing SOC/SOH estimation for a battery management system (BMS)
- Adding thermal-based charge current derating

## 1. CC-CV Charge Algorithm

Two-phase profile, universal across Li-ion chemistries:
1. **Constant Current (CC)**: charge at a fixed current (typically 0.5–1C for AC/onboard, up to 3–4C for DC fast charge) until cell/pack voltage reaches the upper voltage limit (per-chemistry cutoff, e.g. ~4.2 V/cell NMC, ~3.65 V/cell LFP).
2. **Constant Voltage (CV)**: hold pack voltage at the cutoff, allow current to taper naturally as cells approach full SOC.
3. **Termination**: stop when tapered current drops below a threshold (e.g. C/20) or a timeout elapses — always implement both, since a fault (e.g. broken current sensor) can otherwise prevent termination.

Model as a 2-mode state machine (CC → CV → Done) with additional fault modes (overvoltage, overtemperature, comms loss) that always take priority over the nominal CC/CV transitions.

## 2. DC Fast-Charge Communication State Machine

Model the vehicle/EVSE handshake as a Stateflow chart. Structure is similar across CCS (ISO 15118 / DIN 70121) and CHAdeMO, differing mainly in message names/CAN IDs:

```
Idle → Handshake/ParamExchange → CableCheck → Precharge → CurrentDemandLoop → Taper → SessionStop
                                                              │
                                                              ├─ (fault) → WeldingCheck/Fault → Stop
```

- **Handshake**: exchange max voltage/current capability between EVSE and vehicle BMS.
- **Precharge**: EVSE ramps output to match pack voltage before closing main contactors, to avoid inrush/contactor arcing.
- **Current demand loop**: vehicle continuously sends a current request (bounded by cell voltage limits, thermal limits, and SOC-based taper) over CAN; EVSE acknowledges and adjusts; this is a real-time closed loop, not a one-shot handshake — model it as a periodic (e.g. every 250 ms–1 s) request/response transition, with a timeout transition to a fault state if responses stop.
- **Welding/contactor check**: verify contactors opened correctly at session end before signaling connector unlock, to prevent operator shock hazard.
- **Fault states must be reachable from every other state** — always model timeout and out-of-range-value transitions from all charging states, not just the "happy path."

## 3. BMS State Estimation

**SOC (State of Charge)**:
- **Coulomb counting**: SOC(t) = SOC(0) − ∫I·dt / Capacity. Drifts over time from current-sensor bias/quantization error; never use alone for long charge/drive cycles.
- **OCV correction**: at rest (low current), correct drift using the cell's OCV-SOC curve (nonlinear, chemistry-specific, and hysteresis-dependent for LFP).
- **Model-based (EKF/UKF)**: fuse Coulomb counting with an equivalent-circuit model (Rint or Thevenin 1-2 RC-pair model: R0 + R1‖C1 [+ R2‖C2]) via an Extended/Unscented Kalman filter — this is the industry-standard approach because it self-corrects without requiring a full rest period. State vector typically `[SOC, V_RC1, V_RC2]`; measurement is terminal voltage.

**SOH (State of Health)**:
- Capacity fade: SOH_capacity = current full-charge capacity / nameplate capacity, updated from full CC-CV cycle integration.
- Resistance growth: SOH_power = current internal resistance / initial internal resistance (from pulse tests or online R0 estimation) — larger R0 growth reduces available power at a given SOC, independent of capacity fade.
- Track both; a pack can fail power capability well before capacity fade becomes limiting.

## 4. Thermal Management & Derating

- Use a lumped-parameter thermal model per cell/module (thermal mass + conduction/convection resistance to coolant), driven by I²R (Ohmic) heating plus entropic heat terms if higher fidelity is required.
- Derate charge current as a function of both **cell temperature** (upper/lower limits, typically charging disallowed below ~0°C without preheat) and **cell voltage headroom**, taking the minimum of thermal-limited and voltage-limited current at each step.
- Preheat logic: below the minimum charge temperature, request pack preheating (resistive heater or motor-based heating) before allowing CC phase to start — charging a cold Li-ion cell causes lithium plating and irreversible capacity loss.

## Common Pitfalls
- Terminating CC-CV only on a current threshold, with no timeout fallback → hangs forever on a sensor fault.
- No fault-state transitions reachable from every charging state in the Stateflow chart → charger gets stuck if a message is dropped mid-session.
- SOC from Coulomb counting only, no periodic OCV/EKF correction → SOC estimate drifts unbounded over many cycles.
- Charge current derating using only voltage limits, ignoring thermal limits (or vice versa) → thermal runaway risk or unnecessary charge-speed loss.
- Allowing CC phase to start below minimum cell temperature without preheat → irreversible lithium plating damage.

## Verification Checklist
- [ ] CC-CV termination has both a current-taper condition and an independent timeout
- [ ] Charge state machine has fault/timeout transitions reachable from every state, not just terminal states
- [ ] SOC estimator combines Coulomb counting with a model-based correction (OCV or EKF); document expected drift bounds
- [ ] SOH tracks capacity fade and resistance growth as separate metrics
- [ ] Charge current = min(thermal-limited current, voltage-limited current, EVSE-capability current)
- [ ] Preheat logic gates CC start below the chemistry's minimum charge temperature
