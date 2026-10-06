# Multi-Stakeholder Collusion, Micro-Corruption Game Theory & Non-Cooperative Mechanism Design
Path: core/substrate/COLLUSION_GAME_THEORY.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, core/formal/QUEUE_COLLAPSE_MODEL.md, core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md

## 1. The Quad-Actor Payoff Matrix & Collusion Topologies

Commercial weighbridge yards are non-cooperative micro-economic environments. Four distinct rational actors interact across every transaction, each attempting to maximize a distinct utility function:

```text
                  ┌────────────────────────────────────────┐
                  │          THE WEIGHBRIDGE PLATFORM      │
                  └───────────────────┬────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│  CONSIGNOR       │        │  ARHATIYA /      │        │  TRANSPORTER /   │
│  (Farmer/Seller) │        │  COMMERCIAL BUYER│        │  TRUCK DRIVER    │
├──────────────────┤        ├──────────────────┤        ├──────────────────┤
│ Utility Function:│        │ Utility Function:│        │ Utility Function:│
│ Maximize Net kg  │        │ Minimize Payout  │        │ Maximize Speed,  │
│ Minimize Cuts    │        │ Maximize Margin  │        │ Exploit Tare Diff│
└────────┬─────────┘        └────────┬─────────┘        └────────┬─────────┘
         │                           │                           │
         └───────────────────────────┼───────────────────────────┘
                                     │
                                     ▼
                        ┌──────────────────────────┐
                        │   WEIGHBRIDGE CLERK      │
                        │   (Scale Operator)       │
                        ├──────────────────────────┤
                        │ Utility Function:        │
                        │ Maximize Informal Rent,  │
                        │ Minimize Work & Conflict │
                        └──────────────────────────┘
```


### 1.1 Actor Objective Functions & Payoff Vectors

| Actor Class | Primary Economic Objective | Operational Loss Vector | Common Collusive Alliances |
| :--- | :--- | :--- | :--- |
| **Consignor (Farmer)** | Gross mass maximization; zero deduction. | Vulnerable to arbitrary *Dhalta* / *Karda* skims and flat bag tare padding. | Allied with honest drivers against predatory brokers. |
| **Buyer (Arhatiya/Mill)** | Procurement cost minimization; bulk margin. | Loses if inbound gross is under-weighed by rogue supplier-driver pacts. | Allies with Scale Clerk to institutionalize customary yard deductions. |
| **Transporter (Driver)** | Throughput velocity; hauling fee; fuel arbitrage. | Loses time in queues; penalized for transport transit loss (*Ghaat*). | Allies with Scale Clerk to manipulate truck tare (phantom weight). |
| **Weighbridge Clerk** | Personal income supplement; physical self-preservation. | Vulnerable to operator replacement, physical violence, and vigilance raids. | Sells discretionary overrides (frozen tares, unverified locks) for cash. |

---

## 2. Mathematical Modeling of Micro-Corruption Vectors

### 2.1 The Repeat-Game Collusion Equilibrium
In a rural yard, actors interact repeatedly across a multi-month harvesting season ($N \to \infty$). According to the Folk Theorem for repeated games, a collusive strategy profile is sustainable as a sub-game perfect Nash equilibrium if the discount factor $\delta$ satisfies:

$$\delta \ge \frac{C_{\text{defect}}}{B_{\text{collude}} + C_{\text{defect}}}$$

Where:
- $B_{\text{collude}}$: The per-transaction financial rent extracted via weight manipulation (typically ₹500–₹2,000 split among conspirators).
- $C_{\text{defect}}$: The immediate one-shot payoff from honest reporting or whistleblowing.

Because the historical probability of detection ($P_{\text{audit}}$) with disposable paper slips was effectively zero ($P_{\text{audit}} \to 0$), the expected penalty was negligible:
$$\mathbb{E}[\text{Penalty}] = P_{\text{audit}} \times \text{Sanction} \approx 0$$
Hence, collusion was the mathematically dominant rational strategy.

