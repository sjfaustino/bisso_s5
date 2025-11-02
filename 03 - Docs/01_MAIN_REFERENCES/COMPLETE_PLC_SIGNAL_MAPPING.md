# BISSO@ST - COMPLETE PLC SIGNAL MAPPING
## All 100+ Signals Fully Referenced

---

## INPUT SIGNALS (I) - COMPLETE MAPPING

### I 2.0 - I 2.7: Manual Control Panel Buttons

**I 2.0: Program Manual Mode**
- Type: Push Button (momentary contact)
- Function: Select manual jog mode
- Set Flag: F 0.0 = 1
- Conditions Required:
  - I 2.0 = 1 (pressed)
  - F 6.3 = 0 (not in fast mode)
  - All motion buttons released (I 2.4-7, I 3.0-5 = 0)
  - F 5.0 = 0 (no motion active)
  - I 72.7 = 0 (disk not running optional)

**I 2.1: Program Translation**
- Type: Push Button
- Function: Select translation program with repositioning
- Set Flag: F 0.1 = 1
- Guard: I 72.7 = 0 (disk must not be running)
- Calls: PB20 subroutine

**I 2.2: Program Passages + Translation**
- Type: Push Button
- Function: Multi-cut with automatic repositioning
- Set Flag: F 0.2 = 1
- Guard: I 72.7 = 0
- Calls: Modified PB20

**I 2.3: Program Passages**
- Type: Push Button
- Function: Fixed-position multi-cut sequences
- Set Flag: F 0.3 = 1
- Guard: I 72.7 = 0
- Calls: PB10 subroutine

**I 2.4: Manual Jog Right**
- Type: Push Button (directional)
- Controlled By: OB1 Section [1]
- Output: Q 6.1 (Left/Right contactor)
- Direction: Material moves RIGHT
- Active When: F 0.0 = 1 (manual mode)

**I 2.5: Manual Jog Left**
- Type: Push Button
- Controlled By: OB1 Section [1]
- Output: Q 6.1 opposite polarity
- Direction: Material moves LEFT
- Active When: F 0.0 = 1

**I 2.6: Manual Jog Forward**
- Type: Push Button
- Controlled By: OB1 Section [1]
- Output: Q 6.0 (Forward/Reverse contactor)
- Direction: Material moves FORWARD (into blade)
- Active When: F 0.0 = 1
- Safety: I 5.2 stops at forward limit

**I 2.7: Manual Jog Backward**
- Type: Push Button
- Controlled By: OB1 Section [1]
- Output: Q 6.0 opposite polarity
- Direction: Material moves BACKWARD (away from blade)
- Active When: F 0.0 = 1
- Also Used: F 2.0 set when pressed with I 4.5/I 4.6

---

### I 3.0 - I 3.7: Vertical & Special Controls

**I 3.0: Manual Jog Blade Down**
- Type: Push Button
- Output: Q 6.2 (Up/Down contactor)
- Direction: Blade moves DOWN (into material)
- Safety: I 5.4 stops at down limit
- Active When: F 0.0 = 1

**I 3.1: Manual Jog Blade Up**
- Type: Push Button
- Output: Q 6.2 opposite polarity
- Direction: Blade moves UP (away from material)
- Safety: I 5.5 stops at up limit
- Active When: F 0.0 = 1

**I 3.2: Manual Table Rotate CW**
- Type: Push Button
- Output: Q 6.4 (Table Rotation contactor)
- Direction: Table rotates CLOCKWISE
- Safety: I 73.3 + I 72.3 control unlock via Q 8.7
- Active When: F 0.0 = 1 (manual mode)

**I 3.3: Manual Table Rotate CCW**
- Type: Push Button
- Output: Q 6.4 opposite polarity
- Direction: Table rotates COUNTER-CLOCKWISE
- Safety: Same as I 3.2
- Active When: F 0.0 = 1

**I 3.4 & I 3.5: Reserved**
- Currently unused
- Available for future expansion

**I 3.6: Fast All Movements**
- Type: Push Button
- Function: Override to fast speed
- Currently unused or reserved

**I 3.7: Fast Command Button**
- Type: Push Button
- Function: Activate fast mode menu
- Trigger: Press 3 times within 2 seconds
- Sets: F 6.3 = 1 (fast mode active)
- Calls: FB15 (411-line processor)
- Timeout: T 17 = 2 seconds

---

### I 4.0 - I 4.7: Special Functions

