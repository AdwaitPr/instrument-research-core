# Split Cash Drawer Balancing, Asymmetric Float Ledger & Day-End Munim Reconciliation
Path: core/economics/CASH_DRAWER_SETTLEMENT.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md, Income Tax Act 1961 (S. 40A(3) & Rule 6DD)

## 1. The Asymmetric Mandi Cash Flow Architecture

Unlike consumer retail environments where cash flows into the register, agricultural procurement terminals and rural logistics depots operate in an **Outflow-Dominated Regime**:

```text
                  ┌────────────────────────────────────────┐
                  │        BANK / COMMERCIAL CHEST         │
                  └───────────────────┬────────────────────┘
                                      │ Inbound Float (Morning Infusion)
                                      ▼
                  ┌────────────────────────────────────────┐
                  │    BACK-OFFICE VAULT / TIJORI (Reserve)│
                  └───────────────────┬────────────────────┘
                                      │ Transfer Batches (₹50k - ₹2L)
                                      ▼
                  ┌────────────────────────────────────────┐
                  │     SCALE COUNTER FLOAT (Front Desk)   │
                  └───────────────────┬────────────────────┘
                                      │
         ┌────────────────────────────┴───────────────────────────┐
         │                                                         │
         ▼ (95% Outflow)                                           ▼ (5% Inflow)
┌──────────────────────────────────────┐  ┌──────────────────────────────────────┐
│  FARMER / DISBURSAL PAYOUTS          │  │  TARE FEES & WEIGHMENT CHARGES       │
│  • Net Settlement Cash               │  │  • Commercial Scale Fee (₹50 - ₹100) │
│  • Pre-Weighment Advances (Peshgi)   │  │  • Labor Charges Collected           │
│  • Bardana Packaging Buybacks        │  │  • Excess Cash Returns               │
└──────────────────────────────────────┘  └─────────────────────────────────────┘
```


### 1.1 Dual-Chest Physical Separation
To minimize robbery risk and employee shrinkage, cash is partitioned into two distinct physical and logical vaults:
1. **The Counter Float ($V_{\text{counter}}$):** Max liquidity cap: $₹1,00,000$. Kept at the scale operator's desk for immediate transaction disbursements.
2. **The Reserve Vault / Tijori ($V_{\text{vault}}$):** High-capacity safe ($₹5,00,000\text{--}₹25,00,000$) accessible only via Master PIN / Munim dual custody. Replenishes the counter float via logged internal transfer vouchers.

---

## 2. Statutory Compliance Boundary: Section 40A(3) vs. Rule 6DD(e)

The application enforces automatic tax compliance rules to prevent catastrophic income tax disallowances during commercial audits:

```text
                                [TRANSACTION INITIATED]
                                           │
                                           ▼
                       [IS COMMODITY STATUTORY AGRICULTURAL?]
                               (Wheat, Paddy, Mustard, Pulses)
                                           │
                        ┌──────────────────┴──────────────────┐
                        │ YES                                 │ NO (Scrap, Coal, Timber)
                        ▼                                     ▼
           [RULE 6DD(e) EXEMPTION APPLIES]           [STRICT S. 40A(3) APPLIES]
         • Cash Payout > ₹10,000 PERMITTED.        • Cash Payout Cap: EXACTLY ₹10,000.
         • Mandate Cultivator Identity Entry       • If Net Value > ₹10,000:
           (Kisan Credit Card / Aadhaar / Pan).       ──► BLOCK CASH CHECKOUT.
         • Print Statutory Rule 6DD Exemption         ──► Force Split Settlement:
           Declaration on Physical Slip.                  ₹10,000 Cash + Balance via
                                                          Bank Transfer (NEFT/RTGS).

```

---

## 3. Pre-Weighment Advances (*Peshgi*) & Rounding Slippage Calculus

### 3.1 The Advance (*Peshgi*) Ledger Lifecycle

In rural yards, cash is frequently disbursed before the final ticket is printed:

* **State 1 (`PESHGI_ISSUED`):** Cash disbursed to driver/farmer against a registered vehicle number (`UP-34-AT-xxxx`). Counter float decreases immediately:

$$V_{\text{counter}}(t_1) = V_{\text{counter}}(t_0) - \text{Advance}_{\text{paise}}$$


* **State 2 (`TICKET_FINALIZED`):** When net weight is locked, total payable is calculated. The system auto-deducts the advance:

$$\text{FinalDisbursal}_{\text{paise}} = \text{NetPayable}_{\text{paise}} - \text{Advance}_{\text{paise}} - \text{HandlingFees}_{\text{paise}}$$


* **State 3 (`RECONCILED`):** The advance voucher transitions to `SETTLED`, irrevocably linked via foreign key to the finalized `ledger_transactions.ledger_id`.

