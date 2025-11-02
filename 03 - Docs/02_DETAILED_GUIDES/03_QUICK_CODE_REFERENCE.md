# BISSO STEP5 - QUICK CODE REFERENCE
## Fast Lookup Guide for Programmers

---

## ALL FLAGS AT A GLANCE

### Program Mode (Mutually Exclusive)
- **F 0.0** = Manual Mode (individual axis jogging)
- **F 0.1** = Translation Program (horizontal movement sequences)
- **F 0.2** = Passages with Translation
- **F 0.3** = Passages (fixed position multi-cuts)
- **F 0.4** = Tilt mode (blade angle instead of vertical)

### Motion Status
- **F 5.0** = ANY motion active (Q 6.0-6.5 energized)
- **F 5.1** = Program sequence started
- **F 5.2** = Manual override during sequence
- **F 5.3** = In programmed mode
- **F 5.4** = Sequence controller active

### Disk Motor
- **F 1.0** = Disk motor running
- **F 1.1** = Disk startup sequence (initialization)
- **F 0.7** = Speed reference active

### Inverters
- **F 1.3** = Inverter 2 sequence
- **F 1.4** = Inverter 1 sequence
- **F 1.5** = Inverter 2 active
- **F 1.6** = Stop all motion (emergency)
- **F 1.7** = Motion cutoff condition

### Speed Control
- **F 2.0** = Speed adjust in passages
- **F 2.1** = Speed adjust in tilt
- **F 2.2** = Speed adjust general
- **F 2.3** = Speed+ button
- **F 2.4** = Speed- button

### Mode/Position
- **F 3.0** = Tilt mode flag
- **F 3.1** = Vertical mode flag
- **F 3.5** = Speed up pressed
- **F 3.6** = Speed down pressed
- **F 3.7** = Laser on

### Phase Control
- **F 4.0** = Phase 1 active
- **F 4.1** = Phase 2 active
- **F 4.2** = Phase 1 complete
- **F 4.3** = Phase 2 complete
- **F 4.5** = Speed enabled

### Fast Mode
- **F 6.3** = Fast mode active
- **F 7.0-7.5** = Fast mode options
- **F 7.6** = Fast mode toggle
- **F 7.7** = Fast mode active indicator

### Table Position
- **F 10.1** = Table 1 up
- **F 10.2** = Table 1 down
- **F 10.3** = Table 2 up
- **F 10.4** = Table 2 down

---

## ALL TIMERS AT A GLANCE

| Timer | Duration | Purpose |
|-------|----------|---------|
| T 0 | 60s | Disk startup cooldown |
| T 1 | 600s | Sequence timeout (10m) |
| T 2 | 80ms | Star-Delta timing |
| T 3 | 400ms | VFD ramp |
| T 4 | 30s | Power timeout |
| T 5 | 50ms | Inverter 1 pulse |
| T 6 | 50ms | Inverter 2 pulse |
| T 7 | 50ms | Motion pulse |
| T 8 | 20ms | Table lock debounce |
| T 9 | 10ms | Phase 1 |
| T 10 | 10ms | Phase 2 |
| T 11 | 15ms | Stop motion delay |
| T 13 | 5ms | Mode transition |
| T 14 | 30s | Laser safety |
| T 15 | 300s | Laser hard limit (5m) |
| T 17 | 2s | Fast button debounce |
| T 18 | 30s | Startup ramp |
| T 19 | Var | Special timer |
| T 20 | 20s | VFD stabilize |
| T 21 | 20s | Disk startup |
| T 22 | 50ms | Phase timing |
| T 24 | 20ms | Motion detector |
| T 25 | 180s | Hydraulics limit (3m) |
| T 26 | 120s | Pump safety |

---

## CRITICAL INPUT SIGNALS

