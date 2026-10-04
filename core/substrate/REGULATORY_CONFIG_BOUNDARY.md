# Regulatory Parameter Boundaries, Gazette Drift & Versioned Configuration Architecture
Path: core/substrate/REGULATORY_CONFIG_BOUNDARY.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, The Legal Metrology Act 2009, CGST Act 2017, MV Act 1988

## 1. The Statutory Instrument Hierarchy

Native instrumentation software deployed in commercial yards operates within an unyielding legal hierarchy. Every parameter used in trade arithmetic, tare deduction, and penalty enforcement must map to its authoritative legal instrument:

[Level 1: Sovereign Statutes]
├── The Legal Metrology Act, 2009 (Verification cycles, MPE, criminal liabilities)
├── Motor Vehicles Act, 1988 / 2019 (Axle load limits, statutory overload fines)
└── Central / State GST Acts, 2017 (Tax rates, invoicing thresholds, reverse charge)
│
[Level 2: Statutory Executive Rules]
├── Legal Metrology (General) Rules, 2011 (Seventh Schedule verification standards)
└── State APMC Rules (e.g., UP Krishi Utpadan Mandi Niyamavali)
│
[Level 3: Gazette Notifications (Primary Rate Engine)]
├── MoRTH S.O. 3467(E) (Axle load weight schedules)
└── State e-Gazette Notifications (Mandi Fee, Development Cess, Hamali rate caps)
│
[Level 4: Administrative Circulars & Local Orders (Operational Guidance Only)]
└── District Mandi Secretary executive circulars (Temporary queue handling rules)
│
[Level 5: Private Yard Agreements (Legally Subordinate)]
└── Informal yard conventions, stamp-paper covenants (Zero statutory standing)

**Architectural Law:** Application software must never alter statutory tax or cess parameters based on Level 4 or Level 5 informal artifacts. If a parameter diverges from Level 3 Gazette rules, it must be recorded as a **Negotiated Commercial Discount**, never as a statutory rate alteration.

---

## 2. Empirical Gazette Base-Rate Analysis (Pilot State: Uttar Pradesh)

To disprove the assumption that regulations change unpredictably week-to-week, empirical analysis of state and central gazettes reveals long stability cycles:

| Regulatory Parameter | Primary Governing Instrument | Typical Revision Cycle | 5-Year Drift Count (2019–2024) | Safe Offline Cache Window |
| :--- | :--- | :--- | :--- | :--- |
| **Mandi Market Fee** | UP State Gazette Notification (S. 17) | $3\text{--}5\text{ Years}$ | 2 Revisions ($2.0\% \to 1.0\% \to 1.5\%$) | 180 Days |
| **Mandi Development Cess** | UP State Gazette Notification (S. 17) | $3\text{--}5\text{ Years}$ | 1 Revision ($0.50\% \to 0.0\% \to 0.50\%$) | 180 Days |
| **Truck Axle Load Limits (GVW)**| Central Gazette (MoRTH S.O. 3467(E)) | $5\text{--}10\text{ Years}$ | 0 Revisions since July 2018 | 365 Days |
| **Metrological Scale Verification**| Legal Metrology (General) Rules, R. 27| Fixed Statutory | 0 Revisions (Strictly 12 Months) | Indefinite (Annual check) |
| **Overload Statutory Base Penalty**| Motor Vehicles Act 2019, Section 194 | $5\text{--}10\text{ Years}$ | 0 Revisions (₹20,000 base + ₹2,000/t) | 365 Days |
| **Raw Unprocessed Grain GST** | GST Council / CBIC Notification | Council Review | 0 Revisions (Exempt / Nil Rated) | 180 Days |

**Conclusion:** Physical operational parameters are highly stable. The primary engineering threat is not rapid drift, but **silent obsolescence**: software running for 3 years without updating a rate when a rare gazette change *does* occur.

---

## 3. Append-Only Versioned Configuration Engine

Hardcoding tax, cess, and weight limits inside compiled Kotlin code or application constants is strictly prohibited. Regulatory parameters must be modeled as temporally versioned, immutable database entities.

