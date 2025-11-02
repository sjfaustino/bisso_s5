# Elbo Signals - Quick Reference Summary

## All 16 Signals at a Glance

| Signal | Name (Portuguese) | English Translation | Signal Type | State=1 Means | State=0 Means | Uses |
|--------|-------------------|-------------------|-------------|---------------|---------------|------|
| **I 72.0** | Elbo Rapido | Fast Speed | Speed Mode | **FAST speed** selected | Not fast | 14 |
| **I 72.1** | Elbo Medio | Medium Speed | Speed Mode | **MEDIUM speed** selected | Not medium | 5 |
| **I 72.2** | Cmd Rot | Command Rotation | Mode Select | **Rotation mode** enabled | Normal mode | 11 |
| **I 72.3** ⚠️ | Cmd LockTable | Command Lock Table | Safety Lock | **TABLE LOCKED** (prevent movement) | Table free to move | **23** |
| **I 72.4** | Pressostato | Pressure Switch | Status | **Pressure OK** | Low/no pressure | 1 |
| **I 72.5** | VFD1_IsRunning | Inverter 1 Running | Status | **Inverter 1 ON** | Inverter 1 OFF/fault | 11 |
| **I 72.6** | VFD2_IsRunning | Inverter 2 Running | Status | **Inverter 2 ON** | Inverter 2 OFF/fault | 6 |
| **I 72.7** ⚠️ | VFDDisk_IsRunning | Disk Drive Running | Status | **Cutting disk ON** | Cutting disk OFF | 9 |
| **I 73.0** | Eixo Corte | Cutting Axis (Z) | Axis Select | **Vertical motion** selected | Vertical motion NOT selected | 7 |
| **I 73.1** | Eixo Translacao | Translation Axis (Y) | Axis Select | **Horizontal/Left-Right** motion selected | Not selected | 5 |
| **I 73.2** | Eixo Cala | Tilt Axis | Axis Select | **Tilt/Angle** motion selected | Not selected | 2 |
| **I 73.3** | Eixo Rotacao Mesa | Table Rotation Axis | Axis Select | **Table rotation** selected | Not selected | 16 |
| **I 73.4** | Eixo Rotacao Disco | Disk Rotation Axis | Axis Select | **Disk speed control** selected | Not selected | - |
| **I 73.5** | Direcao "+" | Direction: Positive/Forward/Up | Direction | **Positive direction** (UP, RIGHT, FWD, CW) | Neutral/negative | 17 |
| **I 73.6** | Direcao "-" | Direction: Negative/Backward/Down | Direction | **Negative direction** (DOWN, LEFT, BACK, CCW) | Neutral/positive | 17 |
| **I 73.7** ⚠️ | V/S (Velocidade/Sentido) | Velocity/Speed Enable | Master Gate | **Valid speed signal active** - MOTION ENABLED | No speed signal - MOTION BLOCKED | **10** |

---

## Signal Categories Explained

### Category 1: SPEED SELECTION (I 72.0, I 72.1)
**Purpose:** Determine the operating speed level  
**How It Works:** Operator selects fast or medium speed via Elbo controller  
**Effect:** Applied to all axes when motion occurs  
**Example:**
- I 72.0 = 1: All motors run at fast speed (e.g., 100 RPM)
- I 72.1 = 1: All motors run at medium speed (e.g., 50 RPM)
- Both = 0: Default or slow speed

### Category 2: MODE SELECTION (I 72.2, I 72.3)
**Purpose:** Select operational mode or safety state  
**How It Works:**
- **I 72.2:** Switches between regular motion vs rotation/disk control
- **I 72.3:** LOCKS table in place during cutting operations (SAFETY CRITICAL)

### Category 3: STATUS FEEDBACK (I 72.4, I 72.5, I 72.6, I 72.7)
**Purpose:** Monitor system health and enable safeties  
**How It Works:** Feedback from VFD drives and hydraulic system tells PLC if devices are operating normally  
**Effect:** Prevents operation if feedback signals indicate problems

### Category 4: AXIS SELECTION (I 73.0-73.4)
**Purpose:** Select which physical axis to move  
**How It Works:** Only ONE should be active at a time (mutual exclusion)  
**Five Possible Motions:**
1. **I 73.0:** Vertical/Cutting (Z-axis) - Blade up/down
2. **I 73.1:** Translation (Y-axis) - Material left/right
3. **I 73.2:** Tilt - Blade angle adjustment
4. **I 73.3:** Table Rotation - Clockwise/counter-clockwise
5. **I 73.4:** Disk Rotation - Cutting speed adjustment

