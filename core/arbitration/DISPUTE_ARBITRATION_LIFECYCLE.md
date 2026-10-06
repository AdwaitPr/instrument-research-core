# Inter-Mandi Dispute Arbitration, Discrepancy Lifecycles & Transit Loss (Ghaat)
Path: core/arbitration/DISPUTE_ARBITRATION_LIFECYCLE.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md, core/substrate/MEDIA_PERSISTENCE_SPEC.md, Carriage by Road Act 2007 (Sections 10 & 11), Sale of Goods Act 1930

## 1. Origin-Destination Discrepancy Topology & In-Transit Mass Physics

When bulk freight (foodgrains, oilseeds, coal, cement, iron scrap) moves between an origin dispatch facility (e.g., procurement mandi in Sitapur) and a destination receiving facility (e.g., flour mill in Kanpur), measured mass at destination rarely matches origin dispatch mass.

```text
                  ┌────────────────────────────────────────┐
                  │    ORIGIN FACILITY DISPATCH PASS       │
                  │    Net Origin Mass: M_origin           │
                  └───────────────────┬────────────────────┘
                                      │ In-Transit Physical Journey (100 – 800 km)
                                      ▼
                  ┌────────────────────────────────────────┐
                  │    IN-TRANSIT MASS VARIATION FORCES:   │
                  │    • Evaporative Moisture Loss (Sookhat│
                  │    • Spillage & Road Sifting Vibration │
                  │    • Driver/Crew Pilferage (Ghaat)     │
                  │    • Scale-to-Scale Calibration Delta  │
                  └───────────────────┬───────────────────┘
                                      │
                                      ▼
                  ┌────────────────────────────────────────┐
                  │   DESTINATION RECEIVING INBOUND PASS   │
                  │   Net Destination Mass: M_dest         │
                  └───────────────────┬────────────────────┘
                                      │
                                      ▼
                  ┌────────────────────────────────────────┐
                  │       METROLOGICAL VARIANCE CALC:      │
                  │       ΔM = M_origin - M_dest           │
                  └────────────────────────────────────────┘
```

### 1.1 Physical & Metrological Components of Transit Loss

The total observed weight disparity ($\Delta M$) decomposes into four distinct physical vectors:

$$\Delta M = \Delta M_{\text{evaporation}} + \Delta M_{\text{sifting}} + \Delta M_{\text{scale\_delta}} + \Delta M_{\text{theft}}$$

- **Evaporative Drying Shrinkage (Sookhat / Nami Loss):** Freshly harvested grain loaded at 14.5% moisture content drying down to 12.5% during a 3-day transit in hot, dry northern plains weather ($> 42^\circ\text{C}$). Natural water loss directly reduces net mass without loss of dry matter.
- **Mechanical Sifting & Spillage:** Fine particulates and dust sifting through burlap pores and truck bed floorboards under continuous road vibration.
- **Inter-Instrument Calibration Delta ($\Delta M_{\text{scale\_delta}}$):** Two separate legal metrology Class III scales operating at opposite ends of their allowable Maximum Permissible Error (e.g., Origin Scale reads $+0.15\%$ high; Destination Scale reads $-0.15\%$ low, producing an artificial $0.30\%$ ghost discrepancy).
- **Adversarial Siphoning & Pilferage (Ghaat):** Intentional bag skimming, bottom-dump leakage, or auxiliary fuel bladder discharge by the driver/transporter.

---

## 2. Multi-Tier Dispute Escalation State Machine

The application models weight dispute resolution as a deterministic finite state machine, preventing informal verbal deduction fights from halting yard unloading:

```text
                   ┌──────────────────────────────┐
                   │    [1] ORIGIN_DISPATCHED     │
                   └──────────────┬───────────────┘
                                  │ Destination Weighment Complete
                                  ▼
                   ┌──────────────────────────────┐
                   │   [2] DESTINATION_RECORDED   │
                   └──────────────┬───────────────┘
                                  │
         ┌────────────────────────┴────────────────────────┐
         │ |ΔM| ≤ Standard Tolerance (0.5%)                │ |ΔM| > Standard Tolerance
         ▼                                                 ▼
┌──────────────────────────────┐          ┌──────────────────────────────┐
│  [3] AUTO_CLEARED_NOMINAL    │          │   [4] DISCREPANCY_FLAGGED    │
│  • Instant Settlement        │          │   • Quarantines Settlement   │
│  • Release Carrier Freight   │          │   • Prompts Diagnostic Pass  │
└──────────────────────────────┘          └──────────────┬───────────────┘
                                                         │
                                  ┌──────────────────────┴──────────────────────┐
                                  │ Check-Tare & Moisture Match                 │ Discrepancy Unresolved
                                  ▼                                             ▼
                   ┌──────────────────────────────┐              ┌──────────────────────────────┐
                   │ [5] BILATERAL_RECONCILIATION │              │  [6] FORMAL_LEGAL_ARBITRATION│
                   │ • Pro-Rata Evaporative Split │              │  • S.10 Carriage by Road Act │
                   │ • Commercial Debit Note      │              │  • BSA S.63 Evidence Lock    │
                   └──────────────────────────────┘              └──────────────────────────────┘
```

