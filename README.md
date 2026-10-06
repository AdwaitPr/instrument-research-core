# Hostile Edge & Rural Android Product Research Strategy Framework
### Anchor Implementation: Commercial Weighbridge & Mandi Metrology Core (Case Study 01)

A field-grounded, zero-cloud product strategy, market research framework, and formal software specification for mission-critical Android applications operating in hostile rural, industrial, and emerging-market edge environments.

---

## 1. Architectural Structure

The repository is organized into three complementary layers:

```text
├── 1. RESEARCH METHODOLOGIES       -> How to conduct fieldwork & extract invariants
├── 2. STRATEGIC PRODUCT FRAMEWORK  -> Market consumer matrix, product line, GTM & cross-domain porting
└── 3. ANCHOR CASE STUDY 01         -> Full production specification for an industrial instrument (19 Aspects)
```

---

## 2. Layer 1: Research Methodologies (`core/methodologies/`)

Core investigative tools for gathering empirical truth in adversarial field environments:
- [`01_INVARIANT_DECONSTRUCTION.md`](core/methodologies/01_INVARIANT_DECONSTRUCTION.md): Isolating unalterable physical and statutory constraints.
- [`02_ADVERSARIAL_ETHNOGRAPHY.md`](core/methodologies/02_ADVERSARIAL_ETHNOGRAPHY.md): Uncovering collusion, side-channel bribery, and informal workarounds.
- [`03_COGNITIVE_TASK_AUDIT.md`](core/methodologies/03_COGNITIVE_TASK_AUDIT.md): Measuring sensory hostility, glance latencies, and panic reaction states.
- [`04_LEAKAGE_RECONCILIATION.md`](core/methodologies/04_LEAKAGE_RECONCILIATION.md): Auditing physical mass, digital record, and cash discrepancy points.
- [`05_SATURATION_STOP_GATES.md`](core/methodologies/05_SATURATION_STOP_GATES.md): Formal exit criteria to prevent infinite research drift.

---

## 3. Layer 2: Strategic Market & Product Framework (`core/framework/`)

The reusable business, consumer, and product strategy playbook for rural Android engineering:
- [`CONSUMER_DECISION_MATRIX.md`](core/framework/CONSUMER_DECISION_MATRIX.md): Multi-stakeholder B2B/B2G mesh (Arhatiya buyer vs clerk user vs farmer participant vs vigilance auditor), Willingness-To-Pay (WTP), and the Acceptable Transparency Paradox.
- [`PRODUCT_TIERING_STRATEGY.md`](core/framework/PRODUCT_TIERING_STRATEGY.md): Product line packaging across Tier 1 (Farmgate Lite), Tier 2 (Mandi Pro), and Tier 3 (Enterprise Siding); runtime feature gating and offline Ed25519 licensing.
- [`GTM_RURAL_DISTRIBUTION.md`](core/framework/GTM_RURAL_DISTRIBUTION.md): Breaking the Google Play failure mode; leveraging local technicians (Kanta Mistri) as channel partners; USB-OTG auto-flashing; harvest-driven purchasing windows.
- [`CROSS_DOMAIN_ADAPTABILITY.md`](core/framework/CROSS_DOMAIN_ADAPTABILITY.md): Abstraction of the 4-Pillar Engine and concrete architectural mapping to Village Dairy Collection (AMCU), PDS Ration Shops, and Microfinance Loan Collection.

---

## 4. Layer 3: Anchor Case Study 01 — Commercial Weighbridge Core (19 Aspects)

The production-grade specification covering 4 foundational waves (4,281 lines of verified engineering):

| Wave | Domain Focus | Formal Specification Files | Lines |
| :--- | :--- | :--- | :---: |
| **Wave B** | Formal Concurrency & Systems | [`PETRI_STATE_MATRIX.md`](core/formal/PETRI_STATE_MATRIX.md), [`DEGRADED_STATE_LADDER.md`](core/formal/DEGRADED_STATE_LADDER.md), [`QUEUE_COLLAPSE_MODEL.md`](core/formal/QUEUE_COLLAPSE_MODEL.md), [`AIRGAP_EXCHANGE_SPEC.md`](core/formal/AIRGAP_EXCHANGE_SPEC.md) | 907 |
| **Wave A** | Physical Substrate & Hardware | [`DYNAMIC_IMPACT_TELEMETRY.md`](core/substrate/DYNAMIC_IMPACT_TELEMETRY.md), [`HARDWARE_ATTACK_SURFACE.md`](core/substrate/HARDWARE_ATTACK_SURFACE.md), [`MEDIA_PERSISTENCE_SPEC.md`](core/substrate/MEDIA_PERSISTENCE_SPEC.md), [`ELECTRICAL_HOSTILITY_SPECS.md`](core/substrate/ELECTRICAL_HOSTILITY_SPECS.md), [`REGULATORY_CONFIG_BOUNDARY.md`](core/substrate/REGULATORY_CONFIG_BOUNDARY.md), [`METROLOGICAL_FRAGMENTATION_INDEX.md`](core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md) | 1,294 |
| **Wave C** | Institutions & Economics | [`COLLUSION_GAME_THEORY.md`](core/substrate/COLLUSION_GAME_THEORY.md), [`CASH_DRAWER_SETTLEMENT.md`](core/economics/CASH_DRAWER_SETTLEMENT.md), [`AUDIO_HAPTIC_GLANCING_UX.md`](core/human_factors/AUDIO_HAPTIC_GLANCING_UX.md), [`VIGILANCE_AUDIT_PROTOCOL.md`](core/institutions/VIGILANCE_AUDIT_PROTOCOL.md), [`VERNACULAR_SEMIOTIC_DICTIONARY.md`](core/human_factors/VERNACULAR_SEMIOTIC_DICTIONARY.md) | 1,013 |
| **Wave D** | Field Ops & Forensics | [`FLEET_IDENTITY_FASTPATH.md`](core/logistics/FLEET_IDENTITY_FASTPATH.md), [`DISPUTE_ARBITRATION_LIFECYCLE.md`](core/arbitration/DISPUTE_ARBITRATION_LIFECYCLE.md), [`OFFLINE_ANPR_GATE_INTERLOCKS.md`](core/vision/OFFLINE_ANPR_GATE_INTERLOCKS.md), [`LONG_TERM_PROOF_PERSISTENCE.md`](core/forensics/LONG_TERM_PROOF_PERSISTENCE.md) | 1,067 |

---

## 5. Technology Stack & Invariants

- **Platform Target:** Native Android (Kotlin, Jetpack Compose, Room SQLite) & Industrial Linux.
- **Zero-Cloud Autonomy:** Fully functional offline; peer-to-peer sneakernet synchronization; local cryptographic attestation.
- **Statutory Admissibility:** Built to satisfy Bharatiya Sakshya Adhiniyam 2023 (Section 63) and Legal Metrology Act 2009.
