# Master Core Metrological Protocol & System Invariant Matrix
Path: core/MASTER_CORE_PROTOCOL.md
Status: ACTIVE / RATIFIED
Cross-References: Aspects 01 through 19

## 1. System Constitution: The 10 Invariant Axioms

1. **Metrological Ground Truth Invariant (Aspect 01, 16):** Live indicator mass readings are valid if and only if certified within Class III MPE limits ($e=10\text{ kg}$). Any unverified or extrapolated weight is rejected.
2. **Kinematic Dwell Invariant (Aspect 04):** A weight lock ($T_4$) is invalid if triggered in $<1200\text{ ms}$ after vehicle arrival. Signals must be stabilized through a 2nd-order Butterworth low-pass filter ($f_c=1.2\text{ Hz}$) with $\sigma \le 0.5e$.
3. **Queue Collapse Bound (Aspect 03):** Hot-path transaction processing latency must never exceed $\tau_{\text{app}} \le 15.0\text{ s}$ ($6.0\text{ s}$ for cached fast-path fleets) to prevent arrival queue runaway ($\rho < 1.0$).
4. **Zero-Cloud Autonomy Invariant (Aspect 05, 11, 13):** All state evaluations, optical ANPR inferences, vehicle authorizations, and cash settlements operate entirely offline without internet dependencies.
5. **Statutory Cash Compliance Invariant (Aspect 08):** Payouts $>₹10,000$ in cash are hard-blocked for commercial non-exempt goods under Section 40A(3); agricultural exemptions require Rule 6DD(e) identity logging.
6. **Statutory Privacy Firewall (Aspect 09):** In regulatory inspection mode, legal metrology data is visible, while commercial pricing margins, trader fees, and cash drawers are masked.
7. **Tri-Identifier Fleet Correlation (Aspect 11):** High-trust automated clearance requires cross-validation across physical HSRP registration plates, 96-bit FASTag EPC, and 64-bit silicon TID memory banks.
8. **Thermodynamic Loss Baseline (Aspect 12):** Cross-mandi freight shortages must calculate evaporative moisture shrinkage ($\Delta M_{\text{theo}}$) before attributing loss to carrier theft under Section 10 of the Carriage by Road Act.
9. **Optical-Kinematic Axle Interlock (Aspect 13):** Physical barrier gates open if and only if optical curtain axle counts match dynamic strain-gauge shock pulses ($N_{\text{optical}} = N_{\text{jerk}}$).
10. **8-Year Forensic Persistence (Aspect 15, 18):** Certified records are immutably preserved for 8 years using hierarchical Merkle epoch trees, RFC 4998 hash renewal, and Reed-Solomon PAR2 error correction on WORM media.

## 2. Unified Database Entities & Cross-Aspect Integration

The complete schema connects across all operational aspects:
- `ledger_transactions` (Aspect 01, 02)
- `airgap_sync_envelopes` (Aspect 05)
- `cash_drawers` & `cash_journal_entries` (Aspect 08)
- `statutory_inspection_audits` (Aspect 09)
- `loadcell_shock_events` (Aspect 04)
- `fleet_registered_vehicles` (Aspect 11)
- `inter_facility_discrepancy_cases` (Aspect 12)
- `anpr_optical_captures` & `gate_interlock_events` (Aspect 13)
- `forensic_merkle_epochs` & `cold_storage_integrity_scrubs` (Aspect 15)
