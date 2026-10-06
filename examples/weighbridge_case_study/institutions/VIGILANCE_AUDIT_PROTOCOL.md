# Mandi Vigilance Inspection, Flying Squads & Anti-Raid Audit Defenses
Path: core/institutions/VIGILANCE_AUDIT_PROTOCOL.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, core/substrate/MEDIA_PERSISTENCE_SPEC.md, Legal Metrology Act 2009 (Sections 15 & 26), Bharatiya Sakshya Adhiniyam 2023 (Section 63)

## 1. Statutory Enforcement Topology & Multi-Agency Raid Dynamics

Commercial weighbridges and procurement yards operate under constant regulatory scrutiny from overlapping state and central enforcement authorities. An unannounced raid or roadside interception by a "Flying Squad" (Udan Dasta) creates an acute operational crisis.

```text
                  ┌────────────────────────────────────────┐
                  │       THE WEIGHBRIDGE FACILITY         │
                  └───────────────────┬────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│ LEGAL METROLOGY  │        │ MANDI SAMITI     │        │ STATE TAX / GST  │
│ INSPECTOR        │        │ ENFORCEMENT SQUAD│        │ VIGILANCE BUREAU │
├──────────────────┤        ├──────────────────┤        ├──────────────────┤
│ Legal Mandate:   │        │ Legal Mandate:   │        │ Legal Mandate:   │
│ • S. 15 LM Act   │        │ • UP Mandi Act   │        │ • S. 67/68 CGST  │
│ • Seal Integrity │        │ • Fee Evasion    │        │ • E-Way Bill MPE │
│ • MPE Tolerances │        │ • Unauthorized   │        │ • Mass Mismatch  │
│ • Stamping Cert  │        │   Deductions     │        │ • Transit Evasion│
└────────┬─────────┘        └────────┬─────────┘        └────────┬─────────┘
         │                           │                           │
         └───────────────────────────┼───────────────────────────┘
                                     │
                                     ▼
                        ┌──────────────────────────┐
                        │ DISTRICT ADMINISTRATION  │
                        │ & FOOD / CIVIL SUPPLIES  │
                        ├──────────────────────────┤
                        │ Legal Mandate:           │
                        │ • Essential Commodities  │
                        │ • MSP Procurement Raids  │
                        │ • PDS Grain Diversion    │
                        └──────────────────────────┘
```

### 1.1 The Operational Anatomy of a Yard Raid

During an unannounced inspection, enforcement squads leverage information asymmetry and psychological intimidation:

- **Immediate Demand for Registers:** Officers demand physical weighment books (*Kanta Bahi*) to identify missing tickets or serial number gaps.
- **Standard Check-Weighment:** The inspector orders a loaded truck to be re-weighed, or mounts verified 20kg or 50kg cast-iron test weights on the scale deck to measure Maximum Permissible Error (MPE) drift.
- **Hardware & Cabling Scrutiny:** Pit junction boxes and indicator calibration switches are examined for cut seal wires, unauthorized splices, or hidden RF relays.
- **Hardware Seizure Threat:** Officers threaten to confiscate the mobile tablet or desktop PC under Section 15 of the Legal Metrology Act if operators fail to immediately produce legible audit records.

---

## 2. The Read-Only Statutory Inspector Mode (Air-Gapped Audit View)

To prevent hostile officers from browsing private commercial transactions or accidentally mutating database state, the native instrument provides a dedicated Statutory Auditor Mode.

### 2.1 The Two-Touch Audit Portal

The operator triggers Auditor Mode via:
1. Long-pressing the official Legal Metrology seal emblem on the home screen for 3 seconds.
2. Entering the universal Auditor Mode PIN (`0000`) or scanning the Inspector's departmental QR credential.

### 2.2 Functional Behavior of Auditor Mode