### 3.2 Denomination Starvation & Rounding Drift Calculus

Due to the absence of coins and small currency, transactions are rounded to the nearest integer rupee or ₹10 interval:

$$\text{ExactPayable}_{\text{paise}} = \left( \text{NetWeight}_{\text{kg}} \times \frac{\text{Rate}_{\text{paise}}}{100} \right) - \text{Deductions}_{\text{paise}}$$

$$\text{PhysicalDisbursement}_{\text{paise}} = \text{RoundToMultiple}(\text{ExactPayable}_{\text{paise}}, R_{\text{step}})$$

Where $R_{\text{step}} = 1000\text{ paise}$ (₹10 rounding step) or $100\text{ paise}$ (₹1 rounding step).
The per-transaction rounding drift $\delta_i$ is defined as:


$$\delta_i = \text{PhysicalDisbursement}_i - \text{ExactPayable}_i$$

**The Bounded Drift Invariant:** Over an $N$-transaction shift, cumulative rounding drift must satisfy:


$$\vert{}\sum_{i=1}^N \delta_i\vert{} \le N \times \left( \frac{R_{\text{step}}}{2} \right)$$


Any variance outside this boundary is flagged as **Unexplained Cash Shrinkage**.

---

## 4. Denomination Tracking & Shift Reconciliation

At shift change or day-end closing, the Munim executes a physical denomination count. The physical count is validated against the mathematical balance:

$$\text{TheoreticalCash}_{\text{expected}} = \text{Float}_{\text{opening}} + \sum \text{CashInflows} - \sum \text{CashDisbursements} + \sum \text{VaultTransfers}$$

$$\text{PhysicalCash}_{\text{counted}} = \sum_{d \in \{500, 200, 100, 50, 20, 10\}} (N_d \times d \times 100\text{ paise})$$

$$\Delta_{\text{reconciliation}} = \text{PhysicalCash}_{\text{counted}} - \text{TheoreticalCash}_{\text{expected}}$$

1. **Balanced State ($\vert{}\Delta_{\text{reconciliation}}\vert{} \le \text{AllowedRounding}$):** Shift closed; signed summary printed.
2. **Deficit State ($\Delta_{\text{reconciliation}} < -\text{Threshold}$):** System records an amber alert `CASH_SHORTAGE_DETECTED`. Requires supervisory PIN to write off.
3. **Surplus State ($\Delta_{\text{reconciliation}} > +\text{Threshold}$):** System flags `CASH_OVERAGE_DETECTED` (indicates unrecorded handling fee collection or farmer short-changing).



## 5. Room Database Schema Extensions

To guarantee absolute auditability under Indian tax and accounting laws, the cash engine operates on an append-only double-entry ledger:

```sql
-- Architectural Entity: Cash Drawers & Vaults
CREATE TABLE IF NOT EXISTS cash_drawers (
    drawer_id TEXT PRIMARY KEY NOT NULL,          -- 'COUNTER_FLOAT_01', 'VAULT_MAIN'
    drawer_name TEXT NOT NULL,
    max_limit_paise INTEGER NOT NULL,             -- Ceiling limit (e.g. 10000000 = Rs 1,00,000)
    current_custodian_id TEXT NOT NULL,
    is_active INTEGER NOT NULL DEFAULT 1
);

-- Architectural Entity: Append-Only Cash Journal
CREATE TABLE IF NOT EXISTS cash_journal_entries (
    journal_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    drawer_id TEXT NOT NULL REFERENCES cash_drawers(drawer_id),
    transaction_id INTEGER REFERENCES ledger_transactions(ledger_id),
    entry_type TEXT CHECK(entry_type IN (
        'OPENING_FLOAT', 
        'DISBURSEMENT_PAYOUT', 
        'PESHGI_ADVANCE', 
        'FEE_COLLECTION', 
        'VAULT_REPLENISHMENT', 
        'ROUNDING_VARIANCE', 
        'DISCREPANCY_ADJUSTMENT'
    )) NOT NULL,
    amount_paise INTEGER NOT NULL,                -- Positive for debit (inflow), negative for credit (outflow)
    running_balance_paise INTEGER NOT NULL,
    recipient_type TEXT CHECK(recipient_type IN ('CULTIVATOR_FARMER', 'TRANSPORTER', 'COMMISSION_AGENT', 'INTERNAL')),
    recipient_identifier TEXT,                   -- Aadhaar / PAN / KCC Number
    statutory_rule_6dd_flag INTEGER NOT NULL DEFAULT 0,
    created_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_cash_drawer_time 
    ON cash_journal_entries(drawer_id, created_epoch_ms);

-- Architectural Entity: End-of-Day Denomination Audit
CREATE TABLE IF NOT EXISTS cash_reconciliation_audits (
    audit_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    drawer_id TEXT NOT NULL REFERENCES cash_drawers(drawer_id),
    shift_operator_id TEXT NOT NULL,
    supervisor_id TEXT NOT NULL,
    count_500 INTEGER NOT NULL DEFAULT 0,
    count_200 INTEGER NOT NULL DEFAULT 0,
    count_100 INTEGER NOT NULL DEFAULT 0,
    count_50 INTEGER NOT NULL DEFAULT 0,
    count_20 INTEGER NOT NULL DEFAULT 0,
    count_10 INTEGER NOT NULL DEFAULT 0,
    physical_total_paise INTEGER NOT NULL,
    theoretical_total_paise INTEGER NOT NULL,
    discrepancy_paise INTEGER NOT NULL,
    audit_timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Prevent deletion or updates on cash journal (Strict Immutability)
CREATE TRIGGER IF NOT EXISTS abort_cash_journal_tamper
BEFORE UPDATE ON cash_journal_entries
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Cash journal entries are immutable financial records.');
END;

```

