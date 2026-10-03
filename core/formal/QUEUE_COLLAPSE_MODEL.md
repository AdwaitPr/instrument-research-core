# Operational Queuing Theory, Heavy-Traffic Dynamics, and the Critical Threshold of Protocol Collapse (Q*)
Path: core/formal/QUEUE_COLLAPSE_MODEL.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, OIML R 76-1

## 1. Formal Queuing Parameter & Distribution Taxonomy

Commercial weighbridges, mandi gates, and industrial depot entrances operate as single-server queuing systems with non-homogeneous, high-variance arrivals. The operational parameters governing these systems are defined below:

### 1.1 Arrival Distribution Parameters
- $\lambda(t)$ [Arrival Rate]: Arrival intensity as a function of operational shift time (vehicles per second).
- $\lambda_{\text{peak}}$: Peak arrival intensity during burst windows (e.g., harvest morning arrivals: $35\text{--}50\text{ vehicles/hr} = 0.0097\text{--}0.0139\text{ veh/s}$).
- $\lambda_{\text{nominal}}$: Baseline arrival intensity during mid-day operations ($10\text{--}15\text{ vehicles/hr} = 0.0028\text{--}0.0042\text{ veh/s}$).
- $C_a^2 = \frac{\sigma_a^2}{(E[T_a])^2}$ [Squared Coefficient of Variation of Inter-arrival Times]:
  * $C_a^2 = 1.0$: Memoryless Poisson process (uncoordinated arrivals).
  * $C_a^2 > 1.5$: Clustered/platoon batch arrivals (convoy arrivals from harvesting combines or highway freight batches).

### 1.2 Service Time Breakdown (The Service Chain)
Mean service time $\mu^{-1} = E[S]$ is partitioned into four serial, non-overlapping phases:
$$\mu^{-1} = \tau_{\text{physical}} + \tau_{\text{settling}} + \tau_{\text{app}} + \tau_{\text{settlement}}$$

Where:
1. $\tau_{\text{physical}}$ [Mechanical Positioning]: Time taken for vehicle to mount platform, position wheels inside boundaries, and cut engine.
   - Empirical Range: $15\text{--}35\text{ seconds}$ (Tractor-trolleys: $20\text{ s}$; 12-wheel trucks: $35\text{ s}$).
2. $\tau_{\text{settling}}$ [Transducer Equilibrium Dwell]: Mandatory physical and statutory delay for load-cell dampening and verified stable frame emission.
   - Constrained by OIML R 76-1: $\tau_{\text{settling}} \ge 2.0\text{ seconds}$ (Nominal: $2.5\text{--}4.0\text{ s}$).
3. $\tau_{\text{app}}$ [Software Execution Budget]: Time spent by operator interacting with Android terminal (selecting vehicle, latching weight, verifying items).
   - Target Baseline: $\le 10\text{--}15\text{ seconds}$ (Legacy ERPs fail at $45\text{--}90\text{ s}$).
4. $\tau_{\text{settlement}}$ [Custodial & Slip Handoff]: Thermal slip printing, inspection, physical cash exchange, and gate barrier lift.
   - Empirical Range: $10\text{--}20\text{ seconds}$.

$$\text{Nominal Complete Service Cycle } (\mu^{-1}): 20\text{ s} + 3\text{ s} + 12\text{ s} + 15\text{ s} = 50\text{ seconds} \implies \mu_{\text{nominal}} \approx 72\text{ trucks/hr}$$

### 1.3 Behavioral Queue Phenomena Definitions
- **Balking ($B$):** An arriving vehicle arrives at the yard approach, observes physical tailback $Q(t)$, calculates expected wait time $W_q > W_{\text{tolerable}}$, and departs without entering the service lane.
- **Reneging ($R$):** A vehicle enters the approach lane, waits in queue for duration $t_{\text{wait}}$, encounters an operational stall, and turns around or leaves via an escape cut before reaching the scale platform.
- **Priority-Jumping / Line-Cutting ($J$):** A vehicle bypasses physical FIFO order via informal influence, broker collusion, or physical aggression, displacing queued vehicles.
- **Protocol Bypass ($\Phi$):** The operational breakdown where operator and driver collude to artificially compress $\mu^{-1}$ by omitting software or metrological steps (e.g., manual tare reuse, capturing weight in motion).

---

## 2. Heavy-Traffic Dynamics & Software Time Budget ($\tau_{\text{app}}$)

