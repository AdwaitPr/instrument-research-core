# Electrical Hostility, Unconditioned Power & Hardware Energy Topologies
Path: core/substrate/ELECTRICAL_HOSTILITY_SPECS.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, OIML R 76-1, CEA Regulations 2010

## 1. The Rural Grid & Generator Electrical Profile

Weighbridges, agricultural mandis, and industrial freight depots operate in electrically aggressive environments characterized by unstable DISCOM feeder lines, unconditioned diesel generators, and missing grounding infrastructure.

### 1.1 The Operational Power Parameter Envelope

| Electrical Parameter | Nominal Standard (Urban) | Rural Indian Yard Environment | Failure Mechanism & Consequence | System Protective Boundary |
| :--- | :--- | :--- | :--- | :--- |
| **Grid AC Voltage ($V_{\text{RMS}}$)** | $230\,\text{V} \pm 6\%$ | **$140\,\text{V}_{\text{RMS}} \text{ to } 300\,\text{V}_{\text{RMS}}$** | Chronic brownouts sag linear regulators; transient overvoltages blow SMPS input caps. | Ingestion halted if $V_{\text{in}} < 160\,\text{V}$ or $> 270\,\text{V}$. |
| **DG Supply Frequency ($f$)** | $50.0\,\text{Hz} \pm 0.5\,\text{Hz}$ | **$45.0\,\text{Hz} \text{ to } 55.0\,\text{Hz}$** | Notch filter mismatch in indicator ADC; introduces $5\text{--}10\text{ digit}$ display oscillation. | Temporal stability window extended to $t \ge 3500\,\text{ms}$ during DG operation. |
| **Neutral-Earth Voltage ($V_{\text{N-E}}$)**| $< 2.0\,\text{V}_{\text{AC}}$ | **$15.0\,\text{V}_{\text{AC}} \text{ to } > 60.0\,\text{V}_{\text{AC}}$** | Floating neutral injects common-mode current across RS-232 signal ground lines. | Mandatory Galvanic Isolation ($2.5\,\text{kV}_{\text{RMS}}$) on serial interface. |
| **Surge / Transient Spikes** | $\le 500\,\text{V}$ peak | **Up to $4.0\,\text{kV}$** (Inductive kicks) | Welder/crane motor inductive back-EMF punches through USB-UART bridge chips. | Bidirectional TVS Diodes (SMBJ6.0CA) across VBUS, D+, D-, TX, RX. |
| **Grid Outage Frequency** | Rare ($< 1/\text{week}$) | **$4\text{ to } 12\text{ outages/day}$** | Frequent mid-transaction power severance; hardware cold reboots. | SQLite WAL mode (`synchronous = FULL`) + zero-recovery state machine. |

---

## 2. Load Cell Excitation, ADC Stability & Brownout Mechanics

### 2.1 The Wheatstone Bridge & Ratiometric Invariance
Commercial weighbridge platforms use 4, 6, or 8 strain-gauge load cells wired in parallel to form a resistive Wheatstone bridge. The differential output voltage $V_{\text{SIG}}$ is given by:

$$V_{\text{SIG}} = V_{\text{EXC}} \times S \times \left( \frac{\text{Mass}}{\text{Capacity}} \right)$$

Where:
- $V_{\text{EXC}}$: Bridge excitation voltage ($5.0\,\text{V}_{\text{DC}}$ or $10.0\,\text{V}_{\text{DC}}$).
- $S$: Load cell sensitivity (typically $2.0\,\text{mV/V}$ at rated capacity).

The indicator measures $V_{\text{SIG}}$ using an internal analog-to-digital converter (ADC). In a **ratiometric configuration**, the ADC reference voltage is derived directly from the excitation voltage ($V_{\text{REF}} = V_{\text{EXC}}$). The digitized output counts $D_{\text{OUT}}$ follow:

$$D_{\text{OUT}} = 2^N \times \frac{V_{\text{SIG}}}{V_{\text{REF}}} = 2^N \times \frac{V_{\text{EXC}} \times S \times \left( \frac{\text{Mass}}{\text{Capacity}} \right)}{V_{\text{EXC}}} = 2^N \times S \times \left( \frac{\text{Mass}}{\text{Capacity}} \right)$$

**The Ratiometric Law:** As long as $V_{\text{EXC}}$ drops identically across both the bridge and the reference input, $V_{\text{EXC}}$ cancels out algebraically. Supply fluctuations do **not** induce measurement drift—**provided the voltage remains above the regulator dropout threshold**.

### 2.2 The Brownout Catastrophe (Regulator Dropout Collapse)
When rural mains drop below $140\,\text{V}_{\text{AC}}$, the indicator's DC power rail drops below the dropout voltage ($V_{\text{dropout}}$) of its analog voltage regulator:

AC Mains Voltage
230V ┼───────────────────────────╮
│                           ╰─────────╮ (Mains Sag < 140V)
120V ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ╰───────────╮ (Regulator Dropout Floor)
0V  ┴────────────────────────────────────────────────────┴───────────────
Regulated V_EXC
5.0V ┼───────────────────────────╮
│                           ╰──────────╮
3.2V ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ╰─────────── (Reference Collapses Non-linearly)
0V  ┴────────────────────────────────────────────────────
Digital Mass Error
+10t  ┼                                                    ╭───────────── (Wild Drift)
0  ┼───────────────────────────┬────────────────────────┴──────────────
│ Normal Operation          │ Ratiometric Tracking   │ BROWNOUT ERROR
│                           │ Preserved              │ (Invalid Data Emitted)

1. **Phase 1 ($V_{\text{AC}} \ge 160\,\text{V}$):** Regulator fully stable; $V_{\text{EXC}} = 5.000\,\text{V}$; Zero measurement error.
2. **Phase 2 ($140\,\text{V} \le V_{\text{AC}} < 160\,\text{V}$):** Linear regulator enters drop-out; $V_{\text{EXC}}$ drops to $4.4\,\text{V}$. Ratiometric tracking holds; minor thermal noise increase; scale remains within legal MPE.
3. **Phase 3 ($V_{\text{AC}} < 140\,\text{V}$):** Analog rail collapses non-linearly; op-amp bias currents starve; ADC reference tracking decouples; digital output drifts wildly ($\Delta \text{Mass} > 500\text{--}2000\,\text{kg}$) while indicator continues emitting ASCII frames with a spurious "Stable" (ST) status bit!

**System Invariant:** The Android ingestion service must track telemetry frame delta velocities. If mass reading accelerates by $> 500\,\text{kg}/\text{frame}$ without an unstable (US) flag, the software must trigger an immediate **Brownout Quarantine Abort**.

---

## 3. The Cold-Boot Zero Calibration Invalidation Invariant

When an indicator power-cycles following a grid cut or generator switchover, industrial scale firmware executes an internal **Power-Up Zero Setting (PUZS)** sequence:

### 3.1 The Stranded Vehicle Trap
1. A loaded truck ($25,000\,\text{kg}$) sits on the platform deck.
2. Grid drops; cabin lights and scale indicator cut out; Android tablet stays alive on battery.
3. Generator starts up 90 seconds later; indicator powers on.
4. Scale firmware executes PUZS: it reads the active bridge voltage (which corresponds to $25,000\,\text{kg}$) and **calibrates that baseline voltage as 0 kg**.
5. The indicator emits serial frames reporting `ST, +000000 kg`.
6. If the software blindly accepts this reading, the transaction commits a $0\,\text{kg}$ gross weight.
7. When the truck leaves the deck, the bridge relaxes to its true tare, and the indicator reports `ST, -25000 kg`.

