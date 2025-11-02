# BISSO STEP5 PLC - COMPLETE ANNOTATED CODE GUIDE
## Understanding the Full Program Structure (2200+ lines of AWL code)

---

## TABLE OF CONTENTS

1. **Program Overview & Architecture**
2. **Step5/AWL Basics** (for reference)
3. **Main Program Flow (OB1)**
4. **Flag Definitions & Usage**
5. **All Blocks Explained**
6. **Control Sequences**
7. **Flag Cross-Reference**

---

## 1. PROGRAM OVERVIEW & ARCHITECTURE

### Total Code: 2,200+ lines of AWL (Step5 Assembly Language)

### Block Structure:
- **OB1** (659 lines) - Main Organization Block (main loop)
- **FB15** (411 lines) - Fast Command/Help feature processor
- **PB20** (112 lines) - Translation program control
- **PB10** (59 lines) - Passage program control (multi-cut sequences)
- **FB10-FB18** (600+ lines) - Various function blocks
- **PB9, PB11, PB30-51** (400+ lines) - Program sequences

### Three Main Operation Modes:
1. **Manual Mode (F 0.0)** - Individual axis jog control
2. **Programmed Mode (F 0.1-0.3)** - Automatic sequences
3. **Fast Command Mode (F 6.3)** - Special high-speed control

---

## 2. STEP5/AWL BASICS (Quick Reference)

### AWL Instruction Summary:
```
A     = AND (accumulator AND with operand)
AN    = AND NOT (accumulator AND with NOT operand)
O     = OR (accumulator OR with operand)
ON    = OR NOT (accumulator OR with NOT operand)
S     = SET (set bit/flag to 1)
R     = RESET (reset bit/flag to 0)
=     = ASSIGN (load accumulator value to output)
JC    = Jump Conditional (jump if accumulator = 1)
L     = LOAD (load value)
T     = TRANSFER (transfer to register)
SD    = Start Timer Delayed (start timer for duration)
SF    = Start Timer Edge-triggered
SE    = Start Timer Edge-triggered (single edge)
SP    = Start Timer Pulse
CD    = Count Down
C()   = Create Comment block
D()   = Create data block
BE    = Block End
BEC   = Block End with Comment
***   = Section separator
```

### Operand Types:
```
I x.y    = Input bit (x=byte, y=bit)
Q x.y    = Output bit
F x.y    = Flag (internal bit)
T x     = Timer
C x     = Counter
DW x    = Data Word (16-bit value)
KT x.y  = Time constant (x = value, y = time unit: 0.1s, 1s, 10s, etc.)
KC x    = Counter constant (preset value)
FW x    = Fetch Word (get DW value)
```

---

## 3. MAIN PROGRAM FLOW (OB1 - 659 lines)

### OB1 Structure: 10 Sections (separated by *** and [1] through [10])

### **OB1 SECTION [1]: Program Mode Selection (Lines 1-47)**

**Purpose:** Determine which operation mode is selected

```awl
C   DB 10    ;; Load Database 10 at start of each scan

;; PROGRAM MANUAL MODE SELECTION
A   I 2.0        ;; IF Manual Program button pressed
AN  F 6.3        ;; AND NOT in Fast Command mode
AN  I 2.4        ;; AND NOT Right button
AN  I 2.5        ;; AND NOT Left button
AN  I 2.6        ;; AND NOT Front button
AN  I 2.7        ;; AND NOT Back button
AN  I 3.0        ;; AND NOT Down button
AN  I 3.1        ;; AND NOT Up button
AN  I 3.2        ;; AND NOT Table CW button
AN  I 3.3        ;; AND NOT Table CCW button
AN  I 3.4        ;; AND NOT Disk Down button
AN  I 3.5        ;; AND NOT Disk Up button
AN  F 5.0        ;; AND NOT any motion active
S   F 0.0        ;; SET Manual Mode flag

;; RESET MANUAL MODE
ON  I 2.0        ;; If Manual button released
ON  I 4.4        ;; OR Power turned off
R   F 0.0        ;; RESET Manual Mode flag
```

**Flag Logic:**
- **F 0.0** = Manual Mode flag
  - When SET: Operator can manually jog individual axes
  - When RESET: Manual mode inactive
  - **Mutually exclusive** with F 0.1, F 0.2, F 0.3 (programmed modes)

**Similar logic for:**
- **F 0.1** = Translation Program mode (I 2.1 button, but NOT if disk running: I 72.7)
- **F 0.2** = Passages with Translation mode (I 2.2 button)
- **F 0.3** = Passages mode (I 2.3 button)
- **F 0.4** = Special "Tilt" mode selector (via I 73.2, used when operator selected tilt axis)

**Key Insight:** The program **cannot enter programmed mode if disk is running** (I 72.7=1). This prevents dangerous mode switches during cutting.

---

### **OB1 SECTION [2]: Disk Motor Control (Lines 166-231)**

**Purpose:** Control disk/blade motor via VFD or Star-Delta soft-start

