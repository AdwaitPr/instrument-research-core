# Audio-Haptic Feedback, Visual Semiotics & Glancing UX in Hostile Yard Environments
Path: core/human_factors/AUDIO_HAPTIC_GLANCING_UX.md
Status: DRAFT / DESK-VERIFIED
Dependencies: core/formal/PETRI_STATE_MATRIX.md, core/formal/DEGRADED_STATE_LADDER.md, ISO 9241-307, ISO 7000, WCAG 2.1

## 1. Physical Environment Sensory Hostility Profile

Weighbridge terminals operate in extreme sensory environments where standard consumer UI assumptions fail completely.

### 1.1 The Operational Sensory Envelope

```text
+-----------------------------------------------------------------------------------+
|                        THE SCALE CABIN SENSORY PROFILE                            |
+-----------------------------------------------------------------------------------+
| [ACOUSTIC HOSTILITY]                                                              |
| • Diesel Truck Idling:        82 – 88 dBA (Dominant low frequency: 80 – 350 Hz)    |
| • Air Brake Pressure Release: 95 – 102 dBA (High-frequency transient hiss)        |
| • Pneumatic Truck Horn:       105 – 115 dBA (Puncturing alert masking)            |
| • Baseline Ambient Noise:     78 – 85 dBA (Continuous yard floor)                |
+-----------------------------------------------------------------------------------+
| [OPTICAL & SOLAR HOSTILITY]                                                       |
| • Direct Midday Sun:          80,000 – 120,000 lux (Open approach ramp)           |
| • Glare Through Cabin Window: 25,000 – 45,000 lux (Specular glass reflection)     |
| • Standard Office Interior:   300 – 500 lux (Consumer UI baseline)               |
| • Resulting Screen Washout:   Contrast drops from 1000:1 to < 2.5:1 on consumer UI|
+-----------------------------------------------------------------------------------+
| [OPERATIONAL COGNITIVE POSTURE]                                                   |
| • Glancing Interaction:       85% of time eyes look out window; 15% look at screen|
| • Viewing Distance:           1.5 to 2.5 meters (Mounted swivel arm)              |
| • Hand State:                 Dusty, sweaty, coarse, occasional industrial gloves |
+-----------------------------------------------------------------------------------+
```
2. Visual Glancing Architecture: Typography, Contrast & Semiotics
2.1 The 2-Meter Glancing Formula
To enable an operator to read the active live weight while looking out the window at the vehicle's axle alignment, visual elements must subtend a minimum visual angle of θ≥1.5 
∘
 :
h 
char
	
 =2×d×tan( 
2
θ
	
 )
For viewing distance d=2000mm and θ=1.5 
∘
 :
h 
char
	
 =2×2000×tan(0.75 
∘
 )=52.36mm
