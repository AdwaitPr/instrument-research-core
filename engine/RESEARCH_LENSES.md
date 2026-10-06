# Universal Analytical Research Lenses
Path: engine/RESEARCH_LENSES.md
Status: ACTIVE / CORE SPECIFICATION

Use these analytical lenses to evaluate any target product, market, or operational domain specified in the User Brief:

---

## Lens 01: State Concurrency & Transition Rigidity
- **Core Question:** Who or what changes system state concurrently, and where are the unsafe race conditions?
- **Analysis:** Map the operational flow as a deterministic finite-state transition. Identify where human intervention can skip validation steps, reverse irreversible states, or cause deadlocks.

---

## Lens 02: Degradation & Failure Topologies
- **Core Question:** How does the product function when 20%, 50%, or 80% of its operating environment fails?
- **Analysis:** Construct a combinatorial degradation ladder. Define explicit fallback states for zero-network conditions, unconditioned power, hardware peripheral failure, and operator panic.

---

## Lens 03: Queue Congestion & Latency Traps
- **Core Question:** At what operational throughput does the system induce operational collapse or user revolt?
- **Analysis:** Model transaction arrival vs. service rates. Identify the hard latency ceiling ($\tau_{\text{budget}}$) beyond which operators bypass the software entirely and revert to informal methods.

---

## Lens 04: Adversarial Collusion & Incentive Misalignment
- **Core Question:** Where do the economic incentives of the buyer, the daily operator, and the customer directly conflict?
- **Analysis:** Map the game-theoretic payoff matrix. Identify where parties collude to defraud the platform, extract informal bribes, or conceal transactions. Design bifurcated visibility (public verification vs. private commercial ledger).

---

## Lens 05: Sensory Hostility & Cognitive Cockpit Ergonomics
- **Core Question:** What environmental noise, visual fatigue, and literacy constraints impede the user interface?
- **Analysis:** Evaluate ambient sensory conditions (decibel noise, sunlight glare, dirt/dust, motor skill impairments). Require glanceable, high-contrast, multi-modal feedback (audio earcons, haptics) with zero modal traps.

---

## Lens 06: Regulatory Volatility & Evidentiary Proof
- **Core Question:** What statutory laws govern this domain, and how is legal evidence generated on-device?
- **Analysis:** Map statutory retention horizons, digital evidence admissibility rules, and anti-inspection defense views. Ensure audit records are cryptographic, append-only, and locally verifiable.

---

## Lens 07: Consumer Decision Mesh & Willingness-To-Pay (WTP)
- **Core Question:** Who signs the check, who uses the tool, who can veto the sale, and what insurance value does it provide?
- **Analysis:** Distinguish the economic buyer from the daily operator. Calculate price elasticity based on risk avoidance, penalty defense, or direct leakage recovery rather than generic productivity gains.

---

## Lens 08: Go-To-Market & Rural/Edge Distribution Flywheels
- **Core Question:** How does the product physically get installed, serviced, and monetized without cloud app stores?
- **Analysis:** Map trusted local channel partners, hardware repair networks, seasonal working capital cash cycles, and air-gapped update mechanisms.
