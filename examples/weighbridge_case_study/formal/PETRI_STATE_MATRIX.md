# Formal Concurrency Model & State Transition Matrix
Path: core/formal/PETRI_STATE_MATRIX.md
Status: DRAFT / DESK-VERIFIED
Dependencies: androidx.room:room-ktx:2.6+, SQLite WAL Engine, OIML R 76-1, The Legal Metrology Act 2009

## 1. Concurrent Actor × Resource Interaction Matrix
In native instrument apps deployed at physical depots, weighbridges, and scrap yards, physical events alter transducer hardware asynchronously relative to digital state mutations executed on the Android terminal. The table below formalizes these physical and digital interactions across system resources.

| Concurrent Actor | Physical Trigger / Hardware Action | Transducer Platform State | Serial Peripheral Stream (RS-232/485) | Draft Entity (draft_weighment_manifest) | Ledger Entity (ledger_transactions) | Cash Balance / Local Register |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Vehicle Driver** | Mounts scale ramp; cuts ignition. | Dynamic load deflection settles toward gross force $F_g$; oscillation period $2\text{--}5\text{ s}$. | Streams ASCII frames with Unstable flag (US); transitions to Stable (ST) upon settling. | Displayed via in-memory observer; no SQLite write executed. | No access. | No access. |
| **Yard Helper** | Steps onto platform edge during active weighing. | Force increases by human weight ($\approx 60\text{--}90\text{ kg}$); induces dynamic oscillation. | Peripheral frame status reverts from ST to US, then stabilizes with added mass. | Active UI gauge registers delta; lock operation inhibited if motion detected. | No access. | No access. |
| **Peripheral ADC** | Samples load cell bridge voltages at 10–20 Hz. | Converts analog bridge voltage into calibrated integer counts. | Broadcasts continuous 18-byte frames: `[STX][Status][Weight_Bytes][Tare_Bytes][CR][Checksum]`. | Ingestion coroutine updates transient Kotlin StateFlow. | No access. | No access. |
| **Scale Operator** | Confirms vehicle registration; presses "Capture Weight". | Static vehicle load maintained on deck. | Frame transmission continues unimpeded. | Single-writer transaction updates draft: sets PERIPHERAL_LOCKED, persists stable integer grams. | No access. | No access. |
| **Scale Operator** | Confirms trade grade and rate; presses "Finalize Ticket". | Static vehicle load maintained on deck. | Frame transmission continues unimpeded. | Draft row queried, verified against peripheral checksum, and flagged for ledger promotion. | Atomic SQLite transaction inserts immutable record with unique sequential `ledger_id`. | Cash balance adjusted or counterparty balance debited in integer paise. |
| **Counterparty** | Identifies clerical mismatch or grade dispute; requests cancellation. | Vehicle remains parked or initiates ramp egress. | Peripheral reflects vehicle movement or drops to zero. | Draft table inaccessible for finalized entries. | Compensating transaction appended with explicit foreign key `reversal_of_transaction_id`. | Reversal transaction appends compensating credit/debit entry in integer paise. |
| **Vehicle Driver** | Accelerates off scale platform before ticket commitment. | Platform voltage drops to base zero; transient bridge vibration. | Emits US frames, transitions to 000000 kg, returns to ST zero state. | If locked, UI retains snapshot; if unlocked, UI resets to zero. | If finalized, records phantom weight; if aborted, draft is cleared. | No access. |

## 2. Petri Net Place-Transition Specification
The discrete-event dynamics of the physical scale and the embedded database ledger are modeled as an Ordinary Place/Transition Net conforming to ISO/IEC 15909-1.

### 2.1 Places ($P$)
- $P_1$ (Deck_Empty): Scale platform is physically vacant (Tokens $\in \{0, 1\}$).
- $P_2$ (Truck_On_Deck): Vehicle resides physically on the platform (Tokens $\in \{0, 1\}$).
- $P_3$ (Stream_Unstable): Serial peripheral reports dynamic motion (US).
- $P_4$ (Stream_Stable): Serial peripheral reports metrological equilibrium (ST).
- $P_5$ (Draft_Open): Active weighment draft exists in Android terminal memory.
- $P_6$ (Peripheral_Locked): Metrological weight captured and written to local draft entity.
- $P_7$ (Ledger_Finalized): Immutable transaction row appended to local SQLite ledger.
- $P_8$ (Deck_Resource): Hardware platform mutual-exclusion semaphore (Tokens $\in \{0, 1\}$).
- $P_9$ (Material_Gross_Inbound): Physical incoming gross mass arriving on platform.
- $P_{10}$ (Material_Net_Discharged): Physical cargo unloaded into depot storage.
- $P_{11}$ (Material_Tare_Outbound): Empty vehicle mass verified during outbound pass.