### Manual Control Buttons
- **I 2.0** = Program Manual
- **I 2.1** = Program Translation
- **I 2.2** = Program Passages Translation
- **I 2.3** = Program Passages
- **I 2.4** = Manual Right
- **I 2.5** = Manual Left
- **I 2.6** = Manual Front
- **I 2.7** = Manual Back
- **I 3.0** = Manual Down
- **I 3.1** = Manual Up
- **I 3.2** = Manual Table CW
- **I 3.3** = Manual Table CCW
- **I 4.0** = Disk ON
- **I 4.1** = Disk OFF
- **I 4.2** = Star-Delta/VFD selector
- **I 4.3** = Tilt/Vertical selector
- **I 4.4** = Power ON/OFF (CRITICAL)
- **I 4.5** = Speed + button
- **I 4.6** = Speed - button
- **I 4.7** = Laser ON/OFF

### End Switches (Safety)
- **I 5.0** = Right limit
- **I 5.1** = Left limit
- **I 5.2** = Forward limit
- **I 5.3** = Backward limit
- **I 5.4** = Down limit
- **I 5.5** = Up limit
- **I 5.6** = Tilt down limit
- **I 5.7** = Tilt up limit

### Table Control
- **I 9.0** = Table 2 end switch
- **I 9.1** = Table 1 up button
- **I 9.2** = Table 1 down button
- **I 9.3** = Table 2 up button
- **I 9.4** = Table 2 down button

### Elbo Microcontroller
- **I 72.0** = Elbo Rapido (fast)
- **I 72.1** = Elbo Medio (medium)
- **I 72.2** = Cmd Rotation
- **I 72.3** = Cmd LockTable (CRITICAL)
- **I 72.4** = Pressostato
- **I 72.5** = VFD1 running
- **I 72.6** = VFD2 running
- **I 72.7** = Disk running (CRITICAL)
- **I 73.0** = Axis Cut (Z)
- **I 73.1** = Axis Translation (Y)
- **I 73.2** = Axis Tilt
- **I 73.3** = Axis Table Rotation
- **I 73.4** = Axis Disk Rotation
- **I 73.5** = Direction + (forward/up/right)
- **I 73.6** = Direction - (backward/down/left)
- **I 73.7** = Velocity Enable (CRITICAL)

---

## CRITICAL OUTPUT SIGNALS

### Motor Contactors
- **Q 6.0** = Forward/Reverse
- **Q 6.1** = Left/Right
- **Q 6.2** = Up/Down
- **Q 6.3** = Tilt
- **Q 6.4** = Table Rotation
- **Q 6.5** = Disk Rotation

### Special Relays
- **Q 7.0** = Disk line contactor (RL2)
- **Q 7.1** = VFD selector (changes mode)
- **Q 7.2** = Laser
- **Q 7.3** = Oil pump (CRITICAL - controls table)
- **Q 7.4** = Power relay
- **Q 7.5** = Motion pulse
- **Q 7.6-7.7** = Reserved

### Inverter Commands
- **Q 8.0** = Inverter 1 Forward
- **Q 8.1** = Inverter 1 Reverse
- **Q 8.2** = Inverter 1 speed control
- **Q 8.3** = Inverter 2 speed control
- **Q 8.4** = Inverter 2 Forward
- **Q 8.5** = Inverter 2 Reverse
- **Q 8.6** = Spindle motor
- **Q 8.7** = Table lock (CRITICAL)

### Status Lights
- **Q 72.0** = Program active
- **Q 72.1** = Disk running
- **Q 72.2** = End switch hit
- **Q 72.4-72.7** = Direction relays

### Elbo Outputs
- **Q 73.0** = Mode/Program status
- **Q 73.1** = Disk control status
- **Q 73.2** = End switch/limit signal (to Elbo)
- **Q 73.3-73.7** = Special signals

---

## OB1 SECTIONS SUMMARY

| Section | Lines | Purpose |
|---------|-------|---------|
| [1] | 1-47 | Program mode selection (F 0.0-0.4) |
| [2] | 166-231 | Disk motor control & VFD sequencing |
| [3] | 233-243 | Programmed motion timer (T 1) |
| [4] | 244-262 | Speed reference signal |
| [5] | 264-318 | Limit switch safety & motion detection |
| [6] | 319-360 | Main power & table movement |
| [7] | 361-468 | Inverter motion control (F 1.3-1.7) |
| [8] | 470-532 | Table lock & program sequencing (CRITICAL) |
| [9] | 533-600 | Motion command generation |
| [10] | 601-659 | Speed control (DW 1 → QW 66) |

