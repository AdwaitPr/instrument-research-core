# Dynamic Impact, Mechanical Resonance & Damped Strain-Gauge Signal Acquisition
Path: core/substrate/DYNAMIC_IMPACT_TELEMETRY.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/substrate/HARDWARE_ATTACK_SURFACE.md, IS 14331:2010, OIML R 76-1 (Class III Metrology)

## 1. Physical Axle Shock & Weighbridge Structural Dynamics

A commercial vehicle weighbridge is not a static weighing platform. When a 40-tonne multi-axle truck or an un-suspended agricultural tractor mounts the concrete or steel deck at 5–15 km/h, the platform undergoes severe mechanical shock excitation:

```text
       [MOVING AXLE COLLISION AT APPROACH RAMP]
                         │
                         ▼
┌───────────────────────────────────────────────────────┐
│ MECHANICAL FORCING FUNCTION:                           │
│ • Impact Step Impulse: F_impact = m * (g + a_vertical) │
│ • Axle Step Dropping:  Up to 250% Static Load (Peak)   │
└────────────────────────┬───────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────┐
│ WEIGHBRIDGE STRUCTURAL PLATFORM (I-Beam / Concrete):   │
│ • Flexural Deflection (Center Sag)                     │
│ • Low-Frequency Damped Resonance (2.0 – 5.5 Hz)        │
│ • Longitudinal Braking Thrust Force                    │
└────────────────────────┬───────────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────┐
│ ANALOG TRANSDUCER LAYER (4 to 8 Load Cells):           │
│ • Wheatstone Bridge Differential Strain (0 – 20 mV)    │
│ • High-G Shear Spikes & Transverse Side-Loads          │
│ • Rocker Column Restoration Torque                     │
└────────────────────────────────────────────────────────┘
```

### 1.1 The Operational Physical Hostility Profile

- **Axle Drop Impact Spikes:** When a vehicle drops off the approach ramp lip onto the pit-less deck, instantaneous dynamic load exceeds static mass by 180–250%, threatening strain-gauge bonding shear failure.
- **Harmonic Deck Oscillation:** Structural steel decks behave as weakly damped mechanical springs, oscillating at fundamental resonant frequencies of $f_n = 2.0\text{--}5.5\,\text{Hz}$ for $1500\text{--}3500\,\text{ms}$.
- **Braking Thrust & Lateral Shear:** Aggressive air-braking on the scale deck generates horizontal shear forces up to 40% of vertical load, causing load-cell rocker columns to tilt and bind against bumper stops.
- **Thermal Expansion Binding:** Midday solar heating ($> 50^\circ\text{C}$ deck surface) expands steel girders by several millimeters; if longitudinal expansion gap clearance is compromised, structural binding introduces massive non-linear hysteresis.

---

## 2. Mathematical Modeling of Single-Degree-of-Freedom (SDOF) Deck Dynamics

The mechanical interaction between vehicle mass and the weighbridge platform is modeled as a damped second-order dynamic system:

$$M_{\text{eff}} \frac{d^2 x(t)}{dt^2} + C_{\text{deck}} \frac{dx(t)}{dt} + K_{\text{deck}} x(t) = F(t)$$

Where:
- $M_{\text{eff}}$: Combined effective mass (structural deck mass $M_{\text{platform}} + \text{vehicle mass } M_{\text{truck}}$).
- $C_{\text{deck}}$: Equivalent mechanical viscous damping coefficient (structural friction + elastomer mounting pads).
- $K_{\text{deck}}$: Aggregate spring stiffness of the load-cell array ($\sum_{i=1}^{N} k_i$).
- $x(t)$: Vertical deflection at the deck centroid.
- $F(t)$: External dynamic forcing function (axle mounting step function + tire bounce).

### 2.1 Resonant Frequency & Damping Ratio Formulation

The undamped angular natural frequency $\omega_n$ and damping ratio $\zeta$ are defined as:

$$\omega_n = \sqrt{\frac{K_{\text{deck}}}{M_{\text{eff}}}}, \quad \zeta = \frac{C_{\text{deck}}}{2 \sqrt{M_{\text{eff}} K_{\text{deck}}}}$$