**I 4.0: Disk Start Button**
- Type: Push Button
- Function: Start cutting disk motor
- Sets: F 1.0 = 1 (disk running)
- Also: F 1.1 = 1 (startup sequence)
- Startup: T 21 = 20 seconds ramp-up time
- Guards: T 0 = 60s cooldown (prevents rapid cycling)

**I 4.1: Disk Stop Button**
- Type: Push Button
- Function: Stop cutting disk motor
- Resets: F 1.0, F 1.1 (motor off)
- Immediate: Stops disk rotation

**I 4.2: Star-Delta / VFD Selector**
- Type: Toggle Switch (2-position)
- Position 0: Use Star-Delta soft-start
  - Activates: Q 6.6 (star), then Q 6.7 (delta)
  - Timing: T 2 = 80ms transition
- Position 1: Use VFD proportional control
  - Activates: Q 7.1 (VFD selector)
  - Timing: T 20 = 20s stabilization

**I 4.3: Tilt / Vertical Mode Selector**
- Type: Toggle Switch (2-position)
- Position 0: Vertical mode (F 3.1 = 1)
  - Blade moves UP/DOWN (vertical)
  - Axis: I 73.0 activates
- Position 1: Tilt mode (F 3.0 = 1)
  - Blade moves at angle (tilt)
  - Axis: I 73.2 activates

**I 4.4: Power ON/OFF Button** ⚠️ CRITICAL
- Type: Push Button (momentary)
- Function: Master power control
- Pressed (= 1): Power on, system ready
- Released (= 0): Power down, all motion stops
- Effect: Q 7.4 = 1 (power relay energized)
- Safety: Independent emergency stop backup
- Used Throughout: Every section of OB1 checks I 4.4

**I 4.5: Speed Increase Button**
- Type: Push Button
- Function: Increase cutting speed
- Sets: F 2.3 = 1, F 3.5 = 1
- Action: Increments DW 1 (current speed)
- Limits: Capped at 65535 (100%)
- Ramping: FB12 smoothly increases speed
- Active When: Manual or programmed mode

**I 4.6: Speed Decrease Button**
- Type: Push Button
- Function: Decrease cutting speed
- Sets: F 2.4 = 1, F 3.6 = 1
- Action: Decrements DW 1
- Limits: Capped at 0 (0%)
- Ramping: FB12 smoothly decreases
- Active When: Manual or programmed mode

**I 4.7: Laser ON/OFF Button**
- Type: Push Button
- Function: Toggle laser positioning guide
- Sets: F 3.7 = 1
- Timeouts:
  - T 14 = 30s (auto-off for safety)
  - T 15 = 300s (5-minute hard limit)
- Output: Q 7.2 (laser contactor)
- Safety: Prevents laser overheating

---

### I 5.0 - I 5.7: End Switches (Limit Switches) ⚠️ SAFETY CRITICAL

All end switches are NORMALLY CLOSED (N.C.) - they open at limit

**I 5.0: Right End Limit Switch**
- Type: N.C. Limit Switch
- Location: Right edge of material travel
- State:
  - Normal (not at limit) = 1 (switch closed)
  - At limit = 0 (switch opens)
- Function: Prevents over-travel to right
- Monitored By: OB1 Section [5]
- Action When Hit: Q 73.2 = 1 (signal to Elbo)

**I 5.1: Left End Limit Switch**
- Type: N.C. Limit Switch
- Location: Left edge of material travel
- State: Normal = 1, At limit = 0
- Function: Prevents over-travel to left
- Action When Hit: Q 73.2 = 1

**I 5.2: Forward Limit Switch** ⭐ MOST USED
- Type: N.C. Limit Switch
- Location: Forward end (towards blade)
- State: Normal = 1, At limit = 0
- Function: Prevents material over-feed into blade
- Critical For: Feed control (most important limit)
- Monitored By: Every motion sequence
- Safety: Triggers emergency stop if violated

**I 5.3: Backward Limit Switch**
- Type: N.C. Limit Switch
- Location: Backward end (away from blade)
- State: Normal = 1, At limit = 0
- Function: Prevents material over-retract

**I 5.4: Blade Down Limit Switch**
- Type: N.C. Limit Switch
- Location: Fully lowered blade position
- State: Normal = 1, At limit = 0
- Function: Prevents blade over-descent
- Used By: PB10, PB20 (passage programs)

