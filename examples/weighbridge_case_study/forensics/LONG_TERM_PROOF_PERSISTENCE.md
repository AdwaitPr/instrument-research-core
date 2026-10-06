# Long-Term Forensic Proof Persistence & Decentralized Cold-Storage Anchors
Path: core/forensics/LONG_TERM_PROOF_PERSISTENCE.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/substrate/MEDIA_PERSISTENCE_SPEC.md, core/arbitration/DISPUTE_ARBITRATION_LIFECYCLE.md, Information Technology Act 2000 (Section 7), CGST Act 2017 (Section 36), RFC 4998 (ERS)

## 1. Statutory Retention Horizons & Physical Media Decay Dynamics

In commercial freight and agricultural procurement, metrological records are not ephemeral operational logs; they are legal instruments subject to multi-year statutory retention obligations:

```text
                  ┌────────────────────────────────────────┐
                  │    TRANSACTION SETTLED & CERTIFIED     │
                  └───────────────────┬────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
│ CGST ACT 2017    │        │ INCOME TAX ACT   │        │ LIMITATION ACT   │
│ SECTION 36       │        │ SECTION 44AA     │        │ 1963 (SCHEDULE)  │
├──────────────────┤        ├──────────────────┤        ├──────────────────┤
│ Statutory Term:  │        │ Statutory Term:  │        │ Statutory Term:  │
│ 72 Months        │        │ 6 to 8 Years     │        │ 3 to 12 Years    │
│ (6 Years from due│        │ (From end of     │        │ (Commercial      │
│  date of annual  │        │  relevant Assess-│        │  contracts, tort,│
│  return filing)  │        │  ment Year)      │        │  land/asset title│
└────────┬─────────┘        └────────┬─────────┘        └────────┬─────────┘
         │                           │                           │
         └───────────────────────────┼───────────────────────────┘
                                     │
                                     ▼
                        ┌──────────────────────────┐
                        │ MANDATORY RETENTION:     │
                        │ 8-YEAR IMMUTABLE HORIZON │
                        │ Zero Loss, Bit-Rot Proof │
                        └──────────────────────────┘
```

### 1.1 The Physical Physics of Long-Term Media Decay (Bit-Rot)

Operating in unconditioned rural scale cabins (where temperatures regularly exceed 45°C for months) subjects storage media to rapid physical degradation:

- **NAND Flash Charge Leakage (Arrhenius Acceleration):** Floating-gate and 3D TLC/QLC charge-trap NAND cells leak electrons over time. At an elevated ambient storage temperature of 55°C, unpowered NAND data retention degrades exponentially:

$$\text{Retention Life} \propto e^{\frac{E_a}{k_B T}}$$

  A consumer SSD or SD card rated for 10 years at 25°C can experience silent bit-flips and read disturb errors within 12 to 18 months when stored unpowered in a tin-shed weighbridge cabin.
- **Magnetic Media Demagnetization:** Industrial dust particles (>10 μm grain dust and silica) breach standard drive breathers, causing head crashes and oxide coating shedding on magnetic hard drives.
- **Cryptographic Algorithm Obsolescence:** Cryptographic primitives deemed secure at transaction time (e.g., SHA-1 in 2005, or standard SHA-256 facing post-quantum horizons) degrade over decades, requiring verifiable rolling timestamp notary chains.

---

## 2. Hierarchical Merkle Epoch Trees & Dynamic Ledger Pruning

Persisting raw 20 Hz high-frequency telemetry streams (Aspect 04) for every vehicle across an 8-year lifecycle would exhaust local storage capacity. The system enforces a Hierarchical Merkle Pruning Topology:

```text
[LEVEL 0: RAW CONTINUOUS TELEMETRY] (20 Hz ADC Jerk Samples, Optical Blobs)
                 │
                 ├── Active Hot Retention: 30 Days Local SSD
                 ├── Pruned after 30 Days (Discarded once signed hash committed)
                 ▼
[LEVEL 1: TRANSACTION LEAF] (Metrological Certified State Vector)
                 │ Leaf Hash: H_tx = SHA-256(Ticket || Gross || Tare || Time || Signatures)
                 ▼
[LEVEL 2: DAILY MERKLE BATCH] (Compiled at Shift Closing / Midnight)
                 │ Tree Root: R_day = MerkleRoot(H_tx,1, H_tx,2, ..., H_tx,N)
                 ▼
[LEVEL 3: MONTHLY REGULATORY EPOCH] (28 to 31 Daily Roots)
                 │ Super Root: R_month = MerkleRoot(R_day,1, ..., R_day,M)
                 ▼
┌────────────────────────────────────────────────────────┐
│ ANNUAL STATUTORY ARCHIVE BUNDLE:                       │
│ • Root Hash: R_year = MerkleRoot(R_month,1 ... 12)     │
│ • Total Storage Overhead: < 50 MB / year               │
│ • Forensic Power: Any individual ticket proves         │
│   inclusion via a logarithmic Merkle Proof (O(log N))  │
└────────────────────────────────────────────────────────┘
```

### 2.1 The Merkle Inclusion Proof Invariant

Even when detailed ADC telemetry and optical images are pruned after 30 days, the certified weight ticket remains mathematically verifiable forever. A claimant presents the raw ticket data and an authentication path:

$$\text{Path} = \{H_1, H_2, \dots, H_k\}$$

$$\text{Verify}(R_{\text{year}}, \text{TicketData}, \text{Path}) = \text{TRUE}$$

The receipt's presence within the historical annual root $R_{\text{year}}$ provides irrefutable mathematical proof of existence prior to epoch sealing under Section 63 of the Bharatiya Sakshya Adhiniyam, 2023.

---

## 3. Cryptographic Long-Term Timestamping & Hash Agility (RFC 4998)

To guard against cryptographic obsolescence over 8-year periods, the archive implements Evidence Record Syntax (RFC 4998) and timestamp notarization.

### 3.1 Rolling Archive Timestamp Chains

When a cryptographic hash function approaches the end of its security margin, the archive is re-encapsulated without modifying the underlying historical records:

```text
[YEAR 0]: Target Data Bundle ──► SHA-256 Hash ──► RFC 3161 Timestamp Token T_1
                                                         │
                                    (After 5 Years)      ▼
[YEAR 5]: [Bundle || T_1]    ──► SHA-384 Hash ──► RFC 3161 Timestamp Token T_2
                                                         │
                                    (After 10 Years)     ▼
[YEAR 10]: [Bundle || T_1 || T_2] ──► Post-Quantum Hash ──► Quantum-Safe Token T_3
```

Because each renewal covers the preceding timestamp token while the previous cryptographic primitive remains intact, the unbroken evidentiary integrity chain extends indefinitely.

### 3.2 Decentralized Public Calendar Anchoring

To prevent yard owners from covertly backdating records or rewriting local database history, monthly epoch roots ($R_{\text{month}}$) are anchored into public decentralized calendars (OpenTimestamps protocol anchored to public ledger block headers).

- **Zero Ongoing Cost:** Anchors are submitted in batches via cryptographic aggregation.
- **Air-Gapped Operation:** The terminal does not execute transactions on an active blockchain; it generates an offline Merkle tree and exports the root token via sneakernet sync (Aspect 05) to be committed whenever upstream connectivity permits.

---

## 4. Dual-Media WORM Cold Storage & Air-Gapped Vault Archiving

For long-term physical persistence, the software mandates periodic migration to physical Write-Once-Read-Many (WORM) media:

```text
┌─────────────────────────────────────────────────────────────────┐
│              DUAL-MEDIA ARCHIVAL STORAGE TIERS                  │
├─────────────────────┬───────────────────────────────────────────┤
│ STORAGE TIER        │ PHYSICAL MEDIUM & ENVIRONMENTAL LIFESPAN  │
├─────────────────────┼───────────────────────────────────────────┤
│ Tier 1: Local Hot   │ Internal Industrial eMMC / NVMe SSD       │
│                     │ (30-day live operational rolling window)  │
├─────────────────────┼───────────────────────────────────────────┤
│ Tier 2: Cold Vault  │ High-Endurance Industrial MicroSD (pSLC)  │
│                     │ Dedicated read-only WORM hardware mode    │
│                     │ (Annual offline rotation; 5-year cycle)   │
├─────────────────────┼───────────────────────────────────────────┤
│ Tier 3: Permanent   │ Archival Optical M-DISC Blu-ray (BD-R)    │
│ Statutory Archive   │ Inorganic glassy carbon pit layer;        │
│                     │ Immune to magnetic, thermal, and UV decay │
│                     │ (1000-year theoretical archival rating)   │
└─────────────────────┴───────────────────────────────────────────┘
```