### 3.2 The Zero-Integrity Recovery Protocol
To prevent committing corrupted baseline weights, the application executes a mandatory verification loop upon serial reconnect:

[SERIAL_STREAM_RESTORED]
│
▼
[IS_MASS_REPORTED_NEAR_ZERO? (± 50 kg)]
├──► NO: Load remains on deck. Latch LIVE mass. Compare with pre-cut snapshot.
│        If delta < 20 kg, resume draft at PERIPHERAL_LOCKED.
│
└──► YES: Potential Stranded Zero Corruption!
│
▼
[WAS_DECK_OCCUPIED_BEFORE_POWER_CUT?]
├──► NO: Deck was truly empty. Accept zero calibration.
│
└──► YES: CRITICAL METROLOGICAL FAULT DETECTED!
• Enforce System Interlock: LOCK SCALE.
• Sound Audible Alarm: "VEHICLE ON SCALE DURING REBOOT".
• Require Driver Egress to Clear Deck.
• Force Cold Zero Reset (Tare Button / Operator Key).

---

## 4. Android USB Host, Charging Topologies & Thermal Degradation

Deploying consumer or semi-industrial Android tablets for 24/7 weighbridge operation creates an electrical conflict between USB Host communication and device charging.

### 4.1 The USB-OTG Charging Dilemma

Micro-USB Standard Host Cable (Dumb OTG):
Tablet [ID Pin] ──► Shorted to GND (0 Ω) ──► Forces Tablet to SOURCE 5V VBUS
──► Internal Battery Drains in 4–8 Hours!
Micro-USB BC 1.2 Accessory Charger Adapter (ACA) Cable:
Tablet [ID Pin] ──► 124 kΩ Resistor to GND ──► Enables Host Mode (Reads Serial)
Power Supply   ──► Injects +5V into VBUS   ──► Sinks Charge to Internal Battery
USB Type-C DRP / PD Host-Powered Y-Splitter:
Tablet [CC Pin] ──► Pull-down (5.1 kΩ)     ──► Negotiates Dual-Role-Data (DRD)
USB-PD Charger ──► Provides 9V/12V Power   ──► Powers Tablet & Peripheral Simultaneously

### 4.2 The High-Temperature Float-Charge Hazard
In rural weighbridge cabins (uninsulated galvanized iron roofs), summer temperatures reach $45\text{--}48^\circ\text{C}$. Continuous float charging under these conditions accelerates chemical breakdown:

| Cell Temperature | Float Voltage | Pouch State at 6 Months | Cycle Life Retained | Safety Hazard |
| :--- | :--- | :--- | :--- | :--- |
| **$25^\circ\text{C}$ (Lab Baseline)** | $4.20\,\text{V}$ ($100\%$ SoC) | Normal ($0\%$ expansion) | $85\%$ | None |
| **$40^\circ\text{C}$ (Warm Room)** | $4.20\,\text{V}$ ($100\%$ SoC) | Minor gas generation ($<5\%$) | $60\%$ | Accelerated capacity fade |
| **$48^\circ\text{C}$ (Mandi Cabin)**| **$4.20\,\text{V}$ ($100\%$ SoC)** | **Severe Pouch Swelling ($>30\%$)**| **$< 25\%$** | **High: Screen delamination, puncture, fire** |
| **$48^\circ\text{C}$ (Mitigated)** | **$4.00\,\text{V}$ ($75\%$ SoC Cap)** | **Stable ($< 5\%$ expansion)** | **$65\%$** | **Controlled: Safe long-term operation** |

**System Invariant:** On supported hardware, the application binds to Android `BatteryManager`. If battery temperature exceeds $42^\circ\text{C}$ ($T_{\text{batt}} > 420$), the terminal issues an audible warning and requests the operator to disconnect AC charging or activate cabin ventilation.

---

## 5. Industrial Interface Hardening: Galvanic Isolation & TVS Protection