---

## FUNCTION BLOCKS SUMMARY

| Block | Lines | Purpose | Called By |
|-------|-------|---------|-----------|
| **FB10** | 72 | Manual axis speed | OB1 |
| **FB11** | 83 | Direction control | OB1 |
| **FB12** | 59 | Ramp speed calc | OB1 |
| **FB13** | 39 | Motion detector | OB1 |
| **FB14** | 48 | Speed reference | OB1 |
| **FB15** | 411 | Fast mode (LARGEST!) | OB1 |
| **FB16** | 15 | Startup init | OB1 |
| **FB17** | 11 | Parameters | OB1 |
| **FB18** | 66 | Table control | OB1 |
| **FB240-251** | 50 | VFD communication | OB1 |

---

## PROGRAM BLOCKS SUMMARY

| Block | Lines | Purpose |
|-------|-------|---------|
| **PB9** | 11 | Setup |
| **PB10** | 59 | Passages sequence |
| **PB11** | 67 | Manual override |
| **PB20** | 112 | Translation (COMPLEX) |
| **PB21-33** | 400+ | Specific patterns |
| **PB40-51** | 200+ | Advanced sequences |

---

## HOW TO FIND WHAT YOU NEED

### "How do I make manual mode work?"
1. Set **F 0.0 = 1** (manual mode flag)
2. Check **F 5.0 = 0** (no motion active)
3. Read jog buttons: **I 2.4-7, I 3.0-3** 
4. Output to contactors: **Q 6.0-6.5**
5. See: **OB1 Section [1]** and **OB1 Section [7]**

### "How do I add a new programmed sequence?"
1. Create new **PB (Program Block)**
2. Set a new flag like **F 0.5** to trigger it
3. Add condition in **OB1 Section [1]**: `A I 2.X S F 0.5`
4. Add jump in **OB1 Section [8]**: `A F 0.5 JC PB 99`
5. Implement motion logic in your new PB99

### "How do I change speed ramping?"
1. Edit **FB12** ramp rate value
2. Change timer values: **T 3, T 18, T 19**
3. Or modify **DW speed table** (DB10)
4. Test output on **QW 66** (VFD command)

### "How do I add safety condition?"
1. Add new condition check in **OB1 Section [7]**
2. Use **AN** (AND NOT) for safety conditions
3. Force reset: `R Q 6.0 R Q 6.1 ...` etc
4. Set safety flag: `S F 1.6` (stop motion)

### "Why isn't manual mode activating?"
```
Check in order:
1. Is I 2.0 pressed? (button working?)
2. Is F 6.3 = 0? (not in fast mode?)
3. Are all jog buttons (I 2.4-7, 3.0-5) = 0?
4. Is F 5.0 = 0? (no motion active?)
5. Is F 0.0 actually being SET by logic?
6. Check OB1 Section [1] lines 5-18
```

### "Why is motion not stopping?"
```
Check in order:
1. Is I 4.4 = 1? (power pressed?)
2. Is end switch activated? (I 5.x = 1?)
3. Is Q 73.2 being output? (signal to Elbo?)
4. Is F 1.6 being set? (emergency stop?)
5. Check OB1 Section [7] lines 432-468
```

---

## DATA WORDS

| DW | Purpose | Range | Default |
|----|---------|-------|---------|
| **DW 0** | Speed 1 | 0-65535 | 8192 |
| **DW 1** | Current speed | 0-65535 | 16384 |
| **DW 2-17** | Speed presets | 0-65535 | Varies |
| **DW 20-44** | FB output data | Various | Auto |

**QW 66** = Output speed to VFD (sent from DW 1)

---

## TYPICAL OPERATION FLOW

