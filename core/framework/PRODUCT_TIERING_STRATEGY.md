# Product Line Architecture, Tiering Strategy & Commercial Packaging
Path: core/framework/PRODUCT_TIERING_STRATEGY.md
Status: ACTIVE / STRATEGIC FRAMEWORK
Dependencies: core/framework/CONSUMER_DECISION_MATRIX.md, core/formal/QUEUE_COLLAPSE_MODEL.md, core/substrate/HARDWARE_ATTACK_SURFACE.md

---

## 1. Product Line Structure: From Farmgate to Industrial Siding

Rather than a monolithic application, the software architecture decomposes into three distinct commercial product tiers targeting specific operational environments:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                      PRODUCT LINE SPECIFICATION MATRIX                 │
├────────────────────┬────────────────────┬──────────────────────────────┤
│ TIER 1: LITE       │ TIER 2: PROFESSIONAL│ TIER 3: ENTERPRISE           │
│ (Farmgate Mobile)  │ (Mandi Commercial) │ (Industrial Hub & Siding)    │
├────────────────────┼────────────────────┼──────────────────────────────┤
│ Target Market:     │ Target Market:     │ Target Market:               │
│ • Village traders  │ • APMC Mandis      │ • Sugar mills, cement plants │
│ • FPO procurement  │ • Private scales   │ • Mining & railway sidings   │
│ • Rural collectors │ • Grain commission │ • Port container terminals   │
├────────────────────┼────────────────────┼──────────────────────────────┤
│ Hardware Substrate:│ Hardware Substrate:│ Hardware Substrate:          │
│ • Android Phone/Tab│ • Rugged 10" POS   │ • Multi-core IPC / Edge Box  │
│ • Bluetooth ESC/POS│ • RS-232 Serial    │ • 2x RTSP ANPR Cameras       │
│ • Manual / BLE load│ • Split Cash Drawer│ • FASTag UHF RFID Reader     │
│   cells (≤ 5 tonne)│ • High-Speed Th-Prn│ • Dual Boom Barrier Relays   │
├────────────────────┼────────────────────┼──────────────────────────────┤
│ Processing Budget: │ Processing Budget: │ Processing Budget:           │
│ • τ_app ≤ 15.0 s   │ • τ_app ≤ 8.0 s    │ • τ_app ≤ 5.0 s (Fast-Path)  │
├────────────────────┼────────────────────┼──────────────────────────────┤
│ Price & Model:     │ Price & Model:     │ Price & Model:               │
│ • ₹2,999 - ₹4,999  │ • ₹14,999 - ₹24,999│ • ₹75,000 - ₹1,50,000        │
│   perennial / app  │   turnkey bundle   │   turnkey deployment         │
└────────────────────┴────────────────────┴──────────────────────────────┘
```

### 1.1 Operational Deployment Profiles

- **Tier 1 (Lite / Farmgate Mobile):**
  - **Environment:** Open farmgate mud tracks, tractor trolleys, village aggregators, and farmer producer organization (FPO) collection centers.
  - **Constraints:** Battery-powered mobile phones or budget tablets subject to direct sunlight ($>10,000\text{ lux}$) and dirty-hand handling. Manual weight entry or Bluetooth low-energy platform scales ($\le 5\text{ tonne}$).
  - **Primary Metric:** Low initial capital barrier, lightweight footprint, and zero cloud dependency.

- **Tier 2 (Professional / Mandi Commercial):**
  - **Environment:** High-stress APMC mandi weighbridge cabins, private standalone commercial weighbridges (*Dharam Kanta*), and regional commodity trade yards.
  - **Constraints:** Continuous 14-hour operational shifts, severe electrical line surges, acoustic noise ($>80\text{ dB}$), and intense queue pressure.
  - **Hardware:** Rugged 10-inch Android terminal or industrial tablet coupled directly to standard digital indicators via an opto-isolated RS-232 serial cable, controlling an automated dual-lock cash drawer and high-speed thermal slip printer.

- **Tier 3 (Enterprise / Industrial Siding):**
  - **Environment:** Heavy manufacturing sites, 24/7 continuous sugar mill crushing yards, mining bulk dispatches, and intermodal freight terminals.
  - **Constraints:** Extreme vehicle throughput requiring automated fleet processing, automated boom gate control, automated dual ANPR cameras, and FASTag UHF RFID correlation.
  - **Hardware:** Multi-core industrial PC (IPC) or ruggedized Android edge controller with hardware NPU inference and GPIO relay outputs.

---

## 2. Feature Gating & Architectural Decoupling

The codebase uses modular compilation flavors and clean architectural boundaries to prevent piracy while allowing seamless tier upgrades without database schema changes:

```kotlin
// Tactical Feature-Gating Matrix (Kotlin Build Flavors / Modular Capabilities)

package com.instrument.core.framework

enum class SystemCapability {
    MANUAL_BLUETOOTH_PRINTING,
    LOCAL_SQLITE_CIPHER,
    OFFLINE_SNEAKERNET_SYNC,
    RS232_DIRECT_STREAM_LOCK,
    SPLIT_CASH_DRAWER_LEDGER,
    STATUTORY_AUDITOR_MODE,
    OFFLINE_TELEPHONY_OTP,
    ON_DEVICE_NPU_ANPR,
    FASTAG_RFID_INTEGRATION,
    GPIO_BARRIER_INTERLOCK,
    MERKLE_EPOCH_LONGTERM_WORM
}