```awl
;; SELECT VFD CONTROL
A   I 4.2        ;; IF Star-Delta/VFD selector = VFD
AN  F 1.0        ;; AND disk not already running
S   Q 7.1        ;; SET Q 7.1 = VFD Selector relay

;; WAIT FOR VFD TO STABILIZE (20s)
AN  F 1.1        ;; If no active startup sequence
L   KT 020.1     ;; Load 20 seconds (KT 020.1 = 20s)
SD  T 20         ;; Start Timer T20 delayed

;; RESET VFD SELECTOR AFTER TIMEOUT
AN  I 4.2        ;; IF Star-Delta selector = 0
AN  I 73.7       ;; AND Elbo velocity signal = 0
A   T 20         ;; AND Timer T20 expired
R   Q 7.1        ;; RESET VFD selector

;; DISK STARTUP SEQUENCE (Flag F 1.1)
A   I 4.0        ;; IF Disk ON button pressed
A   Q 7.1        ;; AND VFD selector active
AN  T 0          ;; AND T0 not running
S   F 1.1        ;; SET Disk startup sequence flag

;; STARTUP DELAY (20s to stabilize)
A   F 1.1        ;; IF startup sequence active
L   KT 020.1     ;; Load 20 seconds
SD  T 21         ;; Start Timer T21

;; SEND SPINDLE COMMAND TO OUTPUT
A   T 21         ;; IF T21 has counted down
=   Q 8.6        ;; Output spindle motor command

;; DISK OFF SEQUENCE
ON  I 4.4        ;; If Power OFF
ON  I 4.1        ;; OR Disk OFF button
O   T 1          ;; OR Long delay expired
ON  Q 7.1        ;; OR VFD selector off
R   F 1.1        ;; RESET startup sequence flag
```

**Flags Involved:**
- **F 1.0** = Main disk motor running
- **F 1.1** = Disk startup sequence (initialization phase)
- **F 1.2** = (See Section [1])

**Key Logic:**
- Disk motor can be controlled via **VFD (Variable Frequency Drive)** OR **Star-Delta soft-start**
- VFD allows smooth speed control from Elbo proportional input
- Star-Delta provides mechanical soft-start (ramps voltage from 0.58x to full)
- Timer T21 (20s) provides **stabilization delay** before disk reaches operating speed

---

### **OB1 SECTION [3]: Programmed Motion Timer (Lines 233-243)**

**Purpose:** Time how long programmed sequences take

```awl
O(  ;; START: IF in programmed mode
A   F 0.2   ;; Passages with Translation
O   F 0.3   ;; OR Passages
O   F 0.1   ;; OR Translation
)
AN  I 73.5  ;; AND NOT forward direction
AN  I 73.6  ;; AND NOT backward direction
A   I 4.4   ;; AND Power ON
L   KT 600.1    ;; Load 600 seconds (10 minutes)
SD  T 1         ;; Start Timer T1 (long timeout for sequence)
```

**Purpose of T1:**
- Prevents infinite loops if program crashes
- 10 minute maximum per programmed sequence
- If T1 times out, sequence is forcibly stopped

---

### **OB1 SECTION [4]: Speed Reference Signal (Lines 244-262)**

**Purpose:** Provide speed reference signal to VFD or inverters

```awl
;; CONDITION: If any of these conditions true:
O(
A   Q 7.1       ;; VFD selector active
A   Q 8.6       ;; AND spindle command active
A   T 3         ;; AND speed ramp timer active
A   I 73.7      ;; AND Elbo velocity signal present
)
O(
A   F 1.0       ;; OR main disk motor running
A   Q 6.7       ;; AND Delta connection active
)
=   F 0.7       ;; SET Flag 0.7 = Speed reference active

;; START SPEED RAMP TIMER (400ms)
A   Q 7.1       ;; If VFD selector active
A   Q 8.6       ;; AND spindle command active
A   I 73.7      ;; AND velocity signal present
L   KT 400.1    ;; Load 400 milliseconds
SD  T 3         ;; Start Timer T3 (ramp timer)
```

**Flag F 0.7** = Speed reference active
- Indicates to system that speed feedback is being sent
- Used in **Elbo signal communication** (I 73.7 provides speed command)

---

### **OB1 SECTION [5]: Limit Switch Safety & Motion Detection (Lines 264-318)**

**Purpose:** Monitor end switches and prevent over-travel

```awl
;; DETECT FORWARD/REVERSE MOTION WITH LIMIT SWITCH HIT
O(  ;; If Forward contactor active
A   Q 6.0       ;; Q 6.0 = Forward/Reverse contactor
A(              ;; AND
O   I 5.2   ;; Forward limit switch I5.2
O   I 5.3   ;; OR Backward limit switch I5.3
)
)
O(  ;; OR Left/Right motion with end switch
A   Q 6.1
A(
O   I 5.0   ;; Right limit
O   I 5.1   ;; Left limit
)
)
O(  ;; OR Vertical motion with end switch
A   Q 6.2
A(
O   I 5.4   ;; Down limit
O   I 5.5   ;; Up limit
)
)
O(  ;; OR Tilt motion with end switch
A   Q 6.3
A(
O   I 5.6   ;; Tilt down limit
O   I 5.7   ;; Tilt up limit
)
)
=   Q 73.2      ;; OUTPUT: Set Q 73.2 (motion with limit hit signal)
```

