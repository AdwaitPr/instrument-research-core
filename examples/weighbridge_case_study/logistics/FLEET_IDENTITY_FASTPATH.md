# Fleet / Carrier Identity, Telephony Fast-Path & Vehicle Authentication
Path: core/logistics/FLEET_IDENTITY_FASTPATH.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/QUEUE_COLLAPSE_MODEL.md, core/substrate/COLLUSION_GAME_THEORY.md, Central Motor Vehicles Rules (CMVR) 1989 (Rule 50), EPC Gen2 (ISO 18000-6C)

## 1. Multi-Tier Fleet Topology & Identity Verification Hostility

In Indian bulk logistics (mandi procurement, cement dispatch, sugar mills, mining sidings), vehicle identification is complicated by diverse carrier ownership structures, defaced physical license plates, and intermittent cellular connectivity:

```text
                  ┌────────────────────────────────────────┐
                  │       YARD INBOUND TRAFFIC STREAM      │
                  └───────────────────┬────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│ CAPTIVE DEDICATED│        │ CONTRACTED LOGIS-│        │ SPOT-MARKET      │
│ MILL FLEET       │        │ TICS CARRIERS    │        │ SINGLE-TRUCK OWNER│
├──────────────────┤        ├──────────────────┤        ├──────────────────┤
│ Profile:         │        │ Profile:         │        │ Profile:         │
│ • Factory-owned  │        │ • Multi-year tie │        │ • Independent    │
│ • Known tare     │        │ • Fixed route    │        │ • Variable tare  │
│ • Fixed RFID tag │        │ • FastTag bound  │        │ • Feature phone  │
│ • Closed circuit │        │ • Bi-weekly tare │        │ • Cash payment   │
└────────┬─────────┘        └────────┬─────────┘        └────────┬─────────┘
         │                           │                           │
         └───────────────────────────┼───────────────────────────┘
                                     │
                                     ▼
                        ┌──────────────────────────┐
                        │ WEIGHBRIDGE GATE RAMP    │
                        │ Target Processing Time:  │
                        │ τ_app ≤ 6s (Fast-Path)   │
                        │ τ_app ≤ 15s (Spot Entry) │
                        └──────────────────────────┘
```

### 1.1 Indian Registration Plate Linguistic & Syntactic Fragmentation

The vehicle identity ingestion engine must deterministically parse and normalize four distinct Indian vehicle registration syntaxes:

- **Standard State Format (CMVR Rule 50):** `[State Code 2A][RTO Code 2N][Series 1-3A][Registration Number 4N]` (e.g., `UP 32 BN 4521`, `HR 55 T 8802`).
- **Bharat Series (BH Series, 2021 Norms):** `[Year 2N][BH 2A][Registration 4N][Series 1-2A]` (e.g., `22 BH 1421 AB`).
- **Legacy Pre-2001 Formats:** Missing district leading zeroes or non-standard series (e.g., `UPB 4122`, `DL 1G 9021`).
- **Devanagari / Regional Script Plates:** Handwritten vernacular registrations (e.g., `यू.पी. ३२ बी.एन. ४५२१`), requiring canonical ASCII transliteration before database indexing.

---

## 2. Telephony-Based Offline Authentication & Driver Tokenization

Because scale cabins operate under frequent cellular brownouts (Aspect 19), vehicle identity verification cannot rely on live cloud API lookups (e.g., live VAHAN portal queries). The instrument implements an air-gapped telephony pre-authorization protocol:

```text
[UPSTREAM DISPATCH / FARMER HUB] (Network Available)
                 │
                 ├── 1. Issues E-Challan / Dispatch Gate Pass
                 ├── 2. Computes HMAC Token: Token = Truncate6( HMAC_SHA256(Key, Reg + Date) )
                 ▼
[DRIVER CELLULAR TERMINAL]
                 │
                 ├── Receives Token via SMS or WhatsApp (e.g., "782-901")
                 ├── In zero-network areas: Token printed on paper dispatch slip
                 ▼
[AIR-GAPPED WEIGHBRIDGE SCALING RAMP] (Zero Cellular Connectivity)
                 │
                 ├── Operator enters Driver Phone (10 digits) + Token (6 digits)
                 ├── App validates Token against pre-shared root HMAC key
                 ▼
┌────────────────────────────────────────────────────────┐
│ FAST-PATH AUTHENTICATION RESULT:                       │
│ • Valid Token: Ingests Pre-Registered Consignor,       │
│   Expected Commodity, and Target Moisture Profile      │
│ • Elapsed Time: τ_auth ≤ 3.5 seconds                   │
└────────────────────────────────────────────────────────┘
```