When activated, the UI instantly reconfigures:
- **Write Actions Disabled:** New weighments, tare locks, and ticket editing are completely suppressed.
- **Strict Data Segregation (Statutory vs. Proprietary):**
  - **VISIBLE TO INSPECTOR:** Date/Time, Ticket Number, Vehicle Registration, Gross Mass, Tare Mass, Net Mass, Calibration Date, Active Stamping Certificate ID, OIML Accuracy Class, and MPE verification logs.
  - **COMPLETELY CONCEALED:** Purchase prices, commercial margins, farmer khata balances, trader commission rates, and pre-weighment advances (*Peshgi*).
- **One-Touch Verification Export:** Displays a single prominent button: `EXPORT STATUTORY AUDIT FILE (USB)`.

---

## 3. Cryptographic USB Flash Dump & BSA 2023 Section 63 Evidence Generation

When an inspector demands digital records for off-site forensic examination, handing over the operational Android tablet paralyzes yard trade. Instead, the application exports a self-contained, cryptographically signed audit bundle directly to an external USB-OTG flash drive.

### 3.1 The Audit Bundle Architecture (`AUDIT_EXPORT.BIN`)

The generated flash package contains three verifiable artifacts:

```text
/USB_ROOT/WEIGHBRIDGE_AUDIT_[DEVICE_ID]_[TIMESTAMP]/
  ├── 1_STATUTORY_REGISTER.CSV      # Tabular weighment records (Metrological data only)
  ├── 2_CALIBRATION_HISTORY.JSON    # Zero logs, span calibration events, MPE checks
  ├── 3_BSA_SECTION_63_CERT.PDF     # Formally formatted Section 63 BSA certificate
  └── MANIFEST.SIG                  # ECDSA P-256 signature across all file SHA-256 hashes
```

### 3.2 Automated Section 63 BSA 2023 Certificate Generation

Under Section 63 of the Bharatiya Sakshya Adhiniyam, 2023 (formerly Section 65B of the Indian Evidence Act, 1872), electronic records are admissible in court only when accompanied by an authoritative certificate identifying the device, verifying its lawful custody, and affirming its uncorrupted operation.

The software dynamically compiles and signs this certificate, embedding:
- **Device Fingerprint:** Hardware Android ID, IMEI (if present), and burned SoC serial number.
- **Software Determinism:** Git commit hash of running app bytecode and SQLite schema version.
- **Continuous Cryptographic Chain:** Root SHA-256 hash of the `electronic_evidence_ledger` table.
- **Statutory Affirmation Text:** Mandated legal declaration confirming that the device was operating under lawful custody and normal operational conditions throughout the query window.

---

## 4. Test-Weighment Protocol & In-Field MPE Verification

When an inspector places physical test weights on the scale platform to verify Class III accuracy under Schedule VII of the Legal Metrology (General) Rules, 2011, the software provides a structured Inspection Calibration Routine.

### 4.1 Maximum Permissible Error (MPE) Limits for Class III Instruments

For a standard commercial vehicle weighbridge ($	ext{Max} = 50,000\,	ext{kg}$, $e = 10\,	ext{kg}$):

| Load Range ($m$ in verification scale intervals $e$) | Load in Kilograms ($e=10\,	ext{kg}$) | Initial Verification MPE ($\pm$) | In-Service Inspection MPE ($\pm$) |
| :--- | :--- | :--- | :--- |
| $0 \le m \le 500e$ | $0	ext{ to }5,000\,	ext{kg}$ | $\pm 0.5e$ ($\pm 5\,	ext{kg}$) | $\pm 1.0e$ ($\pm 10\,	ext{kg}$) |
| $500e < m \le 2000e$ | $5,010	ext{ to }20,000\,	ext{kg}$ | $\pm 1.0e$ ($\pm 10\,	ext{kg}$) | $\pm 2.0e$ ($\pm 20\,	ext{kg}$) |
| $2000e < m \le 5000e$ | $20,010	ext{ to }50,000\,	ext{kg}$ | $\pm 1.5e$ ($\pm 15\,	ext{kg}$) | $\pm 3.0e$ ($\pm 30\,	ext{kg}$) |

### 4.2 In-Field Inspection Verification Workflow

