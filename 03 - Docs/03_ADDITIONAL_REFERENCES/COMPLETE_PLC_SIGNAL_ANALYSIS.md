# BISSO STEP5 PLC - COMPLETE SIGNAL ANALYSIS
## All Inputs, Outputs, Flags, Timers, Counters & Data

Based on QVL cross-reference analysis of your BISSO@ST Step5 project

---

## TABLE OF CONTENTS

1. INPUT SIGNALS (I)
   - I 2.x - Manual Control Buttons
   - I 3.x - Vertical & Rotation Controls
   - I 4.x - Special Functions
   - I 5.x - End Switches / Limit Switches
   - I 9.x - Table Controls
   - I 72.x & I 73.x - Elbo Microcontroller (already analyzed)

2. OUTPUT SIGNALS (Q)
   - Q 6.x - Motor Contactors
   - Q 7.x - Special Relays & VFDs
   - Q 8.x - Inverter Commands
   - Q 72.x & Q 73.x - Status Lights & Relays

3. FLAGS (F)
   - F 0.x - Program Mode Flags
   - F 1.x - Motion Control Flags
   - F 2.x - Speed Control Flags
   - F 3.x - Special Function Flags
   - F 4.x & F 5.x - System Control Flags
   - F 6.x - Positioning Flags
   - F 8.x & F 9.x - Advanced Flags
   - F 10.x & F 11.x - Table Position Flags

4. TIMERS (T)

5. COUNTERS (C)

6. DATA WORDS (DW)

---

## PART 1: INPUT SIGNALS (I)

### I 2.x - MANUAL CONTROL BUTTONS (Program & Movement Selection)

#### **I 2.0** - Mnp Prog_M (Manual Program Mode Button)
- **Purpose:** Operator selects manual program mode
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/ON, 6/O, 6/ON
- **Total Uses:** 9 places
- **Function:** When pressed, selects manual mode from other program modes
- **Related:** F 0.0 (Program M flag)

#### **I 2.1** - Mnp Prog_T (Program Translation Button)
- **Purpose:** Select translation program (multi-pass horizontal movement)
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/ON
- **Total Uses:** 2 places
- **Function:** Switch to programmed translation sequences
- **Related:** F 0.1 (Program T flag)

#### **I 2.2** - Mnp Prog_CT (Program Passages Translation Button)
- **Purpose:** Select program for passages/cuts with translation
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/ON
- **Total Uses:** 2 places
- **Function:** Programmed multi-passage cuts with horizontal movement
- **Related:** F 0.2 (Program CT flag)

#### **I 2.3** - Mnp Prog_C (Program Passages Button)
- **Purpose:** Select program for standard passages/cuts
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/ON
- **Total Uses:** 2 places
- **Function:** Programmed multi-passage cutting sequence
- **Related:** F 0.3 (Program C flag)

#### **I 2.4** - Mnp Right (Translate Right Button)
- **Purpose:** Manual jog to move material right
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 1/O, 1/A, 1/O
- **Total Uses:** 4 places
- **Function:** Manually move material left/right translation axis
- **Effect:** Controls Q 6.1 (Left/Right contactor)
- **Related:** I 2.5 (Left button - opposite direction)

#### **I 2.5** - Mnp Left (Translate Left Button)
- **Purpose:** Manual jog to move material left
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 1/O, 1/A, 1/O
- **Total Uses:** 4 places
- **Function:** Manually move material in opposite direction to I 2.4
- **Effect:** Controls Q 6.1 (Left/Right contactor)
- **Interlock:** Both I 2.4 and I 2.5 cannot operate Q 6.1 simultaneously

#### **I 2.6** - Mnp Front (Advance Button)
- **Purpose:** Manual jog to advance material forward
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 1/O, 1/A, 1/A, 1/O
- **Total Uses:** 5 places
- **Function:** Manually move material forward (towards blade)
- **Effect:** Controls Q 6.0 (Forward contactor)
- **Related:** I 2.7 (Back button - opposite direction)

#### **I 2.7** - Mnp Back (Retract Button)
- **Purpose:** Manual jog to retract material backward
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 10/A, 1/O, 1/A, 1/A, 1/O
- **Total Uses:** 6 places
- **Function:** Manually move material backward (away from blade)
- **Effect:** Controls Q 6.0 (Reverse contactor)
- **Interlock:** Bidirectional control - either forward or backward, not both
- **Note:** Heavy use of AND NOT (1/AN) suggests safety checking

### I 3.x - VERTICAL & ROTATION CONTROL BUTTONS

#### **I 3.0** - Mnp Down (Blade Down Button)
- **Purpose:** Manual jog to lower cutting blade
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 1/O, 1/A, 1/O (twice)
- **Total Uses:** 5 places
- **Function:** Manually lower vertical cutting axis
- **Effect:** Controls Q 6.2 (Up/Down contactor)
- **Interlock:** Check for end switch I 5.4 (down limit)

#### **I 3.1** - Mnp Up (Blade Up Button)
- **Purpose:** Manual jog to raise cutting blade
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 1/O, 1/A, 1/O (twice)
- **Total Uses:** 5 places
- **Function:** Manually raise vertical cutting axis
- **Effect:** Controls Q 6.2 (Up/Down contactor)
- **Interlock:** Check for end switch I 5.5 (up limit)