### 2.1 Cryptographic Offline Challenge-Response Construction

Let $K_{\text{depot}}$ be the pre-shared symmetrical master key deployed to both upstream procurement centers and local scale terminals via secure sneakernet sync (Aspect 05).

$$\text{Seed} = \text{RegNumber}_{\text{normalized}} \parallel \text{DriverPhone}_{\text{10-digit}} \parallel \text{EpochDay}$$
$$\text{FullHMAC} = \text{HMAC-SHA256}(K_{\text{depot}}, \text{Seed})$$
$$\text{AuthToken} = \text{ExtractDecimal}(\text{FullHMAC}, \text{offset}=0) \pmod{10^6}$$

The 6-digit numeric token is valid for a single calendar day (`EpochDay`). The local SQLite database validates the token mathematically without initiating an outbound network packet.

---

## 3. Fast-Path Tare Caching & Expiry Lifecycle

Under peak harvesting queue pressure (Aspect 03), measuring both gross and tare on every single yard cycle drives system utilization $\rho \to 1.0$, inducing queue collapse. For trusted captive and contracted fleets, the system enables Fast-Path Cached Tare Weighment.

```text
                                [INBOUND TRUCK ARRIVAL]
                                           │
                                           ▼
                            [IS TRUCK IN FLEET REGISTRY?]
                                           │
                        ┌──────────────────┴──────────────────┐
                        │ YES                                 │ NO
                        ▼                                     ▼
           [IS CACHED TARE WITHIN WINDOW?]           [MANDATE FULL DUAL-PASS]
           (Age ≤ 24 Hours && Trust ≥ Level 2)        1. Inbound Gross Pass
                        │                             2. Unload Cargo
            ┌───────────┴───────────┐                 3. Outbound Tare Pass
            │ YES                   │ NO
            ▼                       ▼
    [FAST-PATH GROSS ONLY]   [MANDATE IMMEDIATE
    • Tare retrieved from     TARE UPDATE PASS]
      secure local cache.
    • Print net slip in 6s.
```

### 3.1 The Dynamic Tare Drift Gate

To prevent fraud through unrecorded vehicle modifications (e.g., spare tire removal, fuel tank draining as modeled in Aspect 06), cached tares are subjected to a Random Spot-Audit Invariant:

- A configurable random fraction ($P_{\text{audit}} \approx 10\text{--}15\%$) of fast-path vehicles are flagged for mandatory outbound tare verification.
- **Drift Tolerance:** If observed tare diverges from cached tare by more than 1.5% or 150 kg:

$$\Delta \text{Tare} = |\text{Tare}_{\text{measured}} - \text{Tare}_{\text{cached}}| > 150\,\text{kg}$$

The system revokes the carrier's fast-path privilege, triggers `FLEET_FASTPATH_TRUST_REVOKED`, and mandates full dual-pass weighment for the subsequent 10 trips.

---

## 4. Electronic Vehicle Identification: FASTag (EPC Gen2) & HSRP Correlation

To eliminate manual typing errors and prevent plate-swapping fraud, the terminal interfaces with passive Ultra-High Frequency (UHF) RFID scanners reading the mandatory Government of India FASTag transponder affixed to vehicle windshields.

### 4.1 Multi-Identifier Verification Matrix

A vehicle identity is certified at Integrity Level 3 (TIL-3) if and only if three physical and electronic identifiers correlate:

```text
┌─────────────────────────────────────────────────────────────────┐
│               THE TRI-IDENTIFIER CORRELATION MATRIX             │
├─────────────────────┬───────────────────────────────────────────┤
│ IDENTIFIER          │ ACQUISITION METHOD & PHYSICAL ATTRIBUTES  │
├─────────────────────┼───────────────────────────────────────────┤
│ 1. HSRP Plate       │ Optical camera OCR / manual high-contrast │
│                     │ entry of embossed alphanumeric characters │
├─────────────────────┼───────────────────────────────────────────┤
│ 2. FASTag EPC       │ 865–867 MHz UHF RFID read of 96-bit       │
│                     │ Electronic Product Code (EPC) Bank        │
├─────────────────────┼───────────────────────────────────────────┤
│ 3. Transponder TID  │ Factory-locked 64-bit Tag Identifier (TID)│
│                     │ silicon serial number (Non-cloneable)     │
└─────────────────────┴───────────────────────────────────────────┘
```

