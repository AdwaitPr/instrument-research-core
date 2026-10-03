# Evidentiary Persistence, Thermal Media Degradation & Cryptographic Proof Chains
Path: core/substrate/MEDIA_PERSISTENCE_SPEC.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, CGST Act 2017 S.36, Limitation Act 1963, BSA 2023 S.63

## 1. Physical Chemistry of Thermal Print Degradation

Commercial portable 58mm and 80mm Bluetooth ESC/POS printers deployed in Tier-2/3 physical yards (ATPOS, SEZNIK, Xprinter, Bluprint) utilize direct thermal paper rolls. The physical degradation vectors are governed by chromogenic chemistry:

### 1.1 The Chromogenic Reaction Substrate
- **Active Layer Chemistry:** Economy thermal paper ($48\text{--}55\text{ GSM}$) is coated with an emulsion containing a colorless leuco dye (typically fluoran-based) and an organic acidic developer (Bisphenol A / Bisphenol S, or sulfonylurea derivatives), suspended in a meltable wax-like binder.
- **Image Formation:** When heated to $80\text{--}120^\circ\text{C}$ by the thermal dot-line head (203 DPI), the binder melts, allowing the acid to donate a proton to the leuco dye, cleaving the lactone ring and developing the dark chromophore (absorption band in visible spectrum).

### 1.2 The Three Irreversible Degradation Mechanics
1. **Solvent / Hydrocarbon Dissolution (Diesel & Oil Wipeout):**
   - *Physical Mechanism:* Non-topcoated economy paper has zero chemical barrier. Diesel fuel, engine lubricants, and greasy hand oils dissolve the developer matrix.
   - *Decay Rate:* Complete bleaching to pure white legibility loss occurs within **15 minutes to 4 hours** of liquid contact.
2. **Plasticizer Reverse Reaction (PVC Document Pouches & Dashboard Sleeves):**
   - *Physical Mechanism:* Phthalate and adipate plasticizers used in standard flexible PVC sleeves (driver document folders, visor pouches) migrate into the thermal emulsion. Plasticizers act as preferential solvents that neutralize the acidic dye-developer complex.
   - *Decay Rate:* Legibility drop below 20% contrast occurs within **7 to 28 days**.
3. **Thermal Background Blackout (Solar Cabin Greenhouse Effect):**
   - *Physical Mechanism:* Direct thermal paper possesses a static sensitivity onset threshold ($60\text{--}75^\circ\text{C}$). Enclosed truck cabins parked in northern Indian summer sun ($45^\circ\text{C}$ ambient) reach interior dashboard temperatures of $68\text{--}78^\circ\text{C}$.
   - *Decay Rate:* The unprinted background spontaneously develops and turns solid black, erasing high-contrast text within **24 to 72 hours**.

Evidentiary Contrast Ratio
1.0 ┼───────╮ (Controlled dark office storage: 1–3 years)
│       │
0.6 ┼       ╰─────────╮ (Ambient room exposure: 6–12 months)
│                 │
0.2 ┼ ─ ─ ─ ─ ─ ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ (Legibility Threshold)
│                 ╰───╮ (Sunlight / High Heat: 30–90 days)
0.0 ┼─────────────────────┴───╭─── (Diesel / Plasticizer contact: 4 hrs – 14 days)
└─────────┬───────────────┬────────────────
Day 0          Day 90

---

## 2. The Statutory Evidentiary Lifetime Gap

A fundamental divergence exists between the chemical survival of physical thermal paper and Indian statutory record-retention mandates:

| Statutory / Legal Dimension | Required Evidentiary Lifetime | Governing Legal Instrument | Physical Thermal Paper Survival | Structural Gap |
| :--- | :--- | :--- | :--- | :--- |
| **GST Audit / Account Retention** | **72 Months (6 Years)** from annual return due date | Section 36, CGST Act, 2017 | $1\text{--}3\text{ months}$ (truck/yard storage) | **69–71 Months Missing** |
| **Commercial Contract Disputes** | **36 Months (3 Years)** from delivery of goods | Articles 14 & 15, Limitation Act, 1963 | $1\text{--}3\text{ months}$ (truck/yard storage) | **33–35 Months Missing** |
| **APMC Mandi Audit / Cess Records** | **36 to 60 Months** | UP Krishi Utpadan Mandi Adhiniyam, S. 17 | $1\text{--}3\text{ months}$ (khata storage) | **33–57 Months Missing** |

**Architectural Law:** A physical thermal print slip is legally and physically incapable of serving as an archival repository. It functions strictly as a **transient physical handover token (a 48-hour social weapon)**. Primary evidentiary persistence must reside in an immutable on-device digital chain.

---

## 3. Cryptographic Proof Chain & Secondary Digital Persistence

To guarantee absolute evidentiary admissibility under Section 63 of the **Bharatiya Sakshya Adhiniyam, 2023 (BSA)**, every physical slip emitted must be the visual representation of an immutable digital cryptogram.

### 3.1 Digital Proof Topology
[Physical Transaction Event]
│
├─► 1. Commit to SQLite (ledger_transactions)
│      • Calculate SHA-256(prev_hash + tx_payload)
│      • Generate ECDSA Signature via Android Keystore
│
├─► 2. Render 80mm ESC/POS Slip
│      • Truncated Hash (first 16 hex chars)
│      • Offline-Verifiable Verification QR Code
│
└─► 3. Emit Secondary Digital Token
• Render 1-bit Monochrome Slip Bitmap
• Export to Scoped Storage: /Documents/Ledger/WB_YYYYMMDD_ID.png
• Trigger Intent: Dispatch WhatsApp Image Card / PDF receipt

### 3.2 The Thermal Slip Visual & Data Anatomy (80mm ESC/POS)
The physical receipt layout must maintain readability even through partial thermal fade:

```text
+------------------------------------------------+
|          MANDI WEIGHBRIDGE CUSTODY SLIP        | [Double-Height Bold]
|  Licence: UP/APMC/SIT/2024-918  | WB-ID: WB-01 |
+------------------------------------------------+
| TICKET: WB-20261003-0142                       |
| DATE: 03-Oct-2026 16:42:10 IST                 |
| VEHICLE: UP-34-AT-4921   [TRACTOR-TROLLEY]     |
| COMMODITY: Wheat (Gehun) - Grade A             |
+------------------------------------------------+
| GROSS WEIGHT:                       24,850 kg  | [Bold, High Contrast]
| TARE WEIGHT:                         7,120 kg  |
| NET WEIGHT:                         17,730 kg  | [Double-Width Bold]
| BARDANA DEDUCTION (180 Jute @ 1kg):    180 kg  |
| CHARGEABLE NET WEIGHT:              17,550 kg  |
+------------------------------------------------+
| RATE: Rs. 2,425 / Quintal                      |
| GROSS VALUE:                   Rs. 4,25,587.50 |
| MANDI FEE (1.50%):             Rs.   6,383.81  |
| DEVELOPMENT CESS (0.50%):      Rs.   2,127.94  |
| NET PAYABLE:                   Rs. 4,17,075.75 | [In Paise: 41707575]
+------------------------------------------------+
| FORENSIC PROOF ANCHOR:                         |
| INTEGRITY: TIL-3 [SERIAL + CAMERA VERIFIED]    |
| HASH: a3f8c2e1b9d4f7a6c8e2b5d9                 |
|                                                |
|       [QR CODE: VERIFICATION PAYLOAD]          |
|                                                |
| Scan via app to verify against local node DB   |
+------------------------------------------------+
| NOTICE: Retain digital copy via WhatsApp.      |
| Physical thermal print degrades over time.     |
+------------------------------------------------+
```
### 3.3 Offline QR Verification Payload
The QR code printed on the thermal slip does **not** rely on a cloud web URL. It encodes a structured, self-contained alphanumeric string that any peer node can verify offline:

$$\text{QRPayload} = \text{WB1}\parallel \text{UUID}_{16} \parallel \text{Seq}_8 \parallel \text{Gross}_8 \parallel \text{Tare}_8 \parallel \text{Rate}_8 \parallel \text{Timestamp}_8 \parallel \text{ShortSig}_{32}$$

Scanning the QR code on a second Android device running the instrument app verifies the digital signature against the enrolled master public key in $<50\text{ ms}$, validating total payload authenticity even if the printed text has faded.

---

## 4. Evidentiary Bench Test Protocol (Gemba Media Audit)

Field researchers must execute this low-cost accelerated aging test to calibrate local thermal paper decay curves for target yards:

### 4.1 Sample Collection & Exposure Matrix
Collect 20 identical test tickets printed on the yard's active Bluetooth printer and distribute across 4 test cells (5 tickets per cell):

| Cell ID | Environment Simulation | Temperature / Exposure | Duration | Target Metric |
| :--- | :--- | :--- | :--- | :--- |
| **Cell A (Baseline Control)** | Dark office drawer in cardboard box | $22\text{--}26^\circ\text{C}$, 40% RH, zero light | 90 Days | Optimal degradation floor ($<5\%$ contrast loss). |
| **Cell B (Vehicle Dashboard)** | Under direct truck windshield glass | Solar greenhouse ($55\text{--}75^\circ\text{C}$) | 14 Days | Days until spontaneous background thermal blackout. |
| **Cell C (PVC Folder Contact)** | Compressed inside clear flexible PVC sleeve | Ambient yard cabin ($30\text{--}40^\circ\text{C}$) | 30 Days | Days until text dissolves due to plasticizer migration. |
| **Cell D (Hydrocarbon Smear)** | Light diesel/engine oil smudge applied | Ambient yard cabin ($30\text{--}40^\circ\text{C}$) | 48 Hours | Hours until complete text erasure. |

### 4.2 Measurement & Optical Density Tracking
- Photograph slips every 72 hours using a standardized test rig (fixed height, fixed illumination).
- Compute contrast ratio:
  $$C_R = \frac{L_{\text{background}} - L_{\text{text}}}{L_{\text{background}}}$$
- Record the exact day when $C_R < 0.20$ (point of complete dispute failure).

---

## 5. Room Database Schema Extensions

To guarantee digital compliance with Section 63 of Bharatiya Sakshya Adhiniyam, 2023, the schema stores cryptographic certificates and electronic receipt generation metadata:

```sql
-- Architectural Extension: Tracking Electronic Records for BSA 2023 Compliance
CREATE TABLE IF NOT EXISTS electronic_evidence_ledger (
    evidence_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    transaction_id INTEGER NOT NULL REFERENCES ledger_transactions(ledger_id),
    ticket_uuid TEXT NOT NULL UNIQUE,
    hash_sha256 TEXT NOT NULL,
    signature_ecdsa BLOB NOT NULL,
    signer_key_alias TEXT NOT NULL,
    device_hardware_serial TEXT NOT NULL,
    operating_system_build TEXT NOT NULL,
    app_version_code INTEGER NOT NULL,
    rendered_bitmap_path TEXT NOT NULL,
    digital_export_timestamp_epoch_ms INTEGER NOT NULL,
    is_whatsapp_dispatched INTEGER NOT NULL DEFAULT 0,
    is_pdf_exported INTEGER NOT NULL DEFAULT 0,
    created_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_evidence_tx 
    ON electronic_evidence_ledger(transaction_id);

-- Enforce immutability of evidentiary records
CREATE TRIGGER IF NOT EXISTS abort_evidence_tamper
BEFORE UPDATE ON electronic_evidence_ledger
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Electronic evidence records are immutable under BSA 2023.');
END;
```
## 6. Field Verification Hooks (Gemba Protocols)

