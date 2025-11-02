# BISSO STEP5 - FUNCTION BLOCK GUIDE
## Detailed Analysis of FB10-FB18 & PB Programs

---

## FUNCTION BLOCKS OVERVIEW

Your project uses **two types of code blocks**:

### **FB (Function Blocks)** - Subroutines with local data
- **FB10** - Manual axis speed control
- **FB11** - Forward/Reverse motor direction logic
- **FB12** - Ramp speed calculator
- **FB13** - Motion detector
- **FB14** - Rich potentiometer (speed reference from DW)
- **FB15** - Fast Command mode processor (411 lines - largest block!)
- **FB16** - Startup initialization
- **FB17** - Parameter block
- **FB18** - Table control (FB 18 is called at end of OB1)
- **FB240-FB251** - VFD/Inverter communication blocks

### **PB (Program Blocks)** - Main algorithm blocks
- **PB9** - Setup block
- **PB10** - Passage program sequences (59 lines)
- **PB11** - Manual override handler (67 lines)
- **PB20** - Translation program (112 lines - complex!)
- **PB21-PB51** - Sub-programs for specific sequences

---

## FUNCTION BLOCKS DETAILED

### **FB10 - Manual Axis Speed Control**

**Purpose:** When operator manually jogs an axis, FB10 controls the speed

**Called From:** OB1 Section [7]

**Logic:**
```
IF Manual Mode (F 0.0 = 1) THEN
  Read I 2.4/2.5 (Left/Right buttons)
  Read I 3.0/3.1 (Up/Down buttons)
  Read I 3.2/3.3 (CW/CCW buttons)
  
  IF Speed+ (F 3.5) pressed THEN
    Increment DW speed counter
    Load DW 0-17 (select speed table)
  END
  
  IF Speed- (F 3.6) pressed THEN
    Decrement DW speed counter
  END
  
  Output selected speed to Q8.x (VFD commands)
END
```

**Key Output:** Speed value to VFD (Q 8.x outputs)

---

### **FB11 - Forward/Reverse Direction Logic**

**Purpose:** Determines if motor should go forward or reverse

**Called From:** OB1 Section [7]

**Logic:**
```
IF Inverter sequence active (F 1.4 OR F 1.3) THEN
  IF Forward condition THEN
    Set F 4.0/F 4.2 (forward phase flags)
    Output Q 8.0 (Inv1 Forward)
    Output Q 8.4 (Inv2 Forward)
  END
  
  IF Reverse condition THEN
    Set F 4.1/F 4.3 (reverse phase flags)
    Output Q 8.1 (Inv1 Reverse)
    Output Q 8.5 (Inv2 Reverse)
  END
END
```

**Key Logic:** Never output both forward AND reverse simultaneously - that would be an ERROR

---

### **FB12 - Ramp Speed Calculator**

**Purpose:** Smooth acceleration/deceleration curves

**Called From:** OB1 when speed reference changes

**Algorithm:**
```
Current_Speed = DW 0 (load current value)
Target_Speed = Elbo I 73.7 input (velocity command)

IF Current_Speed < Target_Speed THEN
  Current_Speed += Ramp_Increment
  (accelerate slowly)
ELSE IF Current_Speed > Target_Speed THEN
  Current_Speed -= Ramp_Increment
  (decelerate slowly)
END

Output Current_Speed to Q 8.x
```

**Result:** Motors don't suddenly jump to full speed - they ramp smoothly over time

---

### **FB13 - Motion Detector**

**Purpose:** Detect if motion is currently happening

**Called From:** OB1 Section [5]

**Logic:**
```
IF any Q 6.x contactor active THEN
  Set F 5.0 = 1 (Motion Active)
ELSE
  Set F 5.0 = 0 (No Motion)
END

Output F 5.0 status to diagnostic display
```

**Used By:** 
- Manual mode cannot be selected if F 5.0 = 1
- Program mode cannot start if F 5.0 = 1
- Table position mode cannot be selected if F 5.0 = 1

---

### **FB14 - Speed Reference (Potentiometer/Rich)**

**Purpose:** Read current speed setting and prepare for output to VFD

**Called From:** OB1 Section [10]

**Logic:**
```
Read DW 1 (Current Speed Data Word)
IF Speed+ button (I 4.5) pressed THEN
  Increment value
  Limit to max (65535)
ELSE IF Speed- button (I 4.6) pressed THEN
  Decrement value
  Limit to min (0)
END

Store in DW 1
Output to QW 66 (VFD speed reference)
```

**Output:** QW 66 = 16-bit word sent to VFD for disk RPM control

---

### **FB15 - Fast Command Mode Processor (411 lines!)**