### 2.2 Collusion Strategy Invalidation Condition
The native instrument shifts the game equilibrium by altering detection physics. The software achieves **Collusion Invalidation** if and only if:

$$\mathbb{E}[\text{Penalty}] = P_{\text{detection}}(\text{Cryptographic \& Optical Trail}) \times \text{Sanction} > B_{\text{collude}}$$

By binding every transaction to:
1. Continuous 20 Hz telemetry jerk analysis ($P_{\text{relay\_detect}} \approx 0.95$),
2. Mandatory camera optical photo-locks ($P_{\text{display\_detect}} \approx 0.98$),
3. Out-of-band cryptographic proof delivery to the absent principal,

The probability of detection approaches $1.0$, rendering the expected payoff of fraud negative:
$$\mathbb{E}[\text{Utility}_{\text{fraud}}] = B_{\text{collude}} - (1.0 \times \text{Legal / Economic Termination}) \ll 0$$

---

## 3. The Three Canonical Yard Fraud Mechanics & Algorithmic Countermeasures

### 3.1 Fraud Mechanism 1: The Asymmetric Tare Water/Fuel Arbitrage
- **The Physical Exploit:** Multi-axle commercial trucks possess dual fuel tanks ($300\text{--}400\,\text{L}$ capacity) or aftermarket auxiliary water tanks ($200\text{--}500\,\text{L}$) equipped with fast-drain dump valves.
  1. *Pass 1 (Gross Inbound):* Truck mounts scale with full tanks ($+500\,\text{kg}$ of fluid). Gross weight recorded as $35,500\,\text{kg}$.
  2. *Discharge:* Truck unloads grain inside depot, simultaneously opening dump valves into the yard drainage pit.
  3. *Pass 2 (Tare Outbound):* Truck mounts scale with empty tanks. Tare recorded as $10,000\,\text{kg}$ (instead of true baseline $10,500\,\text{kg}$).
  4. *The Skim:* Net weight recorded as $25,500\,\text{kg}$. The buyer pays for $500\,\text{kg}$ of water/fuel priced as Grade-A Wheat ($\approx ₹12{,}125$ unearned cash).
- **Algorithmic Defense (Historical Tare Drift Watchdog):**
  The native app queries historical tare profiles for the vehicle registration (`UP-34-AT-xxxx`).
  $$\Delta \text{Tare} = \vert{}\text{Tare}_{\text{current}} - \text{Median}(\text{Tare}_{\text{historical}})\vert{}$$
  $$\mathbf{\text{FLAG ANOMALY IF: }} \Delta \text{Tare} > 150\,\text{kg} \quad \text{AND} \quad \text{Confidence}_{\text{samples}} \ge 3$$
  The transaction commits with an amber `TARE_DRIFT_SUSPECT` watermark, triggering a mandatory visual cab/chassis photo requirement.

### 3.2 Fraud Mechanism 2: The Cab Occupancy & Helper Dismount Skim
- **The Physical Exploit:** A truck cab carries the driver and two yard helpers (combined human weight: $180\text{--}240\,\text{kg}$).
  1. *Inbound Weighing:* All three occupants sit in the cab $\to$ Gross inflated by $220\,\text{kg}$.
  2. *Outbound Weighing:* Driver instructs helpers to wait outside on the approach ramp $\to$ Tare deflated by $150\,\text{kg}$.
  3. *The Skim:* An extra $150\text{--}220\,\text{kg}$ of phantom commodity mass is credited.
- **Algorithmic Defense (Camera2 Cab Occupancy Snapshot):**
  Upon firing transition $T_4$ (Lock Weight), the Android terminal triggers the front/rear camera module to capture an optical evidence photo of the vehicle cab windshield. The image hash is committed to `electronic_evidence_ledger`. Discrepancies in passenger occupancy are forensically provable during dispute audits under BSA 2023 Section 63.