#### **I 3.2** - Mnp RotMesHor (Table Rotation Clockwise Button)
- **Purpose:** Rotate work table clockwise
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 8/AN, 1/O
- **Total Uses:** 10 places
- **Function:** Manually rotate table position (CW direction)
- **Effect:** Controls Q 6.4 (Table Rotation contactor)
- **Requires:** I 72.2 = 1 (Rotation mode enabled)

#### **I 3.3** - Mnp RotMesAnt (Table Rotation Counter-Clockwise Button)
- **Purpose:** Rotate work table counter-clockwise
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN, 8/AN, 1/O
- **Total Uses:** 10 places
- **Function:** Manually rotate table position (CCW direction)
- **Effect:** Controls Q 6.4 (Table Rotation contactor) - opposite direction
- **Interlock:** Cannot operate CW and CCW simultaneously

#### **I 3.4** - Mnp RotDskDown (Disk Rotation Down Button)
- **Purpose:** Rotate cutting disk downward (or adjust speed)
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN
- **Total Uses:** 1 place
- **Function:** Manual control of disk/cutter rotation
- **Note:** Minimal use - likely not critical to main operations

#### **I 3.5** - Mnp RotDskUp (Disk Rotation Up Button)
- **Purpose:** Rotate cutting disk upward (or adjust speed)
- **Signal Type:** Button press
- **Cross-Ref:** 1/AN
- **Total Uses:** 1 place
- **Function:** Manual control of disk/cutter rotation
- **Note:** Minimal use

#### **I 3.6** - Mnp FastAll (All Movements Fast Button)
- **Purpose:** Enable fast speed for all axes
- **Signal Type:** Mode selector button
- **Cross-Ref:** 1/AN, 1/AN, 1/O, 1/A (x4), 1/A, 1/A, 1/A, 1/A
- **Total Uses:** 12 places
- **Function:** Override speed selection to run all axes at maximum speed
- **Effect:** Affects multiple speed-related flags
- **Priority:** May override normal speed settings

#### **I 3.7** - Mnp FastCmd (Fast Command Button)
- **Purpose:** Single button for rapid speed access
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/A, 1/AN, 1/AN, 1/O
- **Total Uses:** 5 places
- **Function:** Quick access to fast mode (alternative to I 3.6)
- **Related:** Timer T 17 (2 second timer for button debounce)

### I 4.x - SPECIAL FUNCTION BUTTONS

#### **I 4.0** - Mnp Disk_ON (Disk/Blade On Button)
- **Purpose:** Start cutting disk/blade
- **Signal Type:** Button press
- **Cross-Ref:** 2/A, 2/A
- **Total Uses:** 2 places
- **Function:** Energize cutting disk motor
- **Effect:** Activates Q 7.1 (VFD selector) and Q 6.5 (disk contactor)

#### **I 4.1** - Mnp Disk_OFF (Disk/Blade Off Button)
- **Purpose:** Stop cutting disk/blade
- **Signal Type:** Button press
- **Cross-Ref:** 2/ON, 2/ON
- **Total Uses:** 2 places
- **Function:** De-energize cutting disk motor
- **Effect:** Deactivates disk drive

#### **I 4.2** - Mnp StarDelta-VFD (Star-Delta / VFD Selector)
- **Purpose:** Choose motor starting method for disk (Star-Delta or VFD)
- **Signal Type:** Mode selector
- **Cross-Ref:** 2/A, 2/AN, 2/AN
- **Total Uses:** 3 places
- **Function:** Select soft-start method for disk motor
- **Affects:** Q 6.6 (Star contactor RL1) vs Q 7.1 (VFD selector)

#### **I 4.3** - Mnp Tilt-Vertical (Tilt/Vertical Selector)
- **Purpose:** Select between vertical movement and tilt movement
- **Signal Type:** Mode selector switch
- **Cross-Ref:** 6/A, 6/AN
- **Total Uses:** 6 places
- **Function:** Toggle between Z-axis vertical or tilt axis motion
- **Effect:** Routes up/down buttons to different motors

#### **I 4.4** - Mnp Power (Power ON/OFF)
- **Purpose:** Main power enable/disable
- **Signal Type:** Master switch
- **Cross-Ref:** 1/ON (x5), 1/ON, 1/AN, 2/ON, 3/A, 7/A, 7/ON, 7/AN, 2/ON
- **Total Uses:** 14+ places
- **Function:** Master power control - enables/disables all operations
- **Effect:** Q 7.4 = PowerOn (energizes all subsystems)
- **Critical:** Most used control input (14+ references)

#### **I 4.5** - Mnp + Speed (Speed Increase Button)
- **Purpose:** Increase cutting speed
- **Signal Type:** Button press
- **Cross-Ref:** 10/A, 10/AN, 10/A, 1/A
- **Total Uses:** 4 places
- **Function:** Increment speed level (1-16 stored in DW 0-17)
- **Effect:** Increases RPM of cutting disk via VFD
- **Related:** F 3.5 (Speed+ flag), C 1 (Speed level counter)

