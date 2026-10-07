# HEAD 6: ARCHITECTURE DECISION RULES & PLATFORM SELECTION
Path: engine/methodologies/06_ARCHITECTURE_DECISION_RULES.md
Status: ACTIVE / CORE SPECIFICATION

# LOCATION: engine/methodologies/06_ARCHITECTURE_DECISION_RULES.md

## 1. THEORETICAL FOUNDATION
- Architecture Tradeoff Analysis Method (ATAM): Evaluates architecture decisions against quality attributes.
- Architecture Decision Records (ADR): Captures context, decision, and consequences (Nygard).
- Evolutionary Architecture: Uses automated fitness functions to verify constraints over time.
- Reversibility & Two-Way Doors: Separates irreversible decisions from easily reversed experiments.
- CAP & PACELC Theorems: Frames state consistency versus latency trade-offs during network partitioning.

## 2. RULE TABLE

| Rule | Field-Finding Trigger | Evidence Needed | Architectural Constraint Produced | Decision Left Open | Source Lens |
|---|---|---|---|---|---|
| **R1 Connectivity** | Offline/degraded operation exceeds `<threshold: derive from field data>` share of working time | `[SOURCED-EMPIRICAL]` network availability logs | Critical path must run without network; local persistence is mandatory | Sync model (server-authoritative queue / operation log / merge-based replication) | Lens 01 / Lens 02 |
| **R2 Custody & collusion** | Two or more roles with conflicting incentives touch the same record | `[SOURCED-EMPIRICAL]` role mapping or `[INFERRED-MECHANICAL]` risk analysis | Tamper-evident history and role-separated access | Mechanism (append-only store / hash-chained soft-delete / signed events) | Lens 04 |
| **R3 Numeric exactness** | Legal rounding, tolerance or currency rules govern a quantity | `[SOURCED-REGULATORY]` statute or statutory metrology rule | Exact numeric representation; binary floating point barred for that quantity only | Representation (scaled integer / decimal type) and rounding policy | Lens 01 / Lens 06 |
| **R4 Peripheral dependency** | Required hardware interface (serial, Bluetooth, NFC, USB, camera, etc.) | `[SOURCED-EMPIRICAL]` physical tool inventory | Hard gate: platforms lacking reliable access to that interface are eliminated | Which surviving platform; adapter/abstraction layer design | Lens 01 |
| **R5 Latency ceiling** | Lens 03 produces a $\tau_{\text{budget}}$ beyond which operators bypass the system | `[SOURCED-EMPIRICAL]` time motion audit | No network round-trip on the critical path; budget allocated across components | Local compute vs. edge vs. cloud for non-critical work | Lens 03 |
| **R6 Retention vs. erasure** | Statutory retention horizon and/or erasure/privacy obligation applies | `[SOURCED-REGULATORY]` legal mandate citation | Explicit data lifecycle policy per record class | Resolution when retention and erasure conflict (e.g., redaction, escrow, archival tier) | Lens 06 |
| **R7 Interruption & process loss** | Frequent interruption, low-memory devices, or forced app/tab termination | `[SOURCED-EMPIRICAL]` interruption tally or hardware profile | Draft state survives termination; define persistence granularity | Persistence frequency (per input / per step / per transaction) and recovery UX | Lens 03 |
| **R8 Distribution channel** | Lens 08 shows no standard app store / locked-down or air-gapped installs | `[SOURCED-EMPIRICAL]` IT deployment audit | Distribution/update mechanism is a platform gate | Sideload / web delivery / managed device / offline media | Lens 08 |
| **R9 Buyer ≠ operator** | Lens 07 shows economic buyer differs from daily user | `[SOURCED-EMPIRICAL]` stakeholder mapping | Tenancy and role model must separate oversight view from operator view | Single-tenant vs. multi-tenant; RBAC granularity | Lens 07 |

## 3. PROCEDURE
1. List active triggers (a rule fires only if its evidence is `[SOURCED-REGULATORY]`, `[SOURCED-EMPIRICAL]`, or `[INFERRED-MECHANICAL]`; if only `[HYPOTHETICAL-UNAUDITED]`, log as `ASSUME` requirement and schedule a spike instead of constraining the design).
2. Emit one REQ per fired rule into the Requirements Register.
3. Run hard gates (§4.1).
4. Score survivors (§4.2).
5. Run sensitivity check (§4.3).
6. Write ADRs; write QAS for every Quality requirement.

## 4. PLATFORM DECISION MATRIX

### 4.1 Hard Gates
Options evaluated: `Native mobile`, `Cross-platform mobile`, `Offline-capable web (PWA)`, `Online-first SaaS web`, `Local-first client + server (hybrid)`.

For each option, mark PASS/FAIL against each fired R4/R5/R8/R1 constraint. Any FAIL eliminates the option; the specific reason must be recorded.

### 4.2 Weighted Scoring
Scoring applies to options surviving §4.1 hard gates.

| Driver / Attribute | Weight (1–5) | Linked REQ-ID | Native mobile | Cross-platform mobile | Offline-capable web (PWA) | Online-first SaaS web | Local-first client + server | Evidence Tag & Justification |
|---|---|---|---|---|---|---|---|---|
| Team Skills | `<weight>` | `REQ-###` | 0–3 | 0–3 | 0–3 | 0–3 | 0–3 | `[INFERRED-MECHANICAL]` ... |
| Budget & Runway | `<weight>` | `REQ-###` | 0–3 | 0–3 | 0–3 | 0–3 | 0–3 | `[INFERRED-MECHANICAL]` ... |
| Time-to-Ship | `<weight>` | `REQ-###` | 0–3 | 0–3 | 0–3 | 0–3 | 0–3 | `[INFERRED-MECHANICAL]` ... |
| Target Scale | `<weight>` | `REQ-###` | 0–3 | 0–3 | 0–3 | 0–3 | 0–3 | `[INFERRED-MECHANICAL]` ... |

*EXAMPLE – DELETE:*
| Driver / Attribute | Weight (1–5) | Linked REQ-ID | Native mobile | Cross-platform mobile | Offline-capable web (PWA) | Online-first SaaS web | Local-first client + server | Evidence Tag & Justification |
|---|---|---|---|---|---|---|---|---|
| Offline Reliability | 5 | REQ-001 | 3 | 3 | 2 | 0 | 3 | `[SOURCED-EMPIRICAL]` 80% offline operation |

### 4.3 Sensitivity Check
Shift the highest-weighted driver score/weight by $\pm 1$ and re-rank survivors. If the winning option changes, the decision is **fragile**: it MUST be recorded as fragile in the ADR and a Verification Spike is mandatory.

## 5. ANTI-PATTERNS
- Choosing the technology stack before triggers and invariants are evaluated.
- Promoting a `HYPOTHETICAL-UNAUDITED` finding to a hard constraint without completing a field-check or spike.
- Treating append-only or offline-first as defaults rather than outputs derived from R1/R2 rules.
- Scoring options without assigning evidence tags and justifications to every score cell.
- Leaving "Flip conditions" or "Trade-offs accepted" empty or generic in ADRs.
- Omitting builder constraints (team skills, budget, time-to-first-release) from the weighted scoring matrix.