### 3.3 Fraud Mechanism 3: The Customary *Dhalta* / *Bardana* Skim
- **The Physical Exploit:** The commission agent applies an arbitrary verbal deduction ($1.5\,\text{kg}$ per quintal) claiming "grain moisture loss," while charging a flat $1.0\,\text{kg}$ bag tare on lightweight $130\,\text{g}$ HDPE sacks.
- **Algorithmic Defense (Strict Separation of Physical vs. Commercial Ledgers):**
  The database schema physically prohibits subtracting commercial rebates from the certified metrological net mass:
  $$\text{NetMass}_{\text{certified}} = \text{GrossMass} - \text{TareMass} - (N_{\text{bags}} \times \text{CodifiedBagTare})$$
  Any negotiated commercial discount is recorded strictly as a **Financial Price Deduction in Integer Paise**, never as a reduction in physical commodity mass. The printed slip outputs both values distinctly, preventing brokers from concealing price cuts behind altered weight figures.

---

## 4. The Coercion-Safe Duress Protocol (Anti-Intimidation)

When local yard cartels or aggressive transport mobs physically threaten the scale operator to force an unauthorized ticket clearance, software that locks up completely endangers the operator's physical safety.

### 4.1 The Silent Duress PIN
The native application provides a dual-PIN architecture:
1. **Standard Operator PIN (`1234`):** Commits transactions normally under nominal validation rules.
2. **Duress PIN (`9999` / Inverted PIN):** Entered by the operator under physical coercion or threat of violence.

### 4.2 Duress Execution Mechanics
When the Duress PIN is entered:
- **UI Behavior:** The screen mimics standard success: plays the standard completion chime, displays a green success confirmation, and prints the physical ticket immediately without error dialogs.
- **Database Engine State:**
  1. The transaction is flagged in SQLite with `is_coerced_duress = 1`.
  2. The transaction integrity level is automatically downgraded: `integrity_level = 'TIL_1'`.
  3. The internal commercial settlement lock is permanently set: `is_commercial_settlement_blocked = 1`.
  4. The camera silently captures an ambient burst of 3 frames from the front-facing camera.
  5. During the next ad-hoc P2P anti-entropy exchange (Aspect 05), the duress alert propagates silently to the Master Device and administrative ledger.

---

## 5. Room Database Schema Extensions

To track stakeholder collusion forensics, tare drift histories, and duress events, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Vehicle Historical Tare Baselines
CREATE TABLE IF NOT EXISTS vehicle_tare_baselines (
    vehicle_registration TEXT PRIMARY KEY NOT NULL,
    median_tare_mass_grams INTEGER NOT NULL,
    min_observed_tare_grams INTEGER NOT NULL,
    max_observed_tare_grams INTEGER NOT NULL,
    sample_count INTEGER NOT NULL DEFAULT 1,
    last_updated_epoch_ms INTEGER NOT NULL
);

-- Architectural Extension: Logging Collusion and Duress Forensics
CREATE TABLE IF NOT EXISTS security_collusion_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    transaction_id INTEGER NOT NULL REFERENCES ledger_transactions(ledger_id),
    collusion_type TEXT CHECK(collusion_type IN (
        'TARE_DRIFT_THRESHOLD_EXCEEDED', 
        'CAB_OCCUPANCY_MISMATCH_SUSPECT', 
        'COMMERCIAL_MASS_SKIM_ATTEMPT', 
        'COERCED_DURESS_OVERRIDE'
    )) NOT NULL,
    observed_delta_grams INTEGER NOT NULL,
    operator_actor_id TEXT NOT NULL,
    duress_silent_alarm INTEGER NOT NULL DEFAULT 0,
    optical_cab_blob_id TEXT,
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_collusion_tx 
    ON security_collusion_events(transaction_id);

-- Modify ledger_transactions to anchor collusion and duress flags
ALTER TABLE ledger_transactions 
    ADD COLUMN is_coerced_duress INTEGER NOT NULL DEFAULT 0;
ALTER TABLE ledger_transactions 
    ADD COLUMN is_commercial_settlement_blocked INTEGER NOT NULL DEFAULT 0;