**This outputs to Elbo microcontroller:**
- **Q 73.2** tells Elbo "limit switch active"
- Elbo may respond by stopping that axis

**Speed Safety (CRITICAL):**
```awl
;; IF Tilt mode + Fast speed selected + Forward limit switch
A   I 72.0      ;; IF I 72.0 (Elbo Rapido/Fast)
A   F 0.4       ;; AND Tilt mode selected
L   FW 26       ;; Load special value from DW
SD  T 19        ;; Start special timer T19

;; OUTPUT POSITION FLAG
A   F 0.4       ;; If Tilt mode
A   I 72.0      ;; AND Fast speed
A   I 72.5      ;; AND VFD1 running
A   T 19        ;; AND T19 timed out
=   F 9.4       ;; SET Flag 9.4 (Position flag)
```

---

### **OB1 SECTION [6]: Main Power & Table Movement (Lines 319-360)**

**Purpose:** Control hydraulic system for table up/down

```awl
;; MAIN POWER CONTROL
O   I 2.0       ;; IF Manual mode button pressed
ON  I 2.0       ;; OR Manual mode button released
S   Q 7.4       ;; SET Main Auxiliary Power relay Q 7.4

;; POWER TIMEOUT (30s, then turn off)
A   T 1         ;; If long timeout timer expired
L   KT 030.2    ;; Load 30 seconds
SD  T 4         ;; Start Timer T4
A   T 4         ;; When T4 expires
R   Q 7.4       ;; RESET Power relay (safety: power off)

;; TILT/VERTICAL SELECTOR (I 4.3)
A   I 4.3       ;; IF Tilt/Vertical mode selector = Tilt
AN  F 5.0       ;; AND NOT any motion active
S   F 3.0       ;; SET F 3.0 = Tilt mode
A   F 3.0       ;; Then
R   F 3.1       ;; RESET F 3.1 = Vertical mode

;; OPPOSITE: If Tilt selector = Vertical
AN  I 4.3       ;; IF Tilt/Vertical selector = Vertical
AN  F 5.0       ;; AND NOT any motion active
S   F 3.1       ;; SET F 3.1 = Vertical mode
A   F 3.1       ;; Then
R   F 3.0       ;; RESET F 3.0 = Tilt mode

;; TABLE MOVEMENT DETECTOR (20ms timer)
A(  ;; IF any horizontal/vertical motion
O   Q 6.1   ;; Left/Right contactor
O   Q 6.2   ;; Up/Down contactor
)
AN  T 25        ;; AND T25 (long timer) not active
L   KT 020.1    ;; Load 20ms
SE  T 24        ;; Start Timer T24 edge-triggered
A   T 24        ;; When T24 expires (20ms pulse)
=   Q 7.5       ;; OUTPUT Q 7.5 (motion pulse signal)

;; LONG TIMER FOR TABLE OPERATIONS (180 seconds)
L   KT 180.3    ;; Load 180 seconds (3 minutes)
SE  T 25        ;; Start Timer T25 edge-triggered
JU  FB 18       ;; Jump unconditional to FB18 (table control)
```

**Flags:**
- **F 3.0** = Tilt mode flag (blade angle control)
- **F 3.1** = Vertical mode flag (blade up/down)

**Key Safety:**
- **Timer T25 = 180 seconds (3 minutes)** - ensures hydraulic system doesn't run too long (prevents overheating)
- **Q 7.5** = Motion pulse signal (handshake to external systems)

---

### **OB1 SECTION [7]: Inverter Motion Control (Lines 361-468)**

**Purpose:** Control two inverters (VFDs) for primary and secondary motors

```awl
;; INVERTER 1 STARTUP SEQUENCE
AN  F 1.3       ;; IF Inverter 2 not in sequence
AN  F 1.5       ;; AND NOT Inverter 2 active
S   F 1.4       ;; SET F 1.4 = Inverter 1 sequence

;; INVERTER 1 TIMING
A   F 1.4       ;; IF Inverter 1 sequence active
A   Q 8.2       ;; AND Q 8.2 (special control) active
A(              ;; AND
O   Q 8.0   ;; Forward command
O   Q 8.1   ;; OR Reverse command
)
L   KT 050.1    ;; Load 50ms
SD  T 5         ;; Start Timer T5 (short pulse)

;; STOP INVERTER 1 SEQUENCE
A   I 4.4       ;; IF Power ON
A   T 5         ;; AND T5 expired (50ms passed)
AN  I 73.5      ;; AND NOT forward direction
R   F 1.4       ;; RESET F 1.4 = End sequence

;; PREVENT MULTIPLE INVERTERS ACTIVE
AN  F 1.4       ;; If Inverter 1 sequence inactive
=   Q 73.3      ;; OUTPUT Q 73.3

;; INVERTER 2 SEQUENCE (similar pattern)
AN  F 1.4       ;; IF Inverter 1 NOT in sequence
AN  F 1.5       ;; AND Inverter 2 NOT active
S   F 1.3       ;; SET F 1.3 = Inverter 2 sequence
(... similar timing and logic ...)

;; STOP ALL MOTION (Safety cutoff)
A   F 1.7       ;; IF motion cutoff flag
AN  F 0.0       ;; AND NOT Manual mode
O
A   F 1.7       ;; OR motion cutoff
A   F 0.0       ;; AND Manual mode
A   T 11        ;; AND timing correct
=   F 1.6       ;; SET F 1.6 = Stop all motion flag

;; FORCE RESET ALL CONTACTORS
A   F 1.6       ;; IF stop all motion
R   Q 6.0       ;; RESET Q 6.0 (Forward/Reverse)
R   Q 6.1       ;; RESET Q 6.1 (Left/Right)
R   Q 6.2       ;; RESET Q 6.2 (Up/Down)
R   Q 6.3       ;; RESET Q 6.3 (Tilt)
R   Q 6.4       ;; RESET Q 6.4 (Table Rotation)
R   Q 6.5       ;; RESET Q 6.5 (Disk Rotation)

;; MOTION ACTIVE FLAG
O   Q 6.0       ;; IF any contactor active
O   Q 6.1
O   Q 6.2
O   Q 6.3
O   Q 6.4
O   Q 6.5
=   F 5.0       ;; SET F 5.0 = Motion active
```