### 2.1 Kingman's Heavy-Traffic Approximation for $G/G/1$ Queues
When utilization $\rho = \frac{\lambda}{\mu} \to 1^-$, the steady-state mean queue waiting time $W_q$ is given by:
$$W_q \approx \left(\frac{\rho}{1 - \rho}\right) \left(\frac{C_a^2 + C_s^2}{2}\right) \frac{1}{\mu}$$

Queue length in buffer $L_q$ follows via Little's Law:
$$L_q = \lambda W_q \approx \left(\frac{\rho^2}{1 - \rho}\right) \left(\frac{C_a^2 + C_s^2}{2}\right)$$

### 2.2 Sensitivity Analysis: The Latency Multiplier
Because Kingman's formula contains the asymptotic term $\frac{\rho}{1 - \rho}$, delays scale non-linearly with software sluggishness:

| System State | $\lambda$ (trucks/hr) | $\tau_{\text{app}}$ (s) | Total Service $\mu^{-1}$ (s) | Utilization ($\rho$) | $W_q$ (Kingman, $C_a^2=C_s^2=1$) | Queue Tailback ($L_q$) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Optimized Native** | 45 | **8 s** | 46 s | **0.575** | **31.1 seconds** | **0.39 trucks** |
| **Acceptable Native**| 45 | **15 s** | 53 s | **0.662** | **51.8 seconds** | **0.65 trucks** |
| **Sluggish UI / Lag** | 45 | **30 s** | 68 s | **0.850** | **192.6 seconds (3.2 min)** | **2.41 trucks** |
| **Legacy Desktop/Web**| 45 | **45 s** | 83 s | **1.037** | **$\infty$ (Queue Explodes)** | **Unbounded Growth** |

### 2.3 Mathematical Derivation of Upper Bound for $\tau_{\text{app}}$
To prevent systemic queue explosion during peak arrival windows, the system utilization must not exceed a stability threshold $\rho_{\text{target}} = 0.80$. Given a peak design arrival rate $\lambda_{\text{peak}}$:

$$\rho = \lambda_{\text{peak}} \times \mu^{-1} \le \rho_{\text{target}}$$
$$\mu^{-1} = \tau_{\text{physical}} + \tau_{\text{settling}} + \tau_{\text{app}} + \tau_{\text{settlement}} \le \frac{\rho_{\text{target}}}{\lambda_{\text{peak}}}$$
$$\tau_{\text{app}} \le \frac{\rho_{\text{target}}}{\lambda_{\text{peak}}} - (\tau_{\text{physical}} + \tau_{\text{settling}} + \tau_{\text{settlement}})$$

**Field Benchmark Calculation (Mandi Morning Harvest Peak):**
- Peak Arrival Rate: $\lambda_{\text{peak}} = 45\text{ trucks/hr} = 0.0125\text{ trucks/s}$
- Stability Target: $\rho_{\text{target}} = 0.80$
- Maximum Allowable Total Service Time: $\mu^{-1} \le \frac{0.80}{0.0125} = 64.0\text{ seconds}$
- Fixed Physical Overhead:
  * $\tau_{\text{physical}} = 30.0\text{ s}$
  * $\tau_{\text{settling}} = 3.0\text{ s}$
  * $\tau_{\text{settlement}} = 15.0\text{ s}$
  * Total Physical Overhead $= 48.0\text{ seconds}$

$$\mathbf{\tau_{\text{app}} \le 64.0\text{ s} - 48.0\text{ s} = 16.0\text{ seconds}}$$

**Absolute Rule:** If the native Android software requires more than **16 seconds** of total operator interaction per vehicle, the system is mathematically guaranteed to drive the facility into queue collapse during peak harvest arrival surges.

---

## 3. The Critical Threshold of Protocol Collapse ($Q^*$)

### 3.1 Operational Definition of $Q^*$
The **Critical Threshold of Protocol Collapse ($Q^*$)** is the physical queue length (number of waiting vehicles) at which the psychological, environmental, and commercial pressure on the operator exceeds their adherence to verification protocols, causing the probability of intentional software bypass to exceed $50\%$.

### 3.2 The Logistic Protocol Bypass Model
The probability of an operator executing an unauthorized bypass on an incoming transaction given current queue length $Q$ is modeled by the logistic function:

$$P(\text{Bypass} \mid Q) = \frac{1}{1 + e^{-k(Q - Q^*)}}$$

