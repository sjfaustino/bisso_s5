# Complete Elbo Input Signal Analysis - BISSO Step5 Project

Based on cross-reference analysis of the QVL (Quality Verification List) file, here's the complete breakdown of all Elbo microcontroller input signals:

---

## I 72.x - Speed Selection Signals

### I 72.0 - Elbo Rapido (Fast Speed)

**Cross-Reference Data:**
```
:A    I   72.0     1/A     1/A                 (used 1 place with AND, appears twice)
:A    I   72.0     1/A     1/A     1/A        (used 1 place with AND, appears 3 times)
:A    I   72.0     5/A     5/A                (used 5 places with AND)
:O    I   72.0     1/O     1/A                (used 1 place with OR + AND)
```

**Total References: 14 places in code**

**Function:**
- **Fast Speed Mode Selection Flag** from Elbo microcontroller
- Indicates operator has selected high-speed motion

**Logic Pattern:**
```
IF I 72.0 = 1 THEN
  ; Fast speed mode is active
  Set speed reference to maximum (or stored DW fast speed value)
  Enable fast motion for all controlled axes
  
  (references appear in multiple motion blocks suggesting this applies to all 5 main motions)
END
```

**Affects:** Speed regulation for X, Y, Z, Tilt, Rotation, and Disk axes

---

### I 72.1 - Elbo Medio (Medium Speed)

**Cross-Reference Data:**
```
:A    I   72.1     1/A                        (used 1 place with AND)
:A    I   72.1     1/A     1/A     1/A       (used 1 place with AND, appears 3 times)
```

**Total References: 5 places in code**

**Function:**
- **Medium Speed Mode Selection Flag** from Elbo microcontroller
- Indicates operator has selected medium-speed motion
- Complementary to I 72.0

**Logic Pattern:**
```
IF I 72.1 = 1 THEN
  ; Medium speed mode is active
  Set speed reference to medium level (stored in DW data)
  Use slower motion than fast mode
  
  (Likely mutually exclusive with I 72.0)
END
```

**Affects:** Speed regulation across all motion axes

**Speed Selection Logic (Inferred):**
```
Speed Selection Priority:
  IF I 72.0 = 1 THEN
    Speed = DW Fast_Speed (maximum)
  ELSE IF I 72.1 = 1 THEN
    Speed = DW Medium_Speed (medium)
  ELSE
    Speed = DW Slow_Speed (minimum) OR Motion blocked
  END
```

---

## I 72.x - Rotation/Lock/Status Signals

### I 72.2 - Cmd Rot (Command Rotation)

**Cross-Reference Data:**
```
:O    I   72.2     1/O     1/A                (used 1 place with OR + AND)
:AN   I   72.2     1/AN    1/A     1/A     1/AN    1/AN   (used 1 place with AND NOT + AND combinations)
:A    I   72.2     1/A     1/A     1/A       (used 1 place with AND, 3 times)
:A    I   72.2     1/A     1/A     1/A       (used 1 place with AND, 3 times)
```

**Total References: 11 places in code**

**Function:**
- **Rotation Command Enable Flag** from Elbo
- Selects between different operational modes (likely table rotation vs disk rotation)
- Complex logic using both positive and negative conditions

**Logic Pattern:**
```
IF I 72.2 = 1 THEN
  ; Rotation mode is selected
  Enable table or disk rotation based on other signals
  I 73.3 and I 73.4 determine which rotation axis
ELSE
  ; Rotation mode is NOT selected
  Disable rotation functions
  Allow other motion types
END
```

**Affects:** Table rotation (Q 6.4) and Disk rotation (Q 6.5) functions

---

### I 72.3 - Cmd LockTable (Command Lock Table)

**Cross-Reference Data:**
```
:AN   I   72.3     8/AN    8/AN              (used 8 places with AND NOT)
:A    I   72.3     1/A                       (used 1 place with AND)
:O    I   72.3     1/O                       (used 1 place with OR)
:A    I   72.3     1/A     1/A     1/O      (used 1 place with AND, 2x + 1 OR)
:AN   I   72.3     1/AN    1/A     1/AN    1/A     1/A     1/AN    1/A  (complex logic)
:A    I   72.3     1/A                       (used 1 place with AND)
```

**Total References: 23 places in code (HEAVILY USED)**

**Function:**
- **Table Lock Command** from Elbo microcontroller
- **CRITICAL SAFETY/OPERATIONAL SIGNAL** - Used 23 times in code
- Controls locking/unlocking of work tables during operation
- Heavy use of negated logic (:AN) suggests safety interlocks