**Flags:**
- **F 1.3** = Inverter 2 sequence
- **F 1.4** = Inverter 1 sequence
- **F 1.5** = Inverter 2 active
- **F 1.6** = Stop all motion (safety cutoff)
- **F 1.7** = Motion cutoff condition
- **F 5.0** = Any motion active (used throughout for safety checks)

**Key Logic:** Never run both inverters simultaneously - use **F 1.4** and **F 1.3** to ensure **mutual exclusion**.

---

### **OB1 SECTION [8]: Table Lock & Program Sequencing (Lines 470-532)**

**Purpose:** Control table lock mechanism (critical safety!) and manage programmed sequences

```awl
;; TABLE LOCK MECHANISM (CRITICAL!)
A   I 73.3      ;; IF I 73.3 = Axis Table Rotation selected
S   Q 8.7       ;; SET Q 8.7 = Table lock relay

;; CONDITIONS TO UNLOCK TABLE
A(              ;; IF
O   F 0.1   ;; Translation program
O   F 0.2   ;; OR Passages translation
)
AN  F 0.4       ;; AND NOT Tilt mode
AN  I 72.3      ;; AND NOT Elbo LockTable command
AN  Q 6.4       ;; AND NOT Table Rotation active
O               ;; OR

AN  F 0.4       ;; IF NOT Tilt mode
A   F 0.0       ;; AND Manual mode
AN  I 73.3      ;; AND NOT Axis Table Rotation
AN  I 72.3      ;; AND NOT Elbo LockTable
AN  I 3.2       ;; AND NOT CW button
AN  I 3.3       ;; AND NOT CCW button
AN  Q 6.4       ;; AND NOT Table Rotation active
R   Q 8.7       ;; RESET Q 8.7 = Table unlock

;; TABLE LOCK DEBOUNCE
A   Q 8.7       ;; If table lock active
L   KT 020.1    ;; Load 20ms
SD  T 8         ;; Start Timer T8 (debounce)

;; PROGRAM SEQUENCE FLAGS
O   F 0.1       ;; IF Translation program
O   F 0.2       ;; OR Passages translation
O   F 0.3       ;; OR Passages
=   F 5.3       ;; SET F 5.3 = Programmed mode active

;; SEQUENCE CONTROLLER
O(
A   F 5.1       ;; IF sequence started
AN  F 5.3       ;; AND programmed mode inactive
)
O(
A   F 5.2       ;; OR manual override
AN  F 0.0       ;; AND NOT Manual mode
)
=   F 5.4       ;; SET F 5.4 = Sequence controller

;; START PROGRAM SEQUENCE
A   F 5.3       ;; IF programmed mode active
S   F 5.1       ;; SET F 5.1 = Program sequence started

;; CALL PROGRAM BLOCK (PB10)
A   F 5.1       ;; If sequence started
AN  F 5.2       ;; AND not in manual override
JC  PB 10       ;; JUMP to Program Block 10

;; MANUAL CONTROL OVERRIDE
A   F 0.0       ;; IF Manual mode
S   F 5.2       ;; SET F 5.2 = Manual override

;; RESET PROGRAM SEQUENCE
AN  F 0.1       ;; IF NOT Translation program
AN  F 0.2       ;; AND NOT Passages translation
AN  F 0.3       ;; AND NOT Passages
AN  F 5.0       ;; AND NOT motion active
R   F 5.1       ;; RESET F 5.1 = Stop sequence

;; RESET MANUAL OVERRIDE
AN  F 0.0       ;; IF NOT Manual mode
AN  F 5.0       ;; AND NOT motion active
R   F 5.2       ;; RESET F 5.2 = Manual override off
```

**CRITICAL FLAGS:**
- **F 5.1** = Program sequence started
- **F 5.2** = Manual override active
- **F 5.3** = Programmed mode active
- **F 5.4** = Sequence controller