Because commercial weighbridge decks are underdamped ($\zeta < 0.25$), the damped natural oscillation frequency is:

$$\omega_d = \omega_n \sqrt{1 - \zeta^2} \approx 2\pi (2.0\text{ to }5.5\,\text{Hz})$$

### 2.2 Analytical Settling Dwell Window

Following an abrupt step loading $F_0 = M_{\text{truck}} g$, the transient vertical displacement response is:

$$x(t) = \frac{F_0}{K_{\text{deck}}} \left[1 - e^{-\zeta \omega_n t} \left(\cos(\omega_d t) + \frac{\zeta}{\sqrt{1 - \zeta^2}} \sin(\omega_d t)\right)\right]$$

To satisfy OIML R 76-1 Class III metrological stability ($\le \pm 0.5e$), the physical oscillation amplitude must decay below the verification scale interval:

$$e^{-\zeta \omega_n t_{\text{settle}}} \le \frac{0.5e}{M_{\text{truck}}}$$

For a 40,000 kg truck ($e = 10\,\text{kg}$, $\zeta \approx 0.08$, $f_n \approx 3.2\,\text{Hz}$):

$$t_{\text{settle}} \ge \frac{-\ln(5 / 40,000)}{0.08 \times 2\pi \times 3.2} \approx \frac{8.98}{1.608} \approx 5.58\text{ seconds}$$

> [!IMPORTANT]
> **System Invariant:** Any digital indicator claiming a legally stable reading (`ST`) in less than 1200 ms after a 30-tonne vehicle stops on the platform is either falsifying stability or bypassing physical settling verification.

---

## 3. The Digital Signal Processing (DSP) Telemetry Pipeline

To extract true metrological mass from noisy dynamic raw load-cell signals without adding excessive queue latency, the instrument implements a dual-stage digital filtering topology:

```text
[RAW ANALOG LOAD-CELL ARRAY] (0 – 20 mV Differential)
               │
               ▼
[24-BIT SIGMA-DELTA ADC] (Raw Telemetry Stream @ 50 Hz / 20 Hz)
               │
               ├── Raw Telemetry Buffer: x[n]
               ▼
[STAGE 1: 2nd-ORDER IIR BUTTERWORTH LOW-PASS FILTER]
               │ Cutoff Frequency: f_c = 1.2 Hz (Attenuates 3-5 Hz mechanical bounce)
               ├── Filtered Telemetry: y[n]
               ▼
[STAGE 2: SLIDING-WINDOW STANDARD DEVIATION DETECTOR]
               │ Window Size: N = 20 samples (1.0 second @ 20 Hz)
               │ Computes: σ = sqrt( (1/N) * Σ (y[n-i] - μ)^2 )
               ▼
┌────────────────────────────────────────────────────────┐
│ METROLOGICAL STABILITY EVALUATOR:                      │
│ • If σ ≤ 0.5 * e  AND  |Δμ| < 0.5 * e  over 1.5s:      │
│     ──► EMIT TRANSITION BIT: 'ST' (STABLE)             │
│ • Else:                                                │
│     ──► EMIT TRANSITION BIT: 'US' (UNSTABLE / MOTION)  │
└────────────────────────────────────────────────────────┘
```

### 3.1 IIR Digital Filter Difference Equation

Stage 1 implements a 2nd-order Infinite Impulse Response (IIR) low-pass filter:

$$y[n] = b_0 x[n] + b_1 x[n-1] + b_2 x[n-2] - a_1 y[n-1] - a_2 y[n-2]$$

For sampling rate $f_s = 20\,\text{Hz}$ and cutoff frequency $f_c = 1.2\,\text{Hz}$, coefficients are:
$$b_0 = 0.0278, \quad b_1 = 0.0556, \quad b_2 = 0.0278$$
$$a_1 = -1.4755, \quad a_2 = 0.5866$$

This provides $>24\,\text{dB}$ attenuation at the $3.2\,\text{Hz}$ mechanical resonant peak while introducing less than $350\,\text{ms}$ group delay.

---

## 4. Transducer Shock Overload & Micro-Crack Health Diagnostics

Repeated vehicle impact spikes cause progressive degradation in strain-gauge load cells: zero-balance drift, bridge resistance unbalance, and micro-cracking in the metal flexure shear web.

