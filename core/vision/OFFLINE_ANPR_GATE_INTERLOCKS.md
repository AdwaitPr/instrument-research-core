# Offline ANPR, Optical Axle Count & Entry/Exit Gate Interlocks
Path: core/vision/OFFLINE_ANPR_GATE_INTERLOCKS.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/substrate/HARDWARE_ATTACK_SURFACE.md, core/logistics/FLEET_IDENTITY_FASTPATH.md, Central Motor Vehicles Rules (CMVR) Rule 50, ISO/IEC 14443

## 1. Physical Optical Hostility, Dirt/Mud Occlusion & Plate Edge Cases

In agricultural mandis, cement works, and mining weighbridge gates, optical automated number plate recognition (ANPR) operates in severe physical environments where consumer computer vision models fail:

```text
                  ┌────────────────────────────────────────┐
                  │       INBOUND VEHICLE APPROACH         │
                  └───────────────────┬────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│ PHYSICAL SURFACE │        │ ILLUMINATION &   │        │ GEOMETRIC &      │
│ OCCLUSIONS       │        │ ATMOSPHERIC NOISE│        │ MOUNTING DISTORT │
├──────────────────┤        ├──────────────────┤        ├──────────────────┤
│ Hostility:       │        │ Hostility:       │        │ Hostility:       │
│ • Dried red mud  │        │ • High-beam glare│        │ • Bent/dented HSRP│
│ • Coal/grain dust│        │ • Infrared bounce│        │ • 35° yaw angle  │
│ • Loose wire ties│        │ • Grain chaff fog│        │ • Bumper sag     │
│ • Hand-painted   │        │ • Extreme midday │        │ • Tailgate chain │
│   Devanagari font│        │   shadow contrasts│       │   obstruction    │
└────────┬─────────┘        └────────┬─────────┘        └────────┬─────────┘
         │                           │                           │
         └───────────────────────────┼───────────────────────────┘
                                     │
                                     ▼
                        ┌──────────────────────────┐
                        │ ON-DEVICE NPU PIPELINE   │
                        │ Local INT8 Inference     │
                        │ τ_infer ≤ 120 ms         │
                        │ Zero Cloud Dependencies  │
                        └──────────────────────────┘
```

### 1.1 Structural Classes of Indian Vehicle License Plates

To achieve automated gate clearance without human intervention, the optical inference engine normalizes five plate typologies:

- **High Security Registration Plates (HSRP):** Laser-etched chromium hologram, hot-stamped black foil lettering with blue "IND" legend, standardized font (mandated under CMVR Rule 50).
- **Commercial Yellow Plates:** Black alphanumeric characters embossed on retro-reflective yellow substrate (`#FDD835`).
- **Electric Commercial Plates:** White lettering embossed on retro-reflective green substrate (`#2E7D32`).
- **Non-Standard Hand-Painted Plates:** Irregular stroke widths, regional artistic ligatures, and uneven character kerning common on rural tractor trolleys.
- **Damaged / Splattered Plates:** Occluded by road tar, cow dung, agricultural twine, or physical bending from towing collisions.

---

## 2. On-Device Lightweight Inference Pipeline (Zero-Cloud Embedded Vision)

Because rural yards suffer recurring internet blackouts (Aspect 19), sending high-resolution video streams to cloud vision APIs causes instant queue gridlock. The entire pipeline executes locally on an embedded Neural Processing Unit (NPU) or on-device GPU:

```text
[CAMERA RTSP VIDEO STREAM] (1080p @ 15 FPS / Global Shutter Infrared)
              │
              ▼
[FRAME PRE-PROCESSOR & MOTION ROI CROPPING]
              │ Triggers on Ground Loop or Frame Difference
              ▼
[STAGE 1: BOUNDING BOX DETECTOR (YOLO-v8n-INT8)]
              │ Input: 640x640x3 INT8 Quantized Tensor
              │ Output: Plate Bounding Box [x, y, w, h] + Confidence Score
              ▼
[PERSPECTIVE RECTIFICATION (HOMOGRAPHY WARP)]
              │ Warps skewed 35° oblique capture to planar rectangular 256x64 patch
              ▼
[STAGE 2: ALPHANUMERIC SEQUENCE RECOGNIZER (CRNN + CTC BEAM SEARCH)]
              │ Convolutional Feature Extractor + BiLSTM Recurrent Slices
              │ Connectionist Temporal Classification (CTC) Decoding
              ▼
┌────────────────────────────────────────────────────────┐
│ NORMALIZED CANONICAL OUTPUT:                           │
│ • String: "UP32BN4521"                                 │
│ • Confidence: C_ocr ≥ 0.88                             │
│ • Execution Budget: τ_infer ≤ 110 ms on Edge NPU       │
└────────────────────────────────────────────────────────┘
```