1. Inspector places known mass (e.g., $1,000\,	ext{kg}$ composed of twenty $50\,	ext{kg}$ test blocks).
2. Operator selects `RECORD_OFFICIAL_TEST_WEIGHMENT` in Auditor Mode.
3. The software reads raw live indicator counts, compares indicated mass against certified test mass, and computes error:

$$E = 	ext{Mass}_{	ext{indicated}} - 	ext{Mass}_{	ext{certified\_standard}}$$

4. Evaluation:
   - If $|E| \le 	ext{MPE}_{	ext{in-service}}$, the UI renders a green **PASSED STATUTORY TOLERANCE** badge and generates a printable verification slip signed with the Inspector's badge number.
   - If $|E| > 	ext{MPE}_{	ext{in-service}}$, the system triggers `MPE_EXCEEDED_LOCKOUT`, entering a degraded state that blocks commercial ticket issuance until official re-stamping.

---

## 5. Room Database Schema Extensions

To guarantee tamper-proof tracking of regulatory inspections, test-weighment certifications, and evidentiary flash dumps, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Statutory Inspections & Test-Weighments
CREATE TABLE IF NOT EXISTS statutory_inspection_audits (
    inspection_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    session_uuid TEXT NOT NULL,
    enforcement_agency TEXT CHECK(enforcement_agency IN (
        'LEGAL_METROLOGY', 
        'MANDI_SAMITI_FLYING_SQUAD', 
        'GST_COMMERCIAL_TAX', 
        'DISTRICT_ADMINISTRATION_CIVIL_SUPPLIES'
    )) NOT NULL,
    inspector_badge_number TEXT NOT NULL,
    inspector_officer_name TEXT NOT NULL,
    test_weights_mass_kg INTEGER NOT NULL,          -- Known certified mass placed on deck
    indicated_mass_kg INTEGER NOT NULL,             -- Mass read by scale indicator
    observed_error_kg INTEGER NOT NULL,             -- indicated - certified
    mpe_tolerance_limit_kg INTEGER NOT NULL,        -- Statutory in-service MPE limit
    mpe_inspection_status TEXT CHECK(mpe_inspection_status IN ('PASSED', 'FAILED_EXCEEDED_MPE')) NOT NULL,
    bsa_cert_document_hash TEXT NOT NULL,          -- SHA-256 of generated S.63 certificate
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_inspection_time 
    ON statutory_inspection_audits(timestamp_epoch_ms);

-- Architectural Extension: Tracking Cryptographic Evidence USB Dumps
CREATE TABLE IF NOT EXISTS bsa_evidence_exports (
    export_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    usb_volume_label TEXT NOT NULL,
    exported_record_count INTEGER NOT NULL,
    date_range_start_epoch_ms INTEGER NOT NULL,
    date_range_end_epoch_ms INTEGER NOT NULL,
    manifest_sha256 TEXT NOT NULL,
    ecdsa_p256_signature_hex TEXT NOT NULL,
    exporting_operator_id TEXT NOT NULL,
    export_timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Prevent modification or deletion of official inspection records
CREATE TRIGGER IF NOT EXISTS abort_statutory_audit_tamper
BEFORE UPDATE ON statutory_inspection_audits
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Statutory inspection audit logs are immutable legal records.');
END;

-- Trigger: Automatic operational lockout on failed MPE test
CREATE TRIGGER IF NOT EXISTS trigger_mpe_failure_lockout
AFTER INSERT ON statutory_inspection_audits
FOR EACH ROW
WHEN NEW.mpe_inspection_status = 'FAILED_EXCEEDED_MPE'
BEGIN
    UPDATE system_operational_state 
    SET is_metrological_lockout_active = 1,
        lockout_reason = 'STATUTORY_MPE_ERROR_TOLERANCE_EXCEEDED';
END;
```

---

## 6. Field Verification Hooks (Gemba Protocols)

Enforcement readiness must be verified through the following five field audit simulations:

1. **Simulated Flying Squad Raid Drill:**
   - *Procedure:* Trigger Auditor Mode using the Legal Metrology seal long-press and PIN `0000`. Hand the tablet to an independent reviewer simulating an enforcement officer.
   - *Pass/Fail Criteria:* Verify that all commercial purchase rates, trader margins, and farmer khata ledgers are completely hidden, while all weights, timestamps, and calibration records are immediately legible.

2. **Offline USB-OTG Dump Forensic Integrity Test:**
   - *Procedure:* Insert a standard FAT32 USB flash drive into the Android tablet's OTG port. Execute `EXPORT STATUTORY AUDIT FILE (USB)`. Mount the drive on an air-gapped forensic workstation.
   - *Pass/Fail Criteria:* Verify that `MANIFEST.SIG` validates cleanly against the device public key and that `1_STATUTORY_REGISTER.CSV` contains zero serial number gaps.

3. **Class III In-Service MPE Tolerance Assay:**
   - *Procedure:* Place twenty certified 50kg cast-iron test weights (1,000kg total) on the center and four corners of the weighbridge deck. Record an official test weighment.
   - *Pass/Fail Criteria:* Verify that the software correctly applies the $\pm 1.0e$ ($\pm 10\,	ext{kg}$) in-service tolerance limit, computes observed error $E$, and prints a signed verification slip.

4. **BSA 2023 Section 63 Certificate Admissibility Verification:**
   - *Procedure:* Generate the automated PDF certificate for a 30-day transaction window. Inspect the printed output against statutory legal criteria.
   - *Pass/Fail Criteria:* Verify the certificate explicitly details the hardware SoC serial number, Git bytecode hash, SQLite schema version, and unbroken evidence hash chain root.

5. **Deliberate MPE Failure Lockout Drill:**
   - *Procedure:* Enter a simulated test weight of 1,000kg while placing only 900kg on the platform ($E = -100\,	ext{kg}$, exceeding the $10\,	ext{kg}$ limit).
   - *Pass/Fail Criteria:* Verify that `statutory_inspection_audits` logs `FAILED_EXCEEDED_MPE`, the database trigger sets `is_metrological_lockout_active = 1`, and the system hard-blocks all subsequent commercial ticket printing.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Statutory Privacy Firewall Invariant**: regulatory inspectors have full access to metrological telemetry, but software physically segregates and hides commercial price margins and proprietary financial ledgers.
  * Formally mandates **Automated BSA 2023 Section 63 Certification**: every digital export must be accompanied by an on-device generated, cryptographically anchored certificate satisfying Indian legal admissibility standards.
  * Enforces **Statutory MPE Lockout**: scales failing official check-weighments enter an immediate fail-safe lock state until verified re-stamping.
- **Impact on Room Database Schemas:**
  * Adds `statutory_inspection_audits` and `bsa_evidence_exports` tables.
  * Implements `trigger_mpe_failure_lockout` and `abort_statutory_audit_tamper` triggers.

---

## 8. Digest Card

- **Key Invariants:** Statutory Privacy Firewall (Metrology visible, commercial pricing masked); Automated BSA 2023 Section 63 Evidence PDF generation; Offline USB-OTG Cryptographic Dump; Schedule VII Class III MPE Boundary Checks; Hard Lockout on MPE Failure.
- **Regulatory Agencies Addressed:** Legal Metrology (LM Act S. 15/26), Mandi Samiti (UP Mandi Act S. 17/28), State Tax / GST Vigilance (CGST S. 67/68), District Administration / Civil Supplies.
- **Core Forensic Artifacts:** `1_STATUTORY_REGISTER.CSV`, `2_CALIBRATION_HISTORY.JSON`, `3_BSA_SECTION_63_CERT.PDF`, `MANIFEST.SIG`.
- **Top 3 Gemba Hooks:**
  1. Test Auditor Mode UI to confirm proprietary financial masking under simulated raid.
  2. Verify offline USB flash export and ECDSA P-256 signature chain.
  3. Validate automated MPE tolerance checking and failure lockout trigger.