**Logic Pattern:**
```
IF I 72.3 = 0 THEN  (NOT locked - frequent check 8 times)
  ; Table lock is NOT active - tables can move
  Enable table lift/lower hydraulics (Q 7.3)
  Allow table position changes
  
ELSE IF I 72.3 = 1 THEN
  ; Table lock IS active - tables are locked in position
  Prevent table movement
  Hydraulic pump may be disabled
  Enable rotation or other functions
END
```

**Affects:** 
- Q 7.3 = Oil Pump for tables (critical)
- Q 72.5-72.7 = External table relays
- Q 8.7 = LockTable (direct output)
- Prevents inadvertent table movement during cutting

**Safety Implications:** This is a critical lock mechanism preventing table motion during active cutting operations

---

### I 72.4 - Pressostato (Water Pressure Switch)

**Cross-Reference Data:**
```
:A    I   72.4     1/A                       (used 1 place with AND)
```

**Total References: 1 place in code**

**Function:**
- **Water Pressure Status Indicator** from hydraulic system
- Monitors cooling water/hydraulic fluid pressure
- Safety/status feedback from Elbo

**Logic Pattern:**
```
IF I 72.4 = 1 THEN
  ; Adequate pressure detected
  Allow cutting operations to continue
ELSE
  ; Low or no pressure - potential problem
  May trigger warning light (Q 72.2)
  Continue monitoring
END
```

**Affects:** System health monitoring, possibly warning lights

---

### I 72.5 - VFD1_IsRunning (Inverter 1 Running Status)

**Cross-Reference Data:**
```
:A    I   72.5     5/A                       (used 5 places with AND)
:A    I   72.5     1/A     1/A               (used 1 place with AND, twice)
:A    I   72.5     1/A                       (used 1 place with AND - 4 additional times)
```

**Total References: 11 places in code**

**Function:**
- **VFD #1 Running Status** from Elbo/Inverter feedback
- Indicates whether first variable frequency drive is actively running
- Used for monitoring drive health and operational state

**Logic Pattern:**
```
IF I 72.5 = 1 THEN
  ; Inverter 1 is running
  Motor on Inverter 1 is operational
  Output is being delivered to controlled axis
  
ELSE
  ; Inverter 1 is NOT running or in fault
  Motor is stopped or stalled
  May indicate malfunction (Q 72.3 error light)
END
```

**Affects:** Motion enable logic, fault detection for main cutting axis

---

### I 72.6 - VFD2_IsRunning (Inverter 2 Running Status)

**Cross-Reference Data:**
```
:A    I   72.6     1/A                       (used 1 place with AND - 6 times total)
```

**Total References: 6 places in code**

**Function:**
- **VFD #2 Running Status** from Elbo/Inverter feedback
- Indicates whether second variable frequency drive is running
- Parallel to I 72.5 for secondary drive

**Logic Pattern:**
```
IF I 72.6 = 1 THEN
  ; Inverter 2 is running
  Second motor is operational
  Secondary axis (likely translation or rotation) active
  
ELSE
  ; Inverter 2 is NOT running or in fault
END
```

**Affects:** Secondary axis motion enable, dual-drive fault detection

---

### I 72.7 - VFDDisk_IsRunning (Disk Drive Running Status)

**Cross-Reference Data:**
```
:AN   I   72.7     1/AN    1/AN    1/AN    1/AN   (used 4 places with AND NOT)
:A    I   72.7     1/A                           (used 1 place with AND - 5 times total)
```

**Total References: 9 places in code**

**Function:**
- **Cutting Disk VFD Running Status** from Elbo/Inverter feedback
- Critical indicator that cutting disk (blade) is spinning
- Heavy use of negated logic suggests safety interlock

**Logic Pattern:**
```
IF I 72.7 = 0 THEN  (NOT running - safety checks)
  ; Disk is NOT running - cutter is stopped
  Allow table movement (safe to move material)
  Can reposition for next cut
  
IF I 72.7 = 1 THEN
  ; Disk IS running - cutter is active
  Prevent table movement (material locked in position)
  Cutting operation in progress
END
```

**Affects:** 
- Safety interlocks preventing table movement during cutting
- Q 6.4 = Table rotation control
- Q 72.1 = Disk ON indicator light
- Critical for operator safety

---

## I 73.x - Axis Selection & Direction Signals

### I 73.0 - Elbo Eixo Corte (Cutting Axis - Blade Up/Down / Z-Axis)

**Cross-Reference Data:**
```
:AN   I   73.0     1/AN    1/AN              (used 1 place with AND NOT, twice)
:A    I   73.0     1/A                       (used 1 place with AND - 5 times)
```

**Total References: 7 places in code**