#### **I 4.6** - Mnp - Speed (Speed Decrease Button)
- **Purpose:** Decrease cutting speed
- **Signal Type:** Button press
- **Cross-Ref:** 10/AN, 10/A, 10/A, 1/A
- **Total Uses:** 4 places
- **Function:** Decrement speed level
- **Effect:** Decreases RPM of cutting disk via VFD
- **Related:** F 3.6 (Speed- flag)

#### **I 4.7** - Mnp Laser (Laser On/Off Button)
- **Purpose:** Enable/disable laser positioning guide
- **Signal Type:** Toggle button
- **Cross-Ref:** 1/AN, 1/A, 1/A
- **Total Uses:** 3 places
- **Function:** Turn laser guide on or off
- **Effect:** Q 7.2 (Laser contactor C8)
- **Related:** F 3.7 (Laser on flag), T 14 & T 15 (Laser safety timers)

### I 5.x - END SWITCHES / LIMIT SWITCHES (Critical Safety)

#### **I 5.0** - FC Right (Right Limit Switch)
- **Purpose:** Material has reached right limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 1/AN, 1/AN
- **Total Uses:** 3 places
- **Function:** Prevents over-travel to the right
- **Effect:** Forces Q 6.1 = 0 (stops left/right motion)
- **Action:** Stops material movement immediately

#### **I 5.1** - FC Left (Left Limit Switch)
- **Purpose:** Material has reached left limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 5/A, 1/AN, 1/AN
- **Total Uses:** 4 places
- **Function:** Prevents over-travel to the left
- **Effect:** Stops left/right motion
- **Interlock:** Works with I 5.0 (mutual exclusion)

#### **I 5.2** - FC Fwd (Forward Limit Switch)
- **Purpose:** Material has reached forward limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 5/A, 1/A, 1/AN, 1/A, 1/AN, 1/A (x2)
- **Total Uses:** 8 places
- **Function:** Prevents over-travel toward blade
- **Effect:** Stops forward motion (Q 6.0)
- **Critical:** Used frequently for safety

#### **I 5.3** - FC Back (Backward Limit Switch)
- **Purpose:** Material has reached backward limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 5/A, 1/A, 1/AN, 1/A, 1/AN, 1/A (x2)
- **Total Uses:** 8 places
- **Function:** Prevents over-travel away from blade
- **Effect:** Stops backward motion (Q 6.0 reverse)
- **Critical:** Used frequently for safety

#### **I 5.4** - FC Down (Down Limit Switch - Vertical)
- **Purpose:** Cutting blade has reached bottom limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 1/AN, 1/AN
- **Total Uses:** 3 places
- **Function:** Prevents blade from moving below safe lower limit
- **Effect:** Stops downward motion (Q 6.2)
- **Critical:** Prevents collision with table

#### **I 5.5** - FC Up (Up Limit Switch - Vertical)
- **Purpose:** Cutting blade has reached top limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 1/AN, 1/AN
- **Total Uses:** 3 places
- **Function:** Prevents blade from moving above upper limit
- **Effect:** Stops upward motion (Q 6.2)
- **Critical:** Prevents jam in machine housing

#### **I 5.6** - FC Tilt_Down (Tilt Down Limit Switch)
- **Purpose:** Blade has reached bottom tilt angle limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 1/AN, 1/AN
- **Total Uses:** 3 places
- **Function:** Prevents over-tilt in downward direction
- **Effect:** Stops tilt-down motion (Q 6.3)

#### **I 5.7** - FC Tilt_Up (Tilt Up Limit Switch)
- **Purpose:** Blade has reached top tilt angle limit
- **Signal Type:** Position sensor (Safety)
- **Cross-Ref:** 5/O, 1/AN, 1/AN
- **Total Uses:** 3 places
- **Function:** Prevents over-tilt in upward direction
- **Effect:** Stops tilt-up motion (Q 6.3)

### I 9.x - TABLE CONTROL BUTTONS

#### **I 9.0** - FC Table2_Axle (Table 2 End Switch)
- **Purpose:** Table 2 position sensor
- **Signal Type:** Position sensor
- **Cross-Ref:** 1/A, 1/A, 1/A, 1/AN, 1/AN
- **Total Uses:** 5 places
- **Function:** Detects when Table 2 reaches specific position
- **Related:** F 11.0 (Table 2 position flag)

#### **I 9.1** - Mnp Table1_Up (Raise Table 1 Button)
- **Purpose:** Manually raise work table 1 (hydraulic)
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/A, 1/AN
- **Total Uses:** 3 places
- **Function:** Activate hydraulic pump to lift Table 1
- **Effect:** Q 7.3 (Oil pump), Q 72.5 (Relay up)
- **Related:** F 10.1 (Table 1 up flag), I 72.3 (must not lock during this)

#### **I 9.2** - Mnp Table1_Down (Lower Table 1 Button)
- **Purpose:** Manually lower work table 1
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/AN, 1/A
- **Total Uses:** 3 places
- **Function:** Activate hydraulic pump to lower Table 1
- **Effect:** Q 7.3 (Oil pump), Q 72.7 (Relay down)
- **Related:** F 10.2 (Table 1 down flag)