### 2.1 Quantized Neural Model Specifications

- **Detector:** YOLO-v8 nano quantized to symmetric 8-bit integer weights (INT8), pruned to eliminate non-vehicular anchor heads. Model weight size: $\le 6.2\,\text{MB}$.
- **Character Recognizer:** Depthwise separable CRNN architecture outputting character probability distribution over standard uppercase Latin alphabet (A–Z), Arabic numerals (0–9), and special token space. Model size: $\le 4.8\,\text{MB}$.
- **Synthetic Mud Augmentation:** Training incorporates aggressive adversarial noise transforms (CutMix, synthetic mud splatters, salt-and-pepper sensor noise, and horizontal smear lines).

---

## 3. Optical Axle Profiling & Vehicle Classification Interlock

To prevent freight class misdeclaration (e.g., claiming a 10-wheel tipper is a 6-wheel rigid truck to manipulate baseline tare as modeled in Aspect 06), the entry ramp couples license plate recognition with Automated Side-Profile Axle Counting.

```text
┌─────────────────────────────────────────────────────────────────┐
│              OPTICAL AXLE PROFILE DETECTION CURTAIN             │
├─────────────────────┬───────────────────────────────────────────┤
│ DETECTOR SUBSYSTEM  │ PHYSICAL SENSING METHOD & ENVELOPE        │
├─────────────────────┼───────────────────────────────────────────┤
│ 1. Optical Curtain  │ Dual infrared thru-beam emitter arrays    │
│                     │ spaced 300 mm apart at 400 mm wheel hub   │
│                     │ height across scale entry ramp.           │
├─────────────────────┼───────────────────────────────────────────┤
│ 2. Side-View Camera │ Wide-angle optical lens capturing side    │
│                     │ silhouette; runs wheel hub circle Hough   │
│                     │ transform and contour feature detection.  │
├─────────────────────┼───────────────────────────────────────────┤
│ 3. Axle Classifier  │ Correlates wheel hub pulse count with     │
│                     │ registered fleet chassis configuration.   │
└─────────────────────┴───────────────────────────────────────────┘
```

### 3.1 Mathematical Correlation with Weighbridge Dynamic Step Profile

When the vehicle rolls across the approach ramp onto the live weighbridge platform, the optical wheel count $N_{\text{optical}}$ must strictly match the discrete strain-gauge dynamic jerk count $N_{\text{jerk}}$ (derived from Aspect 04):

$$\text{INTERLOCK INVARIANT: } N_{\text{optical}} = N_{\text{jerk}} = N_{\text{registered\_axles}}$$

If the optical sensor detects 5 axles (10 wheels) while the operator selected a 2 axle profile, the software blocks weighing transition $T_2$ and raises `AXLE_COUNT_CLASS_MISMATCH`.

---

## 4. Physical Barrier Gate Interlocks & Anti-Rollback State Logic

The interface directly controls physical hardware perimeter barriers (automatic boom barriers, red/green traffic LED signal lights, and audible ramp sirens) through isolated industrial digital I/O relays.

### 4.1 Gate State Transition Sequence

To prevent premature vehicle departure or tailgating fraud:

