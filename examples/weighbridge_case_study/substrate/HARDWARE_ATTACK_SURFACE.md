# Physical Hardware Attack Surface, Inline MITM & Defensive Telemetry Topology
Path: core/substrate/HARDWARE_ATTACK_SURFACE.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, The Legal Metrology Act 2009 (Sections 26 & 34), OIML R 76-1

## 1. Physical Weighbridge Threat Taxonomy

In commercial transport logistics (agricultural mandis, scrap yards, mineral depots), financial value is directly proportional to recorded physical mass. Consequently, weighbridge hardware is subject to sophisticated physical and electrical tampering.

[Physical Weighbridge Infrastructure]
│
├──► ATTACK SURFACE 1: Scale Deck & Pit (Analog)
│      ├── Spliced RF Relay Interrupter ("Chipta" / Fob)
│      ├── Junction Box Trimpot Resistor Skimming
│      └── Mechanical Axle Straddling / Ramp Off-Parking
│
├──► ATTACK SURFACE 2: Serial Transmission Line (Digital)
│      ├── Inline Microcontroller RS-232 MITM Spoofer
│      ├── High-Voltage Inductive EMP Injection (Inducing Reset)
│      └── Ground Loop Potential Inversion
│
└──► ATTACK SURFACE 3: Host Processing Terminal (USB / OS)
├── Rogue USB-to-UART Bridge Swapping (Spoofed VID/PID)
├── Optical Display Tinting / Polarized Camouflage
└── Operating System Driver Layer Hooking

---

## 2. Threat Vector Deep-Dive & Failure Physics

### 2.1 Threat Vector A: The RF Relay Interrupter (*Chipta*)
- **Physical Topology:** A miniature radio-frequency receiver module (operating at 315 MHz, 433 MHz, or 2.4 GHz) containing an electro-mechanical or solid-state relay is soldered inside the 4-wire or 6-wire analog load-cell summing junction box or spliced into the main home-run cable.
- **Electrical Mechanics:** When the truck driver or corrupt clerk triggers a pocket key-fob transmitter:
  1. The relay engages, switching a precision metal-film resistor ($10\,\Omega\text{--}100\,\Omega$) in series with the excitation line, or across the bridge signal pair ($+SIG / -SIG$).
  2. The differential voltage drop across the strain-gauge network decreases proportionally.
  3. The indicator digitizes this lower analog voltage, displaying a mass deficit of $500\text{--}5000\,\text{kg}$.
- **Metrological Consequence:** On an inbound gross weighment of wheat, a 2,000 kg reduction cheats the farmer; on an outbound tare weighment of an empty truck, a 2,000 kg inflation cheats the buyer.

### 2.2 Threat Vector B: Inline RS-232 Serial Spoofer
- **Physical Topology:** An inline dongle disguised as a DB9 gender-changer, DB9-to-RJ45 adapter, or cable splice shell placed between the indicator serial port and the Android USB converter.
- **Firmware Mechanics:**
  1. The dongle sniffs continuous ASCII frames (`[STX][Status][Weight][Checksum][CR]`).
  2. When activated via wireless control or an algorithmic trigger (e.g., detecting weights $> 15,000\,\text{kg}$), the internal microcontroller modifies the numeric ASCII string (e.g., changing `024850` to `022850`).
  3. The microcontroller recalculates the valid 8-bit XOR checksum, replaces the tail bytes, and retransmits the frame to the Android host.
- **Metrological Consequence:** The Android terminal receives syntactically perfect, validly checksummed metrological frames containing entirely fabricated weights.

### 2.3 Threat Vector C: Corner-Trimpot Junction Box Manipulation
- **Physical Topology:** Analog summing junction boxes use multi-turn potentiometers on each load-cell channel to normalize corner sensitivity (eccentric loading calibration).
- **Physical Mechanics:** A corrupt technician adjusts the trimpot for Load Cell #1 and #2 (e.g., South Deck), reducing output by 8%, while leaving Load Cells #3 and #4 (North Deck) standard.
- **Operational Exploitation:** The driver is instructed to stop with the heavy cab on the South Deck during inbound weighing, and on the North Deck during outbound weighing, generating systematic weight deficits without external electronic triggers.