**Key Safety:** Table lock (Q 8.7) is **ALWAYS SET** unless specific conditions are met. This is a "fail-safe" approach - default is LOCKED, only UNLOCK when safe to do so.

---

### **OB1 SECTION [9]: Motion Command Generation (Lines 533-600)**

**Purpose:** Generate actual movement commands to motors/inverters

```awl
;; MOTION TIMING PULSE (50ms)
O   Q 6.0       ;; IF any contactor active
O   Q 6.1
O   Q 6.2
O   Q 6.3
O   Q 6.4
O   Q 6.5
L   KT 050.0    ;; Load 50ms
SD  T 7         ;; Start Timer T7

;; PHASE TIMING COUNTERS
A   F 4.0       ;; IF phase 1 active
A   T 9         ;; AND phase 1 timer expired
=   F 4.2       ;; SET F 4.2 = Phase 1 complete

A   F 4.1       ;; IF phase 2 active
A   T 10        ;; AND phase 2 timer expired
=   F 4.3       ;; SET F 4.3 = Phase 2 complete

;; START PHASE TIMERS (10ms each)
A   F 4.0       ;; IF phase 1 active
L   KT 010.0    ;; Load 10ms
SD  T 9         ;; Start Timer T9

A   F 4.1       ;; IF phase 2 active
L   KT 010.0    ;; Load 10ms
SD  T 10        ;; Start Timer T10

;; INVERTER 1 FORWARD COMMAND
AN  F 5.4       ;; IF sequence NOT active
A   F 1.4       ;; AND Inverter 1 sequence
A   F 4.2       ;; AND phase 1 complete
=   Q 8.0       ;; OUTPUT Q 8.0 = Inverter 1 Forward

;; INVERTER 1 REVERSE COMMAND
AN  F 5.4       ;; IF sequence NOT active
A   F 1.4       ;; AND Inverter 1 sequence
A   F 4.3       ;; AND phase 2 complete
=   Q 8.1       ;; OUTPUT Q 8.1 = Inverter 1 Reverse

;; SPEED CONTROL (Depends on both inverters)
A(
O   Q 8.0       ;; IF Inverter 1 Forward
O   Q 8.1       ;; OR Inverter 1 Reverse
)
A   F 4.5       ;; AND Speed flag active
=   Q 8.2       ;; OUTPUT Q 8.2 = Inverter 1 speed control

;; INVERTER 2 COMMANDS (Similar pattern)
A   F 1.3       ;; IF Inverter 2 sequence
=   Q 7.7       ;; OUTPUT Q 7.7 = Inverter 2 contactor

AN  F 5.4       ;; IF sequence NOT active
A   F 1.3       ;; AND Inverter 2 sequence
A   F 4.2       ;; AND phase 1 complete
A   T 22        ;; AND special timer
=   Q 8.4       ;; OUTPUT Q 8.4 = Inverter 2 Forward

AN  F 5.4       ;; IF sequence NOT active
A   F 1.3       ;; AND Inverter 2 sequence
A   F 4.3       ;; AND phase 2 complete
A   T 22        ;; AND special timer
=   Q 8.5       ;; OUTPUT Q 8.5 = Inverter 2 Reverse

;; INVERTER 2 SPEED CONTROL
A(
O   Q 8.4       ;; IF Inverter 2 Forward
O   Q 8.5       ;; OR Inverter 2 Reverse
)
A   F 4.5       ;; AND Speed flag active
=   Q 8.3       ;; OUTPUT Q 8.3 = Inverter 2 speed control
```

**Flags:**
- **F 4.0** = Phase 1 active
- **F 4.1** = Phase 2 active
- **F 4.2** = Phase 1 complete
- **F 4.3** = Phase 2 complete
- **F 4.5** = Speed enabled

**Key Concept:** Motor commands are **phased** - Phase 1 runs for 10ms, then Phase 2 runs. This provides smooth acceleration/deceleration.

---

### **OB1 SECTION [10]: Speed Control (Lines 601-659)**

**Purpose:** Handle speed increase/decrease buttons from operator

```awl
;; SPEED UP BUTTON
A   I 4.5       ;; IF Speed+ button pressed
AN  I 4.6       ;; AND Speed- NOT pressed
=   F 3.5       ;; SET F 3.5 = Speed up flag

;; SPEED DOWN BUTTON
A   I 4.6       ;; IF Speed- button pressed
AN  I 4.5       ;; AND Speed+ NOT pressed
=   F 3.6       ;; SET F 3.6 = Speed down flag

;; SPEED CONTROL IN PASSAGES MODE
A(              ;; IF
O   F 3.5   ;; Speed up pressed
O   F 3.6   ;; OR Speed down pressed
)
A(              ;; AND IN
O   F 0.2   ;; Passages with translation
O   F 0.3   ;; OR Passages mode
)
A   I 2.7       ;; AND Back button pressed
=   F 2.0       ;; SET F 2.0 = Speed adjust passages

;; SPEED CONTROL IN TILT MODE
A(              ;; IF
O   F 3.5   ;; Speed up pressed
O   F 3.6   ;; OR Speed down pressed
)
A   F 0.4       ;; AND Tilt mode
=   F 2.1       ;; SET F 2.1 = Speed adjust tilt

;; SPEED CONTROL IN OTHER MODES
AN  F 2.0       ;; IF NOT in passages adjustment
AN  F 2.1       ;; AND NOT in tilt adjustment
A(              ;; AND
O   F 3.5   ;; Speed up
O   F 3.6   ;; OR Speed down
)
=   F 2.2       ;; SET F 2.2 = Speed adjust general

;; FAST BUTTON SPECIAL HANDLING
A   I 4.5       ;; IF Speed+ pressed
=   F 2.3       ;; SET F 2.3 = Speed+ flag

A   I 4.6       ;; IF Speed- pressed
=   F 2.4       ;; SET F 2.4 = Speed- flag

;; TRANSFER SPEED VALUE TO OUTPUT
L   DW 1        ;; Load Data Word 1 (current speed)
T   QW 66       ;; Transfer to Output Word 66 (speed reference to VFD)
```

