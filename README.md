# instrument-research-core

`instrument-research-core` serves as the foundational research framework and decision engine for operational deconstruction, field diagnostic audits, and architectural system design.

---

## How to Use This Engine

1. Execute field research through the core methodology files (`engine/methodologies/01_INVARIANT_DECONSTRUCTION.md` through `06_ARCHITECTURE_DECISION_RULES.md`) and diagnostic templates (`engine/templates/FIELD_DIAGNOSTIC_PROMPTS.md`, `RESEARCHER_BIAS_SELF_AUDIT.md`, `REQUIREMENTS_REGISTER.md`, `ADR_TEMPLATE.md`, `QUALITY_ATTRIBUTE_SCENARIOS.md`).
2. Map observations using universal analytical lenses (01 to 09).
3. Translate field findings into verifiable requirements and architectural decisions.

---

## Directory Structure

```text
.
├── BRIEF_TEMPLATE.md
├── LICENSE
├── README.md
├── RUN_PROMPT.md
└── engine/
    ├── PROTOCOL.md
    ├── RESEARCH_LENSES.md
    ├── methodologies/
    │   ├── 01_INVARIANT_DECONSTRUCTION.md
    │   ├── 02_ADVERSARIAL_ETHNOGRAPHY.md
    │   ├── 03_COGNITIVE_TASK_AUDIT.md
    │   ├── 04_LEAKAGE_RECONCILIATION.md
    │   ├── 05_SATURATION_STOP_GATES.md
    │   └── 06_ARCHITECTURE_DECISION_RULES.md
    └── templates/
        ├── ADR_TEMPLATE.md
        ├── FIELD_DIAGNOSTIC_PROMPTS.md
        ├── QUALITY_ATTRIBUTE_SCENARIOS.md
        ├── REQUIREMENTS_REGISTER.md
        └── RESEARCHER_BIAS_SELF_AUDIT.md
```

---

## From Findings to Architecture

1. **Findings & Evidence Tagging:** Ground operational claims using mandatory evidence tags (`[SOURCED-REGULATORY]`, `[SOURCED-EMPIRICAL]`, `[INFERRED-MECHANICAL]`, `[HYPOTHETICAL-UNAUDITED]`).
2. **Requirements Register:** Convert field findings into testable requirements using `engine/templates/REQUIREMENTS_REGISTER.md` with explicit evidence inheritance.
3. **Lens 09 Rules & Hard Gates:** Evaluate rules R1–R9 (`engine/methodologies/06_ARCHITECTURE_DECISION_RULES.md`) to run platform hard gates against constraints.
4. **Scored Matrix & Sensitivity Check:** Perform weighted scoring on surviving platforms and run $\pm 1$ sensitivity checks to identify fragile decisions.
5. **ADRs & Quality Attribute Scenarios:** Document choices using `engine/templates/ADR_TEMPLATE.md` and define NFR benchmarks using `engine/templates/QUALITY_ATTRIBUTE_SCENARIOS.md`.
6. **Saturation Stop-Gates:** Validate traceability and fault-injection scenarios using `engine/methodologies/05_SATURATION_STOP_GATES.md`.

*Note: Any legacy domain case studies provided in reference materials serve as historical operational finding illustrations and predate Lens 09.*
