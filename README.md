# Edge & Hostile-Environment Research Engine
### Domain-Agnostic Market, Consumer & Product Systems Research Framework

A lightweight, reusable research operating system designed to be ingested by AI models to conduct field-grounded research on product lines, market dynamics, consumer behavior, and edge system architectures across any emerging or hostile market environment.

---

## 1. How to Use This Engine

This repository separates the **Research Engine** from specific **Case Studies**:
1. Attach the `engine/` directory (under 15k tokens) to your model context:
   - `engine/PROTOCOL.md`: Evidence rules, invariant extraction, and output standards.
   - `engine/RESEARCH_LENSES.md`: 8 domain-agnostic analytical lenses (concurrency, degradation, collusion, cognitive ergonomics, GTM, etc.).
   - `engine/methodologies/`: Deep ethnographic and invariant deconstruction tools.
2. Fill out `BRIEF_TEMPLATE.md` with your target industry, geography, and constraints.
3. Paste `RUN_PROMPT.md` to execute the research.

---

## 2. Directory Structure

```text
├── README.md                  <- Overview and execution guide
├── RUN_PROMPT.md              <- Universal LLM execution prompt
├── BRIEF_TEMPLATE.md          <- User narration and domain input template
│
├── engine/                    <- THE CORE REUSABLE RESEARCH OPERATING SYSTEM
│   ├── PROTOCOL.md            <- Evidence tagging & execution rules
│   ├── RESEARCH_LENSES.md     <- Universal analytical lenses (01 to 08)
│   ├── methodologies/         <- 5 invariant and field deconstruction tools
│   └── templates/             <- Bias self-audit & field prompts
│
└── examples/                  <- ARCHIVED REFERENCE BENCHMARKS
    └── weighbridge_case_study/ <- Worked 19-aspect industrial metrology benchmark (Reference only)
```

---

## 3. Reference Case Study (Weighbridge Core)

The folder `examples/weighbridge_case_study/` contains a fully worked, 4,200+ line technical benchmark demonstrating how this engine was applied to an industrial weighbridge in rural India. It serves as an illustrative benchmark for the depth and structural rigor expected from the engine.