Where:
- $Q$: Instantaneous physical queue length observable by the operator.
- $Q^*$: The inflection point (queue length where bypass probability is exactly 0.50).
- $k$: Operator stress sensitivity parameter ($k > 0$). High $k$ indicates a brittle operator who abruptly switches from strict compliance to total bypass once queue exceeds $Q^*$.

Bypass Probability P(Bypass | Q)
1.0 ┼                                                ╭─────────────────
│                                               ╭╯
0.8 ┼                                              ╭╯
│                                             ╭╯
0.5 ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┼ (Q*, 0.5)
│                                           ╭╯
0.2 ┼                                         ╭╯
│                                       ╭╯
0.0 ┼───────────────────────────────────────╯
└──────────────────┬────────────────────────────┬──────────────────
0                  Q* - 2                     Q*                 Q (Queue Length)

### 3.3 The Economic and Operational Dynamics of $Q^*$
1. **The Physical Choke ($Q_{\text{choke}}$):** Yard physical approach capacity is bounded. For typical rural weighbridges, $Q_{\text{choke}} \approx 8\text{--}15\text{ vehicles}$. Beyond $Q_{\text{choke}}$, vehicles block public thoroughfares, attracting police intervention, horn-blaring, and verbal hostility.
2. **The Operator Coping Mechanism:** The operator cannot speed up $\tau_{\text{physical}}$ (trucks move slowly). They cannot speed up $\tau_{\text{settlement}}$ (cash takes time to count). The **only variable within operator control is $\tau_{\text{settling}} + \tau_{\text{app}}$**.
3. **The Bypass Arsenal:**
   - *Bypass 1 (Tare Reuse):* Operator enters previous day's tare or default tare $\to$ **Saves 100% of Pass 2 cycle ($\approx 50\text{ seconds}$)**.
   - *Bypass 2 (Motion Capture):* Operator forces weight lock before vehicle settles $\to$ **Saves 3 seconds per truck**.
   - *Bypass 3 (Offline Carbon Chit):* Operator turns off terminal, claims "system down," and issues unverified paper tickets $\to$ **Reduces $\tau_{\text{app}}$ to 2 seconds**.

---

## 4. Discrete-Event Simulation Specification (`queue_sim.py`)

The following simulation model implements the non-homogeneous arrival dynamics, Kingman service variations, and logistic operator protocol collapse.