**Flags:**
- **F 2.0** = Speed control in passages mode
- **F 2.1** = Speed control in tilt mode
- **F 2.2** = General speed control
- **F 2.3** = Speed+ button
- **F 2.4** = Speed- button

**Key Output:** 
- **QW 66** = Speed reference word (sent to VFD for cutting disk speed control)
- Speed is stored in **DW 1** (Data Word 1)
- 16-bit value allows 0-65535 RPM range

---

## 4. FLAG DEFINITIONS & COMPLETE USAGE

### **F 0.x - Program Mode Flags** (Mutually Exclusive)

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 0.0** | Manual Mode | Allow individual axis jogging | I 2.0 (Manual button) | I 2.0 release or I 4.4 (power off) |
| **F 0.1** | Translation Program | Programmed horizontal movement sequences | I 2.1 (if I 72.7=0) | I 2.1 release or power off |
| **F 0.2** | Passages Translation | Multi-cut with horizontal repositioning | I 2.2 (if I 72.7=0) | I 2.2 release or power off |
| **F 0.3** | Passages Mode | Multi-cut at fixed position | I 2.3 (if I 72.7=0) | I 2.3 release or power off |
| **F 0.4** | Tilt Mode Selector | Use tilt axis instead of vertical | I 73.2 (via Elbo) | When F 0.4 sequence complete |
| **F 0.5** | Mode Transition | Brief pulse during mode changes | T 13 expires | Never explicitly reset |
| **F 0.6** | Mode Guard | Prevents rapid mode switching | Logic (F 0.5, F 1.2) | When F 0.5 false |
| **F 0.7** | Speed Reference Active | VFD receiving speed signal | Q 7.1 AND Q 8.6 AND T 3 | Implicit |

**Key Rule:** Only ONE of F 0.0-0.3 can be 1 at a time. If one is set, others must be reset.

---

### **F 1.x - Disk/Motor Startup Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 1.0** | Disk Motor Running | Main disk/blade motor is running | I 4.0 AND Q 7.1 AND NOT T 0 | I 4.1 (OFF button) or I 4.4 (power) or T 1 (timeout) |
| **F 1.1** | Disk Startup Sequence | Initialization phase | F 1.0 started | T 21 timeout or power/buttons |
| **F 1.2** | Fast Mode Ready | Fast command mode initialized | T 13 timeout | I 3.7 release |
| **F 1.3** | Inverter 2 Sequence | Secondary drive sequencing | When F 1.4=0 AND F 1.5=0 | T 6 timeout or power off |
| **F 1.4** | Inverter 1 Sequence | Primary drive sequencing | When F 1.3=0 AND F 1.5=0 | T 5 timeout or power off or I 73.5 release |
| **F 1.5** | Inverter 2 Active | Secondary drive running | When F 1.3 true | I 4.4 release (power off) |
| **F 1.6** | Stop All Motion | Emergency stop all contactors | F 1.7 true | Never (must be manually reset) |
| **F 1.7** | Motion Cutoff Condition | Determine when to stop motion | Complex logic | Manual or power off |

---

### **F 2.x - Speed Control Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 2.0** | Speed Adjust Passages | Speed control in passages mode | F 3.5/3.6 AND (F 0.2 OR F 0.3) AND I 2.7 | (Implicit, one-shot) |
| **F 2.1** | Speed Adjust Tilt | Speed control in tilt mode | F 3.5/3.6 AND F 0.4 | (Implicit, one-shot) |
| **F 2.2** | Speed Adjust General | Speed control in other modes | F 3.5/3.6 AND NOT F 2.0 AND NOT F 2.1 | (Implicit, one-shot) |
| **F 2.3** | Speed+ Button | User pressed speed increase | I 4.5 | Manual (goes false when button released) |
| **F 2.4** | Speed- Button | User pressed speed decrease | I 4.6 | Manual (goes false when button released) |
| **F 2.7** | Disk Rotation Active | Disk is rotating (any direction) | Q 6.1 set | Q 6.0 OR Q 8.0/8.4 reset |

---