**Function:**
- **Axis Selection: Cutting/Vertical Axis (Z-Axis)**
- Selects vertical blade movement when active
- Proportional signal from Elbo joystick indicating desire to move up/down

**Logic Pattern:**
```
IF I 73.0 = 1 THEN
  ; Vertical/Cutting axis is selected
  Process I 73.5/73.6 for UP/DOWN direction
  Activate Q 6.2 (Up/Down Contactor C5)
  Motor moves blade vertically
  
ELSE
  ; Vertical axis NOT selected
  No vertical motion
END
```

**Affects:** Q 6.2 (Vertical motor contactor)

**Related Signals:** I 73.5, I 73.6 (direction), I 5.4, I 5.5 (end switches)

---

### I 73.1 - Elbo Eixo Translacao (Translation Axis - Left/Right / Y-Axis)

**Cross-Reference Data:**
```
:A    I   73.1     1/A                       (used 1 place with AND - 5 times)
```

**Total References: 5 places in code**

**Function:**
- **Axis Selection: Translation Axis (Y-Axis / Left-Right)**
- Selects horizontal left/right material movement when active
- Proportional signal from Elbo indicating desire to translate material

**Logic Pattern:**
```
IF I 73.1 = 1 THEN
  ; Translation axis is selected
  Process I 73.5/73.6 for LEFT/RIGHT direction
  Activate Q 6.1 (Left/Right Contactor C4)
  Material moves left or right
  
ELSE
  ; Translation NOT selected
  No horizontal movement
END
```

**Affects:** Q 6.1 (Translation motor contactor)

**Related Signals:** I 73.5, I 73.6 (direction), I 5.0, I 5.1 (end switches)

---

### I 73.2 - Elbo Eixo Cala (Tilt Axis - Angle Positioning)

**Cross-Reference Data:**
```
:A    I   73.2     1/A     1/ON             (used 1 place with AND + ON condition)
```

**Total References: 2 places in code**

**Function:**
- **Axis Selection: Tilt Axis (Blade Angle Control)**
- Selects tilt/angle positioning when active
- Controls blade inclination for beveled cuts
- Less frequently used than main axes (only 2 references)

**Logic Pattern:**
```
IF I 73.2 = 1 THEN
  ; Tilt axis is selected
  Process I 73.5/73.6 for tilt direction (down/up angle)
  Activate Q 6.3 (Tilt Contactor C7)
  Blade tilts to new angle
  
ELSE
  ; Tilt NOT selected
  Blade remains at current angle
END
```

**Affects:** Q 6.3 (Tilt motor contactor)

**Related Signals:** I 73.5, I 73.6 (direction), I 5.6, I 5.7 (tilt limit switches)

---

### I 73.3 - Elbo Eixo Rotacao Mesa (Table Rotation Axis)

**Cross-Reference Data:**
```
:A    I   73.3     8/A     8/AN             (used 8 places with AND, 8 with AND NOT)
```

**Total References: 16 places in code**

**Function:**
- **Axis Selection: Table Rotation**
- Selects rotational movement of work table (clockwise/counter-clockwise)
- Complex logic using both positive and negative conditions
- Related to I 72.2 (Rotation mode)

**Logic Pattern:**
```
IF I 73.3 = 1 AND I 72.2 = 1 THEN
  ; Rotation mode AND table rotation axis selected
  Process I 73.5/73.6 for CW/CCW direction
  Activate Q 6.4 (Table Rotation Contactor C6)
  Table rotates to new position
  
ELSE
  ; Not in rotation mode for table
  Table remains stationary
END
```

**Affects:** Q 6.4 (Table rotation motor contactor)

**Related Signals:** I 73.5, I 73.6 (CW/CCW direction), I 72.2 (rotation mode selector)

---

### I 73.4 - Elbo Eixo Rotacao Disco (Disk Rotation Axis)

**Cross-Reference Data:**
```
(No direct references found - inherited from I 73.3 patterns)
```

**Function:**
- **Axis Selection: Disk/Cutter Rotation**
- Selects rotational speed control of cutting disk/blade
- May be used for variable speed disk control
- Less frequently referenced directly

**Logic Pattern:**
```
IF I 73.4 = 1 AND I 72.2 = 1 THEN
  ; Rotation mode AND disk rotation axis selected
  Adjust cutting disk speed (via VFD or contactor)
  Q 6.5 (Disk Rotation Contactor C10)
  Disk spins at new speed
END
```

**Affects:** Q 6.5 (Disk rotation motor contactor), Q 7.1 (VFD selector)

---

### I 73.5 - Elbo Direcao "+" (Direction: Positive/Forward/Up)