### 2.1 State Transition Conditions & Safeguards

- **State [2] → [3] (`AUTO_CLEARED_NOMINAL`):** Fired if $|\Delta M| \le \text{Threshold}_{\text{commodity}}$ (typically $0.5\%$ for grain; $0.2\%$ for packaged cement). The variance is absorbed as customary transit allowance (*Chhoot*).
- **State [2] → [4] (`DISCREPANCY_FLAGGED`):** Fired if transit loss exceeds tolerance. The destination terminal halts automatic payment clearance, logs an alert, and mandates immediate physical vehicle check-tare.
- **State [4] → [5] (`BILATERAL_RECONCILIATION`):** Commercial counterparties agree on an evaporative adjustment calculation; an automated credit/debit voucher is appended to the ledger.
- **State [4] → [6] (`FORMAL_LEGAL_ARBITRATION`):** Mutual consent fails; the transaction locks into statutory arbitration mode, generating an immutable BSA Section 63 evidentiary package for the District Magistrate or Mandi Samiti Arbitration Panel.

---

## 3. Statutory Carrier Liability Framework: Carriage by Road Act, 2007

Commercial disputes are governed by statutory carrier liability provisions:

### 3.1 Common Carrier Strict Liability (Section 10 & 11)

Under Section 10 of the Carriage by Road Act, 2007, the common carrier is strictly liable for any loss, damage, or non-delivery of consignments entrusted to them, unless the carrier proves that the loss arose from an act of God, act of war, or natural deterioration/inherent vice of goods.

### 3.2 The Statutory Evaporative Defense Burden

To claim the "natural deterioration / inherent moisture loss" defense against a carrier shortage claim, the carrier must prove:
1. Origin moisture content ($M_{\text{origin}}$) and destination moisture content ($M_{\text{dest}}$) were measured using calibrated metrological moisture meters.
2. The observed loss matches the theoretical thermodynamic water weight loss:

$$\Delta M_{\text{theo}} = M_{\text{origin}} \times \left( \frac{\text{Moisture}_{\text{origin}} - \text{Moisture}_{\text{dest}}}{100 - \text{Moisture}_{\text{dest}}} \right)$$

If measured loss exceeds $\Delta M_{\text{theo}} + \text{Tolerance}_{\text{scale}}$, the excess is legally classified as Carrier Pilferage, and Section 11 permits immediate freight payment withholding.

---

## 4. Bilateral Cryptographic Proof-Pack Architecture

To arbitrate between two disconnected facilities without requiring continuous central server availability, the system links transactions using a Bilateral Cryptographic Proof-Pack.

### 4.1 Cross-Facility Manifest Chaining

The origin terminal encodes the dispatch event into an air-gapped cryptographic manifest printed as a dense 2D DataMatrix barcode on the physical lorry receipt (*Bilty* / Consignment Note):

```text
┌─────────────────────────────────────────────────────────────────┐
│               ORIGIN CRYPTOGRAPHIC DISPATCH TOKEN               │
├─────────────────────┬───────────────────────────────────────────┤
│ FIELD               │ METROLOGICAL & CRYPTOGRAPHIC PAYLOAD      │
├─────────────────────┼───────────────────────────────────────────┤
│ Origin Device UID   │ WB-UP-SITAPUR-004                         │
├─────────────────────┼───────────────────────────────────────────┤
│ Consignment UUID    │ 550e8400-e29b-41d4-a716-446655440000      │
├─────────────────────┼───────────────────────────────────────────┤
│ Vehicle Registration│ UP-32-BN-4521                             │
├─────────────────────┼───────────────────────────────────────────┤
│ Certified Net Mass  │ 28,450 kg (e = 10 kg, TIL-3 Certified)    │
├─────────────────────┼───────────────────────────────────────────┤
│ Moisture Baseline   │ 14.2% (ISO 712 Grain Analysis)            │
├─────────────────────┼───────────────────────────────────────────┤
│ Calibration Expiry  │ 2027-03-31 (Stamping Certificate #41209)  │
├─────────────────────┼───────────────────────────────────────────┤
│ Origin Manifest Sig │ ECDSA-P256 Signature across above fields  │
└─────────────────────┴───────────────────────────────────────────┘
```