#### **I 9.3** - Mnp Table2_Up (Raise Table 2 Button)
- **Purpose:** Manually raise work table 2
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/A, 1/AN
- **Total Uses:** 3 places
- **Function:** Activate hydraulic pump to lift Table 2
- **Effect:** Q 7.3 (Oil pump), Q 72.6 (Relay up)
- **Related:** F 10.3 (Table 2 up flag)

#### **I 9.4** - Mnp Table2_Down (Lower Table 2 Button)
- **Purpose:** Manually lower work table 2
- **Signal Type:** Button press
- **Cross-Ref:** 1/A, 1/AN, 1/A
- **Total Uses:** 3 places
- **Function:** Activate hydraulic pump to lower Table 2
- **Effect:** Q 7.3 (Oil pump), Q 72.7 (Relay down)
- **Related:** F 10.4 (Table 2 down flag)

### I 72.x & I 73.x - ELBO MICROCONTROLLER (Already Analyzed)

See: COMPLETE_ELBO_SIGNAL_ANALYSIS.md

---

## PART 2: OUTPUT SIGNALS (Q)

### Q 6.x - MOTOR CONTACTORS (Main Motion Control)

#### **Q 6.0** - Cnt Fwd/Rev (Forward/Reverse Contactor C3)
- **Purpose:** Control forward/reverse motion of cutting/advancement axis
- **Signal Type:** Contactor (relay coil energize/de-energize)
- **Cross-Ref:** 1/A, 5/A, 7/R, 7/O, 9/O (x2)
- **Total Uses:** 6+ places
- **Function:** Energize C3 contactor for forward movement
- **Interlocked With:** Q 6.1 (cannot both be 1 - bidirectional control)
- **Controlled By:** I 2.6 (Front), I 2.7 (Back), I 73.5, I 73.6, I 73.0

#### **Q 6.1** - Cnt Left/Right (Left/Right Translation Contactor C4)
- **Purpose:** Control left/right horizontal movement of material
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 1/A, 5/A, 6/O, 7/R, 7/O, 9/O
- **Total Uses:** 6+ places
- **Function:** Energize C4 contactor for horizontal translation
- **Bidirectional:** Works with pole reversal or dual coils
- **Controlled By:** I 2.4 (Right), I 2.5 (Left), I 73.5, I 73.6, I 73.1

#### **Q 6.2** - Cnt Up/Down (Vertical Contactor C5)
- **Purpose:** Control up/down motion of cutting blade (Z-axis)
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 5/A, 6/O, 7/R, 7/O, 9/O
- **Total Uses:** 5+ places
- **Function:** Energize C5 for vertical movement
- **Controlled By:** I 3.0 (Down), I 3.1 (Up), I 73.5, I 73.6, I 73.0
- **Safety:** Checked against end switches I 5.4, I 5.5

#### **Q 6.3** - Cnt Tilt (Tilt Contactor C7)
- **Purpose:** Control tilt angle of cutting blade
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 5/A, 7/R, 7/O, 9/O
- **Total Uses:** 4+ places
- **Function:** Energize C7 for angular blade positioning
- **Controlled By:** I 3.2, I 3.3 (when I 4.3=1), I 73.5, I 73.6, I 73.2

#### **Q 6.4** - Cnt RotTable (Table Rotation Contactor C6)
- **Purpose:** Control rotation of work table (clockwise/counter-clockwise)
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 1/O, 1/O
- **Total Uses:** 2 places
- **Function:** Energize C6 for table rotation motion
- **Requires:** I 72.2 = 1 (Rotation mode), I 73.3 = 1 (Axis selected)
- **Controlled By:** I 3.2 (CW), I 3.3 (CCW)

#### **Q 6.5** - Cnt RotDsk (Disk Rotation Contactor C10)
- **Purpose:** Control cutting disk rotation
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 1/O, 1/O
- **Total Uses:** 2 places
- **Function:** Energize C10 to spin cutting disk
- **Requires:** I 4.0 (Disk ON button)
- **Related:** Q 6.6, Q 6.7 (Star-Delta selection)
- **Safety:** Interlock with I 72.3 (prevents table move when disk running)

#### **Q 6.6** - Cnt Disk_Star (Disk Star Connection RL1)
- **Purpose:** Connect disk motor in Star configuration for soft start
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 2/= (assignment only)
- **Function:** Provide soft-start for disk motor (reduced current)
- **Mutually Exclusive:** Cannot be 1 when Q 6.7 = 1 (Delta connection)

#### **Q 6.7** - Cnt Disk_Delta (Disk Delta Connection RL3)
- **Purpose:** Connect disk motor in Delta configuration for full power
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 2/=, 4/A
- **Function:** Provide full-power connection to disk motor (after Star run)
- **Mutually Exclusive:** Cannot be 1 when Q 6.6 = 1

### Q 7.x - SPECIAL RELAYS & FUNCTIONS

#### **Q 7.0** - Cnt RL2 (Disk Line Motor Contactor RL2)
- **Purpose:** Main contactor for disk motor power line
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 2/= (assignment only)
- **Function:** Switch disk motor line connection

#### **Q 7.1** - Cnt Disk_VFD (VFD Selector Relay RL INV)
- **Purpose:** Select VFD (variable frequency drive) control for disk
- **Signal Type:** Relay coil
- **Cross-Ref:** 4/A
- **Function:** Switch motor control to VFD instead of Star-Delta
- **When Activated:** Disk speed controlled via VFD proportional input