---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Pre-Dawn Opening Float Assay:**
* *Procedure:* Observe the morning float handover (7:00 AM) from the head Munim to the scale operator.
* *Evaluation Criteria:* Document the starting cash sum (verify if $\ge ₹10,00,000$), count of sealed currency bundles (*gaddis*), and whether serial numbers are recorded.


2. **Denomination Shortage Shadowing:**
* *Procedure:* Shadow 20 consecutive cash disbursements at the scale window.
* *Evaluation Criteria:* Record how many transactions require rounding due to lack of small currency notes. Tally cumulative rounding drift.


3. **Peshgi Unlinked Advance Interrogation:**
* *Procedure:* Check how unlinked advances are issued to incoming drivers who demand diesel money before unloading.
* *Evaluation Criteria:* Verify that advance slips require vehicle registration linking and prevent duplicate payouts.


4. **Section 40A(3) Scrap Cash Enforcement Drill:**
* *Procedure:* Initiate a test transaction for scrap metal valued at ₹35,000. Attempt to select "Full Cash Payment."
* *Evaluation Criteria:* Verify that the application halts checkout, flags Section 40A(3) violation, and enforces bank transfer split settlement.


5. **Surprise Mid-Day Cash Drawer Audit:**
* *Procedure:* Pause scale operations at 1:00 PM for 5 minutes. Count physical currency in the counter drawer and compare against `cash_journal_entries.running_balance_paise`.
* *Evaluation Criteria:* Record discrepancy $\Delta_{\text{midday}}$. Verify that variance remains within allowable rounding bounds ($< ₹500$).



---

## 7. Repo Impact Analysis

* **Updates to `MASTER_CORE_PROTOCOL.md`:**
* Establishes the **Asymmetric Outflow Invariant**: scale terminals are architecturally modeled as cash-disbursing nodes supported by vault replenishment vouchers.
* Formally mandates **Section 40A(3) Compliance Interlocks**: cash payouts $> ₹10,000$ are hard-blocked for non-agricultural commodities; agricultural payouts require Rule 6DD(e) cultivator identification.
* Codifies **Append-Only Double-Entry Cash Accounting**: mutative balance overrides are prohibited; all cash shifts must close with physical denomination counts.


* **Impact on Room Database Schemas:**
* Adds `cash_drawers`, `cash_journal_entries`, and `cash_reconciliation_audits` tables with immutability triggers.



---

## 8. Digest Card

* **Key Invariants:** Outflow-Dominated Regime ($95\%$ cash disbursements); Dual-Chest Hierarchy ($V_{\text{counter}} \le ₹1\text{L}$ vs $V_{\text{vault}} \ge ₹10\text{L}$); Rule 6DD(e) Agricultural Exemption vs S.40A(3) ₹10,000 Scrap Cap; Bounded Rounding Drift Invariant ($\Delta_{\text{round}}$); Append-Only Immutable Journal.
* **Cash Flow Dynamics:** Morning bank infusion $\to$ Vault $\to$ Shift transfers $\to$ Scale window payout $\to$ Evening denomination reconciliation.
* **Forensic Protections:** Denomination counting ($500/200/100/50/20/10$); Automatic advance (*Peshgi*) deductor; Dual-supervisor variance write-off.
* **Top 3 Gemba Hooks:**
1. Inspect morning opening float handover and bundle verification.
2. Measure per-transaction rounding drift from small-note shortages.
3. Test automatic Section 40A(3) blocking on non-exempt scrap payouts $> ₹10,000$.