---

## 3. Algorithmic Telemetry Defense: The Kinematic Jerk Filter

While software cannot prevent analog voltage attenuation at the load cell, it can detect the **unphysical kinetics** of electronic switching versus mechanical mass transfer.

### 3.1 Kinematic Continuity Invariant
A physical vehicle resting on a multi-tonne structural steel bridge platform constitutes a damped second-order harmonic oscillator:

$$m \frac{d^2 x}{dt^2} + c \frac{dx}{dt} + k x = F(t)$$

- **Mechanical Damping Period:** Any sudden change in deck mass induces platform deflection with an oscillation frequency:
  $$\omega_d = \sqrt{\frac{k}{m} - \left(\frac{c}{2m}\right)^2}$$
- **Settling Dwell:** Mechanical dampeners require a physical decay window of $\tau_{\text{settle}} \ge 1200\text{--}3000\,\text{ms}$. During this window, the indicator emits frames with the **Unstable (US)** status bit.

### 3.2 Detecting Step-Function Relay Discontinuities
When an RF relay fakes a mass drop, the bridge voltage steps discontinuously:
$$\Delta t_{\text{switching}} \le 5\,\text{ms}$$

True Physical Mass Unload (Cargo Dismount / Rolling):
Mass ┼──────────────╮
│              ╰──╮ (Dynamic Damping Ringing: US Flag Active)
│                 ╰───╮  ╭───╮
│                     ╰──╯   ╰─────── (ST Flag Set After 2.5s)
┴─────────────────────────────────────── Time
Adversarial RF Relay Step Discontinuity (Chipta Engaged):
Mass ┼──────────────┐
│              │ (Instant Step Drop: Delta t < 50ms, Zero Ringing)
│              └───────────────────── (Indicator Reports ST Instantly)
┴─────────────────────────────────────── Time

### 3.3 Mathematical Tamper Detection Rule
Let $M_k$ be the mass in integer grams at frame index $k$, received at epoch timestamp $t_k$.
- **Mass Velocity:** $v_k = \frac{M_k - M_{k-1}}{t_k - t_{k-1}}$ (grams/millisecond).
- **Kinematic Jerk Metric:** $J_k = \frac{v_k - v_{k-1}}{t_k - t_{k-1}}$.

$$\mathbf{\text{TAMPER ALARM IF: }} \vert{}M_k - M_{k-1}\vert{} \ge 300{,}000\,\text{g} \quad \text{AND} \quad (t_k - t_{k-1}) \le 120\,\text{ms} \quad \text{AND} \quad \text{Status}_{k} == \text{'ST'}$$

**System Rule:** An instant drop $\ge 300\,\text{kg}$ occurring across a single frame interval while maintaining an unbroken **Stable (ST)** status bit is physically impossible on a verified Class III vehicle deck. The app must immediately trigger `TAMPER_ANOMALOUS_STEP_DISCONTINUITY` and lock the transaction.

---

## 4. Multi-Modal Cross-Verification: Optical Photo-Lock Architecture

To defeat inline RS-232 serial spoofers that generate perfect false ASCII frames, the software enforces **cross-channel sensory redundancy**:

[Weighbridge Event]
│
├── Channel 1: Serial Stream (RS-232 / USB)
│     • Read continuous ASCII payload
│     • Extract: Mass_Serial = 24,850 kg
│
└── Channel 2: Optical Camera Capture (Android Camera2 API)
• Capture high-resolution image of physical indicator dial
• Extract: Mass_Optical via on-device OCR / Visual Hash
• Store raw uncompressed JPEG in forensic evidentiary ledger

### 4.1 Optical Parity Verification Rule
$$\Delta_{\text{metrological}} = \vert{}\text{Mass}_{\text{Serial}} - \text{Mass}_{\text{Optical}}\vert{}$$