Connecting an unprotected mobile phone or tablet directly to an external RS-232 indicator cable spanning 15 meters across a scale pit creates a direct path for lightning-induced ground potentials and inductive spikes.

### 5.1 The Isolated Interface Topology
Weighbridge Scale Deck / Indicator
[Indicator TX] ───────────────┐
[Indicator RX] ────────┐      │
[Scale Ground] ──┐     │      │
│     │      │
=│=││=============================
GALVANIC ISOLATION BARRIER (2.5 kV RMS Isolation Rating)
ADuM1201 / Optocoupler + B0505S-1WR2 Isolated 5V-to-5V DC-DC Converter
=│=││=============================
│     │      │
Android USB Host   │     │      │
[Chassis Ground] ┘     │      │
[USB-UART RX]  ◄───────┘      │
[USB-UART TX]  ───────────────┘
[VBUS (+5V)]   ──► [Bidirectional TVS: SMBJ6.0CA] ──► Android Type-C / OTG Port

### 5.2 Mandatory Protective Components
1. **Bidirectional Transient Voltage Suppressors (TVS):** Placed across $D+/D-$, $V_{\text{BUS}}$, and serial $TX/RX$ data lines. Clamps lightning and static discharge transients ($8/20\,\mu\text{s}$ pulse up to $30\,\text{A}$) to $< 9.2\,\text{V}$.
2. **Optocoupler / Magnetic Digital Isolators:** Breaks physical copper continuity between scale ground and tablet ground, eliminating ground-loop currents induced by $V_{\text{N-E}}$ neutral float.
3. **Common-Mode Chokes:** Suppresses RF interference induced by heavy vehicle alternators and nearby electric sub-stations.

---

## 6. Room Database Schema Extensions

To preserve audit trails of brownouts, power cuts, and thermal throttling events, the database schema records hardware electrical telemetry:

```sql
-- Architectural Extension: Tracking Electrical & Thermal Stress Events
CREATE TABLE IF NOT EXISTS hardware_power_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    session_uuid TEXT NOT NULL,
    event_type TEXT CHECK(event_type IN (
        'MAINS_BROWNOUT', 
        'INDICATOR_POWER_CUT', 
        'RECOVERY_STRANDED_ZERO_DETECTED', 
        'THERMAL_OVERHEAT_WARNING', 
        'BATTERY_DEGRADATION_CRITICAL'
    )) NOT NULL,
    battery_level_percent INTEGER NOT NULL,
    battery_temperature_decicelsius INTEGER NOT NULL, -- e.g., 455 = 45.5 C
    battery_voltage_millivolts INTEGER NOT NULL,       -- e.g., 3950 mV
    is_ac_plugged INTEGER NOT NULL,                   -- 1 = Yes, 0 = No
    prior_gross_mass_grams INTEGER,                   -- Latch mass prior to cut
    recovered_gross_mass_grams INTEGER,               -- Latch mass on reboot
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_power_event_time 
    ON hardware_power_events(timestamp_epoch_ms);

-- Modify ledger_transactions to flag transactions completed under degraded power
ALTER TABLE ledger_transactions 
    ADD COLUMN power_degraded_flag INTEGER NOT NULL DEFAULT 0;
```


---

## 7. Field Verification Hooks (Gemba Protocols)

Engineers and field auditors must execute the following five electrical sabotage and stress protocols at an operational scale terminal:

1. **The Variable AC Variac Brownout Stress Test:**
   - *Procedure:* Power the digital scale indicator through an autotransformer (Variac). Slowly reduce voltage from $230\,\text{V}_{\text{AC}}$ down to $120\,\text{V}_{\text{AC}}$ in steps of $10\,\text{V}$ while a certified $10,000\,\text{kg}$ test mass sits on the deck.
   - *Pass/Fail Criteria:* Verify the exact voltage where the indicator display drifts or cuts out. Verify that the Android application halts weight locking before inaccurate data is emitted.