### 4.1 Dynamic Overload Shock Classification

- **Nominal Operational Shock ($< 110\%\,\text{F.S.}$):** Standard vehicle mounting; elastomeric bumper pads absorb kinetic energy.
- **Warning Shock ($110\text{--}150\%\,\text{F.S.}$):** Aggressive braking or speeding entry; logs `DYNAMIC_OVERLOAD_WARNING` in health telemetry.
- **Catastrophic Impact Spike ($> 150\%\,\text{F.S.}$):** Exceeds the mechanical yield point of high-alloy tool steel flexures; permanently alters gauge zero balance. Requires immediate diagnostic isolation.

### 4.2 On-Device Transducer Health Metric (Zero-Point Creep Drift)

The software continuously tracks the unloaded platform dead-load zero reading ($Z_0$) between vehicle transactions:

$$\Delta Z_0 = |Z_0(t) - Z_{0,\text{baseline}}|$$

**Health Lock Rule:** If the unloaded scale zero drifts by more than $3.0e$ ($30\,\text{kg}$) within a single 8-hour shift without temperature variation:
1. System flags `TRANSDUCER_CREEP_ANOMALY`.
2. Downgrades metrological trust to `TIL_1`.
3. Mandates corner-balance impedance inspection to identify damaged individual load cells.
---

## 5. Room Database Schema Extensions

