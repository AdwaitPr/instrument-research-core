# Metrological Fragmentation Index & Rational Unit Conversion Architecture
Path: core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, The Legal Metrology Act 2009 (Sections 11 & 12), IS 1943:2014, IS 12650:2018

## 1. Statutory Exclusivity vs. Vernacular Fragmentation

Under Sections 11 and 12 of The Legal Metrology Act, 2009, the metric system is the exclusive legal standard for commercial trade in India:
- Base Unit of Mass: **Kilogram** ($kg$) -> Stored internally as **Integer Grams** ($g$).
- Base Unit of Value: **Rupee** ($₹$) -> Stored internally as **Integer Paise** ($p$).

Any software recording transactions directly in non-standard units (e.g., Maunds, Brass, Bigha) is legally non-compliant and inadmissible under Indian commercial evidence rules. However, because physical transactions are negotiated verbally in traditional units, the native instrument must provide an **Input Translation Shell** that maps vernacular trade units into canonical metric integers via deterministic rational arithmetic.

---

## 2. Rational Integer Pair Conversion Calculus

To eliminate IEEE 754 binary floating-point drift, all unit conversions must be executed using coprime rational integer pairs:

$$\text{CanonicalValue}_{\text{integer}} = \left( \text{InputQuantity} \times \frac{\text{Numerator}}{\text{Denominator}} \right) \pm \text{Rounding}$$

Where:
- $\text{Numerator} \in \mathbb{Z}^+$
- $\text{Denominator} \in \mathbb{Z}^+$
- Division is executed via strict integer arithmetic with deterministic midpoint rounding (`RoundingMode.HALF_EVEN`).

### 2.1 Vernacular Mass Conversion Matrix

| Trade Unit | Region / Context | Canonical Unit | Rational Ratio ($\frac{\text{Num}}{\text{Den}}$) | Exact Metric Equivalent | Statutory Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Metric Quintal (q)** | Universal National Mandi | Grams ($g$) | $\frac{100,000}{1}$ | $100.000\text{ kg}$ | Standard Legal Unit |
| **Metric Tonne (t)** | Industrial Weighbridges | Grams ($g$) | $\frac{1,000,000}{1}$ | $1000.000\text{ kg}$ | Standard Legal Unit |
| **Katta (HDPE Grain)** | UP/Bihar Grain Mandi | Grams ($g$) | $\frac{50,000}{1}$ | $50.000\text{ kg}$ (Nominal) | Customary Packaging |
| **Bori (Sugar/Standard)**| National Procurement | Grams ($g$) | $\frac{50,000}{1}$ | $50.000\text{ kg}$ (Net) | Customary Packaging |
| **Potato Bori (Cold Storage)**| Agra / Aligarh Belt | Grams ($g$) | $\frac{52,000}{1}$ | $52.000\text{ kg}$ | Local Trade Custom |
| **Metric Maund (Man)**| Northern India Mandi | Grams ($g$) | $\frac{40,000}{1}$ | $40.000\text{ kg}$ | Non-Standard (Customary) |
| **Bengal / Imperial Maund**| Traditional Historical | Grams ($g$) | $\frac{37,324,200}{1000}$ | $37.3242\text{ kg}$ | ILLEGAL (Legacy) |
| **Seer (Ser)** | Historical (1/40 Maund)| Grams ($g$) | $\frac{933,105}{1000}$ | $933.105\text{ g}$ | ILLEGAL (Legacy) |
| **Tola** | Precious Metals / Scrap| Grams ($g$) | $\frac{11,664}{1000}$ | $11.664\text{ g}$ | Non-Standard (Customary) |

### 2.2 Volumetric & Area Fragmentation Matrix