### **F 3.x - Mode/Control Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 3.0** | Tilt Mode | Blade in tilt position | I 4.3=1 AND F 5.0=0 | F 3.1 set |
| **F 3.1** | Vertical Mode | Blade in vertical position | I 4.3=0 AND F 5.0=0 | F 3.0 set |
| **F 3.5** | Speed Up Pressed | Operator pressed speed+ button | I 4.5 AND NOT I 4.6 | Manual release |
| **F 3.6** | Speed Down Pressed | Operator pressed speed- button | I 4.6 AND NOT I 4.5 | Manual release |
| **F 3.7** | Laser On | Laser positioning guide enabled | I 4.7 AND F 3.7=0 AND T 14 expired | I 4.7 release or T 15 timeout |

---

### **F 4.x - Phase/Timing Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 4.0** | Phase 1 Active | First motion phase | Sequence logic | Phase complete |
| **F 4.1** | Phase 2 Active | Second motion phase | Sequence logic | Phase complete |
| **F 4.2** | Phase 1 Complete | Phase 1 finished | T 9 expired | Sequence reset |
| **F 4.3** | Phase 2 Complete | Phase 2 finished | T 10 expired | Sequence reset |
| **F 4.5** | Speed Enabled | Motor speed control active | Sequence logic | Sequence end |

---

### **F 5.x - Sequence & Safety Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 5.0** | Motion Active | ANY contactor energized | Q 6.0 OR Q 6.1 OR Q 6.2 OR Q 6.3 OR Q 6.4 OR Q 6.5 | All contactors reset |
| **F 5.1** | Sequence Started | Programmed sequence begun | F 5.3 true | F 0.1=0 AND F 0.2=0 AND F 0.3=0 AND F 5.0=0 |
| **F 5.2** | Manual Override | Manual mode active during sequence | F 0.0 true | F 0.0=0 AND F 5.0=0 |
| **F 5.3** | Programmed Mode Active | In any programmed mode | F 0.1 OR F 0.2 OR F 0.3 | When all false |
| **F 5.4** | Sequence Controller | Sequence control active | F 5.1 AND NOT F 5.2 OR F 5.2 AND NOT F 0.0 | (Complex) |

---

### **F 6.x - Fast Command Mode Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 6.3** | Fast Mode (Help Feature) | Special fast command mode | C 1 counter=0 AND T 17 expired | I 3.7 release AND (F 7.0 OR F 7.1 OR ... any F 7.x) |

---

### **F 7.x - Fast Mode Sub-Flags** (F 6.3 dependent)

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 7.0-7.5** | Fast Mode Options | Individual fast command selections | (Requires F 6.3=1) | FB15 logic |
| **F 7.6** | Fast Mode Toggle | Toggles fast mode on/off | I 3.7 edge detection | I 3.7 release |
| **F 7.7** | Fast Mode Active | Fast mode currently active | F 7.6 true | I 3.7 release |

---

### **F 8.x - Fast Mode Sequencer**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 8.0, F 8.1, F 8.2** | Sequencer States | Track fast mode state machine | Complex state logic | Next state or reset |

---

### **F 9.x - Advanced Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 9.4** | Position Flag | Special position tracking | I 72.0 AND F 0.4 AND I 72.5 AND T 19 | (Implicit) |

---

### **F 10.x - Table Position Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 10.1** | Table 1 Up | Table 1 in raised position | I 9.1 (button) | I 9.2 (lower button) |
| **F 10.2** | Table 1 Down | Table 1 in lowered position | I 9.2 (button) | I 9.1 (raise button) |
| **F 10.3** | Table 2 Up | Table 2 in raised position | I 9.3 (button) | I 9.4 (lower button) |
| **F 10.4** | Table 2 Down | Table 2 in lowered position | I 9.4 (button) | I 9.3 (raise button) |

---

### **F 11.x - Table Sensor Flags**

| Flag | Name | Purpose | Set By | Reset By |
|------|------|---------|--------|----------|
| **F 11.0** | Table 2 End Switch | Table 2 reached position | I 9.0 (end switch) | Complex logic |
| **F 11.1-11.4** | Manual Table Commands | Track table button presses | I 9.1-9.4 (buttons) | (Implicit) |

---

## 5. TIMER DEFINITIONS & USAGE