### 4.2 Destination Verification Algorithm

Upon truck arrival, the destination terminal scans the 2D DataMatrix code:
1. Validates the origin terminal's ECDSA public key against its cached regional registry.
2. Compares destination gross/tare readings against the signed origin net mass.
3. If $\Delta M$ is anomalous, the app automatically generates an Arbitration Difference Certificate combining the origin signature and destination signature into an unbroken dual-stakeholder Merkle proof.
---

## 5. Room Database Schema Extensions

To track cross-facility discrepancies, thermodynamic moisture loss proofs, and bilateral arbitration workflows, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Origin-Destination Discrepancy Cases
CREATE TABLE IF NOT EXISTS inter_facility_discrepancy_cases (
    case_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    consignment_uuid TEXT UNIQUE NOT NULL,
    vehicle_registration TEXT NOT NULL,
    origin_facility_id TEXT NOT NULL,
    destination_facility_id TEXT NOT NULL,
    origin_certified_net_kg INTEGER NOT NULL,
    destination_measured_net_kg INTEGER NOT NULL,
    gross_discrepancy_delta_kg INTEGER NOT NULL,       -- origin - destination
    origin_moisture_percentage REAL,
    destination_moisture_percentage REAL,
    calculated_evaporative_loss_kg REAL NOT NULL DEFAULT 0.0,
    allowable_transit_tolerance_kg INTEGER NOT NULL,
    adjudicated_unexplained_loss_kg REAL NOT NULL DEFAULT 0.0,
    dispute_status TEXT CHECK(dispute_status IN (
        'AUTO_CLEARED_NOMINAL', 
        'DISCREPANCY_FLAGGED', 
        'UNDER_BILATERAL_RECONCILIATION', 
        'SETTLED_COMMERCIAL_REBATE', 
        'ESCALATED_LEGAL_ARBITRATION'
    )) NOT NULL,
    carrier_freight_withheld_paise INTEGER NOT NULL DEFAULT 0,
    created_epoch_ms INTEGER NOT NULL,
    resolved_epoch_ms INTEGER
);

CREATE INDEX IF NOT EXISTS idx_discrepancy_consignment 
    ON inter_facility_discrepancy_cases(consignment_uuid);