### 4.2 Anti-Cloning Silicon Interlock

Fraudulent fleets occasionally detach FASTags from light commercial vehicles and place them inside heavy multi-axle trucks to bypass toll or weighbridge class restrictions.

1. The instrument maintains an offline lookup table mapping registered vehicle chassis classes to certified unladen tare envelopes (e.g., 6-Wheel Tipper: $6,000\text{--}8,500\,\text{kg}$; 14-Wheel Hauler: $11,000\text{--}14,500\,\text{kg}$).
2. If an RFID scan returns a light vehicle class while the scale indicator measures gross mass $> 25,000\,\text{kg}$, the app immediately flags `RFID_CLASS_MISMATCH_ALARM` and locks the transaction.
---

## 5. Room Database Schema Extensions

To persist pre-registered carrier profiles, cached tare life-cycles, offline telephony challenge tokens, and FASTag silicon IDs, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Registered Fleet Vehicles & Cached Tares
CREATE TABLE IF NOT EXISTS fleet_registered_vehicles (
    vehicle_registration TEXT PRIMARY KEY NOT NULL,
    carrier_company_id TEXT NOT NULL,
    vehicle_axle_class TEXT CHECK(vehicle_axle_class IN (
        'LIGHT_COMMERCIAL_4W', 
        'MEDIUM_RIGID_6W', 
        'HEAVY_HAULER_10W_12W', 
        'MULTI_AXLE_TRAILER_14W_PLUS', 
        'TRACTOR_TROLLEY_RURAL'
    )) NOT NULL,
    cached_tare_mass_kg INTEGER,
    tare_cached_epoch_ms INTEGER,
    tare_trust_tier INTEGER NOT NULL DEFAULT 1,     -- 1 = Spot (Must Dual-Pass), 2 = Contracted, 3 = Captive Mill
    fastpath_tare_eligible INTEGER NOT NULL DEFAULT 0,
    rfid_fastag_epc_hex TEXT,
    rfid_silicon_tid_hex TEXT,
    is_active INTEGER NOT NULL DEFAULT 1
);

CREATE INDEX IF NOT EXISTS idx_fleet_carrier 
    ON fleet_registered_vehicles(carrier_company_id);

-- Architectural Extension: Tracking Offline Telephony HMAC Auth Tokens
CREATE TABLE IF NOT EXISTS offline_telephony_tokens (
    token_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    token_numeric_code TEXT NOT NULL,              -- 6-digit numeric PIN
    driver_phone_number TEXT NOT NULL,
    vehicle_registration TEXT NOT NULL,
    expected_commodity_code TEXT NOT NULL,
    token_expiry_epoch_ms INTEGER NOT NULL,
    is_redeemed INTEGER NOT NULL DEFAULT 0,
    redeemed_at_epoch_ms INTEGER
);

CREATE INDEX IF NOT EXISTS idx_token_lookup 
    ON offline_telephony_tokens(token_numeric_code, vehicle_registration);