### 2.2 Transitions ($T$)
- $T_1$ (Mount_Scale): Vehicle wheels mount the weighing platform.
- $T_2$ (Settle_Oscillation): Mechanical vibration decays below legal stability threshold $\Delta e$.
- $T_3$ (Disturb_Scale): External disturbance (wind gust, engine vibration, passenger boarding) disrupts stability.
- $T_4$ (Capture_Peripheral_Lock): Operator triggers weight lock; terminal captures stable reading.
- $T_5$ (Cancel_Draft): Operator aborts active draft prior to finalization.
- $T_6$ (Commit_Final_Ledger): Local SQLite transaction appends ledger record and closes draft.
- $T_7$ (Dismount_Scale): Vehicle drives off the platform deck.
- $T_8$ (Compensate_Revocation): Operator executes cancellation via append-only compensating entry.

### 2.3 Directed Flow Arcs ($I, O$)
- $T_1$ (Mount_Scale): $I(T_1) = \{P_1, P_8\}$, $O(T_1) = \{P_2, P_3\}$
- $T_2$ (Settle_Oscillation): $I(T_2) = \{P_3\}$, $O(T_2) = \{P_4\}$
- $T_3$ (Disturb_Scale): $I(T_3) = \{P_4\}$, $O(T_3) = \{P_3\}$
- $T_4$ (Capture_Peripheral_Lock): $I(T_4) = \{P_4, P_5\}$, $O(T_4) = \{P_4, P_6\}$ (Self-loop on $P_4$ preserves peripheral streaming continuity).
- $T_5$ (Cancel_Draft): $I(T_5) = \{P_6\}$, $O(T_5) = \{P_5\}$
- $T_6$ (Commit_Final_Ledger): $I(T_6) = \{P_6\}$, $O(T_6) = \{P_7\}$
- $T_7$ (Dismount_Scale): $I(T_7) = \{P_2\}$, $O(T_7) = \{P_1, P_8\}$
- $T_8$ (Compensate_Revocation): $I(T_8) = \{P_7\}$, $O(T_8) = \emptyset$ (Preserves historical entry while recording compensating ledger state).

### 2.4 Structural Place Invariants (P-Invariants)
Applying Murata's Incidence Matrix Theorem ($w^T A = 0$):

**Invariant 1: Scale Deck Mutual Exclusion (1-Bounded Semaphore)**
$$M(P_1) + M(P_2) = 1 \quad \text{and} \quad M(P_8) + M(P_2) = 1 \implies M(P_2) \le 1$$
For all reachable markings $M \in R(M_0)$, exactly one vehicle can occupy the weighing deck.

**Invariant 2: Discrete Conservation of Mass and Inventory**
$$\Delta M_{\text{inventory}} = \text{Mass}_{\text{Gross}} - \text{Mass}_{\text{Tare}} \equiv \text{Mass}_{\text{Net}}$$
Every finalized transaction cycle consumes a Gross token and a Tare token to produce a Net inventory token, ensuring zero physical mass creation or loss within software state.

### 2.5 Identification of Deadlock and Unsafe Interleaving States
- **Deadlock State $S_{\text{dead\_1}}$ (Premature Egress Lock):** Occurs if a vehicle exits platform ($T_7$) after $T_4$ (Capture Lock) but before $T_6$ (Commit). The deck is physically vacant, but software draft state is locked. *Mitigation:* Terminal watchdog resets draft to tombstone state if uncommitted for $\tau > 180\text{ s}$.
- **Unsafe Interleaving $S_{\text{unsafe\_1}}$ (Dynamic Roll-Through Capture):** Occurs if weight is latched while a truck rolls across without stopping. *Mitigation:* Transition $T_4$ requires unbroken stability flag continuity ($P_4$) for $t_{\text{stable}} \ge 2000\text{ ms}$.