#### **Q 7.2** - Cnt Laser (Laser Contactor C8)
- **Purpose:** Control laser positioning guide
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 1/AN, 1/= (assignment)
- **Function:** Energize laser module
- **Controlled By:** I 4.7 (Laser button)
- **Related:** T 14, T 15 (Laser safety timers - 30s and 5 minute)

#### **Q 7.3** - Oil Pump Relay (Hydraulic Pump Control) ⚠️ CRITICAL
- **Purpose:** Control hydraulic oil pump for table lift/lower systems
- **Signal Type:** Relay coil
- **Cross-Ref:** 1/=, 1/A, 1/A, 1/A, 1/A
- **Total Uses:** 5+ places
- **Function:** Energize pump motor when table movement needed
- **CRITICAL SAFETY:** 
  - Controlled by I 72.3 (Table lock) - when locked, pump is de-energized
  - Controlled by I 9.1-9.4 (Table up/down buttons)
  - When I 72.3 = 1: Q 7.3 = 0 (pump OFF - table locked)
  - When I 72.3 = 0: Q 7.3 can be activated
- **Effect:** Enables/disables all table positioning
- **Related:** Q 72.5, Q 72.6, Q 72.7 (directional relays)

#### **Q 7.4** - Power ON (Main Auxiliary Power)
- **Purpose:** Enable/disable all auxiliary power systems
- **Signal Type:** Relay/Contactor
- **Cross-Ref:** 2/= (assignment only)
- **Function:** Master power control
- **Controlled By:** I 4.4 (Power button)
- **Effect:** Powers all control systems when activated

#### **Q 7.5** - Q7.5 (Unused)
- **Status:** Not currently used
- **Cross-Ref:** 6/= (assignment only)

#### **Q 7.6** - Contactor (Reserved)
- **Status:** Reserved for future use or redundant
- **Cross-Ref:** 9/= (assignment)

#### **Q 7.7** - Cnt Inv2 (Inverter 2 Contactor C2)
- **Purpose:** Main power contactor for Inverter/VFD 2
- **Signal Type:** Contactor (relay coil)
- **Cross-Ref:** 9/= (assignment)
- **Function:** Enable/disable Inverter 2 (secondary drive)
- **Related:** I 72.6 (Inverter 2 running status)

### Q 8.x - INVERTER COMMANDS (Speed Control via VFD)

#### **Q 8.0** - Inv1_Fwd (Inverter 1 Forward Command)
- **Purpose:** Command Inverter 1 to rotate motor forward
- **Signal Type:** Digital command output
- **Cross-Ref:** 1/O, 7/O, 9/=, 9/O
- **Function:** Send forward command to primary VFD
- **Controlled By:** Q 6.0 (forward contactor status)

#### **Q 8.1** - Inv1_Rev (Inverter 1 Reverse Command)
- **Purpose:** Command Inverter 1 to rotate motor reverse
- **Signal Type:** Digital command output
- **Cross-Ref:** 7/O, 9/=, 9/O
- **Function:** Send reverse command to primary VFD
- **Controlled By:** Q 6.0 status (reverse mode)

#### **Q 8.2** - Q8.2 (Special Function - Likely Unused)
- **Cross-Ref:** 7/A, 9/=
- **Status:** Minimal use

#### **Q 8.3** - Q8.3 (Special Function - Likely Unused)
- **Cross-Ref:** 7/A, 9/=
- **Status:** Minimal use

#### **Q 8.4** - Inv2_Fwd (Inverter 2 Forward Command)
- **Purpose:** Command Inverter 2 to rotate forward
- **Signal Type:** Digital command output
- **Cross-Ref:** 1/O, 7/O, 9/=, 9/O
- **Function:** Send forward command to secondary VFD
- **Related:** Q 8.0 (similar function for VFD 2)

#### **Q 8.5** - Inv2_Rev (Inverter 2 Reverse Command)
- **Purpose:** Command Inverter 2 to rotate reverse
- **Signal Type:** Digital command output
- **Cross-Ref:** 7/O, 9/=, 9/O
- **Function:** Send reverse command to secondary VFD

#### **Q 8.6** - Motor Spindle (Spindle Motor Control - Via Reference)
- **Purpose:** Control spindle motor (disk/blade)
- **Signal Type:** Output with multiple modes
- **Cross-Ref:** 2/=, 2/O, 4/A, 4/A
- **Function:** Direct spindle control or speed reference

#### **Q 8.7** - LockTable (Table Lock Relay)
- **Purpose:** Electromechanical lock for table during cutting
- **Signal Type:** Relay/Solenoid
- **Cross-Ref:** 1/AN, 1/S, 1/A (x2)
- **Total Uses:** 4 places
- **Function:** Physically lock table position during active cutting
- **Controlled By:** I 72.3 (Cmd LockTable from Elbo)
- **Effect:** Prevents table movement when energized
- **Critical Safety:** Works with I 72.7 (Disk running) to prevent hazards

### Q 72.x & Q 73.x - STATUS LIGHTS & RELAYS