```python
# UNCHECKED/UNRUN
# Discrete-Event Simulation of Mandi Weighbridge Queuing and Protocol Collapse
# Language: Python 3.10+ (Standard Library Heapq Engine)

import heapq
import math
import random
from dataclasses import dataclass
from typing import List, Tuple

@dataclass
class Vehicle:
    vehicle_id: int
    arrival_time: float
    is_two_pass: bool
    bypassed: bool = False
    wait_time: float = 0.0
    service_time: float = 0.0

class WeighbridgeSimulation:
    def __init__(
        self,
        shift_duration_hours: float = 12.0,
        tau_physical_mean: float = 25.0,
        tau_physical_sd: float = 5.0,
        tau_settling_legal: float = 3.0,
        tau_app_nominal: float = 12.0,
        tau_settlement_mean: float = 15.0,
        q_star: float = 10.0,
        k_sensitivity: float = 0.8,
        leakage_cost_per_bypass_paise: int = 432000 # Rs 4,320 in integer paise
    ):
        self.sim_duration_sec = shift_duration_hours * 3600.0
        self.tau_phys_mean = tau_physical_mean
        self.tau_phys_sd = tau_physical_sd
        self.tau_settling_legal = tau_settling_legal
        self.tau_app_nominal = tau_app_nominal
        self.tau_settle_mean = tau_settlement_mean
        self.q_star = q_star
        self.k = k_sensitivity
        self.leakage_per_bypass = leakage_cost_per_bypass_paise
        
        # State Variables
        self.current_time: float = 0.0
        self.event_queue: List[Tuple[float, str, int]] = []
        self.waiting_line: List[Vehicle] = []
        self.server_busy: bool = False
        
        # Metrics
        self.completed_vehicles: List[Vehicle] = []
        self.total_bypasses: int = 0
        self.queue_length_log: List[Tuple[float, int]] = []
        
    def arrival_intensity(self, t_sec: float) -> float:
        """Non-homogeneous arrival curve with morning and evening peak."""
        hour = t_sec / 3600.0
        # Peak 1: Hour 2-4 (08:00 - 10:00 AM) @ 45 trucks/hr
        # Peak 2: Hour 8-10 (02:00 - 04:00 PM) @ 30 trucks/hr
        # Baseline: 12 trucks/hr
        base = 12.0 / 3600.0
        peak1 = 33.0 / 3600.0 * math.exp(-0.5 * ((hour - 3.0) / 1.0) ** 2)
        peak2 = 18.0 / 3600.0 * math.exp(-0.5 * ((hour - 9.0) / 1.2) ** 2)
        return base + peak1 + peak2

    def sample_service_time(self, current_queue: int) -> Tuple[float, bool]:
        """Calculates service time and evaluates logistic bypass probability."""
        # Evaluate bypass probability
        p_bypass = 1.0 / (1.0 + math.exp(-self.k * (current_queue - self.q_star)))
        is_bypass = random.random() < p_bypass
        
        t_phys = max(10.0, random.gauss(self.tau_phys_mean, self.tau_phys_sd))
        t_settle = 0.5 if is_bypass else self.tau_settling_legal # Bypass skips settling
        t_app = 2.0 if is_bypass else self.tau_app_nominal        # Bypass skips app checks
        t_pay = max(5.0, random.expovariate(1.0 / self.tau_settle_mean))
        
        total_service = t_phys + t_settle + t_app + t_pay
        return total_service, is_bypass

    def run(self):
        # Schedule initial arrival
        next_arrival_time = random.expovariate(self.arrival_intensity(0.0))
        heapq.heappush(self.event_queue, (next_arrival_time, "ARRIVAL", 1))
        
        vehicle_id_counter = 1
        
        while self.event_queue and self.current_time < self.sim_duration_sec:
            event_time, event_type, v_id = heapq.heappop(self.event_queue)
            self.current_time = event_time
            self.queue_length_log.append((self.current_time, len(self.waiting_line)))
            
            if event_type == "ARRIVAL":
                v = Vehicle(vehicle_id=v_id, arrival_time=self.current_time, is_two_pass=True)
                if not self.server_busy:
                    self.server_busy = True
                    s_time, bypassed = self.sample_service_time(len(self.waiting_line))
                    v.bypassed = bypassed
                    v.service_time = s_time
                    if bypassed: self.total_bypasses += 1
                    heapq.heappush(self.event_queue, (self.current_time + s_time, "DEPARTURE", v.vehicle_id))
                    self.completed_vehicles.append(v)
                else:
                    self.waiting_line.append(v)
                    
                # Schedule next arrival
                rate = self.arrival_intensity(self.current_time)
                dt = random.expovariate(rate) if rate > 0 else 300.0
                vehicle_id_counter += 1
                heapq.heappush(self.event_queue, (self.current_time + dt, "ARRIVAL", vehicle_id_counter))
                
            elif event_type == "DEPARTURE":
                if self.waiting_line:
                    v_next = self.waiting_line.pop(0)
                    v_next.wait_time = self.current_time - v_next.arrival_time
                    s_time, bypassed = self.sample_service_time(len(self.waiting_line))
                    v_next.bypassed = bypassed
                    v_next.service_time = s_time
                    if bypassed: self.total_bypasses += 1
                    heapq.heappush(self.event_queue, (self.current_time + s_time, "DEPARTURE", v_next.vehicle_id))
                    self.completed_vehicles.append(v_next)
                else:
                    self.server_busy = False

    def generate_report(self):
        total_served = len(self.completed_vehicles)
        avg_wait = sum(v.wait_time for v in self.completed_vehicles) / max(1, total_served)
        max_q = max(q for _, q in self.queue_length_log) if self.queue_length_log else 0
        total_leakage_inr = (self.total_bypasses * self.leakage_per_bypass) / 100.0
        
        print("=== QUEUE COLLAPSE SIMULATION RESULTS ===")
        print(f"Shift Duration: {self.sim_duration_sec / 3600.0:.1f} Hours")
        print(f"Total Vehicles Processed: {total_served}")
        print(f"Average Waiting Time: {avg_wait:.1f} seconds ({avg_wait / 60.0:.2f} mins)")
        print(f"Peak Queue Length Observed: {max_q} vehicles")
        print(f"Total Bypasses Initiated: {self.total_bypasses} ({self.total_bypasses / max(1, total_served) * 100:.1f}%)")
        print(f"Estimated Direct Financial Leakage: INR {total_leakage_inr:,.2f}")

if __name__ == "__main__":
    sim = WeighbridgeSimulation()
    sim.run()
    sim.generate_report()
```