### Category 5: DIRECTION (I 73.5, I 73.6)
**Purpose:** Determine which direction to move selected axis  
**How It Works:** Typically mutually exclusive (both = 0 means stop)
- **I 73.5 = 1:** Positive direction (up, right, forward, clockwise, extend)
- **I 73.6 = 1:** Negative direction (down, left, backward, counter-clockwise, retract)

### Category 6: MASTER ENABLE (I 73.7)
**Purpose:** CRITICAL GATE - Allow or block ALL motion  
**How It Works:**
- **I 73.7 = 0:** No motion possible (safe state - like E-stop)
- **I 73.7 = 1:** Motion allowed if other conditions met

---

## Typical Elbo Control Sequence

```
1. STARTUP
   ├─ I 72.0 or I 72.1 = 1  (Operator selects speed)
   ├─ I 72.3 = ?            (Operator locks/unlocks table)
   ├─ I 72.7 = ?            (Disk status determined)
   └─ Wait for command

2. OPERATOR SELECTS MOTION
   ├─ One of I 73.0-73.4 = 1  (Selects axis)
   └─ One of I 73.5-73.6 = 1  (Selects direction)

3. ELBO PROVIDES SPEED SIGNAL
   └─ I 73.7 = 1  (Valid speed signal from Elbo)

4. PLC PROCESSES LOGIC
   ├─ Check I 73.7 = 1? (Master gate)
   ├─ Check I 72.3 and I 72.7? (Safety interlocks)
   ├─ Select appropriate Q 6.x contactor
   └─ Energize contactor

5. MOTOR MOVES
   ├─ In selected direction (I 73.5 or I 73.6)
   ├─ At selected speed (I 72.0 or I 72.1)
   └─ Until operator releases or end switch hit

6. OPERATOR RELEASES CONTROL
   ├─ I 73.5 or I 73.6 = 0  (No direction)
   ├─ I 73.7 = 0            (Speed signal removed)
   └─ PLC de-energizes Q 6.x → Motor stops

7. SAFETY INTERLOCKS
   ├─ IF I 72.3 = 1 (table locked): Q 7.3 = 0 (oil pump off)
   ├─ IF I 72.7 = 1 (disk running): Prevent table moves
   └─ IF I 73.7 = 0 (no speed): All motion stops
```

---

## Signal Dependencies & Interlocks

```
INTERLOCKING RELATIONSHIPS:

I 72.3 (Table Lock) BLOCKS → Q 7.3 (Oil Pump)
I 72.7 (Disk Running) PREVENTS → Table movement (Q 7.3)
I 73.7 (Velocity Enable) GATES → ALL motion (Q 6.0-6.5)

MUTUAL EXCLUSIONS:
I 72.0 vs I 72.1  (Only one speed at a time)
I 73.0-73.4       (Only one axis at a time)
I 73.5 vs I 73.6  (Typically one direction or none)

REQUIRED CONDITIONS FOR MOTION:
I 73.7 = 1  AND  (I 73.0 OR I 73.1 OR I 73.2 OR I 73.3) AND (I 73.5 OR I 73.6)
     └─────────────────────┬───────────────────────────────────────────┘
                 THEN energize Q 6.x

SAFETY CONDITIONS:
IF I 72.3 = 1  THEN  BLOCK Q 7.3 = 0
IF I 72.7 = 1  THEN  FORCE Q 6.4 & 6.5 = 0  (unless in rotation mode)
```

---

## Fault Detection

### How to Detect Faults Using Elbo Signals

**Signal Mismatch Faults:**
```
If I 72.5 (VFD1 running) = 0 but Q 6.0 (contactor) = 1
  → Contactor may be stuck or VFD failed
  
If I 72.7 (Disk running) = 0 but operator requested disk motion
  → Disk motor failed to start
  → Check disk contactor Q 6.5
```

**Safety Fault:**
```
If I 72.3 = 1 (table locked) but table is moving
  → Lock mechanism failed
  → Hydraulic control failed
  → SAFETY FAULT - Stop operation
```

**Loss of Signal Fault:**
```
If I 73.7 = 0 suddenly during operation
  → Elbo controller lost communication
  → Proportional signal lost
  → System enters safe stop
  → Similar to emergency stop
```

---

## Comparing Signals by Importance

### 🔴 CRITICAL - Safety Signals (5)
1. **I 72.3** - Table Lock (23 uses) - Prevents cutting hazard
2. **I 73.7** - Velocity Enable (10 uses) - Master motion gate
3. **I 72.7** - Disk Running (9 uses) - Safety interlock
4. **I 73.5** - Direction Positive (17 uses) - Can cause motion
5. **I 73.6** - Direction Negative (17 uses) - Can cause motion