Field researchers and systems auditors must execute the following five protocols at an active physical yard to calibrate media decay and evidence workflows:

1. **Aged Ticket Forensic Pull:**
   - *Procedure:* Collect 10 physical paper slips from drivers and munims that are $>60\text{ days}$ old. Record their storage location (truck visor, metal cash box, plastic folder, shirt pocket).
   - *Evaluation Criteria:* Scan ticket text using a standard smartphone camera. Measure optical density drop and check if the printed hash/QR remains machine-decodable.
2. **Thermal Roll Supply Chain Audit:**
   - *Procedure:* Inspect the physical inventory of thermal paper rolls in the weighbridge cabin. Photograph inner plastic/cardboard core markings, box labels, and roll thickness.
   - *Evaluation Criteria:* Identify if rolls are unbranded generic ($48\text{--}55\text{ GSM}$, non-topcoated, sub-₹18) or certified archival grade ($>65\text{ GSM}$, topcoated).
3. **Vehicle Dashboard Thermal Blackout Stress:**
   - *Procedure:* Place an active freshly printed test slip on the dashboard of a truck parked in direct midday sun ($12:00\text{--}16:00\text{ IST}$). Monitor surface temperature with an infrared thermometer.
   - *Evaluation Criteria:* Document the exact temperature and time elapsed when the unprinted background develops black, obliterating receipt text.
4. **Driver PVC Folder Plasticizer Migration Audit:**
   - *Procedure:* Place a freshly printed ticket inside a standard flexible PVC driver document sleeve. Apply light manual pressure ($1\text{ kg}$ weight) inside a warm cabin environment for 7 days.
   - *Evaluation Criteria:* Measure the rate of text bleaching and contrast loss caused by phthalate plasticizer absorption.
5. **BSA 2023 Section 63 Certificate Admissibility Test:**
   - *Procedure:* Generate a sample Section 63 electronic evidence certificate from the on-device SQLite database and present it to an advocate handling commercial contract disputes.
   - *Evaluation Criteria:* Verify that device hardware serial, cryptographic hash chain, and timestamp logging meet commercial court admissibility standards without requiring the physical paper receipt.

---

## 7. Statutory Electronic Evidence Certificate Specification (BSA 2023 Section 63)

Under Section 63 of Bharatiya Sakshya Adhiniyam, 2023, electronic device records are legally admissible in judicial proceedings provided they are accompanied by a certified technical statement. The native application generates this certificate on demand as a signed plain-text / printable artifact:

```text
CERTIFICATE UNDER SECTION 63 OF THE BHARATIYA SAKSHYA ADHINIYAM, 2023
FOR ADMISSIBILITY OF ELECTRONIC RECORDS

I, [Operator / Custodian Name], hereby certify and state as follows:
1. I am the authorized operator/custodian of the mobile weighing instrument terminal (Device Serial: [Device_Serial_No], Android OS Build: [Build_ID], App Version: [Version_Code]).
2. The weighing and financial record identified as Ticket UUID: [Ticket_UUID] was produced by the said device during its ordinary lawful operational activity.
3. The measurement and transactional data was captured directly via serial interface link from verified weighing indicator [Indicator_Make_Model] on [Date_Time_ISO].
4. Throughout the operational period, the device and its internal SQLite database operated properly without unmanaged interruption, data corruption, or unauthorized modification.
5. Cryptographic Anchor:
   - Record SHA-256 Digest: [Full_64_Hex_Digest]
   - Hardware-Backed Digital Signature: [Base64_ECDSA_P256_Signature]
   - Key Alias: [Keystore_Key_Alias] (Stored inside Android Hardware TEE / StrongBox)
6. The electronic record stored in the immutable database has not been altered or deleted.

Signed on [Date_Time] at [Mandi_Location_GPS]:
Signature of Certifier: ___________________________
Designation: Registered Weighbridge In-Charge
```