1. **Nominal State ($\Delta_{\text{metrological}} \le 20\,\text{kg}$):** Serial and optical channels match. Full integrity established (TIL-3).
2. **Adversarial State ($\Delta_{\text{metrological}} > 50\,\text{kg}$):** 
   - Physical indicator VFD shows `24,850 kg`.
   - Serial frame delivered to app reports `22,850 kg` (inline spoofer decremented 2,000 kg).
   - **System Defense:** Software detects mismatch, logs `INLINE_SERIAL_MITM_DETECTED`, embeds the disparity in the BSA Section 63 certificate, and freezes ticket finalization.

---

## 5. Physical Anti-Tamper & Sealing Verification Protocol

Under Rule 13 and the Seventh Schedule of The Legal Metrology (General) Rules, 2011, legal enforceability requires physical lead-and-wire seals applied by an authorized Inspector of Legal Metrology.

+-------------------------------------------------------------------------+
|                  STATUTORY PHYSICAL SEALING TOPOLOGY                    |
+-------------------------------------------------------------------------+
| [SEAL 1: Indicator Enclosure Lead Seal]                                 |
| Prevents opening the metal casing to access the calibration dip-switch. |
|                                                                         |
| [SEAL 2: Summing Junction Box Wire Seal]                                |
| Encapsulates the IP67 pit box to block trimpot adjustment and RF fobs.  |
|                                                                         |
| [SEAL 3: Home-Run Armor Conduit Seal]                                   |
| Continuous steel braided conduit from pit to cabin; zero exposed wires. |
+-------------------------------------------------------------------------+

### 5.1 Physical Seal Inspection Checklist (Pre-Shift Verification)
Before opening trade operations, the native app requires the operator to confirm physical seal integrity:
- [ ] Seal 1 (Indicator Case): Numbered lead seal intact; wire unbroken.
- [ ] Seal 2 (Junction Box Pit): Pit cover lifted; IP67 box seal wire verified against registered serial number.
- [ ] Conduit Check: No plastic tape, splices, or external dongles visible along the DB9 cable run.

---

## 6. Room Database Schema Extensions

