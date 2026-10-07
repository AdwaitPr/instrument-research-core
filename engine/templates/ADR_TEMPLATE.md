# ARCHITECTURE DECISION RECORD TEMPLATE
Path: engine/templates/ADR_TEMPLATE.md
Status: ACTIVE / CORE SPECIFICATION

## Purpose
An Architecture Decision Record (ADR) documents a significant architectural choice, its context, driver requirements, options considered, trade-offs accepted, and conditions that would invalidate the decision.

## Copy-Paste Block

```
ADR-###: <Title>
Status: Proposed | Accepted | Superseded by ADR-###
Drivers: <list of REQ-IDs, at least one>
Options (minimum 2, maximum 4):
  A. <name> – <1-line description>
  B. <name> – <1-line description>
Decision: <chosen option>
Trade-offs accepted: <what is given up>
Door type: One-way | Two-way   (One-way = costly/impossible to reverse after data or users exist)
Flip conditions: <specific field evidence that would reverse this decision>
Verification spike: <cheapest experiment that tests the riskiest assumption, with pass/fail criterion>
Consequences: <new constraints this creates>
```

## Rules
- (a) Every ADR must cite $\ge 1$ REQ-ID in Drivers.
- (b) An ADR whose Drivers are all `ASSUME` must be `Status: Proposed` and Door type must be Two-way, or a Verification spike must be completed first.
- (c) "Flip conditions" may not be empty or generic.