#### **Q 72.0** - Lamp ProgActiv (Program Active Light)
- **Purpose:** Indicator light showing a program is running
- **Signal Type:** LED indicator
- **Cross-Ref:** 1/= (assignment)
- **Function:** Visual feedback that programmed sequence is active
- **Controlled By:** Program mode flags (F 0.0-0.3)

#### **Q 72.1** - Lamp Disc_ON (Disk ON Light)
- **Purpose:** Indicator light showing cutting disk is running
- **Signal Type:** LED indicator
- **Cross-Ref:** 1/= (assignment)
- **Function:** Visual feedback that blade/disk is actively spinning
- **Controlled By:** I 4.0 or VFD running status

#### **Q 72.2** - Lamp FC (End-of-Course Light)
- **Purpose:** Indicator light for limit switch activated
- **Signal Type:** LED indicator
- **Cross-Ref:** 1/= (assignment)
- **Function:** Visual warning that a limit switch has been reached
- **Controlled By:** Any of I 5.x signals

#### **Q 72.4-72.7** - External Table Relays
- **Q 72.4** - (Reserved)
- **Q 72.5** - Rlay Tbl1_Up (Table 1 Raise Relay)
- **Q 72.6** - Rlay Tbl2_Up (Table 2 Raise Relay)
- **Q 72.7** - Rlay Tbl2_Down / Rlay Tbl1_Up (Table Down Relay)

#### **Q 73.0-73.7** - Elbo Output Signals
See: COMPLETE_ELBO_SIGNAL_ANALYSIS.md

---

## PART 3: FLAGS (F) - INTERNAL CONTROL LOGIC

### F 0.x - PROGRAM MODE FLAGS

#### **F 0.0** - F Prog M (Program Manual Mode Flag)
- **Purpose:** Indicates manual program mode is selected
- **Cross-Ref:** 8/AN, 8/A, 8/AN
- **Controlled By:** I 2.0 (Mnp Prog_M button)
- **Effect:** Enables manual axis control (not programmed)
- **Mutually Exclusive:** With F 0.1, 0.2, 0.3

#### **F 0.1** - F Prog T (Program Translation Flag)
- **Purpose:** Indicates translation program mode selected
- **Cross-Ref:** 3/O, 8/O, 8/O, 8/AN
- **Controlled By:** I 2.1 (Translation program button)
- **Effect:** Enables programmed horizontal movement sequences

#### **F 0.2** - F Prog CT (Program Passages Translation Flag)
- **Purpose:** Passages program with translation
- **Cross-Ref:** 8/AN, 10/O
- **Effect:** Multi-pass cutting with horizontal repositioning

#### **F 0.3** - F Prog C (Program Passages Flag)
- **Purpose:** Standard passages program mode
- **Cross-Ref:** 8/AN, 10/O
- **Effect:** Multi-pass cutting at fixed position

### F 1.x - MOTION CONTROL FLAGS (Advanced)

#### **F 1.0** - F Motion Sequence Control
- **Purpose:** Main motion sequence state machine
- **Cross-Ref:** 2/AN, 2/S, 2/A, 2/R, 2/A, 2/A, 2/A
- **Total Uses:** 7+ places
- **Function:** Coordinates programmed motion sequences
- **Heavy Interlock Logic:** Complex set/reset logic

#### **F 1.1, F 1.3, F 1.4, F 1.5** - Motion Sub-Sequences
- **Purpose:** Individual axes sequence control
- **Complex State Machines:** Each manages specific axis motion
- **Heavy Use:** Multiple conditional branches

### F 2.x - SPEED CONTROL FLAGS

#### **F 2.3** - F + Speed
- **Purpose:** Indicates speed-up command active
- **Cross-Ref:** 10/=, 1/=
- **Controlled By:** I 4.5 (Speed increase button)
- **Effect:** Increment speed level counter

#### **F 2.4** - F - Speed
- **Purpose:** Indicates speed-down command active
- **Cross-Ref:** 10/=, 1/=
- **Controlled By:** I 4.6 (Speed decrease button)
- **Effect:** Decrement speed level counter

#### **F 2.5** - F FB12 2.5 (Speed Calculation Flag)
- **Purpose:** Intermediate speed calculation
- **Cross-Ref:** 1/=, 1/AN
- **Function:** Used in speed ramping calculations

### F 3.x - SPECIAL FUNCTION FLAGS

#### **F 3.5** - F Mnp +Speed (Speed+ Button Pressed)
- **Purpose:** Track speed increase button pressed state
- **Cross-Ref:** 10/=, 10/O, 10/O, 10/O
- **Controlled By:** I 4.5 (Speed + button)
- **Effect:** Triggers speed increment logic

#### **F 3.6** - F Mnp -Speed (Speed- Button Pressed)
- **Purpose:** Track speed decrease button pressed state
- **Cross-Ref:** 10/=, 10/O, 10/O, 10/O
- **Controlled By:** I 4.6 (Speed - button)
- **Effect:** Triggers speed decrement logic

#### **F 3.7** - F Laser_ON (Laser Enabled)
- **Purpose:** Indicates laser positioning guide is active
- **Cross-Ref:** 1/A, 1/A
- **Controlled By:** I 4.7 (Laser button)
- **Related:** Q 7.2 (Laser contactor), T 14, T 15 (Laser safety timers)