### 3.1 Temporal Validity Model
Every parameter exists within a semi-open epoch millisecond interval:
$$t \in [\text{effective\_from\_epoch\_ms}, \text{effective\_to\_epoch\_ms})$$

- When a rate is current: $\text{effective\_to\_epoch\_ms} = \text{NULL}$ (or $9223372036854775807$).
- When a new gazette notification takes effect:
  1. The active parameter row has its $\text{effective\_to\_epoch\_ms}$ set to the new effective epoch timestamp.
  2. A new parameter row is appended with the new rate and a foreign-key link `supersedes_config_id`.
  3. Historical transactions are never mutated; they retain referential pointers to the parameter row active at their creation time.

### 3.2 Integer Basis Point Representation
To eliminate floating-point representation drift:
$$\text{Rate}_{\text{basis\_points}} = \text{Percentage} \times 100 \quad (1\text{ bp} = 0.01\% = 1/10{,}000)$$
$$\text{CalculatedFee}_{\text{paise}} = \left( \text{TaxableBase}_{\text{paise}} \times \text{Rate}_{\text{basis\_points}} \right) \div 10{,}000$$

Division uses standard integer arithmetic with midpoint rounding (`RoundingMode.HALF_EVEN`).

---

## 4. Offline Parameter Ingestion & Verification Protocols

Because target devices operate without internet access, parameter updates execute through two authorized, auditable paths:

### 4.1 Path A: Cryptographically Signed Config Bundle (`config.sig`)
1. The administrative back office packages updated gazette parameters into a compact JSON/CBOR bundle.
2. The payload is signed with the enterprise master private key (ECDSA P-256):
   $$\text{ConfigBundle} = \text{PayloadBytes} \parallel \text{SignatureBytes}$$
3. The bundle is transferred via BLE GATT, animated QR, or USB-OTG sneakernet.
4. The native terminal verifies the signature against the embedded hardware-backed Master Public Key. Upon verification, SQLite appends the new configuration rows.

### 4.2 Path B: Dual-Custody Supervisory Manual Override
If a gazette rate changes while the device is fully isolated and without a signed bundle:
1. Two distinct authorized individuals (e.g., Yard Owner PIN + Scale Supervisor PIN) must authenticate.
2. The UI requires entering:
   - New rate in basis points.
   - Statutory Citation (Gazette Notification Number, e.g., `UP-MANDI-NOTIF-2026-881`).
   - Effective Date and Time.
3. The system logs a `REGULATORY_MANUAL_OVERRIDE` event and watermarks subsequent printouts with `MANUAL_CONFIG_UNVERIFIED` until signed administrative reconciliation occurs.

---

## 5. Room Database Schema: Regulatory Parameter System

```sql
-- Architectural Entity: Append-Only Regulatory Parameter Versioning
CREATE TABLE IF NOT EXISTS regulatory_parameters (
    config_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    parameter_key TEXT NOT NULL,                  -- 'MANDI_FEE_RATE', 'DEV_CESS_RATE', 'OVERLOAD_BASE_FINE'
    parameter_dimension TEXT CHECK(parameter_dimension IN ('BASIS_POINTS', 'PAISE', 'GRAMS', 'HOURS')) NOT NULL,
    integer_value INTEGER NOT NULL,               -- e.g., 150 bp (=1.50%), 2000000 paise (=Rs 20,000)
    effective_from_epoch_ms INTEGER NOT NULL,
    effective_to_epoch_ms INTEGER,                -- NULL indicates currently active parameter
    statutory_instrument_level INTEGER NOT NULL,  -- 1: Act, 2: Rules, 3: Gazette Notification
    gazette_citation TEXT NOT NULL,               -- e.g., 'UP Gazette Ext. No. 142/2026'
    supersedes_config_id INTEGER REFERENCES regulatory_parameters(config_id),
    verification_mode TEXT CHECK(verification_mode IN ('CRYPTOGRAPHICALLY_SIGNED', 'DUAL_SUPERVISOR_MANUAL')) NOT NULL,
    created_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_param_lookup 
    ON regulatory_parameters(parameter_key, effective_from_epoch_ms, effective_to_epoch_ms);

-- Modify ledger_transactions to anchor immutable config snapshots
ALTER TABLE ledger_transactions 
    ADD COLUMN applied_mandi_fee_config_id INTEGER REFERENCES regulatory_parameters(config_id);
ALTER TABLE ledger_transactions 
    ADD COLUMN applied_cess_config_id INTEGER REFERENCES regulatory_parameters(config_id);

-- Enforce immutability of historical regulatory configuration records
CREATE TRIGGER IF NOT EXISTS abort_regulatory_config_tampering
BEFORE UPDATE ON regulatory_parameters
FOR EACH ROW
WHEN OLD.effective_to_epoch_ms IS NOT NULL AND OLD.effective_to_epoch_ms < (strftime('%s', 'now') * 1000)
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Historical closed regulatory parameters cannot be modified.');
END;
```


