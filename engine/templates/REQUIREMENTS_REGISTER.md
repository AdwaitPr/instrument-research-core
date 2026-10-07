# REQUIREMENTS REGISTER TEMPLATE
Path: engine/templates/REQUIREMENTS_REGISTER.md
Status: ACTIVE / CORE SPECIFICATION

## Purpose
The Requirements Register converts operational findings and analytical lenses into unambiguous, verifiable system requirements. It establishes a traceable chain from field observations to technical specification and architectural decisions.

## Template

| REQ-ID | Statement | Type | Source (Invariant/Lens ref) | Evidence Tag | Strength | Verification | Linked ADR | Status |
|---|---|---|---|---|---|---|---|---|
| REQ-001 | The system shall persist transaction records locally without network access. | Functional | Invariant Matrix: Process Nodes | `[SOURCED-EMPIRICAL]` | MUST | Test | ADR-001 | Open |
| REQ-002 | The system shall provide single-hand touch targets exceeding 64dp. | Quality | Lens 03 | `[INFERRED-MECHANICAL]` | SHOULD | Metric | ADR-002 | Open |

*EXAMPLE – DELETE:*
| REQ-ID | Statement | Type | Source (Invariant/Lens ref) | Evidence Tag | Strength | Verification | Linked ADR | Status |
|---|---|---|---|---|---|---|---|---|
| REQ-001 | The kiosk shall cache book availability data locally for up to 24 hours. | Functional | Invariant Matrix: Physical Laws | `[SOURCED-EMPIRICAL]` | MUST | Test | ADR-001 | Open |
| REQ-002 | The kiosk shall assume patron RFID scanning speed is below 2 seconds per item. | Assumption | Lens 03 | `[HYPOTHETICAL-UNAUDITED]` | ASSUME | Field-check | ADR-002 | Open |

## Evidence-Inheritance Rules
- `[SOURCED-REGULATORY]` or `[SOURCED-EMPIRICAL]` → may be `MUST`.
- `[INFERRED-MECHANICAL]` → may be `MUST` only if the derivation formula is written in the Statement or Verification cell; otherwise `SHOULD`.
- `[HYPOTHETICAL-UNAUDITED]` → MUST be `Strength = ASSUME`, `Type = Assumption`, and `Verification = Spike` or `Field-check`. It may never be `MUST`.
- A requirement with no `Source` is invalid and must be deleted.
