# Vernacular Trade Semiotics, Ethnomathematics & Oral Deduction Dictionary
Path: core/human_factors/VERNACULAR_SEMIOTIC_DICTIONARY.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md, Nunes & Carraher (1993)

## 1. Ethnomathematics of Agricultural & Scrap Trade

Market operators, truck drivers, and smallholder farmers in Tier-2/3 India execute mental arithmetic through oral ratio systems rather than written symbolic mathematics (Nunes, Schliemann & Carraher 1993). Imposing standard decimal notation (`0.25`, `1.5%`) on native keypads introduces cognitive friction, delays queue clearance, and triggers input errors.

The native instrument bridges this divide by incorporating **vernacular semantic tokens** into the input architecture while maintaining strict integer precision in the storage engine.

---

## 2. Vernacular Ratio & Fractional Arithmetic

Traditional north-Indian wholesale trade uses a base-4 fractional hierarchy. The UI keypad provides dedicated fractional macro keys mapping to exact basis points ($1\text{ bp} = 0.01\%$):

| Vernacular Term | Devanagari | Mathematical Value | Keypad Macro | Integer Representation (Basis Points) |
| :--- | :--- | :--- | :--- | :--- |
| **Paon** | पाव | $\frac{1}{4}$ ($0.25$) | `+¼` | $2500\text{ bp}$ ($25.00\%$) or $250\text{ g}$ per kg |
| **Adha** | आधा | $\frac{1}{2}$ ($0.50$) | `+½` | $5000\text{ bp}$ ($50.00\%$) or $500\text{ g}$ per kg |
| **Paune** | पौने | $X - \frac{1}{4}$ ($0.75$) | `¾` | $7500\text{ bp}$ ($75.00\%$) |
| **Sawa** | सवा | $X + \frac{1}{4}$ ($1.25$) | `1¼` | $12500\text{ bp}$ ($125.00\%$) |
| **Dedh** | डेढ़ | $1\frac{1}{2}$ ($1.50$) | `1½` | $15000\text{ bp}$ ($150.00\%$) |
| **Dhai / Arhai** | ढाई / अढ़ाई | $2\frac{1}{2}$ ($2.50$) | `2½` | $25000\text{ bp}$ ($250.00\%$) |
| **Sadhe** | साढ़े | $X + \frac{1}{2}$ for $X \ge 3$| `+½` | $(X \times 10000) + 5000\text{ bp}$ |

---

## 3. The Mandi Deduction & Operational Argot Dictionary

Informal deductions applied at the scale interface are designated by precise trade terms. Every term represents an operational or financial vector:

| Trade Term | Devanagari | Physical Meaning | Operational Reality | Legal Status (APMC Act) | System Handling |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Bardana** | बारदाना | Container Packaging Tare | Jute or HDPE sack weight. | **Statutory Requirement** | Explicit bag-counter UI; auto-deducts verified bag tare. |
| **Katauti** | कटौती | General Discount / Dockage | Broad deduction for quality variance. | **Statutory Violation** if arbitrary | Segregated to "Negotiated Commercial Rebate" ledger. |
| **Dhalta** | ढलता | Drying / Spoilage Allowance | Flat allowance (e.g., 1–2 kg/quintal) for transit drying. | **ILLEGAL** under Mandi Bye-laws | Disallowed as physical mass cut; mapped to price rebate. |
| **Karda / Garda**| करदा / गरदा| Foreign Matter / Dust | Chaff, mud, pebbles mixed in grain or metal scrap. | Allowable **only** via certified laboratory sieving | Requires optical sample capture if $>2\%$. |
| **Chhanni** | छन्नी | Sieving Loss | Small broken grains falling through mechanical sieves. | Regulated Mandi allowance | Mapped to itemized physical waste stream. |
| **Tulai** | तुलाई | Weighing Service Fee | Remuneration for scale helpers (*Tulaiya*). | **Statutory Rate Cap** (Fixed Paise/Quintal) | Auto-calculated per Mandi fee schedule; non-editable. |
| **Hamali / Palla**| हमाली / पल्ला| Unloading / Lifting Labor | Manual labor fee for moving bags from vehicle to stack. | **Statutory Rate Cap** | Auto-calculated per bag count; paid to labor pool account. |
| **Kachhi Parchi**| कच्ची पर्ची | Informal Slip | Handwritten receipt issued before formal mandi billing. | **Statutory Violation** | Replaced by instant Bluetooth thermal printout. |
| **Pucca Bill** | पक्का बिल | Formal Statutory Invoice | Official Mandi Form 6/9 / Tax Invoice. | **Mandatory Legal Record** | Generated locally on demand with Section 63 BSA certificate. |

