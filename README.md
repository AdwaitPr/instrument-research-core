# Commercial Weighbridge & Mandi Metrology Instrument Core

A field-hardened, legally compliant, zero-cloud software specification and architecture for commercial vehicle weighbridges operating in hostile agricultural procurement and rural freight corridors in India.

## 1. Architectural Wave Structure (19 Formal Aspects)

The specification spans 4 foundational waves comprising 19 formal metrological aspects (4,281 lines of verified specifications):

| Wave | Domain | Aspect & Specification File | Lines | Status |
| :--- | :--- | :--- | :---: | :---: |
| **Wave B** | Formal Systems | [`core/formal/PETRI_STATE_MATRIX.md`](core/formal/PETRI_STATE_MATRIX.md) (Aspect 01) | 217 | COMPLETE |
| | Formal Systems | [`core/formal/DEGRADED_STATE_LADDER.md`](core/formal/DEGRADED_STATE_LADDER.md) (Aspect 02) | 124 | COMPLETE |
| | Formal Systems | [`core/formal/QUEUE_COLLAPSE_MODEL.md`](core/formal/QUEUE_COLLAPSE_MODEL.md) (Aspect 03) | 307 | COMPLETE |
| | Formal Systems | [`core/formal/AIRGAP_EXCHANGE_SPEC.md`](core/formal/AIRGAP_EXCHANGE_SPEC.md) (Aspect 05) | 259 | COMPLETE |
| **Wave A** | Physical Substrate | [`core/substrate/REGULATORY_CONFIG_BOUNDARY.md`](core/substrate/REGULATORY_CONFIG_BOUNDARY.md) (Aspect 10) | 182 | COMPLETE |
| | Physical Substrate | [`core/human_factors/VERNACULAR_SEMIOTIC_DICTIONARY.md`](core/human_factors/VERNACULAR_SEMIOTIC_DICTIONARY.md) (Aspect 14) | 111 | COMPLETE |
| | Physical Substrate | [`core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md`](core/substrate/METROLOGICAL_FRAGMENTATION_INDEX.md) (Aspect 16) | 144 | COMPLETE |
| | Physical Substrate | [`core/substrate/HARDWARE_ATTACK_SURFACE.md`](core/substrate/HARDWARE_ATTACK_SURFACE.md) (Aspect 17) | 235 | COMPLETE |
| | Physical Substrate | [`core/substrate/MEDIA_PERSISTENCE_SPEC.md`](core/substrate/MEDIA_PERSISTENCE_SPEC.md) (Aspect 18) | 242 | COMPLETE |
| | Physical Substrate | [`core/substrate/ELECTRICAL_HOSTILITY_SPECS.md`](core/substrate/ELECTRICAL_HOSTILITY_SPECS.md) (Aspect 19) | 238 | COMPLETE |
| **Wave C** | Institutions & Economics | [`core/substrate/COLLUSION_GAME_THEORY.md`](core/substrate/COLLUSION_GAME_THEORY.md) (Aspect 06) | 230 | COMPLETE |
| | Institutions & Economics | [`core/economics/CASH_DRAWER_SETTLEMENT.md`](core/economics/CASH_DRAWER_SETTLEMENT.md) (Aspect 08) | 249 | COMPLETE |
| | Institutions & Economics | [`core/human_factors/AUDIO_HAPTIC_GLANCING_UX.md`](core/human_factors/AUDIO_HAPTIC_GLANCING_UX.md) (Aspect 07) | 183 | COMPLETE |
| | Institutions & Economics | [`core/institutions/VIGILANCE_AUDIT_PROTOCOL.md`](core/institutions/VIGILANCE_AUDIT_PROTOCOL.md) (Aspect 09) | 240 | COMPLETE |
| **Wave D** | Field Ops & Forensics | [`core/substrate/DYNAMIC_IMPACT_TELEMETRY.md`](core/substrate/DYNAMIC_IMPACT_TELEMETRY.md) (Aspect 04) | 253 | COMPLETE |
| | Field Ops & Forensics | [`core/logistics/FLEET_IDENTITY_FASTPATH.md`](core/logistics/FLEET_IDENTITY_FASTPATH.md) (Aspect 11) | 275 | COMPLETE |
| | Field Ops & Forensics | [`core/arbitration/DISPUTE_ARBITRATION_LIFECYCLE.md`](core/arbitration/DISPUTE_ARBITRATION_LIFECYCLE.md) (Aspect 12) | 266 | COMPLETE |
| | Field Ops & Forensics | [`core/vision/OFFLINE_ANPR_GATE_INTERLOCKS.md`](core/vision/OFFLINE_ANPR_GATE_INTERLOCKS.md) (Aspect 13) | 274 | COMPLETE |
| | Field Ops & Forensics | [`core/forensics/LONG_TERM_PROOF_PERSISTENCE.md`](core/forensics/LONG_TERM_PROOF_PERSISTENCE.md) (Aspect 15) | 252 | COMPLETE |

## 2. Statutory Legal & Regulatory Grounding

All system modules comply with statutory Indian metrological and commercial laws:
- **Legal Metrology Act, 2009 & General Rules 2011 (Schedule VII):** Class III Non-Automatic Weighing Instruments ($e=10\text{ kg}$, $\text{Max}=50\text{--}60\text{ t}$).
- **Bharatiya Sakshya Adhiniyam, 2023 (Section 63):** Automated on-device electronic evidence certification.
- **Carriage by Road Act, 2007 (Sections 10 & 11):** Common carrier liability and thermodynamic moisture loss (*Sookhat*) defenses.
- **Income Tax Act, 1961 (Section 40A(3) vs Rule 6DD(e)):** Agricultural payout cash threshold exceptions.
- **Central Goods and Services Tax Act, 2017 (Section 36):** 72-month immutable record retention.

## 3. Technology Stack & Deployment Constraints

- **Platform:** Native Android (Kotlin, Jetpack Compose, Room SQLite) & Embedded Linux.
- **Connectivity:** 100% Zero-Cloud Autonomous (Sneakernet P2P Sync, Air-Gapped Operation).
- **Security:** Hardware Keystore ECDSA P-256 signatures, Merkle epoch batch trees, WORM storage.