On a standard 10.1-inch 1920×1200 tablet (224PPI), 52.36mm corresponds to a physical height of 460 screen pixels (≥72 sp display font).
2.2 Tactical High-Contrast Palette (WCAG 2.1 AAA Compliant)
Under 40,000lux ambient glare, standard Android material palettes wash out. The interface enforces an Avionics Cockpit Inverse Palette:
Plaintext
┌─────────────────────────────────────────────────────────────────┐
│ BACKGROUND: Pure Deep Black (#000000) - Eliminates OLED Backlight│
├───────────────────┬──────────────────────┬──────────────────────┤
│ STATE             │ FOREGROUND COLOR     │ HEX CODE             │
├───────────────────┼──────────────────────┼──────────────────────┤
│ Live Mass Stream  │ High-Luminance Amber │ #FFB300 (Aviation)   │
│ Weight Locked     │ High-Luminance Green │ #00E676 (Electric)   │
│ Motion / Unstable │ Saturated Yellow     │ #FFEA00 (Safety)     │
│ System Alarm      │ Saturated Crimson    │ #FF1744 (Emergency)  │
│ Static Captions   │ Stark White          │ #FFFFFF (High White) │
└───────────────────┴──────────────────────┴──────────────────────┘
2.3 Indigenous Semiotics & Redundant Shape Anchoring
To eliminate ambiguity for low-literacy operators and prevent color-blindness misinterpretations, states are never communicated by color alone:
Plaintext
+---------------------------------------------------------------------+
| ACTIVE STATE       | GEOMETRIC SHAPE | ICON ANCHOR | DISPLAY TEXT   |
+--------------------+-----------------+-------------+----------------+
| Scale Empty (0 kg) | [ ] Hollow Box  | Flat Ramp   | ZERO (शून्य)   |
| Live Motion        | ╱╲ Zig-Zag Wave | Rolling Axle| MOTION (चालू)  |
| Stable Mass Locked | █ Solid Block   | Padlock     | LOCKED (पक्का) |
| Hardware Tamper    | ▲ Warning Tri   | Flashing Ex | TAMPER (खतरा)  |
+--------------------+-----------------+-------------+----------------+
3. Acoustic Earcon Architecture: Piercing Diesel Masking
Because voice prompts (TTS) require 3–5 seconds and get masked by diesel engine rumble, the system communicates exclusively via psychoacoustic synthetic earcons (<250ms).
3.1 Auditory Masking Avoidance
Diesel Rumble Band (80–500Hz): Low-frequency industrial noise.
Earcon Resonant Band (1.8 kHz to 3.2 kHz): The human ear's maximum sensitivity notch (outer ear canal resonance). Bypasses engine noise without requiring extreme volume.
3.2 The Four Canonical Earcons
Event Name	Frequency Progression (f)	Total Duration (Δt)	Waveform	Perceptual Meaning
EARCON_TICK	Single 2000Hz pulse	15ms	Square wave (Soft)	Tactile keypress feedback.
EARCON_STABLE_LOCK	Dual harmonic: 1760Hz→2637Hz	180ms	Pure Sinusoid	"Ding!" Pitch rises: Success, weight locked.
EARCON_WARNING	Alternating: 880Hz↔440Hz	300ms	Sawtooth	Buzzing drop: Motion rejected, deck unsettled.
EARCON_ALARM	Triple dissonant: 3136Hz+3322Hz	600ms	Frequency Churn	Harsh dissonance: Hardware tamper or duress alert.
4. Haptic Feedback Waveform Specifications
For operators tapping the tablet while looking out the window, Android VibrationEffect waveforms provide deterministic tactile confirmation:
Kotlin
// Tactical Android Haptic Implementation (AOSP Vibrator API)

// 1. Keypress Touch Tick (Subtle confirmation for numeric entry)
val HAPTIC_TOUCH_TICK = VibrationEffect.createPredefined(VibrationEffect.EFFECT_CLICK)

// 2. Weight Lock Latch (Crisp mechanical snap when transition T4 fires)
val HAPTIC_LATCH_SNAP = VibrationEffect.createWaveform(
    longArrayOf(0, 40, 30, 80),      // Timing: pause, pulse 1, pause, pulse 2
    intArrayOf(0, 180, 0, 255),      // Amplitude: rising crisp tactile snap
    -1                               // No repeat
)

// 3. Tamper / Interlock Rejection (Harsh double-stutter alert)
val HAPTIC_REJECT_STUTTER = VibrationEffect.createWaveform(
    longArrayOf(0, 100, 50, 100, 50, 150),
    intArrayOf(0, 255, 0, 255, 0, 255),
    -1
)
5. Room Database Schema Extensions
To analyze operator reaction times, ambient brightness toggles, and sensory accessibility states, the database schema records UX interaction telemetry:
SQL
-- Architectural Extension: Tracking Ergonomic Performance & Glancing Latencies
CREATE TABLE IF NOT EXISTS ui_interaction_telemetry (
    telemetry_id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    session_uuid TEXT NOT NULL,
    screen_name TEXT NOT NULL,                  -- 'GROSS_CAPTURE', 'TARE_CAPTURE'
    ambient_lux_level INTEGER NOT NULL,         -- Read from device light sensor
    high_contrast_mode_active INTEGER NOT NULL DEFAULT 1,
    audio_alert_volume_percent INTEGER NOT NULL,
    glancing_reaction_time_ms INTEGER NOT NULL, -- Time from STABLE state to T4 Lock tap
    mistap_adjacent_count INTEGER NOT NULL DEFAULT 0,
    timestamp_epoch_ms INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_ui_telemetry_time 
    ON ui_interaction_telemetry(timestamp_epoch_ms);

-- Maintain operator accessibility preferences across shift handovers
CREATE TABLE IF NOT EXISTS operator_ergonomic_profiles (
    operator_id TEXT PRIMARY KEY NOT NULL,
    preferred_haptic_strength INTEGER NOT NULL DEFAULT 255, -- 0 to 255
    audio_feedback_enabled INTEGER NOT NULL DEFAULT 1,
    vernacular_semiotic_level TEXT CHECK(vernacular_semiotic_level IN (
        'ICON_ONLY', 
        'ICON_AND_REGIONAL_TEXT', 
        'STANDARD_METROLOGICAL'
    )) NOT NULL DEFAULT 'ICON_AND_REGIONAL_TEXT',
    last_calibrated_epoch_ms INTEGER NOT NULL
);
6. Field Verification Hooks (Gemba Protocols)
Ambient Sound Level (SPL) Meter Logging:
Procedure: Mount a calibrated sound level meter next to the tablet inside the scale cabin. Measure sound pressure level during peak tractor traffic (12:00–14:00 IST).
Pass/Fail Criteria: Verify that the 2.5kHz EARCON_STABLE_LOCK chime is clearly audible above the 85–95dBA diesel idling noise without distortion.
Solar Irradiance Glare Washout Assay:
Procedure: Position the mobile tablet under open sunlight (>80,000lux). View the screen from a distance of 2.0meters at a 45 
∘
  glancing angle.
Pass/Fail Criteria: Verify that active live mass digits (72 sp high-contrast amber #FFB300) remain readable in <200ms without squinting or shielding with hands.
The Blindfold Tactile Weight Lock Drill:
Procedure: Have an operator wear a blindfold. Feed a live serial stream. Trigger the stable lock button.
Pass/Fail Criteria: Verify that the operator can confirm weight lock successfully solely through the HAPTIC_LATCH_SNAP waveform and EARCON_STABLE_LOCK chime.
Gloves-On Mistap Assay:
Procedure: Have an operator wear standard industrial leather/rubber work gloves. Execute 20 consecutive weighment transactions under queue pressure.
Pass/Fail Criteria: Ensure mistap rate on adjacent buttons is <1% across all trials due to the ≥64 dp target size.
Semiotic Symbol Comprehension Test:
Procedure: Present 5 semi-literate agricultural drivers with: (A) Pure English text slips, (B) Pure Devanagari text slips, and (C) Semiotic silhouette cards (loaded vs empty tractor icons).
Pass/Fail Criteria: Measure time to identify Gross vs Tare. The semiotic silhouette card must reduce comprehension latency by ≥60%.
7. Repo Impact Analysis
Updates to MASTER_CORE_PROTOCOL.md:
Establishes the Glancing Interaction Invariant: core weighment interfaces must be fully operable from a 2.0–meter distance with eyes-out-the-window cognitive posture.
Formally bans TTS Voice Synthesis on Hot Paths: latency ceilings (τ 
app
	
 ≤15 s) mandate sub-250ms synthetic earcons over spoken audio.
Enforces Redundant Shape-Color Semiotics: color coding without accompanying geometric icons is prohibited under WCAG 2.1 AAA rules.
Impact on Jetpack Compose UI Layer:
Mandates pure black backgrounds (#000000) and high-luminance amber/green typography in WeighbridgeTheme.
Enforces minimum touch target constraints (≥64×64 dp) on all transaction-critical buttons.
8. Digest Card
Key Invariants: Glancing Posture (Eyes looking out the window 85% of shift); Visual Angle Rule (θ≥1.5 
∘
 →Digit Height ≥52mm); Psychoacoustic Masking Bypass (1.8–3.2kHz earcons pierce diesel rumble); Avionics High-Contrast Palette (#000000 + #FFB300); Multi-Modal Redundancy (Color + Shape + Icon + Sound + Vibration).
Physical Envelope: Ambient noise 85–102dBA; Sunlight 80,000–120,000lux; Viewing distance 1.5–2.5meters.
Core Ergonomics: Sub-250ms synthetic earcons; 64×64dp glove-friendly touch targets; 3-tier Android vibration waveforms.
Top 3 Gemba Hooks:
Measure cabin ambient noise vs 2.5kHz earcon audibility.
Verify 2-meter readability under 80,000lux sunlight.
Validate blindfold weight-lock confirmation via audio-haptic feedback.