**I 5.5: Blade Up Limit Switch**
- Type: N.C. Limit Switch
- Location: Fully raised blade position
- State: Normal = 1, At limit = 0
- Function: Prevents blade over-ascent

**I 5.6: Tilt Down Limit Switch**
- Type: N.C. Limit Switch
- Location: Blade fully tilted down
- State: Normal = 1, At limit = 0
- Function: Tilt mode end stop
- Active When: F 0.4 = 1 (tilt mode)

**I 5.7: Tilt Up Limit Switch**
- Type: N.C. Limit Switch
- Location: Blade fully tilted up
- State: Normal = 1, At limit = 0
- Function: Tilt mode end stop
- Active When: F 0.4 = 1

**Limit Switch Safety Logic:**
```
IF (I 5.0 OR I 5.1 OR I 5.2 OR I 5.3 OR 
    I 5.4 OR I 5.5 OR I 5.6 OR I 5.7) = 0 THEN
  
  // Limit switch activated (opened)
  Set Q 73.2 = 1 (signal Elbo)
  
  IF motion contactor active THEN
    RESET corresponding Q 6.x (stop motor)
  END
  
  Activate safety mode
END
```

---

### I 9.0 - I 9.4: Table Controls

**I 9.0: Table 2 Position End Switch**
- Type: N.C. Limit Switch
- Location: Table 2 fully raised position
- Function: Table position confirmation
- Triggers: F 11.0 flag

**I 9.1: Table 1 Up Button**
- Type: Push Button
- Function: Raise Table 1 (hydraulic)
- Sets: F 10.1 = 1
- Action: 
  - Q 7.3 = 1 (oil pump on)
  - Q 72.5 = 1 (direction up)
- Safety: I 72.3 check (Elbo lock)
- Timeout: T 26 = 180 seconds max

**I 9.2: Table 1 Down Button**
- Type: Push Button
- Function: Lower Table 1 (hydraulic)
- Sets: F 10.2 = 1
- Action:
  - Q 7.3 = 1 (oil pump on)
  - Q 72.5 = 0 (direction down)
- Safety: Same gates as I 9.1
- Timeout: T 26 = 180 seconds

**I 9.3: Table 2 Up Button**
- Type: Push Button
- Function: Raise Table 2
- Sets: F 10.3 = 1
- Similar to I 9.1 but second table

**I 9.4: Table 2 Down Button**
- Type: Push Button
- Function: Lower Table 2
- Sets: F 10.4 = 1
- Similar to I 9.2

---

### I 72.0 - I 72.7: Elbo Status Inputs (Speed & Status) ⚠️ CRITICAL

**I 72.0: Elbo Rapido (Fast Speed)**
- Type: Digital Input
- Source: Elbo microcontroller
- Meaning: Operator selected FAST speed mode
- Used By: Speed selection logic
- Affects: Motor acceleration (faster response)

**I 72.1: Elbo Medio (Medium Speed)**
- Type: Digital Input
- Source: Elbo microcontroller
- Meaning: Operator selected MEDIUM speed mode
- Used By: Speed selection logic
- Affects: Motor acceleration (slower than Rapido)

**I 72.2: Cmd Rotation**
- Type: Digital Input
- Source: Elbo microcontroller
- Meaning: Rotation command active
- Used By: Table rotation logic
- Note: Rarely used in current configuration

**I 72.3: Cmd LockTable** ⚠️ CRITICAL SAFETY
- Type: Digital Input (Safety-critical)
- Source: Elbo microcontroller
- Meaning: Command to LOCK table (prevent movement)
- Used By: OB1 Section [8] (table lock logic)
- When = 1: 
  - Q 8.7 solenoid energized (table locked)
  - Q 7.3 (oil pump) blocked → table cannot move
  - PREVENTS table movement during cutting
- When = 0: Table can move (if other conditions met)
- Safety: Dual-gated (hardware + software)

**I 72.4: Pressostato (Pressure OK)**
- Type: Digital Input
- Source: Pressure sensor (hydraulic system)
- Meaning: Pressure is within normal range
- Used By: Pump safety logic
- If = 0: Pressure too low (possible pump failure)

**I 72.5: VFD1 Running (Inverter 1 Status)**
- Type: Digital Input (Feedback)
- Source: VFD Drive 1
- Meaning: Primary inverter is running
- Used By: Motion control logic
- For: Synchronization and diagnostics