### 4.1 Air-Gapped Archival Envelope Schema (.WBARCHIVE)

The exported archive bundle is structured as an uncompressed, standardized POSIX tar archive with embedded forward error correction:

- `METROLOGICAL_REGISTER.CDB`: Fast-indexed constant database containing all finalized weighment state vectors.
- `MERKLE_PROOFS.TREE`: Binary representation of the hierarchical Merkle tree structure.
- `SIGNATURE_CHAIN.PEM`: Complete chain of device certificates and TSA notary tokens.
- `PAR2_REED_SOLOMON.PAR2`: 10% Reed-Solomon parity recovery files, guaranteeing complete data reconstruction even if physical bit-rot corrupts up to 10% of physical storage sectors.
---

## 5. Room Database Schema Extensions

To track Merkle epoch trees, rolling archive notary timestamps, and periodic storage media integrity scrubs, the database schema extends as follows:

```sql
-- Architectural Extension: Tracking Sealing Merkle Epoch Roots
CREATE TABLE IF NOT EXISTS forensic_merkle_epochs (
    epoch_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    epoch_level TEXT CHECK(epoch_level IN ('DAILY_BATCH', 'MONTHLY_EPOCH', 'ANNUAL_SEAL')) NOT NULL,
    period_start_epoch_ms INTEGER NOT NULL,
    period_end_epoch_ms INTEGER NOT NULL,
    leaf_transaction_count INTEGER NOT NULL,
    merkle_root_sha256_hex TEXT NOT NULL,
    rfc3161_timestamp_token_blob TEXT,
    opentimestamps_ots_proof_blob TEXT,
    is_anchored_public_calendar INTEGER NOT NULL DEFAULT 0,
    sealed_at_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_epoch_period 
    ON forensic_merkle_epochs(period_start_epoch_ms, period_end_epoch_ms);

-- Architectural Extension: Tracking Physical Media Integrity Scrubs (Bit-Rot Audits)
CREATE TABLE IF NOT EXISTS cold_storage_integrity_scrubs (
    scrub_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    media_volume_identifier TEXT NOT NULL,          -- e.g. 'WORM_SD_2026_A', 'MDISC_OPTICAL_01'
    media_tier TEXT CHECK(media_tier IN ('LOCAL_HOT_SSD', 'WORM_PSLC_SD', 'MDISC_BLURAY')) NOT NULL,
    total_bytes_scrubbed INTEGER NOT NULL,
    bit_rot_sectors_detected INTEGER NOT NULL DEFAULT 0,
    reed_solomon_reconstructed INTEGER NOT NULL DEFAULT 0,
    scrub_outcome TEXT CHECK(scrub_outcome IN ('PRISTINE', 'REPAIRED_VIA_PAR2', 'UNRECOVERABLE_CORRUPTION')) NOT NULL,
    audited_by_operator_id TEXT NOT NULL,
    timestamp_epoch_ms INTEGER NOT NULL
);

-- Trigger: Enforce absolute immutability on sealed Merkle epochs
CREATE TRIGGER IF NOT EXISTS abort_merkle_epoch_tamper
BEFORE UPDATE ON forensic_merkle_epochs
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Sealed Merkle roots and notary proofs are immutable legal records.');
END;

-- Trigger: Prohibit manual deletion of physical media scrub logs
CREATE TRIGGER IF NOT EXISTS abort_scrub_audit_deletion
BEFORE DELETE ON cold_storage_integrity_scrubs
BEGIN
    SELECT RAISE(FAIL, 'SECURITY AUDIT: Storage integrity logs cannot be deleted from the database.');
END;
```

---

## 6. Field Verification Hooks (Gemba Protocols)

1. **Logarithmic Merkle Inclusion Path Assay:**
   - *Procedure:* Generate a synthetic month of 2,000 transactions. Seal the monthly epoch root $R_{\text{month}}$. Extract an arbitrary transaction leaf $H_{\text{tx},1421}$ and its accompanying Merkle proof path.
   - *Pass/Fail Criteria:* Verify that an independent verifier script proves receipt inclusion using exactly $\lceil \log_2(2000) \rceil = 11$ hash operations without requiring access to the other 1,999 transactions.