enum class ProductTier(val maxDailyTonnage: Int, val maxConcurrentScales: Int) {
    TIER_1_LITE(maxDailyTonnage = 50, maxConcurrentScales = 1),
    TIER_2_PRO(maxDailyTonnage = 1_000, maxConcurrentScales = 2),
    TIER_3_ENTERPRISE(maxDailyTonnage = 50_000, maxConcurrentScales = 8);

    fun isFeatureSupported(feature: SystemCapability): Boolean = when (feature) {
        SystemCapability.MANUAL_BLUETOOTH_PRINTING   -> true
        SystemCapability.LOCAL_SQLITE_CIPHER        -> true
        SystemCapability.OFFLINE_SNEAKERNET_SYNC    -> true

        SystemCapability.RS232_DIRECT_STREAM_LOCK   -> this >= TIER_2_PRO
        SystemCapability.SPLIT_CASH_DRAWER_LEDGER   -> this >= TIER_2_PRO
        SystemCapability.STATUTORY_AUDITOR_MODE     -> this >= TIER_2_PRO
        SystemCapability.OFFLINE_TELEPHONY_OTP      -> this >= TIER_2_PRO

        SystemCapability.ON_DEVICE_NPU_ANPR         -> this == TIER_3_ENTERPRISE
        SystemCapability.FASTAG_RFID_INTEGRATION    -> this == TIER_3_ENTERPRISE
        SystemCapability.GPIO_BARRIER_INTERLOCK     -> this == TIER_3_ENTERPRISE
        SystemCapability.MERKLE_EPOCH_LONGTERM_WORM -> this == TIER_3_ENTERPRISE
    }
}
```

---

## 3. Air-Gapped Licensing & Offline Monetization Mechanics

Because rural terminals run without continuous internet, standard Google Play In-App Billing or cloud license pings fail completely. The system enforces an **Air-Gapped Cryptographic Licensing Lifecycle**:

```text
[CENTRAL VENDOR DESK] (Connected)
          │
          ├── Generates Cryptographic License Stub:
          │   Payload: { Device_Hardware_ID, Tier, Expiry_Epoch, Max_Tickets }
          │   Signed: Ed25519 Private Master Key
          ▼
[CUSTOMER PHONE / SNEAKERNET USB]
          │
          ├── Received via WhatsApp text or QR Code image
          ▼
[AIR-GAPPED WEIGHBRIDGE TERMINAL]
          │
          ├── Reads License QR code via onboard camera or manual 24-character token
          ├── Validates Ed25519 signature against embedded Master Public Key
          ├── Verifies SoC Hardware ID matches onboard device
          ▼
┌────────────────────────────────────────────────────────┐
│ LICENSE ACTIVATED / EXTENDED FOR 365 DAYS:             │
│ • Local monotonic counter increments                   │
│ • Anti-rollback clock interlock latched                │
│ • Zero cloud pings required for full year              │
└────────────────────────────────────────────────────────┘
```

### 3.1 Cryptographic Anti-Rollback Verification

1. **Hardware Fingerprint Binding:** The terminal calculates a non-volatile device hash from the SoC serial number, eMMC CID, and hardware keystore root key. License tokens generated for Device $A$ fail signature verification if imported into Device $B$.
2. **Monotonic Sequence Enforcement:** To prevent time-rollback fraud (operators rolling system clocks back to bypass expiration), the terminal increments an internal monotonically increasing transaction sequence counter on every write. If system clock timestamp $T_{\text{now}} < T_{\text{last\_committed}}$, the terminal enters a degraded read-only audit state until re-authorized by a signed challenge token.
3. **Grace Period Dynamics:** In the event of annual license expiration during peak harvest, the terminal grants a 7-day emergency grace period ($168\text{ hours}$) with prominent visual countdown indicators on printed thermal tickets, preventing sudden physical stoppage while notifying the owner to contact the local technician.

---

## 4. Economic Defensibility & Customer Lock-In Moats

1. **Historical Ledger Weight Data (High Switching Cost):**
   - Once an owner accumulates 2+ years of farmer settlement ledgers, credit *khata* balances, moisture deduction records, and statutory tax audits locked in the local SQLite database, migrating to a competing software product carries unacceptable administrative risk and bookkeeping disruption.
   - The relational complexity of cross-season credit offsets binds the trading yard to the core engine.

2. **Hardware Calibration Integration:**
   - The software directly interfaces with RS-232 serial streams from legacy digital indicators (Essae, Avery India, Eagle, CAS, Sartorius, Cardinal).
   - Generic software lacking battle-tested continuous streaming parser decoders, baud rate auto-negotiation, and electrical surge recovery fails to operate reliably in dusty cabins.

3. **The Local Mechanic (Mistri) Ecosystem:**
   - Scale maintenance mechanics receive a recurring revenue share (15–20%) on every annual license renewal voucher sold to the yard owner.
   - Converting hardware field technicians into an exclusive sales and tier-1 support force creates an insurmountable local distribution moat against outside SaaS vendors.