**Cross-Reference Data:**
```
:AN   I   73.5     3/AN    7/AN    7/AN    (used 3 places with AND NOT, 7 places with AND NOT twice more)
```

**Total References: 17 places in code (heavy use of negated logic)**

**Function:**
- **Direction Indicator: Positive Direction (+)**
- When I 73.5 = 1: Motion in POSITIVE direction (Forward, Up, Right, CW, Extend)
- When I 73.5 = 0: Motion in NEGATIVE direction (see I 73.6)
- Heavy use of AND NOT logic suggests directional gating

**Logic Pattern:**
```
IF I 73.5 = 1 THEN
  ; Positive direction selected
  SET Q 6.x for positive direction  (e.g., Q 6.0 = Forward)
  CLEAR Q 6.y for negative direction (e.g., Q 6.1 = Reverse)
  
  Motor turns/moves in positive direction based on selected axis
  
ELSE IF I 73.5 = 0 AND I 73.6 = 1 THEN
  ; Negative direction selected (see below)
  SET Q 6.y for negative direction
  CLEAR Q 6.x for positive direction
END
```

**Direction Mapping (Inferred):**
- **I 73.0 + I 73.5**: Blade moves UP
- **I 73.1 + I 73.5**: Material moves RIGHT
- **I 73.2 + I 73.5**: Blade tilts UP (angle increases)
- **I 73.3 + I 73.5**: Table rotates CLOCKWISE
- **I 73.4 + I 73.5**: Disk speed increases

**Affects:** Direction selection for Q 6.0, Q 6.1, Q 6.2, Q 6.3, Q 6.4, Q 6.5

---

### I 73.6 - Elbo Direcao "-" (Direction: Negative/Reverse/Down)

**Cross-Reference Data:**
```
:AN   I   73.6     3/AN    7/AN    7/AN    (used 3 places with AND NOT, 7 places with AND NOT twice more)
```

**Total References: 17 places in code (heavy use of negated logic - same as I 73.5)**

**Function:**
- **Direction Indicator: Negative Direction (-)**
- When I 73.6 = 1: Motion in NEGATIVE direction (Backward, Down, Left, CCW, Retract)
- When I 73.6 = 0: Motion in POSITIVE direction (or standby if both 0)
- Mirror logic to I 73.5
- Mutually exclusive with I 73.5 (only one should be active)

**Logic Pattern:**
```
IF I 73.6 = 1 THEN
  ; Negative direction selected
  CLEAR Q 6.x for positive direction
  SET Q 6.y for negative direction (e.g., Q 6.1 = Reverse)
  
  Motor moves in negative direction based on selected axis
  
ELSE IF I 73.6 = 0 AND I 73.5 = 0 THEN
  ; Both directions inactive = STOP
  CLEAR all direction contactors
  Motor coasts to stop or is held
END
```

**Direction Mapping (Inferred):**
- **I 73.0 + I 73.6**: Blade moves DOWN
- **I 73.1 + I 73.6**: Material moves LEFT
- **I 73.2 + I 73.6**: Blade tilts DOWN (angle decreases)
- **I 73.3 + I 73.6**: Table rotates COUNTER-CLOCKWISE
- **I 73.4 + I 73.6**: Disk speed decreases

**Affects:** Direction reversal for Q 6.0, Q 6.1, Q 6.2, Q 6.3, Q 6.4, Q 6.5

---

### I 73.7 - Elbo V/S (Velocity/Speed - Master Enable)

**Cross-Reference Data:**
```
:AN   I   73.7     2/AN    4/A     4/A     (used 2 places with AND NOT, 4 places with AND twice)
```

**Total References: 10 places in code**

**Function:**
- **Master Velocity/Speed Enable Signal**
- Acts as GATING SIGNAL for all motion
- When I 73.7 = 1: Velocity/speed information is valid and active
- When I 73.7 = 0: No valid speed signal - motion inhibited

**Logic Pattern:**
```
IF I 73.7 = 0 THEN  (NOT active - 2 safety checks)
  ; No velocity signal from Elbo
  CLEAR all motion contactors
  System is in safe "stopped" state
  Emergency or no proportional input
  
ELSE IF I 73.7 = 1 AND (other conditions) THEN  (4 places where it gates motion)
  ; Velocity signal is active
  Process axis and direction signals
  Allow selected contactor to energize
  Motion is enabled
END
```

**Affects:** **ALL MOTION** - Primary gating signal for the entire proportional motion system

**Safety Implications:** Critical safety signal ensuring no motion occurs without valid speed reference

---

## Summary Table