2. **Accelerated High-Temperature NAND Bit-Rot Scrub:**
   - *Procedure:* Flash a sealed `.WBARCHIVE` package onto a commercial SD card. Subject the card to an environmental thermal bake ($60^\circ\text{C}$ for 72 hours). Re-insert into the terminal and run a bit-rot integrity scrub.
   - *Pass/Fail Criteria:* Verify that the storage engine detects any sector bit-flips, verifies whether Reed-Solomon PAR2 reconstruction succeeds, and logs the outcome in `cold_storage_integrity_scrubs`.

3. **RFC 4998 Rolling Timestamp Hash Agility Drill:**
   - *Procedure:* Take an archive bundle timestamped with SHA-256. Simulate hash depreciation. Execute an archive renewal pass using SHA-384 and a simulated secondary RFC 3161 TSA token.
   - *Pass/Fail Criteria:* Confirm that the outer envelope validates the new SHA-384 proof chain while preserving the original historical SHA-256 inner certificate unbroken.

4. **Offline OpenTimestamps Calendar Sneakernet Assay:**
   - *Procedure:* Generate an offline Merkle epoch root on an air-gapped terminal. Export the `.ots` commitment stub to a USB drive. Transfer to an internet-connected gateway and anchor to a public block header. Return the finalized Bitcoin/Litecoin block receipt back to the air-gapped terminal.
   - *Pass/Fail Criteria:* Verify that the terminal verifies the Merkle proof back to the public block header without ever opening an active network socket.

5. **WORM Hardware Write-Protection Lockout Test:**
   - *Procedure:* Insert an industrial pSLC MicroSD card configured with hardware WORM partition locks. Attempt to execute an SQL `UPDATE`, `DELETE`, or POSIX file rewrite command.
   - *Pass/Fail Criteria:* Ensure the physical card firmware rejects the write command with an I/O error (`EPERM` / `EROFS`) and triggers no local filesystem corruption.

---

## 7. Repo Impact Analysis

- **Updates to `MASTER_CORE_PROTOCOL.md`:**
  * Establishes the **8-Year Retention Invariant**: certified metrological records must remain verifiable across the full statutory limitation horizon mandated by Section 44AA of the Income Tax Act and Section 36 of the CGST Act.
  * Formally mandates the **Hierarchical Merkle Pruning Architecture**: raw high-frequency telemetry may be discarded after 30 days provided that the transaction leaf is anchored into daily, monthly, and annual Merkle roots.
  * Codifies **Hash Agility (RFC 4998)**: long-term forensic persistence requires recursive timestamp renewal chains to withstand cryptographic decay over decades.
- **Impact on Room Database Schemas:**
  * Adds `forensic_merkle_epochs` and `cold_storage_integrity_scrubs` tables.
  * Implements `abort_merkle_epoch_tamper` and `abort_scrub_audit_deletion` triggers.

---

## 8. Digest Card

- **Key Invariants:** 8-Year Statutory Horizon (CGST S. 36 / IT Act S. 44AA); Hierarchical Merkle Pruning (Daily $\to$ Monthly $\to$ Annual Roots); Logarithmic Verification ($O(\log N)$); Hash Agility (RFC 4998 rolling renewal); Dual-Media WORM / M-DISC Archiving.
- **Physical Media Hostility:** High-temperature ($> 45^\circ\text{C}$) Arrhenius charge leakage in NAND flash; abrasive dust intrusion; magnetic head crashes; long-term cryptographic obsolescence.
- **Archival Package Architecture:** `.WBARCHIVE` bundle containing `METROLOGICAL_REGISTER.CDB`, `MERKLE_PROOFS.TREE`, `SIGNATURE_CHAIN.PEM`, and 10% Reed-Solomon PAR2 parity blocks.
- **Top 3 Gemba Hooks:**
  1. Verify logarithmic Merkle inclusion proof for a historical ticket using $\le 12$ hashes.
  2. Test Reed-Solomon PAR2 reconstruction of bit-rotted sector data after thermal stress.
  3. Validate hardware WORM card write-rejection during attempted file deletion.