To track dynamic impact shock spikes, harmonic settling latencies, and transducer zero-point creep across operational shifts, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Dynamic Shock Spikes & Structural Overloads
CREATE TABLE IF NOT EXISTS loadcell_shock_events (
    shock_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    session_uuid TEXT NOT NULL,
    peak_indicated_mass_kg INTEGER NOT NULL,
    overload_ratio_percent INTEGER NOT NULL,         -- e.g. 145 = 145% of Rated Scale Capacity
    severity_class TEXT CHECK(severity_class IN (
        'NOMINAL', 
        'WARNING_SHOCK', 
        'CATASTROPHIC_IMPACT'
    )) NOT NULL,
    observed_settling_dwell_ms INTEGER NOT NULL,    -- Measured decay time to ST flag
    harmonic_frequency_hz REAL NOT NULL,            -- Extracted fundamental resonant peak
    transducer_health_flag INTEGER NOT NULL DEFAULT 0,
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_shock_time 
    ON loadcell_shock_events(timestamp_epoch_ms);

-- Architectural Extension: Tracking Dead-Load Zero Creep & Elastic Fatigue
CREATE TABLE IF NOT EXISTS transducer_zero_creep_records (
    record_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    transducer_channel_id TEXT NOT NULL,
    dead_load_zero_kg REAL NOT NULL,
    drift_delta_kg REAL NOT NULL,                   -- Current zero minus calibrated baseline
    ambient_deck_temp_celsius REAL,
    creep_status TEXT CHECK(creep_status IN (
        'NOMINAL_STABLE', 
        'CREEP_WARNING', 
        'LOCKOUT_EXCEEDED'
    )) NOT NULL,
    timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Automatically block commercial ticket issuance on catastrophic shock impact
CREATE TRIGGER IF NOT EXISTS trigger_catastrophic_shock_lockout
AFTER INSERT ON loadcell_shock_events
FOR EACH ROW
WHEN NEW.severity_class = 'CATASTROPHIC_IMPACT'
BEGIN
    UPDATE system_operational_state 
    SET is_metrological_lockout_active = 1,
        lockout_reason = 'TRANSDUCER_CATASTROPHIC_DYNAMIC_OVERLOAD';
END;
```

---

## 6. Field Verification Hooks (Gemba Protocols)

Structural vibration dynamics and transducer health must be validated through five field test procedures:

1. **Abrupt Dynamic Braking Shock Assay:**
   - *Procedure:* Drive a loaded 30-tonne test truck onto the weighbridge deck at $10\,\text{km/h}$ and execute a full emergency air-brake stop directly over the scale centroid.
   - *Pass/Fail Criteria:* Verify that the peak shock event is captured in `loadcell_shock_events`, that rocker column restraint stops prevent binding, and that settling dwell time satisfies $t_{\text{settle}} \ge 1200\,\text{ms}$.

2. **DSP Low-Pass Filter Step Response & Group Delay Test:**
   - *Procedure:* Step-load the scale deck with a calibrated 5,000 kg weight block dropped from an overhead hoist onto a rubber damping mat. Log raw ADC stream ($x[n]$) versus filtered output ($y[n]$).
   - *Pass/Fail Criteria:* Verify that $3.2\,\text{Hz}$ deck ringing is attenuated by $\ge 20\,\text{dB}$ within Stage 1 filtering and that total DSP pipeline delay does not exceed $350\,\text{ms}$.

3. **8-Hour Dead-Load Zero Creep Assay:**
   - *Procedure:* Record platform dead-load zero balance at 06:00 AM. Operate the scale through a standard 200-truck day shift. Clear the platform and re-read dead-load zero at 02:00 PM.
   - *Pass/Fail Criteria:* The observed drift $\Delta Z_0$ must satisfy $|\Delta Z_0| \le 1.0e$ ($10\,\text{kg}$). Any drift exceeding $3.0e$ triggers `CREEP_WARNING`.

4. **Thermal Expansion Bumper Stop Clearance Audit:**
   - *Procedure:* Inspect longitudinal and transverse bumper check-gap clearances at peak afternoon sun (14:00 IST, deck surface $> 50^\circ\text{C}$).
   - *Pass/Fail Criteria:* Verify check-gap clearance is between $2.0\,\text{mm}$ and $4.0\,\text{mm}$ on all four corners. Zero clearance indicates structural binding and requires immediate stop adjustment.

5. **Wheatstone Bridge Electrical Isolation & Impedance Drill:**
   - *Procedure:* Disconnect the load-cell home run cable at the summing junction box. Measure excitation input impedance, signal output impedance, and insulation resistance against the steel chassis ground with a 50V insulation tester.
   - *Pass/Fail Criteria:* Insulation resistance must exceed $5,000\,\text{M}\Omega$. Bridge output resistance across all cells must match within $\pm 0.5\,\Omega$.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Kinematic Dwell Invariant**: digital indicators claiming stable readings in $< 1200\,\text{ms}$ after a multi-tonne vehicle mounts the deck are rejected as fraudulent or unverified.
  * Formally mandates **Dual-Stage DSP Low-Pass Filtering**: raw 20 Hz ADC streams must pass through a 2nd-order Butterworth IIR filter ($f_c = 1.2\,\text{Hz}$) and a sliding-window standard deviation detector ($\sigma \le 0.5e$) before transitioning to the `ST` state.
  * Codifies **Transducer Health & Zero Creep Tracking**: dead-load drift $\Delta Z_0 > 3.0e$ enforces automatic metrological trust downgrade to `TIL_1`.
- **Impact on Room Database Schemas:**
  * Adds `loadcell_shock_events` and `transducer_zero_creep_records` tables.
  * Implements `trigger_catastrophic_shock_lockout` database trigger.

---

## 8. Digest Card

- **Key Invariants:** Damped SDOF Harmonic Resonance ($f_n = 2.0\text{--}5.5\,\text{Hz}$); Kinematic Dwell Window ($t_{\text{settle}} \ge 1200\,\text{ms}$); Dual-Stage DSP Pipeline (2nd-order Butterworth IIR $f_c = 1.2\,\text{Hz}$ + Sliding Window $\sigma \le 0.5e$); Zero-Point Creep Limit ($\Delta Z_0 \le 3.0e$).
- **Physical Dynamic Forcing:** Axle impact shock up to 250% static load; longitudinal braking shear up to 40% vertical load; midday thermal steel expansion binding.
- **Transducer Diagnostic Classes:** Nominal ($< 110\%\,\text{F.S.}$), Warning ($110\text{--}150\%\,\text{F.S.}$), Catastrophic Shock ($> 150\%\,\text{F.S.}$, hard lockout).
- **Top 3 Gemba Hooks:**
  1. Measure settling dwell time during emergency braking shock test.
  2. Inspect deck bumper gap clearance ($2.0\text{--}4.0\,\text{mm}$) under peak afternoon sun.
  3. Validate dead-load zero-point creep ($\Delta Z_0 \le 1.0e$) across an 8-hour shift.