```text
       [BARRIER GATE: LOCKED DOWN (RED LIGHT)]
                         │
                         ├── 1. Vehicle Mounts Scale & Vehicle ID Authenticated
                         ├── 2. Both Axles Completely Within Platform Bounds
                         ├── 3. Metrological Stability Achieved: State 'ST'
                         ├── 4. Transition T4 Fires: Certified Weight Locked
                         ├── 5. Physical Ticket Printed (or Digital Receipt Signed)
                         ▼
       [RELAY TRIGGER: PULSE GPIO_BARRIER_OPEN]
                         │
                         ├── Boom Barrier Elevates to 90°
                         ├── Signal Light Transitions from RED to GREEN
                         ├── Vehicle Commences Departure Off Platform
                         ▼
       [EXIT GROUND LOOP DETECTOR FIRES]
                         │
                         ├── Inductive Loop Detects Vehicle Off Platform
                         ├── Platform Net Mass Returns to Dead Zero (≤ 0.5 * e)
                         ▼
       [RELAY TRIGGER: PULSE GPIO_BARRIER_CLOSE]
                         │
                         ├── Boom Barrier Drops to 0°
                         ├── Signal Light Returns to RED
                         └── Scale Ready for Next Transaction
```

### 4.2 Anti-Rollback & Anti-Tailgating Rules

- **Anti-Rollback Interlock:** If a vehicle begins rolling backward off the ramp before transition $T_4$ locks, the platform weight derivative flips negative ($dM/dt < -500\,\text{kg/s}$). The software aborts the transaction immediately, forces an indicator reset, and keeps the exit barrier locked down.
- **Anti-Tailgating Curtain:** The exit boom barrier will not trigger its close pulse until the optical light curtain confirms the departing vehicle's rear bumper has cleared the mechanical swing envelope, preventing barrier strikes while blocking a trailing vehicle from mounting without zero re-calibration.
---

## 5. Room Database Schema Extensions