| Trade Unit | Region / Context | Canonical Unit | Rational Ratio ($\frac{\text{Num}}{\text{Den}}$) | Conversion Logic |
| :--- | :--- | :--- | :--- | :--- |
| **Brass (Mining)** | UP / MP Sand & Aggregate | Millilitres ($mL$) | $\frac{2,831,685,000}{1000}$ | $100\text{ cu ft} = 2.831685\text{ m}^3$ |
| **Brass $\to$ Weight (Sand)**| River Sand (Moist) | Grams ($g$) | $\frac{4,200,000}{1}$ | Density Estimate: $\approx 4.2\text{ t/Brass}$ |
| **Brass $\to$ Weight (Grit)**| Crushed Stone Aggregate | Grams ($g$) | $\frac{3,800,000}{1}$ | Density Estimate: $\approx 3.8\text{ t/Brass}$ |
| **Pucca Bigha (West UP)**| Meerut / Saharanpur | Sq Millimetres ($mm^2$)| $\frac{2,529,285,264}{1000}$ | $1\text{ Bigha} = 20\text{ Biswa} \approx 2,529.3\text{ m}^2$ |
| **Kaccha Bigha (Central UP)**| Sitapur / Hardoi | Sq Millimetres ($mm^2$)| $\frac{843,000,000}{1000}$ | $1\text{ Bigha} \approx 843\text{ m}^2$ (1/3 Pucca) |

---

## 3. The Container Tare Stack-Up Invariant (Bardana Calculus)

In grain trade, gross weighments include packaging containers (*Bardana*). The software enforces deterministic tare deduction based on container type:

Gross Mass (Load Cell)
├── Bulk Cargo (Direct Tractor-Trolley Loose Grain) ──► Tare = Empty Trolley Mass
└── Bagged Cargo (Stacked Sacks)
├── Standard Jute (IS 1943) ────────► Deduct 1,020 g per sack
├── Lightweight Jute ───────────────► Deduct   850 g per sack
└── Woven HDPE / PP (IS 12650) ─────► Deduct   130 g per sack

### 3.1 Anti-Skimming Tare Formula
$$\text{ChargeableNetWeight}_{\text{grams}} = \text{GrossMass} - \text{VehicleTare} - \sum_{i} (N_i \times \text{ContainerTare}_i)$$

Where:
- $N_i$: Integer count of bags of type $i$.
- $\text{ContainerTare}_{\text{JUTE}} = 1000\text{ g}$ (Codified from IS 1943).
- $\text{ContainerTare}_{\text{HDPE}} = 130\text{ g}$ (Codified from IS 12650).

**System Rule:** The UI must never permit a flat $1\text{ kg}$ deduction if bag type is selected as `HDPE_PP`.

---

## 4. Room Database Schema: Unit Conversion & Context Registry

```sql
CREATE TABLE IF NOT EXISTS metrological_unit_registry (
    unit_code TEXT PRIMARY KEY NOT NULL,
    vernacular_name TEXT NOT NULL,
    target_dimension TEXT CHECK(target_dimension IN ('MASS', 'VOLUME', 'AREA')) NOT NULL,
    canonical_base_unit TEXT NOT NULL,
    conversion_numerator INTEGER NOT NULL,
    conversion_denominator INTEGER NOT NULL,
    is_statutory_metric INTEGER NOT NULL DEFAULT 0,
    state_code TEXT NOT NULL DEFAULT 'IN-UP',
    district_filter TEXT,
    active_from_epoch_ms INTEGER NOT NULL
);

INSERT OR IGNORE INTO metrological_unit_registry 
(unit_code, vernacular_name, target_dimension, canonical_base_unit, conversion_numerator, conversion_denominator, is_statutory_metric) 
VALUES 
('MASS_KG', 'Kilogram', 'MASS', 'GRAM', 1000, 1, 1),
('MASS_QUINTAL', 'Quintal', 'MASS', 'GRAM', 100000, 1, 1),
('MASS_TONNE', 'Metric Tonne', 'MASS', 'GRAM', 1000000, 1, 1),
('MASS_MAUND_METRIC', 'Maund (Pukka)', 'MASS', 'GRAM', 40000, 1, 0),
('MASS_KATTA_WHEAT', 'Katta (Wheat Standard)', 'MASS', 'GRAM', 50000, 1, 0),
('MASS_BAG_JUTE_TARE', 'Jute Gunny Bag Tare', 'MASS', 'GRAM', 1020, 1, 1),
('MASS_BAG_HDPE_TARE', 'HDPE Woven Sack Tare', 'MASS', 'GRAM', 130, 1, 1),
('VOL_BRASS', 'Brass (100 cu ft)', 'VOLUME', 'MILLILITRE', 2831685000, 1000, 0);

CREATE TABLE IF NOT EXISTS transaction_unit_snapshots (
    snapshot_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    transaction_id INTEGER NOT NULL REFERENCES ledger_transactions(ledger_id),
    entered_quantity INTEGER NOT NULL,
    entered_unit_code TEXT NOT NULL REFERENCES metrological_unit_registry(unit_code),
    resolved_canonical_grams INTEGER NOT NULL,
    created_epoch_ms INTEGER NOT NULL
);
```