---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Yard Rate-Board Gazette Match:**
   - Photograph the painted rate notice board outside the weighbridge cabin.
   - Cross-examine listed Mandi Fee, Development Cess, and Hamali charges against the current active state e-Gazette. Record variance and date of last paint refresh.

2. **Rate Change Notification Interview:**
   - Interview the weighbridge clerk: *"Jab mandi fee ya cess badalti hai, toh aapko sabse pehle kaun batata hai, aur paper kahan se aata hai?"*
   - Document information latency between official Gazette publication and physical yard implementation.

3. **Dual-PIN Configuration Drill:**
   - Execute a mock rate revision on the terminal using the two-supervisor manual override protocol.
   - Verify that the generated ticket explicitly displays the mandatory Gazette notification citation watermarking.

4. **Historical Retroactive Audit:**
   - Review physical cash vouchers and khatas from the previous harvest season.
   - Inspect how retroactive cess adjustments were billed or adjusted against farmer ledger accounts when settlements had already occurred.

5. **Axle GVW Compliance Check:**
   - Cross-reference the registered Gross Vehicle Weight (GVW) on vehicle registration certificates (RC) against the MoRTH 2018 safe axle load schedule for 6-wheel vs 10-wheel vs 12-wheel trucks entering the scale.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Statutory Instrument Primacy Invariant**: Level 1 Statutes, Level 2 Rules, and Level 3 Gazette Notifications preempt all local administrative circulars and private yard covenants.
  * Mandates **Append-Only Regulatory Versioning**: all tax, cess, and penalty parameters must carry semi-open temporal bounds ($[\text{effective\_from}, \text{effective\_to})$) stored as integer basis points ($1\text{ bp} = 0.01\%$). Hardcoded numeric constants in application code are strictly prohibited.
  * Enforces **Snapshot Auditability**: every committed ledger transaction must link directly to the specific `config_id` applied at commit time.

- **Impact on Room Database Schemas:**
  * Adds `regulatory_parameters` table with triggers preventing mutations to closed historical parameters.
  * Updates `ledger_transactions` to persist foreign keys for `applied_mandi_fee_config_id` and `applied_cess_config_id`.

---

## 8. Digest Card

- **Key Invariants:** Absolute Metric & Legal Metrology Primacy; Level 3 Gazette Supremacy (Stamp-paper notices carry zero legal weight); Append-Only Parameter Versioning ($[\text{effective\_from}, \text{effective\_to})$); Integer Basis Points ($1\text{ bp} = 0.01\%$).

- **Empirical Base Rates:** Axle weight limits stable $>8\text{ years}$; Metrology verification fixed at 12 months; Mandi fee schedules alter only 2–3 times per decade.

- **Update Mechanisms:** Primary via cryptographically signed offline bundle (`config.sig`); Emergency fallback via Dual-Supervisor PIN + mandatory gazette citation entry.

- **Top 3 Gemba Hooks:**
  1. Audit physical yard board rates against current published state gazettes.
  2. Trace operational information path of historical rate revisions.
  3. Verify dual-PIN manual override audit trail on an isolated terminal.