To persist on-device ANPR inferencing logs, optical axle counts, and barrier relay transitions, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking On-Device Optical ANPR Inferences
CREATE TABLE IF NOT EXISTS anpr_optical_captures (
    capture_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    transaction_id INTEGER REFERENCES ledger_transactions(ledger_id),
    recognized_registration TEXT NOT NULL,
    ocr_confidence_score REAL NOT NULL,             -- e.g. 0.94
    inference_latency_ms INTEGER NOT NULL,          -- Must satisfy <= 120 ms
    plate_structural_type TEXT CHECK(plate_structural_type IN (
        'HSRP_STANDARD', 
        'COMMERCIAL_YELLOW', 
        'EV_GREEN', 
        'HAND_PAINTED_RURAL', 
        'DAMAGED_OCCLUDED'
    )) NOT NULL,
    optical_patch_blob_id TEXT NOT NULL,           -- Local image thumbnail hash
    homography_rectified INTEGER NOT NULL DEFAULT 1,
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_anpr_reg 
    ON anpr_optical_captures(recognized_registration);

-- Architectural Extension: Tracking Gate Interlock & Barrier Actuations
CREATE TABLE IF NOT EXISTS gate_interlock_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    vehicle_registration TEXT NOT NULL,
    optical_axle_count INTEGER NOT NULL,
    dynamic_jerk_axle_count INTEGER NOT NULL,
    axle_match_status TEXT CHECK(axle_match_status IN ('MATCHED', 'MISMATCH_DETECTED')) NOT NULL,
    barrier_actuation_type TEXT CHECK(barrier_actuation_type IN (
        'BARRIER_OPEN_NOMINAL', 
        'BARRIER_FORCE_LOCKED_MISMATCH', 
        'ANTI_ROLLBACK_ABORT_LATCH', 
        'TAILGATING_TRIP_LATCH'
    )) NOT NULL,
    rollback_detected INTEGER NOT NULL DEFAULT 0,
    timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Automatically lock metrological state if optical and dynamic axle counts diverge
CREATE TRIGGER IF NOT EXISTS trigger_axle_mismatch_lockdown
AFTER INSERT ON gate_interlock_events
FOR EACH ROW
WHEN NEW.axle_match_status = 'MISMATCH_DETECTED'
BEGIN
    UPDATE system_operational_state 
    SET is_metrological_lockout_active = 1,
        lockout_reason = 'OPTICAL_VS_DYNAMIC_AXLE_COUNT_MISMATCH';
END;

-- Trigger: Enforce immutability on optical ANPR audit records
CREATE TRIGGER IF NOT EXISTS abort_anpr_record_tamper
BEFORE UPDATE ON anpr_optical_captures
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Optical ANPR records are immutable evidentiary captures.');
END;
```

---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Oblique Yaw & Synthetic Mud Splatter OCR Assay:**
   - *Procedure:* Mount an HSRP test plate at an oblique $35^\circ$ angle relative to the optical camera axis. Apply synthetic mud/slurry over 25% of character area. Trigger entry detection.
   - *Pass/Fail Criteria:* Verify that the perspective homography warp normalizes the bounding box and that the CRNN recognizer outputs correct registration with confidence $C_{\text{ocr}} \ge 0.85$.

2. **Edge NPU Latency Budget & Throughput Profiling:**
   - *Procedure:* Process a batch of 50 vehicle arrival RTSP video frames using on-device INT8 quantization on an offline embedded board.
   - *Pass/Fail Criteria:* Verify that end-to-end inference latency satisfies $\tau_{\text{infer}} \le 110\text{ ms}$ per frame without memory leaks or dropped frames.

3. **Dynamic Axle Count Cross-Validation Assay:**
   - *Procedure:* Drive a 3-axle rigid truck ($N=3$) onto the approach ramp. Record the optical thru-beam curtain count ($N_{\text{optical}}$) and dynamic telemetry jerk count ($N_{\text{jerk}}$).
   - *Pass/Fail Criteria:* Ensure $N_{\text{optical}} = N_{\text{jerk}} = 3$. Simulate a mismatch ($N_{\text{optical}} = 3$, selected profile $= 2$) and verify that transition $T_2$ is blocked.

4. **Anti-Rollback Abort & Boom Barrier Lockout Drill:**
   - *Procedure:* Drive a vehicle onto the scale. Before gross weight lock ($T_4$), shift vehicle into reverse ($dM/dt < -500\,\text{kg/s}$).
   - *Pass/Fail Criteria:* Verify that the terminal instantly aborts the transaction, sets `rollback_detected = 1`, and keeps the exit barrier firmly locked at $0^\circ$.

5. **Anti-Tailgating Light Curtain Trap Test:**
   - *Procedure:* Follow a departing truck with a second vehicle separated by less than $1.5\text{ meters}$.
   - *Pass/Fail Criteria:* Verify that the exit barrier refrains from closing on the vehicle body while the entry barrier remains locked, blocking the trailing truck from capturing a fraudulent tare.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Zero-Cloud Embedded Inference Invariant**: computer vision models must execute entirely on local hardware (INT8 quantized NPU) within $\tau_{\text{infer}} \le 120\text{ ms}$, ensuring zero dependency on cloud connectivity.
  * Formally mandates the **Optical-Kinematic Axle Interlock**: physical vehicle configuration is authenticated by correlating optical light curtain pulses ($N_{\text{optical}}$) with strain-gauge jerk transients ($N_{\text{jerk}}$).
  * Codifies **Physical Barrier Relay Integrity**: exit gates remain mechanically locked until weighing state transitions successfully commit and print.
- **Impact on Room Database Schemas:**
  * Adds `anpr_optical_captures` and `gate_interlock_events` tables.
  * Implements `trigger_axle_mismatch_lockdown` and `abort_anpr_record_tamper` database triggers.

---

## 8. Digest Card

- **Key Invariants:** Zero-Cloud On-Device Vision ($\tau_{\text{infer}} \le 110\,\text{ms}$); Homography Perspective Rectification ($35^\circ$ oblique yaw); Optical-Kinematic Axle Match ($N_{\text{optical}} = N_{\text{jerk}}$); Anti-Rollback Abort ($dM/dt < -500\,\text{kg/s}$); Fail-Safe Boom Barrier GPIO Relays.
- **Plate Typologies Handled:** HSRP (CMVR Rule 50), Commercial Yellow, EV Green, Hand-Painted Rural, and Mud-Splattered Occluded.
- **Edge Model Envelope:** YOLO-v8n-INT8 detector ($\le 6.2\,\text{MB}$) + CRNN-CTC recognizer ($\le 4.8\,\text{MB}$).
- **Top 3 Gemba Hooks:**
  1. Test $35^\circ$ skewed mud-splattered HSRP plate capture accuracy.
  2. Verify local INT8 inference latency budget ($\le 110\,\text{ms}$) on offline hardware.
  3. Validate anti-rollback gate lock and transaction abort during vehicle reversal.