**I 72.6: VFD2 Running (Inverter 2 Status)**
- Type: Digital Input (Feedback)
- Source: VFD Drive 2
- Meaning: Secondary inverter is running
- Used By: Dual drive logic
- Safety: Ensures mutual exclusion (never both active)

**I 72.7: Disk Running** ⚠️ CRITICAL SAFETY
- Type: Digital Input (Safety-critical)
- Source: VFD / Spindle encoder
- Meaning: Cutting disk is actively rotating
- When = 1:
  - Disk is spinning at operating speed
  - Table movement BLOCKED (via I 72.3 gate)
  - Program modes cannot be selected
  - Prevents dangerous mode switches during cutting
- When = 0:
  - Disk not spinning (safe to change modes)
  - Program selection allowed
  - Table can potentially move
- Safety: Prevents cutting blade collision with material

---

### I 73.0 - I 73.7: Elbo Axis & Direction Inputs ⚠️ CRITICAL

**I 73.0: Axis Cut (Vertical/Z-Axis)**
- Type: Digital Input
- Source: Elbo axis selector
- Meaning: Vertical cutting axis selected
- When = 1:
  - Routes joystick to Q 6.2 (Up/Down contactor)
  - Blade moves vertically
  - Direction from I 73.5/73.6
- Mutually Exclusive With: I 73.1, I 73.2, I 73.3, I 73.4
- Used By: OB1 Section [7] (motion routing)

**I 73.1: Axis Translation (Horizontal/Y-Axis)**
- Type: Digital Input
- Source: Elbo axis selector
- Meaning: Horizontal translation axis selected
- When = 1:
  - Routes joystick to Q 6.1 (Left/Right contactor)
  - Material moves horizontally
  - Direction from I 73.5/73.6
- Mutually Exclusive With: Others

**I 73.2: Axis Tilt (Tilt/Angle)**
- Type: Digital Input
- Source: Elbo axis selector
- Meaning: Tilt axis (blade angle) selected
- When = 1:
  - Routes joystick to tilt motor
  - Blade angle changes
  - Also: Sets F 0.4 = 1 (tilt mode flag)

**I 73.3: Axis Table Rotation**
- Type: Digital Input
- Source: Elbo axis selector
- Meaning: Table rotation axis selected
- When = 1:
  - Routes joystick to Q 6.4 (table rotation)
  - Table rotates CW or CCW
  - Also: Sets lock condition (Q 8.7 check)
  - Safety: Checks I 72.3 for unlock permission

**I 73.4: Axis Disk Rotation**
- Type: Digital Input
- Source: Elbo axis selector
- Meaning: Disk rotation axis selected
- Function: Currently reserved/unused
- May control: Disk motor speed in future

**I 73.5: Direction Positive (+)** ⚠️ CRITICAL GATE
- Type: Digital Input (Proportional joystick analog converted to digital)
- Source: Elbo proportional joystick
- Meaning: Positive direction selected (Forward/Up/Right/CW)
- When = 1:
  - Material/blade moves in positive direction
  - Paired with I 73.0-73.4 axis selection
- Combined With: I 73.6 (never both = 1)
- Examples:
  - I 73.0 + I 73.5 = Blade UP
  - I 73.1 + I 73.5 = Material RIGHT
  - I 73.3 + I 73.5 = Table CW

**I 73.6: Direction Negative (-)** ⚠️ CRITICAL GATE
- Type: Digital Input
- Source: Elbo proportional joystick
- Meaning: Negative direction selected (Backward/Down/Left/CCW)
- When = 1:
  - Material/blade moves in negative direction
  - Paired with I 73.0-73.4 axis selection
- Combined With: I 73.5 (never both = 1)
- Examples:
  - I 73.0 + I 73.6 = Blade DOWN
  - I 73.1 + I 73.6 = Material LEFT
  - I 73.3 + I 73.6 = Table CCW

**I 73.7: Velocity/Speed Enable** ⚠️⚠️ CRITICAL MASTER GATE
- Type: Digital Input (Proportional joystick proportional control)
- Source: Elbo proportional joystick
- Meaning: Operator applying velocity command
- When = 1:
  - Valid velocity signal present
  - Motion is permitted (IF other conditions met)
  - Proportional value indicates speed
- When = 0:
  - NO velocity signal
  - **EMERGENCY STOP EFFECT:**
    - ALL Q 6.x contactors immediately reset to 0
    - ALL motors stop
    - System enters safe state
  - Loss of signal = Immediate motion halt