To store hardware attack forensics, peripheral device serials, and tamper alarms, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Hardware Security and Tamper Events
CREATE TABLE IF NOT EXISTS hardware_tamper_events (
    tamper_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    session_uuid TEXT NOT NULL,
    detected_attack_type TEXT CHECK(detected_attack_type IN (
        'ANOMALOUS_STEP_DISCONTINUITY', 
        'INLINE_SERIAL_MITM_DETECTED', 
        'ROGUE_USB_BRIDGE_SUBSTITUTION', 
        'INTER_ARRIVAL_JITTER_ANOMALY', 
        'PHYSICAL_SEAL_BREACH_REPORTED'
    )) NOT NULL,
    confidence_score INTEGER NOT NULL,          -- 0 to 100
    pre_step_mass_grams INTEGER NOT NULL,
    post_step_mass_grams INTEGER NOT NULL,
    delta_interval_ms INTEGER NOT NULL,
    serial_frame_hex TEXT NOT NULL,
    optical_evidence_blob_id TEXT,
    usb_device_vid_pid TEXT NOT NULL,
    usb_hardware_serial TEXT NOT NULL,
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_tamper_time 
    ON hardware_tamper_events(timestamp_epoch_ms);

-- Modify ledger_transactions to anchor hardware attack flags
ALTER TABLE ledger_transactions 
    ADD COLUMN hardware_tamper_flag INTEGER NOT NULL DEFAULT 0;
ALTER TABLE ledger_transactions 
    ADD COLUMN usb_bridge_serial TEXT;

-- Enforce security trigger: Abort finalization if active tamper event is unacknowledged
CREATE TRIGGER IF NOT EXISTS abort_tampered_transaction_insert
BEFORE INSERT ON ledger_transactions
FOR EACH ROW
WHEN NEW.hardware_tamper_flag = 1 AND NEW.integrity_level = 'TIL_3'
BEGIN
    SELECT RAISE(ABORT, 'SECURITY VIOLATION: Cannot commit transaction at TIL-3 while hardware tamper flags are active.');
END;
```


---

## 7. Field Verification Hooks (Gemba Sabotage Protocols)

Field engineers and metrology auditors must execute the following five physical sabotage tests to verify defensive integrity:

1. **Simulated RF Relay Step-Injection Test:**
   - *Procedure:* Place a certified $10,000\,\text{kg}$ test vehicle on the deck. Using an inline test box equipped with a relay-switched resistor, abruptly inject a $1,000\,\text{kg}$ drop in a single frame.
   - *Pass/Fail Criteria:* Verify that the Android app triggers `ANOMALOUS_STEP_DISCONTINUITY` within $< 100\,\text{ms}$, sounds the cabin tamper alarm, and blocks ticket finalization.
2. **The Inline RS-232 MITM Spoofer Test:**
   - *Procedure:* Insert an inline microcontroller between the indicator DB9 output and the tablet USB port. Program the dongle to modify all weights by $-5\%$. Attempt to complete a weighment ticket.
   - *Pass/Fail Criteria:* Capture optical dial photo. Verify that the app detects the discrepancy between optical OCR reading and serial payload, logging `INLINE_SERIAL_MITM_DETECTED`.
3. **USB Serial Bridge Substitution Drill:**
   - *Procedure:* Unplug the registered scale USB cable. Plug in an identical-looking USB-UART cable with a mismatched hardware serial number.
   - *Pass/Fail Criteria:* Verify that the application halts streaming, displays `UNVERIFIED_USB_PERIPHERAL`, and requires supervisor PIN re-authorization.
4. **Physical Summing Box Seal Audit:**
   - *Procedure:* Open the weighbridge pit hatch. Examine the junction box enclosure.
   - *Pass/Fail Criteria:* Check for the presence of official Legal Metrology lead seals. Verify that cable glands are intact and no auxiliary wires exit the enclosure.
5. **Jitter Analysis of Frame Inter-Arrival Times:**
   - *Procedure:* Monitor 1,000 consecutive serial frames at 20 Hz using Android system trace / logcat.
   - *Pass/Fail Criteria:* Compute the standard deviation of inter-arrival times ($\sigma_{\text{jitter}}$). Verify that baseline jitter is $< 2.5\,\text{ms}$. Introduce an inline spoofer and verify that $\sigma_{\text{jitter}}$ increases past the detection threshold ($> 6.0\,\text{ms}$).

---

## 8. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Kinematic Jerk Invariant**: instantaneous step-discontinuities without dynamic mechanical ringing are classified as cyber-physical tamper attacks.
  * Formally mandates **Optical Cross-Channel Parity**: transactions certified at TIL-3 must verify numerical alignment between serialized telemetry and optical camera dial capture.
  * Prohibits unverified USB hardware replacement: locks ingestion to authorized USB-UART hardware serial numbers.
- **Impact on Room Database Schemas:**
  * Adds `hardware_tamper_events` table and associated constraint triggers to prevent committing tickets during active tamper alarms.

---

## 9. Digest Card

- **Key Invariants:** Analog Splices Bypass ADC Logic (RF relays fake true load); Kinetic Continuity Law (Physical vehicles cannot step-drop mass without mechanical ringing); Optical Parity (Serial payload must match photographed dial); Hardware USB Fingerprinting.
- **Attack Surfaces:** (1) Scale Pit RF Relay Fobs (*Chipta*), (2) Inline RS-232 Microcontroller Frame Modifiers, (3) Corner Trimpot De-calibration, (4) Rogue USB Bridge Swapping.
- **Defensive Solutions:** Second-derivative mass velocity filtering ($Jerk > \text{Threshold}$); Dual-channel optical cross-verification; Lead/wire seal statutory chain of custody (Legal Metrology Act S. 26).
- **Top 3 Gemba Hooks:**
  1. Simulated RF step-injection test: verify jerk filter triggers alarm.
  2. Inline serial spoofer test: verify optical vs serial disparity detection.
  3. Pit junction box lead seal audit: verify physical wire seal integrity.