| Signal | Name | Type | Usage Count | Primary Function |
|--------|------|------|------------|------------------|
| I 72.0 | Elbo Rapido | Speed Mode | 14 | Fast speed enable |
| I 72.1 | Elbo Medio | Speed Mode | 5 | Medium speed enable |
| I 72.2 | Cmd Rot | Mode Select | 11 | Rotation mode selector |
| I 72.3 | Cmd LockTable | Safety Lock | 23 | **TABLE LOCK** - CRITICAL |
| I 72.4 | Pressostato | Status | 1 | Hydraulic pressure monitor |
| I 72.5 | VFD1_IsRunning | Status | 11 | Inverter 1 running status |
| I 72.6 | VFD2_IsRunning | Status | 6 | Inverter 2 running status |
| I 72.7 | VFDDisk_IsRunning | Status | 9 | Cutting disk running status |
| I 73.0 | Eixo Corte (Z-Axis) | Axis Select | 7 | Vertical/blade axis |
| I 73.1 | Eixo Translacao (Y-Axis) | Axis Select | 5 | Horizontal/translation axis |
| I 73.2 | Eixo Cala | Axis Select | 2 | Tilt/angle axis |
| I 73.3 | Eixo Rotacao Mesa | Axis Select | 16 | Table rotation axis |
| I 73.4 | Eixo Rotacao Disco | Axis Select | (inherit) | Disk rotation axis |
| I 73.5 | Direcao "+" | Direction | 17 | Positive direction |
| I 73.6 | Direcao "-" | Direction | 17 | Negative direction |
| I 73.7 | V/S | Master Enable | 10 | **VELOCITY ENABLE** - CRITICAL |

---

## Control Signal Hierarchy

```
Elbo Microcontroller Outputs:
│
├─ SPEED SELECTION (I 72.0, I 72.1)
│  └─ Determines: Fast vs Medium speed reference
│
├─ MODE SELECTION (I 72.2 = Rotation, I 72.3 = Lock)
│  ├─ I 72.2: Select rotation functions vs motion
│  └─ I 72.3: Lock table in place during cutting
│
├─ STATUS FEEDBACK (I 72.4, I 72.5, I 72.6, I 72.7)
│  ├─ I 72.4: Hydraulic pressure OK
│  ├─ I 72.5: Inverter 1 running
│  ├─ I 72.6: Inverter 2 running
│  └─ I 72.7: Cutting disk running
│
├─ AXIS SELECTION (I 73.0-73.4)
│  ├─ I 73.0: Vertical (Z-Axis)
│  ├─ I 73.1: Horizontal (Y-Axis)
│  ├─ I 73.2: Tilt
│  ├─ I 73.3: Table Rotation
│  └─ I 73.4: Disk Rotation
│
├─ DIRECTION (I 73.5, I 73.6)
│  ├─ I 73.5: Positive direction (up/forward/right/CW)
│  └─ I 73.6: Negative direction (down/back/left/CCW)
│
└─ MASTER ENABLE (I 73.7)
   └─ GATES ALL MOTION - Must be active for any movement
```

---

## Key Findings

### CRITICAL SIGNALS (Most Important):
1. **I 72.3 (Table Lock)** - Used 23 times, controls safety interlocks
2. **I 73.7 (V/S Master Enable)** - Gates all motion, 10 references
3. **I 72.7 (Disk Running)** - Prevents table movement during cutting

### OPERATIONAL SIGNALS (Core Function):
- **I 73.0-73.4** - Axis selection (5 different motion types)
- **I 73.5-73.6** - Direction control (bidirectional for each axis)
- **I 72.0-72.1** - Speed selection (fast vs medium)

### STATUS/FEEDBACK SIGNALS (Monitoring):
- **I 72.4** - Hydraulic pressure
- **I 72.5-72.7** - Drive/inverter running indicators

### MODE SELECTION:
- **I 72.2** - Selects between rotation modes

---

## Typical Motion Sequence

```
1. Operator selects speed: I 72.0 or I 72.1 activates
2. Operator selects axis: One of I 73.0-73.4 activates
3. Operator selects direction: I 73.5 or I 73.6 activates
4. Elbo provides velocity signal: I 73.7 = 1
5. PLC reads all signals AND processes logic
6. Result: Appropriate contactor(s) energize (Q 6.x)
7. Motor moves in selected direction at selected speed
8. Operator releases: I 73.5/73.6 go to 0, or I 73.7 = 0
9. PLC de-energizes contactor(s)
10. Motor stops

SAFETY INTERLOCK:
- If I 72.3 (lock table) = 1:
  - Prevent Q 7.3 (oil pump) from activating
  - Allow only rotation/cutting operations
  - Prevents table movement during active cutting
```