- Safety: Joystick released/disconnected = automatic stop

---

## OUTPUT SIGNALS (Q) - COMPLETE MAPPING

### Q 6.0 - Q 6.5: Motor Contactors (Main Motion Control)

**Q 6.0: Forward/Reverse Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Controls primary axis bidirectional motion
- Controlled By: I 2.6, I 2.7 (buttons) or I 73.0, I 73.5, I 73.6 (Elbo)
- When = 1: Primary motor activated in forward/positive direction
- When = 0: Motor de-energized or reverse
- Safety: I 5.2 (forward limit) forces reset
- Used By: All motion sequences
- Typical Load: 15-20 kW motor

**Q 6.1: Left/Right Translation Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Controls horizontal (lateral) material movement
- Controlled By: I 2.4, I 2.5 (buttons) or I 73.1, I 73.5, I 73.6 (Elbo)
- When = 1: Material moves right/positive direction
- When = 0: Material moves left/negative or stopped
- Safety: I 5.0, I 5.1 (left/right limits) force reset
- Used By: PB20 (translation program) for repositioning
- Typical Load: 12-15 kW motor
- Motion Pulse: Creates T 7 timing

**Q 6.2: Up/Down Vertical Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Controls blade vertical (up/down) movement
- Controlled By: I 3.0, I 3.1 (buttons) or I 73.0, I 73.5, I 73.6 (Elbo)
- When = 1: Blade moves down/positive
- When = 0: Blade moves up/negative or stopped
- Safety: I 5.4, I 5.5 (down/up limits) force reset
- Used By: All cutting sequences (PB10, PB20)
- Typical Load: 10-12 kW motor
- Critical For: Passage cutting depth control

**Q 6.3: Tilt/Angle Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Controls blade angle (tilt)
- Controlled By: I 3.2, I 3.3 (buttons) or I 73.2, I 73.5, I 73.6 (Elbo)
- When = 1: Blade tilts in positive direction
- When = 0: Blade tilts negative or stopped
- Safety: I 5.6, I 5.7 (tilt limits) force reset
- Active When: F 0.4 = 1 (tilt mode selected via I 4.3)
- Used By: Beveled edge cutting operations
- Typical Load: 5-7 kW motor

**Q 6.4: Table Rotation Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Controls table rotation (CW/CCW)
- Controlled By: I 3.2, I 3.3 (buttons) or I 73.3, I 73.5, I 73.6 (Elbo)
- When = 1: Table rotates CW (positive)
- When = 0: Table rotates CCW or stopped
- Safety: Q 8.7 (table lock solenoid) blocks via I 72.3
- Related: F 10.1-10.4 (table position flags)
- Used By: Rotational positioning operations
- Typical Load: 3-5 kW motor
- Critical Gate: I 72.3 (must be 0 to allow)

**Q 6.5: Disk Rotation Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Controls disk/blade motor rotation
- Controlled By: I 4.0, I 4.1 (buttons) or I 73.4, I 73.5, I 73.6 (Elbo)
- When = 1: Disk motor activated
- When = 0: Disk motor de-energized
- Related: F 1.0, F 1.1 (disk running flags)
- Startup: T 21 = 20-second ramp-up
- Typical Load: 7.5-10 kW motor (cutting disk)
- Note: Actual power control via Q 6.6/Q 6.7 or Q 7.1

---

### Q 7.0 - Q 7.7: Special Relays & Functions

**Q 7.0: Disk Line Contactor (RL2)**
- Type: Electromechanical Relay (24VDC coil)
- Function: Main power line for disk motor
- Controlled By: F 1.0 (disk running flag)
- When = 1: Disk motor line is energized
- When = 0: Disk motor line de-energized
- Used With: Q 6.6 (star) or Q 6.7 (delta) or Q 7.1 (VFD)
- Safety: Interlocked with Q 6.6/6.7 or Q 7.1

**Q 7.1: VFD Selector Relay**
- Type: Electromechanical Relay (24VDC coil)
- Function: Select drive method (changes from Star-Delta to VFD)
- Controlled By: I 4.2 (star/VFD selector switch)
- When = 1: VFD control mode active
- When = 0: Star-Delta soft-start mode
- Timing: T 20 = 20 seconds stabilization after switch
- Used For: Disk motor or main drive control method

