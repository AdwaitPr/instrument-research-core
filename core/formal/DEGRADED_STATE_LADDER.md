Graceful Degradation, Combinatorial Fault Lattices & Degraded-Mode Topologies
Path: core/formal/DEGRADED_STATE_LADDER.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, OIML R 76-1, The Legal Metrology Act 2009, Android USB Host API, Android Camera2 API
1. Peripheral × Failure-Mode Taxonomy
Peripheral Subsystem	Physical / Bus Interface	Fault Mode Class (Avizienis 2004)	Physical Mechanism & Root Cause	Systemic Consequence	Detection Trigger & API Signature
Weighing Indicator	RS-232 via USB-OTG	Physical Disconnect	Cable yank, vehicle rollover, strain on USB port.	Serial telemetry pipeline freezes.	UsbManager.ACTION_USB_DEVICE_DETACHED; usbSerialPort.read() throws IOException.
Weighing Indicator	RS-232 via USB-OTG	Buffer Overflow / Framing Error	Baud rate mismatch, missing ground pin.	Corrupted ASCII frames; bit shifting.	Delimiter missing (\r\n), length 

=18 bytes, or checksum mismatch.
Weighing Indicator	RS-232 via USB-OTG	Kernel TTY Deadlock	Linux kernel USB-serial driver hang.	Port connected, but zero bytes emitted.	Heartbeat watchdog: no valid frame for t>1500 ms.
Thermal Printer	Bluetooth SPP (ESC/POS)	Physical Disconnect / RF Outage	Out of 10m range; 2.4 GHz industrial RF interference.	Physical custody ticket cannot print.	BluetoothSocket.getOutputStream().write() throws IOException.
Thermal Printer	Bluetooth SPP (ESC/POS)	Paper-Out / Mechanical Jam	Paper roll exhausted; cutter jam.	Data accepted in buffer, not printed.	Real-time status query command (DLE EOT 1 / GS a) reports paper end flag.
Camera Module	Integrated MIPI-CSI	Hardware Bus Lock / Device Error	Simultaneous access; power rail brownout.	Optical evidence photo unavailable.	CameraDevice.StateCallback.onError() with ERROR_CAMERA_DEVICE.
Camera Module	Integrated MIPI-CSI	Thermal Throttling Cutoff	Direct sunlight chassis temperature >48 
∘
 C.	OS forcibly halts cameraserver.	PowerManager.OnThermalStatusChangedListener reports THERMAL_STATUS_SEVERE.
Power Subsystem	Li-Ion Battery + DC Float	Abrupt Brownout / Power Cutoff	Vibration disconnect; 12V DC surge.	Immediate system shutdown.	Intent.ACTION_BATTERY_LOW; battery voltage <3.4V.
2. The Combinatorial Degraded State Lattice
2.1 Mathematical Formulation of the Degradation Poset
Let the non-power operational peripherals be P 
dev
	
 ={I,P,C} where I = Indicator, P = Printer, C = Camera.
The operational state space is the Boolean lattice:
L=(P(P 
dev
	
 ),⊆)
Consisting of 2 
3
 =8 distinct reachable operational nodes ordered by capability inclusion.
2.2 Operational Node Definitions & Transaction Integrity Levels (TIL)
Node S 
7
	
 ={I,P,C} (TIL-3: Full Hardware Anchor): Serial stream + Camera photo + Thermal slip. Full commercial trade certification enabled.
Node S 
6
	
 ={I,C} (TIL-2: Secondary Cryptographic Anchor): Serial stream + Camera photo; Printer dead. Digital custody slip with SHA-256 hash dispatched via WhatsApp/SMS/QR.
Node S 
5
	
 ={I,P} (TIL-2b: Blind Verified Weighment): Serial stream + Thermal slip; Camera dead. Legally valid metrological record; photo omitted.
Node S 
4
	
 ={I} (TIL-2c: Minimal Serial Operation): Serial stream only. Valid weight stored locally; printing deferred.
Node S 
3
	
 ={P,C} (TIL-1: Optical Emergency Fallback): Serial dead; Camera + Printer operational. Metrologically unverified; dial photo captured; supervisor PIN required; slip watermarked DEGRADED_OPTICAL_UNVERIFIED.
Node S 
2
	
 ={C} (TIL-1b: Digital Optical Only): Serial and printer dead; Camera operational. Emergency visual evidence only.
Node S 
1
	
 ={P} (TIL-0: Unanchored Printer Operation): STRICTLY PROHIBITED. Pure manual entry. Weighment commits blocked by schema.