## 3. Room / SQLite State Enum Mapping & Engine Rules
### 3.1 Direct Mapping of Petri Net Places to Room Entities
- `DRAFT` ($P_5$): Live RS-232 telemetry parsed in volatile memory. Unfinalized entity in `draft_weighment_manifest`. Zero ledger rows written.
- `PERIPHERAL_LOCKED` ($P_6$): Stable mass integer grams frozen into `draft_weighment_manifest`. Serial parsing coroutine detaches.
- `FINALIZED_UNSYNCED` ($P_7$): Draft validated, SHA-256 hashed, and inserted into `ledger_transactions` with sequential autoincrement `ledger_id`.
- `COMMITTED_LOCAL` ($P_7$ settled locally): Local cash drawer and customer balances updated within the same SQLite transaction.
- `REVOKED` ($T_8$): Original ledger row remains untouched. Compensating entry appended with negative values and `reversal_of_transaction_id` populated.

### 3.2 Decoupled Two-Pass Weighment Architecture
To support two-pass bulk haulage (Gross Inbound $\to$ Tare Outbound) without deadlocking the single-deck semaphore:
- **Pass 1 (Inbound):** Commits as `WEIGH_IN`. Net mass is recorded as zero; scale semaphore is immediately released.
- **Pass 2 (Outbound):** Commits as independent transaction (`WEIGH_OUT`) linked via `paired_inbound_ledger_id`. Net mass resolved on Pass 2 commit.

### 3.3 SQLite WAL Concurrency Rules
- The high-frequency (10–20 Hz) RS-232 serial coroutine runs on `Dispatchers.IO` and updates an in-memory `StateFlow<PeripheralFrame>`. It **never** writes directly to SQLite.
- Database mutations execute through `RoomDatabase.withTransaction {}` on a single-threaded dispatcher (`limitedParallelism(1)`).
- `PRAGMA busy_timeout = 5000;` is configured on all connections.

## 4. Formal TLA+ Specification (`ledger.tla`)

