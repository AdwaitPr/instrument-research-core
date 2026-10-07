# QUALITY ATTRIBUTE SCENARIOS TEMPLATE
Path: engine/templates/QUALITY_ATTRIBUTE_SCENARIOS.md
Status: ACTIVE / CORE SPECIFICATION

## Purpose
Quality Attribute Scenarios translate qualitative operational stressors and latency budgets into precise, testable non-functional requirements (NFRs).

## Template

| QAS-ID | Source Lens | Stimulus | Environment | Artifact | Response | Response Measure | Evidence Tag | Fitness Check |
|---|---|---|---|---|---|---|---|---|
| QAS-001 | Lens 02 | Hardware power loss during receipt issuance | High-queue peak shift | Transaction Ledger | System restores draft state on restart | `<threshold: derive from field data>` recovery time via local WAL replay | `[SOURCED-EMPIRICAL]` | Automated crash-recovery integration test |
| QAS-002 | Lens 03 | High tactile input rate | Direct outdoor glare | Transaction Form UI | Input acknowledgment | `<threshold: derive from field data>` latency ceiling | `[SOURCED-EMPIRICAL]` | Automated UI performance benchmark |

*EXAMPLE – DELETE:*
| QAS-ID | Source Lens | Stimulus | Environment | Artifact | Response | Response Measure | Evidence Tag | Fitness Check |
|---|---|---|---|---|---|---|---|---|
| QAS-001 | Lens 02 | Network disconnection during kiosk checkout | Peak library morning hours | Lending Sync Service | Queue transaction locally | `<threshold: derive from field data>` 0ms blocking wait | `[SOURCED-EMPIRICAL]` | Automated offline integration test |
| QAS-002 | Lens 03 | Rapid touchscreen tap input | Low-light ambient room | Patron Keypad UI | Visual tap acknowledgment | `<threshold: derive from field data>` < 50ms visual response | `[INFERRED-MECHANICAL]` | Frame-time UI test |

## Usage Rules
- Each Quality Attribute Scenario must be referenced by $\ge 1$ REQ-ID of `Type = Quality`.
