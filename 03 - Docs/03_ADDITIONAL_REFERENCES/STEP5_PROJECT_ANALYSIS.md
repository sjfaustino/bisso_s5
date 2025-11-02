# BISSO Stone Bridge Saw - Step5 Project Analysis

## Project Overview

**Project Name:** BISSO@ST (Stone Bridge Saw - Portuguese: Bisso = Stone Saw)  
**PLC Type:** Siemens S5 (Step5/AWLEP)  
**Control System:** Hydraulic stone bridge saw with multiple axes  
**Status:** Operational control system with manual jogging and programmed sequences

---

## System Architecture

### Three Axes
1. **Vertical (Z-Axis)** - Blade Down/Up (I 3.0 / I 3.1)
2. **Horizontal Translation (Y-Axis)** - Left/Right (I 2.5 / I 2.4)
3. **Forward/Reverse (X-Axis)** - Front/Back (I 2.6 / I 2.7)

### Additional Movements
- **Tilt Axis** - Inclined positioning
- **Table Rotation** - Clockwise/Counter-clockwise
- **Disk Rotation** - Cutter disk rotation up/down
- **Table Up/Down** - Two independent tables with hydraulic lifting

---

## Current Control Interface

### Input Devices (Manipulos/Buttons)

**I 2.x - Translation Controls**
- I 2.0 = Program Manual Mode
- I 2.1 = Program Translation Mode
- I 2.2 = Program CT Mode (Passages Translation)
- I 2.3 = Program C Mode (Passages)
- I 2.4 = Right/Left Translator
- I 2.5 = Left/Right Translator
- I 2.6 = Forward/Advance
- I 2.7 = Backward/Retract

**I 3.x - Vertical & Rotation**
- I 3.0 = Down
- I 3.1 = Up
- I 3.2 = Table Rotation Clockwise
- I 3.3 = Table Rotation Counter-clockwise
- I 3.4 = Disk Rotation Down
- I 3.5 = Disk Rotation Up
- I 3.6 = Fast All (Fast speed for all movements)
- I 3.7 = Fast Command Button

**I 4.x - Motor Controls**
- I 4.0 = Disk ON
- I 4.1 = Disk OFF
- I 4.2 = Star-Delta / VFD selector
- I 4.3 = Tilt Vertical selector
- I 4.4 = Power ON/OFF
- I 4.5 = Speed + (increase)
- I 4.6 = Speed - (decrease)
- I 4.7 = Laser ON/OFF

**I 5.x - End Switches (Fim de Curso)**
- I 5.0 = Right Limit
- I 5.1 = Left Limit
- I 5.2 = Forward Limit
- I 5.3 = Backward Limit
- I 5.4 = Down Limit
- I 5.5 = Up Limit
- I 5.6 = Tilt Down Limit
- I 5.7 = Tilt Up Limit

**I 9.x - Table Controls**
- I 9.0 = Table2 End Switch
- I 9.1 = Table1 Up Button
- I 9.2 = Table1 Down Button
- I 9.3 = Table2 Up Button
- I 9.4 = Table2 Down Button

---

## Current Output/Relay Mappings

### Q 6.x - Main Motor Contactors
- Q 6.0 = Forward/Reverse Contactor (C3)
- Q 6.1 = Left/Right Translation Contactor (C4)
- Q 6.2 = Up/Down Vertical Contactor (C5)
- Q 6.3 = Tilt Contactor (C7)
- Q 6.4 = Table Rotation Contactor (C6)
- Q 6.5 = Disk Rotation Contactor (C10)
- Q 6.6 = Disk Star Connection (RL1)
- Q 6.7 = Disk Delta Connection (RL3)

### Q 7.x - Additional Controls
- Q 7.0 = Disk Line Contactor (RL2)
- Q 7.1 = VFD Selector (RL INV)
- Q 7.2 = Laser Contactor (C8)
- Q 7.3 = Hydraulic Oil Pump for Tables
- Q 7.4 = Power ON
- Q 7.6 = Inverter 1 Contactor (C1)
- Q 7.7 = Inverter 2 Contactor (C2)

### Q 8.x - Inverter Commands
- Q 8.0 = Inverter1 Forward
- Q 8.1 = Inverter1 Reverse
- Q 8.4 = Inverter2 Forward
- Q 8.5 = Inverter2 Reverse
- Q 8.7 = Table Lock

### Q 72-73.x - Lights & Relays
- Q 72.0 = Active Program Light
- Q 72.1 = Disk ON Light
- Q 72.2 = End-of-Course Light
- Q 72.5-72.7 = External Table Relays
- Q 73.0-73.7 = Various control relays

---

## Timers Configuration

| Timer | Type | Time | Purpose |
|-------|------|------|---------|
| T 0   | SF   | 60s  | Delayed ON |
| T 1   | SD   | 60s  | Delayed OFF (long) |
| T 2   | SP   | 8s   | Pulse |
| T 3   | SD   | 40s  | Medium delay |
| T 4   | SD   | 30s  | 30s delay |
| T 5-6 | SD   | 5s   | 5s delay |
| T 7   | SD   | 0.5s | Short pulse |
| T 8   | SD   | 2s   | 2s delay |
| T 9-10| SD   | 0.1s | Very short pulse |
| T 11  | SD   | 1.5s | 1.5s delay |
| T 14  | SE   | 30s  | Laser safety timer |
| T 15  | -    | 5m   | Laser 5-minute timer |
| T 17  | SE   | 2s   | Fast command timer |
| T 18  | SF   | 3s   | 3s delayed ON |
| T 25  | SE   | 30m  | Long running timer |
| T 26  | SD   | 120s | Table hydraulics timer |