### F 4.x - SYSTEM CONTROL & CALCULATION FLAGS

#### **F 4.0** - F Speed Calculation / Main Control Flag
- **Purpose:** Core system control state
- **Cross-Ref:** 5/AN, 9/A, 9/A, multiple 1/= assignments
- **Total Uses:** 10+ places
- **Function:** Central logic hub for speed and motion control
- **Heavily Used:** Complex state tracking

### F 5.x - ADVANCED CONTROL FLAGS

#### **F 5.0** - F System Ready / Interlock Check
- **Purpose:** System safety and ready state
- **Cross-Ref:** 1/AN, 6/AN, 6/AN, 7/=, 8/AN, 8/AN
- **Total Uses:** 8+ places
- **Function:** Verify all safety conditions met before operation

#### **F 5.3, F 5.4, F 5.6** - Complex Sequencing
- **Purpose:** Motion phase management
- **Cross-Ref:** Multiple 8/= assignments
- **Function:** State machine phases for programmed sequences

### F 6.x - POSITIONING & MOTION FLAGS

#### **F 6.0** - F PB10 6.0 (Axis Positioning)
- **Cross-Ref:** 1/=, 1/A, 1/A
- **Purpose:** Track positioning status

#### **F 6.1-6.7** - Individual Axis State Flags
- **Purpose:** Track each axis motion state
- **Each Has:** 1/= (assignment), 1/A, 1/A
- **Function:** State tracking for complex sequencing

### F 8.x & F 9.x - ADVANCED MOTION FLAGS

#### **F 8.5** - F FB11 8.5 (Speed Reference Update)
- **Cross-Ref:** 1/=, 1/A, 1/A, 1/AN, 1/A
- **Purpose:** Track when speed needs updating

#### **F 8.6** - F FB11 8.6 (Direction Change)
- **Cross-Ref:** 1/=, 1/O, 1/AN, 1/O
- **Purpose:** Coordinate direction reversals

#### **F 9.2** - F FB12 9.2 (Acceleration/Ramp)
- **Cross-Ref:** 1/=, 1/AN
- **Purpose:** Track ramp-up/down in progress

#### **F 9.4** - F Position Error
- **Cross-Ref:** 5/=, 5/A
- **Purpose:** Flag when position out of tolerance

### F 10.x & F 11.x - TABLE POSITION FLAGS

#### **F 10.0** - F FB18 FW10 (Table General Status)
- **Purpose:** Overall table state tracking

#### **F 10.1** - F Table1_Up (Table 1 in Up Position)
- **Purpose:** Table 1 raised/vertical state
- **Cross-Ref:** 1/=, 1/O, 1/A
- **Related:** I 9.1 (Table 1 up button)

#### **F 10.2** - F Table1_Down (Table 1 in Down Position)
- **Purpose:** Table 1 lowered/horizontal state
- **Cross-Ref:** 1/=, 1/O, 1/A
- **Related:** I 9.2 (Table 1 down button)

#### **F 10.3, F 10.4** - F Table2_Up, F Table2_Down
- **Purpose:** Similar table position tracking for Table 2

#### **F 11.0** - F FC Table2_Axle (Table 2 Position Sensor)
- **Cross-Ref:** 1/=
- **Purpose:** Track Table 2 end switch status

#### **F 11.1-11.4** - F MnpTable Positions
- **Purpose:** Manual table position requests
- **Tracks:** User button presses for table up/down

---

## PART 4: TIMERS (T)

| Timer | Type | Time | Purpose |
|-------|------|------|---------|
| **T 0** | SF | 60s | Initialization timer |
| **T 1** | SD | 60s | Long delay (general purpose) |
| **T 2** | SP | 8s | Pulse timer |
| **T 3** | SD | 40s | Motion timeout |
| **T 4** | SD | 30s | Medium delay |
| **T 5** | SD | 5s | General delay |
| **T 6** | SD | 5s | General delay |
| **T 7** | SD | 0.5s | Short pulse - Motion timing |
| **T 8** | SD | 2s | Debounce/settling |
| **T 9** | SD | 0.1s | Fast pulse |
| **T 10** | SD | 0.1s | Fast pulse |
| **T 11** | SD | 1.5s | Medium pulse |
| **T 13** | SD | 0.5s | Short pulse |
| **T 14** | SE | 30s | **Laser safety timer** |
| **T 15** | - | 5m | **Laser auto shut-off** |
| **T 17** | SE | 2s | Fast button debounce |
| **T 18** | SF | 3s | Delayed start |
| **T 19** | SD | var | Variable timer |
| **T 20** | SD | 2s | Medium delay |
| **T 21** | SD | 2s | Medium delay |
| **T 22** | SD | 5s | General delay |
| **T 24** | SE | 2s | Edge trigger delay |
| **T 25** | SE | 30m | **Long running safety timer** |
| **T 26** | SD | 120s | **Table hydraulic timer** |

### Timer Details

**T 7** - Motion Pulse Timing (0.5s)
- Used extensively in motion control (9 references)
- Generates precise motion timing pulses
- Cross-Ref: 1/A (x9)

**T 14** - Laser Safety Timer (30s)
- Prevents laser from running indefinitely
- Controlled by I 4.7 (Laser button)
- Auto-shutoff after 30 seconds