-- Trigger: Automatically block commercial settlement on coerced duress records
CREATE TRIGGER IF NOT EXISTS enforce_duress_settlement_block
AFTER INSERT ON ledger_transactions
FOR EACH ROW
WHEN NEW.is_coerced_duress = 1
BEGIN
    UPDATE ledger_transactions 
    SET is_commercial_settlement_blocked = 1 
    WHERE ledger_id = NEW.ledger_id;
END;
```


---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Fluid Dump Auxiliary Tank Audit:**
   - *Procedure:* Shadow 5 multi-axle commercial trucks carrying scrap or agricultural produce. Inspect the chassis between the cab and cargo bed for secondary fuel tanks, water drums, or bottom-discharge ball valves.
   - *Evaluation Criteria:* Document frequency of auxiliary tanks and verify whether drivers drain fluids inside the unloading bay.
2. **Passenger Cab Occupancy Drift Test:**
   - *Procedure:* Record cab occupancy on inbound gross weighing (count driver + helpers). Observe whether passengers remain in the vehicle during outbound tare weighing.
   - *Evaluation Criteria:* Measure the observed tare mass delta when helpers dismount ($\approx 60\text{--}180\,\text{kg}$).
3. **The Duress Silent PIN Trigger Drill:**
   - *Procedure:* Execute a simulated robbery or coercion scenario on the scale terminal. Enter the Duress PIN (`9999`).
   - *Evaluation Criteria:* Verify that the terminal emits a physical paper receipt without delay, avoids raising an on-screen alarm, logs `is_coerced_duress = 1` in SQLite, and locks commercial trade settlement.
4. **Historical Tare Anomaly Detection Assay:**
   - *Procedure:* Submit 3 weighment passes for a test truck with a true tare of $8,000\,\text{kg}$. On the 4th pass, mount the vehicle with an artificial $500\,\text{kg}$ weight attached.
   - *Evaluation Criteria:* Verify that the application flags `TARE_DRIFT_THRESHOLD_EXCEEDED` and prompts for secondary supervisory authorization.
5. **Dhalta / Bardana Slip Transparency Audit:**
   - *Procedure:* Review 20 issued tickets printed by the system.
   - *Evaluation Criteria:* Verify that certified metrological net weight is completely decoupled from financial deductions, and that bag tare is deducted based on actual packaging type (IS 1943 vs IS 12650).

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Non-Cooperative Stakeholder Invariant**: system architecture assumes zero trust between the four operational actors; all physical measurements must be anchored cross-modally.
  * Mandates the **Coercion-Safe Duress Protocol**: operators facing physical threats must have an escape path that preserves physical safety while cryptographically quarantining the resulting record.
  * Prohibits **Mass-Skimming Deductions**: customary yard cuts (*Dhalta*, *Karda*) cannot alter certified mass; all trade rebates must be represented strictly as financial discounts in integer paise.
- **Impact on Room Database Schemas:**
  * Adds `vehicle_tare_baselines` and `security_collusion_events` tables.
  * Implements `enforce_duress_settlement_block` database trigger.

---

## 8. Digest Card

- **Key Invariants:** Quad-Actor Non-Cooperative Game (Consignor, Buyer, Driver, Clerk); Folk Theorem Collusion Neutralization ($P_{\text{detect}} \times \text{Penalty} > \text{Rent}$); Tare Baseline Drift Tracking; Decoupling Physical Net Mass from Financial Price Deductions; Silent Coercion Duress Path.
- **Dominant Fraud Vectors:** (1) Auxiliary Fuel/Water Tank Draining, (2) Passenger Cab Dismount Skims, (3) Customary *Dhalta* / *Bardana* Skimming, (4) Direct Operator Intimidation.
- **Defensive Solutions:** Historical tare median anomaly detection ($\pm 150\,\text{kg}$ threshold); Camera optical cab occupancy capture; Duress PIN (`9999`) with silent settlement lock; Out-of-band principal notifications (WhatsApp/SMS).
- **Top 3 Gemba Hooks:**
  1. Inspect truck chassis for hidden fluid dump valves and water ballasts.
  2. Measure passenger dismount weight variations between gross and tare passes.
  3. Validate silent duress PIN execution and database settlement quarantine.