```
SYSTEM START:
  ├─ T 0 = 1 (initialization)
  ├─ All flags = 0
  └─ Ready for operator input

OPERATOR SELECTS MANUAL MODE:
  ├─ Press I 2.0 (Manual button)
  ├─ F 0.0 = 1 (set flag)
  ├─ Can now jog with I 2.4-7

OPERATOR STARTS DISK:
  ├─ Press I 4.0 (Disk ON)
  ├─ F 1.0 = 1, F 1.1 = 1
  ├─ T 21 counts (20s startup)
  ├─ Disk reaches speed

OPERATOR CONTROLS MOTION:
  ├─ Use Elbo joystick OR I 2.4-7 buttons
  ├─ Selects axis (I 73.0-4 or I 3.0-3)
  ├─ Selects direction (I 73.5-6 or I 2.X)
  ├─ OB1 energizes Q 6.x
  └─ Motor runs

OPERATOR RELEASES CONTROL:
  ├─ No direction input (I 73.5-6 = 0 or I 2.X = 0)
  ├─ I 73.7 = 0 (velocity signal lost)
  ├─ OB1 resets Q 6.x
  └─ Motor stops
```

---

## EMERGENCY PROCEDURES

### Force Stop All Motion:
```
Set I 4.4 = 0 (Power OFF button)
→ Kills Q 7.4 power relay
→ Kills all inverters
→ All motors stop
```

### Reset After Crash:
```
1. Press I 4.4 twice (Power OFF then ON)
2. Wait 10 seconds (T 0 timeout)
3. All flags reset to 0
4. Ready to start again
```

### Manual Sequence Reset:
```
If program stuck:
1. Press I 2.0 (Manual mode)
   → Overrides programmed mode
2. Manually move material to safety
3. Press I 4.1 (Disk OFF)
4. Power cycle if needed
```

---

## DEBUGGING TIPS

1. **Check flags first:**
   - Use PLC monitor to see flag values
   - One-shot flags go true/false quickly
   - Look for flags stuck at 1 (stuck condition)

2. **Trace signal flow:**
   - Input: I x.y → Checked in OB1
   - Logic: F x.y → Set/Reset conditions
   - Output: Q x.y → Energizes/De-energizes

3. **Check timers:**
   - Timers running too short: motion cuts off early
   - Timers running too long: system unresponsive
   - Check KT values in code (e.g., KT 020.1 = 20s)

4. **Watch for mutual exclusions:**
   - Only ONE of F 0.0-0.3 can be 1
   - Only ONE of F 3.0-3.1 can be 1
   - Never both F 1.3 and F 1.4

5. **Verify motor control:**
   - Q 6.x should match current motion command
   - Check end switches (I 5.x) aren't false-triggered
   - Verify Q 73.2 output to Elbo when limit hit

---

## AWL INSTRUCTION QUICK REFERENCE

```
A     = AND
AN    = AND NOT
O     = OR
ON    = OR NOT
S     = SET (to 1)
R     = RESET (to 0)
=     = ASSIGN (output)
JC    = Jump Conditional (if accumulator=1)
JU    = Jump Unconditional
BE    = Block End
BEC   = Block End Conditional
L     = LOAD
T     = TRANSFER
SD    = Start Timer Delayed
SF    = Start Timer Flag
SE    = Start Timer Edge
SP    = Start Timer Pulse
CD    = Count Down
C()   = Comment
```

---

## QUICK CODE TEMPLATES

### Add New Manual Jog Button:
```awl
;; IF new button pressed AND motion allowed
A   I x.y           ;; New input
AN  F 5.0           ;; Not moving
AN  F 6.3           ;; Not in fast mode
AN  I 4.4           ;; Power on
S   Q z.w           ;; Set output contactor
```

### Add Speed Ramp:
```awl
;; Load and increment speed
A   F speed_up_flag
L   DW 1            ;; Load current speed
L   KH 100          ;; Add increment
T   DW 1            ;; Store new value
L   DW 1
T   QW 66           ;; Output to VFD
```

### Add Safety Timeout:
```awl
;; Start timer when motion
O   Q 6.0
O   Q 6.1
O   Q 6.2
L   KT 180.3        ;; Load timeout value
SE  T 25            ;; Start edge-triggered

;; Reset all if timeout
A   T 25            ;; Timer expired?
R   Q 6.0           ;; Force reset
R   Q 6.1
R   Q 6.2
```