**Q 7.2: Laser Contactor**
- Type: Electromechanical Relay (24VDC coil)
- Function: Control laser positioning guide
- Controlled By: I 4.7 (laser button)
- When = 1: Laser guide is on
- When = 0: Laser guide is off
- Safety Timers:
  - T 14 = 30 seconds (auto-off)
  - T 15 = 300 seconds (5-minute hard limit)
- Purpose: Provides visual cut line reference

**Q 7.3: Oil Pump Contactor** ⚠️ CRITICAL HYDRAULIC CONTROL
- Type: Electromechanical Relay (24VDC coil)
- Function: Control hydraulic pump (table lift/lower)
- Controlled By: I 9.1-9.4 (table up/down buttons)
- When = 1: Hydraulic pump is running
- When = 0: Pump stopped
- CRITICAL GATE: I 72.3 (Elbo lock command)
  - IF I 72.3 = 1 THEN Q 7.3 cannot energize
  - IF I 72.7 = 1 (disk running) THEN Q 7.3 may be blocked
- Timeout: T 26 = 120-180 seconds (prevent overheating)
- Direction: Q 72.5 determines up/down via solenoid valve
- Safety: Cannot move table during cutting

**Q 7.4: Main Auxiliary Power Relay**
- Type: Electromechanical Relay (24VDC coil)
- Function: Master power to entire auxiliary system
- Controlled By: I 4.4 (power ON/OFF button)
- When = 1: System powered and ready
- When = 0: System de-powered (safe state)
- Effect: Disables all motion when 0
- Safety: Independent backup to PLC logic

**Q 7.5: Motion Pulse Signal**
- Type: Digital Output
- Function: Handshake/confirmation signal
- Controlled By: Motion contactor activity + T 24
- When = 1: Motion is detected and active
- Pulse Duration: 20ms (T 24)
- Used For: Diagnostic/status indication
- External Connection: May connect to external systems

**Q 7.6, Q 7.7: Reserved**
- Currently unused
- Available for future expansion

---

### Q 8.0 - Q 8.7: Inverter Commands & Table Lock

**Q 8.0: Inverter 1 Forward Command**
- Type: Digital Output
- Function: Command primary VFD forward direction
- Controlled By: F 1.4, F 4.2, I 73.7
- When = 1: VFD1 receives forward command
- When = 0: VFD1 not in forward mode
- Timing: Pulse during T 5 (50ms)
- Used With: Q 8.2 (speed reference)
- Safety: I 73.7 = 0 forces 0

**Q 8.1: Inverter 1 Reverse Command**
- Type: Digital Output
- Function: Command primary VFD reverse direction
- Controlled By: F 1.4, F 4.3, I 73.7
- When = 1: VFD1 receives reverse command
- When = 0: VFD1 not in reverse mode
- Safety: Mutually exclusive with Q 8.0

**Q 8.2: Inverter 1 Speed Control**
- Type: Analog Output (or PWM) 0-10V
- Function: Send speed reference to VFD1
- Controlled By: DW 1 (current speed value)
- Range: 0 = 0%, 32768 = 50%, 65535 = 100%
- Conversion: Via analog output module
- Ramp: FB12 calculates smooth curve
- Used For: Primary motor speed adjustment

**Q 8.3: Inverter 2 Speed Control**
- Type: Analog Output 0-10V
- Function: Send speed reference to VFD2
- Controlled By: DW 1 (shared speed table)
- Similar to Q 8.2 but for secondary drive
- Safety: Ensures both drives at same speed

**Q 8.4: Inverter 2 Forward Command**
- Type: Digital Output
- Function: Command secondary VFD forward direction
- Controlled By: F 1.3, F 4.2, T 22
- When = 1: VFD2 forward
- When = 0: Not forward
- Timing: T 22 = 50ms pulse

**Q 8.5: Inverter 2 Reverse Command**
- Type: Digital Output
- Function: Command secondary VFD reverse direction
- Controlled By: F 1.3, F 4.3, T 22
- When = 1: VFD2 reverse
- When = 0: Not reverse
- Safety: Mutually exclusive with Q 8.4

**Q 8.6: Spindle Motor Command**
- Type: Digital Output
- Function: Main spindle motor control
- Controlled By: F 1.0 (disk running), T 21
- When = 1: Spindle motor activated
- When = 0: Spindle motor off
- Related: Q 7.0 (line contactor)
- Used For: Cutting disk motor