-- Architectural Extension: Logging Random Spot-Audit Tare Verifications
CREATE TABLE IF NOT EXISTS fastpath_tare_audits (
    audit_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    vehicle_registration TEXT NOT NULL,
    cached_tare_mass_kg INTEGER NOT NULL,
    measured_tare_mass_kg INTEGER NOT NULL,
    drift_delta_kg INTEGER NOT NULL,               -- measured - cached
    audit_outcome TEXT CHECK(audit_outcome IN ('PASSED_WITHIN_TOLERANCE', 'FAILED_DRIFT_EXCEEDED')) NOT NULL,
    audit_timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Automatically revoke fast-path eligibility if tare drift exceeds 150 kg
CREATE TRIGGER IF NOT EXISTS revoke_fastpath_on_excessive_drift
AFTER INSERT ON fastpath_tare_audits
FOR EACH ROW
WHEN NEW.audit_outcome = 'FAILED_DRIFT_EXCEEDED'
BEGIN
    UPDATE fleet_registered_vehicles 
    SET fastpath_tare_eligible = 0,
        tare_trust_tier = 1 
    WHERE vehicle_registration = NEW.vehicle_registration;
END;
```

---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Air-Gapped HMAC Telephony OTP Validation Assay:**
   - *Procedure:* Disconnect the Android terminal completely from Wi-Fi and mobile data. Generate a 6-digit challenge token at an upstream terminal for vehicle `UP-32-BN-4521`. Input the token and phone number at the air-gapped scale terminal.
   - *Pass/Fail Criteria:* Verify that the terminal authenticates the token in $\tau_{\text{auth}} \le 3.5\text{ seconds}$, correctly populating carrier metadata without raising a network timeout.

2. **FASTag EPC Gen2 vs TID Silicon Anti-Cloning Drill:**
   - *Procedure:* Read an authentic windshield FASTag with an external UHF RFID reader. Clone the 96-bit EPC memory onto a writable test transponder without altering the factory-locked 64-bit TID bank. Present the clone to the weighbridge gate reader.
   - *Pass/Fail Criteria:* Verify that the application detects the mismatched silicon TID, flags `RFID_SILICON_CLONE_DETECTED`, and halts fast-path processing.

3. **Random Tare Spot-Audit Drift Revocation Simulation:**
   - *Procedure:* Register a fleet truck with a cached tare of $10,200\,\text{kg}$. Trigger a randomized spot audit ($P_{\text{audit}}$ forced to 100%). Drive the vehicle onto the scale with an added $300\,\text{kg}$ ballast ($\Delta\text{Tare} = +300\,\text{kg}$).
   - *Pass/Fail Criteria:* Verify that `fastpath_tare_audits` logs `FAILED_DRIFT_EXCEEDED`, the database trigger sets `fastpath_tare_eligible = 0`, and the terminal forces full dual-pass weighment on the next transaction.

4. **High-Throughput 6-Second Fast-Path Ramp Timing Test:**
   - *Procedure:* Execute 10 consecutive weighment transactions using pre-registered captive fleet trucks with cached tare profiles.
   - *Pass/Fail Criteria:* Measure cumulative software latency ($\tau_{\text{app}}$). Total elapsed time from platform stabilization to printed ticket issuance must not exceed $6.0\text{ seconds}$ per vehicle.

5. **Vernacular / BH-Series License Plate Normalization Assay:**
   - *Procedure:* Feed 20 test inputs covering standard state codes, BH series (`22 BH 1421 AB`), legacy plates (`UPB 4122`), and Devanagari script strings (`यू.पी. ३२ बी.एन. ४५२१`).
   - *Pass/Fail Criteria:* Confirm that all variations normalize into canonical ISO/IEC ASCII uppercase tokens and correctly query the corresponding SQLite database records.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Offline HMAC Telephony Invariant**: vehicle and driver pre-authorization must function deterministically without relying on real-time internet connectivity or external API availability.
  * Formally mandates the **Tri-Identifier Correlation Invariant**: high-trust automated transactions (TIL-3) require correlation between physical HSRP registration plates, 96-bit FASTag EPC, and 64-bit silicon TID memory banks.
  * Codifies the **Dynamic Tare Drift Gate**: cached tares are bounded by a $150\,\text{kg}$ drift ceiling; exceeding this threshold revokes fast-path bypass rights and enforces dual-pass weighments.
- **Impact on Room Database Schemas:**
  * Adds `fleet_registered_vehicles`, `offline_telephony_tokens`, and `fastpath_tare_audits` tables.
  * Implements `revoke_fastpath_on_excessive_drift` trigger to prevent collusion-based tare inflation.

---

## 8. Digest Card

- **Key Invariants:** Offline HMAC Telephony Challenge ($\tau_{\text{auth}} \le 3.5\,\text{s}$); Tri-Identifier Matrix (HSRP + EPC + TID); Dynamic Tare Drift Gate ($\Delta\text{Tare} \le 150\,\text{kg}$); Randomized Spot-Audit Invariant ($P_{\text{audit}} \approx 10\text{--}15\%$); Multi-Tier Fleet Partitioning (Captive, Contracted, Spot).
- **Registration Syntax Normalization:** Standard CMVR Rule 50, Bharat Series (BH), Legacy pre-2001, and Devanagari ASCII transliteration.
- **Operational Latency Ceilings:** Fast-path transaction budget $\tau_{\text{app}} \le 6.0\,\text{s}$; Spot-market entry budget $\tau_{\text{app}} \le 15.0\,\text{s}$.
- **Top 3 Gemba Hooks:**
  1. Verify air-gapped HMAC token resolution on disconnected Android tablet.
  2. Test cloned FASTag rejection via mismatched factory-locked 64-bit silicon TID.
  3. Validate automatic fast-path revocation when tare drift exceeds $150\,\text{kg}$.