| Timer | Duration | Purpose | Started By | Used For |
|-------|----------|---------|-----------|----------|
| **T 0** | 60s | Startup initialization | F 1.0 | Prevent rapid disk restarts |
| **T 1** | 600s (10m) | Programmed sequence timeout | OB1 Section 3 | Safety: max sequence time |
| **T 2** | 80ms | Disk star-delta timing | F 1.0 | Star-delta soft-start duration |
| **T 3** | 400ms | Speed ramp timer | I 73.7 AND Q 8.6 | VFD acceleration ramp |
| **T 4** | 30s | Power timeout | T 1 expired | Auto power-off safety |
| **T 5** | 50ms | Inverter 1 pulse | F 1.4 | Command duration |
| **T 6** | 50ms | Inverter 2 pulse | F 1.3 | Command duration |
| **T 7** | 50ms | Motion pulse | Any Q 6.x | Motion handshake |
| **T 8** | 20ms | Table lock debounce | Q 8.7 | Mechanical settling |
| **T 9** | 10ms | Phase 1 timer | F 4.0 | Phase timing |
| **T 10** | 10ms | Phase 2 timer | F 4.1 | Phase timing |
| **T 11** | 15ms | Motion stop delay | F 1.7 | Smooth deceleration |
| **T 13** | 5ms | Mode transition | OB1 Section 1 | Mode change timing |
| **T 14** | 30s | Laser safety timer | I 4.7 AND NOT Q 7.2 | Auto laser shutoff |
| **T 15** | 300s (5m) | Laser hard limit | I 4.7 AND (F 0.1 OR F 0.2 OR F 0.3) | Laser safety |
| **T 17** | 2s | Fast button debounce | I 3.7 | Multi-click detection |
| **T 18** | 30s | Delayed startup | Speed control | Startup ramp |
| **T 19** | Variable | Speed ramp timer | I 72.0 AND F 0.4 | Dynamic timing |
| **T 20** | 20s | VFD stabilization | Q 7.1 active | VFD setup time |
| **T 21** | 20s | Disk startup | F 1.1 | Motor acceleration |
| **T 22** | 50ms | Phase timing | F 1.3 sequence | Sequencer control |
| **T 24** | 20ms | Motion detector | Q 6.1 OR Q 6.2 | Brief motion pulse |
| **T 25** | 180s (3m) | Table hydraulic limit | Motion active | Prevent overheating |
| **T 26** | 120s | Hydraulic pump safety | Q 7.3 active | Oil temperature |

---

## 6. KEY CONTROL SEQUENCES

### **Manual Cutting Sequence:**
```
1. Operator sets mode: Manual (F 0.0 = 1)
2. Operator presses I 4.0 (Disk ON)
   → F 1.1 = 1 (startup sequence)
   → T 21 countdown (20s)
   → Q 6.6 = 1 (Star soft-start)
   → T 2 countdown (80ms)
   → Q 6.7 = 1 (Switch to Delta)
3. Elbo proportional input provides:
   → I 73.0-73.4 (axis selection)
   → I 73.5-73.6 (direction)
   → I 73.7 (velocity enable)
4. OB1 routes to appropriate Q 6.x
5. Motor runs, operator controls with Elbo joystick
6. End switches (I 5.x) stop motion at limits
7. Operator releases or presses I 4.1 (OFF)
   → F 1.0/1.1 reset
   → Contactors de-energized
   → Disk slows down
```

### **Programmed Cutting Sequence:**
```
1. Operator selects program: Passages (F 0.3 = 1)
2. Program is NOT active until:
   - I 72.7 = 0 (disk not running)
   - Mode must be set first
3. OB1 calls PB10 (Program Block 10)
4. PB10 contains pre-programmed moves
5. Each move:
   - Sets appropriate Q 6.x contactor
   - Waits for sensor feedback
   - Times motion (via T 7, T 9, T 10)
6. After programmed sequence:
   - Operator manually moves for next cut
   - Or system loops to repeat pattern
```

### **Safety Cutoff Sequence (Emergency):**
```
Triggered if F 1.7 = 1 (motion cutoff condition):
1. F 1.6 = 1 (Stop all motion)
2. FORCED RESET ALL CONTACTORS:
   - R Q 6.0 (Forward/Reverse)
   - R Q 6.1 (Left/Right)
   - R Q 6.2 (Up/Down)
   - R Q 6.3 (Tilt)
   - R Q 6.4 (Table Rotation)
   - R Q 6.5 (Disk Rotation)
3. All motion stops within 15ms (T 11)
4. System in safe state
```

---

## 7. FLAG CROSS-REFERENCE QUICK LOOKUP

**Need to know what controls a specific flag?**

### "What sets F 0.0 (Manual Mode)?"
- **I 2.0** button pressed
- **F 6.3** must be 0
- All motion buttons (I 2.4-7, I 3.0-5) must be 0
- **F 5.0** (motion active) must be 0

### "What resets F 0.0?"
- I 2.0 button released
- **I 4.4** (Power OFF) = pressed
- Timeout (T 1 expires)

### "What does F 5.0 control?"
- **F 5.0** = 1 blocks manual mode entry
- **F 5.0** = 1 blocks program mode entry
- **F 5.0** = 1 prevents table mode changes
- **F 5.0** = 0 allows all above

### "What must be 0 to start a program?"
- **I 72.7** = 0 (disk must not be running)
- **F 0.0** = 0 (not in manual mode)
- **F 5.0** = 0 (no motion)
- **F 6.3** = 0 (not in fast mode)

### "What stops the disk?"
- **I 4.1** button (Disk OFF)
- **I 4.4** button (Power OFF)
- **T 0** timeout (60s)
- **T 1** timeout (10 minute program timeout)
- Manual: **I 73.7 = 0** (velocity signal loss)

---

## CONCLUSION

This guide explains:
✅ What each flag does  
✅ Why it's set and reset  
✅ What other flags depend on it  
✅ How flags interlock for safety  
✅ The complete program flow  

**The key principle:** Flags form a **state machine** where each state has specific entry and exit conditions. Safety is built in through **mutual exclusions** (e.g., F 0.0-0.3 can't both be 1) and **timeouts** (e.g., T 1 prevents infinite loops).