**T 15** - Laser Auto Shut-off (5 minutes)
- Secondary safety timer
- Hard limit on laser operation

**T 17** - Fast Button Timer (2s)
- Used for speed control debounce
- Cross-Ref: 1/AN, 1/SE, 1/A, 1/A

**T 26** - Table Hydraulics Timer (120s)
- Ensures hydraulic pump doesn't run continuously
- Cross-Ref: Implicit in table operations
- Safety: Prevents overheating

---

## PART 5: COUNTERS (C)

| Counter | Purpose | Total Uses |
|---------|---------|------------|
| **C 1** | C Mnp Fast - 3x counter for speed access | Primary |
| **C 2** | Speed level counter (0-16) | Implicit |
| **C 3** | Passage counter (multi-cut tracking) | Implicit |

### Counter Details

**C 1** - Fast Mode Counter (3x Press)
- Press speed button 3 times to cycle through speed levels
- Used for menu navigation in speed selection
- Cross-Ref: Implied in FB17-18

---

## PART 6: DATA WORDS (DW) - SPEED VALUES

| DW | Purpose | Notes |
|----|---------|-------|
| **DW 0** | DW Speed 0 - Lowest speed reference | 1st speed level |
| **DW 1-2** | DW1, DW2 - Reserved/Undefined | |
| **DW 3** | DW Speed 3 - 3rd speed level | 16 total levels |
| **DW 4** | DW Speed 4 - 4th speed level | |
| **DW 5** | DW5 - Undefined | |
| **DW 6-17** | DW Speed 6-17 - Speed levels 6-17 | |
| **DW 20-44** | Function Block Read-Only Data | Output from FB10-18 |

### Speed Data Storage

The system stores 16 different cutting speeds in DW 0-17. Each speed value represents:
- Percentage of maximum RPM for disk motor
- Feed rate for programmed sequences
- Cutting speed presets for different materials

**Speed Selection Logic:**
```
Speed Counter (C 1 or C 2) points to one of DW 0-17
Current speed = DW[Speed Level]
This value is:
  - Sent to VFD for disk speed control
  - Used for programmed sequence timing
  - Stored/recalled from presets
```

---

## COMPLETE SIGNAL SUMMARY TABLE

### By Signal Type & Usage

**MOST USED SIGNALS (10+ references):**
- I 4.4 (Power button) - 14+
- I 2.7 (Back button) - 6
- I 72.3 (Table lock) - 23 ⚠️
- I 73.7 (V/S enable) - 10 ⚠️
- F 1.0, F 1.3, F 4.0 (Motion control) - 7-10+ each
- F 5.0 (System ready) - 8+

**CRITICAL SAFETY SIGNALS:**
- I 5.0-5.7 (All end switches)
- I 72.3 (Table lock) - 23 uses
- I 72.7 (Disk running)
- I 73.7 (Velocity enable)
- Q 7.3 (Oil pump) - Controlled by I 72.3
- Q 8.7 (Table lock relay)

**CONTROL SEQUENCES:**
- Program Mode: F 0.0-0.3 (Mutually exclusive)
- Speed Control: F 2.3-2.4, C 1, DW 0-17
- Table Position: I 9.1-9.4, F 10.1-10.4, F 11.1-11.4
- Motion: Q 6.0-6.5 (Bidirectional contactors)

---

## CONTROL LOGIC OVERVIEW

```
INPUT → LOGIC → OUTPUT

Manual Jogging:
  I 2.4/2.5 (Right/Left) → FB routing → Q 6.1 (Translation contactor)
  I 3.0/3.1 (Up/Down) → FB routing → Q 6.2 (Vertical contactor)
  I 3.2/3.3 (CW/CCW) → FB routing → Q 6.4 (Rotation contactor)

Programmed Motion:
  I 2.0-2.3 (Program select) → F 0.0-0.3 (flags) → FB17-18 sequences
  
Safety Interlocks:
  I 5.x (End switches) → Stop respective motion
  I 72.3 (Table lock) → Block Q 7.3 (oil pump)
  I 72.7 (Disk running) + I 72.3 (lock) → Prevent table move

Speed Control:
  I 4.5/4.6 (Speed ±) → C 1 counter → DW 0-17 selection → Q 8.0-8.5 (VFD commands)

Table Operations:
  I 9.1-9.4 (Table buttons) → F 10.1-10.4 (states) → Q 7.3 (oil pump) + Q 72.5-72.7 (direction)
```

---

## KEY FINDINGS - ALL SIGNALS

**Most Critical (Safety):**
1. I 72.3 (Table lock) - 23 uses
2. I 5.x (End switches) - Multiple uses each
3. Q 7.3 (Oil pump) - Controls table safety
4. I 4.4 (Power) - Master control

**Most Complex Logic:**
1. Motion sequencing (F 1.x, 4.x, 5.x)
2. Speed calculation (F 2.x, DW 0-17)
3. Table positioning (F 10.x, 11.x)

**Rarely Used:**
- I 3.4, I 3.5 (Disk rotation buttons)
- I 72.4 (Pressure - only 1 use)
- Q 7.5, Q 7.6 (Reserved/unused)