Node S 
0
	
 =∅ (TIL-0: Total Outage): All peripherals dead. Terminal enters read-only diagnostic lockdown.
3. Statutory & Legal Metrology Red-Flags
Section 24 (Verification and Stamping): Unapproved COTS cameras, OCR, or manual numeric touchscreen entries are not stamped measuring instruments under Indian law.
Section 30 & Section 33 Liabilities: Issuing commercial invoices derived from TIL-1 (Optical) or TIL-0 (Manual) weighment exposes the operator and business to criminal penalties and fines up to ₹10,000 under Section 30.
Statutory Quarantine Rule: TIL-1 transactions are restricted to Internal Physical Gate-Passes only. Commercial trade billing is strictly blocked in software (is_commercial_settlement_blocked = 1) until verified.
4. Deterministic Recovery & Buffer Reconciliation
Reconnection State Machine: Detach → Claim Interface → tcflush(fd, TCIOFLUSH) / purgeHwBuffers(true, true) → Validate k≥3 consecutive stable frames with identical mass → Restore NORMAL_STREAMING.
Two-Pass Disconnection Handling: If the scale disconnects after Pass 1 (Gross) but before Pass 2 (Tare), the system refuses to complete Pass 2 at TIL-3. It either waits for hardware reconnection or requires supervisor-authorized TIL-1, tagging the composite record as COMPOSITE_DEGRADED.
5. Room Database Schema Extension
SQL
CREATE TABLE IF NOT EXISTS degraded_mode_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    session_uuid TEXT NOT NULL,
    fault_timestamp_epoch_ms INTEGER NOT NULL,
    recovery_timestamp_epoch_ms INTEGER,
    failed_peripheral TEXT NOT NULL,
    prior_lattice_node TEXT NOT NULL,
    degraded_lattice_node TEXT NOT NULL,
    assigned_til_level TEXT NOT NULL,
    fault_signature TEXT NOT NULL,
    supervisor_pin_hash TEXT,
    optical_evidence_blob_id TEXT,
    device_temperature_celsius INTEGER,
    is_active INTEGER NOT NULL DEFAULT 1
);

CREATE TRIGGER IF NOT EXISTS abort_unanchored_til0_insert
BEFORE INSERT ON ledger_transactions
FOR EACH ROW WHEN NEW.integrity_level = 'TIL_0'
BEGIN
    SELECT RAISE(ABORT, 'METROLOGICAL VIOLATION: TIL-0 unanchored manual entries are strictly prohibited for trade records under Legal Metrology Act 2009.');
END;

CREATE TRIGGER IF NOT EXISTS validate_til1_requirements
BEFORE INSERT ON ledger_transactions
FOR EACH ROW WHEN NEW.integrity_level = 'TIL_1' AND (NEW.degraded_event_id IS NULL)
BEGIN
    SELECT RAISE(ABORT, 'INTEGRITY VIOLATION: TIL-1 optical transactions must link to a valid degraded_mode_event with supervisor authorization.');
END;
6. Field Verification Hooks
USB Hot-Yank Under Load: Sever USB-OTG connection during 20,000 kg live reading; verify UI transitions to Node S 
3
	
  within 200ms and disables TIL-3 finalization.
Kernel TTY Ring-Buffer Flushing: Reconnect serial link under a different test load; verify application discards stale frames and locks only the new reading.
Printer Paper Depletion Mid-Commit: Deplete paper roll during commit; verify transaction persists locally at TIL-2 and emits digital QR code.
Thermal Throttling Resilience: Expose terminal to >45 
∘
 C; verify camera shuts down gracefully while serial weight capture continues uninterrupted (Node S 
5
	
 ).
SQL Trigger Injection Attack: Attempt direct SQLite insert of a TIL_0 record; verify engine aborts insertion with METROLOGICAL VIOLATION.
7. Digest Card
Key Invariants: Boolean Poset Lattice (2 
3
 =8 states); Zero Unanchored Data (TIL-0 blocked by SQL triggers); Kernel TTY buffers purged on reconnect (k≥3 stable frames); Pure integer types.
Transaction Integrity Levels: TIL-3 (Full Hardware Anchor), TIL-2 (Secondary Cryptographic Anchor), TIL-1 (Optical Gate-Pass Only; Trade Billing Blocked), TIL-0 (Illegal Manual Entry; Blocked).
Legal Red-Lines: Billing on TIL-1 violates Section 30 of The Legal Metrology Act, 2009.
Top 3 Gemba Hooks: USB hot-yank reaction time; TTY buffer discard audit; SQLite trigger injection test.