## 5. Field Measurement Protocol (Gemba Stopwatch Methodology)

### 5.1 Physical Setup & Observation Station
- **Observer Position:** Outside the weigh cabin with clear line-of-sight to the weigh platform ramp and approach road queue.
- **Tools:** Dual stopwatch/timing app, optical distance marker (chalk line on approach road marking vehicle spaces), tally board.

### 5.2 Field Tally Sheet Layout
TALLY SHEET: WEIGHBRIDGE QUEUING & BYPASS AUDIT
Location / Yard: _______________________ Date / Shift: ________________
Observer: _____________________________ Commodity: ___________________
[Cols: Tx_ID | Arrive_Time | Enter_Platform | Settle_Lock | Exit_Platform | Q_Length_at_Entry | Stable_Flag_Waited (Y/N) | Tare_Mode (Real/Frozen) | Bypass_Observed (Y/N)]
001   | 08:04:12    | 08:04:35       | 08:04:40    | 08:05:15      | 3                 | Y                       | Real                    | N
002   | 08:05:01    | 08:05:22       | 08:05:25    | 08:05:58      | 5                 | Y                       | Real                    | N
...
024   | 09:14:22    | 09:14:40       | 09:14:41    | 09:14:52      | 14                | N (Forced Lock)         | Frozen Tare (F8)        | Y (CRITICAL Q*)

### 5.3 Deriving $Q^*$ from Field Tally Data
1. Bucket transactions by observed queue length: $Q \in [0\text{--}3], [4\text{--}6], [7\text{--}9], [10\text{--}12], [13\text{--}15], [>15]$.
2. For each bucket, calculate empirical bypass frequency:
   $$\hat{P}(\text{Bypass} \mid Q) = \frac{\sum \text{Bypasses Observed in Bucket}}{\text{Total Transactions in Bucket}}$$
3. Plot $\hat{P}$ vs $Q$ and identify the point where $\hat{P}$ crosses $0.50$. This value is the calibrated $Q^*$ for that operational site.

## 6. Repo Impact Analysis
- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **15-Second Software Execution Budget ($\tau_{\text{app}} \le 15\text{ s}$)** as a hard system invariant. Any UI screen flow requiring $>15\text{ s}$ is rejected.
  * Adds $Q^*$ threshold monitoring: native app tracks queue velocity locally and flags "High Stress Shift" when transaction inter-arrival times compress below $\mu^{-1}$.
- **Impact on Room Schemas & UI Architecture:**
  * Prohibits multi-page form wizards and text-entry search dialogs on transaction hot paths.
  * Mandates single-screen execution with physical volume-key shortcuts to ensure $\tau_{\text{app}}$ remains within the 8–15s envelope.

## 7. Digest Card
- **Key Invariants:** Kingman Heavy-Traffic Scaling ($\frac{\rho}{1-\rho} \times \frac{C_a^2+C_s^2}{2} \times \frac{1}{\mu}$); Non-Homogeneous Finite Peak Bursts ($\lambda(t) > \mu$ for finite $T$); Service Decomposition ($\mu^{-1} = \tau_{\text{physical}} + \tau_{\text{settling}} + \tau_{\text{app}} + \tau_{\text{settlement}}$); Hard Software Budget ($\tau_{\text{app}} \le 15\text{ seconds}$).
- **The $Q^*$ Invariant:** Operator bypass probability follows $P(\text{Bypass} \mid Q) = \frac{1}{1 + e^{-k(Q - Q^*)}}$. Bypasses are human coping mechanisms to avoid queue gridlock ($Q_{\text{choke}}$).
- **Core Tensions:** Statutory settling delay ($\tau_{\text{settling}} \ge 2\text{ s}$) vs line clearance pressure; two-pass weighment deadlocks vs single-pass tare drift risk.
- **Top 3 Gemba Hooks:**
  1. Measure empirical $Q^*$ by correlating queue length with manual tare overrides.
  2. Measure true $\tau_{\text{settlement}}$ (cash counting and paper chit delays) under peak rush.
  3. Validate physical positioning overhead ($\tau_{\text{physical}}$) for tractor-trolleys vs trucks.