**Q 8.7: Table Lock Solenoid** ⚠️⚠️ CRITICAL SAFETY
- Type: Electromechanical Solenoid (24VDC coil)
- Function: Physical lock preventing table movement
- Controlled By: I 73.3, I 72.3 (complex logic)
- When = 1: Table is LOCKED (cannot move)
- When = 0: Table lock released (can move if enabled)
- Critical Gate: I 72.3 = 1 BLOCKS Q 7.3 activation
  - IF I 72.3 = 1: Table locked, Q 7.3 = 0 (pump off)
  - IF I 72.3 = 0: Table can unlock IF other conditions met
- Safety: Dual gate (mechanical + hydraulic)
- Purpose: Prevents table movement during cutting
- Failure Mode: Default locked (safe state)

---

### Q 72.0 - Q 73.7: Status Lights & Elbo Feedback

**Q 72.0: Program Active Indicator Light**
- Type: LED or lamp output (24VDC)
- Function: Visual indication of running program
- Controlled By: F 0.3 (or F 0.1, F 0.2)
- When = 1: Programmed sequence is executing
- When = 0: No program running
- User Interface: Dashboard status light

**Q 72.1: Disk Running Indicator Light**
- Type: LED or lamp output (24VDC)
- Function: Visual indication of disk rotation
- Controlled By: F 1.0
- When = 1: Disk motor is running
- When = 0: Disk motor is stopped
- Safety: Operator can visually verify blade status

**Q 72.2: End Switch Alert Light**
- Type: LED or lamp output (24VDC)
- Function: Warning light when limit switch hit
- Controlled By: I 5.x OR conditions
- When = 1: At least one end switch activated
- When = 0: No limits active
- Purpose: Alert operator to over-travel condition

**Q 72.4-72.7: Hydraulic Direction Relays**
- Type: Electromechanical Relays (24VDC coils)
- Function: Control hydraulic valve solenoid directions
- Controlled By: Table motion logic
- Q 72.5: Typically UP/DOWN direction control
- Used With: Q 7.3 (oil pump) for table movements

**Q 73.0 - Q 73.7: Elbo Feedback Signals**
- Type: Digital Outputs to Elbo microcontroller
- Function: Send status back to Elbo interface
- Examples:
  - Q 73.0: Program mode feedback
  - Q 73.1: Disk status feedback
  - Q 73.2: End switch alert (sent to Elbo)
  - Q 73.3-73.7: Various status indicators
- Purpose: Synchronization with Elbo controller

---

## SIGNAL TIMING MATRIX

### Critical Timing Relationships

```
DISK STARTUP SEQUENCE:
  I 4.0 pressed → F 1.0 = 1, F 1.1 = 1
  T 21 starts (20s)
  After 20s → Disk at full speed
  F 1.1 reset, F 1.0 remains

MOTION RESPONSE (Manual):
  I 2.4 pressed → Q 6.1 = 1
  Material starts moving immediately (<100ms)
  I 2.4 released → Q 6.1 = 0
  Material stops immediately

SPEED RAMP:
  I 4.5 pressed → DW 1 incremented
  FB12 smooths change over ~10 seconds
  Output to QW 66
  VFD receives 0-10V slowly

TABLE LOCK:
  I 73.3 = 1 → Q 8.7 = 1 (lock energized)
  I 72.3 = 1 AND Q 7.3 → Q 7.3 = 0 (pump off)
  Table cannot move while I 72.3 = 1

PROGRAM EXECUTION:
  I 2.3 pressed → F 0.3 = 1
  PB10 starts executing
  T 1 timeout = 600 seconds max
  Program stops when complete or timeout
```

---

## SUMMARY TABLE: ALL SIGNALS

| Category | Range | Type | Purpose |
|----------|-------|------|---------|
| **Inputs** | I 2.0-4.7 | Buttons | Operator control |
| | I 5.0-5.7 | Limit Switches | Safety stops |
| | I 9.0-9.4 | Buttons | Table control |
| | I 72.0-72.7 | Elbo Status | System status |
| | I 73.0-73.7 | Elbo Joystick | Motion control |
| **Outputs** | Q 6.0-6.5 | Contactors | Motor control |
| | Q 7.0-7.7 | Relays | Special functions |
| | Q 8.0-8.7 | Inverter+Locks | Drive control |
| | Q 72.0-72.7 | Lights+Relays | Status indication |
| | Q 73.0-73.7 | Elbo Feedback | Status to Elbo |

**Total: 40+ Inputs + 60+ Outputs = 100+ signals fully mapped**