### 🟡 IMPORTANT - Functional Signals (7)
- I 73.0, 73.1, 73.2, 73.3 - Axis selection (7-16 uses each)
- I 72.0, 72.1 - Speed control (14, 5 uses)
- I 72.2 - Rotation mode (11 uses)

### 🟢 MONITORING - Status Signals (4)
- I 72.4, 72.5, 72.6 - VFD/pressure status (1-11 uses)
- I 73.4 - Disk rotation (indirect/inherited uses)

---

## Integration Example: How Signals Work Together

### Example 1: Cut Motion (Blade Down)
```
Operator Action: Selects vertical axis, pushes lever down at fast speed

Signal State:
  I 72.0 = 1    (Fast selected)
  I 72.3 = 0    (Table unlocked - OK during cut)
  I 72.7 = 1    (Disk is running - cutting in progress)
  I 73.0 = 1    (Vertical axis selected)
  I 73.6 = 1    (Down direction)
  I 73.7 = 1    (Valid speed signal)

PLC Logic:
  IF I 73.7 = 1 AND I 73.0 = 1 AND I 73.6 = 1 THEN
    IF I 72.3 = 0 THEN  (Table not locked is OK during cut)
      SET Q 6.2 = 1  (Energize Down contactor)
    END
  END

Result:
  Q 6.2 = 1 → Blade moves DOWN at FAST speed
```

### Example 2: Table Reposition (Lock Active)
```
Operator Action: After cut, wants to move material, locks table first

Signal State:
  I 72.3 = 1    (Table is LOCKED)
  I 72.7 = 0    (Disk is OFF - not cutting)

PLC Logic:
  IF I 72.3 = 1 THEN
    Q 7.3 = 0   (Force oil pump OFF)
  END

Result:
  Q 7.3 = 0 → Oil pump de-energized
  → Table CANNOT move (hydraulic pressure blocked)
  → SAFE STATE - Cannot move table while locked
  
Fix: Operator must unlock (I 72.3 = 0) before table can move
```

### Example 3: Safety - Loss of Speed Signal
```
Operator is cutting, suddenly Elbo loses signal

Signal State:
  I 73.7 = 0    (Velocity signal is LOST)
  (All other signals may still be active)

PLC Logic:
  IF I 73.7 = 0 THEN
    Q 6.0 = 0   (Clear forward)
    Q 6.1 = 0   (Clear left/right)
    Q 6.2 = 0   (Clear up/down)
    Q 6.3 = 0   (Clear tilt)
    Q 6.4 = 0   (Clear table rotation)
    Q 6.5 = 0   (Clear disk rotation)
  END

Result:
  All Q 6.x = 0 → ALL MOTORS STOP
  System enters safe state
  Like pushing emergency stop
```

---

## Typical Signal Combinations

```
NORMAL CUTTING OPERATION:
  I 72.0 = 1  (Fast)
  I 72.3 = 0  (Table free - can cut)
  I 72.7 = 1  (Disk spinning - cutting)
  I 73.0 = 1  (Vertical axis)
  I 73.5 or 73.6 = 1  (Up or down)
  I 73.7 = 1  (Speed signal active)
  ✓ MOTION ALLOWED

REPOSITIONING MATERIAL:
  I 72.3 = 1  (Table locked - no accidental cut)
  I 72.7 = 0  (Disk stopped - safe)
  I 73.1 = 1  (Translation axis)
  I 73.5 or 73.6 = 1  (Left or right)
  I 73.7 = 1  (Speed signal active)
  ✓ MOTION ALLOWED (but not disk-related)

EMERGENCY STOP:
  I 73.7 = 0  (No speed signal)
  Result: ALL Q 6.x = 0
  ✓ ALL MOTION HALTS IMMEDIATELY

UNSAFE CONDITION:
  I 72.3 = 1  AND  I 72.7 = 1
  (Table locked AND disk running)
  ✓ SYSTEM PREVENTS TABLE MOVE - Correct!
```

---

## Reference Card for Troubleshooting

```
"Motors won't move" Checklist:
  ☐ Is I 73.7 = 1? (If no, that's the problem!)
  ☐ Is I 73.0-73.4 = 1? (Is an axis selected?)
  ☐ Is I 73.5 or 73.6 = 1? (Is direction selected?)
  ☐ Is Q 6.x energizing? (Is output going out?)

"Table stuck during reposition" Checklist:
  ☐ Is I 72.3 = 0? (If = 1, table is locked!)
  ☐ Is I 72.7 = 0? (If = 1, disk running may prevent it)
  ☐ Is Q 7.3 energizing? (Is oil pump on?)

"Unexpected motion" Checklist:
  ☐ Are any I 73.5 or 73.6 signals stuck at 1?
  ☐ Is I 73.7 constantly 1 when should be 0?
  ☐ Check Elbo controller for stuck buttons
```