**Purpose:** Special "help" feature for quick access to functions

**Called From:** OB1 Section [1]

**What is Fast Command Mode?**
- Press **I 3.7 (FastCmd)** button **3 times quickly**
- Activates special fast mode menu
- Provides quick access to presets

**Complex Logic Inside FB15:**
```
State Machine with 8 possible states (F 7.0-F 7.5):
- F 7.0: Fast horizontal movement
- F 7.1: Fast vertical movement
- F 7.2: Fast table rotation
- F 7.3: Fast tilt
- F 7.4: Fast speed increment
- F 7.5: Fast speed decrement

Pressing FastCmd cycles through states
Each state performs specific action
Feature timeout: Auto-deactivates after 2s inactivity
```

**Why So Complex?**
- Allows operator to quickly switch between common operations
- Don't need to manually select axis each time
- Preset motion sequences

---

### **FB16 - Startup Initialization**

**Purpose:** System bootup sequence

**Called From:** OB1 on first scan

**Logic:**
```
Initialize all flags to 0
Set initial timer values
Load default speed values from DW
Perform safety check:
  - Verify all contactors de-energized
  - Verify end switches accessible
  - Verify Elbo communication
System ready flag = 1
```

---

### **FB17 - Parameter Block**

**Purpose:** Store configuration constants

**Contains:**
```
Safe voltage levels
Motor current limits
Timer defaults
Speed ramping curves
Safety margins
```

---

### **FB18 - Table Control**

**Purpose:** Manage hydraulic table raise/lower operations

**Called From:** OB1 Section [6], line 358: JU FB 18

**Logic:**
```
IF Table1 Up button (I 9.1) pressed THEN
  Energize Q 7.3 (Oil Pump)
  Energize Q 72.5 (Direction = Up)
  Set F 10.1 (Table 1 in up position)
  Start T 26 (3 minute timer - prevent overheat)
END

IF Table1 Down button (I 9.2) pressed THEN
  Energize Q 7.3 (Oil Pump)
  De-energize Q 72.5 (Direction = Down)
  Set F 10.2 (Table 1 in down position)
END

;; SAFETY
IF T 26 expires (3 minutes) THEN
  De-energize Q 7.3 (stop pump)
  Prevent further table movement
  Wait for manual reset
END
```

**Critical Safety:** Table can only move for **180 seconds (3 minutes)** before automatic shutoff

---

### **FB240-FB251 - VFD Communication Blocks**

**Purpose:** Send commands to variable frequency drives (inverters)

**FB240:**
```
;; Inverter initialization
A   I 4.0    ;; IF Disk ON
S   Q 7.0    ;; SET Main contactor
L   DW 1     ;; Load speed
T   QW 68    ;; Transfer to Inv1 command
```

**FB241-FB242:** Similar for Inverter 2 and redundant controls

**FB243:** Fault detection - if inverter not responding

**FB250-FB251:** Advanced VFD features (not widely used)

---

## PROGRAM BLOCKS (PB) DETAILED

### **PB9 - Setup**

```
Initialize:
Set all output banks
Load default data values
Verify communications
Test all end switches
Report ready status
```

---

### **PB10 - Passage Program (59 lines)**

**Purpose:** Execute pre-programmed cutting sequence with multiple passes

**Called From:** OB1 Section [8] when F 5.1 = 1 (sequence started)

**Typical Sequence:**
```
1. IF First pass (C 1 = 0) THEN
     Move material to start position
     Lower blade to cut
     Move material forward (F to S)
     Raise blade
2. MOVE TO NEXT PASSAGE POSITION
   Increment passage counter (C 2)
3. REPEAT until passage counter = 0
4. Return material to home position
5. Program complete
```

**Key Variables:**
- **C 2** = Passage counter (tracks how many cuts done)
- **F 5.1** = Program sequence started
- **F 5.4** = Sequence controller enabled

---

### **PB11 - Manual Override Handler (67 lines)**

**Purpose:** Allow operator to interrupt programmed sequence with manual control

**Called From:** OB1 Section [8] when F 5.2 = 1 (manual override)

**Logic:**
```
IF Manual mode selected (F 0.0 = 1) WHILE program running THEN
  Pause programmed sequence
  Allow manual jog with I 2.4-7 (buttons)
  Save current position
ELSE
  Resume programmed sequence
  Continue from saved position
END
```

---

### **PB20 - Translation Program (112 lines - MOST COMPLEX!)**

**Purpose:** Execute cutting with automatic horizontal repositioning