---

## 5. Field Verification Hooks (Gemba Protocol)

1. **Physical Bag Mass Assay:** Weigh 20 empty sacks collected at the scale ramp using a certified Class II scale. Verify distribution of Jute ($950\text{--}1050\text{ g}$) vs HDPE ($120\text{--}140\text{ g}$).
2. **Potato/Specialty Packing Audit:** In cold-storage belts, verify whether local trade packs $50\text{ kg}$, $52\text{ kg}$, or $55\text{ kg}$ per bori.
3. **Maund Ratio Interrogation:** Ask three independent traders: *"Yahan ek man mein kitne kilo mante ho?"* (Verify if metric $40\text{ kg}$ or historical $37.3\text{ kg}$ is assumed).
4. **Brass Aggregate Tare Check:** Shadow a dumper delivering aggregate; cross-reference the volume quoted in "Brass" on the challan against the net weighbridge mass in tonnes. Compute effective bulk density.
5. **District Boundary Drift:** Record bag and land area terminology variations across adjacent district borders (e.g., Lucknow vs Sitapur vs Lakhimpur).

---

## 6. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Codifies the **Input Translation Shell vs Storage Engine Invariant**: UI accepts vernacular units via rational coprime integer pairs; database stores strictly SI metric integers (grams, millilitres, paise).
  * Incorporates **Container Tare Deduction Invariants** (IS 1943 Jute @ 1020g vs IS 12650 HDPE @ 130g) to eliminate flat-rate container skimming.
- **Impact on Room Database Schema:**
  * Adds `metrological_unit_registry` and `transaction_unit_snapshots` to preserve deterministic historical auditability of all applied unit conversions.

---

## 7. Digest Card

- **Key Invariants:** Statutory Exclusivity (Legal Metrology Act 2009 S.11 mandates metric output); Rational Conversion Calculus ($\text{Value} \times \frac{\text{Num}}{\text{Den}}$ in integer arithmetic); Bardana Skim Elimination (Jute @ 1020g vs HDPE @ 130g).
- **Core Conversions:** 1 Quintal = $100,000\text{ g}$; 1 Tonne = $1,000,000\text{ g}$; 1 Metric Maund = $40,000\text{ g}$; 1 Brass = $2,831,685,000\text{ mL}$.
- **Core Tensions:** Legal mandate for metric printouts vs. farmers negotiating in Maunds/Bags; flat 1kg tare skim tradition vs. statutory tare accuracy.
- **Top 3 Gemba Hooks:**
  1. Assay empty HDPE bag tare ($130\text{ g}$) vs Jute ($1020\text{ g}$).
  2. Verify local metric conversion of "Man" ($40\text{ kg}$ vs $37.32\text{ kg}$).
  3. Validate brass volume to metric tonne density ratios at quarry gates.