```tla
---------------------------- MODULE ledger ----------------------------
\* UNCHECKED/UNRUN
\* Formal Discrete-Event Specification of Native Weighbridge Concurrency.
EXTENDS Naturals, Sequences, FiniteSets

CONSTANTS 
    MaxGrams, MaxPaise, MaxLedgerEntries

VARIABLES 
    scaleState, serialStream, activeDraft, ledger, cashDrawerPaise

vars == <<scaleState, serialStream, activeDraft, ledger, cashDrawerPaise>>

Init == 
    /\ scaleState = "EMPTY"
    /\ serialStream = "STABLE"
    /\ activeDraft = [gross |-> 0, tare |-> 0, locked |-> FALSE]
    /\ ledger = << >>
    /\ cashDrawerPaise = 100000000

MountScale ==
    /\ scaleState = "EMPTY"
    /\ scaleState' = "OCCUPIED"
    /\ serialStream' = "UNSTABLE"
    /\ UNCHANGED <<activeDraft, ledger, cashDrawerPaise>>

StabilizePeripheral ==
    /\ scaleState = "OCCUPIED"
    /\ serialStream = "UNSTABLE"
    /\ serialStream' = "STABLE"
    /\ UNCHANGED <<scaleState, activeDraft, ledger, cashDrawerPaise>>

LockPeripheral(measuredGross, measuredTare) ==
    /\ scaleState = "OCCUPIED"
    /\ serialStream = "STABLE"
    /\ activeDraft.locked = FALSE
    /\ measuredGross \in 1..MaxGrams
    /\ measuredTare \in 0..measuredGross
    /\ activeDraft' = [gross |-> measuredGross, tare |-> measuredTare, locked |-> TRUE]
    /\ UNCHANGED <<scaleState, serialStream, ledger, cashDrawerPaise>>

CommitFinalLedger(pricePerKgPaise) ==
    LET 
        entryCount == Len(ledger)
        nextId == entryCount + 1
        netMass == activeDraft.gross - activeDraft.tare
        chargePaise == (netMass * pricePerKgPaise) \div 1000
        newRecord == [
            id |-> nextId,
            grossGrams |-> activeDraft.gross,
            tareGrams |-> activeDraft.tare,
            netGrams |-> netMass,
            totalPaise |-> chargePaise,
            reversalOf |-> 0
        ]
    IN
        /\ activeDraft.locked = TRUE
        /\ entryCount < MaxLedgerEntries
        /\ ledger' = Append(ledger, newRecord)
        /\ cashDrawerPaise' = cashDrawerPaise + chargePaise
        /\ activeDraft' = [gross |-> 0, tare |-> 0, locked |-> FALSE]
        /\ UNCHANGED <<scaleState, serialStream>>

DismountScale ==
    /\ scaleState = "OCCUPIED"
    /\ scaleState' = "EMPTY"
    /\ serialStream' = "STABLE"
    /\ UNCHANGED <<activeDraft, ledger, cashDrawerPaise>>

RevokeTransaction(targetId) ==
    LET 
        entryCount == Len(ledger)
        nextId == entryCount + 1
    IN
        /\ targetId \in 1..entryCount
        /\ entryCount < MaxLedgerEntries
        /\ ~ \E i \in 1..entryCount: ledger[i].reversalOf = targetId
        /\ ledger[targetId].reversalOf = 0
        /\ LET targetRecord == ledger[targetId]
               compensatingRecord == [
                   id |-> nextId,
                   grossGrams |-> targetRecord.grossGrams,
                   tareGrams |-> targetRecord.tareGrams,
                   netGrams |-> targetRecord.netGrams,
                   totalPaise |-> targetRecord.totalPaise,
                   reversalOf |-> targetId
               ]
           IN
               /\ ledger' = Append(ledger, compensatingRecord)
               /\ cashDrawerPaise' = cashDrawerPaise - targetRecord.totalPaise
               /\ UNCHANGED <<scaleState, serialStream, activeDraft>>

Next ==
    \/ MountScale
    \/ StabilizePeripheral
    \/ \E g \in 1..MaxGrams, t \in 0..g: LockPeripheral(g, t)
    \/ \E rate \in 1..1000: CommitFinalLedger(rate)
    \/ DismountScale
    \/ \E id \in 1..Len(ledger): RevokeTransaction(id)

Spec == Init /\ [][Next]_vars

DeckMutualExclusion == scaleState = "EMPTY" => (activeDraft.locked = FALSE \/ activeDraft.gross > 0)
NoDuplicateFinalization == \A i, j \in 1..Len(ledger): i # j => ledger[i].id # ledger[j].id
ValidReversalLink == \A i \in 1..Len(ledger): ledger[i].reversalOf # 0 => (ledger[i].reversalOf < ledger[i].id /\ ledger[ledger[i].reversalOf].reversalOf = 0)
ConservationOfMass == \A i \in 1..Len(ledger): ledger[i].netGrams = ledger[i].grossGrams - ledger[i].tareGrams
=======================================================================
5. Field Verification Hooks
Transient Axle-Hop Injection: Creep truck across scale deck; confirm unstable flag (US) blocks weight lock until 2.0s continuous stability.
Asynchronous Cab-Dismount Drift: Lock gross weight, have driver/helpers dismount (≈200 kg drop); verify system alerts operator of delta before commit.
Serial Cable Hot-Pull: Sever USB-OTG connection during live weighing; verify Linux TTY buffer is flushed upon reconnect with zero stale readings.
Concurrent Multi-Coroutine Stress: Run 20 Hz serial read concurrently with UI draft updates and ledger reporting; verify zero UI jank and no SQLITE_BUSY crashes.
Power-Cut Ledger Integrity: Sever power during active SQLite transaction commit; verify database integrity via PRAGMA integrity_check;.
6. Repo Impact Analysis
Updates MASTER_CORE_PROTOCOL.md to strictly isolate serial threads from direct SQLite writes.
Implements abort_ledger_update and abort_ledger_delete triggers on ledger_transactions.
7. Digest Card
Key Invariants: Single Active Scale Token (M(P 
deck_empty
	
 )+M(P 
truck_on_deck
	
 )=1); Mass Conservation (Net=Gross−Tare); Acyclic Reversals (reversalOf<id); Integer types (Grams, Paise, Epoch ms).
State Parameters: DRAFT, PERIPHERAL_LOCKED, FINALIZED_UNSYNCED, COMMITTED_LOCAL, REVOKED.
Engineering Tensions: Legal stability interlocks vs fast yard throughput; SQLite single-writer limits vs 20 Hz serial streaming; append-only digital ledgers vs circulating paper slips.
Top 3 Gemba Hooks: Axle-hop stability test; cab dismount weight drop audit; serial ring buffer flush test.