**Program Pattern:**
```
PASS 1: Cut at position 0
        - Lower blade
        - Move material forward (cut)
        - Move back to start
        - Raise blade

MOVE TO NEXT POSITION: Material moves right by step distance
        - Move right (I 2.4 logic)
        - Confirm position sensor
        - Wait for settle

PASS 2-N: Repeat cutting at each position

This allows cutting parallel lines across stone width
```

**Complex Parts:**
1. **Position tracking** - Remember where material is
2. **Incremental movement** - Calculate step distance between cuts
3. **Sensor verification** - Confirm material at correct position before cutting
4. **Error recovery** - What if material doesn't reach expected position?

---

### **PB21-PB33 - Specific Sequence Programs**

Each implements a different cutting pattern:
- **PB21:** Border cut
- **PB22:** Cross pattern
- **PB23:** Parallel lines (horizontal)
- **PB24:** Parallel lines (vertical)
- **PB25:** Grid pattern
- **PB26:** Edge chamfer
- **PB27:** Corner cut
- **PB28:** Radius cut
- **PB29:** Special pattern 1
- **PB30:** Special pattern 2
- **PB31:** Specialty
- **PB32:** Reserved
- **PB33:** Custom user program

---

### **PB40-PB51 - Advanced Programs**

For specialized operations or future expansion

---

## DATA BLOCKS

### **DB10 - Speed Table**

```
DW 0:   Speed preset 1 (low speed)
DW 1:   Speed preset 2
...
DW 16:  Speed preset 16 (high speed)
DW 17:  Current working speed

Each DW stores a 16-bit value:
  0 = 0 RPM
  32768 = 50% speed
  65535 = 100% speed
```

---

## COMPLETE CALL HIERARCHY

```
OB1 (Main Loop, runs every 100ms)
├── FB15 (Fast Command - if F 6.3 = 1)
├── FB18 (Table Control)
├── PB10 (Passage Program - if F 5.1 = 1)
│   ├── FB10 (Speed control)
│   ├── FB11 (Direction control)
│   ├── FB12 (Ramp calculator)
│   └── FB13 (Motion detector)
├── PB11 (Manual Override - if F 5.2 = 1)
├── PB20 (Translation Program - if F 0.1 = 1)
│   ├── PB21-PB33 (Sub-sequences)
│   └── (Multiple function block calls)
├── FB14 (Speed reference)
├── FB240-FB243 (VFD commands)
└── (Many conditional jumps)
```

---

## FLAG STATE MACHINE DIAGRAM

```
STARTUP
  ↓
FB16 Initialize
  ↓
[F 0.0 OR F 0.1 OR F 0.2 OR F 0.3] = 1
  ↓
┌─────────────────────────────────────┐
│ WAIT FOR NO MOTION (F 5.0 = 0)      │
└─────────────────────────────────────┘
  ↓
OPERATOR SELECTS MODE
  ├─→ I 2.0 → F 0.0 = 1 (Manual Mode)
  │   ├─→ I 2.4-7 (Jog buttons)
  │   └─→ FB10/11/12 handle motion
  │
  ├─→ I 2.1 → F 0.1 = 1 (Translation Program)
  │   └─→ PB20 executes
  │
  ├─→ I 2.2 → F 0.2 = 1 (Passages + Translation)
  │   └─→ PB20 with passages
  │
  └─→ I 2.3 → F 0.3 = 1 (Passages)
      └─→ PB10 executes

DISK CONTROL (Independent)
  ├─→ I 4.0 (Disk ON) → F 1.0, F 1.1
  ├─→ I 4.1 (Disk OFF) → Reset F 1.0, F 1.1
  └─→ I 4.2 (Star-Delta/VFD selector)

TABLE CONTROL (Via FB18)
  ├─→ I 9.1 (Table Up) → F 10.1
  ├─→ I 9.2 (Table Down) → F 10.2
  ├─→ I 9.3 (Table2 Up) → F 10.3
  └─→ I 9.4 (Table2 Down) → F 10.4

SPEED CONTROL
  ├─→ I 4.5 (+Speed) → F 3.5 → FB14 increment
  └─→ I 4.6 (-Speed) → F 3.6 → FB14 decrement
      └─→ Output to QW 66 (VFD speed)

FAST MODE (If F 6.3 = 1)
  └─→ FB15 processor handles special sequences
```

---

## KEY ALGORITHMS

### **Speed Ramping Algorithm (FB12)**

