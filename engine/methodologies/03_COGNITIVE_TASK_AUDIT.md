# HEAD 3: COGNITIVE TASK ANALYSIS & PHYSICAL FRICTION ERGONOMICS
# LOCATION: /core/methodologies/03_COGNITIVE_TASK_AUDIT.md

## 1. THEORETICAL FOUNDATION
- MIL-STD-1472G / ISO 9241: Human Engineering Criteria for Military and Industrial Systems.
- NASA Task Load Index (NASA-TLX): Multi-dimensional subjective workload assessment.
- Rasmussen’s Skills, Rules, Knowledge (SRK) Framework: Sensory-motor performance under degraded conditions.
- Endsley’s Situation Awareness Model: Perception, Comprehension, and Projection under environmental noise.

## 2. PHYSICAL & COGNITIVE STRESS MATRIX

| Dimension | Standard Operational Limit | Extreme Field Condition | Measurement Method | System Design Constraint |
| :--- | :--- | :--- | :--- | :--- |
| **Acoustic Noise** | $< 70\text{ dBA}$ | $> 85\text{ dBA}$ (Engines, metal clatter) | Calibrated Sound Level Meter | Screen-only or high-power audio; no subtle UI chimes. |
| **Illumination / Glare** | $300 - 500\text{ lux}$ | $> 10,000\text{ lux}$ (Direct outdoor glare) | Photometer / Lux Meter | Extreme contrast monochrome; zero subtle greys. |
| **Mechanical Vibration** | $< 0.5\text{ m/s}^2$ | $> 2.0\text{ m/s}^2$ (Cabins, heavy plant) | 3-axis Accelerometer | Targets $\ge 64\text{dp}$; zero precision swipe gestures. |
| **Manual Dexterity** | Bare, dry fingers | Greasy, wet, dusty, or gloved hands | Input error rate tracking | Physical button triggers or fixed numeric keypads. |
| **Input Latency** | $< 1.0\text{s}$ per keypress | Single-hand thumb input under time pressure | Millisecond timestamp logs | Sub-50ms visual/haptic response; zero soft-keyboard entry. |
| **Interruption Rate** | $< 2$ per hour | $> 10$ per hour (Phone calls, shouts) | Observational tally | Atomic draft persistence per keystroke to survive death. |
| **NASA-TLX Load** | Raw score $< 40$ | Raw score $> 70$ (Sensory/mental overload) | Weighted TLX assessment | Single-intent screens; zero multi-level wizard trees. |

## 3. COGNITIVE & ERGONOMIC FIELD PROMPTS
1. "On a scale of 1 to 10, what is the mental exhaustion level at the end of a rush shift?"
2. "What is the loudest ambient noise condition encountered during standard operations?"
3. "Show me the worst sunlight glare or lighting condition that occurs during the working day."
4. "How often must you enter data with only one hand while holding or operating another tool?"
5. "How often are you forced to restart a transaction because of an interruption or system timeout?"
6. "What is the smallest text or icon size on this screen you can read without squinting?"
7. "What happens to your input speed when a phone call arrives in the middle of data capture?"
8. "What was the specific physical situation the last time an operator made a critical input mistake?"
9. "How contaminated (grease, water, mud, dust) are your hands when you operate this terminal?"
10. "What physical hardware feedback (loud speaker, heavy vibration) reliably cuts through this workspace?"

## 4. RESEARCHER FAILURE MODES
- **Cumulative Load Blindness:** Evaluating noise, glare, and fatigue in isolation rather than their compounded effect.
- **Hardware Homogeneity Assumption:** Testing exclusively on modern high-end test hardware rather than target-class field devices.
- **Self-Report Bias:** Accepting an operator's claim that a task is "easy" while watching high error and correction rates.
- **Neglect of Interruption Cost:** Overlooking the cognitive cost of resuming a workflow after an operational distraction.