2. **The Stranded Vehicle Cold-Boot Interlock Test:**
   - *Procedure:* Drive a loaded truck onto the scale platform. Bring transaction to `PERIPHERAL_LOCKED`. Cut the main power switch to the indicator. Wait 30 seconds. Restore power.
   - *Pass/Fail Criteria:* Verify that upon indicator reboot, the Android app detects that mass dropped to near-zero while a draft was open, sounds the audible alarm, and blocks finalization until the scale is vacated and zeroed.
3. **The 8-Hour Continuous Host-Charging Battery Drain Test:**
   - *Procedure:* Connect the Android tablet to the operational USB serial indicator via the deployed OTG/charging cable. Run live streaming for 8 continuous hours.
   - *Pass/Fail Criteria:* Tablet battery charge percentage must not drop below $90\%$ at the end of the 8-hour shift.
4. **Cabin Solar Thermal Chamber Audit:**
   - *Procedure:* Monitor tablet internal battery temperature (via `dumpsys battery`) between 12:00 PM and 3:00 PM inside a tin-shed cabin during summer ($T_{\text{ambient}} > 40^\circ\text{C}$).
   - *Pass/Fail Criteria:* Verify that the application successfully issues high-temperature warnings when $T_{\text{batt}} \ge 43^\circ\text{C}$ and shifts screen brightness to $30\%$ to reduce internal heat generation.
5. **Neutral-to-Earth Voltage Multimeter Pull:**
   - *Procedure:* Using a calibrated digital multimeter set to AC volts, measure potential difference between cabin Neutral and physical metal building structure / earthing rod.
   - *Pass/Fail Criteria:* Record $V_{\text{N-E}}$. If $V_{\text{N-E}} > 10\,\text{V}_{\text{AC}}$, verify that the deployed USB-serial interface utilizes active galvanic isolation.

---

## 8. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Cold-Boot Zero Calibration Invalidation Invariant**: software must reject indicator zero-resets if a vehicle transaction was in-flight prior to peripheral disconnection.
  * Formally mandates **BC 1.2 ACA ($124\,\text{k}\Omega$) or Type-C DRP Hardware**: standard dumb micro-USB OTG cables are explicitly banned from commercial production deployments.
  * Mandates **Battery Thermal Protection Interlocks**: restricts charging alerts and reduces CPU/display load when device temperature exceeds $42^\circ\text{C}$.
- **Impact on Ingestion Pipeline:**
  * Introduces an analog delta-velocity filter in `SerialReaderCoroutine` to discard transient frames caused by power supply dropout sag.

---

## 9. Digest Card

- **Key Invariants:** Ratiometric Bridge Invariance Holds Only Above Regulator Dropout ($V_{\text{AC}} \ge 160\,\text{V}$); Stranded Cold-Boot Zeroing is a Critical Metrological Hazard; Standard Dumb OTG Drains Tablet Batteries; Lithium Pouch Cells Swell Above $45^\circ\text{C}$ Under Float Charge; Galvanic Isolation ($2.5\,\text{kV}$) is Non-Negotiable.
- **Electrical Envelope:** Rural Mains: $140\text{--}300\,\text{V}_{\text{AC}}$; Generator Drift: $45\text{--}55\,\text{Hz}$ ($\text{THD} > 15\%$); Neutral-Earth Float: up to $60\,\text{V}_{\text{AC}}$.
- **Hardware Solutions:** Type-C DRP / BC 1.2 ACA ($124\,\text{k}\Omega$ ID resistor); ADuM1201 digital isolator; SMBJ6.0CA TVS diodes; 80% SoC charge cap.
- **Top 3 Gemba Hooks:**
  1. Brownout Variac test: identify indicator dropout voltage threshold.
  2. Stranded vehicle reboot test: verify rejection of zero-reset with truck on deck.
  3. Neutral-to-Earth voltage pull: check for floating neutral potential.