---

## 4. UI Ergonomics: The Vernacular Keypad Layout

To prevent cognitive translation lag, the native Compose UI keypad incorporates these operational operators directly into the input surface:

```text
┌─────────────────────────────────────────────────────────────┐
│ ACTIVE FIELD: [ BARDANA DEDUCTION                         ] │
├─────────────────────────────────────────────────────────────┤
│  [ 50 Katta Jute (51.0 kg) ]   [ 50 Katta HDPE (6.5 kg) ]   │  <- Quick Container Macros
│  [ Dedh Kilo (1.5 kg/q)    ]   [ Dhai Kilo (2.5 kg/q)   ]   <- Quick Quality Cuts
├───────────────┬───────────────┬───────────────┬─────────────┤
│       7       │       8       │       9       │   PAO (¼)   │
├───────────────┼──────────────┼───────────────┼─────────────┤
│       4       │       5       │       6       │   ADHA (½)  │
├───────────────┼───────────────┼───────────────┼─────────────┤
│       1       │       2       │       3       │   PAUNE (¾) │
├───────────────┼───────────────┼───────────────┼─────────────┤
│       0       │      00       │     CLEAR     │    ENTER    │
└───────────────┴───────────────┴───────────────┴─────────────┘
```


---

## 5. Field Verification Hooks (Gemba Protocol)

1. **Oral Mental Math Stopwatch:**
   - Time an experienced munim calculating a 1.5% deduction on 24,850 kg mentally versus using an Android calculator keypad.
   - Record calculation latency (target: oral < 3s vs calculator > 12s) and tally arithmetic rounding errors.
2. **Local Argot Classification Audit:**
   - Record the specific local terms used for container packaging deductions (*Bardana* vs *Katta* vs *Khol*) across 15 real transactions.
   - Verify whether local custom deducts gross bag mass or net tare.
3. **Katauti Origin Interrogation:**
   - Interview 3 arhatiyas and commission agents: *"Yeh jo do kilo ki katauti kat rahe ho, yeh kanoon mein hai ya aapas ki samajh hai?"*
   - Document whether the cut is classified as physical trash deduction or an informal commercial discount.
4. **Vernacular Macro Keypad Usability Trial:**
   - Provide an operator with a standard decimal numeric keyboard versus the base-4 macro keypad (`PAO`, `ADHA`, `PAUNE`, `1½`, `2½`).
   - Measure time-to-entry and error rate across 20 consecutive test entries under active line pressure.
5. **Driver Comprehension Verification:**
   - Present printed receipts showing pure metric kg versus receipts displaying both metric kg and vernacular bag counts to 5 drivers.
   - Assess immediate comprehension speed and trust level regarding reported deductions.

---

## 6. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **Vernacular Cognitive Surface Invariant**: input controls and tactile UI widgets must match the ethnomathematical oral structures of the yard; decimal abstractions on hot paths are prohibited.
  * Formally segregates statutory deductions (authorized weighing and loading fees) from informal trade cuts (*Katauti*, *Dhalta*), preventing unauthorized margin masking.
- **Impact on Room Database Schemas:**
  * Requires storing applied vernacular unit snapshots and basis-point deduction flags inside `transaction_unit_snapshots`.
  * Integrates with `core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md` for rational coprime conversion enforcement.

---

## 7. Digest Card

- **Key Invariants:** Ethnomathematics Dominated (Oral ratio systems outperform formal decimals under stress); Base-4 Fractional Grammar (*Pao, Adha, Paune, Sawa, Dedh, Dhai*); Mandi Argot Standardization (*Bardana, Dhalta, Karda, Tulai*); Integer Storage in Basis Points.
- **Cognitive Design Law:** UI input layer speaks the vernacular language of the yard; database storage engine strictly enforces metric integers (Grams, Paise).
- **Core Tensions:** Arhatiyas using arbitrary argot deductions (*Dhalta*) to extract margin vs. APMC statutory prohibitions.
- **Top 3 Gemba Hooks:**
  1. Benchmark oral decomposition calculation speed vs. smartphone keypad entry.
  2. Map local argot terms for quality cuts (*Karda, Garda, Katauti*).
  3. Validate operator error rates using base-4 vernacular macro buttons vs standard decimal keypads.