---

## Counters

| Counter | Purpose |
|---------|---------|
| C 1     | Mnp Fast - 3x press to access speed levels |
| C 2     | Undefined |
| C 3     | Undefined |

---

## Data Words (DW) - Speed Values

**DW 0-17:** Speed settings storage (16 speed levels)
- Used for different movement speeds
- Likely stored as RPM or duty cycle percentages

**DW 20-44:** Read-Only data from Function Blocks
- Output data from FB10, FB11, FB12, FB13, FB15, FB18

---

## Key Functional Blocks (Inferred)

From the code references:
- **FB10** - Table movement logic
- **FB11** - Axis movement coordination
- **FB12-13** - Speed/acceleration calculations
- **FB15** - Timer and speed management
- **FB18** - Main sequencing

---

## Flags (Internal Memory)

### Program Mode Flags
- F 0.0 = Program Manual (F Prog M)
- F 0.1 = Program Translation (F Prog T)
- F 0.2 = Program CT (Passages Translation)
- F 0.3 = Program C (Passages)

### Speed Control Flags
- F 2.3 = +Speed Flag
- F 2.4 = -Speed Flag
- F 3.5 = Manipulo +Speed Pressed
- F 3.6 = Manipulo -Speed Pressed

### Operational Flags
- F 3.7 = Laser ON
- F 10.1-10.4 = Table vertical/horizontal states
- F 11.1-11.4 = Manipulo table vertical/horizontal states

---

## UART Communication Interface

**Current:** Interface = 15 (S5/PG interface)  
**Baud Rate:** 19200 (configurable)  
**Protocol:** X-protocol (Siemens PLC standard)  
**Status:** Configured but not actively monitoring via serial

### Configuration in BISSO@ST.INI:
- AG-Schnittstelle=15 (Interface 15 = Slot-based)
- Baudrate=19200
- S5Timeout=2500ms
- S7Timeout=2500ms
- Blockpause=220ms

---

## Proposed Nextion HMI Integration

### Communication Bridge

Instead of direct HMI connection, create a **communication adapter**:

```
Nextion Display (UART 115200)
         ↓
Adapter Controller (Arduino/Teensy)
         ↓
Step5 PLC (UART 19200)
```

### Alternative: Direct Connection

If your Step5 has a second serial port, you can:
1. Configure second UART for Nextion at 115200
2. Add communications block to Step5 program
3. Create data exchange protocol

### Data Mapping for Nextion

**To Nextion (Display Updates):**
```
POS_X : Current X position (from I 73.1 or calculated)
POS_Y : Current Y position (from I 73.2 or calculated)
POS_Z : Current Z position (from I 73.0 or calculated)
STATUS : System status (READY, MOVING, FAULT, etc.)
SPEED_LEVEL : Current speed (1-16)
LASER_STATUS : Laser ON/OFF (from F 3.7)
FAULT_CODE : Any error conditions
```

**From Nextion (Commands):**
```
MOVE_X : Target X position
MOVE_Y : Target Y position
MOVE_Z : Target Z position
MOVE_HOME : Home all axes
MOVE_STOP : Emergency stop
JOG_X_POS : Jog X positive
JOG_X_NEG : Jog X negative
(Similar for Y and Z)
SPEED_UP : Increase speed
SPEED_DOWN : Decrease speed
LASER_ON : Turn laser on
LASER_OFF : Turn laser off
```

---

## Recommendations for Your Project

### Phase 1: Direct Nextion Integration (Simple)
Add to your Step5 program:
1. Monitor Nextion UART input in a separate function block
2. Parse commands from HMI
3. Update axis control logic based on HMI input
4. Send current state back to display

### Phase 2: Advanced Positioning (Your Goal)
1. Implement absolute positioning counters (currently uses contactors/direction signals)
2. Add encoder feedback or stepper position tracking
3. Create GCode command processing
4. Build sequencing for complex cuts

### Phase 3: Automatic Programs
1. Store predefined sequences in DW arrays
2. Allow Nextion to select and execute programs
3. Implement progress monitoring

---

## Current Program Flow

Based on the inputs/outputs, the current system:

1. **Manual Mode** - Operator presses buttons → contactors fire → motors move → end switches stop
2. **Speed Control** - +/- speed buttons adjust variable frequency drive (VFD)
3. **Program Modes** - Preset programs execute sequences
4. **Safety** - End switches and timer-based limits prevent overtravel

---

## Files in Your Project

| File | Purpose |
|------|---------|
| BISSO_ST.S5D | Main program source |
| BISSO_ST.PAP | Backup source |
| BISSO_ST.QVL | Quality assurance data |
| BISSO_ST.BLG | Block list |
| BISSO_Z0.S5 | Organization unit |
| BISSO_Z0.SBK | Symbol/Name definitions |
| BISSO_Z0.SEQ | Symbol descriptions |
| BISSO_Z0.CF7 | Configuration |
| BISSO@ST.INI | Interface settings |

---

## Next Steps for HMI Integration

1. **Connect Nextion** to second serial port on Step5 or via adapter
2. **Add Communication Block** to handle UART data
3. **Modify Axis Handling** to accept position targets from HMI
4. **Implement Position Feedback** to display on screen
5. **Create GCode Interpreter** to convert HMI commands to contactor signals
6. **Test Safety** - emergency stop, limit switch override, etc.

---

## Questions for Your Implementation

1. Do you have encoder feedback on each axis? (Needed for absolute positioning)
2. Is there a second UART port available on your PLC?
3. What motion speeds are typical for each axis?
4. Are there specific GCode commands you need to support?
5. Do you need automatic homing sequences?