-- Architectural Extension: Storing Bilateral Cryptographic Evidence Packs
CREATE TABLE IF NOT EXISTS arbitration_evidence_bundles (
    bundle_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    consignment_uuid TEXT NOT NULL REFERENCES inter_facility_discrepancy_cases(consignment_uuid),
    origin_ecdsa_signature_hex TEXT NOT NULL,
    destination_ecdsa_signature_hex TEXT NOT NULL,
    dual_manifest_merkle_root_hex TEXT NOT NULL,
    bsa_section_63_cert_hash TEXT NOT NULL,
    arbitrator_panel_id TEXT,
    adjudication_verdict_notes TEXT,
    timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Automatically quarantine commercial settlement on flagged discrepancy
CREATE TRIGGER IF NOT EXISTS quarantine_freight_on_discrepancy
AFTER INSERT ON inter_facility_discrepancy_cases
FOR EACH ROW
WHEN NEW.dispute_status = 'DISCREPANCY_FLAGGED'
BEGIN
    UPDATE inter_facility_discrepancy_cases 
    SET carrier_freight_withheld_paise = (
        SELECT CAST(ROUND(NEW.adjudicated_unexplained_loss_kg * 2500) AS INTEGER) -- Statutory debit estimate
    )
    WHERE case_id = NEW.case_id;
END;

-- Trigger: Prevent deletion or updates on legal arbitration bundles
CREATE TRIGGER IF NOT EXISTS abort_arbitration_evidence_tamper
BEFORE UPDATE ON arbitration_evidence_bundles
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Arbitration evidentiary records are immutable legal artifacts.');
END;
```

---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Origin-Destination DataMatrix Offline Handshake Assay:**
   - *Procedure:* Print an origin dispatch ticket on an air-gapped terminal in Facility A with a 2D DataMatrix code. Physically transport the paper slip to Facility B with zero network connectivity. Scan with the receiving tablet.
   - *Pass/Fail Criteria:* Verify the destination terminal validates the origin ECDSA-P256 signature, matches the vehicle registration, and extracts certified dispatch mass in $< 1.5\text{ seconds}$.

2. **Sookhat Moisture Loss Equilibrium Simulation:**
   - *Procedure:* Input an origin dispatch record of $30,000\,\text{kg}$ wheat at 14.5% moisture. Record destination weighment of $29,420\,\text{kg}$ at 12.6% moisture ($\Delta M = 580\,\text{kg}$).
   - *Pass/Fail Criteria:* Verify the software calculates theoretical evaporative loss $\Delta M_{\text{theo}} \approx 652\,\text{kg}$, confirms that observed shrinkage is fully explained by natural dehydration, and classifies the trip as `AUTO_CLEARED_NOMINAL`.

3. **Scale-to-Scale Artificial Calibration Delta Test:**
   - *Procedure:* Configure Scale A with $+0.15\%$ calibration offset and Scale B with $-0.15\%$ offset (both within Class III MPE). Weigh a test truck at both scales without cargo changes.
   - *Pass/Fail Criteria:* Verify that the application’s inter-instrument calibration model computes $\Delta M_{\text{scale\_delta}}$ ($90\,\text{kg}$ on a 30-tonne load) and prevents false pilferage accusations against the carrier.

4. **Common Carrier Pilferage Withholding Drill (Carriage by Road Act S.10/11):**
   - *Procedure:* Simulate an inbound arrival where measured transit loss exceeds combined evaporative loss and scale tolerances by $650\,\text{kg}$.
   - *Pass/Fail Criteria:* Verify the state transitions immediately to `DISCREPANCY_FLAGGED`, the database trigger computes freight withholding amount in paise, and an automated Section 10 Carrier Shortage Notice is drafted.

5. **Bilateral Dual-Signature Arbitration Proof Assay:**
   - *Procedure:* Force a dispute into `ESCALATED_LEGAL_ARBITRATION`. Export the arbitration bundle.
   - *Pass/Fail Criteria:* Verify `arbitration_evidence_bundles` stores both origin and destination public keys, a dual-manifest Merkle root hash, and a legally admissible BSA Section 63 certificate suitable for submission to the District Mandi Tribunal.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Origin-Destination Discrepancy Invariant**: destination receiving scales must never alter certified origin weighment records; inter-facility variances must be resolved through explicit debit/credit arbitration states.
  * Formally mandates the **Thermodynamic Moisture Loss Model**: evaporative grain shrinkage (*Sookhat*) must be mathematically evaluated using calibrated moisture meter inputs before attributing shortages to carrier fraud.
  * Codifies **Bilateral Cryptographic Proof-Packs**: cross-facility disputes between disconnected terminals must be resolved using air-gapped 2D DataMatrix tokens with dual ECDSA-P256 signatures.
- **Impact on Room Database Schemas:**
  * Adds `inter_facility_discrepancy_cases` and `arbitration_evidence_bundles` tables.
  * Implements `quarantine_freight_on_discrepancy` and `abort_arbitration_evidence_tamper` database triggers.

---

## 8. Digest Card

- **Key Invariants:** Origin-Destination Immutable Baseline; Thermodynamic Moisture Loss Defense ($\Delta M_{\text{theo}}$); Common Carrier Strict Liability (Carriage by Road Act 2007 S. 10/11); Bilateral Dual-Signature Merkle Proofs; Automatic Commercial Freight Quarantining.
- **Physical Variance Vectors:** Evaporative Moisture Loss (*Sookhat*), Mechanical Sifting/Spillage, Inter-Instrument Calibration Tolerance Delta, and Carrier Pilferage (*Ghaat*).
- **Dispute Resolution States:** `ORIGIN_DISPATCHED` $\to$ `DESTINATION_RECORDED` $\to$ `AUTO_CLEARED_NOMINAL` | `DISCREPANCY_FLAGGED` $\to$ `BILATERAL_RECONCILIATION` | `ESCALATED_LEGAL_ARBITRATION`.
- **Top 3 Gemba Hooks:**
  1. Test air-gapped 2D DataMatrix cryptographic handshake between disconnected terminals.
  2. Verify automated evaporative loss calculation against moisture meter differential.
  3. Validate automated freight payment withholding on unexplained shortages under Section 11.