```
CURRENT_SPEED ← Load from DW 1
TARGET_SPEED ← Read from I 73.7 (Elbo velocity)

RAMP_RATE = 100 (increase per scan)

IF CURRENT_SPEED < TARGET_SPEED THEN
  CURRENT_SPEED = CURRENT_SPEED + RAMP_RATE
  IF CURRENT_SPEED > TARGET_SPEED THEN
    CURRENT_SPEED = TARGET_SPEED
  END
END

IF CURRENT_SPEED > TARGET_SPEED THEN
  CURRENT_SPEED = CURRENT_SPEED - RAMP_RATE
  IF CURRENT_SPEED < TARGET_SPEED THEN
    CURRENT_SPEED = TARGET_SPEED
  END
END

STORE CURRENT_SPEED in DW 1
OUTPUT to QW 66 (VFD command)
```

**Result:** Smooth 0-100% acceleration over ~10 seconds

---

### **Motion Sequence Algorithm (PB10)**

```
PASSAGE_COUNT = 0
MAX_PASSAGES = Read from data input

LOOP:
  IF PASSAGE_COUNT < MAX_PASSAGES THEN
    ;; LOWER BLADE
    Set Q 6.2 = Down
    Wait for position sensor
    
    ;; FEED MATERIAL
    Set Q 6.0 = Forward
    Wait for end position
    
    ;; RAISE BLADE
    Set Q 6.2 = Up
    Wait for neutral
    
    ;; MOVE TO NEXT PASSAGE
    Set Q 6.1 = Right
    Move by STEP_DISTANCE
    Wait for position sensor
    
    INCREMENT PASSAGE_COUNT
  ELSE
    ;; SEQUENCE COMPLETE
    Return material to home
    Sound finished signal
    EXIT LOOP
  END
END
```

---

## TIMING DIAGRAM EXAMPLE

**Manual Single Cut (Timeline):**

```
T=0ms:    Operator presses I 4.0 (Disk ON)
          F 1.0 = 1 (Disk running flag)
          F 1.1 = 1 (Startup sequence)
          T 21 starts (20 second ramp)

T=80ms:   T 2 expires (Star-Delta timing)
          Q 6.6 (Star) ← 0
          Q 6.7 (Delta) ← 1
          Motor reaches full speed

T=1000ms: T 21 expires (20 second wait)
          Disk at operating speed
          Q 8.6 = 1 (Spindle command)
          F 0.7 = 1 (Speed reference active)

T=1100ms: Operator selects Vertical axis (I 73.0 = 1)
          Operator selects Down direction (I 73.6 = 1)
          Operator provides velocity (I 73.7 = 1)
          
T=1150ms: OB1 detects motion request
          Routes to Q 6.2 (Down contactor)
          Q 6.2 = 1
          Blade starts moving down

T=2050ms: Blade reaches forward limit (I 5.2 = 1)
          OB1 detects: Q 6.0 AND I 5.2
          Sets Q 73.2 = 1 (signal to Elbo)
          Elbo sees limit and stops velocity input
          I 73.7 = 0

T=2100ms: I 73.7 = 0 for 50ms (T 11 expires)
          OB1 forces F 1.6 = 1 (stop motion)
          RESETS Q 6.0, Q 6.1, Q 6.2, Q 6.3, Q 6.4, Q 6.5
          All motors stop within 15ms

T=2115ms: F 5.0 = 0 (no motion active)
          System ready for next command
```

---

## COMMON ISSUES & SOLUTIONS

**"System won't respond to manual commands"**
→ Check: Is F 5.0 = 1? (Motion active flag stuck)
→ Check: Is I 4.4 = 1? (Power button pressed)
→ Check: Is I 72.7 = 1? (Disk running - may prevent manual mode)

**"Programs run too fast/slow"**
→ Check: FB12 ramp rate in code
→ Check: Timer values (T 7, T 9, T 10) might need adjustment
→ Check: DW 1 speed value range

**"Blade won't stop at limit switch"**
→ Check: I 5.x inputs are working (end switch connection)
→ Check: F 5.0 motion active logic
→ Check: Q 73.2 output to Elbo is set

**"Fast Mode not activating"**
→ Check: Did you press I 3.7 exactly 3 times?
→ Check: F 6.3 flag state
→ Check: FB15 is being called from OB1

---

## QUICK REFERENCE: WHICH FLAG/TIMER FOR WHAT?

| Need | Use This |
|------|----------|
| Manual mode on/off | F 0.0 |
| Program running | F 5.1 |
| Any motion | F 5.0 |
| Disk motor running | F 1.0 |
| Speed ramping | FB12 + T 3, T 18, T 19 |
| Emergency stop | F 1.6 → Reset Q 6.0-6.5 |
| Table safety | T 26 (3 minute limit) |
| Fast mode | F 6.3 + FB15 |
| Laser safety | T 14 (30s) + T 15 (5m) |
| Sequence timeout | T 1 (10 minutes) |

